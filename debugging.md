- [Linux Kernel Debugging](#linux-kernel-debugging)
  - [Step 1 : Getting and understanding the logs](#step-1--getting-and-understanding-the-logs)
    - [Debugging by printing](#debugging-by-printing)
    - [syslog](#syslog)
    - [Probing the system](#probing-the-system)
    - [Git Bisect](#git-bisect)
    - [OOPs Debugging](#oops-debugging)
    - [References](#references)
  - [Step 2 : Debugging with GDB and QEMU/libvirt](#step-2--debugging-with-gdb-and-qemulibvirt)
    - [Debugging flow with GDB and QEMU/libvirt](#debugging-flow-with-gdb-and-qemulibvirt)
    - [Setting up QEMU/libvirt](#setting-up-qemulibvirt)
    - [Debugging with GDB](#debugging-with-gdb)
    - [References](#references-1)

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

1. [Debugging Analysis of Kernel panics and Kernel oopses using System Map - Sanjeev Sharma Blog](https://sanjeev1sharma.wordpress.com/tag/debug-kernel-panics/)
2. [Understanding a Kernel Oops! - Opensourceforu.com](https://www.opensourceforu.com/2011/01/understanding-a-kernel-oops/)
   
## Step 2 : Debugging with GDB and QEMU/libvirt

### Debugging flow with GDB and QEMU/libvirt

### Setting up QEMU/libvirt

### Debugging with GDB

### References
