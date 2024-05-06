# Implementation

> The implementation of system calls is platform dependent, hence refer to the platform-related headers for your platform implementation. In all the notes, I have used x86/x86_64 as the platform

- Each syscall is assigned a `syscall number` that uniquely identifies that syscall
- The parameters of the system calls are `word sized` (32 bit or 64bit)
- There can be a maximum of **6** parameters (x86/x86_64).
- The return type is of type `long` in the kernel to provide support for both x32 and x64.

> During a syscall the process runs in kernel mode (since this is in process context [current](../../misc/current.md) is legal here). For switching to the kernel mode, the user stack is stored and the kernel stack is loaded.

## Step 1 : Assembly code

- The mechanism needs to signal the kernel. This signal is called a `software interrupt`.
- The kernel on receive this interrupt executes the exception handler that is the `syscall handler` in this case.

The assembly instructions for this is defined under `arch/x86/entry/entry_64.S` for x86_64 systems. This file clearly mentions that a syscall for this architecture can have upto **6** arguments in registers. 

The following notes are present in the comments of `entry_SYSCALL_64` assembly code.

```assembly
64-bit SYSCALL instruction entry. Up to 6 arguments in registers.
This is the only entry point used for 64-bit system calls.  The
hardware interface is reasonably well designed and the register to
argument mapping Linux uses fits well with the registers that are
available when SYSCALL is used.
... 
Registers on entry:
rax  system call number
rcx  return address
r11  saved rflags (note: r11 is callee-clobbered register in C ABI)
rdi  arg0
rsi  arg1
rdx  arg2
r10  arg3 (needs to be moved to rcx to conform to C ABI)
r8   arg4
r9   arg5
(note: r12-r15, rbp, rbx are callee-preserved in C ABI)
...
```

The steps to make a system call are :
1. swapping the `GS` register using `swapgs`. The **GS** register holds data – in user mode it stores the base address of the user-space, for kernel-space it stores the per-CPU structure.
2. Save the user stack, this is done using macros. `PUSH_AND_CLEAR_REGS rax=$-ENOSYS` calls `PUSH_REGS` and then clears the registers using `CLEAR_REGS`.
```nasm
.macro PUSH_REGS rdx=%rdx rcx=%rcx rax=%rax save_ret=0 unwind_hint=1
	.if \save_ret
	pushq	%rsi		/* pt_regs->si */
	movq	8(%rsp), %rsi	/* temporarily store the return address in %rsi */
	movq	%rdi, 8(%rsp)	/* pt_regs->di (overwriting original return address) */
	.else
	pushq   %rdi		/* pt_regs->di */
	pushq   %rsi		/* pt_regs->si */
	.endif
	pushq	\rdx		/* pt_regs->dx */
	pushq   \rcx		/* pt_regs->cx */
	pushq   \rax		/* pt_regs->ax */
	pushq   %r8		/* pt_regs->r8 */
	pushq   %r9		/* pt_regs->r9 */
	pushq   %r10		/* pt_regs->r10 */
	pushq   %r11		/* pt_regs->r11 */
	pushq	%rbx		/* pt_regs->rbx */
	pushq	%rbp		/* pt_regs->rbp */
	pushq	%r12		/* pt_regs->r12 */
	pushq	%r13		/* pt_regs->r13 */
	pushq	%r14		/* pt_regs->r14 */
	pushq	%r15		/* pt_regs->r15 */

	.if \unwind_hint
	UNWIND_HINT_REGS
	.endif

	.if \save_ret
	pushq	%rsi		/* return address on top of stack */
	.endif
.endm
```
3. calls the syscall entry fuunction using the instruction `call do_syscall_65`.

## Step 2 : Entry function

The C code for this is present under `arch/x86/entry/common.c`. The function is `do_syscall_64`. This function disables instrumentation using `noinstr` ([noinstr doc](../../misc/noinstr.md)) in it’s declaration.

The C code first calls `syscall_enter_from_user_mode` which does the following :
1. Sets up RCU / Context Tracking and Tracing.
2. Invokes various entry work functions like ptrace, seccomp, audit, syscall tracing, etc.

Once this is done, it calls the syscall invocations. 
```c
	if (!do_syscall_x64(regs, nr) && !do_syscall_x32(regs, nr) && nr != -1) {
		/* Invalid system call, but still a system call. */
			regs->ax = __x64_sys_ni_syscall(regs);
	}
```

## Step 3 : Finding the syscall

The function `do_syscall_x64`  has the following code :
```c
static __always_inline bool do_syscall_x64(struct pt_regs *regs, int nr)
{
	/*
	 * Convert negative numbers to very high and thus out of range
	 * numbers for comparisons.
	 */
	unsigned int unr = nr;

	if (likely(unr < NR_syscalls)) {
		unr = array_index_nospec(unr, NR_syscalls);
		regs->ax = sys_call_table[unr](regs);
		return true;
	}
	return false;
}
```

- Here `sys_call_table` is very important. 
- It is an array that stores the various system call handlers
- It is defined in `arch/x86/entry/syscall_64.c` as : 
```c
asmlinkage const sys_call_ptr_t sys_call_table[] = {
#include <asm/syscalls_64.h>
};
```

- The handlers are defined under `asm/syscalls_64.h` as :
```c
__SYSCALL(0, sys_read)
__SYSCALL(1, sys_write)
__SYSCALL(2, sys_open)
__SYSCALL(3, sys_close)
__SYSCALL(4, sys_newstat)
__SYSCALL(5, sys_newfstat)
```

- Here `__SYSCALL` just expands to `__x64_##function`. Eg : `__x64_sys_read`

## Step 4 : The system calls

- The system calls are declared under `include/linux/sycalls.h`
- The definition is under their respective subsystems. For example : `sys_read` is defined under `fs/read_write.c` as : 
```c
ssize_t ksys_read(unsigned int fd, char __user *buf, size_t count)
{
	struct fd f = fdget_pos(fd);
	ssize_t ret = -EBADF;

	if (f.file) {
		loff_t pos, *ppos = file_ppos(f.file);
		if (ppos) {
			pos = *ppos;
			ppos = &pos;
		}
		ret = vfs_read(f.file, buf, count, ppos);
		if (ret >= 0 && ppos)
			f.file->f_pos = pos;
		fdput_pos(f);
	}
	return ret;
}

...

SYSCALL_DEFINE3(read, unsigned int, fd, char __user *, buf, size_t, count)
{
	return ksys_read(fd, buf, count);
}
```

## Step 5 : Returning from syscalls

- The `do_syscall_64` and `do_syscall_x64` mention earlier store the return values in the eax/rax registers : `regs->ax = sys_call_table[unr](regs);` as mentioned earler.
- The control then returns back to the assembly code under `entry_64.S` which then restores the registers backed-up earlier and then returns using `sysretq`.

## References
- [Syscalls - Linux Kernel Labs](https://linux-kernel-labs.github.io/refs/heads/master/lectures/syscalls.html#linux-system-calls-implementation)
- [Entry/exit handling for exceptions, interrupts, syscalls and KVM - Kernel Docs](https://docs.kernel.org/core-api/entry.html)
- The linux source code (linux-6.8.1)
