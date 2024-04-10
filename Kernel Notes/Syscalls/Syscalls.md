- Provides a layer between the user-space and the hardware  
- These are **not** function calls, but these are **assembly instructions**   
- These instructions perform the following :  
	- setup information to identify the system call and it’s parameters  
	- trigger a kernel mode switch  
	- retrieve the result of the system call  
  
For more detailed information checkout the following :   
[Implementation in Linux](./Implementation.md)  
[vDSO and virtual syscalls](./vDSO%20and%20virtual%20syscalls.md)  
[Userspace from syscalls](./Userspace%20from%20syscalls.md)  
  
#syscalls