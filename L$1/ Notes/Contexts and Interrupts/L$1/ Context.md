---  
tags:  
  - context  
  - syscalls  
---  
- When coming from userspace via [syscalls](../../Syscalls/LSyscalls/.md) or other `traps`.  
- You can **sleep**, you own the CPU (except for `interrupts`). User context is not pre-emptable  
- the [current](../../Misc/LCurrent/.md) pointer (indicating the process currently executing) is valid  
  
#context #syscalls 