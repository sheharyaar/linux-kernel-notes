#### What is KABI or (Kernel ABI) ?  
- ABIs are similar to APIs in that they govern the interpretation of commands and exchange of binary data.   
- For C programs, the ABI generally comprises the return types and parameter lists of functions, the layout of structs, and the meaning, ordering, and range of enumerated types.   
 - The functionality of the ABI requires a joint, ongoing effort by :   
	 - the kernel community, C compilers (such as GCC or clang),   
	 - the developers who create the userspace C library (most commonly glibc) that implements system calls, and binary applications, which much be laid out in accordance with the Executable and Linking Format (ELF)  
![Pasted image 20240401173308.png](./Assets/Pasted%20image%2020240401173308.png)  
 - All of sysfs (`/sys`) and procfs (`/proc`) are guaranteed stable except for `/sys/kernel/debug`  
 - Syscall interface is also guaranteed to be stable  
   
Reference : [https://opensource.com/article/22/12/linux-abi](https://opensource.com/article/22/12/linux-abi)  
Linus on ABI Breakage : [https://lkml.org/lkml/2018/12/22/232](https://lkml.org/lkml/2018/12/22/232)  
  
#kabi #faq #kernel #linux