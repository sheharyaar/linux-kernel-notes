- [Linux Kernel Debugging](#linux-kernel-debugging)
  - [Step 1 : Getting and understanding the logs](#step-1--getting-and-understanding-the-logs)
    - [Debugging by printing](#debugging-by-printing)
    - [syslog](#syslog)
    - [Probing the system](#probing-the-system)
    - [Git Bisect](#git-bisect)
    - [OOPs Debugging](#oops-debugging)
    - [References](#references)
  - [Step 2 : Debugging with GDB and QEMU/libvirt](#step-2--debugging-with-gdb-and-qemulibvirt)
    - [Building the Kernel for GDB Debugging](#building-the-kernel-for-gdb-debugging)
    - [Starting QEMU and Running VM](#starting-qemu-and-running-vm)
    - [Linux Provided GDB Helpers](#linux-provided-gdb-helpers)
    - [QEMU Advanced Options](#qemu-advanced-options)
    - [References](#references-1)
  - [Extras](#extras)
    - [Hung Kernel](#hung-kernel)
    - [Memory Leaks in Userspace : Valgrind](#memory-leaks-in-userspace--valgrind)
    - [Debugging with Syzkaller](#debugging-with-syzkaller)
    - [Debugging with KDB](#debugging-with-kdb)

# Linux Kernel Debugging

## Step 1 : Getting and understanding the logs

### Debugging by printing

- printk can be used anywhere. It can be called from interrupt or process context. even when a lock is held. It can only not be used prior to console initialization.
- We can set a default loglevel to make it easy to debug

### syslog 

- kolgd or the kernel log daemon chooses any of the two potential sources of kernel log information: the /proc file system and the syscall (sys_syslog) interface.
- After reading the logs it sends them to syslogd which then saves it to a file. Default /var/log/messages

### Probing the system

- condition variables can be used while debugging drivers to trigger certain conditions or to print logs under certain conditions

### Git Bisect

- If an issue is identified in a new version, then git bisect can be helpful to perform binary searches on the tree history in order to trace the version that introduced the bug.

### OOPs Debugging

- If an OOPs occurs in the kernel, the kernel dumps information about the various registers, its contents and the call stack.
- The call stack has hexadecimal addresses, to convert those to symbols, we need a symbol table that is provided by System.map which is unique for each kernel version and build configs.
- kallsysm is a feature that allows symbols to be generated during build time so that debugging can be done without System.map file. The following config options are handy to generate debug information when building the kernel : 
```conf
CONFIG_DEBUG_KERNEL=y
CONFIG_KALLSYSMS=y
```
- Once the symbol name and the addresses are known, then we can use gdb to debug the object file of the faulty module.

For a sample Oops error : 

```shell
BUG: unable to handle kernel NULL pointer dereference at (null)
IP: [<ffffffffa03e1012>] my_oops_init+0x12/0x21 [oops]
PGD 7a719067 PUD 7b2b3067 PMD 0
Oops: 0002 [#1] SMP
last sysfs file: /sys/devices/virtual/misc/kvm/uevent
CPU 1
Pid: 2248, comm: insmod Tainted: P       	2.6.33.3-85.fc13.x86_64
RIP: 0010:[<ffffffffa03e1012>]  [<ffffffffa03e1012>] my_oops_init+0x12/0x21 [oops]
RSP: 0018:ffff88007ad4bf08  EFLAGS: 00010292
RAX: 0000000000000018 RBX: ffffffffa03e1000 RCX: 00000000000013b7
RDX: 0000000000000000 RSI: 0000000000000046 RDI: 0000000000000246
RBP: ffff88007ad4bf08 R08: ffff88007af1cba0 R09: 0000000000000004
R10: 0000000000000000 R11: ffff88007ad4bd68 R12: 0000000000000000
R13: 00000000016b0030 R14: 0000000000019db9 R15: 00000000016b0010
FS:  00007fb79dadf700(0000) GS:ffff880001e80000(0000) knlGS:0000000000000000
CS:  0010 DS: 0000 ES: 0000 CR0: 000000008005003b
CR2: 0000000000000000 CR3: 000000007a0f1000 CR4: 00000000000006e0
DR0: 0000000000000000 DR1: 0000000000000000 DR2: 0000000000000000
DR3: 0000000000000000 DR6: 00000000ffff0ff0 DR7: 0000000000000400
Process insmod (pid: 2248, threadinfo ffff88007ad4a000, task ffff88007a222ea0)
Stack:
ffff88007ad4bf38 ffffffff8100205f ffffffffa03de060 ffffffffa03de060
 0000000000000000 00000000016b0030 ffff88007ad4bf78 ffffffff8107aac9
 ffff88007ad4bf78 00007fff69f3e814 0000000000019db9 0000000000020000
Call Trace:
[<ffffffff8100205f>] do_one_initcall+0x59/0x154
[<ffffffff8107aac9>] sys_init_module+0xd1/0x230
[<ffffffff81009b02>] system_call_fastpath+0x16/0x1b
Code: <c7> 04 25 00 00 00 00 00 00 00 00 31 c0 c9 c3 00 00 00 00 00 00 00
RIP  [<ffffffffa03e1012>] my_oops_init+0x12/0x21 [oops]
RSP <ffff88007ad4bf08>
CR2: 0000000000000000
```

- `IP` denotes the instruction pointer at the time of the fault

- `PGD`, `PMD` and `PUD` denote the **Page Global Directory**, **Page Upper Directory**, and **Page Middle Directory**, part of the **kernel's paging mechanism**, used to **translate virtual addresses to physical addresses**. The values show the state of the paging structure at the time of the crash.

- Oops: 002 [#1] here 002 is an error code in hex.
  - `bit 0 == 0` means no page found, 1 means a protection fault
  - `bit 1 == 0` means read, 1 means write
  - `bit 2 == 0` means kernel, 1 means user-mode
  - [#1] — this value is the number of times the Oops occurred. Multiple Oops can be triggered as a cascading effect of the first one.

- `last sysfs file` tells the last sysfs file accessed, which can sometimes help in identifying what the system was doing when the error occurred.
  
- `Pid: 2248, comm: insmod Tainted: P       	2.6.33.3-85.fc13.x86_64 `
. Here Pid denotes the pid of the faulty process and **comm** denotes the command that was invoked which created the process.

- Tainted  shows if the kernel was tainted at the time of the crash, meaning it was in a state that is unsupported by the kernel developers. Possible values are : 
  - **P** — Proprietary module has been loaded.
  - **F** — Module has been forcibly loaded.
  - **S** — SMP with a CPU not designed for SMP.
  - **R** — User forced a module unload.
  - **M** — System experienced a machine check exception.
  - **B** — System has hit bad_page.
  - **U** — Userspace-defined naughtiness.
  - **A** — ACPI table overridden.
  - **W** — Taint on warning.

- The dump further contains the various registers and their values at the time of the fault. It is followed by the stack trace of the invocations leading to the fault.

- Once we have symbols corresponding to the stack trace and the instruction pointer, we can use gdb to load the faulty module or further disassemble it using objdump and proceed with the debugging process.

### References

- [Debugging Techniques - Linux Device Drivers (LDD3)](https://static.lwn.net/images/pdf/LDD3/ch04.pdf)
- [Debugging Analysis of Kernel panics and Kernel oopses using System Map - Sanjeev Sharma Blog](https://sanjeev1sharma.wordpress.com/tag/debug-kernel-panics/)
- [Understanding a Kernel Oops! - Opensourceforu.com](https://www.opensourceforu.com/2011/01/understanding-a-kernel-oops/)
   
## Step 2 : Debugging with GDB and QEMU/libvirt

- QEMU can be used to launch a built kernel and then GDB can be used to attach to the QEMU process and debug the kernel. Remember you should use `gdb` when a target hardware is not available. If you have a target hardware, you should use `kgdb` instead. See : [Using kgdb, kdb and the kernel debugger internals](https://www.kernel.org/doc/html/latest/dev-tools/kgdb.html)
  
- QEMU supports working with gdb via gdb’s remote-connection facility (the “gdbstub”). Basically this means that QEMU can act as a gdbserver, and gdb can connect to it and debug the kernel running inside the QEMU VM.
  
- To launch the kernel with QEMU you require the following components : 
  - The kernel image (bzImage or zImage)
  - The filesystem image (rootfs)
  - Command line arguments to be passed to the kernel

### Building the Kernel for GDB Debugging

1. Kernel configs to enable during build time for easy gdb debugging :
  - `CONFIG_DEBUG_INFO` : This option includes debugging information in the kernel image. This is useful for debugging the kernel using gdb.
  - `CONFIG_DEBUG_INFO_SPLIT` : (optional) This significantly reduces the size of the kernel image and kernel modules installed on the device or VM we will be debugging. Note that this option requires a gcc version greater than or equal to version 4.7
  - `CONFIG_GDB_SCRIPTS` : This option includes a set of gdb scripts that can be used to debug the kernel. Leave `CONFIG_DEBUG_INFO_REDUCED` off to get full debugging information.
  - `CONFIG_FRAME_POINTER` : Enable this if your architecture supports it. This greatly improves the reliability of backtraces.
  - `CONFIG_PREEMPT` : This option allows the kernel to be preempted.
  - `CONFIG_DEBUG_KERNEL` 
  - `CONFIG_KALLSYMS` : This option includes the kernel symbol table in the kernel image.
  - `CONFIG_SPINLOCK_SLEEP` : This option allows spinlocks to sleep, which is useful for debugging.
  - `CONFIG_KGDB`
  - `CONFIG_DYNAMIC_DEBUG`: This option allows you to dynamically enable/disable kernel debug messages.
  - `CONFIG_DEBUG_SLAB, CONFIG_DEBUG_VM, etc.`, can be enabled based on the specific areas of the kernel you are interested in.

2. Disable these options to make debugging easier (comment them out or set them "=n"):
  - `CONFIG_CC_OPTIMIZE_FOR_SIZE`: Disabling this option might be useful as it opts for compiling the kernel with less aggressive optimizations, which makes debugging easier
  - `CONFIG_RANDOMIZE_BASE`: Disabling **KASLR (Kernel Address Space Layout Randomization)** can make debugging simpler by ensuring that kernel symbols are loaded at consistent addresses between boots.

> Note: If you want to build the kernel for both fuzzing and debugging, include kernel config options from those docs too. Eg - kcov, kasan, kmemleak, etc. Refer to [fuzzing docs](./finding-bugs.md) for more details.

If you want to add kgdb and kdb support use :
```
# CONFIG_STRICT_KERNEL_RWX is not set
CONFIG_FRAME_POINTER=y
CONFIG_KGDB=y
CONFIG_KGDB_SERIAL_CONSOLE=y
CONFIG_KGDB_KDB=y
CONFIG_KDB_KEYBOARD=y
```

### Starting QEMU and Running VM

1. Run `make gdb_scripts` to build the gdb scripts (required on kernels v5.1 and above).
   
2. Start QEMU with the following command : 
```shell
qemu-system-x86_64 \
    -kernel $KERNEL_SRC/arch/x86/boot/bzImage \
    -append "console=ttyS0,115200 nokaslr" \ 
    -gdb tcp::1234 \
    -S
```
- `-gdb` tcp::<port> Port to run the gdbserver on
- `-S` Freeze the CPU on startup (useful for debugging early steps in the kernel)
- `-kernel` <path> Path to kernel image to debug
- `-append` <cmdline> Linux kernel command-line parameters. `nokaslr` is used to disable KASLR (Kernel Address Space Layout Randomization).

1. Start gdb: `gdb vmlinux`

> Note: Some distros may restrict auto-loading of gdb scripts to known safe directories. In case gdb reports to refuse loading vmlinux-gdb.py, add: `add-auto-load-safe-path /path/to/linux-build` to `~/.gdbinit`.

### Linux Provided GDB Helpers

The Linux kernel provides a set of gdb helpers to make debugging easier. The number of commands and convenience functions may evolve over the time. To list all the available commands, run `apropos lx`.

```
(gdb) apropos lx
function lx_current -- Return current task
function lx_module -- Find module by name and return the module variable
function lx_per_cpu -- Return per-cpu variable
function lx_task_by_pid -- Find Linux task by PID and return the task_struct variable
function lx_thread_info -- Calculate Linux thread_info from task variable
lx-dmesg -- Print Linux kernel log buffer
lx-lsmod -- List currently loaded modules
lx-symbols -- (Re-)load symbols of Linux kernel and currently loaded modules
```

- `$lx_current()` can pe used to get the current task. Example to get the pid of current task, you can run : `p $lx_current().pid` or to get the comm of the current task, you can run : `p $lx_current().comm`

- Make use of the per-cpu function for the current or a specified CPU:
```
(gdb) p $lx_per_cpu("runqueues").nr_running
$3 = 1
(gdb) p $lx_per_cpu("runqueues", 2).nr_running
$4 = 0
```

- If you’re setting a breakpoint in vfs_open, but only care about the file named test, you might use something like the following: 
```
br vfs_open if $_streq(path.dentry->d_iname, "test")
```

To view more details, checkout [Examples of using the Linux-provided gdb helpers - Kernel docs](https://www.kernel.org/doc/html/latest/dev-tools/gdb-kernel-debugging.html#examples-of-using-the-linux-provided-gdb-helpers)

### QEMU Advanced Options

If you want to examine/change the physical memory you can set the gdbstub to work with the physical memory rather with the virtual one. The memory mode can be checked by sending the following command:

`maintenance packet qqemu.PhyMemMode` : This will return either 0 or 1, 1 indicates you are currently in the physical memory mode.

`maintenance packet Qqemu.PhyMemMode:1` : This will change the memory mode to physical memory.

`maintenance packet Qqemu.PhyMemMode:0` : This will change it back to normal memory mode.


For more information checkout [Examining physical memory - QEMU docs](https://qemu-project.gitlab.io/qemu/system/gdb.html#examining-physical-memory)

### References

- [Linux Device Drivers (LDD3) Chapter 4](https://static.lwn.net/images/pdf/LDD3/ch04.pdf)
- [Debugging kernel and modules via gdb - Kernel Docs](https://www.kernel.org/doc/html/latest/dev-tools/gdb-kernel-debugging.html)
- [GDB usage - QEMU](https://qemu-project.gitlab.io/qemu/system/gdb.html)
- [USING GDB TO DEBUG THE LINUX KERNEL - starlab](https://www.starlab.io/blog/using-gdb-to-debug-the-linux-kernel)
- [Kernel address space layout randomization (KASLR) - LWN](https://lwn.net/Articles/569635/)
- [Debugging Techniques - Linux Device Drivers (LDD3)](https://static.lwn.net/images/pdf/LDD3/ch04.pdf)
- [Debugging the Linux kernel with GDB - Sergio Prado](https://sergioprado.blog/debugging-the-linux-kernel-with-gdb/)

## Extras

### Hung Kernel

If the kernel hangs on executing a code in the kernel, then the following config options can be used to debug the issue :

```conf
CONFIG_SOFTLOCKUP_DETECTOR=y
CONFIG_HUNG_TASK=y
```

After a few seconds (~30s), the kernel generates an Oops and prints the stack trace of the hung task. You can then debug the issue using the same techniques as mentioned in the OOPs Debugging section.

### Memory Leaks in Userspace : Valgrind

- Compile the binary with symbols
- Run the binary with valgrind
- Valgrind generates a report of the memory leaks and the stack trace of the memory allocation

### Debugging with Syzkaller

Complete notes on syzkaller are present in the [Syzkaller](./syzkaller.md) section.

### Debugging with KDB

- If kGDB is not available, then KDB can be used to debug the kernel. KDB is a kernel debugger that can be used to debug the kernel in case of a kernel panic or a hung kernel.

For more information on KDB usage, refer [Using kgdb, kdb and the kernel debugger internals](https://www.kernel.org/doc/html/latest/dev-tools/kgdb.html)