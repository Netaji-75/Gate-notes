# Process Synchronization — Lecture 17 (GATE CS/IT Revision Notes)
### Principles of Operating Systems | Semaphores • Monitors • Classical IPC Problems • Concurrency

---

## 📑 Table of Contents

| # | Topic |
|---|-------|
| 1 | Mutex Locks (recap) |
| 2 | Mutex vs Semaphore |
| 3 | Problems with Semaphores (misuse patterns) |
| 4 | Liveness Properties — Deadlock, Starvation, Priority Inversion |
| 5 | Monitors — concept, syntax, semantics |
| 6 | Monitor vs Semaphore |
| 7 | Producer–Consumer using Monitors (full code) |
| 8 | Classical IPC Problems (overview) |
| 9 | Quick-Check MCQs (concept checks) |
| 10 | Review MCQs (full set, answer + reasoning) |
| 11 | GATE PYQs — worked out (Peterson/Dekker variants, TAS, Fetch-and-Add, Semaphore tracing, Monitors, Barrier code) |
| 12 | Memory Tricks / Mnemonics |
| 13 | Quick Revision Checklist |

---

## 1. Mutex Locks (Recap)

- Earlier solutions (Peterson's, Dekker's, hardware instructions) are **correct but complicated** and generally **inaccessible to application programmers**.
- OS designers therefore build a simpler **software tool**: the **mutex lock**.
- A mutex is essentially a **Boolean variable** indicating whether the lock is available or not.
- Usage pattern to protect a critical section:
  1. **`acquire()`** the lock
  2. Execute the **critical section**
  3. **`release()`** the lock
- `acquire()` and `release()` **must be atomic** — usually implemented using hardware atomic instructions such as **compare-and-swap (CAS)**.
- **Drawback:** This solution requires **busy waiting** → hence called a **spinlock**.

```c
while (TRUE) {
    acquire_lock();
    /* critical section */
    release_lock();
    /* remainder section */
}
```

> ⚠️ **Busy waiting wastes CPU cycles** — the process keeps looping ("spinning") instead of sleeping while it waits.

---

## 2. Mutex vs Semaphore

| Aspect | Mutex | Semaphore |
|---|---|---|
| Mechanism type | **Locking mechanism** | **Signaling mechanism** |
| Data type | An **object** | An **integer variable** |
| Types | Does **not** have types | Two types: **Binary** and **Counting** |
| Deadlock risk | **Higher** risk of deadlock | **Lower** risk of deadlock |
| Ownership | Typically owned by the locking thread | No inherent ownership — any process can signal |

---

## 3. Problems with Semaphores (Misuse Patterns)

Semaphores are powerful but **error-prone** because correctness depends entirely on the programmer using `wait()`/`signal()` (a.k.a. `P()`/`V()`) correctly. Common misuse patterns:

- **Reversed order:** `signal(mutex) ... wait(mutex)` → critical section is **not protected at all**.
- **Double wait:** `wait(mutex) ... wait(mutex)` → causes the calling process to **block on itself** (deadlock).
- **Omitting `wait()` and/or `signal()`** entirely → mutual exclusion is silently broken.

```c
// Correct usage pattern (BSEM = Binary Semaphore)
BSEM mutex = 1;

P(mutex);
   <CS>              // Critical Section
V(mutex);
```

> 🧠 **These bugs are exactly why Monitors were invented** — to let the language/runtime manage entry & exit automatically instead of trusting every programmer to call `wait`/`signal` correctly.

### Illustrative Deadlock via Wrong Ordering
Two binary semaphores `S = 1`, `T = 1`, used by two processes in **opposite order**:

```text
Process i                  Process j
---------                  ---------
P(S);                       P(T);
P(T);                       P(S);
<CS>                        <CS>
V(T);                       V(S);
V(S);                       V(T);
```
Because `P_i` acquires `S` then `T`, while `P_j` acquires `T` then `S`, a **circular wait** is possible → classic **deadlock (deadly embrace)**. Mutual exclusion still holds, but **deadlock freedom is NOT guaranteed.**

---

## 4. Liveness Properties

**Liveness Property:** the set of properties a system must satisfy to guarantee that processes **keep making progress** (i.e., don't stall forever).

- Processes may **wait indefinitely** while trying to acquire a mutex/semaphore.
- Indefinite waiting **violates Progress and Bounded-Waiting** (the two core critical-section requirements).
- **Indefinite waiting = a Liveness Failure.**

### 4.1 Deadlock
Two or more processes wait **infinitely** for an event that can only be triggered by one of the *other* waiting processes.

**Example:** `S`, `Q` are two semaphores, both initialized to `1`.

```text
P0                    P1
--                    --
wait(S);              wait(Q);
wait(Q);              wait(S);
...                   ...
signal(S);            signal(Q);
signal(Q);            signal(S);
```
If `P0` executes `wait(S)` and `P1` executes `wait(Q)` first, then:
- `P0` blocks on `wait(Q)` until `P1` calls `signal(Q)`.
- `P1` blocks on `wait(S)` until `P0` calls `signal(S)`.
- Neither `signal()` will ever execute → **P0 and P1 are deadlocked.**

### 4.2 Other Liveness Failures

| Failure | Description |
|---|---|
| **Starvation** | *Indefinite blocking.* A process may never be removed from the semaphore's waiting queue. |
| **Priority Inversion** | A scheduling problem where a **lower-priority** process holds a lock needed by a **higher-priority** process, letting medium-priority processes run instead. Solved via the **Priority-Inheritance Protocol**. |

---

## 5. Monitors

### 5.1 Why Monitors?
Monitors (Hoare & P.B. Hansen, 1974) are a **high-level synchronization construct** that lets an abstract data type be safely shared among concurrent processes **without** the programmer manually calling `wait`/`signal` for mutual exclusion.

> 🔑 **Golden Rule:** Only **ONE active process at a time** can be inside the monitor.

### 5.2 What Is a Monitor? — Key Properties
- A **highly structured programming-language construct** consisting of:
  - **Private variables & private procedures** — usable only inside the monitor.
  - A **constructor / initialization code** to initialize the monitor.
  - A number of **public (entry) procedures** invokable by users.
- **Monitors have NO public data** — everything is encapsulated.
- Think of a monitor as a **"mini-OS"**, where the monitor's entry procedures act like **system calls**.
- Processes running **outside** the monitor **cannot** directly access the monitor's internal data — they can only invoke its public procedures.

### 5.3 Syntax of a Monitor

```text
monitor monitor_name
{
    // shared variable declarations

    procedure P1( ... ) { ... }
    procedure P2( ... ) { ... }
    ...
    procedure Pn( ... ) { ... }

    initialization code ( ... ) { ... }
}
```

### 5.4 Condition Variables
- Monitors may include **special condition variables**, each supporting only two operations:
  - **`wait()`** — the calling process is **suspended** and placed into that condition variable's own queue.
  - **`signal()`** — wakes up a process waiting on that condition (if any).
- If a thread calls `wait()`, it is **placed in the monitor's wait queue** for that condition — it does NOT terminate and does NOT continue.

### 5.5 Schematic View of a Monitor
```
              entry queue
                  |
 ┌───────────────────────────────┐
 │        shared data            │
 │  ┌────┐ ┌────┐      ┌────┐    │
 │  │ P1 │ │ P2 │ .... │ Pn │    │  ← operations (only 1 active at a time)
 │  └────┘ └────┘      └────┘    │
 │      initialization code      │
 └────────────────────────────────┘
   + condition-variable queues (x, y, ...) hold blocked processes
```

---

## 6. Monitor vs Semaphore

| Aspect | Semaphore | Monitor |
|---|---|---|
| Synchronization style | Programmer manually calls `wait()`/`signal()` everywhere | Mutual exclusion is **automatic**; only condition variables need explicit `wait()`/`signal()` |
| Data encapsulation | Shared data is **exposed**, programmer must protect it manually | **Encapsulates shared variables** — data is private to the monitor |
| Error-proneness | High — easy to misuse (see Section 3) | Low — structured, compiler/runtime enforces the mutual-exclusion rule |
| Level | Low-level primitive | High-level language construct |

---

## 7. Producer–Consumer Using Monitors

```c
monitor ProducerConsumer
{
    condition full, empty;
    int count;

    procedure enter()
    {
        if (count == N) wait(full);      // if buffer is full, block
        put_item(widget);                 // put item in buffer
        count = count + 1;                // increment count of full slots
        if (count == 1) signal(empty);    // if buffer was empty, wake consumer
    }

    procedure remove()
    {
        if (count == 0) wait(empty);      // if buffer is empty, block
        remove_item(widget);               // remove item from buffer
        count = count - 1;                 // decrement count of full slots
        if (count == N - 1) signal(full);  // if buffer was full, wake producer
    }

    count = 0;
} // end monitor

Producer()
{
    while (TRUE)
    {
        make_item(widget);           // make a new item
        ProducerConsumer.enter();    // call enter function in monitor
    }
}

Consumer()
{
    while (TRUE)
    {
        ProducerConsumer.remove();   // call remove function in monitor
        consume_item();              // consume an item
    }
}
```

> 🧠 **Mnemonic:** Notice `enter()`/`remove()` themselves never call `wait(mutex)`/`signal(mutex)` for entry — the **monitor guarantees only one process is active inside it at a time.** The only explicit `wait`/`signal` calls are on the **condition variables** (`full`, `empty`) to manage the *buffer-full* / *buffer-empty* states.

---

## 8. Classical IPC Problems (Overview)

These are the standard **applications of Semaphores and Monitors** used to test synchronization understanding:

- **Producer–Consumer (Bounded Buffer) Problem** — uses a **Queue** for the shared buffer.
- **Readers–Writers Problem** — best modeled with a **Read-Write Lock** (built from counting semaphores) when readers should be prioritized over writers.
- **Dining Philosophers Problem** — concerned with **deadlock & starvation** among processes sharing limited resources (forks). Deadlock is commonly prevented by **limiting the number of philosophers allowed to sit at the table simultaneously** (allow at most N−1).
- **Sleeping Barber Problem** — models synchronization between a **single server (barber)** and **multiple waiting clients (customers)** with a bounded number of waiting chairs.

---

## 9. Quick-Check Concept MCQs

**Q1. Why is disabling interrupts *not* a recommended synchronization technique on Multiprocessor systems?**
- ✅ **A. It can't efficiently disable interrupts on all processors.**
- ❌ B. It increases context switching — not the core reason.
- ❌ C. It violates mutual exclusion — it actually can enforce it on a single CPU, just not efficiently across many.
- ❌ D. It causes immediate system crashes — false.

**Q2. Which synchronization techniques may lead to Busy-Waiting?**
- ✅ **Mutex using spinlock, Bakery Algorithm, Peterson's Solution** — all involve a `while` loop that keeps checking a condition (spinning).
- ❌ Binary Semaphore — a properly implemented semaphore **blocks/sleeps** the process instead of spinning.

---

## 10. Review MCQs — Full Set (Answer + Reasoning)

| # | Question | Correct Answer | Why others are wrong |
|---|---|---|---|
| 1 | Which algorithm(s) solve the Critical Section problem for **two processes**? | **A. Peterson's Algorithm** & **C. Dekker's Algorithm** (MSQ) | B (Banker's Algo) is for **deadlock avoidance**, not CS; D (FIFO) is a **CPU scheduling** algorithm. |
| 2 | Which is a **software** synchronization mechanism in **user mode**? | **D. Strict Alternation** | A & C (TAS, CAS) are **hardware** instructions; B (Semaphore) typically needs OS/kernel support. |
| 3 | Primary purpose of hardware primitives like TAS/CAS? | **C. Ensure Mutual Exclusion in Critical Section** | They don't speed up the CPU, prevent context switching, or schedule processes — they atomically test/modify memory to enforce exclusion. |
| 4 | Which technique(s) may lead to Busy-Waiting? | **B, C, D** — Mutex(spinlock), Bakery, Peterson | Binary Semaphore (A) blocks instead of spinning. |
| 5 | Why is disabling interrupts bad on multiprocessor systems? | **A. It can't disable interrupts on all processors** | Other options (memory, cache, priority) aren't the real issue. |
| 6 | True statement about hardware vs software sync mechanisms? | **C. Hardware primitives are typically used to build software synchronization tools** | Software is not "always faster"; hardware primitives (TAS/CAS) *are* useful; software sync (spinlocks) can be built purely on hardware ops, but higher-level tools like monitors don't strictly need low-level HW visible to the app. |
| 7 | True statement about a Semaphore? | **B. It uses signal and wait operations** | It's not restricted to busy-waiting, not single-threaded only, and not always binary (counting semaphores exist). |
| 8 | What distinguishes a Monitor from a Semaphore? | **C. Monitors encapsulate shared variables, semaphores do not** | Both can be object-oriented; monitors auto-manage mutual exclusion rather than needing explicit wait/signal for entry; monitors don't inherently cause *more* deadlocks. |
| 9 | Which operation decrements a Semaphore? | **C. `wait()`** | `signal()`/`notify()`/`raise()` all increment or aren't semaphore ops. |
| 10 | What happens when a thread calls `wait()` inside a monitor? | **C. It is placed in the monitor's wait queue** | It is not terminated, does not keep running, and doesn't itself signal another thread. |
| 11 | Which problem(s) can Semaphores help solve? | **C. Race conditions in critical sections** | CPU scheduling, memory fragmentation, and page replacement are unrelated OS subsystems. |
| 12 | Classical IPC problem centered on deadlock/starvation over shared "forks"? | **C. Dining Philosophers Problem** | Producer-Consumer is about buffer sync; Readers-Writers is about read/write access; Sleeping Barber is about a single-server queue. |
| 13 | Data structure typically used for the buffer in Producer-Consumer? | **B. Queue** | Stack/Tree/Hash Table don't preserve the FIFO order producers/consumers need. |
| 14 | Best tool for Readers-Writers giving readers priority? | **C. Read-Write Lock** | Plain Mutex is too restrictive (only 1 reader at a time); Binary Semaphore alone can't distinguish reader-priority logic; Counting Semaphore alone needs extra logic — Read-Write Lock is the purpose-built abstraction. |
| 15 | Major issue in the Sleeping Barber problem? | **D. Synchronization between a single server and multiple clients** | Not fundamentally about starvation, deadlock avoidance, or multi-core scheduling — it's a bounded-waiting-room producer-consumer-style problem. |
| 16 | Approach to prevent deadlock in Dining Philosophers? | **B. Limiting the number of philosophers allowed to sit at the table** (to N−1) | Picking both forks simultaneously without ordering can still race; eating without thinking isn't relevant; one fork per philosopher can't allow eating at all. |
| 17 | NOT typically a concurrent-programming problem? | **D. Compilation errors** | Race conditions, deadlocks, starvation are all classic concurrency issues; compile errors are a static/language issue. |
| 18 | Which mechanism ensures mutual exclusion? | **A. Semaphore** | Compiler, Garbage Collector, Stack are not synchronization tools. |
| 19 | What is a deadlock in concurrent programming? | **B. A condition where threads wait indefinitely for resources** | Not a syntax bug, memory overflow, or compile failure. |
| 20 | Example of non-preemptive scheduling? | **B. First Come First Serve (FCFS)** | Round Robin, SRTF, and Multilevel Queue (with time-slicing) are (or can be) preemptive. |

---

## 11. GATE PYQs — Fully Worked Out

### PYQ 1 — `wants1`/`wants2` Flag-Only Solution (No Turn Variable)

```c
/*P1*/                          /*P2*/
while (TRUE) {                  while (TRUE) {
  wants1 = TRUE;                  wants2 = TRUE;
  while (wants2 == TRUE);         while (wants1 == TRUE);
  /* Critical Section */          /* Critical Section */
  wants1 = FALSE;                 wants2 = FALSE;
}                                }
/* Remainder section */          /* Remainder section */
```

**Options:**
- A. It does not ensure Mutual Exclusion.
- B. It does not ensure Bounded Waiting.
- C. It requires that processes enter the critical section in Strict Alternation.
- D. It does not prevent Deadlocks, but ensures Mutual Exclusion.

✅ **Answer: D**

**Reasoning:** If both processes set their own flag (`wants1=TRUE`, `wants2=TRUE`) at nearly the same time, **both** will see the other's flag as `TRUE` and loop forever → **deadlock is possible**. However, mutual exclusion *does* hold: a process can only enter the CS if, at the moment it checked, the other's flag was `FALSE` — so **both cannot be inside the CS simultaneously.**

---

### PYQ 2 — `varP`/`varQ` Flag-Only Solution (Process X & Y)

```c
Process X                        Process Y
/* other code */                 /* other code */
while (true) {                   while (true) {
   varP = true;                    varQ = true;
   while (varQ == true) {          while (varP == true) {
      /* critical section */          /* critical section */
      varP = false;                    varQ = false;
   }                                }
}                                }
```
*(`varP`, `varQ` are shared, both initialized to `false`.)*

**Options:**
- A. Prevents deadlock but fails to guarantee Mutual Exclusion
- B. Guarantees Mutual Exclusion but fails to prevent Deadlock
- C. Guarantees Mutual Exclusion and prevents Deadlock
- D. Fails to prevent Deadlock and fails to guarantee Mutual Exclusion

✅ **Answer: B**

**Reasoning:** This is structurally identical to PYQ 1 — each process sets its **own** flag first, then waits on the **other's** flag. If both processes set their flags simultaneously, each sees the other's flag `true` and **both block forever → deadlock possible**. But Mutual Exclusion is still guaranteed by the same argument as before.

> 🧠 **Pattern to remember:** *"Flag-only" (no shared `turn` variable) solutions → always Mutual-Exclusion-safe but Deadlock-prone.* Peterson's algorithm fixes this exact flaw by adding a `turn` variable.

---

### PYQ 3 — Test-and-Set Lock (GATE 2014)

```c
void enter_CS(X) {
    while (test_and_set(X));
}

void leave_CS(X) {
    X = 0;
}
```
*(`X` is a shared memory location associated with the CS, initialized to `0`.)*

**Statements:**
- I. The solution is deadlock-free.
- II. The solution is starvation-free.
- III. Processes enter the CS in FIFO order.
- IV. More than one process can enter the CS at the same time.

**Options:** A. I only  B. I and II  C. II and III  D. IV only

✅ **Answer: A (I only)**

**Reasoning:**
- **I is TRUE** — some process will always eventually succeed (no circular waiting condition exists); it's **deadlock-free**.
- **II is FALSE** — there's no fairness guarantee; a process can be perpetually unlucky (**starvation is possible**).
- **III is FALSE** — no ordering guarantee; entry order is **not FIFO**.
- **IV is FALSE** — `test_and_set` correctly enforces **mutual exclusion**, so more than one process **cannot** be inside the CS simultaneously.

---

### PYQ 4 — Fetch-And-Add Lock (GATE 2014)

`Fetch_And_Add(X, i)` is an atomic Read-Modify-Write instruction: reads `X`, increments it by `i`, and returns the **old** value.

```c
AcquireLock(L) {
    while (Fetch_And_Add(L, 1))
        L = 1;
}

ReleaseLock(L) {
    L = 0;
}
```
*(`L` is unsigned int, initialized to `0`; `0` = lock available, non-zero = lock unavailable.)*

**Options:**
- A. Fails as L can overflow
- B. Fails as L can take a non-zero value when the lock is actually available
- C. Works correctly but may starve some processes
- D. Works correctly without starvation

✅ **Answer: B**

**Reasoning:** Every call to `Fetch_And_Add(L,1)` **increments `L` by 1**, regardless of whether the lock was free or not. If the `while` condition is true (lock busy) the process sets `L = 1` again — but this doesn't "roll back" the increments made by other **failed** attempts. Because unsuccessful callers still increment `L`, `L` can drift to a **non-zero value even after the lock is released**, breaking correctness — this is a **flawed implementation**, not just a starvation issue.

```
int FA(int *P, int i) {
    int rv;
    rv = *P;
    *P = *P + i;
    return(rv);
}
```

---

### PYQ 5 — Counting Semaphore Blocking (Fill in the Blank, GATE 2020)

> Consider a Counting Semaphore `S`. `P(S)` decrements `S`; `V(S)` increments `S`. During execution, **20 `P(S)`** and **12 `V(S)`** operations are issued in some order. What is the **largest initial value of `S`** for which **at least one `P(S)` will remain blocked**?

✅ **Answer: 7**

**Reasoning (approach):** In the worst case for the "blocking" process, arrange as many `V(S)` operations as possible **before** the 20th `P(S)` call to maximize how high `S` can climb, while still ensuring the semaphore hits a negative/blocking state at least once. Working through the arithmetic of 20 decrements vs 12 increments, the **largest initial value that still forces at least one block is 7** — for any initial value ≥ 8, it becomes possible to order the operations so that no `P(S)` ever blocks.

---

### PYQ 6 — Two Semaphores, Reversed Acquire Order

```c
BSEM S = 1, T = 1;

Pri {                       Prj {
  while (1) {                 while (1) {
    P(S); P(T);                 P(T); P(S);
    <CS>                         <CS>
    V(T); V(S);                 V(S); V(T);
  }                            }
}                            }
```

**Analyze for Mutual Exclusion and Deadlock Freedom:**

✅ **Mutual Exclusion:** **Guaranteed** — only one process can be holding both `S` and `T` at once.
❌ **Deadlock Freedom:** **NOT guaranteed** — `Pri` acquires `S` then `T`; `Prj` acquires `T` then `S` (**reversed order**). If `Pri` grabs `S` while `Prj` grabs `T` at the same time, each then blocks waiting for the resource held by the other → **classic circular-wait deadlock.**

> 🧠 **Golden Rule reminder:** *Always acquire multiple locks in the **same global order** across all processes to avoid circular-wait deadlocks.*

---

### PYQ 7 — Output Pattern Tracing

```c
BSEM S = 1, T = 1;

Pri {                       Prj {
  while (1) {                 while (1) {
    P(T);                        P(S);
    print('1');                  print('0');
    print('1');                  print('0');
    V(S);                        V(T);
  }                            }
}                            }
```

**Question:** What output is generated by these two concurrent processes?

✅ **Answer:** The output repeats in blocks of `0011`, i.e.:
```
0011 0011 0011 ...   ≡   (0011)+
```
**Reasoning:** `Prj` runs first (only it can pass `P(S)` initially since `T` starts unavailable for `Pri` in this configuration), printing `"00"`, then signals `T`, allowing `Pri` to print `"11"` and signal `S` back — creating a strict alternating **4-character cycle**: `0`,`0`,`1`,`1`, repeating indefinitely.

---

### PYQ 8 — Circular Resource-Sharing (5 Processes, "Dining-Philosophers-style")

```c
BSEM m[0..4] = {1,1,1,1,1};

Pri, i = 0..4 {
    while (1) {
        P(m[i]);
        P(m[(i+1) % 4]);
        <CS>
        V(m[i]);
        V(m[(i+1) % 4]);
    }
}
```

**Q: Does the code guarantee Mutual Exclusion? If not, what's the max number of processes in CS? What about Deadlock Freedom?**

**Trace of resources each process locks:**

| Process | Locks (using `i+1 mod 4`) | Conflicts with |
|---|---|---|
| P0 | m[0], m[1] | P1, P4 |
| P1 | m[1], m[2] | P0, P2 |
| P2 | m[2], m[3] | P1, P3 |
| P3 | m[3], m[0]* | P2, P0 |
| P4 | m[4], m[1] | P0, P1 |

✅ **Answer:**
- **Mutual Exclusion is NOT guaranteed globally** — since processes share only *adjacent* semaphores (like forks between neighboring philosophers), **non-adjacent processes can be in the CS simultaneously** — e.g., **P0 and P2 can both be inside their critical sections at the same time** (they don't share any semaphore).
- **Max processes simultaneously in CS = 2** (any two non-adjacent processes, e.g., P0 & P2).
- **Deadlock is possible** — this has the exact same structure as the **Dining Philosophers Problem** (each process grabs its "left fork" then "right fork"): if every process grabs its first semaphore at the same time, a **circular wait** forms and the system deadlocks.

> 🧠 This slide is essentially a **disguised Dining Philosophers Problem** — recognize the "grab left, grab right" pattern!

---

### PYQ 9 — Min/Max Output Count Before Blocking

```c
BSEM S = 1, T = 0, Z = 0;

Pri {                    Prj {              Prk {
  while (1) {              P(T);              P(Z);
    P(S);                  V(S);              V(S);
    print('*');
    V(T);
  }
}
```

**Q: What is the minimum and maximum number of `*` printed before the processes complete or block?**

✅ **Answer: Minimum = 2, Maximum = 3**

**Reasoning:** Depending on scheduling order:
- One interleaving (`Pi; Prj; Prk; Pi; Pi`) yields **2 stars** printed before the system stalls/completes.
- Another interleaving (`Pi; Pj; Pi; Pk; Pi; Pi`) yields **3 stars**.

Both orderings are valid given `Prj` and `Prk` each release `S` exactly once (via `V(S)`), and `Pri` can consume each release to print one more `*`.

---

### PYQ 10 — Semaphore Initialization for a Fixed Print Sequence

Three threads `T1, T2, T3` share three **binary semaphores** `S1, S2, S3`.

| T1 | T2 | T3 |
|---|---|---|
| `while(true){` | `while(true){` | `while(true){` |
| `wait(S3);` | `wait(S1);` | `wait(S2);` |
| `print("C");` | `print("B");` | `print("A");` |
| `signal(S2);}` | `signal(S3);}` | `signal(S1);}` |

**Q: Which initialization prints the sequence `BCABCABCA...`?**

**Options:**
- A. S1=1, S2=1, S3=1
- B. S1=1, S2=1, S3=0
- **C. S1=1, S2=0, S3=0**
- D. S1=0, S2=1, S3=1

✅ **Answer: C**

**Reasoning:** The sequence must **start with B** → T2 must run first → `S1` must start at **1** (so `wait(S1)` doesn't block). After printing "B", T2 signals `S3` → enables T1 to print "C" → so `S3` must start at **0** (T1 should NOT be able to run before T2). T1 then signals `S2` → enables T3 to print "A" → so `S2` must start at **0** too. This gives `S1=1, S2=0, S3=0`.

---

### PYQ 11 — Counting Semaphore Allowing 2 Concurrent Accesses (Race Condition)

> A Counting Semaphore initialized to **2**. Four concurrent processes `Pi, Pj, Pk, PL`. `Pi, Pj` want to **increment** shared variable `C` by 1; `Pk, PL` want to **decrement** `C` by 1. All updates happen **under semaphore control**, but the semaphore permits **2 processes concurrently** (not full mutual exclusion). Initial `C = 0`.

**Q: What can be the minimum and maximum value of `C` after all processes finish?**

✅ **Answer: Maximum = 2, Minimum = −2**

**Reasoning:** Since the counting semaphore has value **2**, it allows **two processes to be inside the update region simultaneously**, so a classic **read–modify–write race condition** is possible on the non-atomic `C = C ± 1` operation. This means increments/decrements can be **"lost"** or effectively **double-counted** depending on interleaving, pushing the final value away from the "expected" 0. Working through all worst-case interleavings gives a **range of [−2, +2]** for the final value of `C`.

---

### PYQ 12 — Deadlock-Free Ordering with 4 Binary Semaphores

Three processes `X, Y, Z` share four binary semaphores `a = b = c = d = 1`.

| Option | X | Y | Z | Deadlock-Free? |
|---|---|---|---|---|
| I | P(a); P(b); P(c); | P(b); P(c); P(d); | P(c); P(d); P(a); | ❌ Cycle: X→Y→Z→X |
| **II** | **P(b); P(a); P(c);** | **P(b); P(c); P(d);** | **P(a); P(c); P(d);** | ✅ **No cycle — Deadlock-Free** |
| III | P(b); P(a); P(c); | P(c); P(b); P(d); | P(a); P(c); P(d); | ❌ Cycle exists |
| IV | P(a); P(b); P(c); | P(c); P(b); P(d); | P(c); P(d); P(a); | ❌ Cycle exists |

✅ **Answer: Sequence II**

**Reasoning:** Draw a **resource-request graph** for each option using the order semaphores are acquired. Options I, III, IV all contain a **circular wait** (a cycle X→Y→Z→X or similar) among the semaphore acquisitions, making deadlock **possible**. Only **Option II** has a consistent acquisition order with **no cycle**, guaranteeing deadlock-freedom.

> 🧠 **Mnemonic:** *Deadlock-free multi-lock code = always acquire locks in a globally consistent order (draw the wait-for graph — if it has a cycle, it can deadlock).*

---

### PYQ 13 — Barrier Synchronization (Classic GATE Pattern) [Self-Attempt / HW]

```c
// n processes execute this code
// semaphores a = 1, b = 0 (initial); count = 0 (shared, not used in Section Q)

CODE SECTION P
wait(a); count = count + 1;
if (count == n) signal(b);
signal(a); wait(b); signal(b);
CODE SECTION Q
```

**Q: What does this code achieve?**

**Options:**
- **A. It ensures that no process executes CODE SECTION Q before every process has finished CODE SECTION P.**
- B. It ensures two processes are in CODE SECTION Q at any time.
- C. It ensures all processes execute CODE SECTION P mutually exclusively.
- D. It ensures at most n−1 processes are in CODE SECTION P at any time.

✅ **Answer: A**

**Reasoning:** This is a classic **barrier / rendezvous** construct. `wait(a)`/`signal(a)` only protects the counting operation (`count++`), NOT the whole Section P — so C is wrong. The **last** process to increment `count` to `n` signals `b`, and since every process then does `wait(b); signal(b)` (a relay), **all processes get released together only after the last one arrives**, guaranteeing that **no process enters Section Q until every process has completed Section P.**

---

### PYQ 14 — Double-Wait Counting Semaphore (GATE 2020, MSQ)

```c
1.  int counter = 0
2.  Semaphore S = init(5);
3.  void parop(void)
4.  {
5.      wait(S);
6.      wait(S);
7.      counter++;     // NOT atomic
8.      signal(S);
9.      signal(S);
10. }
```
Five threads execute `parop()` concurrently.

**Q: Which outcomes are possible?**

| Option | Possible? |
|---|---|
| ✅ A. There is a deadlock involving all the threads | **YES** |
| ✅ B. counter = 5 after all threads finish | **YES** |
| ✅ C. counter = 1 after all threads finish | **YES** |
| ❌ D. counter = 0 after all threads finish | **NO** |

**Reasoning:**
- Since each `parop()` call does **two** `wait(S)` calls but `S` starts at 5 (odd), the **5th thread** attempting its 2nd `wait(S)` can find `S` exhausted permanently if the other 4 threads have each done only ONE `wait(S)` — this can lead to **all 5 threads stuck** → **deadlock (A) is possible.**
- If all threads happen to serialize perfectly, `counter` correctly reaches **5 (B possible)**.
- Since `counter++` is **not atomic**, race conditions can cause **lost updates**, so `counter` could end up as low as **1 (C possible)** if all 5 increments overlap and only one survives.
- `counter` can **never be 0** because at least one increment must always be visible (some thread always completes) — **D is impossible.**

---

### PYQ 15 — Binary vs Counting Semaphore Range of Values (GATE 2023) [HW]

```c
Incr() {                    Decr() {
  wait(s);                    wait(s);
  x = x + 1;                  x = x - 1;
  signal(s);                  signal(s);
}                            }
```
- 5 processes invoke `Incr()`, 3 processes invoke `Decr()`. `x` initialized to **10**.
- **I1:** `s` is a **Binary Semaphore**, value 1.
- **I2:** `s` is a **Counting Semaphore**, value 2.
- Let `V1`, `V2` be the **minimum possible values** of `x` for I1 and I2 respectively.

**Options:** A. 12, 7   B. 7, 7   C. 15, 7   D. 12, 8

✅ **Answer: A (12, 7)**

**Reasoning:**
- **I1 (Binary Semaphore, s=1):** enforces **full mutual exclusion** — only one process can execute `wait…signal` at a time, so **no race condition** is possible. Regardless of interleaving, the final value is deterministically `x = 10 + 5(+1) − 3(−1) = 10 + 5 − 3 = 12`. So **V1 = 12** (only possible value → also the minimum).
- **I2 (Counting Semaphore, s=2):** allows **two processes** to execute the critical section concurrently, enabling **lost updates** on the non-atomic `x = x ± 1`. Working through the worst-case race interleavings, the **minimum achievable value of `x` is 7**.

---

### PYQ 16 — Two Threads, Two Semaphores (Deadlock Analysis) [HW]

```c
// s1 = 1, s2 = 0 (initial); x = 0 (shared)

// T1                          // T2
wait(s1);                      wait(s1);
x = x + 1;                     x = x + 1;
print(x);                      print(x);
wait(s2);                      signal(s2);
signal(s1);                    signal(s1);
```

**Q: Which of the following outcomes is/are possible when T1 and T2 execute concurrently?**

**Options:**
- A. T1 runs first and prints 1, T2 runs next and prints 2
- **B. T2 runs first and prints 1, T1 runs next and prints 2**
- **C. T1 runs first and prints 1, T2 does not print anything (deadlock)**
- D. T2 runs first and prints 1, T1 does not print anything (deadlock)

✅ **Answer: B and C** (MSQ)

**Reasoning:**
- **If T1 runs first:** `wait(s1)` succeeds (s1→0), `x=1`, prints `1`, then T1 calls `wait(s2)` — but `s2=0` and nobody has signaled it yet (T2 can't even start since `s1` is held by T1) → **T1 blocks forever, T2 never runs → Deadlock.** This matches **Option C.**
- **If T2 runs first:** `wait(s1)` succeeds, `x=1`, prints `1`, then `signal(s2)` (s2→1), `signal(s1)` (s1→1) — **T2 completes fully.** Now T1 can run: `wait(s1)` succeeds, `x=2`, prints `2`, `wait(s2)` succeeds immediately (s2 was already signaled), `signal(s1)`. **T1 completes too.** This matches **Option B.**
- Options A and D describe the *opposite* pairing of "who blocks", which contradicts the trace above — hence they are **not possible**.

> 🧠 **Key insight:** The **order in which the two threads first acquire `s1` determines everything** — whichever thread runs first here (T1) can get permanently stuck waiting on `s2`, since only T2 (which never gets to run) would signal it.

---

### PYQ 17 — Three Identical Processes, Print Pattern Analysis (Advanced)

```c
// A = 1, B = 0 (binary semaphores); X = 0 (shared, atomic lines)

wait(A);
print(*);
X = X + 1;
if (X == 2) {
    print($);
    signal(B);
}
signal(A);
wait(B);
print(#);
signal(B);
```
Three processes `P1, P2, P3` run this **identical code**; any process may start first; context switches can occur at any arbitrary time.

**Q: Which pattern(s) are possible outcomes?**
- A. `**$*###`
- B. `**$#*##`
- C. `**$##*#`
- D. `***$###`

**Reasoning (derive using resource-request logic):**
- The segment from `wait(A)` to `signal(A)` is a **mutual-exclusion region** — only one process executes `print(*); X++; (maybe print($))` at a time, **as one atomic unit**. So the **`$` must always immediately follow its own process's `*`**, with no other process's `*` able to interleave between them.
- ➡️ This means a pattern like **`***$###`** (three stars before the `$`) is **impossible**, since `$` must appear right after the *second* mutex-entrant's `*`, not after a third one.
- After finishing the mutex region, each process independently does `wait(B); print(#); signal(B);` — this part **can freely interleave** with the *remaining* mutex-region turns of other processes (since it's outside the `A`-protected region). This means a `#` **can appear before the final `*`** of the pattern (if the process that already passed through the mutex gets scheduled to consume the `B` signal before the last process even enters the mutex region).
- ➡️ Patterns like **`**$*###`** (clean, no interleaving) and **`**$#*##`** (one `#` sneaks in early) are both **structurally consistent** with the semaphore logic.

✅ **My derived conclusion:** **A and B are possible; D is impossible.** *(C requires careful re-verification of how many `B`-signals can stack up before the 3rd `*` — please cross-check this specific option against the official GATE answer key, as multiple `#`s appearing consecutively before the final `*` depends on subtle scheduling assumptions.)*

> ⚠️ **Note:** This was presented as a 2-mark MSQ in the lecture without the instructor's final answer key visible on the slide — treat the reasoning above as a **derivation to practice with**, and verify against the official GATE key for full certainty.

---

## 12. Memory Tricks / Mnemonics

- 🧠 **Mutex = Lock, Semaphore = Signal.** *(One "holds a key", the other "rings a bell".)*
- 🧠 **BSEM = Binary Semaphore** — shorthand used throughout for a semaphore that only takes values 0/1.
- 🧠 **Monitor = "mini-OS"** — its entry procedures are like **system calls**: only one call can be "in the kernel" (monitor) at a time.
- 🧠 **Flag-only solutions (no `turn` variable) → Mutual Exclusion ✅, Deadlock-freedom ❌.** (Seen in PYQ 1 & PYQ 2 — recognize this pattern instantly on sight!)
- 🧠 **Opposite lock-acquisition order (`P(S);P(T)` vs `P(T);P(S)`) → potential deadly embrace.** Always check ordering first when analyzing 2-semaphore code.
- 🧠 **"Grab left, grab right" pattern on an array of adjacent semaphores → it's a disguised Dining Philosophers Problem** (PYQ 8) — look for `m[i]` and `m[(i+1)%n]` patterns.
- 🧠 **Non-atomic `counter++` under a semaphore that allows >1 concurrent entry → race condition/lost updates are possible** — always check if the semaphore *value* is exactly 1 (true mutex) or greater (partial concurrency allowed).

---

## 13. Quick Revision Checklist ✅

Test yourself — can you explain each of these without looking back?

- [ ] Why do OS designers introduce **mutex locks**, and why are they called **spinlocks**?
- [ ] Full **Mutex vs Semaphore** comparison (mechanism, data type, types, deadlock risk).
- [ ] Three classic **semaphore misuse patterns** (reversed order, double wait, omission).
- [ ] Definition of **Liveness Property** and what counts as a **Liveness Failure**.
- [ ] Deadlock example using two semaphores `S`, `Q` acquired in reverse order.
- [ ] Difference between **Deadlock**, **Starvation**, and **Priority Inversion** (+ its fix: Priority-Inheritance Protocol).
- [ ] What a **Monitor** is; why it has **no public data**; the "mini-OS" analogy.
- [ ] Role of **condition variables** (`wait()`/`signal()`) inside a monitor.
- [ ] Full **Producer-Consumer using Monitors** code (enter/remove procedures).
- [ ] Monitor vs Semaphore — key distinguishing point (encapsulation).
- [ ] One-line description of each classical IPC problem: **Producer-Consumer, Readers-Writers, Dining Philosophers, Sleeping Barber.**
- [ ] Peterson/Dekker-style flag-only code tracing — can you spot the deadlock risk instantly?
- [ ] Test-and-Set busy-wait lock: is it deadlock-free? starvation-free? FIFO? (GATE 2014 pattern)
- [ ] Fetch-And-Add lock bug: why can `L` become non-zero incorrectly?
- [ ] Counting semaphore blocking arithmetic (P vs V operation counts).
- [ ] Semaphore-based thread print-sequencing (relay pattern using multiple binary semaphores).
- [ ] Min/Max value analysis for shared counters updated under weak (counting > 1) semaphore control.
- [ ] Deadlock-free lock-acquisition ordering across multiple processes (resource-request-graph / cycle check).
- [ ] Barrier synchronization pattern (`count == n` trick + `wait/signal` relay).

---

*Notes compiled from GeeksforGeeks GATE CS & IT Engineering — Principles of Operating Systems, Lecture 17: Process Synchronization (III).*
