# noinstr

[https://docs.kernel.org/core-api/entry.html](https://docs.kernel.org/core-api/entry.html)

- syscall entry must not be instrumented because :
	- starts in assembly and interacts with love-level C and stack frames (registers should not get overwritten)
	- this function set up `lockdep`, `RCU/Context tracking` and tracing for the call using `syscall_eter_from_user_mode()`. It then invokes various entry functions like ptrace, seccomp, audit, syscall tracing, etc. (RCU is not yet setup at the beginning, hence must not be instrumented)
```c
noinstr void syscall(struct pt_regs *regs, int nr)
{
      arch_syscall_enter(regs);
      nr = syscall_enter_from_user_mode(regs, nr);

      instrumentation_begin();
      if (!invoke_syscall(regs, nr) && nr != -1)
              result_reg(regs) = __sys_ni_syscall(regs);
      instrumentation_end();

      syscall_exit_to_user_mode(regs);
}
```

> Do not nest syscalls. Nested systcalls will cause RCU and/or context tracking to print a warning.
