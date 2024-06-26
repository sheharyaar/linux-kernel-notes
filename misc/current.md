# Current

- this pointer denotes the current running task on a CPU.
- it is accessible only in process context and not interrupts.
- it is defined in `<arch/x86/include/asm/current.h>` for x86 processors
```c
static __always_inline struct task_struct *get_current(void)
{
	if (IS_ENABLED(CONFIG_USE_X86_SEG_SUPPORT))
		return this_cpu_read_const(const_pcpu_hot.current_task);

	return this_cpu_read_stable(pcpu_hot.current_task);
}
```
- returns `task_struct` for the current task running on the CPU

> `this_cpu_read_const` and `this_cpu_read_stable` are used to obtain per-cpu variables. More information is given at `<arch/x86/include/asm/percpu.h>`

```
this_cpu_read() makes gcc load the percpu variable every time it is
accessed while this_cpu_read_stable() allows the value to be cached.
this_cpu_read_stable() is more efficient and can be used if its value
is guaranteed to be valid across cpus.  The current users include
pcpu_hot.current_task and pcpu_hot.top_of_stack, both of which are
actually per-thread variables implemented as per-CPU variables and
thus stable for the duration of the respective task.
```

The source code here is taken from `linux-6.8.1`
