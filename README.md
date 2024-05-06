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

- [Debugging](./debugging.md)
  - [Step 1 : Getting and understanding the logs](./debugging.md#step-1--getting-and-understanding-the-logs)
  - [Step 2 : Debugging with GDB and QEMU/libvirt](./debugging.md#step-2--debugging-with-gdb-and-qemulibvirt)
  - [Extras ](./debugging.md#extras)
- [Finding Bugs](./finding-bugs.md)
- [Event tracing](./tracing.md)
- Kernel Concepts
  - [Interrupts](./concepts/interrupts.md)
    - [Syscalls](./concepts/syscalls/syscalls.md)
- [Kernel Core APIs](./core-apis.md) (*TODO*)
- [Kernel Subsystem APIs](./subsystem-apis.md) (*TODO*)
- [Kernel Testing](./kernel-testing.md) (*TODO*)
- [Kernel Mentorship Program](./mentorship.md) (*TODO*)

## Extras

- [Kernel Boot Process](./kernel-boot.md) (*TODO*)
- [Syzkaller Notes](./syzkaller.md)
- [eBPF Tracing](./ebpf-tracing.md) (*TODO*)
- [RCU in Kernel](./rcu.md) (*TODO*)

## Readings

1. Linux Kernel Development by Robert Love (3rd Edition) [Amazon Link](https://www.amazon.in/Linux-Kernel-Development-Developers-Library/dp/0672329468)
2. Linux Device Drivers 3ed by Johnathan Corbet, Alessandro Rubini, and Greg Kroah-Hartman [Amazon Link](https://www.amazon.in/dp/8173668493/?coliid=I17OESPBYIRQO5&colid=2OMG77SFUJMQ3&psc=1&ref_=list_c_wl_lv_ov_lig_dp_it)
3. Understanding the Linux Kernel 3e: From I/O Ports to Process Management [Amazon Link](https://www.amazon.in/Understanding-Linux-Kernel-Daniel-Bovet/dp/0596005652)
4. Design of the UNIX Operating System by Maurice J. Bach (AT&T Bell Labs) [Amazon Link](https://www.amazon.in/Design-UNIX-Operating-System-1/dp/9332549575/)

> Other readings are mentioned in the respective notes' README.md