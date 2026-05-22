# 🖥️ PART 7: REVISION, PRACTICALS, STUDY PLANS & RESOURCES

---

# INTERVIEW REVISION SECTION

---

## 7.1 Last-Minute Revision Notes

### OS Core Concepts
```
OS = resource manager + abstraction layer between user and hardware
Kernel = core of OS, always in memory, manages everything
User mode (restricted) ←→ Kernel mode (privileged) via system calls/interrupts
Monolithic = all in kernel (fast, Linux) | Micro = minimal kernel (reliable, QNX)
```

### Processes & Threads
```
Process = program in execution | Thread = lightweight unit within process
PCB = process identity card (PID, state, PC, registers, memory, I/O)
States: New → Ready → Running → Waiting → Terminated
fork() returns: 0 to child, child PID to parent | n forks = 2^n processes
Zombie = child done, parent didn't wait() | Orphan = parent died, init adopts
Thread shares: code, data, heap, files | Owns: stack, PC, registers
```

### CPU Scheduling
```
TAT = CT - AT | WT = TAT - BT | RT = First CPU - AT
FCFS: simple, convoy effect, no starvation
SJF: optimal non-preemptive WT | SRTF: optimal overall WT (preemptive)
Priority: starvation → solved by aging
RR: fair, time quantum q matters (too small = overhead, too large = FCFS)
MLFQ: processes move between queues, most practical
Linux CFS: red-black tree, virtual runtime, proportional fairness
```

### Synchronization
```
Race condition = result depends on execution order
CS needs: Mutual Exclusion + Progress + Bounded Waiting
Mutex: binary lock WITH ownership | Semaphore: counting signal WITHOUT ownership
P(wait)/V(signal) — atomic operations
Producer-Consumer: mutex(1) + empty(N) + full(0) — ORDER MATTERS!
Dining Philosophers: 5 forks, circular deadlock risk
```

### Deadlocks
```
4 conditions: Mutual Exclusion + Hold&Wait + No Preemption + Circular Wait
RAG: cycle + single instance = deadlock guaranteed
Banker's: Need = Max - Allocation, find safe sequence where Need ≤ Available
Prevention = break a condition | Avoidance = Banker's | Detection = wait-for graph
Most OSes: ostrich algorithm (ignore)
```

### Memory
```
Logical (CPU) → MMU → Physical (RAM)
Paging: fixed pages/frames, no external fragmentation, internal fragmentation on last page
Page table: page# → frame# | TLB: fast cache for page table
EAT = h(t+m) + (1-h)(t+2m) — h=hit ratio, t=TLB time, m=memory time
Segmentation: variable segments, external fragmentation, logical division
```

### Virtual Memory
```
Demand paging: load pages only when needed
Page fault: page not in RAM → trap → load from disk → restart instruction
FIFO: oldest out, Belady's anomaly | LRU: least recently used, no Belady's
Optimal: farthest future use, impossible | Clock: approximate LRU (reference bit)
Thrashing: too many faults, CPU utilization drops → working set model
COW: fork shares pages, copy only on write
```

### Files & Disk
```
Allocation: Contiguous (fast, external frag) | Linked (no frag, slow random) | Indexed (inode)
FCFS→SSTF→SCAN→C-SCAN→LOOK→C-LOOK disk scheduling
LOOK/C-LOOK most practical (no wasted travel to disk ends)
```

---

## 7.2 Important Formulas

| Formula | Context |
|---------|---------|
| `TAT = Completion Time - Arrival Time` | Scheduling |
| `Waiting Time = TAT - Burst Time` | Scheduling |
| `Response Time = First CPU Time - Arrival Time` | Scheduling |
| `Efficiency (S&W) = 1 / (1 + 2a)` where a = Tp/Tt | Stop-and-Wait flow control |
| `Number of processes after n fork() = 2^n` | Fork counting |
| `Pages = Address Space / Page Size` | Paging |
| `Page table entries = 2^(address bits - offset bits)` | Page table size |
| `Offset bits = log₂(Page Size)` | Address translation |
| `Physical Address = Frame × Page Size + Offset` | Address translation |
| `EAT = h(t+m) + (1-h)(t+2m)` | TLB effective access time |
| `EAT = (1-p) × ma + p × pf_time` | Page fault effective access time |
| `Need = Max - Allocation` | Banker's Algorithm |
| `Speedup = 1 / ((1-P) + P/N)` | Amdahl's Law |
| `GBN sender window ≤ 2^n - 1` | Sliding window |
| `SR sender window ≤ 2^(n-1)` | Selective repeat |

---

## 7.3 Important Algorithms Summary

| Algorithm | Used For | Key Idea |
|-----------|----------|----------|
| **FCFS** | CPU/Disk scheduling | First come, first served |
| **SJF/SRTF** | CPU scheduling | Shortest job/remaining time |
| **Round Robin** | CPU scheduling | Fixed time quantum, rotate |
| **Banker's** | Deadlock avoidance | Find safe sequence |
| **FIFO** | Page replacement | Oldest page out |
| **LRU** | Page replacement | Least recently used out |
| **Optimal** | Page replacement | Farthest future use out (theoretical) |
| **Clock** | Page replacement | Reference bit, second chance |
| **SCAN/C-SCAN** | Disk scheduling | Elevator / circular elevator |
| **Peterson's** | Mutual exclusion | 2-process software solution |
| **Dijkstra** | Routing (OSPF) | Shortest path first |
| **Bellman-Ford** | Routing (RIP) | Distance vector |

---

## 7.4 Memory Tricks & Mnemonics

### Process States
**"New Ready Running Waiting Terminated" → "Never Run Red Wagons Twice"**

### Deadlock Conditions
**"MH-NC" → "My House, No Cleaning"**
- **M**utual Exclusion, **H**old and Wait, **N**o Preemption, **C**ircular Wait

### OSI Layers (also useful for OS networking)
**"Please Do Not Throw Sausage Pizza Away"** (bottom-up)

### Scheduling Algorithms Order (by optimality)
**"FCFS is Fair, SJF is Smart, SRTF is Superior, RR is Reasonable"**

### Page Replacement
**"FIFO is Flawed (Belady's), LRU is Logical, OPT is Oracle"**

### Semaphore Operations
**"P = Probeer (try/wait/down/decrement) | V = Verhoog (signal/up/increment)"**
Or remember: **"P = Put me to sleep | V = V-ake me up"**

### fork() Returns
**"Zero for the Zero-age (child), PID for the Parent"**

### Disk Scheduling
**"FCFS → SSTF → SCAN → C-SCAN → LOOK → C-LOOK"**
**"Fundamental → Smart → Sweep → Circular Sweep → Lazy → Circular Lazy"**

---

## 7.5 Frequently Forgotten Concepts

1. **Page size = Frame size** always (this is the fundamental paging invariant)
2. **SRTF is optimal** for average waiting time, not SJF (SJF is optimal only among non-preemptive)
3. **Belady's anomaly** affects ONLY FIFO, not LRU or OPT (they're stack algorithms)
4. **fork() creates a COPY** — child gets its own address space (COW optimization)
5. **exec() doesn't create a new process** — it replaces the current process image
6. **Mutex has ownership** (only owner unlocks); **semaphore does NOT** (any process can signal)
7. **Deadlock needs ALL 4 conditions** simultaneously — breaking any one prevents it
8. **Aging solves starvation** in priority scheduling, NOT deadlock
9. **TLB miss costs 2 memory accesses** (page table + data); TLB hit costs 1
10. **Internal fragmentation = inside** allocated block (paging); **External = between** blocks (segmentation)
11. **SIGKILL (9) and SIGSTOP (19) cannot be caught or handled**
12. **Context switch overhead** includes saving/restoring registers, TLB flush (for process switch), cache invalidation
13. **Virtual memory ≠ swap space** — virtual memory is the concept; swap is the disk area used
14. **Working set model** prevents thrashing by ensuring each process has enough frames
15. **CFS (Linux)** doesn't use priorities directly — it uses virtual runtime weighted by nice value

---

## 7.6 Revision Strategies

### 1-Day Revision Strategy (8 hours)

| Time | Activity |
|------|----------|
| 9:00-10:00 | Processes, threads, PCB, states, fork() |
| 10:00-11:30 | CPU scheduling — all algorithms, solve 3 Gantt chart problems |
| 11:30-12:30 | Synchronization — mutex, semaphore, producer-consumer, dining philosophers |
| 12:30-1:00 | Lunch |
| 1:00-2:00 | Deadlocks — 4 conditions, Banker's algorithm (solve 1 problem) |
| 2:00-3:00 | Memory — paging, TLB, page replacement (solve 2 problems) |
| 3:00-3:30 | File systems + disk scheduling |
| 3:30-4:30 | ALL comparison tables (study, then try to recreate from memory) |
| 4:30-5:30 | Interview questions — practice answering aloud |
| 5:30-6:00 | Rapid fire + formulas + mnemonics |

### 7-Day Revision Strategy

| Day | Focus | Time |
|-----|-------|------|
| 1 | OS basics + Process + Thread fundamentals | 3 hrs |
| 2 | CPU scheduling (all algorithms + 5 numerical problems) | 3-4 hrs |
| 3 | Synchronization + Classical problems | 3 hrs |
| 4 | Deadlocks + Banker's algorithm (3 problems) | 3 hrs |
| 5 | Memory management + Virtual memory + Page replacement (5 problems) | 3-4 hrs |
| 6 | File systems + Disk scheduling + Linux commands | 3 hrs |
| 7 | All comparison tables + interview questions + rapid fire + revision | 4 hrs |

### 30-Day Mastery Plan

| Week | Days | Topics | Daily Hours |
|------|------|--------|-------------|
| 1 | 1-7 | OS basics, Processes, Threads, Scheduling | 2-3 |
| 2 | 8-14 | Synchronization, Classical problems, Deadlocks | 2-3 |
| 3 | 15-21 | Memory management, Virtual memory, Page replacement | 2-3 |
| 4 | 22-28 | Files, Disk, Linux, Interview prep + practice | 2-3 |
| Final | 29-30 | Full revision + mock interviews | 4 |

---

# PRACTICAL SECTION

---

## Synchronization Coding Problems (C/C++/Python)

### 1. Mutex with Pthreads (C)
```c
#include <pthread.h>
#include <stdio.h>

int counter = 0;
pthread_mutex_t lock;

void* increment(void* arg) {
    for (int i = 0; i < 1000000; i++) {
        pthread_mutex_lock(&lock);
        counter++;
        pthread_mutex_unlock(&lock);
    }
    return NULL;
}

int main() {
    pthread_t t1, t2;
    pthread_mutex_init(&lock, NULL);
    
    pthread_create(&t1, NULL, increment, NULL);
    pthread_create(&t2, NULL, increment, NULL);
    
    pthread_join(t1, NULL);
    pthread_join(t2, NULL);
    
    printf("Counter: %d\n", counter);  // Should be 2000000
    pthread_mutex_destroy(&lock);
    return 0;
}
```

### 2. Producer-Consumer with Semaphores (Python)
```python
import threading
import time
import random

buffer = []
BUFFER_SIZE = 5
empty = threading.Semaphore(BUFFER_SIZE)  # empty slots
full = threading.Semaphore(0)              # filled slots
mutex = threading.Lock()                    # mutual exclusion

def producer():
    for i in range(10):
        item = random.randint(1, 100)
        empty.acquire()      # Wait for empty slot
        mutex.acquire()      # Lock buffer
        buffer.append(item)
        print(f"Produced: {item}, Buffer: {buffer}")
        mutex.release()      # Unlock
        full.release()       # Signal item available
        time.sleep(random.random())

def consumer():
    for i in range(10):
        full.acquire()       # Wait for item
        mutex.acquire()      # Lock buffer
        item = buffer.pop(0)
        print(f"Consumed: {item}, Buffer: {buffer}")
        mutex.release()      # Unlock
        empty.release()      # Signal empty slot
        time.sleep(random.random())

t1 = threading.Thread(target=producer)
t2 = threading.Thread(target=consumer)
t1.start(); t2.start()
t1.join(); t2.join()
```

---

## OS Mini Project Ideas

### Project 1: CPU Scheduling Simulator ⭐
- Input: processes with arrival time, burst time, priority
- Implement: FCFS, SJF, SRTF, Priority, Round Robin
- Output: Gantt chart, TAT, WT, RT for each process
- **Tech:** Python or C
- **Why:** Directly tests interview knowledge; great portfolio project

### Project 2: Page Replacement Simulator
- Input: reference string, number of frames
- Implement: FIFO, LRU, Optimal, Clock
- Output: Page fault count, hit ratio, frame state at each step
- Show Belady's anomaly for FIFO

### Project 3: Banker's Algorithm Visualizer
- Input: allocation matrix, max matrix, available vector
- Detect safe/unsafe state
- Find safe sequence (if exists)
- Show step-by-step resource allocation

### Project 4: Memory Allocation Simulator
- Simulate First Fit, Best Fit, Worst Fit
- Show fragmentation (internal/external)
- Visualize memory blocks

### Project 5: Shell Implementation
- Build a basic Unix shell in C
- Support: commands, arguments, pipes (|), redirection (>, <)
- Implement using fork(), exec(), wait(), pipe()
- **Excellent for understanding process creation!**

### Project 6: Disk Scheduling Visualizer
- Input: request queue, head position, direction
- Implement: FCFS, SSTF, SCAN, C-SCAN, LOOK, C-LOOK
- Output: Head movement path, total seek distance
- Visualize head movement on a number line

### Project 7: Thread Pool Implementation
- Create a pool of N worker threads
- Submit tasks to a queue
- Threads pick up and execute tasks
- Demonstrate synchronization with mutex + condition variables

---

# STUDY PLANS

---

## 30-Day Intensive OS Roadmap

| Day | Topic | Key Activities |
|-----|-------|---------------|
| 1 | What is an OS | Functions, goals, types of OS |
| 2 | OS types + Multi concepts | Batch, time-sharing, RTOS, multiprogramming vs multitasking |
| 3 | Kernel | User/kernel mode, monolithic vs micro, mode switching |
| 4 | Processes | Concept, PCB, memory layout, states, transitions |
| 5 | Context switching + fork() | How switching works, fork tree problems (solve 5+) |
| 6 | Threads | Concept, multithreading, user vs kernel threads, models |
| 7 | Process vs Thread | Deep comparison, interview practice, review week 1 |
| 8 | Scheduling basics | Criteria, preemptive vs non-preemptive, FCFS |
| 9 | SJF + SRTF | Algorithms + solve 3 Gantt chart problems each |
| 10 | Priority + Round Robin | Algorithms + solve 3 problems each, aging concept |
| 11 | MLQ + MLFQ + Review | Advanced schedulers, Linux CFS, solve 5 mixed problems |
| 12 | Race conditions + CS | Critical section, 3 requirements, Peterson's solution |
| 13 | Mutex + Semaphore | Both types, P/V operations, differences |
| 14 | Classical problems | Producer-consumer, readers-writers, dining philosophers |
| 15 | Deadlock fundamentals | 4 conditions, RAG, prevention strategies |
| 16 | Banker's Algorithm | Worked examples, solve 3 problems, avoidance vs prevention |
| 17 | Detection + recovery | Wait-for graph, recovery methods, review sync+deadlock |
| 18 | Memory basics | Address binding, logical vs physical, contiguous allocation |
| 19 | Paging | Concept, address translation, solve 5 problems |
| 20 | Page table + TLB | Multi-level tables, TLB, EAT calculation (3 problems) |
| 21 | Segmentation | Concept, paging vs segmentation comparison |
| 22 | Virtual memory | Demand paging, page fault handling steps |
| 23 | Page replacement: FIFO + OPT | Algorithms + solve 3 problems, Belady's anomaly |
| 24 | Page replacement: LRU + Clock | Algorithms + solve 3 problems |
| 25 | Thrashing + review | Working set, review all memory topics |
| 26 | File systems | Allocation methods, directories, inodes |
| 27 | Disk scheduling | All algorithms + solve 2 head movement problems |
| 28 | System calls + Linux | fork/exec/wait/pipe, essential Linux commands |
| 29 | All comparison tables + interview Qs | Study all 9 tables, practice 20+ interview questions |
| 30 | Final revision | Formulas, mnemonics, rapid fire, mock interview |

---

## 60-Day Standard OS Roadmap

| Week | Topics | Activities |
|------|--------|-----------|
| 1 | OS basics + Kernel | Types, modes, monolithic vs micro |
| 2 | Processes + Threads | PCB, states, fork, threads, models |
| 3 | CPU Scheduling (Part 1) | FCFS, SJF, SRTF (10+ Gantt chart problems) |
| 4 | CPU Scheduling (Part 2) | Priority, RR, MLQ, MLFQ, CFS |
| 5 | Synchronization | Race conditions, mutex, semaphore, Peterson's |
| 6 | Classical problems + Spinlocks | Producer-consumer, readers-writers, dining philosophers |
| 7 | Deadlocks | 4 conditions, RAG, prevention, Banker's (5+ problems) |
| 8 | Memory Management | Paging, segmentation, address translation (10+ problems) |
| 9 | Virtual Memory | Demand paging, page replacement (10+ problems) |
| 10 | Thrashing + Advanced Memory | Working set, COW, memory-mapped I/O |
| 11 | Files + Disk Scheduling | Allocation, inodes, all disk algorithms (5+ problems) |
| 12 | Linux + System Calls | Commands, fork/exec/wait, shell basics |
| 13 | Advanced topics | Virtualization, containers, security |
| 14 | Interview prep | All comparison tables, 30+ questions practice |
| 15 | Mini project | CPU scheduling or page replacement simulator |
| 16 | Revision + mock | Full revision, rapid fire, mock interviews |
| 17 | Buffer | Weak areas, additional numericals |

---

## 90-Day Comprehensive OS Roadmap

| Week | Topics | Activities | Hrs/Day |
|------|--------|-----------|---------|
| 1-2 | Fundamentals + Kernel | Basics through kernel architecture | 1.5 |
| 3-4 | Processes + Threads | Full process lifecycle, threading models, fork problems | 1.5 |
| 5-6 | CPU Scheduling | All algorithms, 20+ Gantt chart problems | 2 |
| 7-8 | Synchronization | All concepts, code 3 classical problems in C/Python | 2 |
| 9-10 | Deadlocks | Theory + Banker's algorithm (10+ problems) | 2 |
| 11-12 | Memory Management | Paging, TLB, segmentation (15+ problems) | 2 |
| 13-14 | Virtual Memory | Page replacement (15+ problems), thrashing | 2 |
| 15-16 | Files + Disk | Allocation, disk scheduling (10+ problems) | 1.5 |
| 17-18 | Linux Mastery | Shell scripting, system calls, strace practice | 2 |
| 19-20 | Mini Project 1 | CPU Scheduling Simulator (build from scratch) | 2 |
| 21-22 | Mini Project 2 | Simple Shell implementation in C | 2 |
| 23 | Advanced Topics | Virtualization, containers, OS security | 1.5 |
| 24-25 | Interview Prep (Beginner-Intermediate) | 25+ questions, all comparison tables | 2 |
| 26-27 | Interview Prep (Advanced) | Product-company questions, tricky Qs | 2 |
| 28-29 | Mock Interviews | Practice with peers, record yourself | 2 |
| 30 | Full Revision | All notes, formulas, rapid fire | 3 |

---

# RESOURCES

---

## 📺 Best YouTube Channels

| Channel | Best For | Language |
|---------|----------|---------|
| **Gate Smashers** | Gate/placement OS (highly recommended) | Hindi |
| **Neso Academy** | Clear theory, good visuals | English |
| **Jenny's Lectures** | University-style, detailed | Hindi/English |
| **Knowledge Gate** | Gate exam focus | Hindi |
| **University of Wisconsin (Remzi)** | OSTEP textbook lectures | English |
| **MIT OCW (6.828/6.S081)** | Advanced OS (xv6 labs) | English |
| **Abdul Bari** | Algorithms used in OS | English |
| **CodeWithHarry (OS playlist)** | Quick Hindi tutorials | Hindi |
| **Hussein Nasser** | System design + OS concepts | English |

## 📚 Best Books

| Book | Author | Best For |
|------|--------|----------|
| **Operating System Concepts (Dinosaur Book)** | Silberschatz, Galvin, Gagne | Standard university textbook — comprehensive |
| **Operating Systems: Three Easy Pieces (OSTEP)** | Remzi & Andrea Arpaci-Dusseau | **FREE online**, best for self-study, excellent explanations |
| **Modern Operating Systems** | Andrew Tanenbaum | Deep theory, written by Minix creator |
| **Operating Systems: Principles and Practice** | Anderson & Dahlin | Practical focus, modern |
| **Linux Kernel Development** | Robert Love | Linux internals (advanced) |
| **The Linux Programming Interface** | Michael Kerrisk | System calls, Linux programming |

> [!TIP]
> **Start with OSTEP** (free at [pages.cs.wisc.edu/~remzi/OSTEP/](https://pages.cs.wisc.edu/~remzi/OSTEP/)). It's the best free OS textbook available.

## 🌐 Best Websites

| Website | Use |
|---------|-----|
| **GeeksforGeeks (OS section)** | Theory + interview questions + numericals |
| **OSTEP (free textbook)** | Complete OS learning |
| **JavaTPoint (OS tutorial)** | Structured tutorials |
| **Tutorialspoint OS** | Quick reference |
| **Operating Systems: Design and Implementation** | Minix-based learning |
| **Linux man pages** | System call documentation |
| **OverTheWire (Bandit)** | Linux command-line practice |

## 🔬 OS Simulators & Practice

| Tool | Purpose |
|------|---------|
| **MOSS (Modern OS Simulators)** | CPU scheduling, page replacement, disk scheduling sims |
| **xv6 (MIT)** | Educational Unix-like OS — read and modify kernel code |
| **Nachos** | Teaching OS — implement scheduling, VM, filesystem |
| **PintOS (Stanford)** | Guided OS implementation project |
| **OS Visualizer** | Online CPU scheduling visualization |
| **Process Explorer (Windows)** | Real-time process/thread monitoring |
| **Linux /proc filesystem** | Explore process info on real system |

## 🎯 Interview Resources

| Resource | Type |
|----------|------|
| **InterviewBit (OS section)** | Curated interview questions |
| **GeeksforGeeks Last Minute Notes (OS)** | Quick revision |
| **LeetCode Discuss (OS tag)** | Company-specific questions |
| **Glassdoor** | Company interview experiences |
| **Gate previous year papers** | Numerical problems |

---

## 📝 Final Advice

> [!IMPORTANT]
> **The #1 OS interview topic:** Processes & Threads (Process vs Thread, fork(), states, scheduling). If you master this one topic deeply, you can handle 40%+ of OS interview questions.

> [!TIP]
> **Study strategy for OS:**
> 1. Understand the WHY before the WHAT (why does paging exist? → to eliminate external fragmentation)
> 2. Draw diagrams — process state diagram, page table, Gantt charts
> 3. Solve numericals — scheduling, Banker's, page replacement, disk scheduling
> 4. Code it — write producer-consumer, build a simple shell, implement scheduling simulator
> 5. Explain it aloud — if you can teach it, you know it (Feynman technique)

> [!NOTE]
> **For product-based companies (Google, Amazon, Microsoft):**
> Focus on: Process vs Thread deeply, synchronization (mutex vs semaphore, deadlock), virtual memory, system calls, Linux internals. They want DEPTH and real-world understanding.

> [!NOTE]
> **For service-based companies (TCS, Infosys, Wipro):**
> Focus on: All scheduling algorithms (numericals!), page replacement (numericals!), Banker's algorithm, comparison tables, process states. They test BREADTH and problem-solving.

---

**You now have everything you need. Start with Day 1 of your chosen study plan. Consistency > intensity. Good luck with your placements! 🚀**
