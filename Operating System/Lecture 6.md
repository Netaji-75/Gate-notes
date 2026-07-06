# Process Concepts — Lecture 06 (Principles of Operating Systems)

> Source: GeeksforGeeks GATE CS&IT — Principles of OS, Lecture 06

## 📑 Topics Covered
| # | Topic |
|---|-------|
| 1 | Program vs Process |
| 2 | Static vs Dynamic Data, Memory Allocation |
| 3 | Process as an Abstract Data Type (ADT) |
| 4 | Process Structure in Memory (Text, Data, Heap, Stack) |
| 5 | Memory Layout of a Typical C Program |
| 6 | Process Attributes |
| 7 | Process Control Block (PCB) / Process Descriptor |
| 8 | Process States & State Transition Diagram |
| 9 | GATE PYQs & Review Questions |

---

## 1. Program vs Process

**Program** = a `.exe` file — a **passive entity** sitting on disk. It contains:
- **Instructions** (code)
- **Data**

A program can exist as:
| Type | Description |
|------|-------------|
| **High-Level (H.L) Source Program** | Human-readable code (before compilation) |
| **Low-Level (L.L) Target Machine Language Program** | Compiled machine code (`.exe`) — one program, can be run many times |

**Process** = a program **in execution** — an **active entity** loaded into RAM (main memory).

> 💡 **Mnemonic**: Program is the *recipe* (static, on disk). Process is the *meal being cooked* (active, in memory, consuming resources).

### Comparison Table: Program vs Process

| Aspect | Program | Process |
|--------|---------|---------|
| State | Passive entity | Active entity |
| Location | Stored on disk (`.exe`) | Loaded into RAM |
| Nature | Executable file | Program in execution |
| Relationship | One program → many processes | Instance of a program |
| Example | A single `.exe` of a text editor | Multiple users running the same editor = multiple processes |

**Other synonymous descriptions of a Process** (as given in lecture):
- Instance of a Program
- Active Entity
- Unit of CPU utilization (**schedulable unit**)
- Locus of control of the OS
- "Animated Spirit" (informal/poetic description used by some textbooks)

---

## 2. Static vs Dynamic Data & Memory Allocation

A program's **Data** splits into two categories:

| Type | Fixed Size? | When Allocated | Example |
|------|-------------|-----------------|---------|
| **Static** | Fixed size | **Before** run-time (compile time) | `int a, b, c;` |
| **Dynamic** | May be known/unknown, variable size | **At run-time (R.T)** | `int *ptr = malloc(sizeof(int)*n);` |

### Example: Static vs Dynamic in C
```c
int n, *ptr;
scanf("%d", &n);
ptr = (int *) malloc(sizeof(int) * n);
```
- `n` and `ptr` are **static** — their own storage (the variable itself, e.g., the pointer `ptr`) is fixed-size and known at compile time.
- The **array pointed to by `ptr`** is **dynamic** — its size (`n` elements) is determined only at run-time, and it's allocated from the **heap**.

**Diagram concept**: `ptr` (a static 2-byte/4-byte slot at some address, e.g., 2000) *holds the address* (e.g., 4010) of the dynamically allocated block on the heap.

---

## 3. Process as an Abstract Data Type (ADT) — Developer's View

From a **developer's perspective**, a Process can be modeled as an ADT with 4 components:

$$
\text{Process ADT} = \langle \text{Definition, Representation, Operations, Implementation} \rangle
$$

Plus its **Attributes** (see Section 5).

| Component | Meaning |
|-----------|---------|
| **Definition (Defn)** | What a process conceptually *is* |
| **Representation (Repr)** | How it is represented internally (data structure — the PCB) |
| **Operations** | create, terminate, suspend, resume, schedule, etc. |
| **Implementation (Impl)** | How OS actually implements those operations |

---

## 4. Process Structure in Memory

Every process, once loaded, occupies a memory image divided into **4 (sometimes 5) segments**:

```
High Memory Address
┌─────────────────────┐
│        Stack         │  ← grows DOWNWARD
│           ↓           │
│                       │
│           ↑           │
│         Heap          │  ← grows UPWARD
├─────────────────────┤
│  Uninitialized Data   │  (.bss)
│   Initialized Data    │  (.bss/.data)
├─────────────────────┤
│    Text / Code        │
└─────────────────────┘
Low Memory Address
```

| Segment | Contents |
|---------|----------|
| **Text / Code** | The compiled program instructions (program counter points here) |
| **Data** | Global and static variables (split into initialized `.data` and uninitialized `.bss`) |
| **Heap** | Memory dynamically allocated during run-time (via `malloc`, etc.); grows **upward** toward higher addresses |
| **Stack** | Temporary data: function parameters, return addresses, local variables; grows **downward** toward lower addresses |

> 💡 **Mnemonic**: Stack and Heap grow **toward each other** — if they collide, you get a **stack overflow / heap collision** (out-of-memory condition).

### Activation Records (Function Call Frames)
Each time a function is called, an **Activation Record** is pushed onto the stack:

$$
\text{Activation Record} = \langle \text{Return Address}, \text{Formal Parameters}, \text{Local Variables} \rangle
$$

Example:
```c
int foo(int k, int l) {
    char c;
    float d;
    int a;
    // ...
}
```
Each call to `foo()` creates a new activation record on the stack containing `k`, `l`, `c`, `d`, `a`, and the return address — this is why **recursive calls consume stack space** proportional to recursion depth.

---

## 5. Memory Layout of a Typical C Program (Galvin-style diagram)

```
High Memory  →  argc, argv
                 stack        ↓ grows down
                 ...
                 heap         ↑ grows up
                 uninitialized data (.bss)
                 initialized data
Low Memory   →   text (code)
```

Mapping of a sample program to memory regions:

```c
#include <stdio.h>
#include <stdlib.h>

int x;              // → uninitialized data (.bss)
int y = 15;          // → initialized data

int main(int argc, char *argv[]) {   // argc, argv → high memory
    int *values;      // → stack (local variable)
    int i;            // → stack

    values = (int *) malloc(sizeof(int) * 5);  // → heap allocation

    for (i = 0; i < 5; i++)
        values[i] = i;

    return 0;
}
```

| Variable/Item | Memory Region |
|---------------|----------------|
| `argc`, `argv` | High memory (top) |
| `values`, `i` (local vars) | Stack |
| Block from `malloc()` | Heap |
| `x` (uninitialized global) | Uninitialized data (.bss) |
| `y = 15` (initialized global) | Initialized data |
| Function code (`main`, etc.) | Text (low memory) |

### Summary: Components of Process in Memory
- Program code = **text section**
- Current activity: **program counter**, **processor registers**
- **Stack**: temporary data (function params, return addresses, local vars)
- **Data section**: global variables
- **Heap**: dynamically allocated run-time memory

> ⚠️ **Critical Rule of Execution**: *"Process execution must progress in sequential fashion."* There is **no parallel execution of instructions within a single, basic process** — instructions execute one after another (Instruction 1 → 2 → 3 → 4...).

---

## 6. Process Attributes

Grouped into 5 categories:

| # | Category | Attributes |
|---|----------|------------|
| 1 | **Identification/Relation** | PID (Process ID), PPID (Parent PID), GID (Group ID) |
| 2 | **CPU-related** | State, Size, Type, Priority, Program Counter (PC), General Registers, Burst Time |
| 3 | **Memory-related** | Memory limits |
| 4 | **Device** | List of open devices |
| 5 | **File** | List of open files |

All these attributes are stored together inside the **Process Control Block (PCB)**, also called the **Process Descriptor**.

The PCB additionally stores:
- **Context** (CPU register state to resume execution later)
- **Environment** (environment variables, etc.)

---

## 7. Process Control Block (PCB) / Process Descriptor

The PCB is a data structure holding **all information the OS needs to manage a process**. Also called **Task Control Block**.

### PCB Contents (the "Passport" of a process)

| Field | Meaning |
|-------|---------|
| **Process State** | running, waiting, ready, etc. |
| **Process Number (PID)** | Unique identifier |
| **Program Counter** | Address of the next instruction to execute |
| **CPU Registers** | Contents of all process-centric registers |
| **CPU Scheduling info** | Priorities, scheduling queue pointers |
| **Memory-management info** | Memory limits / boundaries allocated to process |
| **Accounting info** | CPU time used, elapsed clock time, time limits |
| **I/O status info** | I/O devices allocated, list of open files |

### ⚠️ Common GATE Trap — What is NOT in the PCB
> The **Stack** (and the process's actual data/code) lives in the **process's memory image**, NOT inside the PCB. The PCB only stores a **pointer/limits** related to memory — not the stack contents themselves.

| Contains in PCB? | Item |
|---|---|
| ✅ Yes | Process ID |
| ✅ Yes | Program Counter |
| ✅ Yes | Registers |
| ✅ Yes | Memory limits |
| ✅ Yes | List of open files |
| ❌ **No** | **Stack** (this is in memory, not the PCB) |

> 💡 **Mnemonic**: PCB = the process's **"passport"** — it carries identity & status info, not the process's actual belongings (data/stack).

---

## 8. Process States & State Transition Diagram ⭐

### Basic State Set (Non-Preemptive / Multi-Programmed OS)

$$
\text{States} = \langle \text{New, Ready, Running, Wait/Block, Terminate} \rangle
$$

### State Transition Diagram

```
        Created         Scheduled(Dispatch)      Completion
 New ─────────────→ Ready ───────────────→ Running ───────────→ Terminate
                       ▲                        │
                       │                        │ I/O Request /
                I/O Completion /                │ Service Action
                Event Satisfied                  ▼
                       └──────────────────── Block/Wait
```

| Transition | Trigger |
|------------|---------|
| New → Ready | Process created |
| Ready → Running | Process **scheduled/dispatched** |
| Running → Ready | Time slice expires / preempted (**only in preemptive systems**) |
| Running → Block/Wait | I/O request or waiting for some event/service action |
| Block/Wait → Ready | I/O completed / event satisfied |
| Running → Terminate | Process completes execution |

### Uniprogrammed OS (no Ready state involvement in preemption)
```
New → Running → Terminate
         ↑  │
 I/O Compl. │ I/O Request
         └──┘  (Block/Wait)
```
In a simple **uniprogrammed** system, there's no separate "Ready" queue contention — a process runs until it needs I/O or terminates.

### Extended State Diagram (with Suspend states — for advanced/DFA-based systems)
Adds two more states used in **medium-term scheduling** (swapping to disk):

| State | Meaning |
|-------|---------|
| **Suspend Ready** | Ready process swapped out to disk |
| **Suspend Block** | Blocked process swapped out to disk |

```
Ready ⇄ Suspend Ready  (via "suspend"/"resume", process on disk)
Block/Wait ⇄ Suspend Block  (via "suspend"/"resume", process on disk)
Suspend Block → Suspend Ready (when the awaited event is satisfied while still on disk)
```

> Pre-hard/DFA-based Multi-Programmed OS models incorporate these suspend states to handle memory pressure (swapping).

### Key Rule (GATE-important)
> **No direct transition from Block/Wait → Running.** A blocked process must always go through **Ready** first (it can't jump straight into execution — it must be scheduled/dispatched from Ready).

---

## 9. GATE PYQs & Review Questions

### Q1. What is a process in an operating system?
- A. A program in the secondary memory
- **B. A program in execution ✅**
- C. A set of instructions
- D. A thread of execution

**Why others are wrong**: (A) confuses it with a *program* (passive, on disk); (C) describes just the code/text section, not the active entity; (D) a thread is a unit *within* a process, not the process itself.

---

### Q2. Which of the following is/are NOT a valid Process State?
- **A. Starvation ✅ (not a state)**
- B. Waiting
- **C. Thrashing ✅ (not a state)**
- D. Terminated

**Why**: Starvation and Thrashing are *phenomena/problems* in OS (starvation = indefinite postponement; thrashing = excessive paging), **not** states in the process lifecycle. Waiting and Terminated are valid states.

---

### Q3. The Process Control Block (PCB) does not contain:
- A. Process ID
- B. Program Counter
- **C. Stack ✅ (correct answer — NOT in PCB)**
- D. File descriptors

**Why others are wrong**: PID, Program Counter, and file descriptors are all explicitly part of the PCB; the **Stack** resides in the process's memory image, not inside the PCB itself.

---

### Q4. Which of the following is TRUE about Context Switching?
- A. It is done when a Process completes its execution.
- B. It allows one process to communicate with another.
- **C. It is the process of saving and loading Registers and States. ✅**
- D. It permanently removes the current Process from memory.

**Why others are wrong**: (A) context switch happens on preemption/blocking, not just completion; (B) that's Inter-Process Communication (IPC), a different concept; (D) that describes process termination, not context switching.

---

### Q5. Which component of the OS is responsible for creating and terminating processes?
- A. Scheduler
- B. Loader
- **C. Process Manager ✅**
- D. Shell

**Why others are wrong**: The Scheduler decides *which* process runs next (not creation/termination); the Loader loads a program into memory to start execution but doesn't manage the full lifecycle; the Shell is a user interface for issuing commands, not the OS component itself doing process management.

---

### Q6. What does a process need to perform I/O?
- A. CPU
- B. Registers
- **C. File Descriptors ✅**
- D. Stack

**Why others are wrong**: CPU/Registers/Stack are used for general execution and computation, but performing **I/O specifically requires file descriptors** (or device handles) that identify the I/O resource.

---

### Q7. What is/are the difference(s) between a Process and a Program?
- A. A process is passive, a program is active
- **B. A program is passive, a process is active ✅**
- C. Both are same
- D. A process is always stored in main memory

**Why others are wrong**: (A) is reversed — program is passive (on disk), process is active; (C) they are fundamentally different concepts; (D) while a process's active parts are in RAM, this isn't the defining distinguishing statement being tested here (the correct contrast is passive vs active).

---

### Q8. What is Inter-Process Communication (IPC) used for?
- A. To allocate memory
- **B. To allow processes to communicate with each other ✅**
- C. To schedule CPU time
- D. To interrupt a running process

**Why others are wrong**: (A) memory allocation is a memory-management function; (C) scheduling is done by the CPU scheduler; (D) interrupts are a hardware/OS signaling mechanism, unrelated to IPC's core purpose.

---

### Q9. A Child Process:
- **A. Is created by OS Kernel ✅**
- B. Shares its memory completely with the Parent
- **C. Is a copy of the Parent Process ✅**
- D. Always runs in parallel with the Parent

**Why others are wrong**: (B) a child gets its own memory space (typically copy-on-write, not full sharing); (D) a child does *not always* run in parallel — depends on the scheduler and whether parent waits (`wait()`) for it.

*(Note: this question has two marked-correct options per the source slide — both A and C were marked correct in the lecture.)*

---

### Q10. [GATE Challenge] Preemptive Scheduling — Process State Transitions
Consider the following statements about Process State Transitions for a system using **Preemptive scheduling**:
1. A running process can move to ready state.
2. A ready process can move to running state.
3. A blocked process can move to running state.
4. A blocked process can move to ready state.

Which are TRUE?
- A. I and II
- B. I, II and III
- **C. I, II and IV ✅**
- D. I, II, III and IV

**Why (III) is false**: A blocked process **cannot jump directly to Running** — it must first transition to **Ready**, and only then get scheduled/dispatched into Running. There is no direct Blocked → Running edge in the state diagram.

---

### Q11. [GATE Challenge] Procedure Calls
Which of the following typically occurs when a Procedure call is executed on a processor?
1. Program counter is updated.
2. Stack pointer is updated.
3. Data cache is flushed (to avoid aliasing problems).

- A. I only
- B. II only
- **C. I and II only ✅**
- D. I and III only
- E. I, II, and III

**Why (III) is false**: A procedure call does not require flushing the data cache — cache flushing is unrelated to a normal function call; only PC (jump to function address) and SP (push return address/params/locals) are updated.

---

### Q12. Process State Transition diagram (given) — What does it represent?
- A. A Batch O.S.
- **B. An O.S. with a preemptive scheduler ✅**
- C. An O.S with a non-preemptive scheduler
- D. A Uniprogrammed O.S.

**Why**: The diagram shows a **bidirectional arrow between Running and Ready** (Running ⇄ Ready), which is only possible if the scheduler can **preempt** a running process and send it back to Ready — a signature of **preemptive scheduling**.

---

### Q13. Uni-Processor Process State Transition — Which statements are TRUE?
Given transitions: A (Start→Ready), B (Ready→Running), C (Running→Ready), D (Running→Terminated), E (Blocked→Ready), F (Running→Blocked). Assume Ready state always has some process waiting.

Statements:
1. If a process makes transition D, it results in another process making transition A immediately. — **False**
2. A process P2 in blocked state can make transition E while another process P1 is in running state. — **True**
3. The OS uses Preemptive scheduling. — **True**
4. The OS uses Non-Preemptive scheduling.

- A. I and II
- B. I and III
- **C. II and III ✅**
- D. II and IV

**Why others are wrong**: (I) is false because transition D (termination) doesn't force a *new* process to be created (transition A) — the OS may simply dispatch an already-Ready process; (IV) is false because the presence of the Running→Ready edge (transition C) implies preemption is possible, ruling out purely non-preemptive scheduling.

---

### Q14. Which combination of features suffices to characterize an OS as Multi-Programmed?
(a) More than one program may be loaded into main memory at the same time for execution.
(b) If a program waits for I/O, another program is immediately scheduled.
(c) If a program terminates, another program is immediately scheduled.

- **A. (a) only ✅**
- B. (a) and (b)
- C. (a) and (c)
- D. (a), (b) and (c)

**Why others are wrong**: (b) and (c) describe *scheduling policy behavior* (what happens on I/O wait or termination), which is a separate concern from **multiprogramming**, whose defining property is simply that **multiple programs reside in memory simultaneously** (to keep CPU utilization high). (b)/(c) relate more to how the scheduler picks the next process, not the definition of multiprogramming itself.

---

### Q15. Which of the following process state transitions is/are NOT possible?
- A. Running to Ready
- **B. Waiting to Running ✅ (NOT possible)**
- **C. Ready to Waiting ✅ (NOT possible)**
- D. Running to Terminated

**Why**: A **Waiting** process must go through **Ready** before it can run again — it cannot jump directly to Running. Similarly, a **Ready** process cannot go directly to Waiting — it must first be dispatched to Running, and only *then* can it block on I/O (Running → Waiting).

---

### Q16. A Privileged Instruction may be executed only while the hardware is in Kernel Mode. Which of the following is LEAST likely to be a Privileged Instruction?
- **A. An instruction that changes the value of the program counter ✅ (LEAST likely to be privileged)**
- B. An instruction that sends output to a Printer — privileged
- C. An instruction that modifies a Memory Management Register — privileged
- D. An instruction that halts the CPU — privileged
- E. An instruction that resets the Computer's Time-Of-Day clock — privileged

**Why**: Changing the **program counter** (e.g., via a jump/branch/function call) is a **routine operation that user-mode programs do constantly** (every function call, loop, or conditional branch updates the PC). In contrast, printer I/O, memory-management register changes, halting the CPU, and resetting the system clock are all **system-level operations that could disrupt other processes or hardware**, hence they require kernel/privileged mode.

---

### Q17. Which typically occurs when a Procedure call is executed? *(repeat variant)*
Same as Q11 — Answer: **I and II only** (PC updated, Stack pointer updated; cache flush NOT required).

---

### Q18. In Multi-Programmed systems, for a single copy of a program to be shared by several users, which must be true?
I. The program is macro.
II. The program is recursive.
III. The program is reentrant.

- A. I only
- B. II only
- **C. III only ✅**
- D. I & II only
- E. I, II and III

**Why others are wrong**: A program being **macro** or **recursive** has nothing to do with shareability among multiple users. Only **reentrancy** (the program does not modify its own code and keeps all per-user state in separate stack/data areas) allows a **single copy of code** in memory to be safely executed by multiple processes concurrently.

---

### Q19. Let $t_1$ = time to switch between user and kernel mode, and $t_2$ = time to switch between two processes (context switch). Which is TRUE?

$$t_1 < t_2$$

**Answer: C. $t_1 < t_2$**

**Reasoning**: A full **context switch** (between two different processes) involves *everything* a mode switch does (trap into kernel) **plus** additional overhead: saving/restoring the entire PCB (registers, program counter, memory-management info, etc.) and potentially flushing TLB/cache. A mere **user↔kernel mode switch** (e.g., a system call) is a strict subset of this work — hence $t_1$ (mode switch) is always **less than** $t_2$ (process switch).

---

## 🧠 Memory Tricks & Mnemonics (Recap)

| Trick | Concept |
|-------|---------|
| Program = recipe (static, on disk); Process = meal being cooked (active, in RAM) | Program vs Process |
| Stack ↓ and Heap ↑ grow toward each other | Memory layout |
| PCB = process's "passport" (identity/status), not its "luggage" (data/stack) | What's NOT in PCB |
| A blocked process must "check in" at Ready before it can run — no shortcuts | State transition rule |
| Reentrant code = one shared copy, safe for many users (like a public library book with no scribbling) | Multi-programming/sharing |

---

## ✅ Quick Revision Checklist

- [ ] Difference between **Program** (passive, disk) and **Process** (active, RAM)
- [ ] Static vs Dynamic data allocation (compile-time vs run-time)
- [ ] Process as an **ADT**: Definition, Representation, Operations, Implementation
- [ ] Process memory layout: **Text → Data → Heap → Stack** (low to high memory)
- [ ] Stack grows **down**, Heap grows **up**
- [ ] **Activation Records**: return address, formal parameters, local variables
- [ ] Memory layout of a typical C program (argc/argv, global vars in .data/.bss)
- [ ] **5 categories of Process Attributes**: Identification, CPU-related, Memory-related, Device, File
- [ ] **PCB / Process Descriptor** contents: state, PID, PC, registers, memory limits, accounting info, I/O status
- [ ] ⚠️ **Stack is NOT part of the PCB**
- [ ] Basic process states: **New, Ready, Running, Block/Wait, Terminate**
- [ ] State transitions and their triggers (created, scheduled, dispatched, I/O request, I/O completion, completion)
- [ ] **No direct transition** from Block/Wait → Running (must pass through Ready)
- [ ] Preemptive vs Non-preemptive scheduling — identified via presence/absence of Running→Ready edge
- [ ] Extended states: **Suspend Ready** and **Suspend Block** (swapping to disk)
- [ ] Reentrant programs enable code sharing among multiple users
- [ ] Context switch time ($t_2$) > mode switch time ($t_1$)
- [ ] Privileged instructions vs ordinary instructions (PC updates are NOT privileged; I/O, memory-mgmt registers, halting CPU, clock reset ARE privileged)

---

*Notes compiled from GeeksforGeeks GATE CS&IT — Principles of Operating Systems, Lecture 06: Process Concepts*
