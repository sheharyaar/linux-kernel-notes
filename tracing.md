- [Tracing](#tracing)
  - [Userspace tools](#userspace-tools)
    - [strace and ltrace](#strace-and-ltrace)
  - [Kernelspace tools](#kernelspace-tools)
    - [tracefs](#tracefs)
    - [ftrace](#ftrace)
      - [Tracepoints](#tracepoints)
    - [kprobes](#kprobes)
    - [uprobes](#uprobes)
    - [perf](#perf)
    - [eBPF](#ebpf)
    - [How perf, krpobe and tracefs relate](#how-perf-krpobe-and-tracefs-relate)
    - [Persistance Store Support](#persistance-store-support)
    - [References](#references)
- [References](#references-1)

# Tracing

## Userspace tools

### strace and ltrace

**strace :**

- Traces system calls, signals, andioctl() calls
- Can trace multiple processes simultaneously
- Can filter traces by process ID, system call, or signal
- Can output traces in a variety of formats, including text, JSON, and XML
- Can be used to attach to a running process or to trace a process from start to finish
- To use strace, simply run it with the name of the process you want to trace as an argument. For example, to trace the ls command, run the following command: `strace ls`

```shell
...
openat(AT_FDCWD, ".", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
fstat(3, {st_mode=S_IFDIR|0755, st_size=4096, ...}) = 0
getdents64(3, 0x5af2b17d56f0 /* 5 entries */, 32768) = 184
getdents64(3, 0x5af2b17d56f0 /* 0 entries */, 32768) = 0
close(3)                                = 0
fstat(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(0x88, 0x3), ...}) = 0
write(1, " edited-2.jpg~\t ns3  'WhatsApp I"..., 69 edited-2.jpg~	 ns3  'WhatsApp Image 2023-07-08 at 6.39.50 PM.jpeg~'
) = 69
...
```

**ltrace**

- Traces library calls made by the program

## Kernelspace tools

### tracefs

- tracefs is a pseudo file system that provides an interface to the kernel's trace events
- tracefs is mounted at /sys/kernel/tracing
- tracefs is used by ftrace, perf, and eBPF to access trace events
- tracefs can be mount using `mount -t tracefs nodev /sys/kernel/tracing `
**Note** : For backward compatibility, when mounting the debugfs file system, the tracefs file system will be automatically mounted at: `/sys/kernel/debug/tracing`

### ftrace

Some files available under /sys/kernel/tracing : 

1. `available_events` : lists out events available for tracing in the kernel

```shell
...
sof_intel:sof_intel_hda_irq
sof_intel:sof_intel_ipc_firmware_response
sof_intel:sof_intel_ipc_firmware_initiated
sof_intel:sof_intel_D0I3C_updated
sof_intel:sof_intel_hda_irq_ipc_check
sof_intel:sof_intel_hda_dsp_pcm
sof_intel:sof_intel_hda_dsp_stream_status
sof_intel:sof_intel_hda_dsp_check_stream_irq
sof:sof_widget_setup
Sof:sof_widget_free
...
```

2. `current_trace` : display the current tracer that is configured, if no tracers are enabled it gives **nop**. To enable tracing you can just **echo a tracer to this file**. To stop, you can **echo nop to the same file.** 
  
```shell
echo function > /sys/kernel/tracing/current_tracer
cat /sys/kernel/tracing/trace
    ...
       budgie-wm-1094    [004] d..2. 27575.413078: irq_enter_rcu <-sysvec_irq_work
       budgie-wm-1094    [004] d.h2. 27575.413078: irqtime_account_irq <-sysvec_irq_work
       budgie-wm-1094    [004] d.h2. 27575.413079: __sysvec_irq_work <-sysvec_irq_work
       budgie-wm-1094    [004] d.h2. 27575.413079: signal_irq_work <-irq_work_single
       budgie-wm-1094    [004] d.h2. 27575.413079: ktime_get <-signal_irq_work
       budgie-wm-1094    [004] d.h2. 27575.413079: __rcu_read_lock <-signal_irq_work
       budgie-wm-1094    [004] d.h2. 27575.413080: __rcu_read_unlock <-signal_irq_work
       budgie-wm-1094    [004] d.h2. 27575.413080: _raw_spin_lock <-signal_irq_work
       budgie-wm-1094    [004] d.h3. 27575.413080: irq_enable <-signal_irq_work
       budgie-wm-1094    [004] d.h3. 27575.413080: intel_engine_irq_enable <-signal_irq_work
       budgie-wm-1094    [004] d.h3. 27575.413080: _raw_spin_lock <-intel_engine_irq_enable
       budgie-wm-1094    [004] d.h4. 27575.413081: gen8_logical_ring_enable_irq <-intel_engine_irq_enable
...
echo nop > /sys/kernel/tracing/current_tracer 
```

3. `trace`: holds the output of the trace in a human readable format (described below).
**Note**: tracing is temporarily disabled while this file is being read

4. `trace_pipe`: the output is the same as the “trace” file but this file is meant to be streamed with live tracing.
    
- Reads from this file will block until new data is retrieved.
- Unlike the “trace” file, this file is a consumer. This means reading from this file causes sequential reads to display more current data. 
- Once data is read from this file, it is consumed, and will not be read again with a sequential read. 

5. `available tracers`: holds the different types of tracers that have been compiled into the kernel. The tracers listed here can be configured by echoing their name into current_tracer
  
```shell
cat /sys/kernel/tracing/available_tracers 
timerlat osnoise hwlat blk mmiotrace function_graph wakeup_dl wakeup_rt wakeup function nop
```

#### Tracepoints

- Tracepoints can be used without creating custom kernel modules to register probe functions using the event tracing infrastructure.
  
- To enable a particular event, simply echo it to /sys/kernel/tracing/set_event. 

Example : 
- Step 1 : we will enable an event syscalls:sys_enter_getpeername in our case, this will generate a trace every time getpeername() is executed. This function is used for making DNS queries : 
- Step 2 : we will listen on the trace_pipe in another terminal
- Step 3 : We will make a DNS query using dig program : dig www.httpbin.org
- Step 4 : We will disable tracing by echoing an empty string to the file

```shell
echo "syscalls:sys_enter_getpeername" > set_event
cat trace_pipe
 <...>-570949  [007] ..... 28732.695686: sys_getpeername(fd: f, usockaddr: c001784ae8, usockaddr_len: c001784ae4)
 <...>-570956  [001] ..... 28732.695731: sys_getpeername(fd: 1c, usockaddr: c000aacae8, usockaddr_len: c000aacae4)
 <...>-570957  [005] ..... 28732.695735: sys_getpeername(fd: 1b, usockaddr: c000ab0ae8, usockaddr_len: c000ab0ae4)
 <...>-570953  [002] ..... 28732.699574: sys_getpeername(fd: 1c, usockaddr: c000aacae8, usockaddr_len: c000aacae4)
 <...>-570947  [004] ..... 28732.700133: sys_getpeername(fd: f, usockaddr: c001784ae8, usockaddr_len: c001784ae4)
^C⏎
echo "" > set_event
```

**From the events directory**: events directory has a file named enable, if you echo 1 to the file, the event is enabled to be traced.

```shell
root@rog /s/k/tracing# cd events/syscalls/sys_enter_getpeername/
root@rog /s/k/t/e/s/sys_enter_getpeername# ls
enable  filter  format  hist  id  trigger
root@rog /s/k/t/e/s/sys_enter_getpeername# echo 1 > enable
```

### kprobes

- Based on kprobes (kprobe and kretprobe). It can probe entry and exit of a kernel function.

- Can be added and removed dynamically, on the fly. To enable this feature, build your kernel with `CONFIG_KPROBE_EVENTS=y`.

- Doesn’t need to be activated via current_tracer. Instead of that, add probe points via /sys/kernel/debug/tracing/kprobe_events, and enable it via `/sys/kernel/debug/tracing/events/kprobes/<EVENT>/enable`.

### uprobes

- Uprobe based trace events are similar to kprobe based trace events. To enable this feature, build your kernel with `CONFIG_UPROBE_EVENTS=y`.

- Add probe points via /sys/kernel/debug/tracing/uprobe_events, and enable it via /`sys/kernel/debug/tracing/events/uprobes/<EVENT>/enable`

- However unlike kprobe-event tracer, the uprobe event interface expects the user to calculate the offset of the probepoint in the object.
  
### perf

- Profiler tool for Linux, part of the Linux kernel tools.
- Identifying CPU-bound bottlenecks, cache misses, and branch mispredictions.
- Profiling both userspace and kernelspace code to understand overall system performance.
- Uses hardware counters and software events to collect data. It's directly integrated with the CPU's performance monitoring capabilities, allowing it to gather detailed metrics about hardware and application performance.

### eBPF

eBPF is a large topic and is covered in [eBPF section](./ebpf-tracing.md)

### How perf, krpobe and tracefs relate

### Persistance Store Support

### References

- [Notes on BPF & eBPF - Julia Evans](https://jvns.ca/blog/2017/06/28/notes-on-bpf---ebpf/)
- [Linux tracing systems & how they fit together - Julia Evans](https://jvns.ca/blog/2017/07/05/linux-tracing-systems/)

# References
- [Event Tracing - Kernel Doc](https://www.kernel.org/doc/html/latest/trace/events.html)
- [Tracing the Linux kernel with ftrace](https://sergioprado.blog/tracing-the-linux-kernel-with-ftrace/)
- [Mentorship Session: Tools and Techniques to Debug an Embedded Linux System](https://www.youtube.com/watch?v=Paf-1I7ZUTo)