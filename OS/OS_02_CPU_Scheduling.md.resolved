# 🖥️ PART 2: CPU SCHEDULING

---

# PHASE 4: CPU SCHEDULING

## 🎯 Learning Objectives
- Understand scheduling criteria and what makes a good scheduler
- Master ALL scheduling algorithms with numerical examples
- Draw Gantt charts and calculate performance metrics
- Know which algorithm is used where and why

---

## 4.1 Why CPU Scheduling?

When multiple processes are ready to run, the OS must decide **which one gets the CPU**. The **scheduler** makes this decision.

**Analogy:** A doctor in an ER must decide which patient to treat first — based on urgency (priority), arrival time (FCFS), or expected treatment time (SJF).

### Scheduling Criteria

| Criterion | Goal | Definition |
|-----------|------|-----------|
| **CPU Utilization** | Maximize | % of time CPU is busy (ideal: 40-90%) |
| **Throughput** | Maximize | Number of processes completed per unit time |
| **Turnaround Time (TAT)** | Minimize | Total time from submission to completion = Completion - Arrival |
| **Waiting Time (WT)** | Minimize | Total time spent waiting in ready queue = TAT - Burst Time |
| **Response Time** | Minimize | Time from submission to first response (important for interactive systems) |

**Key Formulas:**
```
Turnaround Time (TAT) = Completion Time - Arrival Time
Waiting Time (WT)     = TAT - Burst Time
Response Time (RT)    = First CPU Time - Arrival Time
```

---

### Preemptive vs Non-Preemptive ⭐

| Feature | Non-Preemptive | Preemptive |
|---------|---------------|------------|
| **CPU release** | Process runs until it finishes or blocks | OS can forcibly take CPU away |
| **When** | Process voluntarily gives up CPU | Time quantum expires or higher priority arrives |
| **Context switches** | Fewer | More |
| **Response time** | Worse | Better |
| **Examples** | FCFS, SJF (non-preemptive) | SRTF, Round Robin, Preemptive Priority |
| **Starvation** | Possible (SJF) | Possible (Priority) |

---

## 4.2 FCFS (First-Come, First-Served)

**Simplest algorithm:** Process that arrives first gets CPU first.

**Properties:**
- Non-preemptive
- Simple FIFO queue
- No starvation
- **Problem: Convoy Effect** — short processes stuck behind long one

### Worked Example

| Process | Arrival Time | Burst Time |
|---------|-------------|------------|
| P1 | 0 | 6 |
| P2 | 1 | 4 |
| P3 | 2 | 2 |
| P4 | 3 | 3 |

**Gantt Chart:**
```
| P1          | P2      | P3  | P4    |
0             6        10    12      15
```

**Calculations:**

| Process | AT | BT | CT | TAT=CT-AT | WT=TAT-BT |
|---------|----|----|-----|-----------|-----------|
| P1 | 0 | 6 | 6 | 6 | 0 |
| P2 | 1 | 4 | 10 | 9 | 5 |
| P3 | 2 | 2 | 12 | 10 | 8 |
| P4 | 3 | 3 | 15 | 12 | 9 |

**Avg TAT = (6+9+10+12)/4 = 9.25**
**Avg WT = (0+5+8+9)/4 = 5.5**

**Convoy Effect:** If P1 had burst time 100, P2-P4 would wait extremely long despite being short — this is the convoy effect. Short processes "convoy" behind the long one.

---

## 4.3 SJF (Shortest Job First) — Non-Preemptive

**Select the process with the smallest burst time** (among those that have arrived).

**Properties:**
- Non-preemptive
- **Optimal** for minimum average waiting time (for non-preemptive)
- **Problem:** Starvation — long processes may never run
- **Problem:** Need to know burst time in advance (impossible in practice — use exponential averaging to predict)

### Worked Example (same processes)

| Process | Arrival Time | Burst Time |
|---------|-------------|------------|
| P1 | 0 | 6 |
| P2 | 1 | 4 |
| P3 | 2 | 2 |
| P4 | 3 | 3 |

**Gantt Chart:**
```
At time 0: Only P1 available → run P1
At time 6: P2(4), P3(2), P4(3) available → shortest = P3(2)
At time 8: P2(4), P4(3) available → shortest = P4(3)
At time 11: P2(4) available → run P2

| P1          | P3  | P4    | P2      |
0             6     8      11       15
```

| Process | AT | BT | CT | TAT | WT |
|---------|----|----|-----|-----|-----|
| P1 | 0 | 6 | 6 | 6 | 0 |
| P2 | 1 | 4 | 15 | 14 | 10 |
| P3 | 2 | 2 | 8 | 6 | 4 |
| P4 | 3 | 3 | 11 | 8 | 5 |

**Avg TAT = (6+14+6+8)/4 = 8.5**
**Avg WT = (0+10+4+5)/4 = 4.75** ← Better than FCFS!

---

## 4.4 SRTF (Shortest Remaining Time First) — Preemptive SJF ⭐⭐

**Preemptive version of SJF:** At every arrival, check if the new process has shorter remaining time than the current one.

### Worked Example

| Process | Arrival Time | Burst Time |
|---------|-------------|------------|
| P1 | 0 | 6 |
| P2 | 1 | 4 |
| P3 | 2 | 2 |
| P4 | 3 | 3 |

**Step-by-step:**
```
t=0: P1 arrives (remaining=6). Run P1.
t=1: P2 arrives (remaining=4). P1 remaining=5. 4 < 5 → Preempt! Run P2.
t=2: P3 arrives (remaining=2). P2 remaining=3. 2 < 3 → Preempt! Run P3.
t=3: P4 arrives (remaining=3). P3 remaining=1. 1 < 3 → Continue P3.
t=4: P3 done. Remaining: P1(5), P2(3), P4(3). Shortest = P2(3) or P4(3). Run P2 (arrived first).
t=7: P2 done. Remaining: P1(5), P4(3). Run P4(3).
t=10: P4 done. Run P1(5).
t=15: P1 done.
```

**Gantt Chart:**
```
| P1 | P2 | P3  | P2    | P4    | P1          |
0    1    2     4       7      10             15
```

| Process | AT | BT | CT | TAT | WT |
|---------|----|----|-----|-----|-----|
| P1 | 0 | 6 | 15 | 15 | 9 |
| P2 | 1 | 4 | 7 | 6 | 2 |
| P3 | 2 | 2 | 4 | 2 | 0 |
| P4 | 3 | 3 | 10 | 7 | 4 |

**Avg TAT = (15+6+2+7)/4 = 7.5**
**Avg WT = (9+2+0+4)/4 = 3.75** ← Best so far!

**SRTF gives optimal average waiting time among all scheduling algorithms.**

---

## 4.5 Priority Scheduling

**Select the process with the highest priority** (lowest number = highest priority, typically).

**Two Variants:**
- **Non-preemptive:** Current process finishes even if higher priority arrives
- **Preemptive:** Higher priority process preempts current one

**Problem:** **Starvation** — low-priority processes may never execute

**Solution:** **Aging** — gradually increase the priority of waiting processes over time

### Worked Example (Non-Preemptive, lower number = higher priority)

| Process | AT | BT | Priority |
|---------|----|----|----------|
| P1 | 0 | 4 | 3 |
| P2 | 1 | 3 | 1 |
| P3 | 2 | 2 | 4 |
| P4 | 3 | 5 | 2 |

```
t=0: Only P1 (priority 3). Run P1.
t=4: P2(1), P3(4), P4(2) available. Highest priority = P2(1). Run P2.
t=7: P3(4), P4(2). Highest = P4(2). Run P4.
t=12: P3(4). Run P3.

| P1      | P2    | P4          | P3  |
0         4      7            12    14
```

**🎤 Interview Q:** *What is starvation? How is it solved?*
**A:** Starvation occurs when a process waits indefinitely because higher-priority processes keep arriving. **Aging** solves this by incrementing the priority of waiting processes over time — eventually, even the lowest-priority process becomes high enough to execute.

---

## 4.6 Round Robin (RR) ⭐⭐⭐

**Most important for interviews!** Each process gets a fixed **time quantum (q)**. After q time units, the process is preempted and placed at the back of the ready queue.

**Properties:**
- Preemptive
- **Fair** — every process gets equal CPU share
- No starvation
- **Designed for time-sharing systems**
- Performance depends heavily on time quantum choice

### Time Quantum Choice
- **q too large** → becomes FCFS (no preemption)
- **q too small** → too many context switches (overhead dominates)
- **Rule of thumb:** 80% of bursts should complete within one quantum

### Worked Example (q = 3)

| Process | Arrival Time | Burst Time |
|---------|-------------|------------|
| P1 | 0 | 5 |
| P2 | 1 | 4 |
| P3 | 2 | 2 |
| P4 | 3 | 1 |

**Ready Queue Evolution:**
```
t=0:  Queue: [P1(5)]           → Run P1 for 3, remaining=2
t=1:  P2 arrives, added to queue
t=2:  P3 arrives, added to queue
t=3:  P4 arrives. P1 preempted. Queue: [P2(4), P3(2), P4(1), P1(2)]
      → Run P2 for 3, remaining=1
t=6:  Queue: [P3(2), P4(1), P1(2), P2(1)]
      → Run P3 for 2 (completes!)
t=8:  Queue: [P4(1), P1(2), P2(1)]
      → Run P4 for 1 (completes!)
t=9:  Queue: [P1(2), P2(1)]
      → Run P1 for 2 (completes!)
t=11: Queue: [P2(1)]
      → Run P2 for 1 (completes!)
```

**Gantt Chart:**
```
| P1    | P2    | P3  | P4| P1  | P2|
0       3       6     8   9    11  12
```

| Process | AT | BT | CT | TAT | WT | RT (Response) |
|---------|----|----|-----|-----|-----|------|
| P1 | 0 | 5 | 11 | 11 | 6 | 0 |
| P2 | 1 | 4 | 12 | 11 | 7 | 2 |
| P3 | 2 | 2 | 8 | 6 | 4 | 4 |
| P4 | 3 | 1 | 9 | 6 | 5 | 5 |

**Avg TAT = (11+11+6+6)/4 = 8.5**
**Avg WT = (6+7+4+5)/4 = 5.5**

**🎤 Interview Q:** *What time quantum is best for Round Robin?*
**A:** There's no universal best. Too small (e.g., 1ms) → excessive context switching overhead. Too large → degenerates to FCFS. A good rule: 80% of CPU bursts should be shorter than the quantum. Typical values: 10-100 milliseconds.

---

## 4.7 Multilevel Queue Scheduling

**Idea:** Separate ready queue for each category of process, each with its own scheduling algorithm.

```
┌─────────────────────────────────────────┐
│  System Processes (Highest Priority)     │ → Priority scheduling
├─────────────────────────────────────────┤
│  Interactive Processes                   │ → Round Robin (q=8)
├─────────────────────────────────────────┤
│  Batch Processes (Lowest Priority)       │ → FCFS
└─────────────────────────────────────────┘
```

**Properties:**
- Processes are **permanently** assigned to a queue (no movement)
- Higher-priority queues must be empty before lower queues get CPU
- **Problem:** Starvation of lower-priority queues
- **Fix:** Give each queue a percentage of CPU time (e.g., 80% interactive, 20% batch)

---

## 4.8 Multilevel Feedback Queue Scheduling (MLFQ) ⭐

**The most sophisticated and commonly used general-purpose scheduler.**

**Key Improvement:** Processes **can move between queues** based on behavior.

```
Queue 0 (Highest Priority): RR with q=8
    ↓ (if process uses entire quantum → demoted)
Queue 1: RR with q=16
    ↓ (if process uses entire quantum → demoted)
Queue 2 (Lowest Priority): FCFS
    ↑ (aging: if process waits too long → promoted)
```

**Rules:**
1. New processes enter the highest-priority queue
2. If a process uses its entire time quantum → demoted to lower queue
3. If a process gives up CPU before quantum (I/O-bound) → stays in same queue
4. **Aging:** Processes waiting too long in lower queues get promoted

**Why it's brilliant:**
- **CPU-bound processes** (use full quantum) → gradually drop to lower queues → get longer time slices but less frequently
- **I/O-bound processes** (short bursts, give up CPU early) → stay in high-priority queues → get fast response
- **Automatically classifies processes** without needing to know burst times!

**🎤 Interview Q:** *What scheduling algorithm does Linux use?*
**A:** Linux uses the **Completely Fair Scheduler (CFS)**, which uses a red-black tree sorted by "virtual runtime" (vruntime). Processes that have used less CPU get higher priority. It aims for fairness — each process gets an equal share of CPU proportional to its weight. It's a variant of fair-share scheduling, inspired by MLFQ concepts.

---

## 4.9 Scheduling Algorithm Comparison ⭐⭐

| Algorithm | Preemptive? | Starvation? | Optimal WT? | Key Feature |
|-----------|------------|-------------|-------------|-------------|
| **FCFS** | No | No | No | Simplest, convoy effect |
| **SJF** | No | Yes | Yes (non-preemptive) | Minimum avg WT (non-preemptive) |
| **SRTF** | Yes | Yes | Yes (overall) | Best avg WT overall |
| **Priority** | Both | Yes | No | Priority-based, needs aging |
| **RR** | Yes | No | No | Fair, good response time |
| **MLQ** | Both | Yes | No | Static queue assignment |
| **MLFQ** | Yes | No (with aging) | No | Adaptive, most practical |

---

## 📝 Phase 4 Revision Box

> **Quick Recall:**
> - TAT = CT - AT | WT = TAT - BT | RT = First CPU - AT
> - FCFS: Simple, convoy effect, no starvation
> - SJF: Optimal non-preemptive WT, starvation possible
> - SRTF: Optimal overall WT, preemptive SJF
> - Priority: Starvation → solved by aging
> - RR: Fair, time quantum matters, no starvation
> - MLFQ: Processes move between queues, most practical scheduler
> - Linux uses CFS (red-black tree, virtual runtime)
> - Preemptive = OS can take CPU away | Non-preemptive = process finishes or blocks voluntarily
