# 🖥️ PART 3: SYNCHRONIZATION & DEADLOCKS

---

# PHASE 5: SYNCHRONIZATION & CONCURRENCY

## 🎯 Learning Objectives
- Understand race conditions and the critical section problem
- Master mutex, semaphore, and monitors
- Solve all classical synchronization problems
- Know Peterson's solution and its limitations

---

## 5.1 The Problem: Race Conditions

A **race condition** occurs when multiple processes/threads access shared data concurrently and the final result depends on the **order of execution** (which is unpredictable).

**Example:**
```
Shared variable: counter = 5

Thread A: counter = counter + 1    Thread B: counter = counter - 1

Expected result: counter = 5 (one increment, one decrement)
But what if they interleave?

Thread A: LOAD counter (5)
Thread B: LOAD counter (5)         ← Both read 5!
Thread A: ADD 1 → 6
Thread B: SUB 1 → 4
Thread A: STORE counter (6)
Thread B: STORE counter (4)        ← Final value = 4 (WRONG!)
```

**Real-World Analogy:** Two people simultaneously updating a shared Google Doc cell — one types "100", the other types "200". Whoever saves last "wins," but both changes should have been applied.

---

## 5.2 Critical Section Problem ⭐⭐⭐

**Critical Section:** A section of code where shared resources are accessed. Only ONE process should execute in its critical section at a time.

### Structure of a Process

```
do {
    [Entry Section]      ← Request permission to enter CS
    
    [Critical Section]   ← Access shared resource (only 1 process here!)
    
    [Exit Section]       ← Release permission
    
    [Remainder Section]  ← Non-critical code
} while (true);
```

### Three Requirements for a Correct Solution ⭐

| Requirement | Definition | Analogy |
|-------------|-----------|---------|
| **Mutual Exclusion** | If process P is in CS, no other process can be in CS | Only one person in the bathroom at a time |
| **Progress** | If no process is in CS, only processes trying to enter should decide who enters — and the decision cannot be postponed indefinitely | If bathroom is free, someone waiting should be able to enter (no deadlock) |
| **Bounded Waiting** | A limit on how many times others can enter CS before a waiting process gets its turn | No process starves — everyone gets a turn eventually |

---

## 5.3 Peterson's Solution

**A software-based solution for 2 processes** — elegant but limited.

```c
// Shared variables
int turn;           // Whose turn to enter CS
bool flag[2];       // flag[i] = true means process i wants to enter

// Process i (i = 0 or 1, j = other)
flag[i] = true;     // "I want to enter"
turn = j;           // "But I'll give you a chance first" (courtesy)
while (flag[j] && turn == j)
    ;               // Wait (busy-wait) — only if j wants to enter AND it's j's turn
// CRITICAL SECTION
flag[i] = false;    // "I'm done"
// REMAINDER SECTION
```

**Why it works:**
- **Mutual exclusion:** Both can't be in CS — if both want in, `turn` can only be 0 OR 1
- **Progress:** If only one wants in, it enters immediately
- **Bounded waiting:** At most one other entry before you get in

**Limitations:**
- Only works for **2 processes**
- Requires **busy waiting** (wastes CPU)
- May not work on modern CPUs due to **instruction reordering** (needs memory barriers)
- Not used in practice — hardware solutions (test-and-set, compare-and-swap) are preferred

---

## 5.4 Mutex (Mutual Exclusion Lock) ⭐⭐

**Simplest synchronization tool** — a lock with two states: locked or unlocked.

```c
mutex_lock lock;

// Process A                    // Process B
acquire(lock);                  acquire(lock);    ← blocks if locked!
// Critical Section             // Critical Section
release(lock);                  release(lock);
```

**Key Properties:**
- **Only the owner (locker) can unlock** — unlike semaphore
- Binary: locked (1) or unlocked (0)
- If locked, requesting process **blocks** (sleep) or **spins** (busy-wait)

**Analogy:** A bathroom door lock — only the person inside can unlock it, and only one person can be inside.

---

## 5.5 Semaphore ⭐⭐⭐

**A more powerful synchronization tool** — an integer variable accessed via two atomic operations.

### Types

| Type | Initial Value | Use |
|------|--------------|-----|
| **Binary Semaphore** | 0 or 1 | Same as mutex (mutual exclusion) |
| **Counting Semaphore** | Any non-negative integer | Limit access to a resource with N instances |

### Operations (Atomic!)

```
wait(S) / P(S) / down(S):          signal(S) / V(S) / up(S):
    while (S <= 0)                      S = S + 1;
        ; // busy wait (or block)
    S = S - 1;
```

**P = Proberen (to test) | V = Verhogen (to increment)** — Dutch, invented by Dijkstra

### Semaphore for Mutual Exclusion
```c
semaphore mutex = 1;

// Process A                    // Process B
wait(mutex);    // mutex: 1→0   wait(mutex);    // blocks! (mutex=0)
// Critical Section             // waits until A signals...
signal(mutex);  // mutex: 0→1   // now enters CS
                                signal(mutex);
```

### Semaphore for Process Synchronization (Ordering)
```c
// Ensure S2 executes AFTER S1
semaphore sync = 0;

// Process A                    // Process B
S1;                             wait(sync);     // blocks (sync=0)
signal(sync);   // sync: 0→1   S2;              // runs after A signals
```

### Counting Semaphore Example
```c
// Database with max 5 connections
semaphore db_conn = 5;

// Each client:
wait(db_conn);      // decrement (5→4→3→2→1→0, then block)
// USE DATABASE
signal(db_conn);    // increment (opens slot for waiting client)
```

---

## 5.6 Mutex vs Semaphore ⭐⭐

| Feature | Mutex | Semaphore |
|---------|-------|-----------|
| **Type** | Locking mechanism | Signaling mechanism |
| **Values** | Binary (0 or 1) | Any non-negative integer |
| **Ownership** | Yes — only owner can unlock | No ownership — any process can signal |
| **Purpose** | Mutual exclusion only | Mutual exclusion + synchronization + resource counting |
| **Analogy** | Key to a bathroom (only holder can unlock) | Parking lot counter (tracks available spots) |
| **Can signal without wait?** | No (must acquire first) | Yes (V can be called without P first) |

**🎤 Interview Q:** *Can a semaphore replace a mutex?*
**A:** A binary semaphore can provide mutual exclusion like a mutex, but they're conceptually different. A mutex has ownership (only locker unlocks it), supports priority inheritance (to prevent priority inversion), and is meant strictly for mutual exclusion. A semaphore is a signaling mechanism with no ownership. In practice, use mutex for locking shared data, semaphore for signaling/ordering.

---

## 5.7 Monitors

**A high-level synchronization construct** that encapsulates shared data and operations, with built-in mutual exclusion.

```
monitor BankAccount {
    int balance;
    condition sufficientFunds;
    
    procedure deposit(amount) {
        balance += amount;
        sufficientFunds.signal();    // Wake up waiting process
    }
    
    procedure withdraw(amount) {
        while (balance < amount)
            sufficientFunds.wait();  // Wait until enough funds
        balance -= amount;
    }
}
```

**Key Idea:** Only **one process can be active** inside the monitor at a time — mutual exclusion is automatic (no explicit lock/unlock needed). Uses **condition variables** for synchronization (wait + signal).

**Analogy:** A private office where only one person can work at a time. Others wait outside. Condition variables are like "call me when X happens" notes.

**In Practice:** Java's `synchronized` keyword is a monitor implementation.

---

## 5.8 Spinlocks

**A lock where the waiting process busy-waits** (spins) in a loop checking if the lock is free.

```c
while (test_and_set(&lock))  // Spin until lock is free
    ;                         // Wasting CPU cycles!
// Critical section
lock = false;
```

**When to use:**
- Expected wait time is **very short** (shorter than context switch overhead)
- On **multi-core systems** (spinning on one core while CS runs on another)
- In **kernel code** (can't sleep while holding some kernel locks)

**When NOT to use:**
- Single-core systems (spinning wastes the only CPU!)
- Long critical sections (wasteful spinning)

---

## 5.9 Classical Synchronization Problems ⭐⭐⭐

### Problem 1: Producer-Consumer (Bounded Buffer)

**Setup:** A producer generates items and places them in a fixed-size buffer. A consumer removes items from the buffer.

**Constraints:**
- Producer must wait if buffer is **full**
- Consumer must wait if buffer is **empty**
- Buffer access must be mutually exclusive

```c
semaphore mutex = 1;     // Mutual exclusion for buffer access
semaphore empty = N;     // Count of empty slots (initially N)
semaphore full = 0;      // Count of full slots (initially 0)

// PRODUCER                      // CONSUMER
while (true) {                   while (true) {
    produce(item);                   wait(full);      // Wait for item
    wait(empty);    // Wait for space    wait(mutex);     // Lock buffer
    wait(mutex);    // Lock buffer       remove(item);
    add(item);                       signal(mutex);   // Unlock
    signal(mutex);  // Unlock            signal(empty);   // Signal space
    signal(full);   // Signal item       consume(item);
}                                }
```

**⚠️ Critical Order:** `wait(empty/full)` BEFORE `wait(mutex)` — reversing causes deadlock!

### Problem 2: Readers-Writers

**Setup:** Multiple readers can read simultaneously, but a writer needs exclusive access.

**Constraints:**
- Multiple readers can read at the same time ✅
- Only one writer at a time ✅
- If a writer is writing, no reader can read ✅

```c
// First Readers-Writers (Readers preference)
semaphore rw_mutex = 1;   // Exclusive access for writers
semaphore mutex = 1;      // Protect read_count
int read_count = 0;       // Number of active readers

// WRITER                        // READER
wait(rw_mutex);                  wait(mutex);
// Write data                    read_count++;
signal(rw_mutex);                if (read_count == 1)
                                     wait(rw_mutex);  // First reader locks out writers
                                 signal(mutex);
                                 // Read data
                                 wait(mutex);
                                 read_count--;
                                 if (read_count == 0)
                                     signal(rw_mutex); // Last reader unlocks for writers
                                 signal(mutex);
```

**Problem:** Writers can **starve** (readers keep arriving, never letting writer in).
**Second variant:** Writer preference — new readers wait if a writer is waiting.

### Problem 3: Dining Philosophers ⭐

**Setup:** 5 philosophers sit around a table with 5 forks. Each needs 2 forks (left + right) to eat.

```
         P0
     🍴      🍴
   P4          P1
   🍴          🍴
     P3      P2
         🍴
```

**Naive Solution (DEADLOCK!):**
```c
// Each philosopher:
wait(fork[i]);           // Pick up left fork
wait(fork[(i+1) % 5]);  // Pick up right fork
// EAT
signal(fork[(i+1) % 5]);
signal(fork[i]);
```
**Deadlock:** All 5 pick up left fork simultaneously → all waiting for right fork → deadlock!

**Solutions:**
1. **Allow at most 4 philosophers** to sit at once (use counting semaphore = 4)
2. **Odd/even strategy:** Odd philosophers pick up left first, even pick up right first
3. **Try-and-release:** If second fork unavailable, put down first fork and retry
4. **Use a monitor** with states (THINKING, HUNGRY, EATING)

---

# PHASE 6: DEADLOCKS

## 🎯 Learning Objectives
- Know the 4 necessary conditions for deadlock
- Understand resource allocation graphs
- Master Banker's Algorithm with numerical examples
- Know prevention, avoidance, detection, and recovery strategies

---

## 6.1 What is a Deadlock?

A set of processes where **each is waiting for a resource held by another** in the set, creating a circular dependency where none can proceed.

**Analogy:** Two cars on a narrow bridge coming from opposite directions — neither can move forward, neither will back up.

**Real-World:** Thread A holds lock X and needs lock Y. Thread B holds lock Y and needs lock X. Both wait forever.

---

## 6.2 Four Necessary Conditions (Coffman Conditions) ⭐⭐⭐

**ALL FOUR must hold simultaneously for deadlock to occur:**

| Condition | Definition | Analogy |
|-----------|-----------|---------|
| **Mutual Exclusion** | At least one resource is non-sharable (only one process can use it) | Only one key to the bathroom |
| **Hold and Wait** | Process holding resource(s) is waiting for additional resource(s) | Holding fork, waiting for knife |
| **No Preemption** | Resources can't be forcibly taken away | Can't snatch someone's fork |
| **Circular Wait** | P1 → P2 → P3 → ... → Pn → P1 (circular chain of waiting) | Musical chairs deadlock |

**🎤 Interview Q:** *If we break any ONE of these conditions, can deadlock occur?*
**A:** No — all four are **necessary**. Breaking any one prevents deadlock. This is the basis of deadlock prevention strategies.

---

## 6.3 Resource Allocation Graph (RAG)

```
Notation:
● Process: Circle (P1, P2)
■ Resource: Rectangle with dots (each dot = instance)
→ Request edge: Process → Resource (P wants R)
→ Assignment edge: Resource → Process (R assigned to P)

DEADLOCK example:
P1 ──wants──► R1 ──held-by──► P2 ──wants──► R2 ──held-by──► P1
             (Cycle exists → potential deadlock)

Rules:
- If no cycle → NO deadlock (guaranteed)
- If cycle AND single instance per resource → DEADLOCK (guaranteed)
- If cycle AND multiple instances → deadlock MAY or MAY NOT exist
```

---

## 6.4 Deadlock Handling Strategies

### Strategy 1: Deadlock Prevention (Break a Condition)

| Condition to Break | How | Drawback |
|-------------------|-----|----------|
| **Mutual Exclusion** | Make resources sharable | Not always possible (e.g., printers) |
| **Hold and Wait** | Request ALL resources at once before starting | Low resource utilization, starvation |
| **No Preemption** | If request fails, release all held resources | Works only for easily-saved resources (CPU, memory) |
| **Circular Wait** | Order all resources numerically; request only in increasing order | Requires fixed ordering, may be impractical |

### Strategy 2: Deadlock Avoidance (Banker's Algorithm) ⭐⭐⭐

**The OS checks if granting a resource request would lead to a potential deadlock. If yes → deny the request.**

**Key Concept — Safe State:**
- A state is **safe** if there exists at least one sequence of process completions where all processes can finish
- A state is **unsafe** if no such sequence exists (deadlock MAY happen)
- **Safe state → never deadlock | Unsafe state → might deadlock**

```
Safe ──► May become Unsafe ──► May lead to Deadlock
 ↑                                      |
 └──────────────────────────────────────┘
         (Avoid unsafe states!)
```

### Banker's Algorithm — Worked Example ⭐⭐⭐

**Given:**
- 5 processes (P0-P4)
- 3 resource types: A=10, B=5, C=7 (total instances)

| Process | Allocation (A,B,C) | Max (A,B,C) | Need (Max-Alloc) |
|---------|-------------------|-------------|-------------------|
| P0 | 0,1,0 | 7,5,3 | 7,4,3 |
| P1 | 2,0,0 | 3,2,2 | 1,2,2 |
| P2 | 3,0,2 | 9,0,2 | 6,0,0 |
| P3 | 2,1,1 | 2,2,2 | 0,1,1 |
| P4 | 0,0,2 | 4,3,3 | 4,3,1 |

**Available = Total - Sum(Allocation) = (10,5,7) - (7,2,5) = (3,3,2)**

**Finding Safe Sequence:**
```
Step 1: Available = (3,3,2)
        Can P1 run? Need(1,2,2) ≤ (3,3,2)? YES → Run P1
        Available = (3,3,2) + (2,0,0) = (5,3,2)

Step 2: Available = (5,3,2)
        Can P3 run? Need(0,1,1) ≤ (5,3,2)? YES → Run P3
        Available = (5,3,2) + (2,1,1) = (7,4,3)

Step 3: Available = (7,4,3)
        Can P4 run? Need(4,3,1) ≤ (7,4,3)? YES → Run P4
        Available = (7,4,3) + (0,0,2) = (7,4,5)

Step 4: Available = (7,4,5)
        Can P2 run? Need(6,0,0) ≤ (7,4,5)? YES → Run P2
        Available = (7,4,5) + (3,0,2) = (10,4,7)

Step 5: Available = (10,4,7)
        Can P0 run? Need(7,4,3) ≤ (10,4,7)? YES → Run P0

Safe sequence: P1 → P3 → P4 → P2 → P0 ✅
```

**If a new request comes, temporarily allocate it and check if the resulting state is safe. If yes → grant. If no → deny.**

### Strategy 3: Deadlock Detection

- Allow deadlocks to occur, then detect and recover
- **Detection:** Run a detection algorithm periodically (similar to Banker's but checks current state)
- **Wait-for Graph:** Simplified RAG with only processes — cycle = deadlock

### Strategy 4: Deadlock Recovery

| Method | How | Impact |
|--------|-----|--------|
| **Process termination** | Kill all deadlocked processes OR kill one at a time | Loss of work |
| **Resource preemption** | Take resources from a process and give to another | Process may need to rollback |
| **Rollback** | Save process state periodically; rollback to safe state | Overhead of checkpointing |

### Strategy 5: Ignore (Ostrich Algorithm)
- **Pretend deadlocks don't happen!**
- Used by most general-purpose OS (Linux, Windows)
- **Why?** Deadlocks are rare; prevention/avoidance is expensive. It's cheaper to just reboot if it happens.

---

## 📝 Phase 5-6 Revision Box

> **Quick Recall:**
> - Race condition = outcome depends on execution order of concurrent processes
> - Critical section needs: Mutual Exclusion + Progress + Bounded Waiting
> - Peterson's solution: 2 processes only, uses flag[] and turn
> - Mutex: binary lock with ownership | Semaphore: integer with P(wait)/V(signal)
> - Counting semaphore: controls access to N-instance resource
> - Producer-Consumer: mutex + empty + full semaphores (ORDER MATTERS!)
> - Dining Philosophers: 5 philosophers, 5 forks, circular deadlock risk
> - Deadlock needs ALL 4: Mutual Exclusion + Hold&Wait + No Preemption + Circular Wait
> - Banker's: Need = Max - Allocation | Check if safe sequence exists
> - Safe state → no deadlock guaranteed | Unsafe → deadlock possible
> - Most OSes use ostrich algorithm (ignore deadlocks)
