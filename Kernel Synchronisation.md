- Sleeping locks and non sleeping locks  
Semaphores are often used for both serialization and waiting, but new use cases should instead use separate serialization and wait mechanisms, such as mutexes and completions.  
  
use seqcount_latch_t when the write side sections cannot be protected from interruption by readers. This is typically the case when the read side can be invoked from NMI handlers.  
  
- semaphores  
- spinlocks  
- mutexes  
- completion variables  
- seqlocks  
  
- Wait-Die, Wound-Wait approach (in locking) → ww-mutexes [https://docs.kernel.org/locking/ww-mutex-design.html](https://docs.kernel.org/locking/ww-mutex-design.html)  
- per-CPU semaphores ?? - [https://docs.kernel.org/locking/percpu-rw-semaphore.html](https://docs.kernel.org/locking/percpu-rw-semaphore.html)  
  
- futex  
- pdata parallel execution → parallel jobs with reordering (eg - encryption/decryption + reordering) [https://docs.kernel.org/core-api/padata.html](https://docs.kernel.org/core-api/padata.html). supports multithreaded jobs, splitting up the job evenly while load balancing and coordinating between threads.  
  
> Important Rules for writing good lock codes : [https://docs.kernel.org/locking/preempt-locking.html](https://docs.kernel.org/locking/preempt-locking.html0) (must refer always)  
  
**Note**: When comparing or studying mutexes and algorithms check for both fairness and correctness. Just like in Distributed systems we check for safety and correctness.