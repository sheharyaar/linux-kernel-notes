# Syscalls

- Provides a layer between the user-space and the hardware
- These are **not** function calls, but these are **assembly instructions** 
- These instructions perform the following :
	- setup information to identify the system call and it’s parameters
	- trigger a kernel mode switch
	- retrieve the result of the system call

For more detailed information checkout the following : 
- [Implementation in Linux](./implementation.md)
- [vDSO and virtual syscalls](./vdso.md)
- [Userspace from syscalls](./userspace.md)
