- [Linux Kernel](#linux-kernel)
  - [Subsystems selected](#subsystems-selected)
  - [Notes](#notes)
  - [Extras](#extras)
  - [Readings](#readings)

# Linux Kernel

For people who are very new to kernel and kernel development: 
- I will **highly recommend** you to first complete [MIT 6.1810: Operating System Engineering](https://pdos.csail.mit.edu/6.828/2023/index.html) Labs and Readings together with a linux kernel book like [Understanding the Linux Kernel 3e: From I/O Ports to Process Management](https://www.amazon.in/Understanding-Linux-Kernel-Daniel-Bovet/dp/0596005652). 
- This book goes into much details which is not needed, just sift through the chapters along with `xv6 book` in the MIT course. 
- Look into theory and complete the labs.

Once you are done with the course, you will have good background to explore the linux kernel source.

## Subsystems selected

1. Devicetree
2. DRM (GPU)
3. eBPF (Tracing)
4. Bcachefs (Filesystem)

## Notes

- [Debugging](./debugging-tracing/debugging.md)
  - [Step 1 : Getting and understanding the logs](./debugging-tracing/debugging.md#step-1--getting-and-understanding-the-logs)
  - [Step 2 : Debugging with GDB and QEMU/libvirt](./debugging-tracing/debugging.md#step-2--debugging-with-gdb-and-qemulibvirt)
  - [Extras ](./debugging-tracing/debugging.md#extras)
- [Finding Bugs](./debugging-tracing/finding-bugs.md)
- [Event tracing](./debugging-tracing/tracing.md)
  - [eBPF tracing](./debugging-tracing/ebpf-tracing.md)(*TODO*)
- Kernel Concepts
  - [Interrupts](./concepts/interrupts.md)
    - [Syscalls](./concepts/syscalls/syscalls.md)
- [Kernel Core APIs](./core-apis/core-apis.md) (*TODO*)
  - Core Utilities
    - [Notification Mechanisms](./core-apis/core-utilities/notification-mechanism.md)
    - [Printk](./core-apis/core-utilities/printk.md)
    - [Symbols and Assembler Notations](./core-apis/core-utilities/symbols-assemblers.md)
    - [Workqueues](./core-apis/core-utilities/workqueue.md)
  - [Kobjects](./core-apis/kobjects.md)
- Kernel Subsystems
  - Devicetree
    - [Devicetree Yamls](./subsystems/device-tree/devicetree-yaml.md)
    - [Testing yaml dtschemas](./subsystems/device-tree/testing-schema.md)
  - Graphics (*TODO*)
- Implementations
  - [Pipes and Splices](./implementations/pipes.md)
- [Kernel Mentorship Program](./lkmp/mentorship.md) (*TODO*)
  - [Prerequisites](./lkmp/Prerequisites.md) (*TODO*)
  - Setting up the Kernel (*TODO*)

## Extras

- Kernel Boot Process (*TODO*)
- [Syzkaller Notes](./debugging-tracing/syzkaller.md)
- [eBPF Tracing](./debugging-tracing/ebpf-tracing.md) (*TODO*)
- [RCU in Kernel](./extras/rcu.md) (*TODO*)

## Readings

1. Linux Kernel Development by Robert Love (3rd Edition) [Amazon Link](https://www.amazon.in/Linux-Kernel-Development-Developers-Library/dp/0672329468)
2. Linux Device Drivers 3ed by Johnathan Corbet, Alessandro Rubini, and Greg Kroah-Hartman [Amazon Link](https://www.amazon.in/dp/8173668493/?coliid=I17OESPBYIRQO5&colid=2OMG77SFUJMQ3&psc=1&ref_=list_c_wl_lv_ov_lig_dp_it)
3. Understanding the Linux Kernel 3e: From I/O Ports to Process Management [Amazon Link](https://www.amazon.in/Understanding-Linux-Kernel-Daniel-Bovet/dp/0596005652)
4. Design of the UNIX Operating System by Maurice J. Bach (AT&T Bell Labs) [Amazon Link](https://www.amazon.in/Design-UNIX-Operating-System-1/dp/9332549575/)

> Other readings are mentioned in the respective notes' README.md