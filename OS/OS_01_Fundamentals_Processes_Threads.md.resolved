# 🖥️ PART 1: OS FUNDAMENTALS, PROCESSES & THREADS

---

# PHASE 1: OPERATING SYSTEM FUNDAMENTALS

## 🎯 Learning Objectives
- Understand what an OS is and why it exists
- Know all types of operating systems
- Master kernel concepts and mode switching
- Distinguish multiprogramming, multiprocessing, multitasking

---

## 1.1 What is an Operating System?

An Operating System is **system software** that acts as an intermediary between the user and computer hardware. It manages hardware resources and provides services to application programs.

**Real-World Analogy:** Think of an OS as the **manager of a restaurant**. The manager doesn't cook food (hardware does the actual work) or eat it (users do), but ensures chefs have ingredients (resource allocation), tables are assigned to customers (process scheduling), bills are correct (security), and everything runs smoothly.

### Why Does an OS Exist?
1. **Abstraction** — Hides complex hardware details from users/programmers
2. **Resource management** — CPU, memory, disk, I/O devices shared fairly
3. **Protection & security** — Prevents one program from crashing another
4. **Convenience** — Provides user-friendly interface to hardware

### Functions of an OS

| Function | What It Does |
|----------|-------------|
| **Process management** | Create, schedule, terminate processes |
| **Memory management** | Allocate/deallocate RAM, virtual memory |
| **File management** | Create, delete, organize files/directories |
| **Device management** | Manage I/O devices via drivers |
| **Security** | Authentication, access control, protection |
| **Networking** | TCP/IP stack, socket management |
| **User interface** | CLI (shell) or GUI (desktop) |

### Goals of an OS
- **Primary:** Convenience (easy to use)
- **Secondary:** Efficiency (optimal resource utilization)

---

## 1.2 Types of Operating Systems

### Batch Operating System
- **How:** Jobs collected, grouped (batched), processed sequentially
- **No user interaction** during execution
- **Problem:** Long wait times, no priority, one job failure holds up batch
- **Example:** Early mainframes (1960s), payroll processing

### Time-Sharing Operating System (Multitasking)
- **How:** CPU time divided into small **time quanta** (slices), each user gets a turn
- **Goal:** Interactive response — each user feels like they have the whole computer
- **Example:** Unix, Linux
- **Key:** Context switching between tasks so fast that users don't notice

### Real-Time Operating System (RTOS)
- **How:** Tasks must complete within **strict time constraints** (deadlines)
- **Hard RTOS:** Missing deadline = catastrophic failure (pacemaker, missile system, ABS brakes)
- **Soft RTOS:** Missing deadline = degraded quality but not failure (video streaming, VoIP)
- **Example:** VxWorks, FreeRTOS, QNX

### Distributed Operating System
- **How:** Multiple independent computers appear as a single system to users
- **Goal:** Resource sharing, fault tolerance, scalability
- **Example:** Google's infrastructure, Hadoop clusters

### Network Operating System
- **How:** Server manages shared resources for connected clients
- **Example:** Windows Server, Novell NetWare

---

## 1.3 Key Multi- Concepts

| Concept | Definition | Key Insight |
|---------|-----------|-------------|
| **Multiprogramming** | Multiple programs loaded in memory simultaneously | CPU never idle — when one program waits (I/O), another runs |
| **Multiprocessing** | Multiple CPUs/cores execute processes simultaneously | True parallelism (not just concurrency) |
| **Multitasking** | Single CPU switches between tasks rapidly (time-sharing) | Concurrent execution via context switching (appears parallel) |
| **Multithreading** | Multiple threads within a single process execute concurrently | Lightweight — threads share process memory |

**Interview Trap — Concurrency vs Parallelism:**
- **Concurrency:** Multiple tasks in progress (may not execute simultaneously — interleaved on 1 CPU)
- **Parallelism:** Multiple tasks executing at the exact same time (requires multiple CPUs/cores)
- **Analogy:** Concurrency = one chef juggling multiple dishes. Parallelism = multiple chefs each cooking a dish.

---

## 1.4 The Kernel

### What is the Kernel?
The kernel is the **core of the OS** — it's the first program loaded at boot and stays in memory permanently. It directly manages hardware and provides services to all other software.

**Analogy:** If the OS is a government, the kernel is the **president/prime minister** — has ultimate authority over all resources.

### User Mode vs Kernel Mode ⭐

```
┌─────────────────────────────────┐
│         USER MODE               │  ← Applications run here
│  (Restricted access)            │  ← Cannot access hardware directly
│  Apps, Shell, Libraries         │  ← If tries → TRAP → switches to kernel
├─────────────────────────────────┤  ← Mode bit: 1 (user) → 0 (kernel)
│         KERNEL MODE             │  ← OS code runs here
│  (Privileged access)            │  ← Full hardware access
│  Process mgmt, Memory, Drivers  │  ← Critical OS operations
└─────────────────────────────────┘
```

**Mode Bit:** Hardware register — 0 = kernel mode, 1 = user mode

**Why Two Modes?**
- **Protection:** Prevents user programs from directly accessing hardware and corrupting OS data
- A buggy application can crash itself but NOT the entire system
- System calls are the controlled gateway from user mode to kernel mode

**When does mode switch happen?**
1. **System call** (process requests OS service) — user → kernel
2. **Interrupt** (hardware signals) — user → kernel
3. **Exception/trap** (divide by zero, page fault) — user → kernel
4. **Return from system call** — kernel → user

**🎤 Interview Q:** *What happens if a user program tries to execute a privileged instruction?*
**A:** The CPU detects the violation (mode bit = 1 but instruction requires mode bit = 0), generates a **trap/exception**, and the OS terminates the offending process. This is how protection works.

---

### Monolithic Kernel vs Microkernel ⭐

| Feature | Monolithic Kernel | Microkernel |
|---------|-------------------|-------------|
| **Structure** | All OS services in one large kernel | Only essential services in kernel; rest in user space |
| **Kernel contains** | Process mgmt, memory, file system, drivers, networking — EVERYTHING | Only IPC, basic scheduling, memory management |
| **Performance** | Fast (no context switches between components) | Slower (frequent IPC between user-space services) |
| **Size** | Large | Small |
| **Reliability** | One bug can crash entire OS | More reliable — crash in one service doesn't crash kernel |
| **Extensibility** | Hard to extend safely | Easy — add services in user space |
| **Examples** | Linux, Unix, MS-DOS | Minix, QNX, L4, Mach |

**Hybrid Kernel:** Combines both — essential services in kernel, some in user space. Examples: Windows NT, macOS (XNU).

```
MONOLITHIC:                          MICROKERNEL:
┌───────────────────┐                ┌───────────────────┐
│    User Space     │                │    User Space     │
│  Applications     │                │ Apps + File Sys + │
│                   │                │ Drivers + Network │
├───────────────────┤                ├───────────────────┤
│   Kernel Space    │                │   Kernel Space    │
│ File Sys | Network│                │    IPC | Sched    │
│ Drivers | Memory  │                │    Memory Mgmt    │
│ Scheduling | IPC  │                │                   │
└───────────────────┘                └───────────────────┘
   Everything in kernel              Minimal kernel, rest
                                     runs as user processes
```

**🎤 Interview Q:** *Why is Linux monolithic if modularity is better?*
**A:** Linux is monolithic for performance — all kernel services share the same address space, avoiding expensive IPC. However, Linux supports **loadable kernel modules (LKMs)** that can be loaded/unloaded dynamically, giving some modularity benefits while maintaining monolithic speed.

---

# PHASE 2: PROCESSES

## 🎯 Learning Objectives
- Understand process concept, lifecycle, and state transitions
- Know PCB structure and context switching
- Master fork() and exec() system calls
- Solve fork() tree problems

---

## 2.1 What is a Process?

A **process** is a **program in execution**. A program is a passive entity (file on disk); a process is an active entity (loaded in memory, executing).

**Analogy:** A recipe (program) sitting in a cookbook does nothing. When a chef starts cooking using that recipe — measuring ingredients, heating the oven — THAT is the process.

### Process Memory Layout

```
High Address
┌─────────────────┐
│      Stack      │  ← Local variables, function call info, return addresses
│    ↓ grows ↓    │    (grows downward)
├─────────────────┤
│                 │  ← Free space between stack and heap
├─────────────────┤
│    ↑ grows ↑    │
│      Heap       │  ← Dynamic memory (malloc, new)
├─────────────────┤    (grows upward)
│      BSS        │  ← Uninitialized global/static variables
├─────────────────┤
│      Data       │  ← Initialized global/static variables
├─────────────────┤
│      Text       │  ← Program code (machine instructions)
Low Address        │    (read-only, sharable)
└─────────────────┘
```

**🎤 Interview Q:** *What's the difference between stack and heap?*
**A:** Stack: automatic allocation, LIFO, fast, limited size, stores local variables and function frames. Heap: manual allocation (malloc/free), no LIFO constraint, slower, larger, stores dynamically allocated objects. Stack overflow = too many function calls (deep recursion); heap overflow = too many allocations without freeing.

---

## 2.2 Process Control Block (PCB) ⭐

The PCB is the **data structure** the OS maintains for each process — it's the process's "identity card."

| PCB Field | Purpose |
|-----------|---------|
| **Process ID (PID)** | Unique identifier |
| **Process state** | Running, ready, waiting, etc. |
| **Program counter (PC)** | Address of next instruction to execute |
| **CPU registers** | Contents of all registers (saved during context switch) |
| **CPU scheduling info** | Priority, scheduling queue pointers |
| **Memory management info** | Page tables, segment tables, base/limit registers |
| **I/O status** | List of open files, allocated I/O devices |
| **Accounting info** | CPU time used, time limits, process creation time |

**Analogy:** PCB is like a patient's medical record in a hospital — tracks everything about them so any doctor (CPU) can continue treatment (execution) at any time.

**🎤 Interview Q:** *Where is the PCB stored?*
**A:** In the **kernel's memory space** — typically in a process table (array or linked list of PCBs). Only the kernel can access it; user processes cannot.

---

## 2.3 Process States & Transitions ⭐⭐

### 5-State Model

```
                    ┌──────────────┐
       Admitted     │              │    Exit
  NEW ────────────► │    READY     │ ──────────► TERMINATED
                    │              │
                    └──────┬───────┘
                           │ Scheduler
                           │ Dispatch
                           ▼
                    ┌──────────────┐
                    │              │
                    │   RUNNING    │
                    │              │
                    └──┬───────┬───┘
                       │       │
           I/O or      │       │ Preempted
           Event wait  │       │ (time quantum expired)
                       ▼       │
                    ┌──────────┴───┐
                    │              │
                    │   WAITING    │───── I/O complete ────► READY
                    │  (Blocked)   │
                    └──────────────┘
```

### State Transitions

| Transition | From → To | Cause |
|-----------|-----------|-------|
| **Admitted** | New → Ready | OS loaded process into memory |
| **Dispatch** | Ready → Running | Scheduler selected this process |
| **Preempt** | Running → Ready | Time quantum expired / higher priority process arrived |
| **I/O Wait** | Running → Waiting | Process needs I/O or resource |
| **I/O Complete** | Waiting → Ready | I/O operation finished |
| **Exit** | Running → Terminated | Process completed or killed |

### 7-State Model (adds Suspend states)
- **Suspended Ready:** Process swapped to disk (but was ready)
- **Suspended Blocked:** Process swapped to disk (and was waiting for I/O)
- **Why suspend?** Memory is full — swap inactive processes to disk to make room

**🎤 Interview Q:** *Can a process go directly from Running to Ready?*
**A:** Yes — this happens during **preemption**: when a higher-priority process arrives or the time quantum expires in Round Robin scheduling.

**🎤 Interview Q:** *Can a process go from Waiting directly to Running?*
**A:** No — when I/O completes, the process moves to **Ready** queue first, then must be scheduled by the CPU scheduler to enter Running state.

---

## 2.4 Context Switching ⭐⭐

### What Is It?
Context switching is the process of **saving the state of the current process** and **loading the state of the next process** so the CPU can execute it.

### How It Works
```
Process A running        Context Switch          Process B running
┌──────────────┐        ┌──────────────┐        ┌──────────────┐
│ Executing    │        │ 1. Save A's  │        │ Executing    │
│ instructions │───────►│    PCB       │───────►│ instructions │
│ on CPU       │        │ 2. Load B's  │        │ on CPU       │
└──────────────┘        │    PCB       │        └──────────────┘
                        │ 3. Restore   │
                        │    registers │
                        │ 4. Resume B  │
                        └──────────────┘
```

### What Gets Saved/Restored?
- Program counter (PC)
- All CPU registers
- Stack pointer
- Memory management info (page table base)
- FPU state

### Context Switch Overhead
- Context switching is **pure overhead** — no useful work is done during the switch
- Typical time: **1-10 microseconds** on modern hardware
- Too many context switches = degraded performance
- Thread context switches are **faster** than process switches (threads share address space)

**🎤 Interview Q:** *How can we reduce context switching overhead?*
**A:** (1) Use threads instead of processes (lighter switch), (2) Increase time quantum in Round Robin (fewer switches but worse response time), (3) Use hardware support (multiple register sets), (4) Avoid unnecessary blocking (use async I/O).

---

# PHASE 3: THREADS

## 🎯 Learning Objectives
- Understand threads and multithreading
- Know user-level vs kernel-level threads
- Master threading models
- Clearly differentiate process vs thread

---

## 3.1 What is a Thread?

A thread is the **smallest unit of CPU execution** within a process. A process can have multiple threads that share the process's resources.

**Analogy:** A process is a factory. Threads are workers in that factory. They share the factory floor (memory), tools (resources), and blueprints (code), but each worker has their own task list (stack, registers, PC).

### What Threads Share (within a process)
- **Code section** (text)
- **Data section** (global variables)
- **Heap** (dynamic memory)
- **Open files and signals**

### What Each Thread Has Its Own
- **Thread ID**
- **Program counter**
- **Register set**
- **Stack** (each thread has its own stack)

```
Process
┌──────────────────────────────────────┐
│  Code  │  Data  │  Heap  │  Files   │  ← Shared by all threads
├────────┴────────┴────────┴──────────┤
│ Thread 1  │ Thread 2  │ Thread 3    │
│ ┌───────┐ │ ┌───────┐ │ ┌───────┐  │
│ │Stack 1│ │ │Stack 2│ │ │Stack 3│  │  ← Each thread has its own
│ │PC 1   │ │ │PC 2   │ │ │PC 3   │  │
│ │Regs 1 │ │ │Regs 2 │ │ │Regs 3 │  │
│ └───────┘ │ └───────┘ │ └───────┘  │
└───────────┴───────────┴─────────────┘
```

---

## 3.2 Why Multithreading?

| Benefit | Explanation |
|---------|-------------|
| **Responsiveness** | UI thread stays active while background thread does work |
| **Resource sharing** | Threads share memory — no need for IPC |
| **Economy** | Thread creation is cheaper than process creation (10-100x faster) |
| **Scalability** | Can leverage multi-core CPUs for true parallelism |

**Real-World Example:** A web browser:
- Thread 1: Renders the page
- Thread 2: Downloads images
- Thread 3: Runs JavaScript
- Thread 4: Handles user input
All share the same process memory (DOM, page data).

---

## 3.3 User-Level vs Kernel-Level Threads ⭐

| Feature | User-Level Threads (ULT) | Kernel-Level Threads (KLT) |
|---------|-------------------------|---------------------------|
| **Managed by** | User-space thread library | OS kernel |
| **OS awareness** | OS doesn't know about them | OS schedules them directly |
| **Creation speed** | Very fast (no kernel involvement) | Slower (system call needed) |
| **Context switch** | Fast (no mode switch) | Slower (mode switch required) |
| **Blocking** | If one thread blocks → entire process blocks! | Only the blocking thread is blocked |
| **Multi-core usage** | Cannot use multiple cores (OS sees 1 process) | Can run on different cores |
| **Examples** | Green threads (Java early), POSIX user threads | Linux pthreads, Windows threads |

**🎤 Interview Q:** *If user threads are faster, why don't we always use them?*
**A:** Critical flaw: if any ULT makes a blocking system call (e.g., I/O), the kernel blocks the entire process because it only sees one schedulable entity. KLTs allow the kernel to schedule threads independently, enabling other threads to continue even when one blocks.

---

## 3.4 Multithreading Models

### Many-to-One
```
User Threads:  T1  T2  T3  T4
                 \  |  |  /
                  \ | | /
Kernel Thread:      K1
```
- Many user threads → 1 kernel thread
- **Pros:** Fast thread management (user space)
- **Cons:** One block = all block; can't use multi-core
- **Example:** Old Solaris Green Threads

### One-to-One
```
User Threads:  T1   T2   T3   T4
                |    |    |    |
Kernel Threads: K1   K2   K3   K4
```
- Each user thread → 1 kernel thread
- **Pros:** True concurrency, blocking doesn't affect others
- **Cons:** Creating each thread requires kernel call; thread count limited
- **Example:** Linux (pthreads), Windows

### Many-to-Many
```
User Threads:  T1   T2   T3   T4   T5
                \   / \   |   / \
                 \ /   \  |  /   \
Kernel Threads:  K1     K2    K3
```
- Many user threads → many kernel threads (≤ user threads)
- **Pros:** Best of both — multiplexed efficiently
- **Cons:** Complex to implement
- **Example:** Solaris (prior to v9)

---

## 3.5 Process vs Thread — Master Comparison ⭐⭐⭐

| Feature | Process | Thread |
|---------|---------|--------|
| **Definition** | Program in execution | Lightweight process within a process |
| **Memory** | Own address space | Shares process address space |
| **Creation time** | Slow (heavy) | Fast (lightweight, 10-100x faster) |
| **Context switch** | Expensive (change page tables, TLB flush) | Cheap (shared address space) |
| **Communication** | IPC needed (pipes, sockets, shared memory) | Direct memory access (shared data) |
| **Isolation** | Fully isolated (one crash doesn't affect others) | Not isolated (one crash can kill all threads) |
| **Resource overhead** | High | Low |
| **Scalability** | Limited by IPC overhead | Better for parallel tasks |
| **Example** | Chrome tabs (each tab = process) | Word processor (spell check thread, UI thread) |

**🎤 Interview Q:** *Why does Chrome use processes for tabs instead of threads?*
**A:** **Isolation and security.** If one tab crashes or is compromised (malicious website), it doesn't affect other tabs. With threads, a crash in one thread can corrupt shared memory and crash all tabs. The tradeoff is higher memory usage.

---

## 3.6 System Calls: fork(), exec(), wait()

### fork() — Create a Child Process ⭐⭐⭐

```c
#include <unistd.h>
#include <stdio.h>

int main() {
    pid_t pid = fork();
    
    if (pid < 0) {
        // Fork failed
        perror("fork failed");
    } else if (pid == 0) {
        // Child process
        printf("I am child, PID: %d, Parent PID: %d\n", getpid(), getppid());
    } else {
        // Parent process
        printf("I am parent, PID: %d, Child PID: %d\n", getpid(), pid);
    }
    return 0;
}
```

**How fork() works:**
1. Creates an **exact copy** of the calling process
2. Child gets a copy of parent's memory, open files, registers
3. **Returns twice:** 0 to child, child's PID to parent
4. **Copy-on-Write (COW):** Pages aren't actually copied until one process modifies them (optimization)

### fork() Tree Problem ⭐⭐⭐

**Most asked numerical in OS interviews!**

```c
fork();
fork();
fork();
// How many processes total?
```

**Answer:** After n fork() calls = **2^n total processes**
- 3 forks → 2³ = **8 processes**

```
Visual fork tree for 3 forks:

          P (original)
         / \
     fork1   fork1
       |       |
      P         C1
     / \       / \
  fork2 fork2 fork2 fork2
    |     |     |     |
    P    C2    C1   C1C2
   / \  / \   / \   / \
  P  C3 C2 C4 C1 C5 C1C2 C6

Total: 8 processes (1 original + 7 children)
```

**Tricky fork() problem:**
```c
fork();
fork();
printf("hello\n");
// How many times is "hello" printed?
```
**Answer:** 2² = 4 times (each of 4 processes prints once)

### exec() — Replace Process Image

```c
// exec replaces the current process with a new program
execl("/bin/ls", "ls", "-la", NULL);
// Code after exec() never runs (process image replaced!)
printf("This will NOT print\n");
```

- **exec() does NOT create a new process** — it replaces the current one
- Common pattern: `fork()` + `exec()` = create child that runs a different program
- This is exactly how shell commands work!

### wait() — Wait for Child

```c
pid_t pid = fork();
if (pid > 0) {
    int status;
    wait(&status);  // Parent waits for child to finish
    printf("Child finished with status: %d\n", status);
}
```

- Parent blocks until any child process terminates
- `waitpid()` waits for a specific child
- **Zombie process:** Child terminated but parent hasn't called wait() — entry remains in process table
- **Orphan process:** Parent terminated before child — child adopted by init/systemd (PID 1)

**🎤 Interview Q:** *What is a zombie process and how do you prevent it?*
**A:** A zombie is a process that has finished execution but its entry remains in the process table because the parent hasn't collected its exit status via wait(). Prevention: (1) Parent calls wait()/waitpid(), (2) Use signal handler for SIGCHLD, (3) Double fork technique. Zombies consume a PID and process table entry — too many can exhaust the system.

**🎤 Interview Q:** *What is an orphan process?*
**A:** A process whose parent has terminated. The orphan is adopted by the init/systemd process (PID 1), which automatically calls wait() when the orphan terminates, preventing zombies.

---

## 📝 Phase 1-3 Revision Box

> **Quick Recall:**
> - OS = interface between user and hardware, manages resources
> - Multiprogramming = multiple programs in memory | Multitasking = time-sharing | Multiprocessing = multiple CPUs
> - Kernel mode = privileged, full access | User mode = restricted
> - Monolithic = everything in kernel (fast) | Micro = minimal kernel (reliable)
> - Process = program in execution | Thread = lightweight unit within process
> - PCB = process identity card stored in kernel space
> - States: New → Ready → Running → Waiting → Terminated
> - fork() returns 0 to child, child PID to parent; n forks = 2^n processes
> - Zombie = child done, parent hasn't wait()'d | Orphan = parent died first
> - Thread shares: code, data, heap, files | Thread owns: stack, PC, registers
> - One-to-One threading model used in Linux (pthreads)
