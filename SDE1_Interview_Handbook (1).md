# SDE1 Interview Preparation Handbook
> **Complete Question Bank — 200 Questions × 6 Subjects = 1200 Questions**
> Subjects: Operating Systems | DBMS | Computer Networks | OOP | Low-Level Design | High-Level Design
> Difficulty: Beginner (Q1–60) | Intermediate (Q61–130) | Advanced (Q131–200)

---

# SUBJECT 1: OPERATING SYSTEMS

## Topic Coverage
- Process Management (Process vs Program, PCB, States, Context Switching)
- CPU Scheduling (FCFS, SJF, Round Robin, Priority, Multilevel Queue)
- Synchronization (Race Conditions, Mutex, Semaphores, Monitors, Classical Problems)
- Deadlocks (Detection, Prevention, Avoidance, Recovery, Banker's Algorithm)
- Memory Management (Paging, Segmentation, Virtual Memory, TLB, Page Replacement)
- File Systems (Inodes, FAT, Directory Structure, Disk Scheduling)
- I/O Management (DMA, Interrupts, Buffering, Spooling)
- Inter-Process Communication (Pipes, Shared Memory, Message Queues, Signals)
- Threads (User vs Kernel, Multithreading Models, Thread Pools)
- Virtualization & Containers

---

## BEGINNER QUESTIONS (Q1–Q60)

---

### Q1. What is the difference between a Program and a Process?

**Topic Mapping:** OS → Process Management → Process vs Program

**Question:** Explain the fundamental difference between a program and a process. Why does this distinction matter?

**Explanation:**
A **program** is a passive entity — a set of instructions stored on disk (executable file). It has no execution state.
A **process** is an active entity — a program in execution. It has its own memory space, CPU registers, program counter, stack, and heap.

**Step-by-Step Answer:**
1. A program is static; it sits on disk. A process is dynamic; it lives in memory.
2. One program can spawn multiple processes (e.g., opening Chrome twice = two processes).
3. A process contains: code segment, data segment, stack, heap, PCB metadata.
4. OS manages processes via the Process Control Block (PCB).

**Example:**
`gcc main.c -o app` → creates a program (`app` binary on disk).
`./app` → OS creates a process from that program, allocates memory, assigns PID.

**Edge Cases / Follow-ups:**
- Can two processes share the same code segment? Yes — shared libraries (e.g., libc).
- What happens if you run the same program twice? Two separate processes with different PIDs but potentially shared read-only code pages.

**Interview Tip:** Always mention PCB, PID, and memory layout when answering. Don't just say "process is running program."

---

### Q2. What are the different states of a process?

**Topic Mapping:** OS → Process Management → Process States

**Question:** Draw and explain the process state diagram with all transitions.

**Explanation:**
Processes transition through states based on scheduling decisions and I/O events.

**States:**
```
New → Ready → Running → Terminated
                ↓           ↑
             Waiting ────────
```
- **New:** Process being created
- **Ready:** In memory, waiting for CPU
- **Running:** Currently executing on CPU
- **Waiting/Blocked:** Waiting for I/O or event
- **Terminated:** Finished execution

**Transitions:**
- New → Ready: admitted by long-term scheduler
- Ready → Running: dispatched by short-term scheduler
- Running → Ready: preempted (time quantum expired)
- Running → Waiting: I/O request issued
- Waiting → Ready: I/O completed
- Running → Terminated: process exits

**Edge Cases / Follow-ups:**
- **Zombie state:** Process finished but parent hasn't called `wait()` — PCB still exists.
- **Orphan state:** Parent died before child — child adopted by `init` (PID 1).
- Some OSes add **Suspended Ready** and **Suspended Wait** states for swapping.

**Interview Tip:** Interviewers love asking about zombie and orphan processes. Always mention them.

---

### Q3. What is a Process Control Block (PCB)?

**Topic Mapping:** OS → Process Management → PCB

**Question:** What information does a PCB store, and why is it critical to the OS?

**Explanation:**
PCB is a data structure maintained by the OS for every process. It's the "passport" of a process.

**PCB Contents:**
| Field | Description |
|-------|-------------|
| Process ID (PID) | Unique identifier |
| Process State | New/Ready/Running/Waiting/Terminated |
| Program Counter | Address of next instruction |
| CPU Registers | All register values (saved on context switch) |
| Memory Limits | Base/limit registers, page table pointer |
| Open File List | File descriptors |
| I/O Status | Pending I/O devices/requests |
| Scheduling Info | Priority, quantum remaining |
| Accounting Info | CPU time used, start time |

**Why Critical:**
Without PCB, the OS cannot perform context switching — it wouldn't know how to restore a process after it's preempted.

**Example:**
When process A is preempted, its entire CPU state (registers, PC) is saved into PCB-A. When scheduled again, OS loads PCB-A back → process resumes as if nothing happened.

**Interview Tip:** PCB is stored in kernel memory (protected). User processes cannot directly access it.

---

### Q4. What is Context Switching?

**Topic Mapping:** OS → Process Management → Context Switching

**Question:** Explain context switching. What is its overhead and how do modern OSes minimize it?

**Explanation:**
Context switching is the mechanism of saving the state of the current process and loading the state of another process so the CPU can switch between them.

**Steps:**
1. OS saves current process state into its PCB (registers, PC, stack pointer)
2. OS updates process state to Ready/Waiting
3. OS selects next process via scheduler
4. OS loads new process state from its PCB
5. CPU jumps to saved Program Counter of new process

**Overhead:**
- Pure overhead — no useful work done during the switch
- Typical cost: 1–10 microseconds
- L1/L2 cache becomes cold (cache miss penalty)

**Minimization Strategies:**
- **Hardware support:** Some CPUs have multiple register banks
- **Threads:** Lighter context switch than processes (shared address space)
- **Fewer preemptions:** Tune time quantum appropriately

**Interview Tip:** Context switch is pure overhead. Threads reduce this overhead because they share address space — only need to save/restore registers, not memory mappings.

---

### Q5. What is FCFS CPU Scheduling?

**Topic Mapping:** OS → CPU Scheduling → FCFS

**Question:** Explain First-Come-First-Served scheduling. What is the convoy effect?

**Explanation:**
FCFS is the simplest scheduling algorithm. Processes are scheduled in the order they arrive. Non-preemptive.

**Example:**
| Process | Arrival | Burst |
|---------|---------|-------|
| P1 | 0 | 10 |
| P2 | 1 | 2 |
| P3 | 2 | 3 |

Execution order: P1(0-10) → P2(10-12) → P3(12-15)
- Waiting time: P1=0, P2=9, P3=10
- Average waiting time = (0+9+10)/3 = 6.33ms

**Convoy Effect:**
Short processes stuck waiting behind a long process. Like cars stuck behind a slow truck on a one-lane road. This leads to poor CPU and I/O device utilization.

**Pros:** Simple, no starvation (every process eventually runs)
**Cons:** High average waiting time, convoy effect, poor for interactive systems

**Interview Tip:** FCFS is never used alone in real systems. Used as a tie-breaker in more complex algorithms.

---

### Q6. What is SJF Scheduling and why is it theoretically optimal?

**Topic Mapping:** OS → CPU Scheduling → SJF

**Question:** Explain Shortest Job First scheduling. Why is it optimal? What's its main limitation?

**Explanation:**
SJF schedules the process with the **smallest burst time** next. Provably minimizes average waiting time among all non-preemptive algorithms.

**Non-Preemptive SJF Example:**
| Process | Arrival | Burst |
|---------|---------|-------|
| P1 | 0 | 7 |
| P2 | 2 | 4 |
| P3 | 4 | 1 |
| P4 | 5 | 4 |

At t=0: only P1 available → run P1 (0-7)
At t=7: P2, P3, P4 available → pick P3 (burst=1) → run (7-8)
Then P2 (burst=4) → run (8-12), then P4 (12-16)

Avg wait = (0 + 6 + 3 + 7)/4 = 4ms

**Main Limitation:** Requires knowing burst time in advance — impossible in practice!

**Solution:** **Exponential Averaging** to predict next burst:
`τ(n+1) = α × t(n) + (1-α) × τ(n)`
where t(n) = actual last burst, τ(n) = last prediction, α ∈ [0,1]

**Preemptive SJF = SRTF (Shortest Remaining Time First):** Can preempt current process if new arrival has shorter remaining time.

**Starvation Risk:** Long processes may never run if short processes keep arriving.

**Interview Tip:** SJF is optimal for minimizing average waiting time — memorize this fact. But always say "it's impractical because burst time is unknown."

---

### Q7. How does Round Robin Scheduling work?

**Topic Mapping:** OS → CPU Scheduling → Round Robin

**Question:** Explain Round Robin scheduling. How do you choose the optimal time quantum?

**Explanation:**
Each process gets a fixed time slice (quantum). If not finished, it's preempted and moved to back of ready queue. Designed for time-sharing systems.

**Example (quantum = 4ms):**
| Process | Burst |
|---------|-------|
| P1 | 10 |
| P2 | 6 |
| P3 | 4 |

Timeline: P1(0-4) → P2(4-8) → P3(8-12) → P1(12-16) → P2(16-18) → P1(18-20)

**Choosing Quantum:**
- Too small → too many context switches → overhead dominates
- Too large → degenerates into FCFS
- Rule of thumb: 80% of processes should complete in one quantum
- Typical: 10–100 ms

**Response Time:** RR gives good response time for interactive processes.
**Turnaround Time:** Usually higher than SJF.

**Interview Tip:** RR is the go-to for time-sharing. Emphasize the quantum tradeoff — it's a common follow-up.

---

### Q8. What is Priority Scheduling and what is Starvation?

**Topic Mapping:** OS → CPU Scheduling → Priority Scheduling

**Question:** How does priority scheduling work? What is starvation and how do you solve it?

**Explanation:**
Each process has a priority number. CPU always picks the highest-priority process. Can be preemptive or non-preemptive.

**Problem — Starvation:**
Low-priority processes may never get CPU if high-priority processes keep arriving. A low-priority process submitted in 1967 at MIT was found still waiting in 1973!

**Solution — Aging:**
Gradually increase the priority of waiting processes over time.
Example: Every 15 minutes, increase priority of all waiting processes by 1.

**Types of Priority:**
- **Static:** Fixed at creation time
- **Dynamic:** Changes over time (aging, based on behavior)

**Internal Priority Factors:** Execution time, memory needs, I/O vs CPU bound
**External Priority Factors:** User-set, payment level, importance

**Interview Tip:** Priority + aging = prevents starvation. This is the key fix to mention.

---

### Q9. What is a Mutex?

**Topic Mapping:** OS → Synchronization → Mutex

**Question:** What is a mutex? How does it differ from a binary semaphore?

**Explanation:**
A **mutex** (mutual exclusion lock) is a synchronization primitive that ensures only one thread accesses a critical section at a time.

**How it works:**
```
lock(mutex)
  // critical section — only one thread here at a time
unlock(mutex)
```

**Mutex vs Binary Semaphore:**
| Feature | Mutex | Binary Semaphore |
|---------|-------|-----------------|
| Owner | Yes — only locker can unlock | No — any thread can signal |
| Purpose | Mutual exclusion only | Signaling + mutual exclusion |
| Priority Inversion | Handles via priority inheritance | Doesn't handle natively |
| Value | 0 or 1 | 0 or 1 |

**Priority Inversion Problem with Mutex:**
Low-priority task L holds mutex. High-priority task H waits. Medium-priority task M preempts L. H is now waiting for M (indirectly)!

**Solution:** Priority Inheritance — temporarily raise L's priority to H's when H is waiting for L's mutex.

**Interview Tip:** The key distinction — mutex has ownership. The thread that locks must be the one to unlock. Semaphore has no such constraint.

---

### Q10. What is a Semaphore?

**Topic Mapping:** OS → Synchronization → Semaphores

**Question:** Explain binary and counting semaphores with examples.

**Explanation:**
A semaphore is an integer variable with two atomic operations: **wait (P)** and **signal (V)**.

```
wait(S):   if S > 0: S-- else block thread
signal(S): if threads waiting: wake one else S++
```

**Binary Semaphore (S ∈ {0,1}):** Works like a mutex but without ownership.

**Counting Semaphore (S ≥ 0):** Controls access to a pool of N resources.

**Example — Counting Semaphore:**
N = 3 database connections available.
`S = 3`
- Thread 1 calls wait → S=2, gets connection
- Thread 2 calls wait → S=1, gets connection
- Thread 3 calls wait → S=0, gets connection
- Thread 4 calls wait → S=0, blocks (no connection available)
- Thread 1 calls signal → S=1, Thread 4 unblocks

**Use Case:** Thread pools, connection pools, bounded buffers.

**Interview Tip:** Semaphores solve both mutual exclusion AND synchronization (ordering). Mutexes only solve mutual exclusion.

---

### Q11. What is a Deadlock?

**Topic Mapping:** OS → Deadlocks → Definition

**Question:** Define deadlock. What are the four necessary conditions for deadlock?

**Explanation:**
A deadlock is a situation where a set of processes are blocked forever, each waiting for a resource held by another process in the same set.

**Four Necessary Conditions (Coffman Conditions):**
All four must hold simultaneously for deadlock to occur.

1. **Mutual Exclusion:** At least one resource must be non-sharable (only one process at a time).
2. **Hold and Wait:** A process holds at least one resource AND waits to acquire others.
3. **No Preemption:** Resources cannot be forcibly taken from a process — only voluntarily released.
4. **Circular Wait:** A circular chain of processes, each waiting for a resource held by the next.

**Example:**
- P1 holds R1, waits for R2
- P2 holds R2, waits for R1
→ Circular wait → Deadlock!

**Interview Tip:** Removing ANY ONE of the four conditions prevents deadlock. This is how deadlock prevention strategies work.

---

### Q12. What is Paging in Memory Management?

**Topic Mapping:** OS → Memory Management → Paging

**Question:** Explain paging. How does address translation work?

**Explanation:**
Paging is a memory management scheme that eliminates external fragmentation by dividing:
- **Physical memory** into fixed-size frames
- **Logical memory (process)** into same-size pages

**Address Translation:**
Logical address = (page number, offset)
Page table maps page number → frame number
Physical address = (frame number, offset)

**Example:**
- Page size = 4KB = 2^12 bytes → offset = 12 bits
- 32-bit address space → page number = 20 bits → page table has 2^20 = 1M entries
- Each entry = 4 bytes → page table size = 4MB per process!

**TLB (Translation Lookaside Buffer):**
Cache for page table entries. ~99% hit rate → most translations in 1 cycle instead of 2 memory accesses.

**Internal Fragmentation:** Last page of process may not be fully used.

**Interview Tip:** Always mention TLB when asked about paging performance. Without TLB, every memory access requires two memory accesses (page table + actual data).

---

### Q13. What is Virtual Memory?

**Topic Mapping:** OS → Memory Management → Virtual Memory

**Question:** What is virtual memory? How does demand paging work?

**Explanation:**
Virtual memory allows processes to use more memory than physically available by using disk as an extension of RAM.

**Key Idea:** Not all pages need to be in memory at once. Load pages only when needed (**demand paging**).

**Demand Paging:**
1. Process accesses a page not in RAM → **Page Fault**
2. OS traps to page fault handler
3. Find page on disk (swap space)
4. Find free frame (or evict a page)
5. Load page from disk into frame
6. Update page table
7. Restart the faulting instruction

**Page Table Entry bits:**
- Valid/Present bit: 1 = in memory, 0 = on disk
- Dirty/Modified bit: 1 = modified, must write to disk before eviction
- Referenced bit: used by replacement algorithms

**Benefits:** Run programs larger than RAM, more programs simultaneously (multiprogramming).

**Interview Tip:** Thrashing = system spends more time paging than executing. Happens when too many processes compete for too few frames.

---

### Q14. What are Page Replacement Algorithms?

**Topic Mapping:** OS → Memory Management → Page Replacement

**Question:** Compare FIFO, Optimal, and LRU page replacement algorithms.

**Explanation:**
When a page fault occurs and no free frame exists, OS must evict a page. Which one?

**FIFO (First-In-First-Out):**
Evict the page that's been in memory the longest.
- Simple, but suffers from **Belady's Anomaly**: more frames can lead to MORE page faults!

**Optimal (OPT):**
Evict the page that won't be used for the longest time in the future.
- Lowest page faults, but **impossible in practice** (need to know the future).
- Used as a benchmark.

**LRU (Least Recently Used):**
Evict the page used least recently.
- Good approximation of Optimal.
- Implementation: Counter (timestamp each access) or Stack.
- Expensive: needs hardware support or software overhead.

**Example (3 frames, reference string: 7,0,1,2,0,3,0,4,2,3):**
- FIFO: 6 page faults
- OPT: 4 page faults
- LRU: 5 page faults

**Interview Tip:** LRU is optimal in principle but expensive. Real OSes use approximations like the **Clock Algorithm (Second Chance)** which uses a reference bit.

---

### Q15. What is the Producer-Consumer Problem?

**Topic Mapping:** OS → Synchronization → Classical Problems → Producer-Consumer

**Question:** Explain the producer-consumer problem and solve it using semaphores.

**Explanation:**
A **producer** generates data and puts it in a bounded buffer. A **consumer** takes data from the buffer. Problem: Synchronize so producer doesn't add to full buffer and consumer doesn't remove from empty buffer.

**Solution using Semaphores:**
```python
# Semaphore initialization
mutex = Semaphore(1)    # mutual exclusion on buffer
empty = Semaphore(N)    # count of empty slots (initially N)
full  = Semaphore(0)    # count of full slots (initially 0)

# Producer
def producer():
    while True:
        item = produce()
        wait(empty)      # decrement empty count; block if 0
        wait(mutex)      # enter critical section
        buffer.add(item)
        signal(mutex)    # leave critical section
        signal(full)     # increment full count

# Consumer
def consumer():
    while True:
        wait(full)       # block if buffer empty
        wait(mutex)
        item = buffer.remove()
        signal(mutex)
        signal(empty)    # increment empty count
        consume(item)
```

**Key Point:** Order of `wait()` matters! If consumer calls `wait(mutex)` before `wait(full)`, it can deadlock.

**Interview Tip:** The order `wait(full/empty)` THEN `wait(mutex)` is critical. Reversing it causes deadlock.

---

### Q16. What is the Dining Philosophers Problem?

**Topic Mapping:** OS → Synchronization → Classical Problems → Dining Philosophers

**Question:** Describe the Dining Philosophers problem and solutions to prevent deadlock.

**Explanation:**
5 philosophers sit around a table. Each needs 2 forks (left + right) to eat. Only 5 forks available (one between each pair). If all pick up left fork simultaneously → deadlock.

**Naive solution (deadlock):**
```python
# Each philosopher:
pick_up(left_fork)
pick_up(right_fork)  # Deadlock! All waiting for right fork
eat()
```

**Solutions:**

1. **Asymmetric solution:** Even philosophers pick left then right. Odd philosophers pick right then left. Breaks circular wait.

2. **Allow at most 4 philosophers at the table simultaneously.** One will always be able to eat.

3. **Pick up both forks atomically (inside mutex):**
```python
wait(mutex)
pick_up(left_fork)
pick_up(right_fork)
signal(mutex)
eat()
put_down(left_fork)
put_down(right_fork)
```
Works but reduces parallelism (only one eats at a time).

**Interview Tip:** This problem illustrates all four Coffman conditions. Any solution breaks at least one condition.

---

### Q17. What is Deadlock Prevention vs Deadlock Avoidance?

**Topic Mapping:** OS → Deadlocks → Prevention vs Avoidance

**Question:** Distinguish between deadlock prevention, avoidance, detection, and recovery.

**Explanation:**

**Prevention:** Eliminate at least one of the 4 Coffman conditions before it occurs:
- Eliminate Mutual Exclusion: use shareable resources (read-only files)
- Eliminate Hold-and-Wait: process must request all resources at once
- Allow Preemption: forcibly take resources from waiting process
- Eliminate Circular Wait: impose total ordering on resource types

**Avoidance:** Allow requests dynamically but only grant if system remains in a **safe state**.
- Safe state: there exists a sequence where every process can finish.
- **Banker's Algorithm:** Before granting a resource, simulate if system stays safe.

**Detection:** Allow deadlocks to occur, then detect and recover.
- Maintain wait-for graph; check for cycles periodically.

**Recovery:**
- **Process termination:** Kill one or all deadlocked processes.
- **Resource preemption:** Forcibly take resources from a process.

**Interview Tip:** Prevention is too restrictive (low utilization). Most real systems use detection + recovery or just ignore deadlocks (ostrich algorithm) for rare cases.

---

### Q18. What is the Banker's Algorithm?

**Topic Mapping:** OS → Deadlocks → Avoidance → Banker's Algorithm

**Question:** Explain the Banker's Algorithm with an example.

**Explanation:**
Before granting a resource request, OS checks if the system will remain in a **safe state**. If not, the process waits.

**Data structures:**
- `Available[m]`: available instances of each resource type
- `Max[n][m]`: maximum demand of each process
- `Allocation[n][m]`: currently allocated to each process
- `Need[n][m]` = Max - Allocation

**Safety Algorithm:**
```
Work = Available
Finish[i] = false for all i
Repeat:
  Find i where: Finish[i]=false AND Need[i] <= Work
  If found: Work = Work + Allocation[i]; Finish[i] = true
  If no such i found: break
If all Finish[i] = true: SAFE STATE
```

**Example:**
3 processes, 3 resource types. If after potential allocation the safety algorithm finds a valid sequence, grant the request.

**Limitation:** Processes must declare maximum resource needs upfront — often not practical.

**Interview Tip:** Banker's Algorithm is theoretically important but rarely used in practice due to its overhead and impracticality.

---

### Q19. What is Thrashing?

**Topic Mapping:** OS → Memory Management → Thrashing

**Question:** What is thrashing? How does the OS detect and prevent it?

**Explanation:**
Thrashing occurs when a process doesn't have enough frames to hold its working set. It spends more time paging in/out than actually executing.

**Cause:** High degree of multiprogramming → each process gets too few frames → frequent page faults → CPU utilization drops → OS adds MORE processes (thinking CPU is idle) → makes it worse.

**Detection:** Monitor CPU utilization vs. degree of multiprogramming. If CPU utilization drops as more processes added → thrashing.

**Prevention:**

1. **Working Set Model:** Track the set of pages a process has referenced in the last Δ time units (working set). Ensure process has enough frames for its working set.

2. **Page Fault Frequency (PFF):** If page fault rate too high → give process more frames. If too low → take frames away.

3. **Reduce Degree of Multiprogramming:** Suspend some processes, free their frames for active processes.

**Interview Tip:** Thrashing = system is memory-bound. The solution is always to reduce multiprogramming or increase RAM.

---

### Q20. What are the disk scheduling algorithms?

**Topic Mapping:** OS → File Systems → Disk Scheduling

**Question:** Compare FCFS, SSTF, SCAN, and C-SCAN disk scheduling algorithms.

**Explanation:**
Disk head movement is the bottleneck. Minimize total head movement (seek time).

**FCFS:** Serve requests in arrival order. Simple, fair, but high total seek distance.

**SSTF (Shortest Seek Time First):** Serve the closest request next. Reduces seek time but can starve distant requests.

**SCAN (Elevator Algorithm):** Head moves in one direction, servicing requests, then reverses. Like an elevator.

**C-SCAN (Circular SCAN):** Like SCAN but after reaching one end, immediately jumps back to start. More uniform wait times.

**Example (head at 50, requests: 82,170,43,140,24,16,190):**
- FCFS: total movement = large
- SSTF: total movement = smaller but possible starvation
- SCAN: moves to 190, then reverses to 16 — total ~236
- C-SCAN: more predictable, better for real systems

**Interview Tip:** C-SCAN is preferred for heavy loads. SSDs don't need disk scheduling (no physical head movement).

---

### Q21. What is an Inode?

**Topic Mapping:** OS → File Systems → Inodes

**Question:** What is an inode? What information does it store?

**Explanation:**
An **inode** (index node) is a data structure in Unix/Linux filesystems that stores metadata about a file (everything except the filename and file data itself).

**Inode Contents:**
- File size
- Permissions (read/write/execute for owner/group/others)
- Owner (UID) and Group (GID)
- Timestamps: access time, modification time, change time
- Link count (number of hard links)
- Pointers to data blocks: direct, single indirect, double indirect, triple indirect

**File Access:**
- Filename → lookup in directory → get inode number → read inode → follow block pointers → read data

**Hard Link vs Soft Link:**
- Hard link: new directory entry pointing to same inode. Both point to same data.
- Soft link (symlink): separate inode, data contains path to target.

**Why filename not in inode?** Allows multiple filenames (hard links) to point to same inode.

**Interview Tip:** Deleting a file = decrement link count. File only removed from disk when link count = 0 AND no process has it open.

---

### Q22. What is Inter-Process Communication (IPC)?

**Topic Mapping:** OS → IPC

**Question:** What are the different IPC mechanisms? Compare them.

**Explanation:**
IPC allows processes to communicate and synchronize. Key mechanisms:

| Mechanism | Speed | Complexity | Persistence | Cross-machine |
|-----------|-------|------------|-------------|---------------|
| Pipe | Fast | Low | No | No |
| Named Pipe (FIFO) | Fast | Low | Yes (as file) | No |
| Shared Memory | Fastest | High (need sync) | No | No |
| Message Queue | Medium | Medium | Yes | No |
| Socket | Medium | Medium | No | Yes |
| Signal | Fast | Low | No | No |

**Shared Memory:** Fastest IPC — processes map same physical memory. Must use semaphores/mutexes for synchronization.

**Pipes:** Unidirectional, parent-child only. `fork()` creates pipe.

**Sockets:** Works across network — basis for all network communication.

**Interview Tip:** Shared memory is fastest but hardest (synchronization burden on programmer). Sockets are slowest but most flexible (cross-machine).

---

### Q23. What are User Threads vs Kernel Threads?

**Topic Mapping:** OS → Threads → User vs Kernel Threads

**Question:** Distinguish user-level and kernel-level threads. What are the multithreading models?

**Explanation:**

**User Threads:**
- Managed by user-space library (no OS involvement)
- Fast thread switch (no system call)
- If one thread blocks (I/O), entire process blocks (OS sees only one thread)
- Examples: POSIX Pthreads, Java threads (historically)

**Kernel Threads:**
- Managed by OS
- Slower (context switch involves kernel)
- If one thread blocks, OS can schedule another from same process
- Examples: Windows threads, modern Linux threads

**Multithreading Models:**

1. **Many-to-One:** Many user threads → one kernel thread. Fast but blocks entire process on I/O.

2. **One-to-One:** Each user thread → one kernel thread. True parallelism. Overhead of kernel thread creation. (Linux, Windows use this)

3. **Many-to-Many:** Many user threads → pool of kernel threads. Best of both worlds. Complex to implement.

**Interview Tip:** Modern OSes (Linux, Windows) use One-to-One. Many-to-Many is theoretically best but rarely implemented.

---

### Q24. What is the difference between Preemptive and Non-Preemptive Scheduling?

**Topic Mapping:** OS → CPU Scheduling → Preemptive vs Non-preemptive

**Question:** When can the OS preempt a running process? What are the tradeoffs?

**Explanation:**

**Non-Preemptive (Cooperative):**
Once a process gets the CPU, it runs until it voluntarily releases it (blocks on I/O, finishes, or yields).
- Pros: No complex synchronization, predictable
- Cons: One process can monopolize CPU; poor for interactive systems
- Examples: Early Windows (3.x), FCFS, non-preemptive SJF

**Preemptive:**
OS can forcibly take CPU from a running process (timer interrupt, higher priority process arrives).
- Pros: Better response time, fair, works for interactive systems
- Cons: Need synchronization (shared data can be in inconsistent state during preemption)
- Examples: Modern Linux, Windows — all use preemptive scheduling

**When Preemption Occurs:**
- Timer interrupt (time quantum expired in Round Robin)
- Higher priority process arrives (priority scheduling)
- Process moves from Running to Waiting (I/O request — technically voluntary)

**Interview Tip:** Real-time systems use preemptive scheduling to ensure deadline guarantees.

---

### Q25. What is the difference between Internal and External Fragmentation?

**Topic Mapping:** OS → Memory Management → Fragmentation

**Question:** Explain internal vs external fragmentation with examples. Which does paging solve?

**Explanation:**

**Internal Fragmentation:**
Wasted space INSIDE an allocated block. Occurs when allocated block is slightly larger than requested.
- Example: Process requests 15KB, OS allocates 16KB page → 1KB wasted inside
- Happens in: Paging, fixed-size partitioning

**External Fragmentation:**
Enough total free memory exists, but it's scattered in small, non-contiguous chunks.
- Example: 30KB total free (15 + 10 + 5KB in separate locations), but process needs 25KB contiguous → fails
- Happens in: Segmentation, dynamic partitioning

**Solutions:**
- External fragmentation → **Compaction** (defragmentation, expensive), **Paging** (eliminates by using non-contiguous allocation)
- Internal fragmentation → **Smaller page sizes** (but larger page tables)

**Paging:** Eliminates external fragmentation (any free frame can be used), but introduces internal fragmentation (last page may be partially filled).

**Interview Tip:** Paging completely eliminates external fragmentation but not internal. Segmentation eliminates internal but not external.

---

### Q26. What are the differences between a Process and a Thread?

**Topic Mapping:** OS → Process Management → Threads

**Question:** Compare processes and threads in terms of memory, creation cost, and communication.

**Explanation:**

| Feature | Process | Thread |
|---------|---------|--------|
| Memory space | Own separate address space | Shared address space within process |
| Creation cost | Expensive (clone address space) | Cheap (share existing) |
| Context switch | Slow (TLB flush, page table swap) | Fast (same address space) |
| Communication | Requires IPC (complex) | Shared memory (simple but risky) |
| Crash isolation | Crash isolated to process | Thread crash can kill entire process |
| Resources | Own file descriptors, etc. | Share file descriptors |

**What Threads Share:**
- Code segment, data segment, heap
- Open files, signals

**What Threads Have Separately:**
- Stack, program counter, registers
- Thread ID

**Interview Tip:** Threads are "lightweight processes." The tradeoff is: threads are efficient but risky (shared memory = potential race conditions). Processes are safe but expensive.

---

### Q27. What is Segmentation?

**Topic Mapping:** OS → Memory Management → Segmentation

**Question:** Explain segmentation. How does it differ from paging?

**Explanation:**
Segmentation divides process memory into logical units (segments) of **variable size**: code, data, stack, heap — each a separate segment.

**Address Structure:** (segment number, offset)
**Segment Table:** segment number → (base address, limit)

**Translation:**
1. Physical address = Base[s] + d (if d < Limit[s], else segmentation fault)

**vs Paging:**
| Feature | Paging | Segmentation |
|---------|--------|--------------|
| Division | Fixed-size pages | Variable-size segments |
| Internal fragmentation | Yes | No |
| External fragmentation | No | Yes |
| User visible | No (transparent) | Yes (programmer sees segments) |
| Sharing | Hard (page granularity) | Easy (share entire segment) |

**Segmentation Fault:** Accessing beyond segment limit → hardware raises exception → OS delivers SIGSEGV.

**Modern OS:** Use paging, not segmentation. x86 originally used both (segmented paging) but 64-bit mode mostly ignores segmentation.

**Interview Tip:** Segfault = segmentation fault = accessing memory outside your segment bounds (or accessing null pointer, which maps to address 0 which is never mapped).

---

### Q28. What is the difference between Multiprogramming, Multitasking, Multiprocessing, and Multithreading?

**Topic Mapping:** OS → Process Management → Execution Models

**Question:** Differentiate multiprogramming, multitasking, multiprocessing, and multithreading.

**Explanation:**

**Multiprogramming:** Multiple programs loaded in memory simultaneously. When one blocks (I/O), CPU switches to another. Goal: maximize CPU utilization. Single CPU.

**Multitasking (Time-sharing):** Extension of multiprogramming. CPU switches rapidly between processes (time slicing) giving illusion of simultaneous execution. Multiple users can interact with system. Single CPU.

**Multiprocessing:** System has multiple CPUs. True parallel execution. Can run multiple processes simultaneously.

**Multithreading:** Single process with multiple threads. True parallelism on multi-core systems (kernel threads).

**Interview Tip:**
- Multiprogramming = one program at a time per CPU, maximize utilization
- Multitasking = rapid switching, gives interactive feel
- Multiprocessing = multiple CPUs, true parallelism
- Multithreading = within a process, multiple flows

---

### Q29. What is Spooling?

**Topic Mapping:** OS → I/O Management → Spooling

**Question:** What is spooling? How does it improve system efficiency?

**Explanation:**
**SPOOL** = Simultaneous Peripheral Operations OnLine.

Spooling uses disk as a buffer between a slow output device (printer) and multiple processes that want to use it.

**How it works:**
1. Process sends print job → OS writes to spool file on disk (fast)
2. Process continues immediately (doesn't wait for printer)
3. Print daemon reads from spool queue, sends to printer at printer speed

**Benefits:**
- Process isn't blocked waiting for slow printer
- Multiple processes can "print" simultaneously (actually writing to disk)
- Jobs executed in order from queue

**Similar concept: Buffering**
Buffering: temporary storage during data transfer between two processes/devices of different speeds.
- Single buffer: producer fills, consumer empties alternately
- Double buffer: producer fills buffer A while consumer empties buffer B, then swap

**Interview Tip:** Printer spooler is the classic example. Job schedulers in batch systems are another form of spooling.

---

### Q30. What is DMA (Direct Memory Access)?

**Topic Mapping:** OS → I/O Management → DMA

**Question:** What is DMA and how does it improve I/O performance?

**Explanation:**
Without DMA: CPU reads each byte from I/O device and writes to memory — wastes CPU cycles on data transfer.

With **DMA:** CPU sets up DMA controller (source, destination, count), then goes back to other work. DMA controller independently transfers data between device and memory. When done, DMA interrupts CPU.

**Benefit:** CPU freed from mechanical data transfer work. Can execute other processes while DMA handles bulk I/O.

**DMA Modes:**
- **Burst mode:** DMA takes full control of bus until transfer complete
- **Cycle stealing:** DMA steals one bus cycle per byte, interleaves with CPU
- **Transparent mode:** DMA only transfers when CPU doesn't need bus

**Used in:** Disk controllers, network cards, USB controllers, GPU transfers.

**Interview Tip:** DMA is how modern I/O achieves high throughput. Without it, GPU rendering would completely stall the CPU.

---

## Q31–Q60 (Beginner — continued)

---

### Q31. What is the difference between a Monolithic Kernel and a Microkernel?

**Topic Mapping:** OS → Kernel Architecture

**Question:** Compare monolithic and microkernel architectures. Give examples.

**Explanation:**

**Monolithic Kernel:** All OS services (memory management, file system, device drivers, IPC, scheduling) run in kernel space. Single large binary.
- Pros: High performance (no inter-process communication between services)
- Cons: A bug in any module can crash entire system; large, complex codebase
- Examples: Linux, Unix, early Windows

**Microkernel:** Only essential services in kernel (IPC, basic scheduling, memory mapping). Everything else (file system, drivers, networking) runs as user-space servers.
- Pros: More stable (driver crash doesn't crash kernel), easier to extend
- Cons: Performance overhead (IPC between user-space services)
- Examples: Mach, QNX, MINIX, seL4

**Hybrid Kernel:** Combines both. Core in kernel, some services in user space.
- Examples: Windows NT, macOS (XNU = Mach + BSD components)

**Interview Tip:** Linux is technically monolithic but modular (loadable kernel modules). Many candidates confuse modular with microkernel.

---

### Q32. What is a System Call?

**Topic Mapping:** OS → System Calls

**Question:** What is a system call? Explain the steps involved when a process makes one.

**Explanation:**
A system call is the interface between user programs and the OS kernel. It lets user processes request privileged operations (file I/O, memory allocation, process creation).

**Steps in a System Call:**
1. User program calls library function (e.g., `read()` in C)
2. Library sets up system call number in register, executes `syscall` instruction (or INT 0x80)
3. CPU switches from user mode to kernel mode (ring 0)
4. OS identifies system call number, dispatches to handler
5. Handler executes the operation
6. Result placed in register, CPU switches back to user mode
7. Library returns result to calling program

**Mode Switch Cost:** ~100ns — not trivial. That's why batching I/O, using mmap, etc. reduces system call frequency.

**Categories:**
- Process control: fork, exec, exit, wait
- File management: open, read, write, close
- Device management: ioctl
- Information: getpid, time
- Communication: pipe, socket

**Interview Tip:** User mode → kernel mode transition = context switch cost without process switch. This is why system calls are expensive.

---

### Q33. What is the `fork()` system call?

**Topic Mapping:** OS → Process Management → fork()

**Question:** Explain fork(). What does it return? What's Copy-on-Write?

**Explanation:**
`fork()` creates a new process (child) that is an exact copy of the parent.

**Return values:**
- In parent: returns child's PID
- In child: returns 0
- On failure: returns -1

```c
pid_t pid = fork();
if (pid == 0) {
    // child process
    printf("Child: PID=%d\n", getpid());
} else if (pid > 0) {
    // parent process
    printf("Parent: child PID=%d\n", pid);
} else {
    // fork failed
    perror("fork");
}
```

**Copy-on-Write (COW):**
Fork doesn't immediately copy parent's memory pages. Child shares parent's pages (read-only). When either process modifies a page → OS creates a copy of that page for the modifying process.

**Benefit:** `fork()` + immediate `exec()` (common pattern) is cheap — no pages actually copied.

**After fork():** Child inherits open file descriptors, environment, code, data. Has its own PID, separate address space (via COW).

**Interview Tip:** COW makes fork() fast. `exec()` after fork() replaces child's address space entirely — used to run a different program.

---

### Q34. What is the `exec()` system call?

**Topic Mapping:** OS → Process Management → exec()

**Question:** What does exec() do? How is it different from fork()?

**Explanation:**
`exec()` replaces the current process's memory image with a new program. It does NOT create a new process.

**fork() + exec() pattern:**
```c
pid_t pid = fork();
if (pid == 0) {
    // child
    execl("/bin/ls", "ls", "-la", NULL);
    // If exec succeeds, this line is never reached
    perror("exec");
    exit(1);
}
// parent continues here
```

**What exec() replaces:**
- Code segment
- Data segment
- Stack
- Heap

**What exec() preserves:**
- PID (same process)
- Open file descriptors (unless FD_CLOEXEC set)
- Environment (can be modified)

**exec() family:** execl, execv, execle, execve, execlp, execvp

**Interview Tip:** Shell uses fork+exec for every command: `ls -la` → shell forks → child execs `/bin/ls`.

---

### Q35. What is a Thread Pool?

**Topic Mapping:** OS → Threads → Thread Pool

**Question:** What is a thread pool? Why is it preferred over creating new threads per request?

**Explanation:**
A thread pool is a pre-created set of worker threads waiting for tasks. Instead of creating/destroying a thread per request, tasks are queued and executed by existing pool threads.

**Problem with thread-per-request:**
- Thread creation: ~1ms + memory (default stack = 1MB)
- High request load → thousands of threads → memory exhaustion
- Context switch overhead with too many threads

**Thread Pool Design:**
```
[Task Queue] → [Worker Thread 1]
             → [Worker Thread 2]
             → [Worker Thread N]
```

**Components:**
- Fixed-size pool of N threads
- Task queue (bounded or unbounded)
- Threads block on empty queue, wake when task arrives

**Benefits:**
- Reuse threads (avoid creation overhead)
- Control max concurrency
- Prevent resource exhaustion

**Real-world:** Java `ExecutorService`, Python `ThreadPoolExecutor`, server frameworks (Tomcat, Nginx worker processes)

**Interview Tip:** Thread pool size = number of CPU cores for CPU-bound tasks. For I/O-bound: more threads (they spend time waiting).

---

### Q36. What is a Race Condition?

**Topic Mapping:** OS → Synchronization → Race Conditions

**Question:** What is a race condition? Give a concrete example and explain how to fix it.

**Explanation:**
A race condition occurs when the outcome depends on the relative order of execution of multiple threads — behavior is non-deterministic.

**Example:**
```python
# Shared variable
balance = 1000

# Thread 1 (deposit 500)         # Thread 2 (withdraw 200)
temp1 = balance        # 1000    temp2 = balance  # 1000
temp1 = temp1 + 500    # 1500    temp2 = temp2 - 200  # 800
balance = temp1        # 1500    balance = temp2  # 800  ← OVERWRITES!
```
Final balance = 800, not 1300! The interleaving caused data corruption.

**Critical Section:** The code that accesses shared resources and must not be executed concurrently.

**Requirements for Critical Section solution:**
1. **Mutual Exclusion:** Only one process in critical section at a time
2. **Progress:** If no process in CS, one of the waiting ones must eventually enter
3. **Bounded Waiting:** Each process must eventually get in (no starvation)

**Fix:** Use mutex/lock around balance modification.

**Interview Tip:** Race conditions are hard to debug because they're timing-dependent and may not reproduce consistently.

---

### Q37. What is the Critical Section Problem?

**Topic Mapping:** OS → Synchronization → Critical Section

**Question:** State the critical section problem requirements. Explain Peterson's Solution.

**Explanation:**
**Peterson's Solution** (for 2 processes):
```python
# Shared
flag = [False, False]  # flag[i] = process i wants to enter
turn = 0               # whose turn it is

# Process i (j = 1-i)
flag[i] = True
turn = j  # give the other process a chance
while flag[j] and turn == j:
    pass  # busy wait
# Critical Section
flag[i] = False
# Remainder Section
```

**Why it works:**
- If both want to enter: `turn` is either 0 or 1 → one of them proceeds
- If only one wants: it enters immediately
- Satisfies all three requirements: Mutual Exclusion, Progress, Bounded Waiting

**Limitation:** Busy waiting (wastes CPU). Only for 2 processes. May not work on modern CPUs with out-of-order execution (needs memory barriers).

**Modern Solution:** Hardware atomic operations (test-and-set, compare-and-swap) + OS synchronization primitives.

**Interview Tip:** Peterson's is important theoretically. Real systems use hardware primitives + OS locks.

---

### Q38. What is Thrashing vs Working Set?

**Topic Mapping:** OS → Memory Management → Working Set

**Question:** Define the working set model. How does it relate to thrashing?

**Explanation:**
**Working Set:** The set of pages a process has referenced in the last Δ page references (the working set window).

`WS(t, Δ) = {pages referenced in time interval (t-Δ, t]}`

**Key insight:** Programs exhibit **locality of reference** — at any time, they access a small set of pages repeatedly (temporal and spatial locality).

**Working Set Model:**
- Track working set of each process
- Total demand: D = Σ |WS(i)|
- If D > total frames available → thrashing imminent
- OS should reduce multiprogramming (suspend some processes)

**Working Set vs Page Fault Rate:**
- If WS fits in allocated frames → low page fault rate
- If WS doesn't fit → constant page faults → thrashing

**Approximation:** Exact WS tracking expensive. Use reference bit + interval timer to approximate.

**Interview Tip:** Working set model explains WHY thrashing happens — process doesn't have enough frames for its working set. Solution = give it more frames or suspend it.

---

### Q39. What is a Spin Lock?

**Topic Mapping:** OS → Synchronization → Spin Lock

**Question:** What is a spin lock? When should you use it vs a blocking mutex?

**Explanation:**
A **spin lock** is a lock where a waiting thread loops (spins) continuously checking if the lock is available, rather than sleeping.

```c
while (test_and_set(&lock)) {
    ; // spin — busy wait
}
// critical section
lock = 0;
```

**Pros:**
- No context switch overhead (fast for short waits)
- Works in interrupt handlers (can't sleep in interrupt context)

**Cons:**
- Wastes CPU cycles while spinning
- If lock held for long time, spinning is very wasteful

**When to Use:**
- Critical section is very short (< context switch time)
- Multi-core system (another core can actually release the lock while you spin)
- Interrupt handlers or kernel code that can't sleep

**When NOT to Use:**
- Single-core (spinning is pointless — holder can't run while you spin)
- Long critical sections

**Interview Tip:** Spin locks are common in OS kernel code. User-space code should almost always use blocking mutexes (futex in Linux).

---

### Q40. What is Belady's Anomaly?

**Topic Mapping:** OS → Memory Management → Page Replacement → Belady's Anomaly

**Question:** What is Belady's Anomaly? Which algorithm suffers from it?

**Explanation:**
Belady's Anomaly: For **FIFO** page replacement, increasing the number of frames can sometimes INCREASE the number of page faults.

**Example:**
Reference string: 1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5

With 3 frames: 9 page faults
With 4 frames: 10 page faults!

This is counterintuitive — more memory should mean fewer page faults.

**Why FIFO suffers from it:** FIFO evicts the oldest page, which may be heavily used. Adding a frame changes which page is "oldest" in a way that can hurt.

**Which algorithms DON'T suffer:**
- **OPT (Optimal):** Provably never suffers
- **LRU:** Also doesn't suffer (can be proven)
- **Stack algorithms:** Any algorithm where the set of pages in memory with n frames is always a subset of the set with n+1 frames

**Interview Tip:** Belady's Anomaly is a theoretical insight — real OSes avoid FIFO because of it. LRU is immune.

---

### Q41. What is a Monitor?

**Topic Mapping:** OS → Synchronization → Monitors

**Question:** What is a monitor? How does it differ from semaphores?

**Explanation:**
A **monitor** is a high-level synchronization construct — a module with:
- Shared data (encapsulated)
- Procedures (methods) — only one can execute at a time
- Condition variables (for waiting/signaling)

```python
monitor BoundedBuffer:
    buffer[N]
    count = 0
    not_full = Condition()   # wait when buffer full
    not_empty = Condition()  # wait when buffer empty

    procedure insert(item):
        if count == N: not_full.wait()  # releases monitor lock, sleeps
        buffer.add(item)
        count++
        not_empty.signal()  # wake one waiting consumer

    procedure remove():
        if count == 0: not_empty.wait()
        item = buffer.remove()
        count--
        not_full.signal()
        return item
```

**Monitor vs Semaphore:**
- Semaphores: low-level, easy to misuse (forget wait/signal → bugs)
- Monitors: high-level, automatic mutual exclusion, easier to use correctly
- Java `synchronized` blocks implement a form of monitor

**Interview Tip:** Java's `synchronized` + `wait()`/`notify()` is a monitor implementation. Python's `threading.Condition` is another.

---

### Q42. What is the Readers-Writers Problem?

**Topic Mapping:** OS → Synchronization → Classical Problems → Readers-Writers

**Question:** Explain the Readers-Writers problem and its two variants.

**Explanation:**
Multiple readers can read shared data simultaneously, but writers need exclusive access.

**Variant 1 — First Readers-Writers (Reader Priority):**
No reader waits unless a writer is writing. Problem: writers can starve.

```python
mutex = Semaphore(1)      # protects read_count
wrt = Semaphore(1)        # exclusive write access
read_count = 0

Reader:
    wait(mutex)
    read_count++
    if read_count == 1: wait(wrt)  # first reader locks out writers
    signal(mutex)
    # READ
    wait(mutex)
    read_count--
    if read_count == 0: signal(wrt)  # last reader releases
    signal(mutex)

Writer:
    wait(wrt)
    # WRITE
    signal(wrt)
```

**Variant 2 — Second Readers-Writers (Writer Priority):**
Once writer ready, no new readers start. Problem: readers can starve.

**Variant 3 — Fair (No Starvation):**
Use additional semaphore for fair queuing.

**Real-world:** `pthread_rwlock_t`, Java `ReadWriteLock`, database shared/exclusive locks.

**Interview Tip:** Real databases use this: multiple concurrent reads, but writes need exclusive access. Mention `SHARED LOCK` vs `EXCLUSIVE LOCK`.

---

### Q43. What is Segmentation Fault?

**Topic Mapping:** OS → Memory Management → Segmentation Fault

**Question:** What causes a segmentation fault? How does the OS handle it?

**Explanation:**
A segmentation fault (SIGSEGV) occurs when a program accesses memory it's not allowed to access.

**Common Causes:**
1. **Null pointer dereference:** `int *p = NULL; *p = 5;`
2. **Array out of bounds:** `arr[100]` when arr has 10 elements
3. **Stack overflow:** Infinite recursion
4. **Use after free:** Accessing freed heap memory
5. **Writing to read-only memory:** Modifying string literal

**OS Mechanism:**
1. Program accesses invalid address
2. MMU raises hardware exception (page fault or protection fault)
3. OS catches exception
4. OS checks: is this address in process's valid virtual address space?
5. If NO → send SIGSEGV signal to process
6. Default SIGSEGV handler → terminate process, dump core

**Stack Overflow:** Each function call uses stack space. Infinite recursion exhausts stack → hits guard page → segfault.

**Interview Tip:** C/C++ has no bounds checking — segfaults common. Languages like Java/Python raise exceptions instead (ArrayIndexOutOfBoundsException etc.).

---

### Q44. What is the difference between Long-Term, Short-Term, and Medium-Term Schedulers?

**Topic Mapping:** OS → CPU Scheduling → Schedulers

**Question:** Distinguish between the three types of schedulers in an OS.

**Explanation:**

**Long-Term Scheduler (Job Scheduler):**
- Controls degree of multiprogramming (how many processes in memory)
- Selects from job pool (disk) which jobs to admit to ready queue
- Runs infrequently (seconds to minutes)
- Key decision: balance CPU-bound vs I/O-bound processes

**Short-Term Scheduler (CPU Scheduler):**
- Decides which ready process gets the CPU next
- Runs very frequently (milliseconds — after every timer interrupt)
- Must be extremely fast
- Algorithms: FCFS, SJF, Round Robin, Priority

**Medium-Term Scheduler (Swapper):**
- Decides which process to swap out to disk (free memory)
- Helps reduce degree of multiprogramming when memory tight
- Swapping: move process from memory to disk
- Swap back in when process ready to run and memory available

**Interview Tip:** Modern OSes often lack a distinct long-term scheduler — processes are admitted immediately. The medium-term scheduler is the swapper, handling memory pressure.

---

### Q45. What is Aging in Scheduling?

**Topic Mapping:** OS → CPU Scheduling → Aging

**Question:** How does aging prevent starvation in priority scheduling?

**Explanation:**
Aging is a technique where the priority of a process gradually increases the longer it waits in the ready queue.

**Implementation:**
- Every 15 minutes (or N scheduler runs), increase priority of all waiting processes by 1
- Eventually, even the lowest-priority process will have high enough priority to run

**Example:**
- Process P (initial priority = 1) arrives in system
- High-priority processes (priority 100) keep arriving
- After 1 hour: P's priority has increased to ~5
- After a few hours: P's priority high enough to run

**Combined Algorithm:**
- SJF + Aging: prevents starvation of long processes
- Priority + Aging: prevents starvation of low-priority processes

**Real OS:** Linux's Completely Fair Scheduler (CFS) handles this differently — uses virtual runtime. Processes that have run less get higher priority naturally.

**Interview Tip:** Aging is a general anti-starvation technique. Anytime you have a priority-based system with dynamic workloads, aging is needed.

---

### Q46. What is the difference between Paging and Segmentation?

**Topic Mapping:** OS → Memory Management → Paging vs Segmentation

**Question:** When would you use segmentation over paging?

**Explanation:**
Already covered basic differences in Q27. Here, focus on use cases:

**Use Segmentation When:**
- Logical division of program matters (code/stack/heap separate protection)
- Sharing code segment between processes (one code segment, multiple data segments)
- Growing data structures (heap segment grows, stack segment grows separately)

**Use Paging When:**
- Simple memory management needed
- Eliminate external fragmentation is priority
- Physical memory management is paramount

**Modern Reality: Segmented Paging**
x86 used both historically. 64-bit Linux/Windows: segmentation mostly disabled (flat model), paging used exclusively.

**Protected Memory:**
Segmentation's strength: different segments can have different permissions (code = read+execute, data = read+write, stack = read+write).
Modern paging achieves same via page table protection bits.

**Interview Tip:** Know that modern OSes use paging primarily. Segmentation is historical/legacy in x86-64.

---

### Q47. What is a TLB (Translation Lookaside Buffer)?

**Topic Mapping:** OS → Memory Management → TLB

**Question:** Explain TLB. What is TLB shootdown?

**Explanation:**
TLB is a hardware cache inside the CPU's MMU (Memory Management Unit) that stores recent virtual→physical address translations.

**Why needed:** Without TLB, every memory access = 2 memory accesses (page table + data). With TLB hit (~99%), translation takes 1 CPU cycle.

**TLB Miss:** Translation not cached → walk page table → update TLB → retry.

**TLB Structure:**
- Fully associative cache (any entry can map to any slot)
- 64–1024 entries typically
- Entry: (virtual page number, physical frame number, protection bits, valid bit)

**TLB and Context Switch:**
On process switch, TLB entries become invalid (different address space). Must flush TLB → cold TLB = many misses initially.

**ASID (Address Space ID):**
TLB entries tagged with ASID → no flush needed on context switch! OS maintains ASID per process. (ARM and MIPS architectures)

**TLB Shootdown (Multi-core):**
When one CPU modifies a page table entry, it must invalidate corresponding TLB entry on ALL other CPUs. Done via IPI (Inter-Processor Interrupt). Expensive.

**Interview Tip:** TLB shootdown is a major source of overhead in multi-core systems with heavy memory remapping.

---

### Q48. What is Swapping?

**Topic Mapping:** OS → Memory Management → Swapping

**Question:** Explain swapping. How does it differ from paging?

**Explanation:**
**Swapping:** Moving an entire process from memory to disk (swap space) to free memory for other processes. Later, swap it back in to continue execution.

- Unit: entire process
- Granularity: coarse (megabytes to gigabytes)
- Managed by medium-term scheduler
- When to swap out: low memory, process blocked for long time

**vs Virtual Memory/Paging:**
- Paging swaps individual pages (4KB units), not entire processes
- Virtual memory paging is transparent to process
- Swapping is coarser and more disruptive

**Modern Swap:**
Modern OSes use swap space (swap partition or swap file) as backing store for virtual memory pages. When memory pressure high, pages are written to swap and freed.

Linux: `swapon /dev/sda2` activates swap partition.
`/proc/swaps` shows active swap.

**Swap Death:** System with insufficient RAM + heavy load → continuously swapping → extremely slow (disk I/O for every memory access).

**Interview Tip:** Modern "swapping" usually means page-level swapping (paging to disk), not whole-process swapping. Linux OOM killer kills processes before full swap death.

---

### Q49. What is the Difference Between Absolute and Relative Path?

**Topic Mapping:** OS → File Systems → Paths

**Question:** Explain absolute vs relative paths. What is the current working directory?

**Explanation:**
**Absolute Path:** Full path from root directory.
- Linux/Mac: `/home/user/documents/file.txt`
- Windows: `C:\Users\user\Documents\file.txt`
- Always the same regardless of current directory

**Relative Path:** Path relative to current working directory (CWD).
- `documents/file.txt` (if CWD = `/home/user`)
- `../other/file.txt` (go up one level)
- `./script.sh` (in current directory)

**Special Directories:**
- `.` = current directory
- `..` = parent directory
- `~` = home directory (shell expansion, not OS)

**Current Working Directory:**
Every process has a CWD. Relative paths resolved from it. `chdir()` system call changes CWD. `getcwd()` returns it.

**Path Resolution:**
`open("docs/file.txt")` → OS prepends CWD → resolves symlinks → finds inode.

**Interview Tip:** When asked about file system operations, always clarify absolute vs relative. Bugs often come from incorrect assumptions about CWD.

---

### Q50. What are Hard Links and Soft Links?

**Topic Mapping:** OS → File Systems → Links

**Question:** Explain hard links and symbolic links. What happens when source file is deleted?

**Explanation:**

**Hard Link:**
- Creates new directory entry pointing to same inode as original file
- Both links share same inode, same data blocks
- Link count incremented in inode
- Deleting original just decrements link count; data removed only when count=0
- Cannot span filesystems (inode numbers local to filesystem)
- Cannot link to directories (prevents circular loops)

```bash
ln file.txt hardlink.txt    # creates hard link
ls -li  # both show same inode number
```

**Symbolic Link (Soft Link):**
- New inode created with data containing path to target
- Essentially a pointer/shortcut
- Deleting original → symlink becomes dangling (broken)
- Can span filesystems, can link directories
- Slightly slower (extra I/O to read symlink target)

```bash
ln -s /path/to/file.txt symlink.txt
ls -la  # shows symlink.txt -> /path/to/file.txt
```

**Interview Tip:** Package managers heavily use symlinks (e.g., `/usr/bin/python -> /usr/bin/python3.11`). Hard links used for space-efficient backups.

---

### Q51. What is a File Descriptor?

**Topic Mapping:** OS → File Systems → File Descriptors

**Question:** What is a file descriptor? What are the standard file descriptors?

**Explanation:**
A file descriptor (FD) is a non-negative integer that represents an open file (or socket, pipe, etc.) in a process. It's an index into the process's open file table.

**Standard FDs:**
- 0: stdin (standard input)
- 1: stdout (standard output)
- 2: stderr (standard error)

**File Descriptor Table:**
Each process has an FD table (in PCB). Each entry points to a system-wide open file table entry, which points to the inode.

**Inheritance:** Child processes inherit parent's open FDs after `fork()`. This is how shell redirection and pipes work.

**Redirection:**
```bash
ls > output.txt  # shell: open output.txt as FD 1 (dup2(fd, 1))
```

**Socket as FD:** Everything in Unix is a file. Network sockets are FDs too → `read()`/`write()` on socket FD.

**RLIMIT_NOFILE:** Maximum open FDs per process. Default ~1024. Can cause "too many open files" error if not managed.

**Interview Tip:** FD leaks (opening files, not closing them) are a common bug. Always close FDs when done.

---

### Q52. What is the difference between Preemptive and Non-preemptive Kernels?

**Topic Mapping:** OS → Kernel → Preemptibility

**Question:** Can the kernel itself be preempted? What is a preemptible kernel?

**Explanation:**
**Non-preemptible Kernel (traditional Unix):**
A process running in kernel mode (executing a system call) cannot be preempted. Must complete or voluntarily yield. Simple but poor latency for real-time tasks.

**Preemptible Kernel (Linux 2.6+):**
A process in kernel mode CAN be preempted by a higher-priority process (if kernel code is in a preemptible section). Reduces latency for real-time tasks.

**Why tricky?** Kernel code accesses shared data structures. Preemption can cause races in kernel itself. Protected with spin locks, disabling preemption in critical sections.

**Implications:**
- Non-preemptible: simpler, no race conditions in kernel, but high latency
- Preemptible: complex, race conditions possible (need careful locking), but good latency

**Real-time Linux (PREEMPT_RT):** Patches Linux kernel to make almost everything preemptible. Used in robotics, audio production, industrial control.

**Interview Tip:** Linux kernel is preemptible by default since 2.6. The PREEMPT_RT patch makes it fully preemptible for hard real-time applications.

---

### Q53. What is Copy-on-Write (COW)?

**Topic Mapping:** OS → Memory Management → Copy-on-Write

**Question:** Explain Copy-on-Write. Where is it used beyond fork()?

**Explanation:**
COW is an optimization: instead of copying data immediately, mark it shared and copy only when one party modifies it.

**In fork():**
- Child shares parent's pages (read-only)
- When child (or parent) writes to a page → page fault → OS creates private copy
- Programs that fork+exec immediately: no pages ever copied (exec replaces address space)

**In other contexts:**

1. **File systems (ZFS, Btrfs, APFS):** On write, don't modify original block → write to new block → update metadata. Original block preserved. Enables snapshots cheaply.

2. **Redis persistence (BGSAVE):** `fork()` child for snapshotting → child sees snapshot of memory at fork time → parent continues serving requests (COW keeps snapshot consistent).

3. **String implementations:** Some languages use COW strings. Copying string = same buffer until one is modified.

4. **Kubernetes/Docker image layers:** Layers are COW — container sees all layers, modifications go to writable top layer.

**Interview Tip:** COW is fundamental to efficient systems. Snapshots, fork(), and container layers all rely on it.

---

### Q54. What is the Difference Between Logical and Physical Address?

**Topic Mapping:** OS → Memory Management → Address Spaces

**Question:** Distinguish between logical and physical addresses. Who translates them?

**Explanation:**
**Logical Address (Virtual Address):**
- Generated by CPU during program execution
- Relative to the process's own address space
- What the program "sees" — starts from 0
- Example: pointer value in C program = logical address

**Physical Address:**
- Actual address in RAM hardware
- What memory chips respond to
- Example: DRAM chip row/column address

**Translation:** MMU (Memory Management Unit) hardware translates logical → physical using page tables.

```
CPU generates logical address (0x1000)
    ↓
MMU looks up page table (TLB first)
    ↓
Physical address (0x8000)
    ↓
RAM access
```

**Address Binding:**
When does logical address get bound to physical?
- **Compile time:** Known at compile time (absolute code) — must load at specific address
- **Load time:** OS picks address when program loads (relocatable code)
- **Execution time:** Address can change during execution (requires hardware MMU — modern systems)

**Interview Tip:** Logical/virtual addresses = what programmers deal with. Physical addresses = what RAM sees. MMU does the mapping.

---

### Q55. What is Demand Paging vs Pre-paging?

**Topic Mapping:** OS → Memory Management → Demand Paging

**Question:** Compare demand paging and pre-paging. What are the tradeoffs?

**Explanation:**
**Demand Paging:**
Load pages from disk only when they're referenced (page fault occurs).
- Pros: Only load what's needed, fast startup, supports larger-than-RAM programs
- Cons: Page faults have latency (~10ms disk access)

**Pre-paging:**
Load multiple pages at once (predict future needs and load them proactively).
- Pros: Fewer page faults if prediction is good
- Cons: Wasted I/O if pre-fetched pages not used

**Prepaging at Start:** Load all pages needed by process upfront. Reduces startup page faults but slow startup.

**Real-world Pre-fetching:**
- OS detects sequential access pattern → prefetch next pages
- `madvise()` system call: programmer hints to OS about access patterns
  - `MADV_SEQUENTIAL`: prefetch ahead
  - `MADV_RANDOM`: disable prefetch
  - `MADV_WILLNEED`: prefetch now

**Interview Tip:** Modern OSes use demand paging + intelligent prefetching. The key is matching prefetch strategy to access pattern.

---

### Q56. What is a Race Condition in OS vs Concurrency?

**Topic Mapping:** OS → Synchronization → Race Conditions

**Question:** Can race conditions occur in the OS kernel? How does the kernel protect against them?

**Explanation:**
Yes — OS kernel is multi-threaded (handles interrupts, multiple CPUs), so race conditions in kernel are a serious concern.

**Kernel Race Condition Scenarios:**
1. Two CPUs simultaneously modify the same process's page table
2. Interrupt handler and system call handler access same data structure
3. Two kernel threads manipulate the same file inode

**Kernel Synchronization Primitives:**

1. **Spin locks:** Used where sleeping is impossible (interrupt context). Short hold times.
2. **Mutexes (kernel mutex):** For longer critical sections where sleeping is allowed.
3. **Read-Write Locks:** For data structures read frequently, written rarely.
4. **RCU (Read-Copy-Update):** Lock-free for reads. Writers create new version, update pointer atomically, wait for all readers to finish, then free old version. Extremely efficient for read-heavy data.
5. **Atomic operations:** CPU hardware provides atomic increment, CAS (compare-and-swap).
6. **Disabling interrupts:** On uniprocessor, disable interrupts during critical section.

**Interview Tip:** Linux kernel uses RCU extensively for routing tables, networking, file system dentry cache — all read-heavy.

---

### Q57. What is an Interrupt?

**Topic Mapping:** OS → I/O Management → Interrupts

**Question:** What is an interrupt? Distinguish between hardware and software interrupts.

**Explanation:**
An interrupt is a signal to the CPU that an event requires immediate attention. CPU stops current execution, saves state, runs interrupt handler, resumes.

**Hardware Interrupts:**
- Generated by I/O devices: keyboard, disk controller, NIC
- Example: Disk completes DMA transfer → sends interrupt → OS marks waiting process as ready
- Timer interrupt: used for time-slicing (scheduler)
- Asynchronous — happen at any time

**Software Interrupts (Traps/Exceptions):**
- Generated by CPU itself or software
- System calls: `INT 0x80` or `SYSCALL` instruction
- Page faults, divide by zero, illegal instruction
- Synchronous — result of current instruction

**Interrupt Handling:**
1. CPU completes current instruction
2. Saves state (registers, PC) to kernel stack
3. Looks up interrupt vector table → gets handler address
4. Jumps to handler (ISR — Interrupt Service Routine)
5. ISR runs in kernel mode
6. ISR finishes → restores state → resumes interrupted code

**Interrupt Priority:** Hardware interrupts can be masked/disabled. Nested interrupts possible (higher priority interrupts a lower-priority ISR).

**Interview Tip:** Timer interrupt is what enables preemptive multitasking. Every X ms, timer fires → OS scheduler runs → may switch processes.

---

### Q58. What is the OOM Killer?

**Topic Mapping:** OS → Memory Management → OOM Killer

**Question:** What is the Linux OOM Killer? How does it choose which process to kill?

**Explanation:**
When Linux exhausts all available memory AND swap, the **Out-of-Memory (OOM) Killer** is invoked. It kills one or more processes to free memory and prevent system crash.

**OOM Score:**
Each process has an `oom_score` (0-1000). Higher = more likely to be killed.

**Factors in oom_score calculation:**
- Memory consumption (larger processes score higher)
- Process runtime (newly started processes more likely to be killed)
- Priority/nice value
- `/proc/[pid]/oom_score_adj`: user can adjust (-1000 to +1000)

**Setting oom_score_adj:**
```bash
echo -1000 > /proc/$(pidof critical_service)/oom_score_adj  # protect from OOM killer
echo 1000 > /proc/$(pidof low_priority)/oom_score_adj      # make it first target
```

**OOM Killer in Production:** Often caused by memory leaks or underestimating memory requirements. `dmesg | grep -i "oom"` to check logs.

**Interview Tip:** In Kubernetes, each pod has resource limits. If exceeded, OOM killer terminates the pod's container. Always set proper memory requests and limits.

---

### Q59. What is `mmap()`?

**Topic Mapping:** OS → Memory Management → mmap

**Question:** What is mmap()? How does it differ from read()/write()?

**Explanation:**
`mmap()` maps a file (or anonymous memory) directly into the process's virtual address space. The process can then access file contents as if they were in memory (no explicit read/write calls).

```c
void *addr = mmap(NULL, size, PROT_READ|PROT_WRITE, MAP_SHARED, fd, offset);
// Now read/write addr directly — accesses the file
```

**vs Traditional I/O:**
- `read()`: kernel copies from page cache to user buffer (double copy)
- `mmap()`: page cache directly visible in user address space (single mapping, no copy)

**Benefits:**
- Zero-copy I/O (especially for large files)
- Efficient for random access (OS handles page loading on demand)
- Multiple processes mapping same file → shared memory automatically

**Use Cases:**
- Large file processing (log analysis)
- Shared memory IPC (MAP_SHARED + MAP_ANONYMOUS)
- Dynamic linker (maps shared library code)
- Database engines (PostgreSQL, SQLite use mmap)

**Interview Tip:** mmap is zero-copy for file I/O. Combined with `MADV_SEQUENTIAL`, it's faster than read() for sequential access of large files.

---

### Q60. What is the Difference Between Kernel Mode and User Mode?

**Topic Mapping:** OS → Kernel → Protection Rings

**Question:** Explain kernel mode vs user mode. Why is this distinction important?

**Explanation:**
Modern CPUs operate in different privilege levels (protection rings).

**User Mode (Ring 3):**
- Normal application code runs here
- Cannot directly access hardware
- Cannot execute privileged instructions (like `HLT`, `IN`, `OUT`)
- Memory access restricted to own address space
- Access to kernel: only through system calls

**Kernel Mode (Ring 0):**
- OS kernel runs here
- Full access to all hardware and instructions
- Can access any memory address
- Executes interrupt handlers, system calls

**Why This Distinction:**
- **Security:** User programs can't corrupt kernel or other processes
- **Stability:** Buggy user program can't crash OS

**Transition:**
- User → Kernel: system call, interrupt, exception
- Kernel → User: return from system call/interrupt

**Protection Rings (x86):**
- Ring 0: Kernel
- Ring 1, 2: Rarely used (device drivers historically)
- Ring 3: User applications

**Hypervisor (Ring -1/VMX root mode):** Hypervisors run at even higher privilege than OS kernel.

**Interview Tip:** Every system call = user mode → kernel mode → user mode. This dual-mode protection is the foundation of OS security.

---


---

## INTERMEDIATE QUESTIONS (Q61–Q130)

---

### Q61. How does the Linux Completely Fair Scheduler (CFS) work?

**Topic Mapping:** OS → CPU Scheduling → CFS

**Question:** Explain CFS. What is virtual runtime and the red-black tree?

**Explanation:**
CFS (Linux 2.6.23+) aims to give each runnable process a "completely fair" share of CPU time.

**Virtual Runtime (vruntime):**
Each process tracks how much CPU time it has "consumed" normalized by weight (priority). Process with smallest vruntime runs next.

```
vruntime += actual_run_time × (NICE_0_weight / process_weight)
```
Higher priority (lower nice value) = smaller weight divisor = vruntime grows slower = runs more.

**Red-Black Tree:**
All runnable processes stored in a red-black tree keyed by vruntime. Leftmost node (smallest vruntime) = next to run. O(log N) operations.

**Scheduling Period:** CFS tries to run every process at least once per scheduling period (default: 6ms × N processes, min 0.75ms per process).

**Why CFS is great:**
- No starvation (every process runs proportionally)
- O(log N) scheduling decisions
- Handles mixed workloads (CPU-bound + I/O-bound) fairly

**Interview Tip:** Linux CFS uses vruntime + red-black tree. Understanding this impresses interviewers. For comparison, Windows uses multilevel feedback queues.

---

### Q62. What is Priority Inversion and how does Priority Inheritance solve it?

**Topic Mapping:** OS → Synchronization → Priority Inversion

**Question:** Describe priority inversion. What happened in the Mars Pathfinder mission?

**Explanation:**
**Priority Inversion:** High-priority task indirectly blocked by low-priority task through a medium-priority task.

**Scenario:**
- H (high priority), M (medium priority), L (low priority)
- L holds mutex → H waits for mutex → M preempts L (M has higher priority than L) → L can't run → mutex not released → H waits indefinitely for M to finish!

**Mars Pathfinder (1997):**
A low-priority meteorological task held a mutex needed by a high-priority bus management task. A medium-priority communication task kept preempting the low-priority task. Watchdog timer detected bus management task not running → system reset repeatedly. Fixed by enabling priority inheritance.

**Priority Inheritance:**
When a high-priority task waits for a mutex held by a low-priority task, temporarily raise the low-priority task's priority to equal the waiting task's priority. L runs with H's priority → finishes quickly → releases mutex → H unblocks → L returns to its original priority.

**Priority Ceiling:**
Alternative: mutex has a "ceiling priority" equal to highest priority of any task that could lock it. Any task locking it temporarily gets ceiling priority.

**Interview Tip:** Priority inversion is common in embedded/real-time systems. Mention Mars Pathfinder — it's a famous real-world example that impresses interviewers.

---

### Q63. Explain the difference between Cooperative and Preemptive Multitasking with OS examples.

**Topic Mapping:** OS → CPU Scheduling → Multitasking

**Question:** When does cooperative multitasking fail? What real systems used it?

**Explanation:**
**Cooperative Multitasking:**
Process voluntarily yields CPU. OS cannot forcibly preempt.
- **Pros:** Simple, no race conditions at user level, deterministic
- **Cons:** One misbehaving process can freeze entire system
- **Examples:** Windows 3.x, Mac OS 9, early DOS programs, Node.js event loop (cooperative for JavaScript)

**Failure case:** A buggy process enters infinite loop → never yields → system hangs.

**Preemptive Multitasking:**
OS forcibly takes CPU using timer interrupt. Process doesn't need to cooperate.
- **Pros:** Fair, no single process can monopolize CPU, essential for multi-user systems
- **Cons:** Race conditions possible (process preempted at any point)
- **Examples:** Linux, Windows NT+, macOS, Android, iOS

**Hybrid cases:**
- **Go runtime:** Uses cooperative preemption (goroutines yield at function calls/loops). Go 1.14+ added signal-based preemption for CPU-bound goroutines.
- **JavaScript (Node.js):** Single-threaded event loop. Cooperative — long computation blocks event loop.

**Interview Tip:** Node.js uses cooperative multitasking for JavaScript, but OS-level threads handle I/O preemptively in libuv. This is why long computation in Node.js blocks all requests.

---

### Q64. What is the Two-Level Page Table?

**Topic Mapping:** OS → Memory Management → Multilevel Paging

**Question:** Why is a simple page table impractical for 64-bit systems? Explain multi-level paging.

**Explanation:**
**Problem with single-level page table:**
- 32-bit: 4GB address space / 4KB pages = 1M entries × 4 bytes = 4MB per process
- 64-bit: 2^64 / 4KB = 2^52 entries × 8 bytes = 32 petabytes per process! Impossible.

**Solution: Multi-level (Hierarchical) Page Table**
Split page number into multiple levels. Only allocate page table entries for actually-used address ranges.

**x86-64 uses 4-level paging:**
48-bit virtual address → 4 × 9-bit indices + 12-bit offset

```
Virtual Address: [PML4 idx 9bit][PDP idx 9bit][PD idx 9bit][PT idx 9bit][offset 12bit]
PML4 (512 entries) → PDP → PD → PT → Physical Frame
```

**Benefit:** Sparse allocation. A process using only 100MB of a 128TB virtual address space only needs page table pages for those 100MB, not the entire space.

**Trade-off:** 4 memory accesses for a translation (vs 1 for single-level). TLB mitigates this (99%+ hit rate).

**Interview Tip:** x86-64 uses 4-level page tables (recently extended to 5-level for even larger address spaces). TLB makes this practical.

---

### Q65. How does the Linux kernel use the slab allocator?

**Topic Mapping:** OS → Memory Management → Kernel Allocator

**Question:** What is the slab allocator? Why is it better than general malloc for kernel objects?

**Explanation:**
The kernel frequently allocates and frees fixed-size objects (inodes, dentries, PCBs, sockets). General-purpose allocators have overhead for each allocation and suffer from fragmentation.

**Slab Allocator (Jeff Bonwick, 1994):**
Pre-allocates "slabs" (pages) divided into fixed-size chunks for specific object types. When an object is freed, it's returned to the slab (not back to general memory) and kept initialized for next allocation.

**Three slab states:**
- **Full:** All objects in use
- **Partial:** Some objects used, some free
- **Empty:** All objects free

**Benefits:**
1. O(1) allocation (no searching for right-sized chunk)
2. Objects returned initialized (constructors already run) → cache warming
3. No fragmentation for fixed-size objects
4. Cache-friendly (objects of same type together → hot cache)

**SLUB (modern Linux):** Improved slab allocator with less metadata overhead, used in Linux since 2.6.22.

**Interview Tip:** The slab allocator is why Linux kernel can allocate thousands of inodes/second efficiently. Application-level: similar concept in memory pools/object pools in games/databases.

---

### Q66. What is the difference between Hard Real-Time and Soft Real-Time systems?

**Topic Mapping:** OS → Real-Time Systems

**Question:** Distinguish hard real-time from soft real-time. Give examples.

**Explanation:**
**Hard Real-Time:**
Missing a deadline is a system failure — catastrophic consequences.
- Timing guarantees are absolute
- Examples: Airbag controller (miss 50ms deadline → person dies), anti-lock brakes, nuclear plant control, pacemakers, flight control systems
- OS: bare-metal RTOS (FreeRTOS, VxWorks, QNX)

**Soft Real-Time:**
Missing a deadline is undesirable but not catastrophic. System degrades gracefully.
- Performance metric, not correctness metric
- Examples: Video streaming (dropped frame = annoying), online gaming (lag spike = frustrating), phone call (slight audio delay)
- OS: Linux with PREEMPT_RT, Windows

**Firm Real-Time:**
Between hard and soft. Results after deadline are useless but no catastrophic failure.
- Example: Financial trading system (late trade = missed opportunity, not disaster)

**RTOS Characteristics:**
- Deterministic scheduling (priority-based, preemptive)
- Bounded interrupt latency
- Priority inheritance support
- Minimal OS jitter

**Interview Tip:** Many embedded systems are hard real-time. Linux is soft real-time at best. The PREEMPT_RT patch improves latency but still isn't hard real-time.

---

### Q67. Explain the concept of a System Call Table.

**Topic Mapping:** OS → System Calls → System Call Table

**Question:** How does the OS dispatch a system call to the right handler?

**Explanation:**
The system call table (syscall table) is an array in kernel memory mapping system call numbers to their handler functions.

**Linux System Call Dispatch:**
1. User calls `read(fd, buf, count)` → glibc wrapper
2. glibc: puts syscall number (`__NR_read` = 0) in `rax`, args in `rdi`, `rsi`, `rdx`
3. Executes `syscall` instruction → switches to kernel mode
4. CPU jumps to `SYSCALL_entry_trampoline`
5. Kernel uses `rax` to index `sys_call_table[]`
6. `sys_call_table[0] = sys_read` → called with saved arguments
7. Handler executes, returns result in `rax`
8. `sysret` instruction → user mode

**Security:** Syscall number must be < table size. Each handler validates arguments (user pointers must be in user space!).

**View on Linux:**
```bash
cat /usr/include/asm/unistd_64.h  # syscall numbers
ausyscall --dump  # list all syscalls
strace ls  # trace syscalls made by ls
```

**Interview Tip:** `strace` shows syscalls made by any program. Incredibly useful for debugging "what is this program actually doing?" Use `strace -e openat,read,write ls` to filter.

---

### Q68. What is the difference between Concurrency and Parallelism?

**Topic Mapping:** OS → Threads → Concurrency vs Parallelism

**Question:** Are concurrent programs always parallel? Explain with an OS-level perspective.

**Explanation:**
**Concurrency:** Multiple tasks are in progress simultaneously (interleaved on single core possible). About DEALING WITH multiple things.

**Parallelism:** Multiple tasks execute at the exact same instant (requires multiple cores). About DOING multiple things simultaneously.

**Single-Core System:**
- Concurrent: YES (time-sliced via scheduler)
- Parallel: NO (only one instruction executing at a time)

**Multi-Core System:**
- Concurrent: YES
- Parallel: YES (if tasks actually run on different cores)

**OS Perspective:**
- OS scheduler creates concurrency even on single core
- Multi-core OS enables parallelism with proper thread affinity and load balancing

**Go's famous quote (Rob Pike):** "Concurrency is about structure, parallelism is about execution."

**Examples:**
- Server handling 10,000 connections on 8 cores: concurrent (10K tasks) + parallel (8 simultaneous) + async I/O
- MapReduce: parallel (multiple machines processing chunks simultaneously)

**GIL (Python):** CPython has Global Interpreter Lock → Python threads are concurrent but NOT parallel (can't use multiple cores for CPU-bound work). Use multiprocessing instead.

**Interview Tip:** Concurrency ≠ Parallelism. A concurrent system doesn't need multiple cores. This distinction is fundamental and often confused.

---

### Q69. What is a Kernel Module?

**Topic Mapping:** OS → Kernel → Kernel Modules

**Question:** What are loadable kernel modules? How do they work in Linux?

**Explanation:**
Kernel modules (LKM — Loadable Kernel Modules) are pieces of code that can be loaded/unloaded into the running kernel without rebooting.

**Purpose:** Add functionality (device drivers, file system support, network protocols) without recompiling entire kernel.

**Linux Module Operations:**
```bash
lsmod          # list loaded modules
insmod mod.ko  # insert module (requires root)
rmmod mod      # remove module
modprobe mod   # load with dependencies
```

**Module Structure (skeleton):**
```c
#include <linux/module.h>
#include <linux/kernel.h>

static int __init my_init(void) {
    printk(KERN_INFO "Module loaded\n");
    return 0;
}
static void __exit my_exit(void) {
    printk(KERN_INFO "Module removed\n");
}
module_init(my_init);
module_exit(my_exit);
MODULE_LICENSE("GPL");
```

**Security Concern:** Kernel modules run in kernel mode (ring 0) — a malicious or buggy module can crash or compromise the entire system.

**Secure Boot / Module Signing:** Modules must be cryptographically signed to prevent loading malicious modules.

**Interview Tip:** Device drivers in Linux are almost always kernel modules. When you plug in a USB device, the USB driver module handles it. `lsmod` shows what's loaded on your Linux machine.

---

### Q70. What is Thrashing and how does the Working Set prevent it (detailed)?

**Topic Mapping:** OS → Memory Management → Thrashing

**Question:** Derive the relationship between CPU utilization and degree of multiprogramming. At what point does thrashing occur?

**Explanation:**
**CPU Utilization Curve:**

```
CPU Utilization
     |        ___
     |      /     \
     |    /         \  ← thrashing begins here
     |  /
     | /
     +------------------→ Degree of Multiprogramming (N)
```

**Why the curve drops:**
- Low N: CPU idles when processes do I/O. Add more processes → CPU busier.
- Optimal N: CPU highly utilized.
- High N: Each process has too few frames → frequent page faults → process waits for disk → CPU thinks it should add more processes → worse page faults → death spiral.

**Working Set Model Math:**
- `WS(i)` = working set size of process i
- `D = Σ WS(i)` = total demand
- `m` = total physical frames
- If `D > m` → thrashing imminent
- OS should reduce N until `D ≤ m`

**Detection:** Monitor page fault rate per process.
- Too high → give more frames (increase WS window)
- Too low → take frames away (reduce waste)

**Prevention:**
1. Limit degree of multiprogramming (swap out processes)
2. Add more RAM
3. Use working set model to allocate frames

**Interview Tip:** The key insight: CPU utilization is a misleading metric during thrashing. CPU appears busy (running page fault handlers), but no useful work is done. Always monitor page fault rate alongside CPU utilization.

---

### Q71. How does the OS handle a Page Fault?

**Topic Mapping:** OS → Memory Management → Page Fault Handling

**Question:** Walk through every step of a page fault from hardware to resuming execution.

**Explanation:**
**Complete Page Fault Flow:**

1. **CPU generates virtual address** (e.g., mov eax, [0x1000])
2. **MMU looks up TLB** → miss → walks page table
3. **Page table entry: valid bit = 0** → MMU raises page fault exception
4. **CPU traps to OS page fault handler** (saves all registers, switches to kernel mode)
5. **OS page fault handler runs:**
   a. Gets faulting address from CR2 register (x86)
   b. Checks: is this address in process's valid virtual address space (VMA)?
   c. If NO: segfault (SIGSEGV) → process killed
   d. If YES: is page in swap? On disk? Or new (zero page)?
6. **Find a free frame:**
   - If available: use it
   - If not: run page replacement algorithm → choose victim frame
   - If victim is dirty (modified): write victim page to swap first (disk I/O)
7. **Load needed page from disk into frame** (disk I/O, process blocks)
8. **Update page table:** set valid bit = 1, set frame number
9. **Invalidate TLB entry** (if victim was cached)
10. **Resume faulting process:** restart the faulting instruction (not the next one!)

**Key:** Instruction is **restarted** (not resumed from next instruction) because it didn't complete.

**Interview Tip:** Why restart, not resume? Because the faulting instruction didn't complete its execution — it needs to retry with the page now present.

---

### Q72. Explain the Banker's Algorithm with multiple resource types.

**Topic Mapping:** OS → Deadlocks → Banker's Algorithm (Advanced)

**Question:** Run the Banker's Algorithm safety check with a full example (3 processes, 3 resource types).

**Explanation:**

**Initial State:**
```
Allocation    Max      Available
     A B C    A B C    A B C
P0 [ 0 1 0] [7 5 3]  [3 3 2]
P1 [ 2 0 0] [3 2 2]
P2 [ 3 0 2] [9 0 2]
P3 [ 2 1 1] [2 2 2]
P4 [ 0 0 2] [4 3 3]

Need = Max - Allocation:
P0: [7 4 3]
P1: [1 2 2]
P2: [6 0 0]
P3: [0 1 1]
P4: [4 3 1]
```

**Safety Algorithm Execution:**
```
Work = [3 3 2]
Step 1: Find process where Need ≤ Work
  P1 Need=[1,2,2] ≤ [3,3,2] ✓ → Work=[3+2,3+0,2+0]=[5,3,2], Finish[1]=true
Step 2: 
  P3 Need=[0,1,1] ≤ [5,3,2] ✓ → Work=[5+2,3+1,2+1]=[7,4,3], Finish[3]=true
Step 3:
  P4 Need=[4,3,1] ≤ [7,4,3] ✓ → Work=[7+0,4+0,3+2]=[7,4,5], Finish[4]=true
Step 4:
  P0 Need=[7,4,3] ≤ [7,4,5] ✓ → Work=[7+0,4+1,5+0]=[7,5,5], Finish[0]=true
Step 5:
  P2 Need=[6,0,0] ≤ [7,5,5] ✓ → Safe!

Safe sequence: <P1, P3, P4, P0, P2>
```

**Resource Request Algorithm:**
If P1 requests [1,0,2]:
1. Request ≤ Need? [1,0,2] ≤ [1,2,2]? Yes
2. Request ≤ Available? [1,0,2] ≤ [3,3,2]? Yes
3. Tentatively allocate. Run safety check. If safe, grant. Else, P1 waits.

**Interview Tip:** Walk through the algorithm step by step. Examiners love this question.

---

### Q73. What are the different types of kernel synchronization in Linux?

**Topic Mapping:** OS → Synchronization → Kernel Synchronization

**Question:** Describe spinlock, mutex, semaphore, RCU in the context of Linux kernel. When to use each?

**Explanation:**

**Linux Kernel Sync Primitives:**

| Primitive | Sleepable? | SMP-safe? | Use Case |
|-----------|------------|-----------|----------|
| Spinlock | No | Yes | Very short critical sections, interrupt context |
| Mutex | Yes | Yes | Long critical sections, process context |
| Semaphore | Yes | Yes | Resource counting, older API |
| RWLock | No (spin) | Yes | Infrequent writes, many reads |
| RW Semaphore | Yes | Yes | Long read-heavy critical sections |
| RCU | Yes (readers) | Yes | Very frequent reads, rare writes |
| Seqlock | No | Yes | Rarely written data (e.g., time) |

**RCU Deep Dive:**
```
Reader: rcu_read_lock() → access data → rcu_read_unlock() (no blocking, no spinlock!)
Writer: copy data → modify copy → rcu_assign_pointer(old, new) → synchronize_rcu() → free old
```
Readers are completely lock-free. Writes wait for all current readers to finish (grace period).

**Seqlock:** Writer increments sequence counter before+after write. Reader checks counter before+after read — if odd or changed → retry. Used for `jiffies` (system timer).

**Interview Tip:** RCU is Linux's crown jewel for synchronization. Routing table lookups, inode cache — all use RCU. Know that readers never block in RCU.

---

### Q74. What is Memory-Mapped I/O vs Port-Mapped I/O?

**Topic Mapping:** OS → I/O Management → I/O Addressing

**Question:** How does the CPU communicate with I/O devices? Compare MMIO and PMIO.

**Explanation:**
**Port-Mapped I/O (PMIO / Isolated I/O):**
- Special I/O address space, separate from memory address space
- Accessed via special instructions: `IN port, reg` and `OUT port, val` (x86)
- Limited address space (65536 ports in x86)
- Example: `IN AL, 0x60` reads keyboard data register

**Memory-Mapped I/O (MMIO):**
- Device registers mapped into physical address space
- Access with regular memory instructions (MOV, LOAD, STORE)
- Some physical addresses don't correspond to RAM — they go to device registers
- More common in modern systems
- Example: GPU framebuffer mapped at high physical address, CPU writes pixels directly

**Advantages of MMIO:**
- All memory addressing modes available (pointer arithmetic, etc.)
- Simpler programming (no special instructions)
- Works with all CPU architectures (not x86-specific)

**Example — MMIO in ARM:**
Raspberry Pi GPIOs controlled by writing to memory-mapped registers at specific physical addresses.

**DMA + MMIO:** DMA controller uses MMIO to access device registers (source/dest/count setup), then transfers data directly without CPU.

**Interview Tip:** Modern systems heavily use MMIO. GPU drivers, network cards, storage controllers all use MMIO. Understanding MMIO is key to driver development.

---

### Q75. Explain Process Synchronization using Monitors in Java.

**Topic Mapping:** OS → Synchronization → Monitors → Java

**Question:** How does Java's synchronized keyword implement a monitor? Explain wait/notify.

**Explanation:**
Every Java object has an intrinsic lock (monitor). `synchronized` methods/blocks acquire this lock.

**Java Monitor:**
```java
class BoundedBuffer {
    private int[] buffer;
    private int count, in, out;
    
    public synchronized void insert(int item) throws InterruptedException {
        while (count == buffer.length) {
            wait();  // releases lock, waits in "wait set"
        }
        buffer[in] = item;
        in = (in + 1) % buffer.length;
        count++;
        notifyAll();  // wake all waiting threads
    }
    
    public synchronized int remove() throws InterruptedException {
        while (count == 0) {
            wait();  // releases lock, waits
        }
        int item = buffer[out];
        out = (out + 1) % buffer.length;
        count--;
        notifyAll();
        return item;
    }
}
```

**wait() semantics:**
1. Releases the lock
2. Suspends calling thread (moves to wait set)
3. When notified: re-acquires lock, checks condition (spurious wakeups possible!)
4. ALWAYS use `while` (not `if`) to check condition after wake-up

**notify() vs notifyAll():**
- `notify()`: wakes ONE arbitrary waiting thread
- `notifyAll()`: wakes ALL waiting threads (they all compete for lock again)
- Prefer `notifyAll()` unless exactly one thread can proceed

**Interview Tip:** Java's `synchronized` + `wait/notify` = monitor. Always use `while` loop to re-check condition after `wait()` (spurious wakeups are real in the JVM spec).

---

### Q76. What is the difference between Preemptive Priority Scheduling and Round Robin with Priorities?

**Topic Mapping:** OS → CPU Scheduling → Priority + Round Robin

**Question:** How do real OSes combine priority scheduling with time-slicing?

**Explanation:**
**Pure Priority Scheduling:** Always run highest-priority runnable process. Within same priority: FCFS. Problem: Low-priority processes can starve.

**Round Robin within Priority Levels:**
Multiple priority queues. Within each priority level, RR scheduling. When higher-priority process arrives, preempts current.

**Multilevel Queue Scheduling:**
```
Priority 0 (Real-time): FIFO/RR [quantum=10ms]
Priority 1 (System): RR [quantum=10ms]
Priority 2 (Interactive): RR [quantum=20ms]
Priority 3 (Batch): RR [quantum=100ms]
Priority 4 (Idle): FCFS
```

**Multilevel Feedback Queue (MFQ):**
Processes can move between queues based on behavior.
- New process → highest queue (short quantum, best response)
- If quantum expires (CPU-bound) → demoted to lower queue (longer quantum)
- If process waits for I/O → promoted (I/O-bound should be responsive)
- Aging: long-waiting low-priority process → promoted

**Real OS:**
- Windows: 32 priority levels. Threads move up on I/O completion, down on CPU exhaustion.
- Linux CFS: No fixed queues. Uses vruntime. Equivalent effect achieved continuously.

**Interview Tip:** MFBQ is considered by many to be the most general scheduling algorithm. It can approximate FCFS, SJF, and RR based on configuration.

---

### Q77. What is a Zombie Process vs an Orphan Process?

**Topic Mapping:** OS → Process Management → Zombie and Orphan

**Question:** What happens when a parent doesn't call wait()? What if the parent dies first?

**Explanation:**
**Zombie Process:**
- Child has finished executing (called `exit()`)
- But parent has NOT called `wait()` to collect exit status
- Child's PCB still exists (holds exit code, PID, resource usage stats)
- Process appears in `ps` as `Z` state
- Takes up no memory/CPU (dead), just a PCB slot
- If parent never calls `wait()`: zombie accumulates → if enough zombies, PID table exhausted → cannot create new processes

**Fix:** Parent must call `wait()` or `waitpid()`. Signal handler for `SIGCHLD` can call `waitpid(-1, NULL, WNOHANG)` to reap all dead children.

**Orphan Process:**
- Parent terminates before child finishes
- Child has no parent to collect its exit status
- Linux: orphan is "reparented" to PID 1 (init/systemd)
- init periodically calls `wait()` → automatically reaps orphans
- Not a problem — init handles it

**Detection:**
```bash
ps aux | grep Z  # find zombie processes
```

**Interview Tip:** Zombie = dead child, living parent who ignored it. Orphan = living child, dead parent → adopted by init. Zombies accumulate if parent is buggy. Orphans are automatically handled.

---

### Q78. How does the OS implement semaphores?

**Topic Mapping:** OS → Synchronization → Semaphore Implementation

**Question:** How does the kernel implement wait() and signal() without busy-waiting?

**Explanation:**
Semaphore with blocking (no busy-wait):

```c
typedef struct {
    int value;
    struct process *list;  // waiting queue
} semaphore;

void wait(semaphore *S) {
    S->value--;
    if (S->value < 0) {
        // add current process to S->list
        block();  // put process in waiting state, context switch
    }
}

void signal(semaphore *S) {
    S->value++;
    if (S->value <= 0) {
        // remove process P from S->list
        wakeup(P);  // put P in ready queue
    }
}
```

**Key insight:** When S->value < 0, its absolute value = number of waiting processes.

**Atomicity:** wait() and signal() themselves must be atomic (on multiprocessor: use spin locks to protect the semaphore's value and list).

**Linux futex (fast userspace mutex):**
User-space semaphores use futex:
- If no contention: operation in user space (no syscall!)
- If contention: fall back to kernel `futex()` syscall to block/wake
- Most lock/unlock operations are fast (no syscall)

**Interview Tip:** The real magic is that `block()` is a kernel operation that saves the process state and context-switches. The process doesn't consume CPU while blocked.

---

### Q79. What is the difference between Kernel Stack and User Stack?

**Topic Mapping:** OS → Process Management → Stacks

**Question:** Every process has two stacks. Explain why and how they're used.

**Explanation:**
**User Stack:**
- In process's virtual address space (user mode)
- Used for: function calls, local variables, return addresses
- Default size: 8MB (Linux), typically grows downward
- Protected from other processes
- If overflows: segfault (stack guard page)

**Kernel Stack:**
- Separate stack in kernel address space
- Used when process runs in kernel mode (system call, interrupt)
- Typically small: 4KB or 8KB (per-thread in Linux)
- Cannot page fault or sleep in some contexts (why kernel stack is small)

**Why two stacks?**
- Security: user-space code can corrupt its own user stack. Kernel must use a separate, protected stack.
- Context: kernel code needs a stack during system call execution; user stack may be in invalid state (or user intentionally manipulated it).

**Stack Switch:**
- User → kernel mode: CPU saves user `rsp` (stack pointer), switches to per-CPU kernel stack. On x86-64, this uses TSS (Task State Segment) to find kernel stack address.

**Kernel Stack Overflow:**
A kernel stack overflow is catastrophic (can corrupt adjacent kernel memory). That's why kernel stacks are kept small and recursive algorithms are avoided in kernel code.

**Interview Tip:** When a process makes a system call, it switches to the kernel stack. The user stack is irrelevant during kernel execution.

---

### Q80. Explain Segmentation Fault, Bus Error, and Stack Overflow — how does each occur at the OS level?

**Topic Mapping:** OS → Memory Management → Hardware Exceptions

**Question:** At the hardware/OS level, what causes each of these errors?

**Explanation:**

**Segmentation Fault (SIGSEGV):**
- Accessing virtual address not in process's valid memory regions (VMAs)
- OR violating memory protection (writing to read-only page)
- Hardware: MMU raises protection fault or page fault with no valid mapping
- OS: sends SIGSEGV, process terminates with core dump by default
```c
int *p = NULL; *p = 5;  // null deref — address 0 never mapped
char *s = "hello"; s[0] = 'H';  // write to read-only string literal page
```

**Bus Error (SIGBUS):**
- Misaligned memory access (accessing int at odd address on strict alignment architectures)
- OR accessing memory-mapped file that was truncated (SIGBUS not SIGSEGV)
- x86 handles some misalignment in hardware; ARM/SPARC raise bus error
```c
char buf[8];
int *p = (int*)(buf + 1);  // misaligned pointer
*p = 42;  // bus error on strict architectures
```

**Stack Overflow:**
- Process exhausts stack memory (infinite recursion, very large local variables)
- Stack grows until it hits the guard page (special page with no permissions)
- Guard page → page fault → OS sees fault in guard page region → SIGSEGV
- Stack cannot grow further because hitting heap or other mapping

**Interview Tip:** All three cause a signal to be sent to the process. SIGSEGV is most common. Stack overflow also causes SIGSEGV (not a special "stack overflow" signal).

---

### Q81–Q100 (OS Intermediate continued)

### Q81. What is Memory Protection and how is it implemented?

**Topic Mapping:** OS → Memory Management → Memory Protection

**Answer Summary:** Memory protection prevents one process from accessing another's memory. Implemented via:
1. **Hardware:** Page table protection bits (read/write/execute per page). MMU checks on every access.
2. **Bounds registers:** Base + limit (simple systems).
3. **Virtual address spaces:** Each process has its own page table — cannot address another's physical frames.
4. **NX bit (No-Execute):** Pages marked data cannot be executed. Prevents code injection attacks.

**W^X (Write XOR Execute):** A page should never be both writable AND executable simultaneously. Prevents shellcode injection.

**Real-world:** ASLR (Address Space Layout Randomization) + NX + stack canaries = basic exploit mitigations. All implemented with OS + hardware cooperation.

---

### Q82. What is a Page Table Walk?

**Topic Mapping:** OS → Memory Management → Page Table Walk

**Answer Summary:** When TLB misses, hardware page table walker (x86: automatic hardware walker) traverses page table levels:
1. Read CR3 → PML4 base address
2. Index into PML4 with bits 47:39 of virtual address
3. Follow pointer to PDP, then PD, then PT
4. Extract physical frame number from PT entry
5. Add offset → physical address
6. Update TLB with new mapping

Software-managed TLBs (MIPS): OS handles TLB miss in software (TLB miss handler).

**Cost:** 4 memory accesses (4-level page table). Mitigated by TLB (~99% hit rate) and page walker caches.

---

### Q83. What is the difference between fork() and vfork()?

**Topic Mapping:** OS → Process Management → vfork

**Answer Summary:**
- `fork()`: Child gets copy-on-write copy of parent's address space. Parent and child run concurrently.
- `vfork()`: Child shares parent's address space (no copy, no COW). Parent is **suspended** until child calls `exec()` or `exit()`. Child must NOT modify shared memory.

`vfork()` is faster than `fork()` when immediately followed by `exec()` (no page table copy, no COW setup). But dangerous — child corrupting parent's memory causes undefined behavior.

Modern advice: Use `posix_spawn()` instead (combines fork+exec efficiently without vfork's dangers).

---

### Q84. How does the OS implement pipes?

**Topic Mapping:** OS → IPC → Pipes

**Answer Summary:**
A pipe is a kernel-managed circular buffer (typically 64KB in Linux) in kernel memory.

Implementation:
1. `pipe(fd[2])` creates two FDs: `fd[0]` (read end), `fd[1]` (write end)
2. Write to `fd[1]`: data goes into kernel buffer
3. Read from `fd[0]`: data comes from kernel buffer
4. If buffer full: writer blocks. If empty: reader blocks.
5. When last write FD closed: reader gets EOF (read returns 0)

**Shell pipes:** `ls | grep txt`
```
fork() → child1 runs ls
fork() → child2 runs grep
pipe() → child1's stdout connected to pipe write end
pipe() → child2's stdin connected to pipe read end
```

**Named Pipes (FIFO):** `mkfifo /tmp/myfifo` — appears as a file, can be opened by unrelated processes. Same kernel buffer mechanism.

---

### Q85. What is the difference between Signal and Interrupt?

**Topic Mapping:** OS → IPC → Signals

**Answer Summary:**

| Feature | Interrupt | Signal |
|---------|-----------|--------|
| Origin | Hardware device or CPU | OS or another process |
| Target | CPU (hardware level) | Process (software level) |
| Handler | Interrupt Service Routine (kernel) | Signal handler (user-space) |
| Mechanism | Hardware interrupt line or instruction | OS sends to process via PCB |
| Example | Timer, disk I/O complete | SIGKILL, SIGSEGV, SIGTERM |

**Signal mechanism:**
- Process receives signal → OS marks it in PCB
- Before returning to user mode: OS checks pending signals → calls signal handler
- Signal handler runs in user-space (but on a separate signal stack)
- After handler: resume normal execution (or not, if handler calls exit)

**Common signals:**
- `SIGTERM` (15): polite termination request
- `SIGKILL` (9): unconditional kill (cannot be caught or ignored)
- `SIGSEGV` (11): segmentation fault
- `SIGCHLD`: child process terminated
- `SIGALRM`: timer expired

---

### Q86. What is an Inverse Page Table?

**Topic Mapping:** OS → Memory Management → Inverted Page Table

**Answer Summary:**
Standard page table: one entry per virtual page (grows with virtual address space — impractical for 64-bit).

**Inverted Page Table:** One entry per physical frame (grows with RAM, not virtual address space).
- Entry: (PID, virtual page number) → index = physical frame number
- Table size proportional to physical RAM, not virtual address space

**Problem:** Translating virtual → physical requires searching the inverted table (O(N) for N frames).

**Solution:** Hash table indexed by (PID, virtual page number) → O(1) average lookup.

**Used in:** IBM RISC System/6000, PowerPC, IA-64 (Itanium).

**Trade-off:** Memory efficient but harder to share pages (one entry per frame, but sharing means multiple virtual pages → same frame).

---

### Q87. Explain Copy-on-Write in File Systems (ZFS/Btrfs).

**Topic Mapping:** OS → File Systems → COW File Systems

**Answer Summary:**
Traditional file systems (ext4): overwrite data in place. If power fails mid-write → corruption.

COW file systems (ZFS, Btrfs, APFS):
1. Never overwrite existing data
2. Write new data to new blocks
3. Update metadata to point to new blocks (atomically)
4. Old blocks released

**Benefits:**
- **Atomic writes:** Power failure leaves old data intact (always consistent)
- **Snapshots:** Snapshot = save pointer to current root. COW means snapshots are cheap (just track which blocks belong to snapshot). Snapshot + new writes = new blocks written, old blocks preserved.
- **Clone (dedup):** Block shared between snapshot and live filesystem — no data duplication.

**ZFS additional features:** Checksums on every block, self-healing RAID-Z, end-to-end data integrity.

**Interview Tip:** COW file systems handle sudden power loss gracefully. Traditional file systems need `fsck` after crash. COW + checksums eliminates silent data corruption.

---

### Q88. What is RAID? Explain RAID 0, 1, 5, 6, and 10.

**Topic Mapping:** OS → Storage → RAID

**Answer Summary:**

| RAID | Drives | Fault Tolerance | Storage Efficiency | Use Case |
|------|--------|-----------------|-------------------|----------|
| RAID 0 (Striping) | ≥2 | None (worse!) | 100% | Performance only |
| RAID 1 (Mirror) | ≥2 | 1 drive failure | 50% | High availability |
| RAID 5 (Parity) | ≥3 | 1 drive failure | (N-1)/N | Balanced |
| RAID 6 (Dual parity) | ≥4 | 2 drive failures | (N-2)/N | High durability |
| RAID 10 (1+0) | ≥4 | 1 per mirror pair | 50% | Performance + HA |

**RAID 5:** Parity striped across drives. If one drive fails, reconstruct from parity + other drives. Slow write (parity calculation). Large rebuild time = vulnerable window.

**RAID 6 vs 5:** Two drives can fail simultaneously. Important as drive sizes grow (larger drives → longer rebuild time → higher chance of second failure during rebuild).

**Software RAID (mdadm in Linux) vs Hardware RAID (dedicated controller).**

---

### Q89. What is the /proc filesystem in Linux?

**Topic Mapping:** OS → File Systems → /proc

**Answer Summary:**
`/proc` is a virtual filesystem (no disk backing). Files are generated on-the-fly by the kernel when read. Provides interface to kernel data structures.

**Key files:**
- `/proc/[PID]/` — directory per process
  - `cmdline`: command line arguments
  - `status`: process state, memory usage, PID, etc.
  - `maps`: virtual memory areas (VMAs)
  - `fd/`: open file descriptors
  - `oom_score`: OOM killer score
- `/proc/cpuinfo` — CPU details
- `/proc/meminfo` — memory statistics
- `/proc/interrupts` — interrupt counts per IRQ
- `/proc/net/tcp` — TCP connections

**Write to /proc for configuration:**
```bash
echo 1 > /proc/sys/net/ipv4/ip_forward  # enable IP forwarding
echo 3 > /proc/sys/vm/drop_caches       # drop page/inode/dentry caches
```

**Interview Tip:** `ps`, `top`, `netstat` all read from `/proc`. Understanding `/proc` helps debugging and performance tuning without special tools.

---

### Q90. What is the Completely Fair Scheduler's handling of I/O-bound vs CPU-bound processes?

**Topic Mapping:** OS → CPU Scheduling → CFS and Process Types

**Answer Summary:**
CFS naturally favors I/O-bound processes because they use little CPU (small actual vruntime) — when they become runnable, their vruntime is small → they run immediately.

**I/O-bound process:**
- Sleeps frequently (waiting for I/O)
- Accumulates little vruntime
- When woken up: has smallest vruntime → scheduled immediately
- Gets fast response time (good for interactive apps)

**CPU-bound process:**
- Runs until preempted
- Accumulates vruntime quickly
- Scheduled less frequently (fair share, but not preferred)

**Sleeper Fairness:**
A process that slept for a long time could have very small vruntime → completely starve others when it wakes. CFS caps the lag: a process's vruntime is set to `min(current vruntime in tree, min_vruntime - threshold)` on wake-up.

**Interview Tip:** CFS's elegance: no explicit classification of I/O-bound vs CPU-bound. The vruntime mechanism naturally gives I/O-bound processes better responsiveness.

---

### Q91–Q100 (OS Intermediate Summary Questions)

### Q91. What is Address Space Layout Randomization (ASLR)?

**Topic Mapping:** OS → Security → ASLR

**Answer Summary:** ASLR randomizes the virtual addresses of key regions (stack, heap, code, libraries) each time a program runs. Attackers cannot hardcode addresses for buffer overflow exploits (return-to-libc, ROP attacks) because addresses change. Combined with NX (no-execute) and PIE (position-independent executables), forms a strong defense.

Enabled in Linux: `echo 2 > /proc/sys/kernel/randomize_va_space`

---

### Q92. What is a page cache?

**Topic Mapping:** OS → Memory Management → Page Cache

**Answer Summary:** Linux page cache (buffer cache): recently read file data cached in free memory. Next read of same data → served from memory (cache hit). File writes → written to cache first (write-back), synced to disk later. `sync`/`fsync()` flushes to disk.

Page cache is why `free` shows most memory as "used" — Linux never lets memory go idle. `available` column = memory that can be freed for applications (page cache evictable).

---

### Q93. What is lazy allocation (overcommit)?

**Topic Mapping:** OS → Memory Management → Memory Overcommit

**Answer Summary:** Linux overcommits memory. `malloc(10GB)` on a system with 4GB RAM + 4GB swap succeeds — OS doesn't allocate physical memory yet. Physical pages allocated on first access (demand paging). Works because most processes don't use all requested memory.

Risk: if processes actually try to use overcommitted memory → OOM killer activates.

`/proc/sys/vm/overcommit_memory`: 0=heuristic, 1=always allow, 2=never overcommit.

---

### Q94. What is a Spinlock vs Mutex — when does spin lock win?

**Topic Mapping:** OS → Synchronization → Spinlock vs Mutex

**Answer Summary:** Spinlock wins when:
1. Critical section is shorter than cost of context switch (~1-10μs)
2. Multiple cores available (spinning core can wait while holding core finishes)
3. Interrupt context (cannot sleep)

Mutex wins when:
1. Critical section is long
2. Single core (spinning is wasteful — holder can't run)
3. User-space applications

Modern Linux: mutexes are adaptive — spin briefly, then sleep if lock not released quickly.

---

### Q95. What is huge pages (THP — Transparent Huge Pages)?

**Topic Mapping:** OS → Memory Management → Huge Pages

**Answer Summary:** Standard pages = 4KB. Huge pages = 2MB (or 1GB). With huge pages:
- TLB covers more memory per entry (2MB vs 4KB → 512× more coverage)
- Fewer TLB misses for large working sets (databases, JVM heap)
- Fewer page table levels needed

Linux THP: Automatically promotes contiguous 4KB pages to 2MB huge pages when beneficial. Some databases (Oracle, MongoDB) disable THP because it can cause latency spikes during promotion/demotion.

Explicit huge pages: `mmap()` with `MAP_HUGETLB` or pre-allocated huge pages in `/proc/sys/vm/nr_hugepages`.

---

### Q96. What is a context switch vs a mode switch?

**Topic Mapping:** OS → Process Management → Context Switch

**Answer Summary:**

**Mode switch:** CPU changes privilege level (user → kernel). Happens on system call, interrupt. Does NOT necessarily switch processes. Saves minimal state (user rsp, rip). Fast (~100 cycles).

**Context switch:** OS switches from one process/thread to another. Requires: save all registers + PC + stack pointer of current process to PCB, load new process's state from PCB, update page table pointer (CR3 on x86), flush TLB (unless ASID support). Slow (~1000-10000 cycles + cache miss penalty).

Mode switch is a subset of context switch. Every context switch involves a mode switch, but not vice versa.

---

### Q97. What is the difference between mmap and brk/sbrk?

**Topic Mapping:** OS → Memory Management → Heap Allocation

**Answer Summary:**
Both are system calls for memory allocation.

`brk()`/`sbrk()`: extend/shrink the process's heap by moving the "program break" (top of data segment). Simple, contiguous. Old glibc `malloc` uses this for small allocations.

`mmap(MAP_ANONYMOUS)`: allocate anywhere in virtual address space. More flexible. Used for large allocations (>128KB in glibc). Returns mapped region that can be `munmap`ped independently.

Modern glibc malloc: `brk()` for small allocations (fast), `mmap()` for large allocations (can be released individually without fragmenting heap).

---

### Q98. What is a process group and session?

**Topic Mapping:** OS → Process Management → Process Groups

**Answer Summary:**
- **Process group:** Set of processes with same PGID. Signals can be sent to entire group (`kill(-pgid, SIGTERM)`). Shell pipeline creates a process group.
- **Session:** Set of process groups. Created by `setsid()`. Session has controlling terminal.
- **Controlling terminal:** First terminal opened by session leader. SIGHUP sent to foreground process group when terminal disconnects.
- **Shell:** Each command pipeline = one process group. Foreground group gets keyboard signals. `CTRL+C` sends SIGINT to entire foreground process group.

---

### Q99. What is the init process and systemd?

**Topic Mapping:** OS → Process Management → Init

**Answer Summary:**
- PID 1 is the first process started by the kernel after boot.
- Responsible for starting all other processes (init scripts, services).
- Adopts orphan processes.
- If PID 1 dies → kernel panic.

**Traditional init (SysV):** Sequential startup scripts (`/etc/init.d/`). Slow boot (sequential).

**systemd (modern Linux):** Parallel service startup, dependency management, socket activation, cgroups integration, logging (journald). Much faster boot. Controversial but now standard on most distros.

`systemctl start/stop/status nginx` — manage services.
`journalctl -u nginx` — view service logs.

---

### Q100. What is a Futex?

**Topic Mapping:** OS → Synchronization → Futex

**Answer Summary:**
Futex (Fast Userspace muTEX) — hybrid synchronization: fast path in user space, slow path in kernel.

**Uncontended lock:** Atomically update a shared integer in user memory (no syscall needed). Extremely fast.

**Contended lock:** If another thread holds the lock, call `futex(FUTEX_WAIT)` syscall → kernel blocks the thread. When lock released: `futex(FUTEX_WAKE)` → kernel wakes waiter.

Most lock/unlock operations → no syscall (fast path). Only contention → syscall.

All modern synchronization primitives in Linux (pthreads mutex, Java synchronized, Go sync.Mutex) are implemented with futexes under the hood.


### Q101. What is the difference between synchronous and asynchronous I/O?

**Topic Mapping:** OS → I/O Management → Sync vs Async I/O

**Explanation:**

**Synchronous I/O (blocking):**
- `read()` call: thread blocks until data is ready
- Simple programming model
- Wastes thread during I/O wait
- Default behavior

**Asynchronous I/O (non-blocking):**
- `aio_read()` or `io_uring`: returns immediately
- Process continues; kernel notifies when I/O completes (callback, signal, or polling)
- Complex programming model but scalable

**Non-blocking I/O + select/epoll:**
- Set FD non-blocking (`O_NONBLOCK`)
- Poll multiple FDs with `epoll_wait()` — kernel tells which are ready
- Event-driven: handle whichever FD is ready

**io_uring (Linux 5.1+):** Modern async I/O — submit operations to ring buffer, kernel processes them, completions appear in completion ring. Near zero-copy, zero-syscall for batch operations. Used by PostgreSQL, Nginx, io_uring-backed runtimes.

**Models:** Blocking → Non-blocking polling → Select/poll → Epoll → io_uring (least to most efficient for high concurrency).

---

### Q102. What is epoll and why is it better than select/poll?

**Topic Mapping:** OS → I/O Management → Event Notification

**Explanation:**

**select():** Pass array of FDs → kernel scans all → returns which are ready. O(N) per call. Max FDs: 1024 (FD_SETSIZE).

**poll():** Similar to select but no FD limit. Still O(N) — kernel scans all FDs each call.

**epoll (Linux 2.6):** 
1. `epoll_create()` → create epoll instance
2. `epoll_ctl(ADD)` → register FD to watch (O(1) with red-black tree internally)
3. `epoll_wait()` → blocks until some FD is ready → returns ONLY ready FDs

**Key advantage:** O(1) per ready event (vs O(N) total for select/poll). For 10,000 connections with 10 active: epoll returns 10 events, not 10,000.

**Level-triggered vs Edge-triggered:**
- LT (default): notify while data available (keep notifying until handled)
- ET: notify only on new data arrival (must read all data at once or miss it)

**Used by:** Nginx, Node.js (libuv), Redis, all high-performance servers.

**Interview Tip:** "C10K problem" (10,000 concurrent connections) motivated epoll's creation. Classic interview question for backend/networking roles.

---

### Q103. What is a semaphore vs condition variable?

**Topic Mapping:** OS → Synchronization → Semaphore vs Condition Variable

**Explanation:**

**Semaphore:**
- Integer counter
- `wait()` decrements, blocks if 0
- `signal()` increments, wakes waiter
- Can be posted from any context (including interrupt handlers, signal handlers)
- No notion of "associated mutex"
- Value persists (signal before wait = wait succeeds immediately)

**Condition Variable:**
- Always used WITH a mutex
- `wait(cv, mutex)`: atomically releases mutex and sleeps
- `signal(cv)`: wakes one waiter (which re-acquires mutex)
- Signal without waiter = lost (no history)
- `pthread_cond_t` in POSIX

**When to use which:**
- Signaling across unrelated components → semaphore
- Waiting for a condition within a critical section → condition variable
- Producer-consumer: either works, but CV with mutex = safer for complex conditions

**Spurious wakeups:** Condition variables can wake without `signal()` being called (OS artifact). Always use `while` loop:
```c
while (!condition) pthread_cond_wait(&cv, &mutex);
```

---

### Q104. What is a lock-free data structure?

**Topic Mapping:** OS → Synchronization → Lock-Free Programming

**Explanation:**
Lock-free data structures use atomic operations (CAS — Compare-And-Swap) instead of locks. At least one thread always makes progress (no deadlock possible).

**CAS (Compare-And-Swap):**
```c
bool CAS(int *addr, int expected, int new_val) {
    if (*addr == expected) {
        *addr = new_val;
        return true;
    }
    return false;
}
```

**Lock-free stack (push):**
```c
void push(Node *node) {
    do {
        node->next = top;  // read current top
    } while (!CAS(&top, node->next, node));  // retry if top changed
}
```

**ABA Problem:** top was A, changed to B, changed back to A. CAS sees A → succeeds, but state has changed! Solution: version counter (ABA → A1B2A3, CAS checks version too).

**Wait-free:** Even stronger guarantee — every thread completes in bounded steps. Lock-free only guarantees SOME thread progresses.

**Used in:** Java `java.util.concurrent`, Linux kernel (RCU), high-performance databases.

---

### Q105. What is the difference between soft links and mounts?

**Topic Mapping:** OS → File Systems → Mounts

**Explanation:**
**Symlink (soft link):** File whose content is a path to target. Resolved at each access.

**Mount:** Attaches a filesystem at a mount point. The mount point directory's original content is hidden; the mounted filesystem's root is visible instead.

**Mount types:**
```bash
mount /dev/sdb1 /mnt/disk          # mount block device
mount -t tmpfs tmpfs /tmp           # tmpfs (RAM-backed filesystem)
mount --bind /real/dir /fake/dir    # bind mount — same dir, two paths
mount -t nfs server:/share /mnt     # NFS (network filesystem)
```

**bind mount:** Makes a directory accessible at another path. Used extensively in Docker (container volumes = bind mounts).

**Namespace mounts (Linux):** Each process can have its own mount namespace. Containers get their own filesystem view. `unshare -m bash` creates new mount namespace.

**`/proc/mounts`** or `mount` command shows all active mounts.

---

### Q106. What is cgroup (control group)?

**Topic Mapping:** OS → Virtualization → cgroups

**Explanation:**
cgroups (Linux kernel feature) limit and account for resource usage (CPU, memory, I/O, network) for groups of processes.

**Subsystems (controllers):**
- `cpu`: CPU shares, quotas
- `memory`: memory limits, OOM behavior
- `blkio`: disk I/O throttling
- `net_cls`: network packet tagging
- `devices`: device access control

**Usage:**
```bash
# Create cgroup, set memory limit to 512MB, add process
mkdir /sys/fs/cgroup/memory/mygroup
echo 536870912 > /sys/fs/cgroup/memory/mygroup/memory.limit_in_bytes
echo $PID > /sys/fs/cgroup/memory/mygroup/tasks
```

**Docker = namespaces + cgroups + overlay filesystem**
- cgroups: resource limits per container
- namespaces: isolation (PID, network, mount, IPC, UTS, user)
- overlay fs: layered filesystem for container images

**cgroups v2 (unified hierarchy):** Single hierarchy, thread-aware, better support for nested limits. Kubernetes now requires cgroupsv2.

---

### Q107. What is a namespace in Linux?

**Topic Mapping:** OS → Virtualization → Linux Namespaces

**Explanation:**
Namespaces isolate specific kernel resources for a group of processes. Each process sees its own view of the system.

| Namespace | Isolates |
|-----------|----------|
| PID | Process IDs (container has PID 1 as its init) |
| Network | Network interfaces, routing, ports |
| Mount | Filesystem tree |
| IPC | System V IPC, POSIX message queues |
| UTS | Hostname, domain name |
| User | UIDs/GIDs (root in container ≠ root on host) |
| Time | System time (Linux 5.6+) |

**Creating namespace:**
```bash
unshare --pid --fork --mount-proc bash  # new PID namespace
```

**Docker containers:** Each container gets its own set of namespaces → isolated from host and other containers.

**Interview Tip:** Containers are NOT virtual machines. VMs emulate hardware. Containers are processes with namespace isolation + cgroup resource limits. Much lighter weight.

---

### Q108. What is the difference between process scheduling and I/O scheduling?

**Topic Mapping:** OS → Scheduling → I/O Scheduling

**Explanation:**
**Process/CPU Scheduling:** Decides which ready process gets the CPU. Goal: fair CPU utilization, good response time.

**I/O Scheduling (Block I/O scheduler):** Decides order of disk I/O requests. Goal: minimize seek time (for HDDs), maximize throughput.

**Linux I/O schedulers:**
- **CFQ (Completely Fair Queuing):** Per-process queues, fair I/O bandwidth. Good for desktops.
- **Deadline:** Ensures requests complete by deadline. Good for databases.
- **NOOP:** Simple FIFO merge. Best for SSDs (no seek optimization needed).
- **BFQ (Budget Fair Queuing):** Modern default, interactive-friendly.
- **mq-deadline:** Multi-queue variant for NVMe.

**SSD vs HDD:**
- HDD: Seek time matters → SCAN/C-SCAN
- SSD: No physical seek → NOOP or none (NVMe has hardware queuing)

```bash
cat /sys/block/sda/queue/scheduler  # see current scheduler
echo mq-deadline > /sys/block/nvme0n1/queue/scheduler  # change it
```

---

### Q109. What is a read-write lock and when should you use it?

**Topic Mapping:** OS → Synchronization → Read-Write Locks

**Explanation:**
Read-write lock (rwlock) allows:
- Multiple concurrent readers (read lock = shared)
- Only one writer at a time, exclusive of all readers (write lock = exclusive)

**When to use:** Data structure read frequently, written rarely. E.g., routing table, configuration data, caches.

```c
pthread_rwlock_t rwlock;

// Reader:
pthread_rwlock_rdlock(&rwlock);
// read data
pthread_rwlock_unlock(&rwlock);

// Writer:
pthread_rwlock_wrlock(&rwlock);
// modify data
pthread_rwlock_unlock(&rwlock);
```

**Writer starvation:** If readers keep arriving, writers never get access. Most implementations prioritize pending writers (new readers wait if writer waiting).

**vs Mutex:** Mutex = exclusive always. rwlock = shared for reads. rwlock has overhead (more complex state). Only faster than mutex if reads significantly outnumber writes.

**Java:** `ReentrantReadWriteLock`. Linux kernel: `rwlock_t` (spin) and `rw_semaphore` (sleeping).

---

### Q110. What is the difference between preemptive and non-preemptive kernels in terms of race conditions?

**Topic Mapping:** OS → Kernel → Preemption and Race Conditions

**Explanation:**
**Non-preemptible kernel:**
- Process running in kernel mode cannot be preempted
- Only one CPU runs kernel code at a time (on uniprocessor)
- On uniprocessor: no need for locks in kernel (only need to disable interrupts)
- Simpler but poor latency

**Preemptible kernel:**
- Kernel code can be interrupted at almost any point
- A higher-priority process can preempt current process even in kernel mode
- Race conditions in kernel data structures are possible
- Need spinlocks even in process context
- Better latency, more complex

**SMP (Symmetric Multiprocessing):**
- Multiple CPUs run kernel code simultaneously
- Even non-preemptible kernel needs locks for SMP
- `per_cpu` variables: separate copy per CPU → no sharing → no locks needed for per-CPU data

**Key insight:** Preemptibility × SMP = maximum complexity. Linux kernel has extensive locking infrastructure to handle both.

---

### Q111–Q130 (OS Intermediate — Final Block)

### Q111. What is virtual memory and how does it enable memory isolation?

**Answer Summary:** Each process has its own virtual address space (e.g., 0 to 2^48 bytes). Different processes' virtual addresses map to different physical addresses via their own page tables. Process A cannot access Process B's memory even if they have the same virtual address — different page tables → different physical pages. OS enforces this by loading each process's page table base (CR3 on x86) when scheduling it.

---

### Q112. What is the "Thundering Herd" problem?

**Answer Summary:** When many threads/processes wait on the same resource (condition variable, accept() on socket), and the resource becomes available, ALL waiters are woken simultaneously. Only one can proceed; the rest go back to sleep — wasted context switches.

**Fix:** Use `notify()` instead of `notifyAll()` when only one thread can proceed. Linux kernel uses `accept4()` with `SOCK_NONBLOCK` and exclusive wake-up (SO_REUSEPORT).

Nginx avoids thundering herd with accept mutex (only one worker accepts at a time) or SO_REUSEPORT (kernel-side load balancing).

---

### Q113. What is the difference between heap allocation and stack allocation?

**Answer Summary:**

| Feature | Stack | Heap |
|---------|-------|------|
| Allocation | Automatic (compiler) | Manual (malloc/new) |
| Speed | O(1) — just move stack pointer | Slower (find free block) |
| Size | Limited (~8MB) | Large (virtual address space) |
| Lifetime | Tied to scope/function | Programmer-controlled |
| Fragmentation | None | Yes |
| Thread safety | Per-thread stack (safe) | Shared (need synchronization) |

Stack overflow = stack reaches guard page → SIGSEGV. Heap exhaustion = malloc returns NULL.

---

### Q114. What is memory leak and how do you detect it?

**Answer Summary:** Memory leak = allocating heap memory without freeing it. The process's memory usage grows unboundedly. 

Detection tools:
- **Valgrind:** Instrument every malloc/free. Reports unfreed allocations at exit. Slow (10-50x).
- **AddressSanitizer (ASan):** Compiler instrumentation. Detects leaks, buffer overflows, use-after-free. ~2x slowdown.
- `/proc/[PID]/status`: watch `VmRSS` (resident memory) grow over time.
- Language GC (Java, Go, Python): no manual memory management → no memory leaks (but GC pauses).

---

### Q115. What is OOM Killer's oom_score algorithm?

**Answer Summary:** Linux OOM killer scores each process 0-1000. Score factors:
1. Resident memory (RSS): larger = higher score
2. Page table overhead
3. Swap usage
4. Child RSS (shared memory)
5. `oom_score_adj` (-1000 to +1000): user-adjustable offset

Process with highest score gets killed. System processes (root-owned) get small bonus. `/proc/[PID]/oom_score` shows current score.

---

### Q116. Explain the fork bomb.

**Answer Summary:** `:(){ :|:& };:` (bash fork bomb): function that calls itself twice in background, both call themselves, exponential process creation. Within seconds, PID table exhausted, system unresponsive.

**Defense:** `ulimit -u 100` (max 100 processes per user). cgroups pids controller. Modern Linux: systemd-level process limits per user (`UserTasksMax` in `/etc/systemd/logind.conf`).

---

### Q117. What is a barrier in parallel programming?

**Answer Summary:** A synchronization point where all threads must arrive before any can proceed. Used to synchronize phases in parallel algorithms.

```c
pthread_barrier_t barrier;
pthread_barrier_init(&barrier, NULL, 4);  // 4 threads must arrive

// Each thread:
// ... do phase 1 work ...
pthread_barrier_wait(&barrier);  // wait for all 4 threads
// ... all threads proceed to phase 2 ...
```

Used in: OpenMP parallel loops, MPI distributed computing, GPU kernel launches.

---

### Q118. What is the difference between a trap and an interrupt?

**Answer Summary:**
- **Interrupt:** Asynchronous — caused by external event (hardware). CPU checks after each instruction. Can occur at any time.
- **Trap (Exception):** Synchronous — caused by current instruction (divide by zero, page fault, system call). Occurs at specific instruction.
- **Fault:** Exception that can be corrected and instruction restarted (page fault).
- **Abort:** Non-recoverable hardware error.

System calls use `SYSCALL` instruction = software trap. Division by zero = fault. Page fault = fault (OS handles, restarts instruction).

---

### Q119. What is instruction-level parallelism and out-of-order execution?

**Answer Summary:** Modern CPUs don't execute instructions in program order. The CPU reorders instructions to avoid stalls (data dependencies, cache misses). Results appear in program order (in-order commit/retirement). This is transparent to single-threaded programs.

**Impact on OS/synchronization:** On multiprocessor, out-of-order execution + weak memory models can cause other CPUs to see memory operations in different order → need memory barriers (`mfence`, `sfence`, `lfence` on x86).

This is why locks in multi-threaded code need memory barriers — to ensure all CPUs see consistent view of memory.

---

### Q120. What is a tickless kernel?

**Answer Summary:** Traditional Linux kernel: timer tick every 1ms (HZ=1000) → wakes CPU, runs scheduler, updates jiffies. Even idle CPU gets 1000 interrupts/second.

Tickless kernel (NOHZ, Linux 3.x+): On idle CPUs, timer is stopped. CPU sleeps until next event (interrupt, timer). Saves power (critical for laptops, mobile, cloud VMs). Also "full tickless": even running CPUs can suppress periodic ticks if only one task running.

Benefits: Better battery life, better performance in VMs (no spurious VM exits for timer ticks).

---

### Q121–Q130 (OS Intermediate — Last 10)

### Q121. What is a memory barrier/fence?
**Answer Summary:** A CPU instruction that ensures all memory operations before the barrier complete before any after it. Prevents CPU/compiler reordering. Types: LoadLoad, StoreStore, LoadStore, StoreLoad (full barrier). `__sync_synchronize()` in GCC. `std::atomic_thread_fence()` in C++11. Critical in lock implementations and lock-free programming.

### Q122. What is huge page support in Linux for databases?
**Answer Summary:** Oracle, PostgreSQL, MongoDB configure huge pages (2MB) for their buffer pools. Less TLB pressure for large datasets. Configured via `/proc/sys/vm/nr_hugepages`. MySQL InnoDB buffer pool benefits significantly. Kubernetes: `hugepages-2Mi` resource request in pod spec.

### Q123. What is the process of booting a Linux system?
**Answer Summary:** BIOS/UEFI → GRUB bootloader → loads kernel (vmlinuz) + initramfs into memory → kernel initializes hardware → mounts initramfs (temporary root) → runs `/init` → mounts real root filesystem → executes `/sbin/init` (PID 1) → systemd starts all services → multi-user target reached.

### Q124. What is loadavg in Linux?
**Answer Summary:** `load average: 1.23, 0.87, 0.91` = exponential moving average of runnable + uninterruptible process count over 1, 5, 15 minutes. `loadavg ≤ CPU count` = healthy. `loadavg > CPU count` = overloaded. Includes processes in D state (disk I/O wait) — important distinction from Windows.

### Q125. What is the difference between process memory and working set?
**Answer Summary:** Process memory = total virtual address space size. RSS (Resident Set Size) = pages currently in physical memory. Working Set = pages actively used in recent time window. RSS > Working Set (includes cold pages). Working Set ≈ frames needed to avoid thrashing. `top` shows VIRT (virtual), RES (RSS), SHR (shared).

### Q126. What is inter-processor interrupt (IPI)?
**Answer Summary:** Signal sent from one CPU to another via APIC (Advanced Programmable Interrupt Controller). Used for: TLB shootdown (invalidate TLB on all CPUs after page table update), scheduler IPI (tell CPU to reschedule), remote function call. Critical in multiprocessor systems. Heavy TLB shootdown workloads can significantly degrade multi-core performance.

### Q127. What is the difference between kernel threads and user threads?
**Answer Summary (extended):** Kernel threads: OS-managed, scheduled by OS, true parallelism on SMP, system call in one thread doesn't block others. User threads: managed by library, OS sees single thread per process, custom scheduling (can be cooperative), lighter weight. Modern: Go goroutines (user-space, M:N with kernel threads), Java threads (OS threads since Java 1.4).

### Q128. What is the maximum number of processes in Linux?
**Answer Summary:** Limited by PID namespace. Default PID max: 32768 (`/proc/sys/kernel/pid_max`). Can be raised to 4M in 64-bit. Each process requires: PCB (kernel memory), kernel stack (8KB), page tables, VMAs. System memory is the practical limit. `ulimit -u` limits per-user. cgroups `pids.max` limits per-cgroup.

### Q129. What is a jiffie?
**Answer Summary:** Jiffy = unit of kernel time = 1/HZ seconds. HZ=1000 (most Linux configs) → 1 jiffy = 1ms. `jiffies` = global counter incremented each timer tick. Used throughout kernel for timeouts, scheduling quanta. `jiffies_to_msecs(j)` converts. With tickless kernel, `jiffies` updated lazily from hardware clock.

### Q130. What is the Completely Unambiguous Scheduler (CFS) limitation?
**Answer Summary:** CFS works well for general workloads but has issues:
1. Real-time tasks need SCHED_FIFO or SCHED_RR (bypass CFS entirely)
2. Large number of processes → red-black tree operations add latency
3. VMs: "CPU steal" (hypervisor steals CPU time) not visible to CFS
4. NUMA-unaware (later improved with NUMA-aware scheduling)
5. cgroup CPU bandwidth control needs careful tuning to avoid latency


---

## ADVANCED QUESTIONS — OS (Q131–Q200)

### Q131. How does Linux implement NUMA-aware scheduling?
**Topic Mapping:** OS → Advanced Scheduling → NUMA
**Answer:** NUMA (Non-Uniform Memory Access): each CPU socket has local RAM (~100ns) vs remote RAM (~300ns). Linux NUMA balancing: kernel periodically marks pages PROT_NONE. On next access → page fault → OS checks if faulting CPU's NUMA node differs from page's node → if yes, migrate page or task. `numactl --hardware` shows topology. `numactl --membind=0 --cpunodebind=0 ./app` pins app to node 0. Impact: 2–3× performance degradation on 2-socket server if ignored. PostgreSQL, MySQL, JVM all benefit from NUMA pinning.

### Q132. What is the Linux VFS (Virtual Filesystem Switch)?
**Topic Mapping:** OS → File Systems → VFS
**Answer:** VFS is an abstraction layer providing a uniform interface for all filesystem types. Objects: superblock (mounted FS metadata), inode (file metadata + ops vtable), dentry (path component → inode cache), file (per-open-file state). Each object has an operations table (function pointers). Path resolution: root inode → dcache lookup each component → inode → create file object → return fd. dcache makes path resolution O(1) via hash table. Without dcache, every stat() would need multiple disk reads.

### Q133. What is io_uring and how does it achieve near-zero syscall overhead?
**Topic Mapping:** OS → I/O → io_uring
**Answer:** io_uring (Linux 5.1, 2019): two ring buffers mmap'd into user space — Submission Queue (SQ) and Completion Queue (CQ). User writes SQEs to SQ, kernel reads and processes them, writes CQEs to CQ. SQPOLL mode: kernel thread polls SQ → zero syscalls for submission. User polls CQ for results → zero syscalls for completion. Near-zero overhead for high-throughput I/O. PostgreSQL, RocksDB, Nginx adopting io_uring.

### Q134. HugeTLB vs THP — compare and when does each fail?
**Topic Mapping:** OS → Memory Management → Huge Pages
**Answer:** Standard page = 4KB. 2MB huge pages: TLB covers 1GB instead of 2MB → dramatic TLB pressure reduction. HugeTLB (explicit): reserved at boot, guaranteed, non-swappable, requires MAP_HUGETLB in app. THP (Transparent): khugepaged daemon auto-promotes 512 contiguous 4KB pages → transparent but causes latency spikes during promotion. MongoDB and Redis disable THP: `echo never > /sys/kernel/mm/transparent_hugepage/enabled`. Oracle DB and JVM use explicit HugeTLB for buffer pools.

### Q135. What are Linux scheduling domains?
**Topic Mapping:** OS → Scheduling → Scheduling Domains
**Answer:** CPUs organized in hierarchy: SMT (hyper-threads on same core) → MC (cores on same die, share L3) → NUMA (cross-socket). Load balancing frequency decreases up the hierarchy: SMT = very frequent (cheap), MC = moderate, NUMA = rare (expensive cross-socket migration). Task migrates only if imbalance cost > migration cost. `taskset -c 0-3` pins to specific CPUs. `/sys/devices/system/cpu/cpu0/topology/` exposes topology info.

### Q136. How does the Linux buddy system work for physical page allocation?
**Topic Mapping:** OS → Memory Management → Buddy System
**Answer:** Free memory split into blocks of 2^k pages (k=0 to 10). Each size has a free list. Allocation: find smallest 2^k ≥ request; if unavailable, split next larger block into two "buddies", use one, add other to smaller list. Freeing: if buddy also free, merge into larger block (recursively). Buddy address = block_addr XOR (1 << k). Eliminates external fragmentation of physical pages. Slab allocator sits on top for sub-page kernel object allocation. Zone allocator: ZONE_DMA (0–16MB), ZONE_NORMAL, ZONE_HIGHMEM.

### Q137. Walk through the kernel code path for a COW page fault after fork().
**Topic Mapping:** OS → Memory Management → COW Implementation
**Answer:** After fork(): child's PTEs set to read-only (same physical frames as parent, COW-flagged). Child writes → protection fault → MMU raises exception → `do_page_fault()` → checks VMA: write to read-only COW page → `do_cow_fault()`: alloc_page() from buddy, copy old page content, update child's PTE to new frame (writable), decrement old page ref count. If old page ref=1 (only parent): mark parent's PTE writable too. TLB invalidated. Faulting instruction restarted (not resumed). Key: instruction is RESTARTED because it never completed.

### Q138. How does the Linux OOM killer select its victim?
**Topic Mapping:** OS → Memory Management → OOM Killer
**Answer:** When all memory + swap exhausted: `out_of_memory()` called. Score per process: oom_badness = (RSS + swap_pages + page_table_pages) adjusted by oom_score_adj (-1000 to +1000). Highest score = killed. PID 1 (init) protected (adj = -1000). Kubernetes: BestEffort pods get adj=1000 (killed first), Guaranteed pods get adj=-998 (protected). Check: `dmesg | grep -i oom`. Protect critical process: `echo -1000 > /proc/$(pidof sshd)/oom_score_adj`.

### Q139. What does PREEMPT_RT add to Linux and what latency does it achieve?
**Topic Mapping:** OS → Real-Time → PREEMPT_RT
**Answer:** Standard Linux worst-case latency: 10–50ms (non-preemptible kernel sections). PREEMPT_RT converts: spinlocks → sleeping mutexes (higher-priority tasks can preempt spinlock holders), IRQ handlers → preemptible kernel threads, raw spinlocks remain for truly critical sections. Result: worst-case latency 50–200μs. Used for: robotics, audio production, industrial control. Test with `cyclictest`. NOT suitable for hard real-time (pacemakers, avionics). Partially merged into mainline Linux 5.15+.

### Q140. How does the Linux kernel protect itself from user-space exploits?
**Topic Mapping:** OS → Security → Kernel Self-Protection
**Answer:** SMEP: CPU refuses to execute user-space pages in kernel mode. SMAP: CPU blocks kernel from reading user-space memory unless explicitly enabled (STAC/CLAC). KASLR: kernel loaded at random address each boot. Stack canaries: random value before return address, checked on return. KPTI (post-Meltdown): separate page tables for user/kernel mode — kernel mappings absent in user-mode page tables; 5–30% overhead on syscall-heavy workloads. CFI: only valid function entry points as jump targets. Module signing: prevent loading unsigned kernel modules.

### Q141. How does mmap interact with the Linux page cache?
**Topic Mapping:** OS → Memory Management → mmap + Page Cache
**Answer:** mmap(fd) → allocates VMA, no pages loaded. First access → page fault → OS checks page cache: file page already cached? Yes → map existing physical page into process PTE (zero copy, zero disk I/O). No → load from disk into page cache → map into PTE. Multiple processes mmap same file → same physical pages, shared automatically. vs read(): data goes page cache → kernel buffer → user buffer (2 copies). mmap = page cache directly visible in user address space (0 copies). Dirty pages written back by pdflush. msync() forces immediate writeback. Databases debate mmap vs read() for buffer management — mmap loses control of eviction policy.

### Q142. How does Linux `perf` use hardware PMU counters?
**Topic Mapping:** OS → Performance → Profiling
**Answer:** Modern CPUs have 4–8 hardware counters (PMU) counting: instructions retired, CPU cycles, L1/L2/L3 cache misses, branch mispredictions, TLB misses. `perf_event_open()` syscall configures counters. On overflow → NMI fires → kernel captures PC + call stack → sample recorded. `perf stat` counts events; `perf record -g` samples with stacks; `perf report` shows hottest functions. CPI (cycles per instruction) < 1 = good; > 2 = cache/memory bottleneck. Flame graphs: `perf script | stackcollapse-perf.pl | flamegraph.pl > out.svg`.

### Q143. What is Meltdown/Spectre and how does the OS mitigate them?
**Topic Mapping:** OS → Security → CPU Vulnerabilities
**Answer:** Meltdown (Intel): out-of-order execution allows user-mode reads of kernel memory before privilege check; cache timing leaks data. Mitigation: KPTI — remove kernel mappings from user-mode page tables; every syscall = page table switch → 5–30% overhead. Spectre (all CPUs): tricks branch predictor to speculatively execute code leaking secrets via cache side-channel. Mitigations: Retpoline (replace indirect jumps with return trampoline), IBRS/IBPB microcode updates, array index masking. Net overhead: 5–30% for syscall-heavy workloads like databases and web servers.

### Q144. How does Linux safely copy data between user and kernel space?
**Topic Mapping:** OS → Memory Management → User-Kernel Transfer
**Answer:** Direct kernel deref of user pointer is dangerous: pointer may be invalid, swapped out, or pointing to kernel space (security hole). Linux provides: `copy_from_user(kernel_buf, user_ptr, size)` and `copy_to_user(user_ptr, kernel_buf, size)`. These: validate pointer is in user address space, enable SMAP (allow kernel to access user space temporarily), perform copy (may page fault mid-copy → handled), disable SMAP, return bytes not copied. SMAP prevents: attacker crafting user pointer to kernel exploit code. For DMA: pin user pages → give physical address to DMA controller → hardware writes directly to user buffer (true zero-copy).

### Q145. What is eBPF and how does it safely run user code in the kernel?
**Topic Mapping:** OS → Advanced → eBPF
**Answer:** eBPF: user-defined programs run safely in Linux kernel without kernel module or source changes. Safety: kernel verifier checks bytecode: no infinite loops (bounded loops only), all pointer dereferences valid, no illegal ops. JIT-compiled to native code → near-native speed. Attachment points: network (XDP, tc), tracing (kprobes, uprobes, tracepoints), security (LSM hooks). eBPF maps: key-value stores shared between kernel eBPF and user space. Use cases: Cilium (K8s networking), Katran (Facebook LB), bpftrace/bcc (observability), Falco (security), Datadog agent. Replaces kernel modules for many tracing/networking use cases safely.

### Q146. What is a kernel crash dump and kdump?
**Topic Mapping:** OS → Reliability → Crash Dump
**Answer:** On kernel panic (null deref in kernel, hardware error, corrupted memory): system can dump memory for post-mortem analysis. kdump: secondary "capture kernel" preloaded at boot in reserved memory. On panic: kexec transfers control to capture kernel (no hardware reset). Capture kernel reads primary kernel's memory via /dev/crash → writes vmcore file to disk. `crash` tool analyzes vmcore: backtrace, register state, variable inspection. `dmesg` and `/var/log/kdump/` for analysis. Essential for production incident investigation.

### Q147. What is CPU affinity and why does it matter for performance?
**Topic Mapping:** OS → Scheduling → CPU Affinity
**Answer:** CPU affinity pins process/thread to specific cores. Benefits: (1) Cache warmth — always on same core → L1/L2 always hot. (2) NUMA locality — pin to same NUMA node as memory → no remote access. (3) Predictability — real-time systems need deterministic CPU access. (4) Avoid false sharing — pin threads to different cores. `taskset -c 0-3 ./app`. Java: JNA + sched_setaffinity. Go: `runtime.LockOSThread()` + syscall. Kubernetes: Guaranteed QoS pods get exclusive CPUs via CPU Manager policy = static.

### Q148. What are hardirq and softirq (top-half and bottom-half)?
**Topic Mapping:** OS → I/O → Interrupt Handling
**Answer:** Hardware IRQ arrives → hardirq (top half): interrupts disabled, must be very fast (microseconds), only acknowledges hardware and queues work. Softirq (bottom half): deferred work after hardirq returns, interrupts re-enabled, handles bulk processing. Types: NET_RX_SOFTIRQ (network packet processing), TIMER_SOFTIRQ, TASKLET_SOFTIRQ. Tasklets: serialized per-CPU softirq work, cannot run concurrently with themselves. Workqueues: can sleep (unlike softirqs), run in kernel thread context, for work needing blocking I/O. Two-level design keeps interrupt latency low while handling bulk work efficiently.

### Q149. How does Linux cgroup CPU scheduling work?
**Topic Mapping:** OS → Scheduling → cgroup CPU
**Answer:** cgroup CPU controller creates hierarchy of scheduling groups. CFS enforces fair share at each level. Example: two users each with 50% CPU share — User A (10 processes) and User B (1 process). Without groups: each process gets 1/11. With groups: each user gets 50%, User A's processes split their 50% (5% each). `cpu.shares`/`cpu.weight` = relative weight. `cpu.cfs_quota_us` / `cpu.cfs_period_us` = hard CPU bandwidth limit (Kubernetes `resources.limits.cpu` → quota). Throttling: process exhausts quota before period ends → TASK_THROTTLED until next period.

### Q150. What is memory hotplug in Linux?
**Topic Mapping:** OS → Memory Management → Hotplug
**Answer:** Servers/cloud VMs support adding/removing RAM at runtime. Adding: memory registered in sections (128MB+), added to buddy allocator, drivers notified. Removing (harder): must migrate all pages off the section (only possible for ZONE_MOVABLE pages), offline section, remove from buddy. Challenge: kernel static data (page tables, module text) lives in non-movable memory. ZONE_MOVABLE: reserve top of memory for movable pages only → enables clean hotremove. Used by: cloud hypervisors (memory balloon), server memory replacement without downtime.

### Q151. What is Linux's memory reclaim mechanism (kswapd)?
**Topic Mapping:** OS → Memory Management → Memory Reclaim
**Answer:** kswapd: kernel thread that reclaims memory when free memory drops below `low` watermark. Three watermarks per zone: min, low, high. Below low: kswapd wakes and reclaims. Below min: direct reclaim (allocating process itself reclaims). Reclaim targets: clean page cache pages (just drop — can reload from disk), dirty page cache pages (write to disk first then drop), anonymous pages → swap (write to swap space). LRU lists: active_anon, inactive_anon, active_file, inactive_file. Pages move from active to inactive (second-chance) before eviction. `vm.swappiness=0` → prefer reclaiming file pages over anonymous.

### Q152. How do rootkits intercept syscalls and how does the kernel defend?
**Topic Mapping:** OS → Security → Rootkits
**Answer:** Rootkit technique: load kernel module → overwrite syscall table entry (e.g., sys_getdents → malicious_getdents that hides files/processes). Kernel defenses: `CONFIG_STRICT_KERNEL_RWX` — syscall table in read-only memory → cannot overwrite. Module signing: rootkit module won't load on Secure Boot systems. Kprobes: legitimate kernel function hooking via breakpoints (read-only, kernel-approved). Detection: compare syscall table entries to expected kernel symbols. Tools: rootkit hunter, unhide. Modern defense: hardware-enforced kernel integrity (Intel CET, ARM Pointer Authentication).

### Q153. What is false sharing and how do you prevent it?
**Topic Mapping:** OS → Synchronization → False Sharing
**Answer:** Cache line = 64 bytes. Two threads on different CPUs each access different variables that share the same cache line. One thread writes → cache line invalidated on all CPUs → other thread must reload the entire line even though its variable didn't change. Net effect: no logical sharing but full cache-coherence overhead. Prevention: pad structs to cache line boundary: `__attribute__((aligned(64)))` or `alignas(64)` in C++. Java: `@Contended` annotation. Linux kernel: `DEFINE_PER_CPU` for per-CPU counters — each CPU has its own copy in its own cache line.

### Q154. What is the Linux tickless kernel (NOHZ)?
**Topic Mapping:** OS → Power Management → Tickless Kernel
**Answer:** Traditional: timer fires 1000×/sec (HZ=1000) → CPU wakes even when idle → cannot enter deep C-states. NOHZ: idle CPU stops timer → CPU enters deep sleep until next event (interrupt, expiring timer). Power savings: 10–30% on idle/lightly loaded systems. Critical for laptops, mobile, cloud (billing per CPU-time). NOHZ_FULL: even busy CPUs (with single runnable task) skip periodic ticks — HPC and real-time use case. `/proc/timer_list` shows next timer events. `nohz_full=1-7` kernel parameter enables full tickless on those CPUs.

### Q155. How does the kernel prevent stack overflow?
**Topic Mapping:** OS → Memory Protection → Stack Overflow
**Answer:** Kernel stack = 8KB or 16KB — very small. Prevention: (1) Stack canary (CONFIG_STACKPROTECTOR): random value at bottom of stack, checked on function return — overflow detected before worse damage. (2) Guard page: virtual page below stack with no permissions → overflow → page fault → panic. (3) CONFIG_VMAP_STACK: kernel stack backed by vmalloc with guard pages on both sides (since Linux 4.9). (4) Shadow Call Stack (ARM): separate stack for return addresses only, hardware-protected. (5) Kernel coding discipline: avoid deep recursion, use iterative algorithms.

### Q156. Explain perf_event hardware counters and CPI analysis.
**Topic Mapping:** OS → Performance Analysis
**Answer:** CPI (Cycles Per Instruction) = cycles / instructions_retired. CPI < 1: good (CPU retiring >1 instruction/cycle via superscalar). CPI > 2: bottlenecked (cache misses, memory bandwidth, branch mispredictions). Diagnosis: `perf stat -e instructions,cycles,cache-misses,branch-misses ./app`. High cache-misses + high CPI → memory bottleneck → optimize data layout, add prefetch hints. High branch-misses → restructure code (sort data before processing). LLC-misses especially costly (40+ cycles each). DRAM access = 100ns ≈ 300 cycles stall at 3GHz.

### Q157. What is the Linux audit subsystem?
**Topic Mapping:** OS → Security → Audit
**Answer:** Linux audit (auditd) logs security-relevant events: file accesses, system calls, login events, privilege escalation. Kernel: audit_log_*() functions write to ring buffer. auditd daemon reads and logs. `auditctl -w /etc/passwd -p rwa -k passwd_watch` — watch file. `ausearch -k passwd_watch` — search logs. Used for: PCI-DSS/HIPAA compliance, intrusion detection, forensics. SELinux AVC denials logged via audit. `aureport --auth` shows authentication events. High-volume audit logging can impact performance; filter carefully.

### Q158. How does Linux coordinate CPU frequency with the scheduler?
**Topic Mapping:** OS → Power Management → CPU Frequency Scaling
**Answer:** CPUFreq governors control P-state (frequency) selection. `schedutil` (modern default): frequency proportional to CPU utilization from scheduler's perspective — updated on every scheduler tick. `performance`: always max frequency (for benchmarks/real-time). `powersave`: always min frequency. C-states: deeper sleep = more power saving = longer wakeup latency. C0=active, C6=deep sleep (voltage reduced, L3 flushed). Scheduler selects deepest C-state compatible with next expected wakeup. Intel P-state driver with hardware-managed frequency: CPU adjusts frequency autonomously within OS-set limits.

### Q159. What is the difference between hard and soft real-time systems?
**Topic Mapping:** OS → Real-Time Systems
**Answer:** Hard real-time: missing deadline = system failure. Safety-critical: airbag (must fire in <50ms), pacemaker, flight control. OS: bare-metal RTOS (FreeRTOS, VxWorks, QNX). Characteristics: deterministic scheduling, bounded interrupt latency, priority inheritance. Soft real-time: missing deadline = degraded experience, not failure. Video streaming (dropped frame), gaming (lag spike), phone calls. OS: Linux (soft), Linux+PREEMPT_RT (better). Firm real-time: results after deadline useless but not catastrophic. Financial trading: late trade = missed opportunity. Linux is soft real-time at best without PREEMPT_RT.

### Q160. What is vDSO (virtual Dynamic Shared Object)?
**Topic Mapping:** OS → Performance → vDSO
**Answer:** vDSO: kernel maps small read-only memory region into every process's address space containing kernel code callable without a syscall (no mode switch). Current use: `clock_gettime()`, `gettimeofday()`, `time()`. Kernel writes current time to vDSO memory on every tick. Process reads directly — no syscall, no context switch. Speedup: 10M time calls/sec without vDSO overhead. `strace` doesn't show vDSO calls (not real syscalls). `ldd /bin/ls | grep vdso` shows it mapped. Clock resolution limited to timer frequency without syscall, but sufficient for most use cases.

### Q161. What is address sanitizer (ASan) and how does it instrument memory?
**Topic Mapping:** OS → Debugging → ASan
**Answer:** ASan: compiler instrumentation detecting memory errors at runtime. Shadow memory: 1 byte of shadow tracks 8 bytes of app memory. Shadow byte 0 = all 8 bytes valid; 1–7 = first N bytes valid; negative = poisoned (freed, redzone, etc.). Every memory access instrumented: check shadow byte — invalid → print error + stack trace. Detects: heap buffer overflow, stack buffer overflow, use-after-free, use-after-return, global buffer overflow. `gcc -fsanitize=address -fno-omit-frame-pointer`. ~2× memory overhead, ~2× slowdown. Use in CI/testing, not production. LeakSanitizer integrated for memory leaks.

### Q162. What is KASAN (Kernel Address Sanitizer)?
**Topic Mapping:** OS → Debugging → KASAN
**Answer:** KASAN = ASan for the kernel. Detects: use-after-free, out-of-bounds, stack overflows in kernel code. `CONFIG_KASAN=y`. Software tag-based KASAN: every pointer tagged, every access checks tag. Shadow memory tracks valid/invalid kernel addresses. ~2–3× memory overhead, significant slowdown. Used only in kernel development/testing environments — not production. Finds subtle kernel memory bugs that cause hard-to-reproduce crashes. Alternative: KMSAN (Kernel Memory Sanitizer) detects uninitialized memory reads.

### Q163. What is the difference between cooperative and preemptive goroutine scheduling in Go?
**Topic Mapping:** OS → Language Runtimes → Go Scheduler
**Answer:** Go runtime has its own M:N scheduler (goroutines:OS threads). Pre-Go 1.14: cooperative — goroutines yield only at function calls, channel ops, syscalls. CPU-bound goroutine with no function calls → starves others. Go 1.14+: signal-based preemption — runtime sends SIGURG to OS thread → goroutine preempted at safe point. Go scheduler: G (goroutine), M (OS thread), P (logical processor = GOMAXPROCS). Work-stealing: idle P steals goroutines from busy P's queue. Goroutine creation: ~2KB stack (grows dynamically) vs 8MB OS thread stack.

### Q164. What is the Linux memory model and how does it affect lock-free code?
**Topic Mapping:** OS → Concurrency → Memory Model
**Answer:** x86 TSO (Total Store Order): strong model. Only StoreLoad can reorder (store buffer). ARM: weak model — many reorderings possible, explicit barriers needed for everything. Linux kernel abstracts: `smp_rmb()` (read barrier), `smp_wmb()` (write barrier), `smp_mb()` (full barrier) — expand to correct instructions per arch. For lock-free code: must use `READ_ONCE()` / `WRITE_ONCE()` to prevent compiler reordering AND appropriate smp_*mb() for CPU reordering. C++11: `std::atomic` with `memory_order_acquire`/`release`/`seq_cst`. Seq_cst = full fence both sides (expensive on ARM).

### Q165. What is transparent huge page failure in production?
**Topic Mapping:** OS → Memory → THP Production Issues
**Answer:** THP latency spike scenario: khugepaged finds 512 contiguous 4KB pages, merges them into 2MB huge page. Process accesses page during merge → stall. Or: existing huge page split back into 4KB pages under memory pressure → stall. MongoDB 3.x+: THP causes multi-second write latency spikes during compaction. Redis: khugepaged scanning causes periodic latency outliers. PostgreSQL: less affected but still recommends testing. Diagnosis: `grep AnonHugePages /proc/meminfo` growing → THP active. Fix: `echo never > /sys/kernel/mm/transparent_hugepage/enabled`. Cloud: many AMIs/container images now disable THP by default.

### Q166. What is the perf stat output and how to interpret it?
**Topic Mapping:** OS → Performance → perf stat
**Answer:**
```
perf stat ./app:
  10,000,000 instructions   # total instructions retired
   8,000,000 cycles          # total CPU cycles
       1.25  insn per cycle  # IPC (inverse of CPI) — good if >1
     500,000 cache-misses    # LLC (last-level cache) misses
      50,000 branch-misses   # mispredicted branches
```
IPC > 1 = superscalar execution (good). Cache-miss rate = cache-misses/instructions. > 1% = significant cache pressure. Branch miss rate = branch-misses/branches. > 5% = branch predictor struggling (sort data or use branchless code). Memory bandwidth: `perf stat -e mem-loads,mem-stores` to estimate.

### Q167. What is process pinning in real-time Linux?
**Topic Mapping:** OS → Real-Time → Process Pinning
**Answer:** Real-time process setup: (1) `SCHED_FIFO` or `SCHED_RR` policy — preempts CFS tasks. `chrt -f 99 ./app` sets priority 99. (2) CPU isolation: `isolcpus=2,3` kernel param removes CPUs 2,3 from general scheduler. Only explicitly pinned tasks run there. (3) Memory locking: `mlockall(MCL_CURRENT|MCL_FUTURE)` — locks all pages in RAM, no page faults during execution. (4) Disable IRQ balancing: pin IRQs away from RT CPUs. (5) Disable CPU frequency scaling: `cpufreq-set -g performance`. Together: deterministic sub-millisecond latency.

### Q168. Explain Linux memory zones and DMA constraints.
**Topic Mapping:** OS → Memory Management → Memory Zones
**Answer:** ZONE_DMA (0–16MB): legacy ISA DMA devices can only address first 16MB. ZONE_DMA32 (0–4GB): 32-bit DMA devices. ZONE_NORMAL: kernel directly mapped memory (on 64-bit: effectively all RAM). ZONE_HIGHMEM (32-bit only, >896MB): not permanently mapped in kernel. ZONE_MOVABLE: only movable pages → clean hotremove and compaction. Allocations specify zone via GFP flags: `GFP_DMA` → ZONE_DMA, `GFP_DMA32` → ZONE_DMA32, `GFP_KERNEL` → ZONE_NORMAL. Zone fallback: if preferred zone full, try next higher zone. `cat /proc/zoneinfo` shows per-zone stats.

### Q169. What is the difference between `volatile` and `std::atomic` in C++?
**Topic Mapping:** OS → Concurrency → Memory Ordering
**Answer:** `volatile` in C/C++: prevents compiler from optimizing away reads/writes (useful for memory-mapped I/O, signal handlers). Does NOT prevent CPU reordering. Does NOT ensure visibility across threads. Not sufficient for multi-threading synchronization. `std::atomic<int>`: prevents compiler reordering AND ensures CPU memory ordering (default: seq_cst = full fence). `atomic.load(memory_order_relaxed)`: no ordering guarantee but still atomic (no torn reads). `memory_order_acquire/release`: pairs for lock-free producer-consumer. Java `volatile`: stronger than C++ volatile — provides happens-before guarantee but not compound-operation atomicity.

### Q170. What is Linux kernel's RCU (Read-Copy-Update)?
**Topic Mapping:** OS → Synchronization → RCU
**Answer:** RCU: lock-free for readers, allows concurrent reads and occasional writes. Reader: `rcu_read_lock()` → read → `rcu_read_unlock()` — NO spinlock, NO memory barrier on x86 (free!). Writer: copy data → modify copy → `rcu_assign_pointer(ptr, new)` (atomic pointer swap) → `synchronize_rcu()` (waits for all current readers to finish their read-side critical section) → free old data. Grace period: time between pointer swap and safe free of old data. Used for: routing tables, dentry cache, network protocol lists, module list — all very read-heavy. Readers never blocked by writers. One of Linux's most important performance innovations.

### Q171. How does Linux handle huge workloads with epoll?
**Topic Mapping:** OS → I/O → epoll Internals
**Answer:** epoll internally uses: red-black tree to store registered FDs (O(log N) add/remove), ready list (linked list of ready FDs). `epoll_ctl(ADD)` → adds FD to rb-tree, registers callback with socket. When data arrives on socket → callback fires → adds FD to ready list. `epoll_wait()` → transfers ready list to user space → O(events_ready) not O(total_FDs). Edge-triggered (ET) mode: callback fires only on state change (new data arrival). Level-triggered (LT): callback fires while data available. ET requires non-blocking FDs and reading all available data. Nginx uses ET by default for performance.

### Q172. What is copy-on-write in Redis BGSAVE?
**Topic Mapping:** OS → Memory → COW in Practice
**Answer:** Redis `BGSAVE` (background save): `fork()` creates child process. Child sees entire Redis dataset (snapshot of memory at fork time) via COW. Child iterates memory, writes RDB file to disk. Parent continues serving client requests — writes to dataset cause COW: parent gets private copy of modified pages. Result: child's snapshot remains consistent (original data) while parent continues serving. Memory overhead = pages modified during BGSAVE (can be significant under heavy write load — 2× memory in worst case). `INFO persistence` shows `rdb_changes_since_last_save`. Actual COW pages tracked in `/proc/<pid>/status VmRSS`.

### Q173. What is the difference between SIGTERM and SIGKILL handling?
**Topic Mapping:** OS → Process Management → Signals
**Answer:** SIGTERM (15): polite termination request. Process can catch → run cleanup (flush buffers, close connections, delete temp files) → exit gracefully. Default handler: terminate. `kill <pid>` sends SIGTERM. SIGKILL (9): cannot be caught, blocked, or ignored. Kernel forcibly terminates process immediately. No cleanup. Resources freed by kernel. `kill -9 <pid>`. Usage: send SIGTERM first → wait 5-10 seconds → if still running → SIGKILL. SIGSTOP (19): cannot be caught/ignored, pauses process. SIGCONT (18): resumes stopped process. Graceful shutdown pattern: SIGTERM → drain connections → SIGKILL as fallback.

### Q174. Explain copy_to_user security implications.
**Topic Mapping:** OS → Security → Kernel-User Boundary
**Answer:** If kernel blindly dereferences user-provided pointers: (1) Pointer to kernel space → kernel reads its own memory for attacker. (2) Pointer to unmapped page → kernel page faults (bad in non-pageable kernel context). (3) TOCTOU: pointer valid at check time but changed by another thread before use. `copy_from_user()` validates pointer is in user address space before access. SMAP prevents accidental kernel access to user memory (must explicitly enable/disable). Access token approach: verify pointer once, pin page, use physical address. Spectre v1: bounds check on array index before user-provided index doesn't prevent speculative access — need masking.

### Q175. What is huge TLB impact on database performance?
**Topic Mapping:** OS → Memory → TLB and Databases
**Answer:** PostgreSQL shared_buffers = 32GB, standard 4KB pages: 32GB/4KB = 8M page table entries. TLB (512 entries) covers 512×4KB = 2MB — tiny fraction of buffer pool. Result: constant TLB misses for random I/O → each miss = page table walk = 4 memory accesses = ~40ns. With 2MB huge pages: TLB covers 512×2MB = 1GB. 32GB buffer pool = 32 TLB entries needed. TLB miss rate → near zero. Benchmark: 10–30% PostgreSQL throughput improvement with huge pages for memory-intensive workloads. Configure: `vm.nr_hugepages = 16384` (32GB / 2MB). `postgresql.conf: huge_pages = on`.

### Q176. What is PCI MSI/MSI-X and how does the OS handle it?
**Topic Mapping:** OS → Hardware → PCI Interrupts
**Answer:** Legacy PCI: shared IRQ lines (multiple devices share interrupt → OS polls each to find which device). MSI (Message Signaled Interrupts): device writes to a specific memory address to trigger interrupt → no shared IRQ lines → unambiguous source. MSI-X: up to 2048 independent MSI vectors per device. Modern NVMe SSDs: one MSI-X vector per queue (32+ queues) → each CPU core handles its own queue → no cross-CPU interrupt routing. NIC: separate MSI-X per RX/TX queue → RSS (Receive Side Scaling) distributes packets across CPUs via MSI-X. OS: `lspci -v` shows MSI-X capability. `cat /proc/interrupts` shows per-CPU interrupt counts.

### Q177. What is the slab allocator and why is it critical for kernel performance?
**Topic Mapping:** OS → Memory → Slab Allocator
**Answer:** Kernel frequently allocates/frees same-sized objects (inodes, dentries, task_struct, sk_buff). General allocator has overhead. Slab: pre-allocates "slabs" (pages from buddy) divided into fixed-size object slots for each type. Free object returned to slab (not buddy) — kept initialized (constructor already run). Three slab states: full (all used), partial (some free), empty (all free). Allocation: O(1) — pop from partial slab's free list. Benefits: (1) No fragmentation for fixed-size objects. (2) Cache-warm allocation (objects stay in CPU cache). (3) Constructor called once at allocation, not on every reuse. SLUB (modern Linux): reduced metadata, per-CPU caches, less locking.

### Q178. How does Linux implement process credentials and Linux capabilities?
**Topic Mapping:** OS → Security → Capabilities
**Answer:** Traditional Unix: binary root/non-root. Linux capabilities: break root into ~40 discrete capabilities. Key ones: CAP_NET_ADMIN (network config), CAP_SYS_ADMIN (broad admin, mount/ptrace/cgroups), CAP_NET_BIND_SERVICE (bind ports < 1024), CAP_CHOWN (change ownership), CAP_DAC_OVERRIDE (bypass file permissions), CAP_KILL (send signals to any process). Sets per-process: effective (what process can do now), permitted (max it can ever gain), inheritable (passed across exec), ambient (inherited without explicit allow). `setcap cap_net_bind_service+ep /usr/bin/nginx` — nginx binds port 80 without full root. `getcap /usr/bin/ping` — check capabilities. Containers: run with minimal capability set (drop CAP_SYS_ADMIN etc.) for security.

### Q179. What is zRAM and when is it beneficial?
**Topic Mapping:** OS → Memory → Compressed RAM
**Answer:** zRAM: virtual block device using compressed RAM as swap space. Instead of swapping to disk (ms latency): compress page (LZ4: ~1μs), store in zRAM. Typical compression ratio: 3–5×. Net effect: ~3–5× more usable RAM for the cost of CPU time. Used by: Android (primary swap mechanism since Android 4.4), ChromeOS, embedded Linux with no swap disk. Trade-off: CPU overhead for compression vs disk I/O savings — worth it when disk is slow (eMMC, HDD) or absent. Configuration: `modprobe zram; echo 4G > /sys/block/zram0/disksize; mkswap /dev/zram0; swapon /dev/zram0`.

### Q180. What is the Linux OOM reaper?
**Topic Mapping:** OS → Memory → OOM Reaper
**Answer:** OOM killer selects victim and sends SIGKILL. But: victim may be sleeping in uninterruptible state (D state, waiting for I/O) → SIGKILL not processed immediately → system stays OOM while waiting. OOM reaper (Linux 4.6+): dedicated kernel thread that directly reclaims victim's anonymous memory (doesn't wait for process to exit). Steps: mark victim as being reaped → walk victim's page tables → unmap and free anonymous pages → huge memory freed immediately. Victim eventually exits when I/O completes, but memory already reclaimed. Prevents system hang during OOM despite slow-to-die process.

### Q181. What is the difference between RSS and PSS memory metrics?
**Topic Mapping:** OS → Memory Measurement
**Answer:** RSS (Resident Set Size): physical pages currently in RAM for this process, including ALL shared library pages counted in full. Misleading for shared libraries. PSS (Proportional Set Size): shared pages counted proportionally (1/N for N processes sharing). Example: nginx master + 4 workers, each uses 50MB RSS (10MB unique + 40MB shared libc). Total PSS per process: 10MB + 40MB/5 = 18MB. Total system cost ≈ 5×18MB = 90MB (vs 5×50MB = 250MB RSS). `/proc/[pid]/smaps` has PSS per VMA. `smem` tool reports PSS. USS (Unique Set Size): pages unique to this process = 10MB in example. Best metric for "what does killing this process free?"

### Q182. What is dirty page writeback and how to tune it?
**Topic Mapping:** OS → Memory → Writeback
**Answer:** Page cache dirty pages written back by kworker threads. Thresholds: `vm.dirty_background_ratio=10` (% of RAM): background writeback starts. `vm.dirty_ratio=20`: process writing is synchronously throttled. `vm.dirty_writeback_centisecs=500`: writeback every 5 seconds. Tuning for databases: lower dirty_ratio (5%) → less data loss on crash. For throughput: higher dirty_ratio → batch more writes → better disk utilization. `vm.dirty_expire_centisecs=3000`: dirty pages older than 30s forced to writeback. `iostat -x 1` shows write throughput. Databases bypass writeback with `fsync()` or `O_DIRECT` for durability guarantees.

### Q183. What is interrupt coalescing and NAPI in networking?
**Topic Mapping:** OS → Networking → Interrupt Handling
**Answer:** Per-packet interrupt at low load: good latency. At 1M pps: 1M interrupts/sec → CPU overwhelmed. Interrupt coalescing: NIC waits (up to usecs or N packets) before interrupting. `ethtool -C eth0 rx-usecs 50 rx-frames 32`. Trade-off: latency vs throughput. NAPI (New API): hybrid. First packet arrives → interrupt → switch to polling mode. Poll until no more packets, then re-enable interrupts. Avoids interrupt storm at high rates, low latency at low rates. All modern Linux NIC drivers use NAPI. High-frequency trading: disable coalescing (`rx-usecs 0`). Bulk data servers: enable coalescing for throughput.

### Q184. How does Linux implement per-CPU variables?
**Topic Mapping:** OS → Synchronization → Per-CPU
**Answer:** Per-CPU variables: each CPU has its own copy, eliminating cache-line bouncing and synchronization. `DEFINE_PER_CPU(long, packet_count)`. Access: `get_cpu()` (disable preemption + get CPU ID) → `per_cpu(packet_count, cpu)` → `put_cpu()`. Or: `this_cpu_inc(packet_count)` (inc on current CPU, single instruction, no preemption needed on x86 due to single-CPU ops being atomic with preemption disabled). Performance: ~10× faster than global atomic for per-CPU counters. Linux uses extensively: scheduler statistics, network counters, memory allocator caches (SLUB per-CPU cache), interrupt counters.

### Q185. What is memory-mapped I/O vs port-mapped I/O?
**Topic Mapping:** OS → Hardware → I/O Addressing
**Answer:** Port-Mapped I/O (PMIO): separate 64K I/O address space on x86. `IN`/`OUT` instructions. Legacy (keyboard, serial ports). Memory-Mapped I/O (MMIO): device registers appear at specific physical addresses. Accessed with regular load/store instructions. More common in modern systems — works on all architectures. GPU framebuffer, NVMe registers, ARM peripherals all use MMIO. Kernel: `ioremap(phys_addr, size)` maps device MMIO into kernel virtual address space. `readl()`/`writel()` — memory barriers included. `iounmap()` when done. Example: Raspberry Pi GPIO control = write to physical address 0x3F200000 (mapped via ioremap).

### Q186. What is the Linux scheduler's treatment of sleeping processes?
**Topic Mapping:** OS → Scheduling → Sleep/Wake
**Answer:** Process calls `wait_event(wq, condition)`: checks condition, if false → adds itself to wait queue, sets state TASK_INTERRUPTIBLE (or TASK_UNINTERRUPTIBLE for D state), calls `schedule()` → voluntary context switch. When event occurs: `wake_up(wq)` → sets all waiters TASK_RUNNING → adds to run queue → CFS schedules. TASK_INTERRUPTIBLE: woken by SIGTERM/SIGKILL (used for most waits). TASK_UNINTERRUPTIBLE (D state): not woken by signals (disk I/O wait — must complete). Cannot kill D-state process. `ps aux | grep " D "` shows D-state. Usually resolves when I/O completes; if persistent → kernel bug or hung storage.

### Q187. What is the Linux tmpfs filesystem?
**Topic Mapping:** OS → File Systems → tmpfs
**Answer:** tmpfs: RAM-backed virtual filesystem. Pages allocated from page cache — automatically swapped out under memory pressure. `mount -t tmpfs tmpfs /tmp -o size=2G`. Features: POSIX-compliant, supports files/dirs/symlinks/sockets, file permissions, can hold any file type. Performance: no disk I/O, no filesystem journal overhead → very fast. Auto-sized: uses only as much RAM as actually stored. Common uses: `/tmp`, `/dev/shm` (POSIX shared memory), container overlay upper layers, browser temporary files. Limitation: data lost on reboot (RAM-backed). Docker container's writable layer uses overlay2 which uses tmpfs for upper dir.

### Q188. What is CPU time steal in virtualization?
**Topic Mapping:** OS → Virtualization → CPU Steal
**Answer:** CPU steal (st in `top`): time hypervisor spent running other VMs when this VM wanted to run. VM thinks it has a CPU but hypervisor gave it to another VM. Symptom: high steal % → VM gets less CPU than requested → application appears slow even though VM's own CPU utilization is low. Causes: VM overprovisioning on host, noisy neighbors. Detection: `top` shows `%st` in CPU line. `vmstat 1` shows `st` column. Impact: scheduling latency spikes → bad for real-time or latency-sensitive apps. Solution: dedicated CPU pinning (no overprovisioning), bare metal, or reserved instances (cloud). Linux CFS doesn't know about steal — can't compensate.

### Q189. What is the difference between blocking and non-blocking file descriptors?
**Topic Mapping:** OS → I/O → Blocking vs Non-blocking
**Answer:** Blocking FD (default): `read()` blocks until data available. Simple programming model. Thread idles during wait. Non-blocking FD (`O_NONBLOCK`): `read()` returns EAGAIN immediately if no data. Application must retry or use select/poll/epoll to know when ready. Required for: single-threaded servers handling many connections, event loops (Node.js, Redis, Nginx). `fcntl(fd, F_SETFL, O_NONBLOCK)` makes existing FD non-blocking. `accept4(SOCK_NONBLOCK)` creates non-blocking socket directly. Async I/O (io_uring): submit read → continue → completion appears in CQ later. No blocking, no polling loop needed.

### Q190. How does Linux handle OOM in containers/Kubernetes?
**Topic Mapping:** OS → Memory → Container OOM
**Answer:** Container memory limit enforced by cgroup memory controller. When container exceeds limit: cgroup-level OOM killer fires (only considers processes in that cgroup, not system-wide). `memory.limit_in_bytes` = container limit. Kubernetes: `resources.limits.memory: 512Mi` → sets cgroup limit. Container OOM: process killed (exit code 137 = 128 + 9 for SIGKILL). `kubectl describe pod` shows `OOMKilled: true`. Prevention: set accurate `requests` (scheduling) and `limits` (enforcement). Guaranteed QoS: requests = limits → process never OOM-killed unless system-level OOM. BestEffort (no requests/limits): first killed in system OOM.

### Q191. What is the buddy allocator's fragmentation problem?
**Topic Mapping:** OS → Memory → Buddy Fragmentation
**Answer:** After long uptime with many small allocations: no 512-page contiguous region available even if 512 pages are free (scattered in 1-page and 2-page blocks). THP allocation fails. Solution: (1) Memory compaction: `echo 1 > /proc/sys/vm/compact_memory` — migrate movable pages, merge free blocks. (2) Reserve huge pages at boot before fragmentation: `hugepages=1024`. (3) ZONE_MOVABLE: keep movable pages separate for cleaner compaction. (4) Restart memory-intensive application to defragment. `cat /proc/buddyinfo` shows free blocks per order per zone — fragmentation visible as many order-0 (1-page) blocks, no high-order blocks.

### Q192. What are the OS-level tunings for high-performance databases?
**Topic Mapping:** OS → Performance → Database Tuning
**Answer:** Key Linux tunings for databases: (1) `vm.swappiness=1` — minimize swapping, prefer file cache reclaim. (2) Disable THP: `echo never > /sys/kernel/mm/transparent_hugepage/enabled`. (3) Enable huge pages: `vm.nr_hugepages=<N>`. (4) `vm.dirty_ratio=5` — limit dirty pages, reduce write stall. (5) I/O scheduler: `echo mq-deadline > /sys/block/sda/queue/scheduler` for HDD; `none` for NVMe. (6) `ulimit -n 65536` — max open files. (7) NUMA binding: `numactl --membind=0`. (8) Disable power saving: `cpufreq-set -g performance`. (9) `sysctl net.core.somaxconn=65535` for TCP backlog. (10) `fs.file-max=2097152`.

### Q193. What is memory overcommit and when does it matter?
**Topic Mapping:** OS → Memory → Overcommit
**Answer:** Linux overcommits: `malloc(10GB)` on 4GB RAM succeeds — physical pages allocated only on first write (demand paging + zero page). Works because most processes never use all allocated memory. Modes: `vm.overcommit_memory=0` (heuristic: allow reasonable overcommit), `=1` (always allow — used in HPC/ML where large virtual allocations common), `=2` (never overcommit: CommitLimit = swap + overcommit_ratio% of RAM). Mode 2 consequences: `malloc()` fails predictably before OOM — no sudden kills. Used in: financial systems, safety-critical apps needing predictable behavior. `cat /proc/meminfo | grep Commit` shows CommitLimit and CommittedAS (total committed).

### Q194. What is the role of the memory compaction daemon (kcompactd)?
**Topic Mapping:** OS → Memory → Memory Compaction
**Answer:** kcompactd: kernel thread performing proactive memory compaction to make higher-order (huge page) allocations possible. Triggered when: huge page allocation fails, high-order free memory drops below threshold, direct compaction too slow. Algorithm: scan zone from both ends — migrate movable pages from low addresses to high addresses, freeing low addresses as contiguous huge page blocks. Pages must be MIGRATE_MOVABLE type (user anon pages, file-backed pages, page cache). MIGRATE_UNMOVABLE (kernel code, slab) cannot be moved. Compaction has overhead: scanning all pages, migrating, TLB shootdown. `cat /proc/vmstat | grep compact` shows compaction stats.

### Q195. What is the Linux scheduler's CPU bandwidth control for cgroups?
**Topic Mapping:** OS → Scheduling → CPU Bandwidth
**Answer:** CFS bandwidth control: `cpu.cfs_period_us` (default 100ms) and `cpu.cfs_quota_us`. Ratio = CPU limit. Example: quota=50000, period=100000 → 50% of one CPU. quota=200000, period=100000 → 2 full CPUs. Implementation: each period, each cgroup gets quota replenished. When cgroup exhausts quota → throttled (TASK_THROTTLED) until next period. Kubernetes: `resources.limits.cpu: "2"` → quota=200000, period=100000. Problem: CPU throttling causes latency spikes even when CPUs available. Solution: increase period or reduce oversubscription. `throttled_time` in cpu.stat shows throttle overhead.

### Q196. Explain the process of loading a shared library.
**Topic Mapping:** OS → Process Management → Dynamic Linking
**Answer:** ELF executable has `.dynamic` section with list of needed shared libraries. `ld.so` (dynamic linker, itself a shared library mapped by kernel) runs before main(). Steps: (1) Parse `.dynamic` → find needed libraries. (2) Search: `LD_LIBRARY_PATH`, `/etc/ld.so.cache`, `/lib`, `/usr/lib`. (3) Open library file → check if already loaded (by device+inode, not name). (4) Map library into process address space via mmap (read-only code pages shared across all processes using library). (5) Resolve symbols: PLT (Procedure Linkage Table) / GOT (Global Offset Table) — lazy binding: first call → ld.so resolves symbol → patches GOT. Subsequent calls: direct. `LD_PRELOAD`: force loading specific library first — used for profiling, debugging, intercepting library calls.

### Q197. What are Linux CPU C-states and their impact on latency?
**Topic Mapping:** OS → Power Management → C-States
**Answer:** C-states = CPU idle states. Deeper sleep = more power saving = longer wakeup latency. C0: active (running). C1: halt (clock gated, <1μs exit). C3: sleep (L2 flush, ~100μs exit). C6: deep sleep (core voltage reduced, ~500μs exit). C10: package sleep (all cores + uncore powered down, ~1ms exit). For latency-sensitive applications: disable deep C-states. `cpupower idle-set -D 2` limits to C2 max. Or: busy-wait (spin) to prevent entering C-state — extreme measure. `powertop` shows C-state residency. Cloud: C-state transitions contribute to scheduling jitter in VMs. `GRUB_CMDLINE: idle=poll` — never enters C-state (burns power, eliminates jitter).

### Q198. What is the difference between `mlock()` and `MAP_LOCKED`?
**Topic Mapping:** OS → Memory → Memory Locking
**Answer:** `mlock(addr, len)`: lock specific memory region in RAM — no swapping, no page faults. `mlockall(MCL_CURRENT | MCL_FUTURE)`: lock all current and future allocations. Required for: real-time processes (no page fault latency), security-sensitive data (don't write passwords to swap), DMA buffers. `MAP_LOCKED` flag in mmap(): equivalent to mlock for mapped region. Requires: `CAP_IPC_LOCK` capability or root, OR `ulimit -l` must be large enough (`ulimit -l unlimited`). Locked memory not counted in RSS for OOM purposes — won't be killed. Check: `cat /proc/meminfo | grep Mlocked`. Cost: locked memory never reclaimed → may cause memory pressure for other processes.

### Q199. What is the difference between iowait and actual disk I/O time?
**Topic Mapping:** OS → I/O → iowait
**Answer:** `iowait` (shown in `top`/`vmstat` as `wa`): percentage of time CPU was idle AND at least one I/O was outstanding. Common misunderstanding: iowait ≠ CPU blocked waiting for I/O. iowait is a subset of idle time — CPU could have run other tasks but none were ready. High iowait: lots of I/O outstanding + no other runnable tasks. Low iowait with slow I/O: CPU busy with other tasks even while I/O happens. True I/O performance: `iostat -x 1` → `%util` (device utilization), `r_await`/`w_await` (I/O latency), `rkB/s`/`wkB/s` (throughput). `iotop` shows per-process I/O. `blktrace` for detailed I/O tracing.

### Q200. What is the Linux kernel's approach to scheduling on heterogeneous cores (big.LITTLE / Intel P+E)?
**Topic Mapping:** OS → Scheduling → Asymmetric Multiprocessing
**Answer:** Modern CPUs: performance cores (P-cores: fast, high power) + efficiency cores (E-cores: slow, low power). ARM big.LITTLE (Cortex-A77 big + Cortex-A55 LITTLE), Intel 12th+ gen. Linux asymmetric scheduling: CPU capacities in `/sys/devices/system/cpu/cpu*/cpu_capacity` (P-core=1024, E-core=512). Scheduler prefers placing tasks on P-cores; background/low-priority → E-cores. schedutil governor × capacity → appropriate frequency. Android: foreground apps → P-cores (fast response), background → E-cores (battery). Kubernetes: CPU Manager doesn't yet distinguish P/E cores — relies on OS scheduler. Benchmarking: must control which cores are measured — P and E cores report different performance for same workload.

---

## ADVANCED QUESTIONS — OS (Q131–Q200)

---

### Q131. How does Linux implement NUMA-aware scheduling?

**Topic Mapping:** OS → Advanced Scheduling → NUMA

**Explanation:**
NUMA (Non-Uniform Memory Access): multi-socket servers have local RAM per socket. Local access ~100 ns; cross-socket ~300 ns+.

**NUMA Balancing:** Linux periodically sets pages PROT_NONE → on next access, page fault fires → OS checks: is faulting CPU on different NUMA node than page's physical location? If yes: migrate task to page's node OR migrate page to task's node based on affinity scores (`task_numa_fault()`).

```bash
numactl --hardware                          # show topology
numactl --membind=0 --cpunodebind=0 ./app   # pin to node 0
numastat -p <pid>                           # per-node memory stats
```

**Impact:** Ignoring NUMA on 2-socket server → 2–3× slowdown for memory-intensive work. PostgreSQL, JVM, MySQL all benefit from explicit NUMA pinning.

**Interview Tip:** Any systems/database role — mention NUMA. Immediately shows hardware-level depth.

---

### Q132. What is the Linux VFS (Virtual Filesystem Switch)?

**Topic Mapping:** OS → File Systems → VFS

**Explanation:**
VFS is an abstraction layer giving a uniform interface over all filesystem types. User calls `open()`, `read()`, `write()` — same syscall whether FS is ext4, NFS, tmpfs, or procfs.

**Four VFS objects:**
- **superblock** — mounted filesystem metadata
- **inode** — file/directory metadata + operations vtable  
- **dentry** — cached path component (name→inode); dentry cache (dcache) makes path resolution O(1)
- **file** — per-open-file state (offset, flags, pointer to dentry)

**Each object has a function-pointer table:**
```c
struct inode_operations {
    struct dentry *(*lookup)(struct inode *, struct dentry *, unsigned int);
    int (*create)(struct inode *, struct dentry *, umode_t, bool);
    int (*mkdir)(struct inode *, struct dentry *, umode_t);
};
```

**Path resolution `/home/user/file.txt`:**
root inode → dcache hit "home" → dcache hit "user" → dcache hit "file.txt" → inode → file object → fd returned.

**Interview Tip:** Without dcache every `stat()` requires multiple disk reads. VFS + dcache is why Linux file operations feel instant.

---

### Q133. What is io_uring and how does it achieve near-zero syscall overhead?

**Topic Mapping:** OS → I/O → io_uring

**Explanation:**
io_uring (Linux 5.1, 2019) uses two ring buffers mmap'd into user space — Submission Queue (SQ) and Completion Queue (CQ) — so user code can submit and harvest I/O without a syscall in the fast path.

```
User Space                  Kernel
  SQ (write SQEs) ──────►  io_uring instance
  CQ (read CQEs)  ◄──────  completed I/O placed here
```

**SQPOLL mode:** A kernel thread polls SQ continuously → zero syscalls even for submissions. User writes SQE → kernel sees it without being woken.

**Supported ops:** read, write, accept, connect, recv, send, splice, fsync, statx, openat, …

**Who uses it:** PostgreSQL (experimental), Nginx, RocksDB, Tokio (Rust async runtime).

**Interview Tip:** io_uring reduces per-request overhead from ~1 µs (syscall) to ~100 ns (ring write). For 1M IOPS workloads, that's the difference between needing 1 core vs 10 cores.

---

### Q134. Compare HugeTLB (explicit) vs THP (Transparent Huge Pages).

**Topic Mapping:** OS → Memory Management → Huge Pages

**Explanation:**
Standard pages = 4 KB → TLB with 512 entries covers 2 MB. 2 MB huge pages → same TLB covers 1 GB. Dramatic TLB-miss reduction for large working sets.

**HugeTLB (explicit):**
```bash
echo 1024 > /proc/sys/vm/nr_hugepages   # reserve 2 GB upfront
# App uses MAP_HUGETLB in mmap()
```
Guaranteed allocation; non-swappable; requires app changes. Oracle DB, JVM `-XX:+UseLargePages`.

**THP (Transparent Huge Pages):**
`khugepaged` daemon automatically promotes 512 contiguous 4 KB pages to one 2 MB page. Transparent to app — but promotion/demotion causes latency spikes.

MongoDB, Redis: **disable THP**:
```bash
echo never > /sys/kernel/mm/transparent_hugepage/enabled
```

**Interview Tip:** "THP latency spikes" is a famous production gotcha. Every senior SRE has seen it. Mentioning it scores points.

---

### Q135. What are Linux scheduling domains?

**Topic Mapping:** OS → Scheduling → Scheduling Domains

**Explanation:**
Linux organises CPUs into a topology-aware hierarchy of scheduling domains:

```
Root domain (all CPUs)
├── NUMA node 0  (socket 0, CPUs 0–3)
│   └── MC domain (share L3)
│       ├── SMT pair (CPU 0 + HT sibling)
│       └── SMT pair (CPU 2 + HT sibling)
└── NUMA node 1  (socket 1, CPUs 4–7)
    └── MC domain
```

Load-balance frequency and cost per level:
- **SMT** — balance very frequently, very cheap (same physical core)  
- **MC**  — moderate (different core, same die, shared L3)  
- **NUMA** — rarely (cross-socket migration destroys cache locality)

A task migrates only if imbalance cost > migration cost. `taskset -c 0-3` pins a process to a subset, preventing cross-domain migration.

**Interview Tip:** Understanding scheduling domains explains why `taskset` can improve performance on NUMA machines — you prevent the scheduler from making expensive cross-node migrations.

---

### Q136. How does the Linux buddy system allocate physical memory?

**Topic Mapping:** OS → Memory Management → Buddy Allocator

**Explanation:**
Physical memory is divided into blocks of power-of-2 page counts (1, 2, 4, 8 … 1024 pages). Each size has a free list.

**Allocation of N pages:**
1. Round N up to next power of 2.
2. Check that free list; if empty, split a larger block → two "buddies".
3. Return one buddy; put the other in the smaller free list.

**Freeing:**
```
Buddy address = block_addr XOR (1 << order)
```
If buddy is also free → merge → recurse upward.

**Zones:** ZONE_DMA (0–16 MB), ZONE_NORMAL (16 MB–896 MB on 32-bit), ZONE_HIGHMEM / ZONE_MOVABLE for larger address spaces.

**Slab on top:** Buddy gives pages (4 KB minimum). Slab carves pages into fixed-size object caches (inode = 328 B, task_struct = ~9 KB) → eliminates internal fragmentation for kernel objects.

**Interview Tip:** Buddy eliminates external fragmentation of physical pages. Slab eliminates internal fragmentation for kernel objects. Together they make kernel allocation O(1).

---

### Q137. Walk through a COW page fault after fork() at the kernel level.

**Topic Mapping:** OS → Memory Management → COW Implementation

**Explanation:**

**After `fork()`:**
All parent's PTEs copied to child, marked read-only. Both parent and child point to same physical frames.

**Child writes to shared page:**
1. CPU: write to read-only PTE → protection fault.
2. x86 pushes faulting address into CR2; CPU traps to `do_page_fault()`.
3. Linux finds VMA — write fault + VMA writable but PTE read-only → COW fault.
4. `do_cow_fault()`:
   a. `alloc_page()` — get fresh frame from buddy.
   b. `copy_user_highpage()` — copy old page content to new frame.
   c. Update child's PTE: new frame, writable.
   d. Decrement old page's `_refcount`; if refcount drops to 1 (only parent), mark parent's PTE writable too.
5. Flush TLB for this address on current CPU.
6. Restart the faulting instruction (not advance to next — instruction didn't complete).

**Interview Tip:** Why restart rather than resume? The faulting instruction never finished — it must re-execute now that the mapping is present and writable.

---

### Q138. How does Linux select the OOM victim in detail?

**Topic Mapping:** OS → Memory Management → OOM Killer

**Explanation:**
When all physical + swap memory is exhausted and reclaim fails, `out_of_memory()` is called.

**Score calculation (simplified):**
```
oom_badness = (RSS + swap_pages + page_table_pages)
            × (oom_score_adj + 1000) / 1000
```
- Larger memory user → higher score.
- `oom_score_adj` (-1000 … +1000): admin-tunable per process.
  - -1000: fully protected (sshd, systemd).
  - +1000: killed first.

**Result:** Process with highest score gets SIGKILL. OOM killer logs to `dmesg`.

**Kubernetes:** BestEffort pods get `oom_score_adj = 1000`; Guaranteed pods get `-997`. So BestEffort always die first. Always set `resources.requests` and `resources.limits` to avoid surprise kills.

**Interview Tip:** In Kubernetes, `OOMKilled` exit code = container exceeded its memory limit. Fix: increase `limits.memory` or fix the memory leak.

---

### Q139. What does PREEMPT_RT add to Linux?

**Topic Mapping:** OS → Real-Time → PREEMPT_RT

**Explanation:**
Standard Linux: worst-case scheduling latency 10–50 ms (non-preemptible kernel sections, long IRQ handlers).

**PREEMPT_RT converts:**
- Spinlocks → sleeping mutexes (holder can be preempted by higher-priority task).
- Hard-IRQ handlers → threaded IRQs (preemptible kernel threads).
- Timer handling → threaded, with priority.

**Results:**
- Worst-case latency: 50–200 µs (vs 10–50 ms without).
- Not hard real-time but suitable for robotics, audio, industrial automation.

**Test tool:**
```bash
cyclictest -l1000000 -m -n -a0 -t1 -p99 -i200 -h400
```
Measures scheduling jitter. PREEMPT_RT: max < 200 µs; vanilla Linux: max > 10 ms.

**Status:** PREEMPT_RT partially merged into mainline (Linux 5.15+). Full integration ongoing.

---

### Q140. What kernel self-protection mechanisms prevent exploitation?

**Topic Mapping:** OS → Security → Kernel Self-Protection

**Explanation:**

| Mechanism | What it prevents |
|-----------|-----------------|
| **SMEP** | Kernel executing user-space code (code-injection) |
| **SMAP** | Kernel reading/writing user-space without explicit enable (pointer-dereference attacks) |
| **KASLR** | Hardcoded kernel addresses (forces attacker to leak address first) |
| **Stack canary** | Kernel stack smashing detected before return |
| **KPTI** | Meltdown side-channel (user-mode page tables have no kernel mappings) |
| **CFI** | ROP chains (only valid call targets allowed) |
| **RO after init** | Kernel code/data read-only after init; modules must be signed |

**KPTI cost:** 5–30% overhead for syscall-heavy workloads (page-table switch on every syscall). Intel PCID reduces TLB flush cost.

**Interview Tip:** Meltdown → KPTI. Spectre → Retpoline + IBRS. Both are OS-level mitigations for CPU microarchitecture bugs. Know the tradeoff: security vs performance.

---

### Q141. How does mmap() interact with the page cache?

**Topic Mapping:** OS → Memory Management → mmap + Page Cache

**Explanation:**
`mmap(fd)` creates a VMA in the process's address space backed by the file. No pages loaded yet.

**On first access:**
1. Page fault → OS checks page cache: is this file page already in RAM?
2. **Hit:** map existing cache page into process PTE → zero disk I/O.
3. **Miss:** read from disk into page cache → map into PTE.

**Key insight:** mmap pages ARE page cache pages. Multiple processes mapping the same file share the exact same physical pages — true zero-copy sharing.

**vs `read()`:**
- `read()`: disk → page cache → kernel buffer → user buffer (2 copies).
- `mmap()`: disk → page cache, directly visible in user address space (0 extra copies).

**Dirty pages:** mmap write → page marked dirty → `pdflush`/`wb_workfn` writes to disk asynchronously. `msync()` forces immediate flush.

**Interview Tip:** Databases argue for/against mmap. Pro: OS handles caching. Con: OS may evict unpredictably, no control over I/O ordering. PostgreSQL uses explicit buffer management instead of mmap for its shared_buffers.

---

### Q142. How does Linux `perf` use hardware performance counters?

**Topic Mapping:** OS → Performance Profiling → perf

**Explanation:**
Modern CPUs have 4–8 hardware PMU (Performance Monitoring Unit) counters that can count: instructions retired, CPU cycles, L1/L2/L3 cache misses, branch mispredictions, TLB misses, memory bandwidth.

**Workflow:**
1. `perf_event_open()` syscall programs a PMU counter.
2. Counter overflows → NMI fires → kernel captures current PC + call stack → sample stored.
3. `perf report` aggregates samples → hottest functions shown.

```bash
perf stat ./app                 # count hardware events
perf record -g ./app            # record samples with call stacks
perf report                     # interactive flame-chart view
perf top                        # live system-wide view
```

**Flame graphs:**
```bash
perf record -F 99 -g ./app
perf script | stackcollapse-perf.pl | flamegraph.pl > out.svg
```

**eBPF + perf:** Attach eBPF programs to perf events with near-zero overhead. Used by `bpftrace`, Datadog, Cilium.

**Interview Tip:** Being able to describe perf + flame graphs in an interview is a strong signal you've done real performance work.

---

### Q143. Explain Meltdown and Spectre, and their OS mitigations.

**Topic Mapping:** OS → Security → CPU Vulnerabilities

**Explanation:**

**Meltdown (2018 — mostly Intel):**
CPU speculatively executes a kernel memory read while the privilege check is in-flight. Result lands in cache before the exception is raised. Attacker measures cache timing → reads kernel memory from user space.

**OS fix → KPTI:** Keep separate page tables for user mode (no kernel mappings). Mode switch on every syscall → 5–30% overhead for syscall-heavy apps.

**Spectre (2018 — Intel + AMD + ARM):**
Tricks branch predictor into speculatively executing attacker-chosen code; leaks data via cache side channel. Multiple variants (v1: bounds-check bypass, v2: branch target injection).

**OS fixes:**
- **Retpoline:** Replaces indirect jumps with a return-trampoline sequence; CPU cannot predict the target → v2 defeated.
- **IBRS / IBPB:** Microcode updates to flush branch predictors on privilege transitions.
- **Array index masking:** Masks array indices before use even after bounds check — defeats v1.

**Performance impact:** KPTI ~5–30%. Spectre mitigations ~2–15%. Significant for databases and web servers with high syscall rates.

---

### Q144. How does Linux safely transfer data between user and kernel space?

**Topic Mapping:** OS → Memory Management → User-Kernel Copy

**Explanation:**
The kernel cannot blindly dereference user pointers — they may be invalid, swapped out, or maliciously crafted to point into kernel space.

```c
copy_from_user(kernel_buf, user_ptr, size);  // user → kernel
copy_to_user(user_ptr, kernel_buf, size);    // kernel → user
get_user(kvar, uptr);   // single scalar from user
put_user(kvar, uptr);   // single scalar to user
```

**What `copy_from_user` does:**
1. `access_ok()`: validate user_ptr is in user address range (not kernel space).
2. Enable SMAP (`stac` instruction) — allows kernel to touch user pages temporarily.
3. Copy data byte-by-byte; if page fault mid-copy → fixup table handles it, returns bytes-not-copied.
4. Disable SMAP (`clac`).

**SMAP protection:** Without SMAP a kernel bug could follow a user-supplied pointer into kernel-exploit shellcode. SMAP prevents accidental (or deliberate) user-space dereferences in kernel mode.

**Zero-copy DMA:** For bulk I/O, pin user pages (`get_user_pages()`), pass physical addresses to DMA controller → hardware writes directly to user buffer. No kernel copy at all.

---

### Q145. What is eBPF and how does it safely run user code in the kernel?

**Topic Mapping:** OS → Advanced → eBPF

**Explanation:**
eBPF lets user-defined programs run safely inside the Linux kernel without modifying kernel source or loading traditional modules.

**Safety pipeline:**
1. Write eBPF program in C → compile to eBPF bytecode with `clang -target bpf`.
2. Load via `bpf()` syscall → kernel **verifier** checks:
   - Bounded loops only (terminates).
   - All pointer dereferences in-bounds.
   - No illegal instructions.
3. JIT-compiled to native machine code → runs at near-native speed.

**Attachment points:** XDP (before network stack), tc (traffic control), kprobes, uprobes, tracepoints, LSM hooks, perf events.

**Use cases:**
- **Observability:** `bpftrace`, `bcc` — trace I/O, CPU, memory with zero app changes.
- **Networking:** Cilium (Kubernetes CNI with full eBPF dataplane), Facebook Katran LB.
- **Security:** seccomp-BPF (syscall filter), Falco runtime security.

**eBPF maps:** Key-value stores shared between eBPF programs and user space — used for state and communication.

**Interview Tip:** eBPF is the most important Linux innovation of the 2020s. Kubernetes networking (Cilium), observability (Datadog, Pixie), and security all build on it.

---

### Q146. What is a kernel crash dump and how is it captured?

**Topic Mapping:** OS → Reliability → kdump

**Explanation:**
When the kernel panics (null-deref in kernel, corrupted memory, hardware error) it can dump the entire memory state for post-mortem analysis.

**kdump mechanism:**
1. At boot, reserve a small memory region ("crash kernel") and load a tiny second kernel there.
2. On panic: primary kernel hands control to crash kernel via `kexec` (no hardware reset).
3. Crash kernel uses `/dev/crash` to read primary kernel's memory → writes `vmcore` dump file.
4. `crash vmcore /boot/System.map` allows interactive post-mortem: backtrace, variable inspection.

**Why kexec instead of reset?** Hardware reset clears RAM; kexec preserves it for analysis.

**Interview Tip:** Real SREs and kernel engineers use kdump. Knowing it exists and why it works via kexec without a reset is impressive detail.

---

### Q147. What is CPU affinity and when does it improve performance?

**Topic Mapping:** OS → Scheduling → CPU Affinity

**Explanation:**
CPU affinity pins a thread/process to specific cores so the scheduler never migrates it.

```bash
taskset -c 0-3 ./myapp          # run on cores 0–3
taskset -cp 0 $PID              # pin existing process to core 0
# Java / Python need JNA or subprocess call for per-thread affinity
```

**Why it helps:**
1. **Cache warmth:** Same core → L1/L2 always hot → fewer cache misses.
2. **NUMA locality:** Pin to same NUMA node as allocated memory → no remote access penalty.
3. **Determinism:** Real-time tasks get guaranteed CPU time without migration jitter.
4. **Avoid false sharing:** Pin cooperating threads to same physical core (share L1) or separate cores (no false sharing) depending on access pattern.

**When it hurts:** Over-pinning on a busy system → pinned core at 100% while other cores idle.

---

### Q148. What is the difference between hardirq and softirq?

**Topic Mapping:** OS → I/O → Interrupt Handling

**Explanation:**
Two-level interrupt handling prevents long ISRs from blocking new hardware interrupts.

**hardirq (top half):**
- Runs with interrupts disabled on that CPU.
- Must be extremely fast: acknowledge device, save minimal state, schedule softirq work.
- Cannot sleep.

**softirq (bottom half):**
- Runs with interrupts re-enabled — other hardware interrupts can arrive.
- Handles bulk work deferred from hardirq: `NET_RX_SOFTIRQ` (packet processing), `TIMER_SOFTIRQ`, `BLOCK_SOFTIRQ`.
- Runs in interrupt context (still no sleeping).

**Tasklets:** Serialised per-CPU softirq units. Won't run concurrently with themselves. Used by most drivers.

**Workqueues:** Can sleep (unlike tasklets). Run in a kernel thread (`kworker`). For work that needs blocking/sleeping (e.g., disk I/O completion that allocates memory).

**Interview Tip:** "Why can't you sleep in interrupt context?" → No process context; the scheduler needs a process to context-switch to. This is why the two-level split exists.

---

### Q149. How does cgroup CPU scheduling (bandwidth control) work?

**Topic Mapping:** OS → Scheduling → cgroup CFS Bandwidth

**Explanation:**
Linux CFS + cgroups enforces per-group CPU quotas.

**cgroupv1 parameters:**
- `cpu.shares` — relative weight (default 1024). Like niceness but group-wide.
- `cpu.cfs_quota_us` / `cpu.cfs_period_us` — hard cap: quota µs of CPU per period µs.

**Example:** `quota=50000, period=100000` → group gets at most 50% of one CPU core.

**Group hierarchy scheduling:**
```
/ (root, 100%)
├── userA (50% share)  ← 10 processes share 50% between them
└── userB (50% share)  ← 1 process gets full 50%
```
Fair at each level — prevents one user's many processes from starving another user.

**Kubernetes:** `resources.limits.cpu: "2"` → `cpu.cfs_quota_us = 200000`, `period = 100000` → container gets max 2 cores. `resources.requests.cpu` sets `cpu.shares`.

**Interview Tip:** CPU throttling in Kubernetes (visible in `kubectl top` or Prometheus `container_cpu_throttled_seconds_total`) is caused by cgroup bandwidth limits being hit. Increase `limits.cpu` or reduce workload.

---

### Q150. What is memory hotplug in Linux?

**Topic Mapping:** OS → Memory Management → Hotplug

**Explanation:**
Server-class hardware and cloud VMs support adding/removing RAM while the system is running.

**Adding memory:**
BIOS/firmware signals new memory region → kernel receives ACPI notification → memory added in sections (128 MB+ each) → section handed to buddy allocator → immediately usable.

**Removing memory (harder):**
Must evacuate all pages from target section first:
1. Mark section ZONE_MOVABLE → only movable/reclaimable pages allocated here.
2. `memory_hotremove()`: migrate movable pages away, reclaim reclaimable ones.
3. If non-movable pages remain (kernel static data, pinned DMA buffers) → removal fails.
4. On success: offline section → notify drivers → remove from buddy.

**Cloud use:** Live VM resize (AWS EC2 memory balloon, Azure memory hot-add) relies on memory hotplug. Kubernetes: memory requests/limits affect balloon driver behaviour in VMs.

---

### Q151. What is the difference between TASK_INTERRUPTIBLE and TASK_UNINTERRUPTIBLE?

**Topic Mapping:** OS → Process Management → Sleep States

**Explanation:**
Both are sleeping (blocked) states, but differ in whether signals can wake the process.

**TASK_INTERRUPTIBLE (S state in `ps`):**
- Process waiting for an event (timer, lock, socket data).
- Can be woken by a signal (SIGTERM, SIGKILL, etc.) — used for most user-space waits.
- `wait_event_interruptible()` in kernel.

**TASK_UNINTERRUPTIBLE (D state in `ps`):**
- Process waiting for hardware I/O or kernel lock that cannot be safely interrupted.
- Signals are ignored — cannot be killed with `kill -9` until the wait completes.
- `wait_event()` in kernel.
- Long D-state → usually stuck disk I/O (NFS hang, dying disk).

**Detecting stuck D-state processes:**
```bash
ps aux | awk '$8 == "D"'    # find D-state processes
cat /proc/<pid>/wchan       # what kernel function it's waiting in
```

**Interview Tip:** "Why can't I kill a process with kill -9?" → It's in D state (TASK_UNINTERRUPTIBLE). Fix the underlying I/O stall, not the process.

---

### Q152. What is a system call hook and how do rootkits abuse it?

**Topic Mapping:** OS → Security → Syscall Hooking

**Explanation:**
The syscall dispatch table (`sys_call_table[]`) maps syscall numbers to handler functions. A rootkit with kernel access overwrites entries:

```c
// Rootkit overwrites sys_call_table[__NR_getdents64]
// with malicious handler that filters out rootkit files
original_getdents = sys_call_table[__NR_getdents64];
sys_call_table[__NR_getdents64] = evil_getdents;
```

Result: `ls` never shows rootkit files because getdents64 is hooked.

**Modern kernel defenses:**
- `CONFIG_STRICT_KERNEL_RWX`: `sys_call_table` lives in read-only memory → cannot be overwritten without disabling write-protect (detectable).
- **Module signing:** Rootkit module rejected at `insmod` time on secure-boot systems.
- **kprobes:** Legitimate safe hooking for tracing — uses a different mechanism (breakpoints), kernel-approved.

**Detection:** `unhide`, `rkhunter`; compare `sys_call_table` entries to known-good values from `System.map`.

---

### Q153. What is the role of guard pages in stack overflow detection?

**Topic Mapping:** OS → Memory Management → Stack Guard

**Explanation:**
Each thread's stack has a **guard page** — a virtual page with no permissions (`PROT_NONE`) placed just below the stack region.

**Stack overflow detection:**
1. Infinite recursion exhausts stack space.
2. Stack pointer crosses into guard page.
3. CPU: write to unmapped page → page fault.
4. OS: fault address in guard page region → send SIGSEGV to process (controlled crash).

**Why not just let it grow?** Stack could silently overwrite the heap or other mappings, causing data corruption that's impossible to debug. Guard page converts silent corruption into a deterministic crash.

**Kernel stack overflow:**
Kernel stack is only 8 KB. If exhausted → overwrites adjacent kernel memory → system corruption. `CONFIG_STACKPROTECTOR` adds a canary checked on function return. `CONFIG_VMAP_STACK` (Linux 4.9+): maps kernel stack with virtual memory so guard pages work for kernel stacks too.

---

### Q154. What is memory compaction and when does it run?

**Topic Mapping:** OS → Memory Management → Compaction

**Explanation:**
Physical memory fragments over time: free pages scattered in small non-contiguous groups → cannot satisfy huge-page or large contiguous allocation requests.

**Compaction:** Migrate movable pages together to create contiguous free regions.

```
Before: [used][free][used][used][free][used][free][free]
After:  [used][used][used][used][free][free][free][free]
```

**`kcompactd` daemon:** Wakes when direct reclaim fails to find contiguous free pages or when THP promotion is attempted. Runs per-NUMA-node.

**Manual trigger:**
```bash
echo 1 > /proc/sys/vm/compact_memory
```

**Cost:** Compaction migrates pages (copy data + update PTEs + TLB shootdown) → expensive, causes latency spikes. That's another reason THP is disabled for latency-sensitive applications.

**ZONE_MOVABLE:** A memory zone containing only movable/reclaimable pages, making compaction in that zone efficient.

---

### Q155. Explain the Linux audit subsystem.

**Topic Mapping:** OS → Security → Audit

**Explanation:**
Linux audit (`auditd`) logs security-relevant kernel events: file accesses, syscalls, user logins, privilege escalations.

**How it works:**
1. Audit rules configured via `auditctl`.
2. Kernel generates audit records in a ring buffer when rules match.
3. `auditd` daemon reads from ring buffer → writes to `/var/log/audit/audit.log`.

```bash
auditctl -w /etc/passwd -p rwa -k passwd_watch    # watch file
auditctl -a always,exit -F arch=b64 -S execve     # log all exec
ausearch -k passwd_watch                           # query log
aureport --summary                                 # report
```

**Compliance:** PCI-DSS, HIPAA, SOX require audit trails. `auditd` is the standard solution on Linux.

**SELinux integration:** SELinux denials automatically generate audit records — `ausearch -m avc` shows all denials.

---

### Q156. What is false sharing in multi-core systems?

**Topic Mapping:** OS → Concurrency → Cache False Sharing

**Explanation:**
Two threads access different variables that happen to occupy the **same cache line** (64 bytes on x86). One thread writes → entire cache line marked dirty → invalidated on all other CPUs → other thread must reload from LLC or RAM — even though it touched a *different* variable.

**Example:**
```c
struct Counters {
    long a;  // Thread 1 increments this
    long b;  // Thread 2 increments this
} counters;
// Both a and b fit in one 64-byte cache line → false sharing!
```

**Fix: pad to cache line boundary:**
```c
struct Counters {
    long a;
    char _pad1[56];   // pad to 64 bytes
    long b;
    char _pad2[56];
} __attribute__((aligned(64)));

// Or C++17:
alignas(64) long a;
alignas(64) long b;
```

**Linux kernel:** `DEFINE_PER_CPU(long, my_counter)` gives each CPU its own copy in a separate cache line → zero false sharing.

**Interview Tip:** False sharing can cause 10–100× slowdown on tight loops. Always pad hot shared data structures to 64-byte alignment.

---

### Q157. What is ASLR and how does it work at the kernel level?

**Topic Mapping:** OS → Security → ASLR

**Explanation:**
ASLR (Address Space Layout Randomisation) randomises the base addresses of stack, heap, code, and shared libraries each time a process starts. Attackers can't hardcode addresses in exploits.

**Randomised regions:**
- Stack base (random offset from top of address space)
- `mmap()` base (shared libraries, anonymous mappings)
- Heap `brk()` start
- PIE executable base (if compiled with `-fPIE -pie`)

**Kernel implementation:**
`arch_mmap_rnd()` generates random bits; added to base address during `load_elf_binary()`. Amount of randomness: 28 bits on x86-64 → 2^28 = 268M possible stack locations.

```bash
cat /proc/sys/kernel/randomize_va_space
# 0=off, 1=partial, 2=full (default)
```

**Bypasses:** Info-leak vulnerability reveals address → ASLR defeated. That's why info-leak mitigations (KPTI, KASLR) are also needed.

**Interview Tip:** ASLR + NX (no-execute) + stack canaries = the standard exploit-mitigation trinity. Mention all three together.

---

### Q158. What is the vDSO (virtual Dynamic Shared Object)?

**Topic Mapping:** OS → System Calls → vDSO

**Explanation:**
For certain frequently-called syscalls (`gettimeofday`, `clock_gettime`, `time`, `getcpu`), the overhead of a full user→kernel mode switch is wasteful. vDSO eliminates it.

**How:**
1. Kernel maps a read-only page into every process's address space at a random address.
2. That page contains kernel code + a data region the kernel updates on every tick (current time, CPU ID).
3. `glibc` calls vDSO function directly (no `syscall` instruction) → reads time from shared page.
4. Result: `clock_gettime()` costs ~5 ns instead of ~100 ns.

```bash
cat /proc/<pid>/maps | grep vdso   # see vDSO mapping
ldd /bin/ls | grep vdso
```

**Interview Tip:** vDSO explains why `gettimeofday()` doesn't appear in `strace` output — it never enters the kernel. This is a great detail to mention when discussing syscall overhead.

---

### Q159. What is a memory-mapped region (VMA) in Linux?

**Topic Mapping:** OS → Memory Management → VMA

**Explanation:**
Every region of a process's virtual address space is described by a **VMA** (Virtual Memory Area) — a kernel structure tracking:

```c
struct vm_area_struct {
    unsigned long vm_start;   // region start address
    unsigned long vm_end;     // region end address
    pgprot_t vm_page_prot;    // protection flags (R/W/X)
    unsigned long vm_flags;   // MAP_SHARED, MAP_PRIVATE, etc.
    struct file *vm_file;     // backing file (or NULL for anonymous)
    unsigned long vm_pgoff;   // offset in file
    struct vm_operations_struct *vm_ops;  // fault handler, etc.
};
```

**VMAs form a sorted red-black tree** (+ linked list) per process. Page-fault handler: find VMA for faulting address → call `vm_ops->fault()` → appropriate page loaded.

```bash
cat /proc/<pid>/maps   # show all VMAs for process
# format: start-end perms offset dev inode path
```

**Interview Tip:** `Segmentation fault` = faulting address has no VMA (or VMA doesn't allow the access type). The kernel finds this by walking the VMA tree — O(log N).

---

### Q160. What is CPU frequency scaling and why does it matter for benchmarks?

**Topic Mapping:** OS → Power Management → CPU Frequency

**Explanation:**
Modern CPUs run at variable frequency:
- **Base clock:** 2.5 GHz (guaranteed sustainable rate)  
- **Turbo Boost:** Up to 4.5 GHz (if power/thermal budget allows, single-core burst)

Frequency varies with: number of active cores, temperature, power budget, workload type.

**Impact on benchmarks:** Variable frequency → non-reproducible timings. Same benchmark run at base clock vs turbo can differ 40%.

**Lock frequency for reproducible benchmarks:**
```bash
# Set performance governor (max frequency always)
echo performance | tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor

# Disable Intel Turbo Boost
echo 1 > /sys/devices/system/cpu/intel_pstate/no_turbo

# Disable HyperThreading (if testing single-threaded code)
echo 0 > /sys/devices/system/cpu/cpu1/online   # offline HT sibling
```

**Interview Tip:** Serious performance engineers always lock CPU frequency before benchmarking. Without it, results are noise.

---

### Q161. What is the Linux per-CPU variable mechanism?

**Topic Mapping:** OS → Concurrency → Per-CPU Variables

**Explanation:**
Shared global counters require atomic operations (expensive: cache-line bouncing between all CPUs). Per-CPU variables give each CPU its own copy — no sharing, no atomics needed.

```c
DEFINE_PER_CPU(long, packet_count);

// Increment on current CPU (preemption disabled automatically):
this_cpu_inc(packet_count);

// Read another CPU's value:
long val = per_cpu(packet_count, cpu_id);

// Sum across all CPUs:
long total = 0;
for_each_possible_cpu(cpu)
    total += per_cpu(packet_count, cpu);
```

**Performance:** ~10× faster than `atomic64_inc()` for per-CPU counters. Used throughout the Linux kernel for stats, scheduler data, SLUB allocator caches.

**Preemption must be disabled** when accessing per-CPU data — if preempted and migrated to another CPU mid-access, you'd be reading/writing the wrong CPU's copy. `this_cpu_*` macros handle this automatically.

---

### Q162. What is a tickless kernel (NOHZ) and what are its benefits?

**Topic Mapping:** OS → Power → Tickless Kernel

**Explanation:**
Traditional kernel: timer fires 1000×/second (HZ=1000) → wakes CPU, runs scheduler, updates `jiffies` — even on idle CPUs.

**NOHZ (No HZ / tickless idle):**
When a CPU has no runnable tasks, stop its periodic timer → CPU enters deep C-state sleep → significant power saving.

**NOHZ_FULL (full tickless):**
Even CPUs with exactly one runnable task can suppress ticks (no need to run scheduler if only one task). Used in HPC and real-time workloads to eliminate jitter from periodic timer interrupts.

```bash
# Kernel boot parameter to enable NOHZ_FULL on CPUs 1-7:
nohz_full=1-7 rcu_nocbs=1-7
```

**Power savings:** 10–30% on lightly loaded servers and laptops. Critical for battery life on mobile.

**Interview Tip:** Cloud providers benefit enormously from NOHZ — idle VM vCPUs that don't interrupt the host CPU save significant energy across large fleets.

---

### Q163. What is the difference between synchronous and asynchronous I/O?

**Topic Mapping:** OS → I/O Models

**Explanation:**

| Model | Syscall returns | Thread state during I/O |
|-------|-----------------|------------------------|
| Blocking I/O | After data ready | Blocked (TASK_INTERRUPTIBLE) |
| Non-blocking I/O | Immediately (EAGAIN if not ready) | Running (must poll) |
| I/O Multiplexing (`epoll`) | When any FD ready | Blocked in epoll_wait |
| Async I/O (`io_uring`) | Immediately | Running (CQ polled later) |

**epoll scalability:** O(1) per ready event (vs O(N) for `select`/`poll`). Nginx, Node.js (libuv), Redis all use epoll. The "C10K problem" was solved by epoll.

**Level-triggered vs Edge-triggered epoll:**
- **LT (default):** Notify while data available — keep notifying until consumed.  
- **ET:** Notify only on new data arrival — must consume all data immediately or miss it. Slightly faster but harder to use correctly.

**Interview Tip:** Node.js event loop uses epoll (via libuv). One thread handles 10K connections because I/O events are multiplexed, not one-thread-per-connection.

---

### Q164. How does Linux implement the `epoll` mechanism internally?

**Topic Mapping:** OS → I/O → epoll internals

**Explanation:**

`epoll_create()` → creates `eventpoll` struct with:
- **Red-black tree** of monitored FDs (for O(log N) add/remove).
- **Linked list of ready events** (returned by `epoll_wait`).

`epoll_ctl(ADD, fd)`:
- Insert FD into red-black tree.
- Register a **wait queue callback** on the file's internal wait queue.

**When FD becomes ready** (data arrives on socket):
- Network stack wakes the file's wait queue.
- Callback fires → adds FD's `epitem` to the ready list.

`epoll_wait()`:
- If ready list non-empty → return events immediately.
- Else → sleep until callback adds to ready list.

**Why O(1) per event:** The callback fires only for FDs that actually become ready. Sleeping processes don't scan all FDs — kernel pushes events to them.

**Interview Tip:** This is the key insight: epoll is event-driven (kernel pushes). select/poll is polling (user pulls through all FDs every call).

---

### Q165. What is the copy_on_write mechanism for page tables themselves?

**Topic Mapping:** OS → Memory Management → Page Table COW

**Explanation:**
After `fork()`, the child gets a copy of the parent's page table entries — but the page tables themselves are new copies (not shared). Only the physical pages they point to are shared (COW).

**Why not share page tables?** When one process modifies its page table (COW fault, new mmap), it would incorrectly affect the other process's mappings.

**Memory overhead of fork():**
- Physical pages: shared (COW) → O(1) overhead initially.
- Page tables: copied → O(virtual address space size / page table coverage).
- For a process with 4 GB virtual space: 4-level page table copy = a few MB.

**`vfork()` optimisation:** Shares even the page tables — parent suspended until child execs. Not COW — child literally runs in parent's address space. Dangerous but used for performance in `posix_spawn` implementations.

---

### Q166. What is the Linux OOM killer's interaction with cgroups?

**Topic Mapping:** OS → Memory Management → cgroup OOM

**Explanation:**
In cgroup v1/v2, each cgroup can have a memory limit. When a cgroup exceeds its limit, OOM killing is scoped to that cgroup — only processes within the cgroup are candidates for killing.

```bash
# cgroup v2: set 512 MB limit
echo 536870912 > /sys/fs/cgroup/mygroup/memory.max
```

**OOM kill sequence for cgroup:**
1. cgroup allocation exceeds `memory.max`.
2. Kernel tries to reclaim pages from cgroup's file cache.
3. If reclaim insufficient → invoke cgroup-scoped OOM killer.
4. Selects victim from cgroup's process list (same badness algorithm).
5. Sends SIGKILL → process freed → allocation may succeed.

**Kubernetes:** Each pod has its own memory cgroup. `limits.memory: 512Mi` → `memory.max = 512MB`. Pod's container exceeds limit → cgroup OOM → container restarted with `OOMKilled` reason.

**Interview Tip:** Kubernetes `OOMKilled` = cgroup memory limit exceeded. NOT host-level OOM. Fix: increase `limits.memory` or fix memory leak.

---

### Q167. What is the write-back cache and pdflush/writeback?

**Topic Mapping:** OS → File Systems → Write-back

**Explanation:**
Linux uses write-back caching: writes go to page cache first (fast), flushed to disk asynchronously by `wb_workfn` (formerly pdflush).

**Key tunables:**
```bash
/proc/sys/vm/dirty_ratio           # % of RAM: throttle writes at this dirty %
/proc/sys/vm/dirty_background_ratio # % of RAM: start background writeback here
/proc/sys/vm/dirty_expire_centisecs # pages older than this get written back
/proc/sys/vm/dirty_writeback_centisecs # writeback thread wakeup interval
```

**Typical defaults:** background_ratio=10%, ratio=20%, expire=3000 cs (30 s).

**`fsync()`:** Forces all dirty pages of an FD to disk before returning. Used by databases for durability. `fdatasync()`: syncs data only (not metadata timestamps) — slightly faster.

**Data loss window:** With write-back, up to `dirty_expire_centisecs` of writes can be lost on sudden power failure. Databases bypass page cache with `O_DIRECT | O_SYNC` for WAL writes.

---

### Q168. What is POSIX and why does it matter?

**Topic Mapping:** OS → Standards → POSIX

**Explanation:**
POSIX (Portable Operating System Interface) is an IEEE standard defining the API for Unix-like systems: process management (`fork`, `exec`, `wait`), file I/O, threads (`pthread`), signals, IPC, and more.

**Why it matters:**
- Programs written to POSIX compile and run on Linux, macOS, FreeBSD, Solaris with minimal changes.
- Portability for system software: databases, web servers, language runtimes.
- Contract between OS and application: OS must implement these semantics.

**Linux POSIX compliance:** Nearly complete. Some extensions (epoll, inotify, io_uring) are Linux-specific.

**Windows:** Not POSIX. WSL2 (Windows Subsystem for Linux 2) runs a real Linux kernel inside a lightweight VM → full POSIX compliance within WSL2 environment.

**Interview Tip:** When asked "why is your code platform-specific?" — check if you're using Linux-only extensions (epoll, signalfd, inotify). POSIX alternatives exist for each (select/poll, signals, stat polling).

---

### Q169. What is kernel bypass networking (DPDK)?

**Topic Mapping:** OS → Networking → Kernel Bypass

**Explanation:**
Traditional network path: NIC → kernel interrupt → kernel TCP/IP stack → syscall → user-space app. Each packet: 2 context switches, multiple memory copies, ~10 µs latency.

**DPDK (Data Plane Development Kit):**
- User-space PMD (Poll Mode Driver) talks to NIC directly via MMIO.
- No interrupts — CPU polls NIC ring buffer continuously (busy-wait).
- No kernel involvement — no syscalls, no context switches.
- Packets processed in user space: 10–100M packets/second per core.

**Trade-offs:**
- Dedicated CPU core spinning at 100% even at low load (busy poll).
- Complex programming model (no sockets API).
- Only for specialised use: telecom, CDN, high-frequency trading, OVS-DPDK.

**Alternatives:** XDP (eBPF at NIC driver level, stays in kernel but bypasses full stack), AF_XDP (zero-copy to user space via eBPF).

**Interview Tip:** DPDK is overkill for most applications. XDP/AF_XDP are the modern middle ground — high performance without dedicating entire cores to polling.

---

### Q170. What is a memory model and why does it matter for concurrent code?

**Topic Mapping:** OS → Concurrency → Memory Models

**Explanation:**
A memory model defines the ordering guarantees between memory operations across CPUs/threads. CPUs and compilers reorder operations for performance — the memory model defines what reorderings are legal.

**x86 TSO (Total Store Order) — strong model:**
- Reads not reordered past reads.
- Writes not reordered past writes.
- Only StoreLoad reordering allowed (write may not be visible to other CPUs immediately).
- x86 rarely needs explicit barriers in practice.

**ARM / POWER — weak model:**
- Almost any reordering allowed.
- Requires explicit memory barriers (`dmb`, `dsb`) for correct multi-threaded code.

**C++11 memory model:**
`std::atomic<T>` with memory orders:
- `memory_order_seq_cst` — sequential consistency (default, safest, most expensive).
- `memory_order_acquire/release` — pairs for synchronisation (cheaper).
- `memory_order_relaxed` — no ordering guarantees (only atomicity).

**Interview Tip:** Java `volatile` guarantees visibility but not compound atomicity. C++ `volatile` guarantees neither visibility nor atomicity — don't use for threading. Use `std::atomic`.

---

### Q171. What is the Linux scheduler's handling of real-time tasks?

**Topic Mapping:** OS → Scheduling → Real-Time Scheduling

**Explanation:**
Linux has two real-time scheduling policies that bypass CFS entirely:

**SCHED_FIFO:**
- No time quantum — runs until it blocks or a higher-priority RT task preempts.
- Priority 1–99 (higher = more urgent).
- Starvation possible: a SCHED_FIFO task at priority 99 blocks all others forever if it never sleeps.

**SCHED_RR (Round Robin RT):**
- Like SCHED_FIFO but with a time quantum.
- Same-priority tasks round-robin; higher priority preempts immediately.

**Setting RT priority:**
```bash
chrt -f 50 ./realtime_app     # SCHED_FIFO, priority 50
chrt -p -f 50 $PID            # change existing process
```

**RT throttling:** `kernel.sched_rt_runtime_us` (default 950ms per 1000ms) — RT tasks can only consume 95% of CPU. Prevents a buggy RT task from hard-locking the system.

**CFS normal tasks:** Priority 0 (below all RT tasks).

---

### Q172. What is the difference between a process group, session, and controlling terminal?

**Topic Mapping:** OS → Process Management → Groups and Sessions

**Explanation:**
- **Process group:** Set of related processes sharing a PGID. Shell pipeline = one process group. `kill(-pgid, sig)` sends signal to entire group.
- **Session:** Set of process groups created by `setsid()`. Has one session leader (SID = leader's PID).
- **Controlling terminal:** First terminal opened by session leader. SIGHUP sent to foreground group when terminal disconnects (SSH session ends).
- **Foreground process group:** Gets keyboard signals. Ctrl+C → SIGINT → entire foreground group. Ctrl+Z → SIGTSTP → stop entire group.
- **Background groups:** Don't receive keyboard signals. `bg` / `fg` shell commands move groups.

**`nohup ./app &`:** Ignores SIGHUP + redirects stdout/stderr — process survives terminal disconnect.

---

### Q173. What is an anonymous mapping vs a file-backed mapping?

**Topic Mapping:** OS → Memory Management → mmap Types

**Explanation:**

**File-backed mapping:** `mmap(fd, ...)` — VMA backed by a file. Pages loaded from file on fault. Modified pages written back to file (if MAP_SHARED).

**Anonymous mapping:** `mmap(NULL, size, PROT_READ|PROT_WRITE, MAP_ANONYMOUS|MAP_PRIVATE, -1, 0)` — VMA backed by nothing (swap if needed). Pages initialised to zero (from zero page via COW). Used for: heap (`brk`/`sbrk` uses it internally), thread stacks, `malloc` for large allocations.

**MAP_SHARED vs MAP_PRIVATE:**
- MAP_SHARED: writes visible to other processes mapping same file/region; written back to file.
- MAP_PRIVATE: COW — writes create private copies, not visible to others, not written to file.

**Anonymous MAP_SHARED:** Used for shared memory IPC between related processes (after fork). Changes visible to all sharers but not backed by any file.

---

### Q174. What is the slab allocator and why is it better than raw buddy allocation for kernel objects?

**Topic Mapping:** OS → Memory Management → Slab

**Explanation:**
Buddy system minimum allocation = 1 page = 4096 bytes. A kernel `inode` is 328 bytes — using a full page wastes 3768 bytes (91% waste).

**Slab allocator (Bonwick 1994):**
1. One slab = one or more pages carved into fixed-size object slots.
2. Object caches per type: `inode_cache`, `task_struct_cache`, `dentry_cache`.
3. Free list within slab — allocation = pop from free list → O(1).
4. Objects returned to slab on free — kept initialised (constructor already ran → faster reuse).
5. Full slabs → return pages to buddy; partial slabs → serve next allocation.

**SLUB (modern Linux default):** Simplified slab with less per-object metadata overhead. Per-CPU partial lists reduce lock contention.

**Benefits:**
- O(1) allocation/free.
- Zero fragmentation for fixed-size objects.
- Cache-warm objects (same type together in same cache lines).
- Objects kept initialised → constructor amortised.

---

### Q175. What is zRAM and when is it used?

**Topic Mapping:** OS → Memory Management → zRAM

**Explanation:**
zRAM creates a virtual block device backed by compressed RAM. Used as a swap device — instead of swapping to slow disk, pages are compressed and stored in RAM itself.

```bash
modprobe zram
echo 2G > /sys/block/zram0/disksize    # 2 GB compressed pool
mkswap /dev/zram0
swapon -p 100 /dev/zram0               # high priority swap
```

**Compression ratio:** LZ4 typically 3–5×. Effective: 2 GB zRAM pool ≈ 6–10 GB of swappable space.

**Used by:** Android (primary swap mechanism since Android 4.4), ChromeOS, embedded Linux, low-RAM servers.

**Trade-off:** CPU cost for compression/decompression vs disk I/O latency. For memory-pressured systems without fast NVMe, zRAM is almost always better than no swap.

---

### Q176. What is AddressSanitizer (ASan) and how does it detect memory bugs?

**Topic Mapping:** OS → Memory Safety → ASan

**Explanation:**
ASan instruments every memory access at compile time and uses shadow memory to track validity.

**Shadow memory:** 1 shadow byte per 8 application bytes. Shadow value:
- 0 → all 8 bytes accessible.
- 1–7 → first N bytes accessible.
- Negative values → poisoned (freed, redzone, stack guard, etc.).

**On every load/store:**
```c
// Compiler inserts before every access:
if (shadow_byte(addr) != 0) report_error();
```

**What it detects:**
- Heap buffer overflow / underflow.
- Stack buffer overflow.
- Use-after-free (freed memory is re-poisoned).
- Use-after-return (stack frame poisoned after return).
- Global buffer overflow.
- Memory leaks (with LeakSanitizer).

```bash
gcc -fsanitize=address,undefined -fno-omit-frame-pointer -g ./app
```

**Overhead:** ~2× slowdown, ~2× memory. Acceptable for testing; too slow for production.

---

### Q177. What is the role of `initramfs` in the Linux boot process?

**Topic Mapping:** OS → Boot → initramfs

**Explanation:**
`initramfs` (initial RAM filesystem) is a compressed archive (cpio + gzip/lz4) embedded in the kernel image or loaded separately by the bootloader.

**Boot sequence:**
```
BIOS/UEFI → GRUB → loads vmlinuz + initramfs into RAM
→ kernel decompresses initramfs → mounts it as rootfs (tmpfs)
→ runs /init (or /sbin/init) from initramfs
→ initramfs loads drivers (disk, filesystem, LVM, LUKS)
→ mounts real root filesystem
→ pivots root (chroot to real /)
→ execs /sbin/init (PID 1 = systemd)
```

**Why needed?**
The kernel may not have built-in drivers for the disk controller or filesystem containing the real root. initramfs provides them as loadable modules.

**Also handles:**
- LUKS encrypted root (asks for passphrase).
- LVM volume activation.
- RAID assembly.
- Network boot (iSCSI, NFS root).

---

### Q178. What is kernel live patching (kpatch / livepatch)?

**Topic Mapping:** OS → Reliability → Live Patching

**Explanation:**
Live patching applies security fixes to a running kernel without rebooting — critical for high-availability systems.

**How it works:**
1. Generate patch: compile fixed kernel function → produce a `.ko` module with only changed functions.
2. Load module → uses `ftrace` to redirect calls to original function → trampolines to patched version.
3. Must ensure no active callers of original function at the moment of redirect (uses `stop_machine` or per-CPU patching with safe points).

**Linux livepatch infrastructure** (upstream since 4.0):
- `klp_patch` structure registers new function implementations.
- `klp_enable_patch()` atomically switches callers.

**Users:** Red Hat (kpatch), SUSE (kGraft), Canonical (Livepatch), Oracle (Ksplice — oldest). Cloud providers patch hypervisors without customer-visible maintenance windows.

---

### Q179. What is the difference between `volatile`, `atomic`, and memory barriers in C?

**Topic Mapping:** OS → Concurrency → Memory Ordering

**Explanation:**

**`volatile` (C/C++):**
- Prevents compiler from caching the value in a register.
- Forces every read/write to go to memory.
- Does NOT prevent CPU reordering.
- Does NOT ensure visibility across threads.
- Use for: MMIO registers, setjmp/longjmp, `sig_atomic_t`. NOT for threading.

**`std::atomic<T>` (C++11):**
- Prevents compiler caching AND generates appropriate CPU memory barriers.
- Default `memory_order_seq_cst`: full sequential consistency — all CPUs see same order.
- Can use weaker orders (`acquire/release`, `relaxed`) for performance.
- Correct for multi-threaded access.

**Memory barriers (explicit):**
```c
__sync_synchronize();          // full barrier (GCC built-in)
asm volatile("mfence" ::: "memory");  // x86 full barrier
smp_mb();                      // Linux kernel full barrier
```

**Java `volatile`:** Stronger than C `volatile` — provides visibility guarantee (happens-before). But still not atomic for compound operations (`i++`).

---

### Q180. What is the difference between `futex` and a traditional mutex?

**Topic Mapping:** OS → Synchronization → Futex

**Explanation:**
A traditional mutex always makes a syscall to lock/unlock (expensive: ~1 µs per call). A futex (Fast Userspace muTEX) is fast in the common uncontended case.

**Futex fast path (no syscall):**
```c
// Lock: atomically set 0 → 1 (CAS)
if (atomic_cmp_xchg(futex_word, 0, 1) == 0)
    return;  // got lock, zero syscalls!

// Unlock: atomically set 1 → 0
if (atomic_xchg(futex_word, 0) == 1)
    return;  // released, no waiters, zero syscalls!
```

**Futex slow path (syscall only when contended):**
```c
// Lock failed: syscall to sleep
futex(futex_word, FUTEX_WAIT, 1, NULL);   // block until word != 1

// Unlock with waiters: syscall to wake
futex(futex_word, FUTEX_WAKE, 1);          // wake 1 waiter
```

**Result:** 99%+ of lock/unlock operations → zero syscalls. Only contention → syscall.

All `pthreads` mutex, Java `synchronized`, Go `sync.Mutex`, Python `threading.Lock` use futex internally on Linux.

---

### Q181. What is the process scheduler's `load_balance()` and when does it run?

**Topic Mapping:** OS → Scheduling → Load Balancing

**Explanation:**
Load balancing ensures runnable tasks are distributed across CPUs to prevent idle CPUs while others are overloaded.

**Triggers:**
1. **Periodic:** `scheduler_tick()` fires every 1 ms → checks if current CPU's run queue is imbalanced.
2. **Idle:** CPU becomes idle → `idle_balance()` steals tasks immediately.
3. **Newly runnable task:** Task woken on CPU A → may be migrated to less-loaded CPU B (`select_task_rq_fair()`).

**Migration decision:**
Calculate "load" per CPU (sum of task weights in run queue). If imbalance > threshold: pull tasks from busiest CPU in scheduling domain.

**`push` vs `pull`:**
- Idle CPU pulls from busy ones (common path).
- Very busy CPU can push tasks to idle ones.

**Cache affinity hysteresis:** Scheduler won't migrate unless imbalance is significant (avoids destroying cache warmth for tiny load differences).

---

### Q182. What is the difference between `kfree()` and `vfree()` in the kernel?

**Topic Mapping:** OS → Memory Management → Kernel Heap

**Explanation:**

| Function | Allocator | Virtual memory | Max size |
|----------|-----------|---------------|----------|
| `kmalloc()` / `kfree()` | Slab | Physically contiguous + virtually contiguous | ~4 MB |
| `vmalloc()` / `vfree()` | Buddy (per page) | Virtually contiguous, NOT physically | Limited by vmalloc space |
| `get_free_pages()` / `free_pages()` | Buddy | Physically + virtually contiguous | ~4 MB (order 10) |

**`kmalloc`:** Fast (slab), physically contiguous (needed for DMA). Use for small, DMA-able allocations.

**`vmalloc`:** Slower (must set up page table mappings). Pages scattered physically. Use for large allocations that don't need physical contiguity (module code, large data structures).

**`GFP_KERNEL` vs `GFP_ATOMIC`:**
- `GFP_KERNEL`: may sleep (reclaim, compact) — use in process context.
- `GFP_ATOMIC`: never sleeps — use in interrupt context or with spinlocks held.

---

### Q183. Explain the ext4 journal and how it ensures consistency.

**Topic Mapping:** OS → File Systems → ext4 Journal

**Explanation:**
ext4 uses a journal (log) to ensure filesystem consistency after crashes. Before modifying filesystem metadata on disk, changes are written to the journal first (write-ahead logging).

**Journal modes:**

| Mode | What's journaled | Safety | Performance |
|------|-----------------|--------|-------------|
| `data=writeback` | Metadata only | Data may be stale after crash | Fastest |
| `data=ordered` (default) | Metadata; data written first | Data consistent | Good |
| `data=journal` | Both data and metadata | Safest | Slowest (double write) |

**Journal commit cycle:**
1. Collect transactions into journal block groups.
2. Write journal descriptor + data → disk.
3. Write journal commit block → disk.
4. Mark journal as committed.
5. Checkpoint: eventually write actual filesystem changes → disk; then journal space freed.

**Recovery after crash:** Replay all committed but not checkpointed journal entries. Incomplete (no commit block) → discard. Result: always consistent metadata.

---

### Q184. What is the difference between `sendfile()` and traditional `read()` + `write()`?

**Topic Mapping:** OS → I/O → Zero-Copy

**Explanation:**
**Traditional file-to-socket copy:**
```
disk → DMA → page cache → CPU copies → socket buffer → NIC
```
Two copies: page cache → user buffer (`read()`), user buffer → socket buffer (`write()`).

**`sendfile()`:**
```
disk → DMA → page cache → (kernel internal transfer) → socket buffer → NIC
```
Zero copies to user space. Page cache → socket buffer happens entirely in kernel.
Two fewer context switches and one fewer memory copy.

```c
sendfile(socket_fd, file_fd, &offset, count);
```

**Who uses it:** Nginx (`sendfile on;` in config), Apache, Lighttpd, Redis AOF replication. Static file serving is dramatically faster.

**`splice()` (more general):** Moves data between two file descriptors using kernel pipe buffers. No user-space involvement. Basis for more complex zero-copy pipelines.

---

### Q185. What is the difference between `read()` consistency and `fsync()` durability?

**Topic Mapping:** OS → File Systems → Consistency vs Durability

**Explanation:**

**Consistency:** Does a `read()` after a `write()` return the written data? Yes — always. Even before fsync, subsequent reads from the same process (or any process) see the written data via page cache.

**Durability:** Is the data on persistent storage? Not until `fsync()` called. Written data sits in page cache (volatile RAM) until synced.

```
write() → page cache (consistent, not durable)
fsync() → page cache + disk (consistent + durable)
```

**Database WAL pattern:**
1. Write to WAL buffer in memory.
2. `fsync(wal_fd)` → WAL on disk (durable).
3. Acknowledge transaction to client.
4. Later: checkpoint writes actual data pages.

**`O_SYNC`:** Every `write()` is implicitly fsynced — very slow. `O_DSYNC`: syncs data but not metadata (timestamps) — slightly faster.

**Interview Tip:** "Is my write durable?" → Only after fsync. Many "lost update" bugs in production are actually "we forgot to fsync the WAL."

---

### Q186. What is Linux namespaces isolation model?

**Topic Mapping:** OS → Virtualization → Namespaces

**Explanation:**
Linux namespaces give processes an isolated view of global resources:

| Namespace | Isolates | Created with |
|-----------|----------|-------------|
| PID | Process IDs — PID 1 inside container | `CLONE_NEWPID` |
| Network | Interfaces, routes, ports, iptables | `CLONE_NEWNET` |
| Mount | Filesystem tree | `CLONE_NEWNS` |
| IPC | System V IPC, POSIX MQs | `CLONE_NEWIPC` |
| UTS | Hostname, NIS domain | `CLONE_NEWUTS` |
| User | UIDs/GIDs — root inside ≠ root outside | `CLONE_NEWUSER` |
| Time | System clock offsets | `CLONE_NEWTIME` (Linux 5.6) |

**Container = namespaces + cgroups + overlay FS.** No hypervisor, no hardware emulation. Just kernel-level isolation.

**Docker under the hood:**
```bash
unshare --pid --net --mount --ipc --uts --fork bash
# Now in new PID+net+mount+IPC+UTS namespace
```

**Interview Tip:** Containers are NOT VMs. VMs emulate hardware; containers are processes with namespace isolation. Much lighter — container starts in milliseconds; VM takes seconds.

---

### Q187. What is huge page fragmentation and how is it prevented?

**Topic Mapping:** OS → Memory Management → Huge Page Fragmentation

**Explanation:**
Huge page allocation requires 512 contiguous 4 KB frames in the buddy system. After months of uptime with many small allocations, free frames are scattered → no 512-contiguous block available → huge page allocation fails.

**Prevention strategies:**

1. **Reserve at boot:** `hugepages=1024` kernel parameter allocates before fragmentation begins. Memory reserved permanently.
2. **ZONE_MOVABLE:** All movable pages in a dedicated zone; compaction in that zone is efficient.
3. **Memory compaction:** `kcompactd` or `echo 1 > /proc/sys/vm/compact_memory` migrates movable pages to consolidate free space.
4. **THP `defer` mode:** THP tries compaction before giving up — `echo defer > /sys/kernel/mm/transparent_hugepage/defrag`.

**Production reality:** Long-running databases often lose access to huge pages after weeks. Scheduled maintenance window to compact or reboot is common.

---

### Q188. What is the Linux `procfs` and `sysfs` interface?

**Topic Mapping:** OS → Interface → /proc and /sys

**Explanation:**

**/proc (procfs):**
- Virtual FS — no disk storage. Generated by kernel on read.
- Per-process info: `/proc/<pid>/status`, `maps`, `fd/`, `cmdline`.
- System info: `/proc/cpuinfo`, `meminfo`, `interrupts`, `net/tcp`.
- Writable for configuration: `echo 1 > /proc/sys/net/ipv4/ip_forward`.

**/sys (sysfs):**
- Exports kernel object model: devices, drivers, buses, modules.
- Structured: `/sys/class/net/eth0/`, `/sys/block/sda/queue/`.
- Used by udev for hotplug, by admin tools for tuning.
- Writable: `echo mq-deadline > /sys/block/nvme0n1/queue/scheduler`.

**Key difference:**
- `/proc`: process info + kernel parameters. Historical, somewhat unstructured.
- `/sys`: device/driver model. Structured, one value per file. Modern.

**Interview Tip:** `ps`, `top`, `netstat`, `ss`, `lsblk` all read from `/proc` and `/sys`. Understanding them = understanding how Linux monitoring tools work.

---

### Q189. What is cgroup v2 and how does it differ from v1?

**Topic Mapping:** OS → Virtualization → cgroups v2

**Explanation:**

**cgroup v1 problems:**
- Multiple independent hierarchies (one per subsystem: memory, cpu, blkio separately).
- Subsystems could conflict — process in memory-cgroup-A and cpu-cgroup-B.
- Thread-level grouping inconsistent.

**cgroup v2 (unified hierarchy):**
- Single hierarchy for all resource controllers.
- Process can only be in one cgroup (leaf node).
- Thread-aware: `cgroup.threads` for thread-level control.
- Better delegation: user namespaces + cgroup namespaces allow non-root to manage sub-hierarchies.

**Key controllers in v2:**
- `cpu.weight` (relative share), `cpu.max` (hard quota)
- `memory.max` (hard limit), `memory.high` (soft limit — triggers reclaim)
- `io.max` (per-device IOPS/bandwidth limits)
- `pids.max` (fork bomb protection)

**Kubernetes:** Requires cgroup v2 for features like memory QoS (`memory.high` for guaranteed pods). Enable with `systemd.unified_cgroup_hierarchy=1` boot parameter.

---

### Q190. What is the difference between `wait()`, `waitpid()`, and `waitid()`?

**Topic Mapping:** OS → Process Management → Wait Syscalls

**Explanation:**
Parent must collect exit status of dead children — otherwise they remain as zombies.

**`wait(int *status)`:**
- Blocks until ANY child terminates.
- Returns PID of dead child and its exit status.
- Simple but can't choose which child to wait for.

**`waitpid(pid_t pid, int *status, int options)`:**
- `pid > 0`: wait for specific child PID.
- `pid = -1`: wait for any child (like `wait()`).
- `pid = 0`: any child in same process group.
- `WNOHANG` option: non-blocking — returns 0 immediately if no child exited.
- `WUNTRACED`: also return for stopped children (SIGSTOP).

**`waitid(idtype, id, siginfo, options)`:**
- More detailed info (exact signal that killed child, resource usage).
- `WNOWAIT`: peek at exit status without consuming it.

**Signal handler pattern:**
```c
signal(SIGCHLD, sigchld_handler);
void sigchld_handler(int sig) {
    while (waitpid(-1, NULL, WNOHANG) > 0);  // reap all dead children
}
```

---

### Q191. What is the buddy system's interaction with NUMA?

**Topic Mapping:** OS → Memory Management → NUMA + Buddy

**Explanation:**
Each NUMA node has its own buddy allocator with its own free lists. When kernel allocates memory, it prefers the local node's buddy allocator.

**Allocation order:**
1. Try current NUMA node's buddy (fast, local).
2. If insufficient: try adjacent nodes (slower, remote access).
3. Configured by NUMA policy (`MPOL_BIND`, `MPOL_PREFERRED`, `MPOL_INTERLEAVE`).

**`numactl --membind=0` enforces:** Only allocate from node 0 — fails with OOM rather than allocating remotely.

**`numactl --interleave=all`:** Round-robin allocation across nodes — good for workloads where memory is accessed from all CPUs equally (avoids hot-node bottleneck).

**Kernel zones per NUMA node:** Each node has ZONE_NORMAL, ZONE_MOVABLE etc. Node-local zone preferred always.

---

### Q192. What is `ptrace` and how do debuggers use it?

**Topic Mapping:** OS → Process Management → ptrace

**Explanation:**
`ptrace` is the syscall that lets one process (tracer) observe and control another (tracee). Foundation of `gdb`, `strace`, `ltrace`.

**Key operations:**
- `PTRACE_ATTACH`: attach to running process (sends SIGSTOP).
- `PTRACE_PEEKDATA / POKEDATA`: read/write tracee's memory.
- `PTRACE_GETREGS / SETREGS`: read/write CPU registers.
- `PTRACE_SINGLESTEP`: execute one instruction, then stop.
- `PTRACE_SYSCALL`: stop at each syscall entry/exit (used by strace).
- `PTRACE_CONT`: resume tracee.

**How GDB sets a breakpoint:**
1. `PTRACE_PEEKDATA`: read instruction at target address.
2. Save original byte.
3. `PTRACE_POKEDATA`: write `0xCC` (INT3) at address.
4. Tracee hits INT3 → SIGTRAP → GDB notified.
5. GDB restores original byte → let user inspect state → `PTRACE_SINGLESTEP` → re-insert breakpoint → `PTRACE_CONT`.

**Security:** Restricted by `YAMA` LSM (`ptrace_scope`). Docker containers: ptrace of host processes blocked by default.

---

### Q193. What is the `select()` impedance mismatch and why epoll solves it?

**Topic Mapping:** OS → I/O → Scalability

**Explanation:**
**`select()` / `poll()` problem — O(N) per call:**
1. User builds FD set of N monitored FDs.
2. Syscall: kernel iterates all N FDs checking readiness.
3. Returns FD set with ready bits.
4. User iterates N FDs again to find which are ready.
5. Next call: repeat from step 1 — kernel iterates all N again.

For 10,000 connections with 10 active: 10,000 FDs checked every call even though only 10 are ready. O(N) wasted work.

**epoll — O(1) per ready event:**
- FDs registered once (red-black tree, O(log N)).
- Kernel sets up callbacks on registered FDs.
- `epoll_wait()` blocks until any FD fires callback → event added to ready list.
- Returns ONLY ready events — no scanning.

For 10,000 connections with 10 active: `epoll_wait()` returns 10 events directly. O(10) not O(10,000).

---

### Q194. What is a spinlock and what is its implementation?

**Topic Mapping:** OS → Synchronization → Spinlock Implementation

**Explanation:**
A spinlock is a lock where the waiter busy-loops (spins) rather than sleeping.

**x86 implementation (simplified):**
```c
typedef struct { int locked; } spinlock_t;

void spin_lock(spinlock_t *lock) {
    while (!__sync_bool_compare_and_swap(&lock->locked, 0, 1)) {
        // Spin: try CAS(0→1) until succeeds
        // On x86: add PAUSE instruction to reduce power + avoid memory-order issues
        asm volatile("pause");
    }
}

void spin_unlock(spinlock_t *lock) {
    __atomic_store_n(&lock->locked, 0, __ATOMIC_RELEASE);
}
```

**`PAUSE` instruction:** Hints to CPU it's in a spin-wait loop. Reduces power consumption and avoids memory order violations from aggressive speculation. On Hyper-Threading: `PAUSE` yields the CPU to the HT sibling.

**Ticket spinlock (fair version):**
Like a deli counter — take a number on lock(), wait for your number to be called on unlock(). Prevents starvation — threads get lock in FIFO order.

**Linux queued spinlocks (qspinlock):** Even more scalable — uses a queue of waiting CPUs, each spinning on its own cache line (avoids all-CPUs hammering same lock cache line).

---

### Q195. What is the Linux OOM score and how to protect critical processes?

**Topic Mapping:** OS → Memory → OOM Protection

**Explanation:**
```bash
cat /proc/<pid>/oom_score       # current score 0–1000
cat /proc/<pid>/oom_score_adj   # adjustment -1000 to +1000
```

**Protecting critical processes:**
```bash
# Fully protect (never kill):
echo -1000 > /proc/$(pidof sshd)/oom_score_adj

# Make first target:
echo 1000 > /proc/$(pidof lowprio)/oom_score_adj
```

**systemd integration:**
```ini
# /etc/systemd/system/myservice.service
[Service]
OOMScoreAdjust=-900   # protect from OOM killer
```

**Kubernetes QoS classes → oom_score_adj:**
- Guaranteed: `-997` (protected)
- Burstable: proportional to limit/request ratio
- BestEffort: `1000` (killed first)

Always set `resources.requests` and `resources.limits` in Kubernetes — otherwise your pod is BestEffort and dies first on memory pressure.

---

### Q196. What is the difference between physical and virtual memory limits in Linux?

**Topic Mapping:** OS → Memory Management → Limits

**Explanation:**

**Physical memory limits:**
- Actual RAM installed: `/proc/meminfo` `MemTotal`.
- Swap space: `/proc/swaps`.
- Process RSS (Resident Set Size): how much physical RAM process currently uses. `ps aux` column `RSS`.

**Virtual memory limits:**
- Per-process: 128 TB on 64-bit Linux (47-bit virtual addresses, user half).
- `VIRT` in `top` = virtual address space size. Often much larger than RSS (unmapped pages, mmap'd files counted but not loaded).
- `ulimit -v` limits virtual address space per process.

**Key metrics:**
- `VIRT` (virtual) >> `RES` (physical in RAM) >> `SHR` (shared/file-backed).
- OOM killer looks at RSS + swap, not VIRT.
- Java heap: `Xmx4g` → 4 GB RSS max (once fully used). VIRT may be much larger (class metadata, code cache, JIT).

---

### Q197. What is `kswapd` and when does it run?

**Topic Mapping:** OS → Memory Management → kswapd

**Explanation:**
`kswapd` (one per NUMA node) is a kernel thread that performs background page reclaim to maintain a pool of free pages.

**Watermarks (per zone):**
```
Pages free:   high ──── low ──── min
              ↑ kswapd sleeps   ↑ kswapd wakes   ↑ direct reclaim
```

- **High watermark:** kswapd sleeps — enough free pages.
- **Low watermark:** kswapd wakes — starts reclaiming (LRU page eviction, swap-out).
- **Min watermark:** Direct reclaim — allocating process itself reclaims pages (very slow, high latency).

**What kswapd reclaims:**
1. Clean file-backed pages (just discard — can reload from disk).
2. Dirty file-backed pages (write back first).
3. Anonymous pages (write to swap).
4. Slab caches (inode/dentry caches shrink via shrinkers).

**`vm.swappiness`:** 0 = strongly prefer reclaiming file cache over swapping. 100 = swap equally. Default 60.

---

### Q198. What is the Linux `blk-mq` (multiqueue block layer)?

**Topic Mapping:** OS → I/O → Block Layer

**Explanation:**
Traditional Linux block layer: single request queue protected by a single lock → bottleneck for NVMe SSDs capable of millions of IOPS.

**blk-mq (multi-queue):** Introduced Linux 3.13. Per-CPU software queues + multiple hardware dispatch queues.

```
CPU 0 → SW Queue 0 ──┐
CPU 1 → SW Queue 1 ──┼──► HW Queue 0 → NVMe controller core 0
CPU 2 → SW Queue 2 ──┤    HW Queue 1 → NVMe controller core 1
CPU 3 → SW Queue 3 ──┘    HW Queue 2 → NVMe controller core 2
```

**Benefits:**
- Per-CPU SW queues: no lock contention between CPUs for request submission.
- Multiple HW queues: NVMe drives have 64K queues → fully parallel.
- Result: Linux can sustain millions of IOPS on NVMe (previously bottlenecked at ~200K on single-queue).

**All modern NVMe drivers use blk-mq.** SATA drives: blk-mq with single HW queue (hardware only has one queue anyway).

---

### Q199. What is the difference between `mprotect()` and `munmap()`?

**Topic Mapping:** OS → Memory Management → mprotect/munmap

**Explanation:**

**`mprotect(addr, len, prot)`:**
Changes protection flags of a virtual memory region. Does NOT unmap or free memory.
```c
mprotect(page, 4096, PROT_READ);          // make read-only
mprotect(page, 4096, PROT_NONE);          // make inaccessible (guard page)
mprotect(page, 4096, PROT_READ|PROT_WRITE|PROT_EXEC); // RWX (dangerous)
```
Use cases: stack guard pages, JIT code (mmap anonymous → PROT_WRITE for generation → PROT_READ|EXEC for execution — W^X policy).

**`munmap(addr, len)`:**
Removes the virtual memory mapping entirely. Pages are freed (or file mapping broken). Accessing unmapped address → SIGSEGV.
```c
void *mem = mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_ANONYMOUS|MAP_PRIVATE, -1, 0);
// use mem...
munmap(mem, 4096);   // release mapping; mem is now invalid
```

**`madvise(addr, len, advice)`:**
Hint to kernel about expected access pattern:
- `MADV_SEQUENTIAL`: prefetch ahead aggressively.
- `MADV_RANDOM`: disable prefetch.
- `MADV_FREE`: pages can be reclaimed (lazy free).
- `MADV_DONTNEED`: release physical pages immediately (keep VMA).

---

### Q200. How does the Linux kernel handle SMP (Symmetric Multi-Processing) boot?

**Topic Mapping:** OS → SMP → Boot

**Explanation:**
On multi-core/multi-socket machines:

**Boot sequence:**
1. One CPU (Bootstrap Processor, BSP) executes BIOS + bootloader + starts Linux kernel.
2. BSP initialises all kernel data structures (memory zones, IRQ handlers, scheduler).
3. BSP sends INIT + SIPI (Startup Inter-Processor Interrupt) via APIC to each Application Processor (AP).
4. Each AP starts executing at 16-bit real mode trampoline code (at a fixed low-memory address set by BSP).
5. AP switches to 32-bit protected mode → 64-bit long mode.
6. AP initialises its local APIC, TSS, per-CPU variables.
7. AP calls `cpu_startup_entry()` → enters idle loop → scheduler starts assigning tasks.

**`/proc/cpuinfo`:** Shows all CPUs that came online.

**CPU hotplug:** `echo 0 > /sys/devices/system/cpu/cpu3/online` takes CPU3 offline. Tasks migrated away, IRQs reassigned. `echo 1` to bring back.

**Interview Tip:** Knowing the SMP boot sequence (BSP + SIPI to APs) shows deep hardware/OS knowledge — relevant for embedded systems, hypervisor, and OS kernel interviews.

---

# ✅ OS SECTION COMPLETE — Q1 to Q200

