The `vDSO` (virtual dynamic shared object) is a **small shared library** that the kernel automatically maps into the **address space of all user-space applications**.  
  
### Need for vDSO  
  
- Some system calls are called so often that the overhead in triggering the software interrupt (`int $0x80`, See [Syscall implementation in Linux](./Implementation.md) for more details) that this dominates the overall performance, for example, `gettimeofday` is called both directly by user-space applications as well as indirectly by the C library.   
- Faster methods to load a system calls is present but that is available **only** in new architectures. This has to be determined by the C library at runtime, so instead of this, the functionality is provided by **vDSO**. It only has functions that return the same value in both privileged and non privileged mode.  
## Base address  
  
- The base address of the vDSO (if one exists) is passed by the kernel to each program in the initial auxiliary vector (see [getauxval(3)](https://man7.org/linux/man-pages/man3/getauxval.3.html)), via the **AT_SYSINFO_EHDR** tag.   
- You must **not assume** the vDSO is mapped at any particular location in the user's memory map.  The base address will usually be **randomized at run time every time** a new process image is created (during `execve`).  
## File format  
  
- Since the vDSO is a **fully formed ELF image**, you can do **symbol lookups** on it.    
- This allows new symbols to be added with newer kernel releases, and **allows the C library to detect available functionality at run time** when running under different kernel versions.    
- Oftentimes the C library will do detection with the first call and then cache the result for subsequent calls.  
- All symbols are also **versioned** (using the GNU version format). This allows the kernel to **update the function signature without breaking backward compatibility**.    
- When looking up a symbol in the vDSO, you **must always include the version** to match the ABI you expect.  
## Notes  
  
- When tracing systems calls with `strace`, symbols (system calls) that are exported by the vDSO will **not** appear in the trace output.  Those system calls will likewise **not be visible** to `seccomp` filters.  
  
> Note that the vDSO that is used is based on the ABI of your user-space code and not the ABI of the kernel.  Thus, for example, when you run an i386 32-bit ELF binary, you'll get the same vDSO regardless of whether you run it under an i386 32-bit kernel or under an x86-64 64-bit kernel.  
  
## References  
  
- [vdso - Manpage](https://man7.org/linux/man-pages/man7/vdso.7.html)  
  
#syscalls 