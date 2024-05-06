# Notification Mechanism

 - Built on top of the standard pipe driver
 - can be enabled by `CONFIG_WATCH_QUEUE`
 - [splicing](../../implementations/pipes.md#splice) is disabled on these pipes to prevent inter-leaving of kernel messages
 - owner of the pipe has to tell the kernel, the sources from which it would like to get notifications
 - a message will be discarded if there isn’t a slot available in the ring or if no preallocated message buffer is available. In both of these cases
 - the kernel **does not wait** for the consumers to collect it, but rather just continues on. 
	 - This means that **notifications can be generated whilst spinlocks are held** and also protects the kernel from being held up indefinitely by a userspace malfunction.

Please refer [General notification mechanism - Kernel Docs](https://docs.kernel.org/core-api/watch_queue.html) for information on Mesage Structure, Watch Queue, Watch List and Event Filtering.
