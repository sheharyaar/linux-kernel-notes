The kernel provides some functions to access the user-space from the kernel space during syscalls. Some of these are :  
  
- `get_user()`  
- `put_user()`  
- `copy_from_user()`  
- `copy_to_user()`  
  
These functions also check if the pointer is in user-space and also handle faults if the pointer is invalid. For invalid pointers, they return non-zero value.  
  
> Remember syscalls run in process context so [current](../../output/Misc/Current.md) is valid.  
  
  
#syscalls 