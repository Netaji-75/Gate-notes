# Process Synchronization — Lecture 15 (GATE CS/IT Revision Notes)
### Hardware & Software Synchronization Tools, Priority Inversion, Sleep–Wakeup

---

## 📑 Topics Covered (Quick TOC)

| # | Topic |
|---|-------|
| 1 | Requirements of the Critical Section (CS) Problem |
| 2 | Hardware Synchronization — Disabling Interrupts |
| 3 | Hardware Instructions — TSL, SWAP, Compare-and-Swap (CAS) |
| 4 | Bounded-Wait Solution using CAS |
| 5 | Mutex Locks & Spinlocks |
| 6 | Priority Inversion Problem + Priority Inheritance |
| 7 | Livelock, Deadlock, Starvation — differences |
| 8 | Sleep & Wakeup Mechanism (blocking solution) |
| 9 | Producer–Consumer using Sleep–Wakeup (and its flaw) |
| 10 | Solved Scheduling Example (LRTF) — bonus/aside |
| 11 | GATE PYQs & Review MCQs |

---

## 1. Requirements of the Critical Section (CS) Problem

Any valid CS solution **must** satisfy three conditions:

1. **Mutual Exclusion (M.E.)** – Only one process/thread may execute inside its critical section at a time.
2. **Progress** – If no process is in CS, and some processes want to enter, the decision of who enters next cannot be postponed indefinitely (only competing processes participate in this decision).
3. **Bounded Waiting** – There must exist a bound on the number of times other processes are allowed to enter their CS after a process has requested entry and before its request is granted (prevents starvation).

> 💡 **Mnemonic:** **M-P-B** → **M**utual Exclusion, **P**rogress, **B**ounded Wait — the "3 pillars" every CS solution is graded against in GATE.

---

## 2. Hardware Synchronization — Disabling Interrupts

### 2.1 Idea
Many systems provide hardware support to implement critical sections. On a **uniprocessor**, disabling interrupts prevents preemption of the currently running code.

```
Entry:  Disable ALL Interrupts
        <Critical Section>
Exit:   Enable ALL Interrupts
```

### 2.2 Uniprocessor vs Multiprocessor

| Aspect | Uniprocessor | Multiprocessor |
|---|---|---|
| **Action** | Disable interrupts | Disable interrupts across **all** processors |
| **Result** | Currently running code executes without preemption | Message passing between processors causes severe delays |
| **Verdict** | Works, but creates a rigid, non-responsive system | Highly inefficient; **not broadly scalable** |

### 2.3 Problems With This Approach
- What if the CS code runs for a very long time (e.g., an hour)? The whole system becomes unresponsive.
- Can some processes **starve** — never get to enter their CS?
- Doesn't scale to multiple CPUs (needs inter-processor interrupt disabling → costly message passing).

**Conclusion:** Disabling interrupts is impractical for application programmers and does not scale on multiprocessor systems. This motivates **hardware atomic instructions**.

---

## 3. Hardware Instructions (Atomic Instructions)

Special hardware instructions let us **test-and-modify** a word, or **swap** two words **atomically** (uninterruptedly).

| Instruction | Also Known As |
|---|---|
| Test-and-Set | TSL |
| Swap | Lock & Key based |
| Compare-and-Swap | CAS |

**Key properties of hardware solutions (all three):**
- H/W solutions
- Involve **Busy-Waiting**
- Are **Multi-Process** solutions (work for n processes, not just 2)
- Conceptually an **Extended Lock Variable**

### 3.1 Test-and-Set Lock (TSL)

**Definition (pseudocode):**
```c
boolean Test_and_Set(boolean *target) {
    boolean rv = *target;   // save current value
    *target = true;         // always set to true
    return rv;               // return the OLD value
}
```

**Properties:**
- Executed **atomically**
- Returns the **original** value of `target`
- Always **sets** `target` to `true`

**Assembly-level view (classic Tanenbaum version):**
```asm
enter_region:
    TSL REGISTER, LOCK       ; copy lock to register and set lock to 1
    CMP REGISTER, #0         ; was lock zero?
    JNE enter_region         ; if it was non-zero, lock was set, so loop
    RET                      ; return to caller; critical region entered

leave_region:
    MOVE LOCK, #0            ; store a 0 in lock
    RET                      ; return to caller
```

**Usage (C-style busy-wait lock):**
```c
boolean lock = FALSE;   // shared variable

do {
    while (Test_and_Set(&lock))
        ; /* do nothing — busy wait */
    /* critical section */
    lock = FALSE;
    /* remainder section */
} while (TRUE);
```

⚠️ **This solution does NOT guarantee Bounded Waiting** — a process may keep losing the race to acquire the lock indefinitely (though Mutual Exclusion + Progress hold).

---

### 3.2 SWAP Instruction (Lock & Key Based)

**Definition:**
```c
void SWAP(bool *a, bool *b) {
    bool t;
    t  = *a;     // atomic block
    *a = *b;
    *b = t;
}
```

**Usage — Lock & Key pattern:**
```c
bool lock = FALSE;   // shared

void Process(int i) {   // i = 1 ... n
    bool key;
    while (1) {
        /* a */ Non_CS();
        /* b */ key = TRUE;
        /* c */ do {
                    SWAP(&lock, &key);
                } while (key == TRUE);
        /* d */ <CS>;
        /* e */ lock = FALSE;
    }
}
```

⚠️ Just like TSL, this **fails to satisfy Bounded Waiting** — no guarantee on how long a process waits before entering CS.

---

### 3.3 Compare-and-Swap (CAS)

**Definition:**
```c
int Compare_and_Swap(int *value, int expected, int new_value) {
    int temp = *value;
    if (*value == expected)
        *value = new_value;
    return temp;   // returns ORIGINAL value
}
```

**Properties:**
- Executed atomically
- Returns the **original** value of `value`
- Swap happens **only if** `*value == expected`

**Usage:**
```c
int lock = 0;   // shared, initialized to 0

while (true) {
    while (Compare_and_Swap(&lock, 0, 1) != 0)
        ; /* do nothing */
    /* critical section */
    lock = 0;
    /* remainder section */
}
```

**Does CAS solve the CS problem?**

| Property | Satisfied? |
|---|---|
| Mutual Exclusion | ✅ |
| Progress | ✅ |
| Bounded Wait | ❌ |

---

## 4. Bounded-Wait Solution Using Compare-and-Swap

To fix the Bounded Wait limitation of plain CAS, we add a `waiting[]` array (one flag per process) so processes hand off the lock in a round-robin/FIFO-like fashion.

```c
// shared: int lock = 0; bool waiting[n] = {false};

while (true) {
    /* ---------- ENTRY SECTION ---------- */
    waiting[i] = true;
    key = 1;
    while (waiting[i] && key == 1)
        key = Compare_and_Swap(&lock, 0, 1);
    waiting[i] = false;

    /* critical section */

    /* ---------- EXIT SECTION ---------- */
    j = (i + 1) % n;
    while ((j != i) && !waiting[j])
        j = (j + 1) % n;
    if (j == i)
        lock = 0;
    else
        waiting[j] = false;

    /* remainder section */
}
```

**How it works:** On exit, process `i` scans the `waiting[]` array (starting from `i+1`) to find the *next* waiting process and directly hands it the CS access — this guarantees a bound on how many other processes can cut in line.

> 📝 **Homework flagged in lecture:** Practice tracing through with `P1, P2, P3` to verify Bounded Wait is satisfied.

---

## 5. MUTEX Locks (Software Abstraction)

The hardware-level solutions above are **complicated and generally inaccessible to application programmers**. So OS designers build a simpler software tool: the **mutex lock**.

- **Mutex** = boolean variable indicating whether the lock is available or not.
- Protect a CS by:
  1. First `acquire()` the lock
  2. Then `release()` the lock
- Calls to `acquire()` / `release()` must themselves be **atomic** — usually implemented via hardware atomic instructions like CAS.
- Because this still requires **busy waiting**, this type of lock is called a **spinlock**.

```c
while (True) {
    acquire lock
        critical section
    release lock
    remainder section
}
```

> 💡 **Mnemonic:** Mutex built on CAS + busy-wait = **Spinlock** (it "spins" the CPU while waiting).

---

## 6. Priority Inversion Problem

### 6.1 Setup
- **Busy-waiting mechanism** + **Preemptive priority-based scheduling** together can cause a subtle bug called **Priority Inversion**.

### 6.2 Classic Scenario (Tanenbaum-style, 2 priority levels: H > L)

- Ready queue has `P_L` (low priority) and `P_H` (high priority); CPU currently runs `P_L`.
- `P_L` enters its CS (using a spinlock via TSL/SWAP/CAS/etc.).
- `P_H` arrives, preempts `P_L` (since H > L) and tries to enter the **same** CS.
- `P_H` gets **busy-waiting/spinning** on the lock held by `P_L` — but `P_L` never gets the CPU back (since `P_H` has higher priority and keeps spinning), so `P_L` can **never release the lock**.
- Net effect: the **high-priority process is indirectly blocked forever by the low-priority process** — priorities are effectively "inverted."

**Conceptually this is similar to:**
- Spinlock-based Busy Waiting
- **Deadlock** and **Livelock** (conceptually same failure mode here)

### 6.3 Three-Priority Version (Galvin textbook: L < M < H)

- `P_L` holds the lock and is in CS.
- `P_M` arrives (medium priority) — since `P_M` doesn't need the lock, it simply preempts `P_L` on the CPU (because `M > L`), and now `P_M` runs indefinitely on the CPU.
- `P_H` arrives, needs the same lock → gets blocked in a **block queue** waiting for `P_L` to release it.
- But `P_L` is *not running* (preempted by `P_M`), so it never releases the lock, and `P_H` (highest priority!) is stuck waiting behind `P_M` — a **medium**-priority process is effectively blocking the **highest**-priority process.

### 6.4 Solution — Priority Inheritance

- **Priority Inheritance Protocol:** When a high-priority process needs a resource (lock) held by a lower-priority process, the low-priority process **temporarily inherits** the higher priority until it releases the resource.
- This ensures the lock-holder isn't preempted by medium-priority processes, so it finishes the CS and releases the lock quickly.

**Without Priority Inheritance vs With Priority Inheritance (timeline):**

| | Without Priority Inheritance | With Priority Inheritance |
|---|---|---|
| Low-priority thread | Locks resource (A), runs, later gets preempted (B), waits a long time to unlock (C) | Locks resource (A), briefly boosted to run at high priority when high-priority thread waits (B), quickly unlocks (C) |
| Medium-priority thread | Runs freely while low-priority thread waits — causing the delay | Only runs *after* high-priority thread's request is resolved |
| High-priority thread | Waits a long time for the lock (blocked behind medium-priority work) | Gets the lock quickly once low-priority thread's boosted execution finishes |

> 💡 **Memory trick:** Think of it like a **VIP lending their badge temporarily** to whoever is holding something they need — so that person can finish fast and hand it over, instead of getting stuck behind someone of "medium" importance.

**Real-world analogy from the slides:** A Mars rover scenario — a **high-priority critical task**, a **medium-priority comms task**, and a **low-priority data task**, all sharing one resource (lock). If the data task holds the lock and gets preempted by the comms task, the critical task is stuck waiting — exactly the priority inversion bug (this is based on a real bug found on the **Mars Pathfinder** mission).

---

## 7. Deadlock vs Livelock vs Starvation

| Concept | Definition | Example |
|---|---|---|
| **Deadlock** | Two or more processes/threads are blocked, waiting for each other (an infinite wait) | Traffic gridlock |
| **Livelock** | Processes/threads keep running (are *not* blocked) but make **no progress** — an "active" form of deadlock, because they keep changing state in response to each other without resolving | Two people repeatedly stepping the same direction in a corridor trying to let each other pass; a couple sharing one spoon, each too polite to eat first |
| **Starvation** | Some processes/threads get deferred **forever** because another process dominates and never lets them progress | One process always winning the race for a resource, denying others forever |

**Important relationship (highlighted in lecture):**
> **Deadlock and Livelock both lead to Starvation. But the inverse is NOT necessarily true** — starvation can happen without deadlock or livelock.

**Deadlock vs Starvation — subtle difference:**
- With **starvation**, there always *exists* some schedule that could eventually feed the starving process — it *might* resolve itself if you're lucky.
- Once **deadlock** occurs, it **cannot be resolved by any future schedule** (though other schedules might have *avoided* the deadlock in the first place).

---

## 8. Sleep & Wakeup (Blocking Solution — OS Primitive)

### 8.1 Motivation
Peterson's and TSL-based solutions are logically correct but suffer from **busy waiting**, wasting CPU cycles. **Sleep & Wakeup** is a blocking alternative provided by the OS.

- **`sleep()`** — a system call that **blocks** (suspends) the caller until another process wakes it up.
- **`wakeup(process)`** — takes one parameter: the process to be awakened.
- Blocked processes go into a **Block Queue**, not the Ready Queue, so they don't consume CPU cycles while waiting.

> 💡 **Mnemonic:** Peterson/TSL = "keep checking the door" (busy-wait). Sleep-Wakeup = "go to sleep, someone will knock" (blocking) — much more CPU-efficient.

---

## 9. Producer–Consumer Problem Using Sleep–Wakeup

### 9.1 Code

```c
#define N 100          /* buffer size */
int count = 0;         /* items currently in buffer */

void Producer(void) {
    int item;
    int in = 0;
    while (1) {
        /* a */ item = Produce();
        /* b */ if (count == N)
                    sleep();
        /* c */ Buffer[in] = item;
        /* d */ in = (in + 1) % N;
        /* e */ count = count + 1;
        /* f */ if (count == 1)
                    wakeup(Consumer);
    }
}

void Consumer(void) {
    int item;
    int out = 0;
    while (1) {
        /* a */ if (count == 0)
                    sleep();
        /* b */ item = Buffer[out];
        /* c */ out = (out + 1) % N;
        /* d */ count = count - 1;
        /* e */ if (count == N - 1)
                    wakeup(Producer);
        /* f */ Process(item);
    }
}
```

### 9.2 The Race-Condition Flaw (Classic GATE-relevant bug!)

Because the `count == 0` (or `count == N`) check and the subsequent `sleep()` call are **not atomic**, the following race can occur, leading to a **deadlock**:

**Scenario:**
1. `count = 0`. Consumer checks `if (count == 0)` → true → **about to call `sleep()`**, but gets interrupted **before** actually sleeping.
2. Producer runs: produces an item, `count` becomes `1`, and since `count == 1`, it calls `wakeup(Consumer)`. But Consumer wasn't asleep yet, so **this wakeup signal is lost** (no queued/pending wakeup mechanism).
3. Producer keeps running, buffer fills up (`count == N`), so Producer calls `sleep()`.
4. Now Consumer finally executes its `sleep()` call (from step 1) — Consumer goes to sleep too.
5. **Both Producer and Consumer are now asleep, in the Block Queue** — and neither will ever wake up the other again → **Deadlock**.

> ⚠️ **Root cause:** A **lost wakeup call** — the check-then-sleep sequence is not atomic, and wakeup signals sent to an "awake but about-to-sleep" process are simply lost rather than remembered.
>
> This flaw is exactly why we need proper synchronization primitives like **Semaphores** (next lecture topic) that avoid the lost-wakeup problem by making wakeup counts "stick" even if the process hasn't slept yet.

---

## 10. Bonus/Aside: LRTF Scheduling Numerical (Board Example)

*(This appeared briefly on a scheduling board example unrelated to sync, included for completeness.)*

**Given:**

| Process | Arrival Time (AT) | Burst Time (BT) |
|---|---|---|
| P1 | 0 | 7 |
| P2 | 0 | 7 |
| P3 | 0 | 8 |

**Gantt Chart (LRTF = Longest Remaining Time First, tie broken by lower PID):**

```
| P3 | P2 | P3 | P2 | P3 | P1 | P2 | P3 | P1 | P2 | P3 |
0    4    5    6    7    8    9   10   11   12   13   14
              ↑ P1 completes at some point (marked Pr in slide)
```

**Completion times (from slide):** P1 = 12, P2 = 13, P3 = 14

$$
\text{Average Turnaround Time (Avg. TAT)} = \frac{12 + 13 + 14}{3} = \frac{39}{3} = 13
$$

**Tie-Breaker Rule used:** In case of a tie between processes w.r.t. Burst Time, favor the process with the **lower PID**.

---

## 11. GATE PYQs & In-Lecture Review Questions

### Q1. Why is disabling interrupts not a recommended synchronization technique on multiprocessor systems?
- A. It can't efficiently/can't disable interrupts on all processors ✅
- B. It increases context switching
- C. It reduces cache performance
- D. It increases thread priority

**Answer: A.** Disabling interrupts on multiprocessors requires expensive message passing to every CPU, which doesn't scale. (B, C, D are unrelated/unsupported claims not discussed in the lecture.)

---

### Q2. Which of the following Synchronization Techniques may lead to Busy-Waiting?
- A. Binary Semaphore
- B. Mutex using spinlock ✅
- C. Bakery Algorithm
- D. Peterson Solution

**Answer: B.** A spinlock-based mutex *by definition* busy-waits. (Note: technically Bakery Algorithm and Peterson's solution are *also* busy-waiting solutions in classical OS theory — but per the lecture's quiz, the intended single answer is Mutex/spinlock as the direct example just discussed. Binary semaphores are blocking, not busy-waiting.)

---

### Q3. Which Algorithm solves the Critical Section problem for Two Processes?
- A. Peterson's Algorithm ✅
- B. Dijkstra's Banker's Algorithm
- C. Dekker's Algorithm
- D. FIFO Scheduling

**Answer: A.** Peterson's Algorithm is the classic 2-process software CS solution. (B is for deadlock avoidance, not CS; C — Dekker's is actually also valid for 2 processes but not the answer intended here per the lecture; D is a scheduling policy, unrelated to CS.)

---

### Q4. Which of the following is a Software Synchronization Mechanism in User Mode?
- A. Test-and-Set instruction
- B. Semaphore
- C. Compare-and-Swap instruction
- D. Strict Alternation ✅

**Answer: D.** Strict Alternation is a pure software (no special HW instruction) synchronization approach. (A, C are hardware atomic instructions; B — semaphore is typically an OS-provided/kernel-assisted primitive.)

---

### Q5. What is the primary purpose of Hardware synchronization primitives like Test-and-Set and Compare-and-Swap?
- A. Increase CPU speed
- B. Prevent context switching
- C. Ensure Mutual Exclusion in Critical Section ✅
- D. Schedule processes in round-robin fashion

**Answer: C.** That is their entire design purpose. (A, B, D are unrelated side-effects/false claims.)

---

### Q6. (Repeated) Which Synchronization Techniques may lead to Busy-Waiting?
- A. Binary Semaphore
- B. Mutex using spinlock ✅
- C. Bakery Algorithm
- D. Peterson Solution

**Answer: B** (see Q2 explanation). Context note given in the lecture: *"Spinlocks force a process to loop continuously ('spin') while waiting for a lock, wasting CPU cycles."*

---

### Q7. Why is disabling interrupts not a recommended synchronization technique on Multiprocessor systems? (Repeated)
- A. It can't disable interrupts on all processors ✅
- B. It increases memory usage
- C. It reduces cache performance
- D. It increases thread priority

**Answer: A.** Same reasoning as Q1.

---

### Q8. Which of the following is true about hardware vs software synchronization mechanisms?
- A. Software mechanisms are always faster than hardware
- B. Hardware primitives are not useful for synchronization
- C. Hardware primitives are typically used to build software synchronization tools ✅
- D. Software synchronization cannot work without hardware support

**Answer: C.** As shown in the lecture, mutex/spinlock (a software tool) is implemented using CAS (a hardware primitive). (A and B are false; D is an overstatement — pure software solutions like Peterson's/Bakery exist without special HW instructions.)

---

### Q9. Which of the following is true about a Semaphore?
- A. It is a busy-waiting synchronization mechanism
- B. It uses signal and wait operations ✅
- C. It can only be used for single-threaded applications
- D. It is always binary in nature

**Answer: B.** Semaphores classically use `wait()`/`P()` and `signal()`/`V()` operations. (A is false — semaphores block rather than busy-wait; C is false — used across multi-process/multi-thread; D is false — counting semaphores also exist.)

---

### Q10. What distinguishes a Monitor from a Semaphore?
- A. Semaphores are object-oriented, monitors are not
- B. Monitors use explicit wait and signal functions
- C. Monitors encapsulate shared variables, semaphores do not ✅
- D. Monitors cause more deadlocks than semaphores

**Answer: C.** Monitors bundle shared data + procedures + synchronization into one construct. (A is reversed/false; B is more true of semaphores, monitors use condition variables typically implicitly tied to the monitor; D is an unsupported claim.)

---

### Q11. Which of the following operations causes a Semaphore to decrement?
- A. signal()
- B. raise()
- C. wait() ✅
- D. notify()

**Answer: C.** `wait()` (a.k.a. `P()`) decrements the semaphore; `signal()`/`V()` increments it. (B, D are not standard semaphore operations.)

---

### Q12. In monitors, what happens if a thread calls `wait()`?
- A. It is terminated immediately
- B. It continues execution
- C. It is placed in the monitor's wait queue ✅
- D. It signals another thread to enter the monitor

**Answer: C.** The calling thread is blocked and queued until signaled. (A, B, D contradict the actual blocking semantics of monitor `wait()`.)

---

### Q13. Which of the following problems can Semaphores help solve?
- A. CPU scheduling
- B. Memory fragmentation
- C. Race conditions in critical sections ✅
- D. Page replacement

**Answer: C.** This is the core use-case of semaphores. (A, B, D are unrelated OS subsystems.)

---

### Q14. Which classical IPC problem is mainly concerned with deadlock and starvation among multiple processes sharing limited resources like forks?
- A. Producer-Consumer Problem
- B. Readers-Writers Problem
- C. Dining Philosophers Problem ✅
- D. Sleeping Barber Problem

**Answer: C.** Dining Philosophers is the classic deadlock/starvation-focused problem with shared "fork" resources. (A is about buffer synchronization; B is about read/write access priority; D is about limited waiting-room capacity.)

---

### Q15. In the Producer-Consumer problem, which data structure is most commonly used for managing the buffer between producer and consumer?
- A. Stack
- B. Queue ✅
- C. Tree
- D. Hash Table

**Answer: B.** A circular queue/buffer (FIFO) is used, as seen in the `Buffer[in]`/`Buffer[out]` code above. (A, C, D don't fit the FIFO produce/consume access pattern.)

---

### Q16. Which synchronization tool is best suited to resolve the Readers-Writers problem while prioritizing readers over writers?
- A. Mutex
- B. Counting Semaphore
- C. Read-Write Lock ✅
- D. Binary Semaphore

**Answer: C.** A Read-Write lock natively distinguishes reader vs writer access and can be tuned for reader-priority. (A, D allow only one holder at a time — no differentiation between reader/writer; B alone doesn't inherently express reader-priority policy.)

---

### Q17. What is the major issue addressed in the Sleeping Barber problem?
- A. Resource starvation
- B. Deadlock avoidance
- C. Process scheduling in multi-core systems
- D. Synchronization between a single server (barber) and multiple waiting customers ✅

**Answer: D.** The Sleeping Barber problem models a single-server, multiple-client synchronization scenario with limited waiting room. (A, B, C are related concepts but not the *primary* focus of this specific problem.)

---

### Q18. In the Dining Philosophers problem, which approach helps to prevent deadlock?
- A. Philosophers pick both forks simultaneously
- B. Limiting the number of philosophers allowed to sit at the table ✅
- C. Allowing philosophers to eat without thinking
- D. Giving each philosopher only one fork

**Answer: B.** Limiting concurrent diners to n-1 (out of n) is a classic deadlock-prevention technique. (A is actually also a valid solution in some formulations, but the option marked correct in the lecture is B; C is irrelevant; D guarantees deadlock since no one could ever get 2 forks.)

---

### Q19. Which of the following problems is NOT typically associated with concurrent programming?
- A. Race conditions
- B. Deadlocks
- C. Starvation
- D. Compilation errors ✅

**Answer: D.** Compilation errors are a language/syntax issue, unrelated to runtime concurrency. (A, B, C are all classic concurrency hazards.)

---

### Q20. Which synchronization mechanism ensures mutual exclusion?
- A. Semaphore ✅
- B. Compiler
- C. Garbage Collector
- D. Stack

**Answer: A.** Semaphores (and mutexes) are designed specifically to enforce mutual exclusion. (B, C, D are unrelated system components.)

---

### Q21. In concurrent programming, what is a deadlock?
- A. A bug in the syntax of the code
- B. A condition where threads wait indefinitely for resources ✅
- C. A memory overflow situation
- D. A compilation failure

**Answer: B.** Matches the formal deadlock definition covered in Section 7. (A, C, D are unrelated errors.)

---

### Q22. Which of the following is an example of non-preemptive scheduling in concurrency?
- A. Round Robin
- B. First Come First Serve ✅
- C. Shortest Remaining Time First
- D. Multilevel Queue

**Answer: B.** FCFS is non-preemptive by nature — once a process starts, it runs to completion. (A is preemptive by design via time-slicing; C is a preemptive variant of SJF; D can include preemptive sub-queues.)

---

### GATE PYQ (Strict Alternation — Mutual Exclusion Failure)

**Q23.** Consider the code below for two processes P1, P2 using shared variables `wants1`, `wants2` (both init to FALSE):

```c
/* P1 */                          /* P2 */
while (TRUE) {                    while (TRUE) {
    wants1 = TRUE;                    wants2 = TRUE;
    while (wants2 == TRUE);           while (wants1 == TRUE);
    /* Critical Section */            /* Critical Section */
    wants1 = FALSE;                   wants2 = FALSE;
}                                  }
/* Remainder section */           /* Remainder section */
```

- A. It does not ensure Mutual Exclusion.
- B. It does not ensure Bounded Waiting.
- C. It requires that processes enter the critical section in Strict Alternation. ✅
- D. It does not prevent Deadlocks, but ensures Mutual Exclusion.

**Answer: C.** Since both `wants1 = TRUE` and `wants2 = TRUE` set *before* checking the other's flag, if both execute concurrently, **both** flags become TRUE, and each spins forever on the other's flag → **deadlock**, *not* strict alternation issue per se — however, per lecture's marked answer, the flaw framed is about entry order strictness. (Note: this is a variant of the classic "flag-based" naive solution; watch for GATE variants where both options A and C get debated — always re-derive by hand.)

*(Reasoning shortcut for GATE: trace both processes setting their flag TRUE "simultaneously" before checking the other's flag — this is the signature setup that leads to mutual-exclusion or deadlock issues in flag-based solutions; always simulate step-by-step.)*

---

### GATE PYQ (varP / varQ Synchronization Construct)

**Q24.** Two processes X, Y access CS via:

```c
/* Process X */                        /* Process Y */
while (true) {                         while (true) {
    varP = true;                           varQ = true;
    while (varQ == true)                   while (varP == true) {
    {                                           /* critical section */
        /* critical section */                 varQ = false;
        varP = false;                      }
    }
}                                       }
```
`varP`, `varQ` are shared, both initialized to `false`.

- A. The proposed solution prevents deadlock but fails to guarantee Mutual Exclusion
- B. The proposed solution guarantees Mutual Exclusion but fails to prevent Deadlock ✅
- C. The proposed solution guarantees Mutual Exclusion and prevents Deadlock
- D. The proposed solution fails to prevent Deadlock and fails to guarantee Mutual Exclusion

**Answer: B.** This is the classic "opposite of strict alternation" flag-swap pattern: if both `varP = true` and `varQ = true` execute before either checks the other's flag, **both processes spin forever** in their `while` loop → **deadlock**. However, whenever one *does* get in, only one at a time is ever inside CS → **Mutual Exclusion holds**. (A is wrong since deadlock *is* possible; C is wrong because deadlock *can* occur; D is wrong because ME does hold.)

---

### GATE PYQ (Test-and-Set based CS — Deadlock/Starvation/FIFO/Concurrent-entry)

**Q25.** `enter_CS(X)` / `leave_CS(X)` implemented via test-and-set:
```c
void enter_CS(X) {
    while (test-and-set(X));
}
void leave_CS(X) {
    X = 0;
}
```
X is a shared memory location, initialized to 0. Consider:
- I. The above solution to the CS problem is deadlock-free.
- II. The solution is starvation-free.
- III. The processes enter CS in FIFO order.
- IV. More than one process can enter CS at the same time.

Which of the above statements are TRUE?
- A. I only ✅
- B. I and II
- C. II and III
- D. IV only

**Answer: A.** TSL-based mutual exclusion is **deadlock-free** (some process always succeeds — Progress holds) but **not starvation-free** (no Bounded Wait guarantee — a specific process could keep losing the race forever), **not FIFO** (no ordering guarantee), and **only one** process can be in CS at a time (test-and-set guarantees ME) — so II, III, IV are all false.

---

### GATE PYQ (Fetch-and-Add based Busy-Wait Lock)

**Q26.** `Fetch_And_Add(X, i)` atomically reads `X`, increments it by `i`, and returns the **old** value. `L` is shared, unsigned integer, initialized to 0 (0 = lock available, non-zero = lock held).

```c
AcquireLock(L) {
    while (Fetch_And_Add(L, 1))
        L = 1;
}
ReleaseLock(L) {
    L = 0;
}
```

- A. Fails as L can overflow
- B. Fails as L can take on a non-zero value when the lock is actually available ✅
- C. Works correctly but may starve some processes
- D. Works correctly without starvation

**Answer: B.** Because `Fetch_And_Add` **always increments** L (even for a thread that fails to get the lock, it may still leave L in an inconsistent state) — once a competing thread does `Fetch_And_Add(L, 1)` and later fails, it forcibly sets `L = 1` in the loop; but multiple failing threads can push `L` to values > 1 while the actual lock-holder release sets it only to 0 momentarily, so the acquire logic and the release logic **don't correctly coordinate** — this can leave `L` in a non-zero state even when no one is genuinely holding the lock. (A is a possible but non-primary issue since the real logical bug is the mismatched semantics; C, D are wrong because the algorithm is fundamentally broken, not just prone to unfairness.)

---

## 12. Memory Tricks / Mnemonics Recap 🧠

| Concept | Mnemonic |
|---|---|
| CS Problem Requirements | **M-P-B**: Mutual Exclusion → Progress → Bounded Wait |
| Hardware Atomic Instructions | All 3 (TSL/SWAP/CAS) = H/W + Busy-Wait + Multi-process + "Extended Lock Variable" |
| Mutex + CAS + Busy-wait | = **Spinlock** ("spins" the CPU while waiting) |
| Priority Inversion Fix | **Priority Inheritance** — low-priority lock-holder temporarily "borrows" high priority to finish fast |
| Deadlock vs Livelock | Deadlock = frozen & blocked; Livelock = "busy but stuck" (like two polite people blocking a hallway) |
| Deadlock vs Starvation | Starvation *might* resolve itself (lucky schedule exists); Deadlock *never* resolves once it happens |
| Sleep-Wakeup vs Busy-wait | Busy-wait = "keep checking the door"; Sleep-Wakeup = "sleep till someone knocks" (CPU-efficient, but risks **lost wakeup**) |
| Producer-Consumer bug | The "**lost wakeup**" problem — check-then-sleep isn't atomic → leads to deadlock; motivates semaphores |

---

## 13. ✅ Quick Revision Checklist

Use this to self-test before deep re-reading:

- [ ] Can you state and explain all 3 CS problem requirements (ME, Progress, Bounded Wait)?
- [ ] Why does disabling interrupts fail on multiprocessor systems?
- [ ] Can you write the pseudocode for **Test-and-Set (TSL)** from memory, and explain why it fails Bounded Wait?
- [ ] Can you write the **SWAP**-based lock & key solution?
- [ ] Can you write **Compare-and-Swap (CAS)** and explain its atomic semantics (`temp`, `expected`, `new_value`)?
- [ ] Can you describe the **Bounded-Wait fix using CAS + waiting[] array**?
- [ ] What is a **Mutex Lock**, and why is it also called a **spinlock**?
- [ ] Can you explain the **Priority Inversion problem** with both the 2-level (H,L) and 3-level (L,M,H) examples?
- [ ] What is **Priority Inheritance**, and how does it fix priority inversion? Can you sketch the before/after timeline diagram?
- [ ] Can you distinguish **Deadlock vs Livelock vs Starvation** with examples?
- [ ] Why does Deadlock/Livelock imply Starvation, but not vice versa?
- [ ] What is the difference between **Sleep-Wakeup** and busy-waiting solutions?
- [ ] Can you write the **Producer-Consumer using Sleep-Wakeup** code and trace through the **lost wakeup / deadlock scenario** step by step?
- [ ] Do you understand why this lost-wakeup flaw motivates the need for **Semaphores** (next topic)?
- [ ] Can you solve a basic Gantt chart / Average TAT numerical (bonus LRTF example)?
- [ ] Have you attempted all GATE PYQs above without peeking at answers first?

---

*Compiled from GeeksforGeeks GATE CS/IT — "Principles of Operating Systems," Lecture 15: Process Synchronization (III).*
