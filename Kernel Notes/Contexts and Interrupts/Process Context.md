- When coming from userspace via [syscalls](../../output/Syscalls/Syscalls.md) or other `traps`.  
- You can **sleep**, you own the CPU (except for `interrupts`). User context is not pre-emptable  
- the [current](../../output/Misc/Current.md) pointer (indicating the process currently executing) is valid  
  
#context #syscalls 