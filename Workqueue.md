- an independent thread serves as the `async execution context`.   
- this thread is called `worker`  
- the queue is called `workqueue`  
- while there are work items on the workqueue the worker executes the functions associated with the work items one after the other.   
- when there is no work item left on the workqueue the worker becomes idle.   
- **Concurrency Managed Workqueue** (cmwq) is a reimplementation of original **wq** with focus on the following goals :  
	- Maintain compatibility with the original workqueue API.  
	- **Use per-CPU unified worker pools** shared by all wq to provide flexible level of concurrency on demand without wasting a lot of resource.  
	- **Automatically regulate worker pool** and level of concurrency so that the API users don’t need to worry about such details.  
## Design  
1. `work item` - it is a simple struct that holds pointer to the function that needs to be executed. this work item can then be queued on the workqueue.  
> **Note:** A work item can be executed **either** a thread context or BH (softirq/bottom half) context.  
2. `worker threads` - special purpose threads, called \[k\]workers, execute the work items and are managed worker-pools.  
3. `worker-pools` : there are **2** worker-pools (for **each possible CPU**):  
	1. for normal work items  
	2. high priority work items  
	There are some extra dynamic worker-pools to serve work items queued on unbound workqeueus.  
4.  Since the BH can have only one concurrent context, each **per-CPU BH worker pool** contains **only one pseudo worker** which represents the BH execution context. A BH workqueue can be considered a convenience interface to softirq.  
5. unless specifically overridden, a work item of a bound workqueue will be queued on the worklist of either normal or highpri **worker-pool** that is **associated to the CPU the issuer is running on**.  
  
### Scheduling  
- Each worker-pool bound to an actual CPU implements concurrency management by **hooking into** the `scheduler`.  
- he worker-pool is notified whenever an **active worker wakes up** or sleeps and **keeps track** of the **number of the currently runnable workers**.  
- As long as there are one or more runnable workers on the CPU, the worker-pool doesn’t start execution of a new work, but, **when the last running worker goes to sleep**, it immediately schedules a new worker so that the CPU doesn’t sit idle while there are pending work items.  
- This allows using a minimal number of workers without losing execution bandwidth.  
- Keeping idle workers around doesn’t cost other than the memory space for kthreads, so cmwq holds onto idle ones for a while before killing them.  
- All work items which might be used on **code paths that handle memory reclaim** are **required to be queued** on wq’s that have a `rescue-worker` reserved for execution under memory pressure.  
  
For more information on affinity, performance and detailed topic, refer to : [Workqueue - Kernel Docs](https://docs.kernel.org/core-api/workqueue.html)  
