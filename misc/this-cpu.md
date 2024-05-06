# this_cpu

- way of optimizing access to per CPU variables associated with the **currently executing** processor.
- done using `segment register` → a *dedicated* register where the CPU permanently stored the beginning of **per-cpu area** of the specific processor.
- **no atomicity issues** between the calculation of the *offset* and the operation on data (the CPU is itself executing it).
> Hence, it is **not necessary** to disable preemption or interrupts to ensure that the processor is not changed between the calculation of the address and the operation on the data.

- Frequently processors have special lower latency instructions that can operate without the typical synchronization overhead, but still provide some sort of relaxed atomicity guarantees. The x86, for example, can execute RMW (Read Modify Write) instructions like inc/dec/cmpxchg without the lock prefix and the associated latency penalty.
- Access to the variable without the lock prefix is not synchronized but synchronization is not necessary since we are dealing with per cpu data specific to the currently executing processor. 
- Only the current processor should be accessing that variable and therefore there are no concurrency issues with other processors in the system.
```
this_cpu_read(pcp)
this_cpu_write(pcp, val)
this_cpu_add(pcp, val)
this_cpu_and(pcp, val)
this_cpu_or(pcp, val)
this_cpu_add_return(pcp, val)
this_cpu_xchg(pcp, nval)
this_cpu_cmpxchg(pcp, oval, nval)
this_cpu_sub(pcp, val)
this_cpu_inc(pcp)
this_cpu_dec(pcp)
this_cpu_sub_return(pcp, val)
this_cpu_inc_return(pcp)
this_cpu_dec_return(pcp)
```

## Inner working of this_cpu operations

On x86 the `fs`: or the `gs`: segment registers contain the **base of the per cpu area**. It is then possible to simply use the segment override to relocate a per cpu relative address to the proper per cpu area for the processor. So the relocation to the per cpu base is encoded in the instruction via a segment register prefix.

For example : 
```c
DEFINE_PER_CPU(int, x);
int z;

z = this_cpu_read(x);
```
results in a single instruction (hence atomic):
```asm
mov ax, gs:[x]
```

For detailed description, See : [https://docs.kernel.org/core-api/this_cpu_ops.html](https://docs.kernel.org/core-api/this_cpu_ops.html)
