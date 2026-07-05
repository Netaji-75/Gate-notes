# CPU Scheduling — Lecture 8 Notes (Threads + Scheduling Basics)

*GATE CS/IT — Principles of Operating Systems | Revision Blog*

---

## 📌 Topics Covered
1. Types of Threads (ULT vs KLT)
2. Multithreading Models
3. Threading Issues
4. Case Study: Process vs Thread Model
5. OS-specific Thread Implementations
6. Introduction to CPU Scheduling
7. Scheduling Criteria & Key Time Metrics
8. FCFS Scheduling Algorithm

---

## 1. Types of Threads

A **thread** is a lightweight process within a process — the smallest unit of CPU execution. There are two fundamental types:

| | User Level Thread (ULT) | Kernel Level Thread (KLT) |
|---|---|---|
| **Created/Managed by** | User-level thread library | Operating System (Kernel) |
| **OS Awareness** | OS doesn't recognize them — sees only 1 process | OS recognizes and schedules each thread |
| **Examples** | POSIX Pthreads, Win32 threads, Java threads | Windows XP/2000, Solaris, Linux, Tru64 UNIX, Mac OS X |

**Mental model:**
- ULTs live entirely in **User Mode (UM)** — the kernel only sees the process, not the individual threads.
- KLTs are created and tracked by the **Kernel Mode (KM)** — kernel maintains a Thread Control Block (TCB) per thread.

### Benefits of ULTs
1. **Flexibility**
2. **Transparency**
3. **Fastest context switching** (no user→kernel mode switch needed)

### Pros and Cons of ULTs

**Advantages:**
- Fast/lightweight — no system calls needed to manage threads (thread library handles everything)
- Can be implemented even on an OS that doesn't support threading
- Fast switching — no user-to-kernel mode transition

**Disadvantages:**
- **Scheduling issue**: if 1 thread blocks on I/O, the entire process (and all its threads) blocks — even if another thread was runnable
- **Lack of coordination** between kernel and threads (e.g., a process with 100 threads gets the same CPU timeslice as a process with just 1 thread)
- **Requires non-blocking system calls** — if one thread makes a blocking call, all threads must wait

### Pros and Cons of KLTs

**Pros:**
- Scheduler can give more CPU time to a process having fewer threads
- Great for applications that block frequently (one thread blocking doesn't block others)

**Cons:**
- Slower — every operation requires a kernel invocation (system call overhead)
- Kernel overhead — must manage & schedule threads *and* processes; needs a full **TCB** per thread

### 🔑 Complete Comparison Table

| Parameter | User Level Thread | Kernel Level Thread |
|---|---|---|
| Implemented by | Users / thread library | Operating System |
| Recognized by OS? | No | Yes |
| Implementation | Easy | Complicated |
| Context switch time | Less (faster) | More (slower) |
| Hardware support needed | No | Yes |
| Blocking operation | Blocks entire process | Other threads can continue |
| Multithreading across CPUs | Not possible | Possible (kernel distributes threads) |
| Creation/Management | Quick | Slower |
| OS dependency | Any OS can support | OS-specific |
| Examples | Java threads, POSIX threads | Windows, Solaris |

> 💡 **Exam tip:** The one-line differentiator to remember — *"If one ULT blocks, the whole process blocks. If one KLT blocks, other KLTs of the same process can still run."*

---

## 2. Multithreading Models

These models describe how **ULTs map to KLTs**:

### (a) Many-to-One
- Many user threads map to **one** kernel thread.
- Thread management done in user space → very efficient.
- ❌ **Flaw**: if one user thread makes a blocking system call, the *entire process* blocks (since there's only 1 KLT).
- ❌ Cannot take advantage of multiprocessing (only 1 kernel thread = only 1 CPU core usable at a time).

### (b) One-to-One
- Each user thread maps to its **own** kernel thread.
- ✅ True concurrency — threads can run on different processors.
- ✅ If one thread blocks, others can continue.
- ❌ High overhead — creating a user thread requires creating a corresponding kernel thread (expensive).
- Example: Windows, Linux.

### (c) Many-to-Many
- Multiplexes many user threads to an **equal or smaller** number of kernel threads.
- Best of both worlds: OS can create a sufficient number of kernel threads, and app developers can create as many user threads as needed without kernel overhead for each one.

| Model | Blocking Behavior | Multiprocessing | Overhead |
|---|---|---|---|
| Many-to-One | Whole system blocks | ❌ No | Low |
| One-to-One | Only that thread blocks | ✅ Yes | High |
| Many-to-Many | Only that thread blocks (usually) | ✅ Yes | Balanced |

---

## 3. Threading Issues

### (a) Semantics of `fork()` and `exec()`
- Key question: **Does `fork()` duplicate only the calling thread, or all threads of the process?**
- Some UNIX systems provide two versions of fork to handle this.

### (b) Thread Cancellation
Terminating a thread before it has finished its work. Two approaches:

| Type | Behavior |
|---|---|
| **Asynchronous cancellation** | Terminates the target thread immediately (risk: may corrupt shared data mid-update) |
| **Deferred cancellation** | Target thread periodically checks a flag to see if it should terminate gracefully (safer) |

### (c) Signal Handling
Signals notify a process that a particular event has occurred (UNIX systems). Flow:
1. Signal is **generated** by a particular event
2. Signal is **delivered** to a process
3. Signal is **handled**

**Options for delivering signals in a multithreaded process:**
- Deliver to the thread to which the signal applies
- Deliver to *every* thread in the process
- Deliver to certain specific threads
- Assign one dedicated thread to receive all signals for the process

### (d) Thread Pools
**Problem solved**: creating a new thread for every task has overhead.
**Solution**: Create a number of threads at process startup and place them in a pool where they wait for work.

**Advantages:**
- **Speed**: Servicing a request with an existing idle thread is faster than creating a new one.
- **Resource bounding**: Limits the total number of threads in the app to the pool size (prevents memory exhaustion).

### (e) Thread-Specific Data
Allows each thread to have its own copy of data (not covered in depth in this lecture but listed as a topic).

---

## 4. Case Study: Multi-Process vs Multi-Threaded Execution

**Problem**: Sum all numbers from 0 to 10,000,000 using 4 processors.

### Approach A: Multi-Process Execution
```c
unsigned long addall() {
    int i = 0;
    unsigned long sum = 0;
    while (i < 10000000) {
        sum += i; i++;
    }
    return sum;
}
```
- Requires **4 separate `fork()`** system calls
- Each process is **completely isolated** — own instructions, data, stack, heap
- Needs **IPC (Inter-Process Communication)** to combine results → more system calls
- **Significant overhead**

### Approach B: Multi-Threaded Execution (using Pthreads)
```c
#include <pthread.h>
unsigned long sum[4];

void *thread_fn(void *arg) {
    long id = (long) arg;
    int start = id * 2500000;
    int i = 0;
    while (i < 2500000) {
        sum[id] += (i + start);
        i++;
    }
    return NULL;
}

int main() {
    pthread_t t1, t2, t3, t4;
    pthread_create(&t1, NULL, thread_fn, (void *)0);
    pthread_create(&t2, NULL, thread_fn, (void *)1);
    pthread_create(&t3, NULL, thread_fn, (void *)2);
    pthread_create(&t4, NULL, thread_fn, (void *)3);
    pthread_join(t1, NULL);
    pthread_join(t2, NULL);
    pthread_join(t3, NULL);
    pthread_join(t4, NULL);
    printf("%lu\n", sum[0]+sum[1]+sum[2]+sum[3]);
    return 0;
}
```
> Compile with: `gcc threads.c -lpthread` then run `./a.out`

**Properties:**
- Only **1 `fork()`** needed; 4 lightweight threads created — much cheaper
- Threads are **not isolated** from each other — share instructions, global variables, heap
- Each thread has its **own stack** only
- Management requires fewer/no expensive system calls

| | Multi-Process | Multi-Threaded |
|---|---|---|
| fork() calls | 4 | 1 |
| Isolation | Full (own memory map) | None (shared address space) |
| Communication | IPC needed | Shared memory (direct) |
| Overhead | High | Low |

---

## 5. Knowledge Checks / GATE-Style Recap

**Q: What is a Thread?**
✅ A lightweight process within a process.

**Q: True about Multithreading?**
✅ Threads within the same process share the same memory space.

**Q: Which of the following is/are shared by all threads in a process?**
- I. Program counter — ❌ private
- II. Stack — ❌ private
- III. Address space — ✅ shared
- IV. Registers — ❌ private

**Answer: (B) III only.**
> 🏠 **Memory trick**: *"The process owns the house (heap/globals); the threads just live in it."* → Threads share the **heap and global variables**, but each has its **own stack, PC, and registers**.

---

## 6. OS-Specific Thread Implementations

### Linux Threads
- Linux calls them **"tasks"**, not "threads"
- Thread creation via **`clone()`** system call
- `clone()` allows a child task to **share the address space** of the parent task (process)

### Java Threads
- Managed by the **JVM**
- Created by:
  - Extending the `Thread` class
  - Implementing the `Runnable` interface

**Java Thread States:** `new → runnable → blocked → runnable → dead`
- `start()`: new → runnable
- `sleep()`/I/O: runnable → blocked
- I/O available: blocked → runnable
- `run()` exits: runnable → dead

### Windows Threads
- Implements **one-to-one** mapping
- Each thread contains: a thread ID, register set, separate user/kernel stacks, private data storage area
- Register set + stacks + private storage = **"context"** of the thread
- Primary data structures:
  - **ETHREAD** (executive thread block)
  - **KTHREAD** (kernel thread block)
  - **TEB** (thread environment block)

---

## 7. Introduction to CPU Scheduling

### Why CPU Scheduling?
The CPU is extremely fast but can only execute **one process at a time**. The Ready Queue is where the OS makes its most critical decisions — like an infinite number of lanes converging into a single toll booth.

### The Process Lifecycle
```
New → Ready → Running → Terminated
         ↑         ↓
         └── Waiting (I/O)
```

**CPU Scheduling occurs on 4 transitions:**
1. Running → Terminated (**Non-preemptive**)
2. Running → Waiting/I/O event wait (**Non-preemptive**)
3. Running → Ready (interrupt) (**Preemptive**)
4. Waiting → Ready (**Preemptive**)

> Transitions 1 & 2 = **Non-Preemptive** (process voluntarily leaves CPU)
> Transitions 3 & 4 = **Preemptive** (OS forcibly takes CPU away)

### Preemptive vs Non-Preemptive
| | Non-Preemptive (Freight Train) | Preemptive (Redirectable Taxi) |
|---|---|---|
| Behavior | Once running, cannot be stopped until it terminates or voluntarily waits | OS can forcefully evict a process via timer interrupt or higher-priority arrival |
| Overhead | Lower | Higher (context-switching cost) |
| Responsiveness | Lower | Highly responsive |

### CPU vs I/O Bursts
Process execution = a **cycle of CPU execution and I/O wait**.
- **CPU-bound process**: long CPU bursts, little I/O (shallow waiting valleys)
- **I/O-bound process**: short CPU bursts, lots of I/O (wide waiting valleys)
- Ideally, the system admits a **mix** of CPU-bound and I/O-bound processes to maximize both CPU and I/O utilization.

**Burst Time** = duration of a burst → splits into **CPU Burst** and **I/O Burst**.

### The Dispatcher & Context Switch Cost
Every context switch = **pure overhead** (zero useful work done):
1. **Save state** — OS pauses the process, saves CPU registers to memory (PCB)
2. **The idle swap** — CPU chamber is briefly empty
3. **Load state** — OS loads the next process's registers, resumes execution

---

## 8. Scheduling Criteria (Mission Control Dashboard)

| Maximize ↗ | Minimize ↓ |
|---|---|
| **CPU Utilization** — keep CPU as busy as possible | **Turnaround Time (TAT)** — submission to completion |
| **Throughput** — # of processes completed per time unit | **Waiting Time (WT)** — total time spent in ready queue |
| | **Response Time (RT)** — submission to *first* response (not full output) |

---

## 9. Key Process Time Metrics — Formulas

For process $P_i$:

| Symbol | Meaning |
|---|---|
| AT / $A_i$ | Arrival Time (a.k.a. Submission Time) |
| BT / $X_i$ | Burst Time (CPU time needed) |
| IOBT / $Y_i$ | I/O Burst Time |
| CT / $C_i$ | Completion Time |
| WT | Waiting Time |
| TAT | Turnaround Time |

### Core Formulas:

$$\text{Turnaround Time (TAT)} = CT - AT$$

$$\text{Waiting Time (WT)} = TAT - (BT + IOBT)$$

> If $IOBT = 0$: $\quad WT = TAT - BT$

$$\text{Response Time} = \text{First Waiting Time } (WT_1)$$

### For n Processes:

$$\text{Average TAT} = \frac{1}{n}\sum (C_i - A_i)$$

$$\text{Average WT} = \frac{1}{n}\sum \left[(C_i - A_i) - (X_i + Y_i)\right]$$

$$\text{Weighted TAT} = \frac{C_i - A_i}{X_i}$$

### Schedule Length (L) for n processes:

$$L = \text{Max}(C_i) - \text{Min}(A_i)$$

$$\text{Throughput} (\eta) = \frac{n}{L}$$

### With Scheduling Overhead ($\delta$)
If each context switch costs $\delta$ (dispatch overhead) time units:

$$WT = TAT - (BT + IOBT + \delta)$$

---

## 10. FCFS (First Come First Serve) Scheduling

### Key Properties
- Simplest CPU scheduling algorithm
- The process that requests the CPU **first** gets it **first**
- Implemented via a **FIFO queue**

**When analyzing any scheduling algorithm, always define:**
1. **Selection criteria** → for FCFS: Arrival Time (AT)
2. **Mode of operation** → Non-Preemptive
3. **Tie breaker** → Lower Process ID (PID)

### ⚠️ The Fatal Flaw: Convoy Effect
A massive, slow-moving (e.g., I/O-bound) process traps fast, short processes behind it, drastically increasing the **average waiting time** — like a shopping queue where one customer with a full cart holds up everyone with just 1-2 items.

### Worked Example 1 (δ = 0, all arrive at t=0)

| P.No | AT | BT | CT | TAT | WT |
|---|---|---|---|---|---|
| 1 | 0 | 2 | 2 | 2 | 0 |
| 2 | 0 | 3 | 5 | 5 | 2 |
| 3 | 0 | 5 | 10 | 10 | 5 |
| 4 | 0 | 1 | 11 | 11 | 10 |
| 5 | 0 | 4 | 15 | 15 | 11 |

**Gantt Chart:**
```
| P1 | P2 | P3 | P4 | P5 |
0    2    5    10   11   15
```
Schedule Length **L = 15**

### Worked Example 2 (δ = 2 — scheduling overhead included)

| P.No | AT | BT |
|---|---|---|
| 1 | 5 | 6 |
| 2 | 12 | 3 |
| 3 | 14 | 1 |
| 4 | 15 | 3 |
| 5 | 20 | 4 |

Using $WT = TAT - (BT + \delta)$:

| Process | TAT | WT |
|---|---|---|
| P1 | 8 | 0 |
| P2 | 6 | 1 |
| P3 | 7 | 4 |
| P4 | 11 | 6 |
| P5 | 12 | 6 |

### Worked Example 3 — Average Response Time (δ = 0)

| P.No | AT | BT |
|---|---|---|
| 1 | 22 | 5 |
| 2 | 8 | 3 |
| 3 | 18 | 2 |
| 4 | 2 | 5 |
| 5 | 15 | 5 |

Order of execution by arrival: P4, P2, P5, P3, P1

$$\text{Avg RT} = \frac{0+0+2+0+0}{5} = \frac{2}{5} = 0.4$$

### ⭐ Worked Example 4 — FCFS with Non-Zero IOBT & Scheduling Overhead

Note: $BT + IOBT = \text{Service Time}$

| P.No | AT | BT | IOBT | BT (2nd) |
|---|---|---|---|---|
| 1 | 0 | 3 | 5 | 3 |
| 2 | 1 | 1 | 10 | 4 |

With $\delta = 1$, Schedule Length **L = 21**

$$\%\ \text{CPU Idleness} = \frac{6}{21}$$

$$\text{Avg WT} = \frac{3 + \ldots}{2} = 1.5$$

$$\text{Avg RT} = \frac{6+3}{2} = 1.5$$

---

## 11. GATE PYQ Practice

**Q1. Which one of the following statements about ULT & KLT is FALSE?**
- (A) Context switch time is longer for kernel level threads than user level threads — **True**
- (B) User level threads do not need any hardware support — **True**
- (C) Related kernel level threads can be scheduled on different processors in a multiprocessor system — **True**
- (D) Blocking one kernel level thread blocks all related threads — **FALSE ✅** (this is the answer — this is actually a property of ULTs, not KLTs)

**Q2. Which of the following are true about user-level vs kernel-supported threads?**
- I. Context switch is faster with kernel-supported threads — ❌ False (ULT switching is faster)
- II. For user-level threads, a system call can block the entire process — ✅ True
- III. Kernel supported threads can be scheduled independently — ✅ True
- IV. User level threads are transparent to the kernel — ✅ True

**Answer: (A) II, III and IV only**

**Q3. Which one of the following is FALSE?**
- (A) User level threads are not scheduled by the kernel — True
- (B) When a user level thread is blocked, all other threads of its process are blocked — True
- (C) Context switching between user level threads is faster than between kernel level threads — True
- (D) Kernel level threads cannot share the code segment — **FALSE ✅** (they *can* share code segment)

**Q4. Threads of a process share...**
✅ **(D) Both heap and global variables** *(not the stack, registers, or PC)*

---

## 📝 Quick Revision Checklist
- [ ] ULT vs KLT — differences table
- [ ] Multithreading models (Many-to-One, One-to-One, Many-to-Many) + trade-offs
- [ ] Threading issues: fork/exec semantics, cancellation types, signal handling, thread pools
- [ ] Process vs Thread model — overhead comparison
- [ ] Formulas: TAT, WT, RT, Schedule Length, Throughput
- [ ] FCFS: selection criteria, mode, tie-breaker, Convoy Effect
- [ ] Practice all 4 GATE PYQs until you can answer without looking

---
*Next lecture will likely cover: SJF, Round Robin, Priority Scheduling, Multilevel Queue/Feedback Queue.*
