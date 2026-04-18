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

