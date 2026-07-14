# Process Synchronization — Lecture 16: Semaphores
*GeeksforGeeks GATE CS&IT — Principles of Operating Systems*

---

## 📑 Topics Covered

| # | Topic |
|---|-------|
| 1 | What is Process Synchronization (Process Coordination) |
| 2 | Semaphores — definition, ADT, types |
| 3 | Busy-Waiting implementation of Semaphores (`wait`/`signal`) |
| 4 | Blocking (Queue-based) implementation of Semaphores |
| 5 | Counting Semaphore vs Binary Semaphore (BSEM = Mutex) |
| 6 | Solving CS problem, Producer→Consumer ordering with Semaphores |
| 7 | Client–Server example using Counting Semaphore |
| 8 | Strong vs Weak Semaphores |
| 9 | Mutex Locks (spinlocks) |
| 10 | Mutex vs Semaphore |
| 11 | Problems with incorrect Semaphore usage |
| 12 | Liveness, Deadlock, Starvation, Priority Inversion |
| 13 | Quick-check MCQs + GATE-style PYQs (Semaphore code tracing) |

> 💡 **Mnemonic:** Semaphore = a **railway/naval flag signal** 🚩 — it doesn't physically stop the train, it just *signals* whether it's safe to proceed. That's why Semaphore is called a **signaling mechanism**, unlike Mutex which is a **locking mechanism** (a literal lock/key).

---

## 1. What is Process Synchronization?

**Process Synchronization** (a.k.a. **Process Coordination**) is the mechanism that ensures concurrent processes/threads execute in a controlled, predictable order when they share resources — preventing race conditions.

- Achieved using **Synchronization Mechanisms (Tools)** such as: Mutex Locks, Semaphores, Monitors.
- This lecture focuses on **Semaphores**.

---

## 2. Semaphore — Core Definition

> **Historical note:** Semaphores were introduced by **Edsger Wybe Dijkstra** (1930–2002), Dutch computer scientist, 1972 Turing Award winner (structured programming). The ACM PODC Influential Paper Award was renamed the **Dijkstra Prize** in his honor.

**Definition:** A Semaphore is a synchronization tool implemented in the **OS kernel as an Abstract Data Type (ADT)**. It is a variable that takes only **integral values**, and — being an ADT — supports exactly **two atomic operations**:

| Operation | Aliases | Effect |
|---|---|---|
| **Down** | Wait / **P()** | Decrements the semaphore value |
| **Up** | Signal / **V()** | Increments the semaphore value |

- Semaphore operations are **atomic** (indivisible) — no other process can be interleaved mid-operation.
- Semaphore ops are typically invoked like a **system call** (kernel-mediated), since the ADT lives in kernel space.
- A semaphore variable is **always associated with a value** — this value represents the **state of the semaphore**.
- Semaphore provides **more sophisticated synchronization than Mutex locks**; it is a **general-purpose utility** compared to other synchronization mechanisms.

### Types of Semaphore (based on range of values)

| Type | Range | Also known as |
|---|---|---|
| **Binary Semaphore (BSEM)** | `{0, 1}` | Mutex |
| **Counting Semaphore (CSEM)** | `(-∞, +∞)` — unrestricted domain | General Semaphore |

- A **Counting Semaphore can be implemented using a Binary Semaphore**.
- With semaphores, we can solve a wide variety of process-synchronization problems (mutual exclusion, ordering, producer-consumer, etc.).

---

## 3. Semaphore Implementation #1 — Busy-Waiting (Spinlock-style)

```c
// Declaration (user mode)
SEM S;
S = 1;

wait(S){
    while (S <= 0);   // busy wait — spin here
    S--;
}

signal(S){
    S++;
}
```

- `wait()`/`signal()` were **originally named `P()` and `V()`**.
- **Must be atomic (indivisible)** — otherwise race conditions on `S` itself occur.
- The `while(S<=0);` loop is called **busy waiting** — CPU cycles wasted spinning instead of sleeping.

### Example 1 — Mutual Exclusion (Critical Section) using Binary Semaphore

```c
semaphore mutex = 1;   // BSEM
wait(mutex);
    <Critical Section>
signal(mutex);
```

### Example 2 — Ordering two statements (S1 must happen before S2) using Counting Semaphore

Given processes **P1** and **P2**, and statements S1 (in P1) must execute before S2 (in P2):

```c
semaphore synch = 0;   // initialized to 0

P1:                     P2:
    S1;                     wait(synch);
    signal(synch);          S2;
```

Since `synch` starts at 0, P2 is **forced to block** at `wait(synch)` until P1 executes `signal(synch)` — guaranteeing S1 → S2 ordering.

---

## 4. Semaphore Implementation #2 — Blocking (Queue-based, no busy-waiting)

Instead of spinning, a blocked process is **put to sleep** and placed in a waiting queue associated with the semaphore.

```c
// Semaphore ADT structure
typedef struct {
    int value;
    struct process *list;   // linked list of PCBs blocked on this semaphore
} semaphore;
```

Two supporting kernel primitives:
- **`block()`** — suspends the calling process, places it on the semaphore's waiting queue.
- **`wakeup(P)`** — removes process P from a waiting queue and moves it to the ready queue.

```c
wait(semaphore *s){
    s->value--;
    if (s->value < 0) {
        add this process to s->list;   // add PCB to waiting queue
        block();
    }
}

signal(semaphore *s){
    s->value++;
    if (s->value <= 0) {
        remove a process P from s->list;
        wakeup(P);
    }
}
```

> 💡 **Key insight:** In the blocking implementation, a **negative** semaphore value = the absolute number of processes currently blocked/waiting on it.

### Worked Example — Client–Server Problem (Counting Semaphore as a resource-limiter)

A server has a DB resource pool that can serve up to **900** concurrent client requests. Clients `C1, C2, C3, …` connect to the server.

```c
CSEM S;
S = 900;

P(S);
    <Access DB>
V(S);
```

Here the counting semaphore **limits concurrency** to 900 simultaneous DB accesses — a classic **resource-pool / connection-limiting** use case for counting semaphores.

---

## 5. Binary Semaphore (BSEM) — Detailed Implementation

```c
typedef struct {
    enum Value(0,1);     // only 0 or 1
    QueueType L;          // waiting queue
} BSEM;

BSEM S;
S = 1;
Down(S);
    <Critical Section>
Up(S);
```

```c
Down(BSEM *S) {
    if (S->value == 1) {
        S->value = 0;
        return;                 // success — enter CS
    } else {
        put this process (PCB) in S->L (queue);
        Block();                // go to sleep
    }
}

Up(BSEM *S) {
    if (S->L is not empty) {
        select a process P and remove it;
        wakeup(P);               // hand CS directly to P (no value change!)
    } else {
        S->value = 1;            // no one waiting, just free the lock
    }
}
```

> 💡 **Note the subtlety:** In `Up()`, if the queue is **not empty**, the value is *not* set to 1 — control (and implicit ownership) is passed **directly** to the woken-up process. Value only becomes 1 when nobody is waiting.

### Trace: 5 Cases of Binary Semaphore Operations

| Case | Initial S | Operation | New S | Status |
|---|---|---|---|---|
| 1 | S = 1 | P(S) | S = 0 | **Success** |
| 2 | S = 0 | P(S) | S = 0 | **Unsuccessful** (blocked) |
| 3 | S = 1 | V(S) | S = 1 | **Success** (no-op, already free) |
| 4 | S = 0 | V(S) | Q not empty → wakes a process (value stays effectively "handed over"); Q empty → S=1 | **Success** |
| 5 | S = 0 (initial) | V(S) — first operation | S = 1 | Initializes lock to free state |

---

## 6. Counting Semaphore — Numerical Trace Example

**Given:** `S = 10` (initial value). Operations issued (in order): `15×P, 3×V, 8×P, 5×V, 10×P, 2×V`

| Step | Operation | Running Value |
|---|---|---|
| 15 × P(S) | −15 | 10 − 15 = **−5** |
| 3 × V(S) | +3 | −5 + 3 = **−2** |
| 8 × P(S) | −8 | −2 − 8 = **−10** |
| 5 × V(S) | +5 | −10 + 5 = **−5** |
| 10 × P(S) | −10 | −5 − 10 = **−15** |
| 2 × V(S) | +2 | −15 + 2 = **−13** |

**Final value: `S = −13`**

> 💡 Formula: For a counting semaphore, **final value = initial value − (total P ops) + (total V ops)**, regardless of interleaving order (only the *final* value is order-independent; whether a *particular* P blocks *along the way* does depend on order).

$$S_{final} = S_{initial} - N_{P} + N_{V}$$

Where:
- $S_{initial}$ = starting value of semaphore
- $N_P$ = total number of `P()`/`wait()` operations issued
- $N_V$ = total number of `V()`/`signal()` operations issued

---

## 7. Strong vs Weak Semaphores

A **queue** holds processes waiting on the semaphore. How that queue releases processes distinguishes two semaphore variants:

| | Strong Semaphore | Weak Semaphore |
|---|---|---|
| **Release order** | Process blocked **longest** is released first (**FIFO**) | **No guaranteed order** |
| **Starvation** | **Freedom from starvation guaranteed** | Starvation **possible** |
| **Queue discipline** | Maintains strict FIFO queue | No guaranteed queue discipline |

---

## 8. Mutex Locks

- OS designers provide simple software tools to solve the CS problem for application programmers (since raw semaphore-blocking implementations are complex/kernel-only).
- **Simplest tool = Mutex Lock**: a **Boolean variable** indicating lock availability.
- Protect a Critical Section using:
  1. `acquire()` the lock
  2. `release()` the lock
- `acquire()`/`release()` **must be atomic** — usually implemented via hardware atomic instructions like **compare-and-swap (CAS)**.
- Because acquiring a busy mutex requires the caller to **loop/spin**, this solution requires **busy waiting** → hence a mutex implemented this way is called a **spinlock**.

```c
while (True) {
    acquire lock;
    critical section;
    release lock;
    remainder section;
}
```

### Mutex vs Semaphore

| Aspect | Mutex | Semaphore |
|---|---|---|
| **Mechanism type** | **Locking** mechanism | **Signaling** mechanism |
| **Nature** | An **object** | An **integer variable** |
| **Types** | No types (single kind) | Two types: **Binary** & **Counting** |
| **Deadlock risk** | **Higher** risk of deadlock | **Lower** risk of deadlock |

---

## 9. Problems with Incorrect Semaphore Usage

Improper ordering/omission of semaphore operations breaks synchronization guarantees:

- `signal(mutex) ... wait(mutex)` — **wrong order** (signaling before acquiring)
- `wait(mutex) ... signal(mutex)` — omitted `signal` elsewhere can cause deadlock
- **Omitting `wait(mutex)` and/or `signal(mutex)`** entirely defeats mutual exclusion

> 💡 **Rule of thumb:** Every `wait()` must be matched by exactly one `signal()`, in the correct program order, on every code path (including error/exception paths).

---

## 10. Liveness Properties (Progress Failures)

**Liveness Property:** the set of properties a system must satisfy to ensure that processes **make forward progress** (as opposed to just being "safe" via mutual exclusion). Processes waiting **indefinitely** for a sync tool violates the **Progress** and **Bounded-Waiting** criteria.

**Indefinite waiting = Liveness Failure.** There are 3 classic types:

### (a) Deadlock
Two or more processes wait **infinitely** for an event that can only be caused by one of the very processes that are waiting (**circular wait**).

**Classic Example:** Semaphores `S` and `Q`, both initialized to 1.

```c
P0:                    P1:
wait(S);               wait(Q);
wait(Q);               wait(S);
...                     ...
signal(S);             signal(Q);
signal(Q);             signal(S);
```

If `P0` executes `wait(S)` and `P1` executes `wait(Q)` (interleaved), then:
- P0's `wait(Q)` blocks until P1 does `signal(Q)`.
- But P1 is blocked at `wait(S)`, waiting until P0 does `signal(S)`.
- Neither `signal()` will ever execute → **P0 and P1 are deadlocked**.

> 💡 **Mnemonic:** This is the textbook **"lock ordering violation"** — P0 locks S→Q, P1 locks Q→S (reverse order) → classic circular-wait deadlock. **Always acquire multiple locks in the same global order to avoid this.**

### (b) Starvation
**Indefinite blocking** — a process may never be removed from the semaphore's waiting queue (even though the system as a whole keeps making progress). Common with **Weak Semaphores**.

### (c) Priority Inversion
A **scheduling problem**: a **lower-priority** process holds a lock needed by a **higher-priority** process, so the higher-priority process is effectively "blocked" by a lower one.
- **Solved via:** **Priority-Inheritance Protocol** (temporarily boosts the priority of the lock-holder to that of the highest-priority waiter).

---

## 🧠 Quick Revision Checklist

- [ ] Semaphore = kernel ADT, integer value, atomic `wait()`/`signal()` (a.k.a `P()`/`V()`)
- [ ] Two types: **Binary (0/1, = Mutex)** and **Counting (unrestricted range)**
- [ ] Busy-waiting implementation: `while(S<=0);` then `S--` — wastes CPU (spinlock)
- [ ] Blocking implementation: negative value = number of processes waiting; uses `block()`/`wakeup()`
- [ ] Formula: $S_{final} = S_{initial} - N_P + N_V$
- [ ] **Strong semaphore** = FIFO release, starvation-free; **Weak semaphore** = no order guarantee
- [ ] **Mutex Lock** = Boolean lock, `acquire()`/`release()`, atomic via CAS, busy-wait ⇒ spinlock
- [ ] Mutex = locking mechanism/object; Semaphore = signaling mechanism/integer, has types
- [ ] Incorrect semaphore usage (wrong order/omission) → violates mutual exclusion or causes deadlock
- [ ] Liveness failures: **Deadlock** (circular wait), **Starvation** (indefinite blocking), **Priority Inversion** (solved via Priority-Inheritance Protocol)
- [ ] Can trace semaphore value given a sequence of P/V operations
- [ ] Can trace binary-semaphore-protected concurrent code for **Mutual Exclusion** and **Deadlock Freedom**
- [ ] Can reason about **lock-ordering deadlocks** (acquire locks in consistent global order to avoid circular wait)

---

## 📝 Practice: Quick-Check MCQs (from lecture)

**Q1. Why is disabling interrupts not a recommended synchronization technique on Multiprocessor systems?**
- **✅ A. It can't efficiently disable interrupts on all processors.**
- B. It increases context switching. — *Irrelevant; the real issue is scope, not scheduling overhead.*
- C. It violates mutual exclusion. — *Disabling interrupts on a single CPU does enforce ME on that CPU; the problem is it doesn't cover other CPUs.*
- D. It causes immediate system crashes. — *False; it just fails to protect against other CPUs.*

**Q2. Which synchronization technique may lead to Busy-Waiting?**
- A. Binary Semaphore (blocking-mode) — *not necessarily busy-wait if implemented with a queue.*
- **✅ B. Mutex using spinlock** — *by definition, a spinlock loops (spins) while waiting.*
- C. Bakery Algorithm — *waits via checking tickets, but classically also busy-waits; however, the lecture marks B as the intended answer here since spinlocks are explicitly busy-wait by design.*
- D. Peterson Solution — *also busy-waits in strict implementations, but lecture's marked answer is B.*

> ⚠️ Note: technically Peterson's Solution and the Bakery Algorithm *also* busy-wait in their classic form — but the lecture's "Quick Check" specifically marks **Option B (Mutex using spinlock)** as correct, likely emphasizing that spinlocks are the canonical, guaranteed busy-wait mechanism among the options.

---

## 📝 GATE-Style MCQs (Concept Review)

Below are the knowledge-check questions from the lecture, along with best-reasoned correct answers and rationale.

**1. Which Algorithm solves the Critical Section problem for Two Processes?**
A) Peterson's Algorithm ✅ B) Dijkstra's Banker's Algorithm C) Dekker's Algorithm D) FIFO Scheduling
> **Answer: A.** Peterson's Algorithm is the standard two-process software CS solution taught in this context. (Note: Dekker's Algorithm historically *also* solves 2-process CS — it predates Peterson's — but Peterson's is the canonical answer expected here.) Banker's Algorithm is for **deadlock avoidance**, not CS; FIFO Scheduling is a **CPU scheduling** policy, unrelated to CS.

**2. Which of the following is a Software Synchronization Mechanism in User Mode?**
A) Test-and-Set instruction B) Semaphore C) Compare-and-Swap instruction D) Strict Alternation ✅
> **Answer: D.** Strict Alternation is a pure software (variable-based) technique. TAS/CAS are **hardware** instructions; Semaphore is typically a **kernel-level ADT**, not purely user-mode software.

**3. What is the primary purpose of Hardware synchronization primitives like Test-and-Set and Compare-and-Swap?**
A) Increase CPU speed B) Prevent context switching C) Ensure Mutual Exclusion in Critical Section ✅ D) Schedule processes in round-robin fashion
> **Answer: C.** These atomic instructions are used to build locks that guarantee ME. Options A, B, D are unrelated goals.

**4. Which of the following Synchronization Techniques may lead to Busy-Waiting?**
A) Binary Semaphore B) Mutex using spinlock ✅ C) Bakery Algorithm D) Peterson Solution
> **Answer: B** (as marked in lecture). Spinlocks explicitly loop/spin — the defining trait of busy-waiting.

**5. Why is disabling interrupts not a recommended synchronization technique on Multiprocessor systems?**
A) It can't disable interrupts on all processors ✅ B) It increases memory usage C) It reduces cache performance D) It increases thread priority
> **Answer: A.** Disabling interrupts is a per-CPU action; on other CPUs, interrupts (and hence concurrent execution) continue, breaking mutual exclusion.

**6. Which of the following is true about hardware vs software synchronization mechanisms?**
A) Software mechanisms are always faster than hardware B) Hardware primitives are not useful for synchronization C) Hardware primitives are typically used to build software synchronization tools ✅ D) Software synchronization cannot work without hardware support
> **Answer: C.** Mutex locks, semaphores, etc., are software abstractions commonly *built on top of* hardware atomic instructions like TAS/CAS.

**7. Which of the following is true about a Semaphore?**
A) It is a busy-waiting synchronization mechanism B) It uses signal and wait operations ✅ C) It can only be used for single-threaded applications D) It is always binary in nature
> **Answer: B.** Semaphores are defined by their `wait()`/`signal()` (P/V) operations; they can use blocking (non-busy-wait) implementations; they support multi-threaded/multi-process use; they can be binary *or* counting.

**8. What distinguishes a Monitor from a Semaphore?**
A) Semaphores are object-oriented, monitors are not B) Monitors use explicit wait and signal functions C) Monitors encapsulate shared variables, semaphores do not ✅ D) Monitors cause more deadlocks than semaphores
> **Answer: C.** Monitors are a higher-level, structured construct that **bundles shared data + procedures + implicit mutual exclusion** together (object-oriented in spirit), unlike raw semaphores which are just standalone integer variables.

**9. Which of the following operations causes a Semaphore to decrement?**
A) signal() B) raise() C) wait() ✅ D) notify()
> **Answer: C.** `wait()`/`P()` decrements; `signal()`/`V()` increments.

**10. In monitors, what happens if a thread calls wait()?**
A) It is terminated immediately B) It continues execution C) It is placed in the monitor's wait queue ✅ D) It signals another thread to enter the monitor
> **Answer: C.** `wait()` inside a monitor suspends the calling thread and places it in a condition-variable wait queue, releasing the monitor lock.

**11. Which of the following problems can Semaphores help solve?**
A) CPU scheduling B) Memory fragmentation C) Race conditions in critical sections ✅ D) Page replacement
> **Answer: C.** Semaphores are a process-synchronization tool, directly targeting race conditions; the other options are unrelated OS subsystems.

**12. Which classical IPC problem is mainly concerned with deadlock and starvation among multiple processes sharing limited resources like forks?**
A) Producer-Consumer Problem B) Readers-Writers Problem C) Dining Philosophers Problem ✅ D) Sleeping Barber Problem
> **Answer: C.** The Dining Philosophers problem explicitly models resource contention (forks) with deadlock/starvation risks.

**13. In the Producer-Consumer problem, which data structure is most commonly used for managing the buffer?**
A) Stack B) Queue ✅ C) Tree D) Hash Table
> **Answer: B.** A bounded/circular **Queue** naturally models FIFO producer→consumer item flow.

**14. Which synchronization tool is best suited to resolve the Readers-Writers problem while prioritizing readers over writers?**
A) Mutex B) Counting Semaphore C) Read-Write Lock ✅ D) Binary Semaphore
> **Answer: C.** A Read-Write Lock is purpose-built for reader/writer prioritization policies (though the classical textbook solution is often implemented *using* semaphores/mutex internally).

**15. What is the major issue addressed in the Sleeping Barber problem?**
A) Resource starvation B) Deadlock avoidance C) Process scheduling in multi-core systems D) Synchronization between a single server (barber) and multiple customers ✅
> **Answer: D.** The Sleeping Barber problem models synchronizing one server resource against multiple waiting clients (an analogy for bounded-buffer producer-consumer-like coordination).

**16. In the Dining Philosophers problem, which approach helps to prevent deadlock?**
A) Philosophers pick both forks simultaneously B) Limiting the number of philosophers allowed to sit at the table ✅ C) Allowing philosophers to eat without thinking D) Giving each philosopher only one fork
> **Answer: B.** Limiting concurrent diners to (N−1) ensures at least one philosopher can always get both forks (pigeonhole principle) — a standard deadlock-prevention technique.

**17. Which of the following is NOT typically associated with concurrent programming?**
A) Race conditions B) Deadlocks C) Starvation D) Compilation errors ✅
> **Answer: D.** Compilation errors are a static/syntactic issue, unrelated to runtime concurrency behavior.

**18. Which synchronization mechanism ensures mutual exclusion?**
A) Semaphore ✅ B) Compiler C) Garbage Collector D) Stack
> **Answer: A.** Semaphores (specifically binary/mutex semaphores) are designed to enforce mutual exclusion.

**19. In concurrent programming, what is a deadlock?**
A) A bug in the syntax of the code B) A condition where threads wait indefinitely for resources ✅ C) A memory overflow situation D) A compilation failure
> **Answer: B.** By definition, deadlock = indefinite circular waiting for resources.

**20. Which of the following is an example of non-preemptive scheduling in concurrency?**
A) Round Robin B) First Come First Serve ✅ C) Shortest Remaining Time First D) Multilevel Queue
> **Answer: B.** FCFS runs a process to completion once started — the textbook non-preemptive example. Round Robin and SRTF are preemptive; Multilevel Queue can mix both depending on configuration.

---

## 📝 GATE PYQs — Semaphore/Lock Code Tracing

### Q1. Strict-Alternation-style flag code (P1/P2)

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

A) It does not ensure Mutual Exclusion.
B) It does not ensure Bounded Waiting.
C) It requires that processes enter the critical section in Strict Alternation.
D) **It does not prevent Deadlocks, but ensures Mutual Exclusion.** ✅

> **Reasoning:** If both processes set their own flag (`wants1=TRUE`, `wants2=TRUE`) *before* either checks the other's flag, **both will loop forever** checking each other's flag → **deadlock (livelock)**. However, Mutual Exclusion *is* still preserved: a process can only enter CS if the *other's* flag was false at the check — so both can never be inside CS simultaneously. Hence **D** is correct; A is wrong (ME *is* guaranteed); B is a weaker/different claim not the core issue; C is false (no forced strict alternation is required by this code, unlike the classic "turn"-variable Alternation approach).

### Q2. Same pattern, renamed variables (`varP`/`varQ`, Process X / Process Y)

```c
Process X                          Process Y
/* other code for X */             /* other code for Y */
while (true) {                     while (true) {
   varP = true;                       varQ = true;
   while (varQ == true) {              while (varP == true) {
      /* critical section */             /* critical section */
      varP = false;                      varQ = false;
   }                                   }
}                                   }
/* other code for X */             /* other code for Y */
```

**Question:** varP and varQ are shared variables, both initialized to false. Which statement is true?

A) The proposed solution prevents deadlock but fails to guarantee Mutual Exclusion.
B) **The proposed solution guarantees Mutual Exclusion but fails to prevent Deadlock.** ✅
C) The proposed solution guarantees Mutual Exclusion and prevents Deadlock.
D) The proposed solution fails to prevent Deadlock and fails to guarantee Mutual Exclusion.

> **Reasoning:** Identical logic to Q1 above (this is literally the same code pattern under different names) — ME holds, but deadlock is possible if both processes set their flags concurrently before checking. **Answer: B**.

### Q3. `enter_CS`/`leave_CS` using Test-and-Set

```c
void enter_CS(X) {
    while (test-and-set(X));
}

void leave_CS(X) {
    X = 0;
}
```
X is a memory location associated with the CS, initialized to 0.

**Statements:**
I. The above solution to the CS problem is deadlock-free.
II. The solution is starvation-free.
III. The processes enter CS in FIFO order.
IV. More than one process can enter CS at the same time.

**Which of the above statements are TRUE?**
A) I only ✅  B) I and II  C) II and III  D) IV only

> **Reasoning:**
> - **I – TRUE:** Since `test-and-set` atomically sets X and returns the old value, whenever X=0, *some* process will succeed in acquiring it — the system as a whole always makes progress (no permanent circular wait) → **deadlock-free**.
> - **II – FALSE:** There's no fairness guarantee — a particular process could be repeatedly overtaken by others indefinitely → **not starvation-free**.
> - **III – FALSE:** No ordering is enforced; any waiting process can grab the lock next — **not FIFO**.
> - **IV – FALSE:** Test-and-set guarantees only one process can successfully set X=1 at a time → mutual exclusion holds, so **not** more than one process in CS.
>
> **Answer: A (I only)**.

### Q4. Fetch-and-Add busy-wait lock

`Fetch_And_Add(X, i)`: atomically reads X, increments it by `i`, and **returns the OLD value** of X. `L` is an unsigned integer shared variable, initialized to 0 (0 = lock available, non-zero = lock unavailable).

```c
AcquireLock(L) {
    while (Fetch_And_Add(L, 1))
        L = 1;
}

ReleaseLock(L) {
    L = 0;
}
```

A) Fails as L can overflow.
B) **Fails as L can take on a non-zero value when the lock is actually available.** ✅
C) Works correctly but may starve some processes.
D) Works correctly without starvation.

> **Reasoning:** `Fetch_And_Add(L,1)` **always** increments L by 1 on *every* call — regardless of whether the lock was free or not — because it's called unconditionally inside the `while` condition every loop iteration. This means L keeps incrementing due to repeated failed attempts, and even after a successful release (`L=0`), stray/leftover increments from other threads can leave L in an incorrect non-zero state when the lock should logically be free. **Answer: B.**

### Q5. Counting Semaphore — largest initial value causing at least one block

**Given:** Counting Semaphore `S`. `P(S)` decrements S, `V(S)` increments S. During execution, **20 `P(S)`** and **12 `V(S)`** operations are issued (in *some* order). Find the **largest initial value of S** for which **at least one `P(S)` will remain blocked**.

**Solution:**
- For a `P(S)` to block, the semaphore must go **negative** at some point during execution.
- We want the *largest* starting value `x` for which it's still possible to force the value down to at least −1.
- Worst case (all P's issued before any V's): value drops by 20 then rises by 12 → final = $x - 20 + 12$
- To guarantee blocking, we need this to reach **at least −1**:

$$x - 20 + 12 = -1 \implies x = 7$$

**Answer: `x = 7`**

### Q6. Two Binary Semaphores, reverse lock-order (S, T)

```c
BSEM S = 1, T = 1;

Pri {                          Prj {
  while (1) {                    while (1) {
     P(S);                          P(T);
     P(T);                          P(S);
       <CS>                          <CS>
     V(T);                          V(S);
     V(S);                          V(T);
  }                               }
}                               }
```

**Analyze for Mutual Exclusion and Deadlock Freedom.**

> **Answer:**
> - **Mutual Exclusion: Guaranteed.** The CS is protected by *both* S and T being held, and both are binary semaphores — only one process can hold each at any given moment, so only one process can ever have both simultaneously.
> - **Deadlock Freedom: NOT guaranteed.** `Pri` acquires in order **S → T**, while `Prj` acquires in order **T → S** (reverse order). If `Pri` grabs S and `Prj` grabs T "simultaneously," each then waits for the resource held by the other → **classic circular-wait deadlock**.
> - 💡 **Fix:** Always acquire locks in the **same global order** in every process to avoid this exact bug.

### Q7. Two Binary Semaphores as a "ping-pong" token (S=1, T=0)

```c
BSEM S = 1, T = 0;

Pri {                          Prj {
  while (1) {                    while (1) {
     P(T);                          P(S);
     Print('1');                    Print('0');
     Print('1');                    Print('0');
     V(S);                          V(T);
  }                               }
}                               }
```

**What is the output generated?**

> **Trace:** Initially T=0 → `Pri` blocks at `P(T)`. S=1 → `Prj` succeeds: prints `0`,`0`, then `V(T)` (T→1). Now `Pri` unblocks: prints `1`,`1`, then `V(S)` (S→1) → `Prj` can run again.
>
> **Output: `0 0 1 1 0 0 1 1 0 0 1 1 …`** (repeats forever as `"0011"`).

### Q8. Circular Semaphore Array (m[0..4] with mod-4 neighbor locking)

```c
BSEM m[0....4] = [1];   // 4 semaphores, all init to 1
Pri, i = 0, 4 {          // 5 processes, i = 0,1,2,3,4
   while (1) {
      P(m[i]);
      P(m[(i+1) % 4]);
        <CS>
      V(m[i]);
      V(m[(i+1) % 4]);
   }
}
```

**Does the above code guarantee Mutual Exclusion? If no, what is Max Processes in CS? What about Deadlock Freedom?**

> **Reasoning (Dining-Philosophers-style analysis):**
> - This is structurally identical to the **Dining Philosophers** problem: 5 processes but only **4** shared semaphores arranged in a cycle (via `%4`).
> - **Mutual Exclusion between *adjacent* processes:** guaranteed for any pair sharing a common semaphore index (they can't both hold that shared lock simultaneously).
> - **Max processes in CS simultaneously:** Since acquiring locks is pairwise on a cycle of size 4, at most **2 non-adjacent processes** can be in CS at once (each grabs 2 of the 4 available semaphores, and 4 semaphores / 2 per process = 2 processes max).
> - **Deadlock:** Because there are **5 processes but only 4 semaphores**, by the **pigeonhole principle**, it's impossible for all 5 to simultaneously hold their *first* required semaphore (since there are only 4 distinct first-semaphores available) — so **at least one process is always guaranteed to make progress**, meaning the system is **deadlock-free** (same logic as "N−1 philosophers allowed to sit" deadlock-prevention trick, but here it emerges naturally from the count mismatch).

### Q9. Three processes, chained star-printer (S=1, T=0, Z=0)

```c
BSEM S = 1, T = 0, Z = 0;

Pri {                    Prj {              Prk {
  while (1) {              P(T);              P(Z);
     P(S);                 V(S);              V(S);
     Print(*);           }                  }
     V(T);
     V(Z);
  }
}
```

**What is the minimum and maximum number of stars ('*') output before the process gets completed or blocked?**

> **Trace:**
> 1. `Pri` runs: `P(S)` succeeds (S:1→0), prints `*` **(star #1)**, `V(T)` (T→1), `V(Z)` (Z→1).
> 2. `Pri` loops back, tries `P(S)` again — but S=0, so `Pri` **blocks**.
> 3. `Prj` (was blocked on `P(T)` since T was 0) now succeeds (T:1→0), does `V(S)` (S→1). `Prj` **terminates** (it's not in a loop).
> 4. `Prk` (was blocked on `P(Z)`) now succeeds (Z:1→0), does `V(S)`. Since S is *binary*, this `V(S)` has no further effect if S is already 1. `Prk` **terminates**.
> 5. `Pri` unblocks: `P(S)` succeeds (S→0), prints `*` **(star #2)**, `V(T)`, `V(Z)` — but `Prj`/`Prk` have already finished, so these signals go unused.
> 6. `Pri` loops again: `P(S)` — S=0, and **no process remains** to ever call `V(S)` again → `Pri` **blocks forever**.
>
> **Answer: Minimum = Maximum = 2 stars** are printed (deterministic outcome — `Prj`/`Prk` finishing order doesn't change the total, since S is binary and clamps at value 1).

### Q10. Three threads, cyclic signaling to print "BCABCABCA…"

Threads T1, T2, T3 synchronized via binary semaphores S1, S2, S3 (context-switch order arbitrary):

| T1 | T2 | T3 |
|---|---|---|
| `while(true){` | `while(true){` | `while(true){` |
| `Wait(S3);` | `Wait(S1)` | `Wait(S2)` |
| `Print("C");` | `Print("B");` | `Print("A");` |
| `Signal(S2);}` | `Signal(S3);}` | `Signal(S1);}` |

**Which initialization of the semaphores prints the sequence `BCABCABCA…`?**

A) S1=1; S2=1; S3=1
B) S1=1; S2=1; S3=0
C) **S1=1; S2=0; S3=0** ✅
D) S1=0; S2=1; S3=1

> **Reasoning:** For `B` to print first, **T2** (which waits on S1) must run first ⇒ **S1 must start at 1**. All others (S2, S3) must start at **0** so that T1 (waits on S3) and T3 (waits on S2) are initially blocked, forcing the order **B → C → A → B → C → A …**:
> - T2: `Wait(S1)`✓(S1=1→0), print `B`, `Signal(S3)`(S3→1)
> - T1: `Wait(S3)`✓(now 1), print `C`, `Signal(S2)`(S2→1)
> - T3: `Wait(S2)`✓(now 1), print `A`, `Signal(S1)`(S1→1) → cycle repeats.
>
> **Answer: C (S1=1, S2=0, S3=0)**

### Q11. Counting Semaphore init to 2, mixed increment/decrement processes

Counting Semaphore initialized to **2**; 4 concurrent processes $P_i, P_j$ (increment shared variable C by 1 each) and $P_k, P_L$ (decrement C by 1 each), all updates performed **under semaphore control**. Initial `C = 0`. **Find min and max value of C after all processes finish.**

> **Reasoning:** Because the counting semaphore is initialized to **2**, it allows **up to 2 processes to enter the "critical" update section concurrently** — this is *not* full mutual exclusion (that would require initializing to 1). With 2-way concurrency permitted, **race conditions (lost updates)** on the shared variable `C` become possible if two processes read-modify-write `C` concurrently.
> - **Maximum C:** achieved if increments "win" the races and decrements' effects get overwritten/lost → up to **+2** (best case, only partial loss of decrement effects).
> - **Minimum C:** achieved if decrements "win" the races and increments get lost → down to **−2** (worst case).
>
> **Typical answer range: Minimum = −2, Maximum = +2** (compare with the *fully mutually-exclusive* case where C would always be deterministically $0$, i.e., $(+1)+(+1)+(-1)+(-1) = 0$).
> 💡 **Takeaway:** A counting semaphore initialized to `N > 1` **does not provide full mutual exclusion** — only a binary semaphore (or counting semaphore = 1) guarantees a fully race-free, deterministic final value.

### Q12. Deadlock-free ordering of 4 Binary Semaphores across 3 processes

Three processes X, Y, Z each acquire (via `P()`) four binary semaphores `a, b, c, d` in the sequences below. **Which sequence is Deadlock-Free?**

| Option | X | Y | Z |
|---|---|---|---|
| (I) | P(a);P(b);P(c) | P(b);P(c);P(d) | P(c);P(d);P(a) |
| **(II)** ✅ | P(b);P(a);P(c) | P(b);P(c);P(d) | P(a);P(c);P(d) |
| (III) | P(b);P(a);P(c) | P(c);P(b);P(d) | P(a);P(c);P(d) |
| (IV) | P(a);P(b);P(c) | P(c);P(b);P(d) | P(c);P(d);P(a) |

> **Method:** Build the **resource-acquisition graph** per option and check for a **cycle** in the "who might wait for whom" relation. A sequence is deadlock-free if no circular dependency (circular wait) can arise among the processes for any interleaving.
> - Option **(II)** is the one where no circular-wait pattern can be constructed among X, Y, Z regardless of scheduling — hence it is the **Deadlock-Free** option.
> - **Answer: (II)**
> 💡 Use this method for similar questions: draw out, for each process, which locks it holds *while waiting* for the next lock, then check if any cyclic chain "X waits for lock held by Y, Y waits for lock held by Z, Z waits for lock held by X" is possible.

### Q13. Barrier synchronization using count + 2 semaphores

`n` processes execute the following, sharing semaphores `a=1`, `b=0`, and shared variable `count=0` (not used in CODE SECTION P):

```c
CODE SECTION P
wait(a);
count = count + 1;
if (count == n) signal(b);
signal(a);
wait(b);
signal(b);
CODE SECTION Q
```

**What does the code achieve?**

A) **It ensures that no process executes CODE SECTION Q before every process has finished CODE SECTION P.** ✅
B) It ensures that two processes are in CODE SECTION Q at any time.
C) It ensures that all processes execute CODE SECTION P mutually exclusively.
D) It ensures that at most n−1 processes are in CODE SECTION P at any time.

> **Reasoning:** This is the classic **Barrier Synchronization** pattern. `count` (protected by mutex `a`) tallies how many processes have finished Section P; only when the **last** (n-th) process arrives does it `signal(b)`, unblocking everyone waiting at `wait(b)`. This guarantees **no process enters Section Q until ALL n processes have completed Section P** — the definition of a synchronization barrier. **Answer: A.** (C is close but incomplete — mutual exclusion in P is a *side-mechanism*, not the code's *purpose*; the actual purpose/achievement is the barrier described in A.)

### Q14. Non-atomic increment guarded by counting semaphore (S init = 5)

```c
1. int counter = 0
2. Semaphore S = init(5);
3. void parop(void)
4. {
5.     wait(S);
6.     wait(S);
7.     counter++;      // NOT atomic
8.     signal(S);
9.     signal(S);
10. }
```

**If five threads execute `parop` concurrently, which of the following program behavior(s) is/are possible?**

A) **There is a deadlock involving all the threads.** ✅
B) The value of counter is 5 after all threads successfully complete `parop`.
C) **The value of counter is 1 after all threads successfully complete `parop`.** ✅
D) The value of counter is 0 after all threads successfully complete `parop`.

> **Reasoning:**
> - Each thread calls `wait(S)` **twice** before proceeding — i.e., each thread consumes **2 permits** from the semaphore.
> - **Deadlock scenario (A):** If all 5 threads execute their *first* `wait(S)` before any executes the *second*, S drops to 0 after 5 first-waits — now **every thread is stuck at its second `wait(S)`** because no thread has reached `signal(S)` yet to release permits → **complete deadlock**. So **A is possible**.
> - **counter = 1 (C):** Since `counter++` is **not atomic**, if two threads manage to pass both waits "concurrently" (only 2 threads can hold both permits simultaneously since S=5 allows at most ⌊5/2⌋ = 2 pairs), a classic **lost-update race** can occur where both threads read `counter=0`, both write `counter=1` — so despite 5 threads incrementing, the final value can end up as low as **1**. So **C is possible**.
> - **counter = 5 (B):** would require **no lost updates at all** (i.e., increments never race) — while *possible* in principle with a very fortunate serialized schedule, this option is typically excluded in this well-known question's accepted answer set; the officially recognized correct options are **A and C**.
> - **counter = 0 (D) is impossible:** the very first thread to successfully execute `counter++` writes at least `1` (since `counter` starts at 0), and no subsequent lost update can push a *written* value back down to 0 — so once ANY thread completes the increment, the final counter can never be 0.
>
> **Answer: A and C**

### Q15. GATE 2023 — Incr()/Decr() with binary vs counting semaphore

```c
Incr() {                  Decr() {
   wait(s);                  wait(s);
   x = x + 1;                x = x - 1;
   signal(s);                signal(s);
}                          }
```

Five processes invoke `Incr()`, three processes invoke `Decr()`. `x` is a shared variable initialized to **10**.
- $I_1$: `s` value = 1 (**Binary semaphore**)
- $I_2$: `s` value = 2 (**Counting semaphore**)

Let $V_1$ and $V_2$ be the **minimum possible values** of `x` under $I_1$ and $I_2$ respectively.

A) **12, 7** ✅  B) 7, 7  C) 15, 7  D) 12, 8

> **Reasoning:**
> - **Under $I_1$ (binary semaphore = mutex):** Full mutual exclusion → *no* race conditions possible. Regardless of scheduling order, the final value is always deterministic:
>   $$x_{final} = 10 + 5(+1) - 3(-1) = 10 + 5 - 3 = 12$$
>   So $V_1 = 12$ (only possible value, hence also the minimum).
> - **Under $I_2$ (counting semaphore = 2):** Up to **2 processes** can be inside the critical section concurrently → **lost updates possible**. Working through the worst-case interleavings (pairing operations to maximize lost increments), the minimum achievable value of `x` works out to **7**.
>
> **Answer: A (12, 7)** — *this is a known GATE 2023 CS question.*

### Q16. Two threads with reverse-order semaphore acquisition (T1/T2)

Semaphores `s1` (init 1), `s2` (init 0); shared variable `x` (init 0):

```c
// T1                        // T2
wait(s1);                    wait(s1);
x = x + 1;                   x = x + 1;
print(x);                    print(x);
wait(s2);                    signal(s2);
signal(s1);                  signal(s1);
```

**Which of the following outcomes is/are possible when T1 and T2 execute concurrently?**

A) T1 runs first and prints 1, T2 runs next and prints 2.
B) **T2 runs first and prints 1, T1 runs next and prints 2.** ✅
C) **T1 runs first and prints 1, T2 does not print anything (deadlock).** ✅
D) T2 runs first and prints 1, T1 does not print anything (deadlock).

> **Reasoning:**
> - **If T1 runs first:** `wait(s1)` succeeds (s1→0), x=1, print(1), then `wait(s2)` — but s2=0 initially and only **T2** can signal it. Meanwhile **T2** tries `wait(s1)` — but s1=0 (T1 hasn't released it yet, since T1 is stuck at `wait(s2)`) → **T2 also blocks**. Now both threads are stuck → **deadlock** (matches option **C**).
> - **If T2 runs first:** `wait(s1)` succeeds, x=1, print(1), `signal(s2)` (s2→1), `signal(s1)` (s1→1) — **T2 fully completes**, releasing s1. Now T1 runs: `wait(s1)` succeeds, x=2, print(2), `wait(s2)` succeeds (already 1), `signal(s1)` — **T1 completes cleanly** (matches option **B**).
> - So the *only* two possible outcomes are **B** and **C** — option A is impossible (T1-first always deadlocks, never completes to print 2), and option D is impossible (T2-first never deadlocks).
>
> **Answer: B and C**

### Q17. MSQ — Three identical processes with a "counting trigger" semaphore

Three processes P1, P2, P3 run identical code. `A` (init 1), `B` (init 0) are binary semaphores; `X` (shared, init 0). Each line executes atomically.

```c
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

**Assuming any process can run first and context-switching can happen at any arbitrary point/order, which of the following output patterns is/are possible?**

A) **`**$*###`** ✅  B) **`**$#*##`** ✅  C) **`**$##*#`** ✅  D) `***$###`

> **Reasoning:**
> - Because `A` is a binary semaphore (mutex) around the `print(*); X++; [maybe print $]` block, the **stars are printed strictly one-at-a-time**, and the `$` (if printed) must appear **immediately after** the 2nd star with **nothing in between** — since the 2nd process to enter still holds `A` when it checks `X==2` and prints `$`.
> - This immediately **rules out option D** (`***$###`) — having all 3 stars print *before* the `$` would require the 3rd process to slip in its star between the 2nd process's star and its own `$` print, which is impossible while the 2nd process still holds the mutex `A`.
> - Once `$` is printed (and `B` gets signaled for the first time), the `#`-printing chain can begin — but this can interleave with the still-pending 3rd process's star in various valid ways, producing outputs **A**, **B**, and **C**, since:
>   - The 1st/2nd processes (already past the `A`-protected section) may complete their `#` phase before the 3rd process even starts its `*` phase (giving patterns like B, C).
>   - Or the 3rd star can print before any `#` (giving pattern A).
> - A `#` can **never** appear before its own process's `*` (program order), and a `#` can never appear before the very first `$` (since `B` starts at 0 and only gets signaled after `X==2`).
>
> **Answer: A, B, and C** (D is not possible)

---

## 🏁 End-of-Lecture Summary Table (Cheat Sheet)

| Concept | One-liner |
|---|---|
| Semaphore | Kernel ADT, integer value, atomic `wait()`/`signal()` |
| Binary Semaphore (BSEM) | Value ∈ {0,1}; same as Mutex |
| Counting Semaphore (CSEM) | Value ∈ (−∞, +∞); general resource counter |
| Busy-wait implementation | `while(S<=0);S--;` — wastes CPU (spinlock) |
| Blocking implementation | Negative value = # processes waiting; uses `block()`/`wakeup()` |
| Strong Semaphore | FIFO release order, starvation-free |
| Weak Semaphore | No release-order guarantee, starvation possible |
| Mutex Lock | Boolean lock; `acquire()`/`release()`; built via CAS; busy-wait ⇒ spinlock |
| Deadlock | Circular wait — avoid via consistent lock ordering |
| Starvation | Indefinite blocking of a specific process |
| Priority Inversion | Low-priority holds lock needed by high-priority; fixed via Priority-Inheritance Protocol |

---
*Notes compiled from GeeksforGeeks GATE CS&IT — "Principles of Operating Systems", Lecture 16: Process Synchronization (Semaphores).*
