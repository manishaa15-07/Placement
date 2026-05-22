# 🖥️ PART 5: FILE SYSTEMS, DISK SCHEDULING, LINUX & ADVANCED TOPICS

---

# PHASE 9: FILE SYSTEMS

## 🎯 Learning Objectives
- Understand file concepts and allocation methods
- Know directory structures and inode architecture
- Understand free space management techniques

---

## 9.1 File Concepts

A **file** is a named collection of related data stored on secondary storage (disk).

### File Attributes
- **Name:** Human-readable identifier
- **Type:** Extension (.txt, .c, .pdf)
- **Size:** Current size in bytes
- **Location:** Pointer to file location on disk
- **Protection:** Access control (read/write/execute, owner/group/others)
- **Timestamps:** Created, modified, last accessed

### File Operations
Create, Open, Read, Write, Seek (reposition), Delete, Truncate, Close

---

## 9.2 File Allocation Methods ⭐

### 1. Contiguous Allocation

```
File A: blocks [3][4][5][6]    (start=3, length=4)
File B: blocks [8][9]          (start=8, length=2)

Disk: [_][_][_][A][A][A][A][_][B][B][_][_]
       0  1  2  3  4  5  6  7  8  9  10 11
```

| Pros | Cons |
|------|------|
| Simple — just store start + length | **External fragmentation** |
| Fast sequential AND random access | File size must be known at creation |
| Minimal seeks (blocks are adjacent) | Difficult to grow file |

### 2. Linked Allocation

```
File A: 3 → 7 → 2 → 10 → NULL

Disk: [_][_][A3][A1][_][_][_][A2][_][_][A4][_]
       0  1   2   3  4  5  6  7  8  9  10  11
       
Each block has pointer to next block.
```

| Pros | Cons |
|------|------|
| No external fragmentation | Slow random access (must traverse links) |
| File can grow easily | Pointer takes space in each block |
| No need to know file size at creation | If one pointer is corrupted → lose rest of file |

**FAT (File Allocation Table):** Improvement — store all pointers in a separate table (FAT) in memory. Used in FAT12/16/32 file systems (USB drives).

### 3. Indexed Allocation

```
File A: Index block = block 5
        Index block content: [3, 7, 2, 10]

Index block:     Data blocks:
┌─────────┐     ┌─────────┐
│ 5: [3]  │────►│ Block 3 │
│    [7]  │────►│ Block 7 │
│    [2]  │────►│ Block 2 │
│    [10] │────►│ Block 10│
└─────────┘     └─────────┘
```

| Pros | Cons |
|------|------|
| Fast random access (index lookup) | Overhead of index block |
| No external fragmentation | Index block size limits file size |
| Supports file growth | Small files waste index block space |

**Inode (Unix):** Uses a multi-level indexed structure:
```
Inode:
├── 12 direct pointers → 12 blocks
├── 1 single indirect → pointer block → 256 blocks (for 1KB block, 4B pointers)
├── 1 double indirect → pointer block → 256 pointer blocks → 65,536 blocks
└── 1 triple indirect → ... → 16,777,216 blocks
```

This allows small files to be fast (direct pointers) while supporting very large files (multi-level indirection).

---

## 9.3 Directory Structures

| Structure | Description | Use |
|-----------|-------------|-----|
| **Single-level** | All files in one directory | Simple, naming conflicts |
| **Two-level** | Each user has own directory | User isolation, no sharing |
| **Tree-structured** | Hierarchical directories | Most common (Unix, Windows) |
| **Acyclic graph** | Tree + shared files/directories (links) | Unix (hard links, soft links) |

### Hard Link vs Soft (Symbolic) Link

| Feature | Hard Link | Soft Link (Symlink) |
|---------|-----------|-------------------|
| **Points to** | Same inode (same data) | Path name of target |
| **Delete original** | Data still accessible via hard link | Soft link becomes broken (dangling) |
| **Cross filesystem** | No (must be same filesystem) | Yes |
| **Directories** | No (to prevent cycles) | Yes |
| **Inode** | Same as original | Different inode |

---

## 9.4 Free Space Management

| Method | How | Pros | Cons |
|--------|-----|------|------|
| **Bit vector (bitmap)** | 1 bit per block (0=free, 1=used) | Fast to find free blocks, simple | Large bitmap for large disks |
| **Linked list** | Free blocks linked together | No wasted space | Traversal to find free block |
| **Grouping** | First free block stores addresses of n free blocks | Fast allocation of multiple blocks | Complex |
| **Counting** | Store (start, count) pairs of contiguous free blocks | Compact for contiguous free space | Complex management |

---

# PHASE 10: DISK SCHEDULING

## 🎯 Learning Objectives
- Master all disk scheduling algorithms with numerical examples
- Calculate total head movement for each algorithm
- Know which algorithm is used in practice

---

## 10.1 Disk Structure

```
             Track
          ┌─────────┐
     ╭────┤ Sector  ├────╮
    ╱     └─────────┘     ╲
   ╱    ┌─────────────┐    ╲
  │     │   Platter    │     │
  │     │  ┌───────┐   │     │
  │     │  │Spindle│   │     │
  │     │  └───────┘   │     │
   ╲    └─────────────┘    ╱
    ╲                     ╱
     ╰────── R/W Head ──╯

Seek time: Time to move head to correct track (most expensive!)
Rotational latency: Time for sector to rotate under head
Transfer time: Time to transfer data
```

**Total access time = Seek time + Rotational latency + Transfer time**

**Goal of disk scheduling:** Minimize total **seek time** (head movement).

---

## 10.2 Disk Scheduling Algorithms

**For all examples:**
- Request queue: **98, 183, 37, 122, 14, 124, 65, 67**
- Initial head position: **53**
- Disk range: **0-199**

---

### FCFS (First-Come, First-Served)

Service requests in order of arrival.

```
53 → 98 → 183 → 37 → 122 → 14 → 124 → 65 → 67

Movement: |53-98| + |98-183| + |183-37| + |37-122| + |122-14| + |14-124| + |124-65| + |65-67|
         = 45 + 85 + 146 + 85 + 108 + 110 + 59 + 2
         = 640 cylinders
```

**Total head movement: 640**

---

### SSTF (Shortest Seek Time First)

Service the request **closest to current head position**.

```
53 → 65 → 67 → 37 → 14 → 98 → 122 → 124 → 183

Movement: 12 + 2 + 30 + 23 + 84 + 24 + 2 + 59 = 236 cylinders
```

**Total head movement: 236** ← Much better! But can cause **starvation** of far requests.

---

### SCAN (Elevator Algorithm) ⭐

Head moves in one direction, servicing all requests, then reverses.

```
Direction: toward 0 first (left)
53 → 37 → 14 → 0 → 65 → 67 → 98 → 122 → 124 → 183

Movement: 16 + 23 + 14 + 65 + 2 + 31 + 24 + 2 + 59 = 236 cylinders
```

**Total head movement: 236** (goes to 0 end even if no requests there)

**Analogy:** Like an elevator — goes all the way up, then all the way down, picking up passengers along the way.

---

### C-SCAN (Circular SCAN)

Head moves in one direction, services requests, jumps back to start (without servicing on return), then continues in same direction.

```
Direction: toward 199 (right)
53 → 65 → 67 → 98 → 122 → 124 → 183 → 199 → 0 → 14 → 37

Movement: 12 + 2 + 31 + 24 + 2 + 59 + 16 + 199 + 14 + 23 = 382
(jump 199→0 is counted as movement but no servicing on return)
```

**Provides more uniform wait time** than SCAN.

---

### LOOK

Like SCAN but head only goes as far as the **last request** in each direction (doesn't go to disk ends).

```
Direction: toward 0 first
53 → 37 → 14 → 65 → 67 → 98 → 122 → 124 → 183

Movement: 16 + 23 + 51 + 2 + 31 + 24 + 2 + 59 = 208 cylinders
```

**Total: 208** ← Better than SCAN (no wasted travel to disk ends)

---

### C-LOOK

Like C-SCAN but only goes to the last request (not disk ends).

```
Direction: toward 199 (right)
53 → 65 → 67 → 98 → 122 → 124 → 183 → 14 → 37

Movement: 12 + 2 + 31 + 24 + 2 + 59 + 169 + 23 = 322
(jump from 183 to 14)
```

---

### Disk Scheduling Comparison

| Algorithm | Head Movement (example) | Starvation? | Notes |
|-----------|------------------------|-------------|-------|
| **FCFS** | 640 | No | Simple, fair, but high seek time |
| **SSTF** | 236 | Yes | Good performance, but starvation risk |
| **SCAN** | 236 | No | Elevator algorithm, uniform |
| **C-SCAN** | 382 | No | More uniform wait time |
| **LOOK** | 208 | No | SCAN without going to disk ends |
| **C-LOOK** | 322 | No | Most practical, used in real systems |

---

# PHASE 11: SYSTEM CALLS & LINUX

## 🎯 Learning Objectives
- Know important system calls and when they're used
- Master essential Linux commands
- Understand process management in Linux

---

## 11.1 System Calls

**A system call is the programmatic interface** between user programs and the OS kernel. It's how applications request OS services.

```
Application (User Mode)
         │
         │ System Call (e.g., read())
         │
         ▼ ──── TRAP ────
Kernel (Kernel Mode)
         │
         │ Execute requested service
         │
         ▼ ──── RETURN ────
Application (User Mode)
```

### Important System Calls

| Category | System Call | Purpose |
|----------|-----------|---------|
| **Process** | `fork()` | Create child process |
| | `exec()` | Replace process image with new program |
| | `wait()` | Wait for child to terminate |
| | `exit()` | Terminate process |
| | `getpid()` | Get process ID |
| | `kill()` | Send signal to process |
| **File** | `open()` | Open a file |
| | `read()` | Read from file |
| | `write()` | Write to file |
| | `close()` | Close file |
| | `lseek()` | Reposition file pointer |
| | `stat()` | Get file metadata |
| **IPC** | `pipe()` | Create unidirectional communication channel |
| | `shmget()` | Create shared memory segment |
| | `msgget()` | Create message queue |
| **Memory** | `mmap()` | Map file into memory |
| | `brk()/sbrk()` | Change data segment size (heap) |

### pipe() — Inter-Process Communication

```c
int fd[2];    // fd[0] = read end, fd[1] = write end
pipe(fd);

if (fork() == 0) {
    // Child reads
    close(fd[1]);           // Close write end
    read(fd[0], buf, size); // Read from pipe
    close(fd[0]);
} else {
    // Parent writes
    close(fd[0]);            // Close read end
    write(fd[1], "Hello", 5);// Write to pipe
    close(fd[1]);
}
```

### Signals

| Signal | Number | Default Action | Trigger |
|--------|--------|---------------|---------|
| `SIGINT` | 2 | Terminate | Ctrl+C |
| `SIGKILL` | 9 | Terminate (cannot be caught!) | `kill -9` |
| `SIGTERM` | 15 | Terminate (graceful) | `kill` default |
| `SIGSEGV` | 11 | Core dump | Segmentation fault |
| `SIGSTOP` | 19 | Stop process | Cannot be caught |
| `SIGCHLD` | 17 | Ignore | Child terminated |
| `SIGALRM` | 14 | Terminate | Timer expired |

---

## 11.2 Essential Linux Commands

### Process Management

```bash
ps aux                  # List all processes
ps -ef                  # Full-format listing
top / htop              # Live process monitor
kill PID                # Send SIGTERM to process
kill -9 PID             # Force kill (SIGKILL)
nice -n 10 ./program    # Run with lower priority
renice -5 -p PID        # Change priority of running process
nohup ./program &       # Run process immune to hangup (background)
jobs                    # List background jobs
fg %1                   # Bring job 1 to foreground
bg %1                   # Resume job 1 in background
```

### File & Directory Management

```bash
ls -la                  # List files with details and hidden
chmod 755 file          # Change permissions (rwxr-xr-x)
chown user:group file   # Change ownership
find / -name "*.log"    # Find files by name
grep -r "pattern" dir/  # Search for pattern in files recursively
du -sh directory/       # Disk usage summary
df -h                   # Disk space of filesystems
ln -s target linkname   # Create symbolic link
ln target linkname      # Create hard link
```

### System Monitoring

```bash
free -h                 # Memory usage (human-readable)
vmstat 1                # Virtual memory statistics every 1 second
iostat                  # Disk I/O statistics
uptime                  # System uptime and load average
dmesg                   # Kernel ring buffer messages
strace ./program        # Trace system calls of a program
lsof -p PID            # List open files by process
```

### File Permissions

```
chmod notation:
Owner (u) | Group (g) | Others (o)
r(4) w(2) x(1) | r(4) w(2) x(1) | r(4) w(2) x(1)

Example: 755 = rwx r-x r-x
         644 = rw- r-- r--
```

---

# PHASE 12: ADVANCED OS TOPICS

## 12.1 Virtualization

**Concept:** Run multiple OS instances on a single physical machine using a **hypervisor** (Virtual Machine Monitor).

### Type 1 vs Type 2 Hypervisor

| Feature | Type 1 (Bare-metal) | Type 2 (Hosted) |
|---------|---------------------|-----------------|
| **Runs on** | Directly on hardware | On top of a host OS |
| **Performance** | Better (no host OS overhead) | Slower (extra OS layer) |
| **Examples** | VMware ESXi, Xen, Hyper-V | VirtualBox, VMware Workstation |
| **Use** | Data centers, cloud | Development, testing |

### Containers vs Virtual Machines ⭐

| Feature | VM | Container |
|---------|-----|-----------|
| **Isolation** | Full OS-level (own kernel) | Process-level (shared kernel) |
| **Size** | Gigabytes | Megabytes |
| **Boot time** | Minutes | Seconds |
| **Overhead** | High (full OS per VM) | Low (shares host kernel) |
| **Portability** | Less portable | Highly portable |
| **Examples** | VMware, VirtualBox | Docker, Podman |

```
Virtual Machines:              Containers:
┌────┐ ┌────┐ ┌────┐         ┌────┐ ┌────┐ ┌────┐
│App1│ │App2│ │App3│         │App1│ │App2│ │App3│
├────┤ ├────┤ ├────┤         ├────┤ ├────┤ ├────┤
│OS 1│ │OS 2│ │OS 3│         │Libs│ │Libs│ │Libs│
├────┴─┴────┴─┴────┤         ├────┴─┴────┴─┴────┤
│    Hypervisor     │         │  Container Runtime │
├───────────────────┤         ├───────────────────┤
│    Hardware       │         │    Host OS        │
└───────────────────┘         ├───────────────────┤
                              │    Hardware       │
                              └───────────────────┘
```

## 12.2 Multi-Core Challenges
- **Cache coherence:** Multiple cores have local caches — if one core updates a value, other caches may have stale data
- **Memory ordering:** Compilers and CPUs may reorder instructions — need memory barriers for correctness
- **Scalability:** Not all algorithms scale linearly with core count (Amdahl's Law)
- **Amdahl's Law:** Speedup = 1 / ((1-P) + P/N) where P = parallelizable fraction, N = cores

## 12.3 OS Security Basics
- **Access control:** Discretionary (DAC — owner sets permissions) vs Mandatory (MAC — system policy)
- **Buffer overflow:** Writing beyond buffer boundary → can overwrite return address → code execution
- **Privilege escalation:** Gaining higher privileges than intended
- **ASLR:** Address Space Layout Randomization — randomize memory layout to prevent attacks
- **DEP/NX bit:** Data Execution Prevention — mark memory regions as non-executable

---

## 📝 Phase 9-12 Revision Box

> **Quick Recall:**
> - File allocation: Contiguous (fast but fragmentation) | Linked (no fragmentation, slow random access) | Indexed (fast random, index overhead)
> - Unix inode: 12 direct + single/double/triple indirect pointers
> - Hard link: same inode | Symlink: points to path (can dangle)
> - Disk scheduling: FCFS (fair, slow) | SSTF (fast, starvation) | SCAN (elevator) | C-SCAN (uniform) | LOOK (practical)
> - System calls: fork/exec/wait/exit for processes, open/read/write/close for files, pipe for IPC
> - SIGKILL (9) cannot be caught! SIGTERM (15) can be handled gracefully
> - chmod 755 = rwxr-xr-x
> - VM: full OS isolation, heavy | Container: shared kernel, lightweight
> - Type 1 hypervisor: bare metal (ESXi) | Type 2: hosted (VirtualBox)
> - Amdahl's Law: speedup limited by serial fraction
