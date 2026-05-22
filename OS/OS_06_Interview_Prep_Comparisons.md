# 🖥️ PART 6: INTERVIEW PREPARATION & COMPARISON TABLES

---

# INTERVIEW PREPARATION

---

## Beginner Interview Questions (Freshers / Internships)

### Q1: What is an operating system?
**A:** An OS is system software that manages hardware resources (CPU, memory, disk, I/O) and provides services to user applications. It acts as an intermediary between users and hardware, providing abstraction, resource management, and protection.

### Q2: What is a process? How is it different from a program?
**A:** A program is a passive entity — a file on disk containing code. A process is a program **in execution** — loaded in memory with its own address space, registers, stack, heap, and OS-managed state (PCB). One program can create multiple processes.

### Q3: What is a thread?
**A:** A thread is the smallest unit of CPU execution within a process. Threads within a process share code, data, heap, and files, but each has its own stack, program counter, and registers. Threads are lighter than processes to create and switch between.

### Q4: What is context switching?
**A:** Context switching saves the state (PC, registers, memory info) of the current process/thread and loads the saved state of the next one. It's pure overhead — no useful work is done during the switch. Thread switches are faster than process switches because threads share address space.

### Q5: What is virtual memory?
**A:** Virtual memory allows processes to use more memory than physically available by keeping only needed pages in RAM and the rest on disk. It uses demand paging — pages are loaded only when accessed (lazy loading). Each process gets the illusion of a large, contiguous address space.

### Q6: What is a deadlock?
**A:** A deadlock occurs when a set of processes are each waiting for a resource held by another process in the set, creating a circular dependency where none can proceed. Four conditions must hold simultaneously: Mutual Exclusion, Hold and Wait, No Preemption, and Circular Wait.

### Q7: What is paging?
**A:** Paging divides logical memory into fixed-size pages and physical memory into same-size frames. A page table maps pages to frames. Any page can go in any frame → eliminates external fragmentation. The MMU translates logical addresses to physical addresses using the page table.

### Q8: What is a system call?
**A:** A system call is the interface between user programs and the OS kernel. When a program needs OS services (file I/O, process creation, memory allocation), it makes a system call which triggers a trap to kernel mode, executes the service, and returns to user mode.

### Q9: What are the different types of OS?
**A:** Batch OS (jobs batched and processed), Time-sharing (CPU time divided among users), Real-time (strict deadline constraints — hard vs soft), Distributed (multiple computers as one system), Network OS (server manages shared resources for clients).

### Q10: What is the difference between user mode and kernel mode?
**A:** User mode has restricted access — programs can't directly access hardware or critical OS data structures. Kernel mode has full privilege. Mode switching happens via system calls, interrupts, or exceptions. The mode bit (0=kernel, 1=user) in hardware enforces this.

---

## Intermediate Interview Questions

### Q11: Explain the difference between preemptive and non-preemptive scheduling.
**A:** Non-preemptive: process runs until it finishes or blocks (voluntarily). Preemptive: OS can forcibly take the CPU away (time quantum expires, higher priority arrives). Preemptive gives better response time but more context switches. FCFS and SJF are non-preemptive; RR and SRTF are preemptive.

### Q12: Explain the Banker's Algorithm with an example.
**A:** Banker's Algorithm avoids deadlock by checking if granting a request leads to a safe state. Given allocation matrix, max matrix, and available vector: compute Need = Max - Allocation. Find a safe sequence where each process's Need ≤ Available, then add its Allocation back to Available. If all processes can finish → safe state. [See Part 3 for full worked example.]

### Q13: What is thrashing? How is it handled?
**A:** Thrashing is when the system spends more time paging than executing — caused by too many processes with insufficient memory frames. CPU utilization drops despite high multiprogramming. Solution: Working Set Model (track actively used pages), Page Fault Frequency monitoring, or reducing multiprogramming degree.

### Q14: Explain page replacement algorithms.
**A:** When memory is full and a new page is needed: FIFO evicts oldest page (simple but has Belady's anomaly), LRU evicts least recently used (near-optimal, no Belady's), Optimal evicts page not used for longest future time (ideal but impossible — needs future knowledge), Clock approximates LRU using reference bit (used in practice).

### Q15: What is the critical section problem? What are its requirements?
**A:** The critical section is code accessing shared resources — only one process should execute it at a time. Three requirements: (1) Mutual Exclusion — only one in CS, (2) Progress — if CS is free, some waiting process enters without delay, (3) Bounded Waiting — limit on how long a process waits. Solutions: mutex, semaphore, Peterson's (2 processes).

### Q16: Explain the Producer-Consumer problem.
**A:** Producer creates items into a bounded buffer; consumer removes them. Constraints: producer waits if buffer full, consumer waits if buffer empty, buffer access must be mutually exclusive. Solution uses 3 semaphores: mutex (=1) for mutual exclusion, empty (=N) for empty slots, full (=0) for filled slots. Critical: wait(empty/full) BEFORE wait(mutex) to avoid deadlock.

### Q17: What is a TLB and how does it improve performance?
**A:** TLB (Translation Lookaside Buffer) is a small, fast hardware cache for recently used page table entries. Without TLB, every memory access requires 2 accesses (page table + data). With TLB, a hit requires only 1 access (TLB is checked in parallel). EAT = hit_ratio × (TLB + mem) + miss_ratio × (TLB + 2×mem). Typical hit ratio: 99% → near-zero overhead.

### Q18: What is a zombie process? What is an orphan process?
**A:** Zombie: child process terminated but parent hasn't called wait() — entry remains in process table consuming PID. Fix: parent calls wait(), signal handler for SIGCHLD, or double fork. Orphan: parent terminated first — child adopted by init (PID 1), which automatically wait()s. Orphans are harmless; zombies can exhaust PIDs.

### Q19: Compare SCAN and C-SCAN disk scheduling.
**A:** SCAN (elevator): head moves in one direction servicing requests, then reverses. Problem: requests at the edges just visited wait longest. C-SCAN: head services in one direction, then jumps back to start (no servicing on return). Provides more uniform wait time — requests at both ends wait similar time.

### Q20: What is the difference between fork() and exec()?
**A:** fork() creates a new child process as a copy of the parent (same code, duplicated memory via COW). Returns 0 to child, child PID to parent. exec() replaces the current process's code/data with a new program — same PID, completely different program. Common pattern: fork() + exec() = shell runs commands.

---

## Advanced Interview Questions (Product-Based Companies)

### Q21: How does Linux CFS scheduler work?
**A:** CFS (Completely Fair Scheduler) uses a red-black tree sorted by "virtual runtime" (vruntime). Each process has a vruntime that increases as it uses CPU. The process with the lowest vruntime (least CPU time so far) gets scheduled next. This ensures fairness — each process gets proportional CPU share. Nice values adjust the rate at which vruntime increases (lower nice = slower vruntime growth = more CPU).

### Q22: Explain Copy-on-Write in fork().
**A:** When fork() is called, the child's pages initially point to the same physical frames as the parent (marked read-only). No actual copying occurs. When either process tries to write to a page, a page fault occurs, the OS copies just that page, and gives the writer its own copy. This makes fork() nearly instantaneous and saves memory when followed by exec() (child replaces its entire address space anyway).

### Q23: How does a mutex handle priority inversion?
**A:** Priority inversion: high-priority process H waits for a lock held by low-priority process L, while medium-priority process M runs (M preempts L, so H waits even longer). Solutions: (1) Priority Inheritance — L temporarily inherits H's priority while holding the lock, preventing M from preempting. (2) Priority Ceiling — lock's priority = highest priority of any process that may use it.

### Q24: Explain memory-mapped file I/O (mmap).
**A:** mmap() maps a file directly into the process's virtual address space. The file's contents appear as part of memory — reads/writes to that memory region automatically read/write the file. Benefits: no system call overhead for each read/write, kernel and user share same physical pages (zero-copy), efficient for random access. Used for: shared libraries, database storage engines, interprocess shared memory.

### Q25: What happens during a page fault at the hardware level?
**A:** (1) CPU generates address → MMU checks page table → invalid bit set → hardware trap to OS. (2) CPU saves current state (registers, PC). (3) OS page fault handler identifies the faulting address (from CR2 register on x86). (4) OS checks if address is valid (in process's address space). (5) If valid: find/evict a frame, issue disk I/O to load page. (6) Update page table entry (frame number, valid bit). (7) Flush TLB entry. (8) Restore process state and re-execute the faulting instruction.

### Q26: How do real-time operating systems guarantee deadlines?
**A:** RTOS features: (1) Deterministic scheduling — worst-case execution time (WCET) is known. (2) Priority-based preemptive scheduling — highest priority always runs. (3) Minimal interrupt latency — interrupt handling is fast and bounded. (4) No virtual memory (no unpredictable page faults). (5) Rate Monotonic Scheduling — shorter period = higher priority. (6) Earliest Deadline First — task with nearest deadline runs. (7) Memory locking — critical pages locked in RAM.

### Q27: Explain the differences between processes in Linux, Windows, and macOS.
**A:** Linux: processes created via fork()/clone(); threads via clone() with shared flags; everything is a task_struct internally. Windows: processes via CreateProcess() (not fork-based); threads via CreateThread(); separate process and thread objects. macOS (XNU): Mach microkernel tasks + BSD processes layered; supports both POSIX (fork/exec) and Mach APIs; Grand Central Dispatch (GCD) for concurrency.

### Q28: What is a spinlock and when is it preferred over a mutex?
**A:** Spinlock: process busy-waits (loops) checking if lock is free — wastes CPU but avoids context switch overhead. Preferred when: (1) critical section is very short (shorter than context switch time), (2) multi-core system (spinning on one core while lock holder runs on another), (3) in kernel interrupt handlers (can't sleep while handling interrupts). Never use on single-core (spinning wastes the only CPU, lock holder can never run).

### Q29: How does Linux handle memory overcommit?
**A:** Linux allows processes to allocate more virtual memory than physically available (overcommit). malloc() succeeds but physical pages aren't allocated until accessed (lazy allocation). If the system runs out of memory, the OOM (Out-of-Memory) Killer selects and kills a process to free memory. The OOM score considers process size, priority, and other factors. This can be configured via /proc/sys/vm/overcommit_memory.

### Q30: Explain how a shell executes a piped command like `ls | grep .txt | wc -l`.
**A:** Shell creates 2 pipes. For 3 commands: fork() 3 child processes. Child 1 (ls): redirect stdout to pipe1's write end, exec("ls"). Child 2 (grep): redirect stdin from pipe1's read end, stdout to pipe2's write end, exec("grep .txt"). Child 3 (wc): redirect stdin from pipe2's read end, exec("wc -l"). Parent wait()s for all 3. Pipes are unidirectional, read/write ends connected between processes.

---

## Tricky Conceptual Questions

### Q31: Can a single-threaded process have a deadlock?
**A:** Not a traditional deadlock (needs circular wait among multiple entities). But a single thread CAN deadlock by trying to acquire a non-recursive mutex it already holds — it blocks forever waiting for itself to release the lock. This is a self-deadlock.

### Q32: If we increase the number of frames, will page faults always decrease?
**A:** For most algorithms (LRU, OPT) — yes, more frames → fewer faults. But for FIFO, **Belady's Anomaly** can cause MORE faults with more frames! This is why FIFO is generally not preferred.

### Q33: Is Round Robin always fair?
**A:** It's fair in terms of CPU time allocation (equal time quanta), but not necessarily in terms of throughput or completion time. CPU-bound processes benefit more because they use full quanta; I/O-bound processes often give up CPU early (before quantum expires), effectively getting less CPU time proportionally.

### Q34: Can a process in the waiting state be killed?
**A:** Yes — SIGKILL (9) terminates a process regardless of its state. However, processes in uninterruptible sleep (D state in Linux — typically waiting for I/O) cannot be killed until the I/O completes. This is the only state where even SIGKILL has no immediate effect.

### Q35: Why does paging eliminate external fragmentation but not internal?
**A:** Paging uses fixed-size pages/frames — any free frame can satisfy any page request, so there are never "holes" between allocations (no external fragmentation). But the last page of a process may not be fully used — if a process needs 4.1 pages, it gets 5 pages, wasting 0.9 pages of space (internal fragmentation).

---

## Rapid-Fire Questions

| Question | Answer |
|----------|--------|
| Smallest schedulable unit? | Thread |
| What does PCB contain? | PID, state, PC, registers, scheduling info, memory info, I/O status |
| n fork() calls = how many processes? | 2^n |
| SRTF is preemptive version of? | SJF |
| Which scheduling algorithm is optimal? | SRTF (shortest avg waiting time) |
| Page fault rate high = ? | Thrashing |
| Page + Frame size relationship? | Always equal |
| LRU suffers from Belady's anomaly? | No (it's a stack algorithm) |
| Which algorithm does Linux use? | CFS (Completely Fair Scheduler) |
| fork() returns what to child? | 0 |
| fork() returns what to parent? | Child's PID |
| Zombie vs orphan — which is harmful? | Zombie (consumes PID and table entry) |
| Semaphore invented by? | Dijkstra |
| Deadlock conditions count? | 4 (Coffman conditions) |
| Banker's algo type? | Deadlock avoidance |
| What is aging? | Gradually increase priority of waiting processes |
| SCAN algorithm also called? | Elevator algorithm |
| Mutex has ownership? | Yes |
| Semaphore has ownership? | No |
| Signal that can't be caught? | SIGKILL (9) and SIGSTOP (19) |
| chmod 777 means? | rwxrwxrwx (full permissions for all) |
| Logical address generated by? | CPU |
| Physical address generated by? | MMU |
| Copy-on-Write used in? | fork() |
| TLB is cache for? | Page table entries |

---

# COMPARISON TABLES

## Table 1: Process vs Thread

| Feature | Process | Thread |
|---------|---------|--------|
| Definition | Program in execution | Lightweight unit within process |
| Memory | Own address space | Shares process memory |
| Creation | Slow (heavy) | Fast (10-100x lighter) |
| Context switch | Expensive (TLB flush, page table change) | Cheap (shared address space) |
| Communication | IPC (pipes, sockets, shared memory) | Direct (shared data) |
| Isolation | Full isolation (crash is contained) | Not isolated (crash kills all threads) |
| Example | Chrome tabs | Word processor threads |

## Table 2: Paging vs Segmentation

| Feature | Paging | Segmentation |
|---------|--------|-------------|
| Division | Fixed-size pages | Variable-size segments |
| Basis | Physical (arbitrary) | Logical (code, data, stack) |
| Fragmentation | Internal only | External only |
| Address format | Page# + Offset | Segment# + Offset |
| Table | Page table (frame#) | Segment table (base + limit) |
| User visible | No (transparent) | Yes (programmer-defined) |
| Sharing | Harder (page granularity) | Easier (share a segment) |

## Table 3: Mutex vs Semaphore

| Feature | Mutex | Semaphore |
|---------|-------|-----------|
| Type | Locking mechanism | Signaling mechanism |
| Values | 0 or 1 (binary) | 0 to N (counting) |
| Ownership | Yes (only owner unlocks) | No (any process can signal) |
| Purpose | Mutual exclusion | Mutual exclusion + sync + counting |
| Priority inheritance | Supported | Not supported |
| Analogy | Bathroom key | Parking lot counter |

## Table 4: User Thread vs Kernel Thread

| Feature | User Thread | Kernel Thread |
|---------|-------------|---------------|
| Managed by | Thread library (user space) | OS kernel |
| OS awareness | Invisible to OS | OS schedules them |
| Creation | Very fast (no system call) | Slower (system call) |
| Blocking | One blocks → all block | Only blocking thread waits |
| Multi-core | Cannot use | Can use multiple cores |
| Example | Green threads | pthreads, Windows threads |

## Table 5: FCFS vs SJF

| Feature | FCFS | SJF |
|---------|------|-----|
| Selection | Arrival order | Shortest burst time |
| Preemptive? | No | No (SRTF is preemptive version) |
| Starvation? | No | Yes (long processes) |
| Optimal? | No | Yes (min avg WT for non-preemptive) |
| Key problem | Convoy effect | Starvation + unknown burst times |
| Complexity | Simple | Needs burst prediction |

## Table 6: Internal vs External Fragmentation

| Feature | Internal | External |
|---------|----------|----------|
| Where | Inside allocated block | Between allocated blocks |
| Cause | Block > process need | Many scattered free blocks |
| Occurs in | Paging, fixed partitioning | Segmentation, variable partitioning |
| Solution | Smaller pages | Compaction, paging |

## Table 7: Monolithic Kernel vs Microkernel

| Feature | Monolithic | Microkernel |
|---------|-----------|-------------|
| Structure | Everything in kernel | Minimal kernel |
| Contains | All OS services | Only IPC, scheduling, memory |
| Performance | Fast (no IPC overhead) | Slower (user-space services need IPC) |
| Reliability | One bug crashes all | Isolated failures |
| Size | Large | Small |
| Example | Linux | Minix, QNX, L4 |

## Table 8: Multitasking vs Multithreading

| Feature | Multitasking | Multithreading |
|---------|-------------|----------------|
| Unit | Processes | Threads |
| Memory | Separate address spaces | Shared address space |
| Overhead | Higher (process switch) | Lower (thread switch) |
| Communication | IPC needed | Direct memory access |
| Example | Running browser + terminal | Browser tabs using threads |

## Table 9: Deadlock Prevention vs Avoidance

| Feature | Prevention | Avoidance |
|---------|-----------|-----------|
| Approach | Break one of 4 conditions | Don't enter unsafe state |
| When | Design time | Runtime |
| Algorithm | Ordering, no hold-and-wait | Banker's Algorithm |
| Resource utilization | Low (conservative) | Better (more flexible) |
| Overhead | Low runtime overhead | High (check on every request) |
| Practical? | Sometimes restrictive | Needs max resource info in advance |
