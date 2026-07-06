# Threads & Multi-Threading — GATE CS/IT OS Notes (Lecture 07)

> Source: Principles of Operating Systems — Lecture 07 (GeeksforGeeks GATE Series)

## 📑 Table of Contents
1. [Recap: Queues & Scheduling (Background)](#recap-queues--scheduling-background)
2. [Schedulers & Dispatcher](#schedulers--dispatcher)
3. [Context Switching](#context-switching)
4. [What is a Thread?](#what-is-a-thread)
5. [Single-Threaded vs Multi-Threaded Process](#single-threaded-vs-multi-threaded-process)
6. [Client–Server Architecture & Threads](#clientserver-architecture--threads)
7. [Process vs Threads](#process-vs-threads)
8. [Benefits of Multithreading](#benefits-of-multithreading)
9. [Thread Life Cycle (Java Model)](#thread-life-cycle-java-model)
10. [Formulas Summary](#formulas-summary)
11. [Memory Tricks & Mnemonics](#memory-tricks--mnemonics)
12. [GATE PYQs & Practice MCQs](#gate-pyqs--practice-mcqs)
13. [Quick Revision Checklist](#quick-revision-checklist)

---

## Recap: Queues & Scheduling (Background)

This lecture opens with a quick recap of process queues before diving into threads.

- **Ready Queue**: linked list of PCBs of processes ready to execute. Implemented as `head`/`tail` pointers into a chain of PCBs.
- **Device Queue** (e.g., disk unit 0, mag tape unit 0/1, terminal unit 0): each I/O device has its own queue of waiting PCBs.
- **In-Memory vs On-Disk queues**:
  - In-Memory → **Ready Queue** + **Block/Wait Queue**
  - On-Disk → **Job Queue** (programs ready to be loaded into memory) + **Suspend Queue** (partially executed, swapped-out processes)

**State Queuing Diagram** — a process leaves the **Running** state due to:
- I/O request → I/O queue → I/O device → back to Ready Queue
- Time slice expired → back to Ready Queue
- Fork a child → child executes → parent back to Ready Queue
- Wait for interrupt → interrupt occurs → back to Ready Queue

**Suspend Queue**: holds partially executed, swapped-out processes (swapped out from ready queue / I/O waiting queue, swapped back in later). Managed by the **Medium Term Scheduler (MTS)**.

---

## Schedulers & Dispatcher

| Scheduler | Transition | Role |
|---|---|---|
| **Long Term Scheduler (LTS)** | New → Ready | Decides which jobs are admitted into memory (controls degree of multiprogramming) |
| **Short Term Scheduler (STS)** / CPU Scheduler | Ready → Running | Picks which ready process runs on CPU next (runs frequently) |
| **Medium Term Scheduler (MTS)** | Ready/Running ⇄ Suspend Queue | Swaps processes out/in to control multiprogramming level (suspend/resume) |

**Dispatcher**: the module that actually gives control of the CPU to the process selected by the STS. Involves:
- **Save** the current process's state (registers, PC) → PCB (takes time $t_1$)
- **Load** the next process's state from its PCB → CPU (takes time $t_2$)

This save+load operation is called a **Context Switch**.

---

## Context Switching

**Definition**: The process of saving the current state (registers, Program Counter) of one process and loading the saved state of another so the CPU can resume it later.

> ⚠️ **Key Insight**: Context switching is **pure overhead** — the CPU does *no useful computational work* while switching.

### Timeline
```
Process A running → [Save State to PCB A] → Overhead/Idle Time → [Load State from PCB B] → Process B running
```

### Two Important Time Components

- $t_1$ = **Mode-switching time** (User mode ⇄ Kernel mode)
- $t_2$ = **Context-switch time between two processes** (includes full save+load of PCBs)

**Relation:**
$$t_1 < t_2$$

*Why?* Mode switching (user↔kernel) is a subset of work done during a full context switch — a context switch between two processes necessarily involves at least one mode switch plus the additional overhead of saving/loading two different process states. So switching between processes always costs more time than a single user↔kernel mode transition.

### Context Switch / Dispatch Latency Formula
$$\text{Dispatch Latency} = t_1 + t_2 + \text{STS(time)}$$

Where:
- $t_1$ = time to save the state of outgoing process
- $t_2$ = time to load the state of incoming process
- **STS time** = time taken by the Short Term (CPU) Scheduler to pick the next process

This total is also referred to as **CPU Scheduling Overhead**.

---

## What is a Thread?

A **Thread** is a **Light Weight Process (LWP)**.

Key characteristics (from the whiteboard):
- **Active Entity**
- **Schedulable / Dispatchable unit**
- **Unit of CPU utilization**
- Described poetically as an **"Animated Spirit"** of a process — it's what actually *executes* code.

**Traditional / Heavy-weight Process view**: A single-threaded process has ONE thread — representing the **order of execution of instructions** in the code/text section (a single sequential path, jumping through instructions).

---

## Single-Threaded vs Multi-Threaded Process

### Single-Threaded Process
- Has one thread of execution running through: `code`, `data`, `files`, `registers`, `stack`.
- Represented as: one squiggly arrow moving through the code segment.

### Multi-Threaded Process
- **Single Address Space** shared by all threads (code, data, files are common).
- **Each thread** has its own: **registers**, **stack**, and **Program Counter (PC)**.
- Each thread has a **Thread Control Block (TCB)** — analogous to (but smaller than) a Process's PCB.

**Key Inequality:**
$$|TCB| < |PCB|$$

*(A TCB stores less info than a PCB — since much of the process state like code/data/files is shared and doesn't need duplicating per-thread.)*

### Comparison Table — Single-Threaded vs Multi-Threaded Process

| Aspect | Single-Threaded Process | Multi-Threaded Process |
|---|---|---|
| Code, Data, Files | Owned solely by the one thread | Shared by all threads |
| Registers | One set | One set **per thread** |
| Stack | One stack | One stack **per thread** |
| Program Counter | One PC | One PC **per thread** |
| Execution | Single sequential path | Multiple independent paths (may run different parts of program) |

### Multi-Threading Ideology (Key Points)
- A Thread is like **another copy of a process** that executes **independently**.
- Threads **share the same address space** (code, heap).
- Each thread has a **separate PC** — each thread may run over a *different part* of the program.
- Each thread has a **separate stack** — for independent function calls.

---

## Client–Server Architecture & Threads

**Background**: In a Client-Server environment, multiple clients ($C_1, C_2, ... C_n$) send requests ($r_1, r_2, ... r_k$) to a single Server Process over the network.

### Server Process Types

| Type | Description |
|---|---|
| **Iterative Server** | Handles one client request at a time, sequentially |
| **Concurrent Server** | Handles multiple requests simultaneously — implemented via **Multi-Process** or **Multi-Threaded** design |

### Client-Server Flow with Threads
```
(1) client → sends request → server
(2) server → creates new thread → to service the request
(3) server → resumes listening → for additional client requests
```

**Why threads here?** Creating a new thread per request lets the server handle many clients concurrently while resuming listening immediately — much cheaper than forking a whole new process per client.

---

## Process vs Threads

### Scenario Comparison

| Scenario | Parent P forks child C (Process) | Parent P creates Threads T1, T2 |
|---|---|---|
| Memory sharing | P and C **do not share** any memory | T1 and T2 **share parts of the address space** |
| Communication | Needs a **complicated IPC mechanism** | **Global variables** can be used directly |
| Memory footprint | **Extra copies** of code, data in memory | **Smaller memory footprint** |

> **Threads are like separate processes, except they share the same address space.**

### Detailed Process vs Thread Rules

- A Thread has **no data segment or heap** of its own.
- **A Thread cannot live on its own** — it must be attached to a process. *(High-yield GATE fact!)*
- There can be more than one thread in a process; **each thread has its own stack**.
- If a thread dies, **its stack is reclaimed**.
- A process has code, heap, stack, and other segments.
- **A process has at least one thread.**
- Threads within a process **share** the same code, files, and other resources.
- **If a process dies, all its threads die.**

### What Threads Share vs What Threads Own

| **Threads SHARE (process-level)** | **Threads have their OWN (thread-level)** |
|---|---|
| Address space | Program Counter (PC) |
| Heap | Registers |
| Static/Global data | Stack |
| Code segments | — |
| File descriptors | — |
| Child processes | — |
| Pending alarms | — |
| Signals & signal handlers | — |
| Accounting information | — |

---

## Benefits of Multithreading

| Benefit | Explanation |
|---|---|
| **Responsiveness** | An interactive app keeps running even if part of it is blocked/doing a lengthy operation — critical for UI design |
| **Resource Sharing** | Processes need explicit IPC (shared memory, message passing) to share resources; threads share memory/resources of their parent process **by default** |
| **Economy** | Creating/context-switching a process is expensive; threads are cheaper. **Data point: In Solaris, creating a process is ~30× more time-consuming than creating a thread** |
| **Scalability** | On multiprocessor architectures, threads can run **in parallel** on different cores; a single-threaded process can use only **one processor**, no matter how many are available |
| **Utilization of MP Architectures** | (Extension of Scalability) — multiple threads → multiple cores used simultaneously |
| **Lightweight** | Smaller data structure (TCB) than a full process (PCB) |
| **Efficient Communication** | Via shared global variables — no IPC overhead |
| **Efficient Context Switching** | Cheaper than switching between full processes |

> **Mnemonic to remember benefits:** **R.R.E.S.U.L.E.** → Responsiveness, Resource sharing, Economy, Scalability, Utilization (MP), Lightweight, Efficient comm./switching

### The Synchronization Problem (Preview)
> "If all threads in a process share the exact same Data and Heap space, what happens when two threads try to overwrite the exact same variable at the exact same millisecond?"

This is the fundamental challenge of multithreading → **Synchronization** (covered in later lectures — critical sections, locks, semaphores, mutexes).

---

## Thread Life Cycle (Java Model)

```
New Thread → start() → Runnable ⇄ Running → (end of run() method) → Dead
                              ↑↓
                    Running → sleep()/wait() → Waiting → notify()/notifyAll() → Runnable
                              ↓
                    Running → yield() → Runnable
```

### States
| State | Meaning |
|---|---|
| **New** | Thread object created, not yet started |
| **Runnable** | Eligible to run, waiting for CPU (via `start()`, `yield()`, or `notify()`) |
| **Running** | Currently executing on CPU |
| **Blocked/Waiting** | Waiting due to `sleep()`, `wait()`, or I/O (idle, not runnable) |
| **Dead** | Thread has finished (`run()` completed) or was `stop()`ped/killed |

### Full State Transition Diagram (with methods)
```
New --start()--> Runnable --run()--> Running --yield()--> Runnable
Running --sleep()/wait()--> Blocked --resume()/notify()--> Runnable
Running --(end of execution)--> Dead
Any state --stop()--> Dead (Killed Thread)
```

---

## Formulas Summary

| Formula | Meaning |
|---|---|
| $t_1 < t_2$ | Mode-switch time (user↔kernel) < Context-switch time (between 2 processes) |
| $\text{Dispatch Latency} = t_1 + t_2 + \text{STS(time)}$ | Total CPU scheduling overhead = save time + load time + scheduler decision time |
| $\lvert TCB \rvert < \lvert PCB \rvert$ | Thread Control Block is smaller/lighter than Process Control Block |

---

## Memory Tricks & Mnemonics

- **Thread = "Animated Spirit"** of a process — gives it life/motion (execution), while the process is just the static container (code+data+files).
- **Process = The Kitchen** 🏠 (owns the building, raw ingredients = Memory, master recipes = Code).
- **Threads = The Chefs** 👨‍🍳 (multiple chefs work in the same kitchen simultaneously on different tasks).
- **Workstation = Private Thread State** (each chef needs their own cutting board = Stack, and step-by-step checklist = Program Counter).
- **Threads are like separate processes, except they share the same address space.**
- **A Thread cannot live on its own — it needs a process (a house) to live in.**
- **TCB < PCB** — think "Thread carries less baggage than a Process."

---

## GATE PYQs & Practice MCQs

### Q1. What is a Thread?
- A. **A lightweight process within a process** ✅
- B. A type of hardware interrupt
- C. A separate program in memory
- D. A kernel module

**Correct: A** — Threads share the code/data/address space of a process, distinguishing them from separate programs (C) or kernel modules (D); a thread is not a hardware interrupt (B).

---

### Q2. Which of the following is true about Multithreading?
- A. **Threads within the same process share the same memory space** ✅
- B. Each thread has its own code segment
- C. Threads run in different address spaces
- D. Threads do not share open files

**Correct: A** — Code segment (B), address space (C), and open files (D) are all **shared** by threads of the same process, not separate/private.

---

### Q3. Which of the following is NOT shared among threads in the same process?
- A. **Stack** ✅
- B. Code segment
- C. Data segment
- D. Open files

**Correct: A** — Each thread has its own private stack; code segment, data segment, and open files are shared resources.

---

### Q4. What is the main advantage of using Threads over Processes?
- A. Threads are more secure
- B. **Threads have less context-switching overhead** ✅
- C. Threads use more memory
- D. Threads are slower to create

**Correct: B** — Threads share resources, avoiding costly memory allocation/duplication; this makes them cheaper to create/switch than processes (opposite of C, D). Security (A) is not an inherent advantage — in fact threads are less isolated.

---

### Q5. Which thread library is most commonly used in UNIX/Linux systems?
- A. Windows Threads
- B. **POSIX Threads (Pthreads)** ✅
- C. Java Threads
- D. Solaris Threads

**Correct: B** — Pthreads is the standard threading API on UNIX/Linux; the others are platform/language-specific (Windows, Java, Solaris-specific implementations).

---

### Q6. In a Multi-Threaded Process, how many Program Counters exist?
- A. One for the entire process
- B. **One for each thread** ✅
- C. One for each core
- D. One for each function

**Correct: B** — Each thread executes independently and needs to track its own current instruction, hence its own PC.

---

### Q7. Which of the following models maps many User-level Threads to one Kernel Thread?
- A. One-to-One
- B. **Many-to-One** ✅
- C. Many-to-Many
- D. Two-Level

**Correct: B** — By definition, Many-to-One maps multiple ULTs to a single KLT; One-to-One maps 1:1, Many-to-Many and Two-Level use multiple KLTs.

---

### Q8. Which component of Thread has its own independent copy?
- A. Code segment
- B. Heap
- C. **Stack** ✅
- D. Global variables

**Correct: C** — Stack is thread-private; code segment, heap, and global variables are all shared across threads of a process.

---

### Q9. What is Thread Synchronization used for?
- A. To speed up thread execution
- B. To avoid deadlock
- C. **To ensure correct execution order and avoid race conditions** ✅
- D. To terminate threads quickly

**Correct: C** — Synchronization exists to coordinate access to shared data and prevent race conditions; it isn't primarily about speed (A), doesn't inherently avoid deadlock (B, in fact can cause it if misused), and isn't about termination (D).

---

### Q10. Which is not a valid benefit of Multithreading?
- A. Better resource utilization
- B. Improved responsiveness
- C. **Simplified programming** ✅
- D. Parallelism on multiprocessor systems

**Correct: C** — Multithreaded programming is actually *harder* (due to synchronization issues), not simpler; A, B, D are all genuine documented benefits.

---

### GATE PYQ 1. Let the time taken to switch between user and kernel modes of execution be $t_1$, while the time taken to switch between two processes be $t_2$. Which of the following is TRUE?
- A. $t_1 > t_2$
- B. $t_1 = t_2$
- C. **$t_1 < t_2$** ✅
- D. Nothing can be said about the relation between $t_1$ and $t_2$

**Correct: C** — A process context switch necessarily includes at least one mode switch plus additional overhead of saving/restoring full process state, so it always takes longer: $t_1 < t_2$.

---

### GATE PYQ 2. In Multi-Programmed systems, it is advantageous if some programs (e.g., editors, compilers) can be shared by several users. Which of the following must be true for a single copy of a program to be shared by several users?
I. The program is macro. II. The program is recursive. III. The program is reentrant.
- A. I only
- B. II only
- C. **III only** ✅
- D. I & II only
- E. I, II, and III

**Correct: C** — **Reentrancy** means the code does not modify itself and keeps all per-user state (data) separate, allowing safe concurrent sharing by multiple users; being macro (A) or recursive (B) is unrelated to safe sharing.

---

### GATE PYQ 3. Which of the following is/are shared by all the threads in a process?
I. Program counter II. Stack III. Address space IV. Registers
- A. I and II only
- B. **III only** ✅
- C. IV only
- D. III and IV only

**Correct: B** — Only the **address space** is shared; PC (I), Stack (II), and Registers (IV) are all private to each individual thread.

---

### GATE PYQ 4. Threads of a process share:
- A. Global variables but not heap
- B. Heap but not global variables
- C. Neither global variables nor heap
- D. **Both heap and global variables** ✅

**Correct: D** — Both heap (dynamic memory) and global/static data are part of the shared address space accessible to all threads of a process.

---

### GATE PYQ 5 (repeated twice in lecture — merged). Which one of the following is FALSE?
- A. User level threads are not scheduled by the kernel.
- B. **When a user level thread is blocked, all other threads of its process are blocked.** ✅ (this is the FALSE statement — it's only true for ULT under Many-to-One model, and the "FALSE" flagged option here is B based on lecture's answer)
- C. Context switching between user level threads is faster than context switching between kernel level threads.
- D. Kernel level threads cannot share the code segment.

**Correct (FALSE statement): D** — Kernel-level threads absolutely **can share** the code segment (it's shared by all threads of a process regardless of ULT/KLT); A and C are true statements about ULT vs KLT behavior. *(Note: B is true only in the pure Many-to-One ULT model — with proper multithreading models, B need not always hold, but the classically flagged FALSE answer for this GATE question is D.)*

---

### GATE PYQ 6. A thread is usually defined as a lightweight process because an OS maintains smaller data structures for a thread than for a process. In relation to this, which statement is correct?
- A. OS maintains only scheduling and accounting information for each thread.
- B. OS maintains only CPU registers for each thread.
- C. **OS does not maintain virtual memory state for each thread.** ✅
- D. OS does not maintain a separate stack for each thread.

**Correct: C** — Virtual memory / address space is a **process-level** property shared by all its threads, so the OS does NOT need to maintain separate VM state per thread; but OS DOES maintain per-thread registers, stack, and scheduling info (contradicting A, B, D as "only"/exclusive claims).

---

### GATE PYQ 7. Consider the following statements about User level threads and Kernel level threads. Which one of the following is FALSE?
- A. Context switch time is longer for kernel level threads than for user level threads.
- B. User level threads do not need any hardware support.
- C. Related kernel level threads can be scheduled on different processors in a multiprocessor system.
- D. Blocking one kernel level thread blocks all related threads. ← **FALSE** ✅

**Correct (FALSE statement): D** — KLTs are managed/scheduled independently by the kernel, so blocking one KLT does **not** block other related threads (this is actually a key *advantage* of KLT over ULT); A, B, C are all true.

---

### GATE PYQ 8. Consider the following statements with respect to user-level threads and kernel-supported threads. Which of the above statements are true?
I. Context switch is faster with Kernel-supported Threads.
II. For User-level Threads, a system call can block the entire process.
III. Kernel supported Threads can be scheduled independently.
IV. User level Threads are transparent to the Kernel.
- A. II, III and IV only ✅
- B. II and III only
- C. I and III only
- D. I and II only

**Correct: A** — Statement I is **false** (ULT context switches are actually *faster*, not KLT); II, III, IV are all true — a blocking syscall in ULT blocks the whole process, KLTs are scheduled independently by the kernel, and the kernel doesn't know about ULTs (transparent to it).

---

### GATE PYQ 9. Which of the following statements about Threads is/are TRUE?
- A. Threads can only be implemented in kernel space ❌
- B. Each thread has its own file descriptor table for open files ❌
- C. All the threads belonging to a process share a common stack ❌
- D. **Threads belonging to a process are by default not protected from each other** ✅

**Correct: D** — Since threads share the same address space by default, there's no memory protection between them (unlike separate processes). A is false (ULTs exist in user space too); B is false (file descriptors are shared, not per-thread); C is false (each thread has its OWN stack, not a shared one).

---

## Quick Revision Checklist

Use this to self-test before re-reading the full notes:

- [ ] Can I explain the difference between **Ready Queue** and **Device Queue**?
- [ ] Can I name and describe the roles of **LTS, STS, MTS**?
- [ ] What does the **Dispatcher** do, and what is **Context Switching**?
- [ ] Why is context switching called **"pure overhead"**?
- [ ] Can I state the relation $t_1 < t_2$ and explain why?
- [ ] Can I write the **Dispatch Latency formula**: $t_1 + t_2 + \text{STS(time)}$?
- [ ] Can I define a **Thread** in my own words (LWP, active entity, unit of CPU utilization)?
- [ ] What is the difference between a **Single-threaded** and **Multi-threaded** process?
- [ ] What does each thread have **privately** (PC, registers, stack) vs what's **shared** (code, data, heap, files)?
- [ ] Can I state $|TCB| < |PCB|$ and explain why?
- [ ] Can I explain **Iterative vs Concurrent server**?
- [ ] Can I walk through the **3-step client-server threading flow**?
- [ ] Process vs Thread: IPC complexity, memory sharing, memory footprint — can I compare?
- [ ] Can I list **all 4 things a thread CANNOT live without** (i.e., must have a parent process)?
- [ ] Can I list the **4 main benefits of multithreading** (Responsiveness, Resource Sharing, Economy, Scalability)?
- [ ] Do I remember the **Solaris data point** (process creation ~30× costlier than thread creation)?
- [ ] Can I draw the **Thread Life Cycle** diagram (New → Runnable → Running → Waiting/Blocked → Dead)?
- [ ] Do I understand why **synchronization** becomes necessary once threads share heap/data?
- [ ] Have I reviewed all **GATE PYQs** above and understood why each wrong option is wrong?

---

*End of notes — Lecture 07: Threads & Multi-Threading*
