# Process Synchronization — Classical IPC Problems (Lecture 18)

**Source:** GATE CS/IT — Principles of Operating Systems, Lecture 18 (GeeksforGeeks GATE)

## 📑 Topics Covered

| # | Topic |
|---|---|
| 1 | Producer–Consumer (Bounded Buffer) Problem — semaphore solution |
| 2 | Reader–Writer Problem — First Readers-Writers (semaphore solution) |
| 3 | Dining Philosophers Problem — naive deadlock-prone solution + fixes |
| 4 | Sleeping Barber (Barber Shop) Problem |
| 5 | Monitors — Producer-Consumer & Dining Philosophers using monitors |
| 6 | Concurrency — Precedence Graphs, Bernstein's Conditions, Concurrency vs Parallelism |
| 7 | GATE PYQs & Review Questions on all the above |

---

## 1. Producer–Consumer Problem (Bounded Buffer)

**Setup:** A *Producer* creates items and places them into a shared circular `Buffer[N]`; a *Consumer* removes items from it. The buffer has `N` slots — producer must not overwrite when full, consumer must not read when empty. Access to the buffer indices must be mutually exclusive.

- `in` → next free slot index (producer writes here)
- `out` → next full slot index (consumer reads here)

### Semaphore-based Solution

```c
#define N 100
int Buffer[N];

CSEM Empty = N;   // counting semaphore: number of EMPTY slots
CSEM Full  = 0;   // counting semaphore: number of FULL slots
BSEM mutex = 1;   // binary semaphore: protects Buffer as a Critical Section

void Producer(void)
{
    int itemp, in = 0;
    while (1)
    {
        itemp = Produce();      // a) produce an item
        Wait(Empty);             // b) wait for an empty slot
        Wait(mutex);              // c) enter critical section
        Buffer[in] = itemp;       // d) place item in buffer
        in = (in + 1) % N;        // e) advance in-pointer
        Signal(mutex);             // f) leave critical section
        Signal(Full);               // g) signal one more full slot
    }
}

void Consumer(void)
{
    int itemc, out = 0;
    while (1)
    {
        Wait(Full);               // a) wait for a full slot
        Wait(mutex);                // b) enter critical section
        itemc = Buffer[out];         // c) remove item from buffer
        out = (out + 1) % N;          // d) advance out-pointer
        Signal(mutex);                  // e) leave critical section
        Signal(Empty);                   // f) signal one more empty slot
        Process(itemc);                   // g) consume the item
    }
}
```

> ⚠️ **Order of `Wait()` calls matters!** Always `Wait()` on the *counting* semaphore (`Empty`/`Full`) **before** `Wait(mutex)`. If a process does `Wait(mutex)` first and then blocks on `Wait(Empty)`/`Wait(Full)` while holding `mutex`, the other process can never acquire `mutex` to proceed → **Deadlock**.
>
> Example: If Consumer does `Wait(mutex)` then `Wait(Full)` while buffer is empty → Consumer blocks *while holding mutex*. Producer needs `mutex` to add an item and signal `Full`, but can't get it → both processes wait forever.

**🧠 Mnemonic:** *"Count first, then commit."* Always check availability (counting semaphore) before entering the room (mutex).

---

## 2. Reader–Writer Problem (First Readers-Writers)

**Motivation:** A shared database (or any shared record) is accessed by:
- **Readers** — never modify the database, multiple readers can read simultaneously.
- **Writers** — read *and* modify; only one writer at a time, and no reader may be active while a writer writes.

**Goal:** Allow multiple concurrent readers, but exclusive access for a writer. A single lock on the whole DB is too restrictive since it would block simultaneous reads.

### Access Rules
- If a **writer** is active → no readers or other writers allowed.
- If a **reader** is active → no writers allowed, but other readers may join.

This variant (**First Readers-Writers**) favors **readers over writers** — a reader arriving while other readers are active does not wait for a queued writer, so writers can starve.

### Semaphore-based Solution

```c
int RC = 0;          // ReaderCount: number of active readers
BSEM mutex = 1;       // protects RC
BSEM db = 1;           // protects the actual database access

void Writer(void)
{
    while (1)
    {
        Down(db);         // a) exclusive lock on DB
        <DB_WRITE>;        // b) write to DB
        Up(db);             // c) release DB
    }
}

void Reader(void)
{
    while (1)
    {
        P(mutex);                   // a) protect RC
        RC = RC + 1;                  // b) one more reader
        if (RC == 1) P(db);            // c) FIRST reader locks DB from writers
        V(mutex);                       // d) release RC lock
        <DB_READ>;                        // e) read DB
        P(mutex);                          // f) protect RC again (uses Down/P interchangeably)
        RC = RC - 1;                         // g) one less reader
        if (RC == 0) V(db);                   // h) LAST reader unlocks DB for writers
    }
}
```

**🧠 Key insight:** Only the **first** reader to arrive locks `db` (blocking writers), and only the **last** reader to leave unlocks it. Readers in between just increment/decrement `RC` — that's why `RC` needs its own `mutex`.

---

## 3. Dining Philosophers Problem

**Setup:** `N` (typically 5) philosophers sit at a circular table. Between every pair of adjacent philosophers lies **one fork**. A philosopher needs **both** the left and right fork to eat. Philosophers alternate between *thinking* and *eating*. Condition: `N ≥ 2`.

### Naive Solution (causes Deadlock!)

```c
#define N 5
void Philosopher(int i)
{
    while (1)
    {
        Think(i);                    // a) think
        Take_fork(i);                  // b) pick up left fork
        Take_fork((i + 1) % N);          // c) pick up right fork
        Eat(i);                            // d) eat
        Put_fork(i);                         // e) put down left fork
        Put_fork((i + 1) % N);                 // f) put down right fork
    }
}
```

**Why it deadlocks:** If **all 5 philosophers simultaneously** pick up their **left** fork first, every fork is held by exactly one philosopher, and every philosopher is now waiting forever for their right fork (held by their neighbor). This is a **circular wait** — one of the four necessary conditions for deadlock — visualized as:
`P0←f0, P1←f1, P2←f2, P3←f3, P4←f4` (each Pi holds fi and waits for f(i+1)%N) → classic circular-wait deadlock.

### Solutions to Avoid Deadlock

```
Deadlock Avoidance
├── Semaphore-based
│     └── Tanenbaum's Solution (uses a global mutex + per-philosopher semaphores + state array)
└── Non-Semaphore-based (break the circular-wait symmetry)
      ├── Asymmetric fork order:
      │     (N-1) philosophers pick Left→Right, 1 philosopher picks Right→Left
      └── Odd/Even parity rule:
            Odd-numbered philosophers pick Left→Right, Even-numbered pick Right→Left
```

**🧠 Mnemonic:** *"Break the symmetry, break the deadlock."* As long as **not everyone** reaches for the same-side fork first, the circular wait chain can't form.

---

## 4. Sleeping Barber (Barber Shop) Problem

**Scenario:** A barbershop has **one barber chair** (cutting room) and a **waiting room** with a limited number of chairs.

**Rules:**
1. There is **one barber**, one barber's chair, and a waiting room with a fixed number of chairs.
2. When a customer arrives: if the barber is **sleeping**, the customer wakes him and sits in the barber's chair.
3. If the barber is **busy**, the customer sits in the waiting room (if a seat is free) and waits their turn.
4. When the barber finishes a haircut, he checks the waiting room: if customers are waiting, he brings one in; if none, he goes back to the chair and **sleeps**.
5. If a customer arrives and **all waiting chairs are occupied**, the customer **leaves** (balks) without a haircut.

**🧠 Mnemonic:** Barber = **Consumer** (of customers), Waiting Room = **Bounded Buffer**, Customers = **Producers**. It's a variant of the Producer-Consumer problem with a *bounded buffer + a "balking" customer if buffer is full* + a "sleeping" consumer if buffer is empty.

---

## 5. Solving Classical Problems with Monitors

A **Monitor** is a high-level synchronization construct: it bundles shared data + procedures that operate on it, and guarantees that **only one process can execute any procedure of the monitor at a time** (built-in mutual exclusion). Blocking/waiting is done via **condition variables** with `wait()` and `signal()`.

### 5.1 Producer–Consumer using a Monitor

```c
monitor ProducerConsumer
{
    condition full, empty;
    int count;

    procedure enter()
    {
        if (count == N) wait(full);     // if buffer is full, block
        put_item(widget);                 // put item in buffer
        count = count + 1;                  // increment count of full slots
        if (count == 1) signal(empty);        // if buffer WAS empty, wake a waiting consumer
    }

    procedure remove()
    {
        if (count == 0) wait(empty);     // if buffer is empty, block
        remove_item(widget);               // remove item from buffer
        count = count - 1;                   // decrement count of full slots
        if (count == N - 1) signal(full);      // if buffer WAS full, wake a waiting producer
    }

    count = 0;
} // end monitor

Producer()
{
    while (TRUE)
    {
        make_item(widget);            // make a new item
        ProducerConsumer.enter();       // call enter procedure of monitor
    }
}

Consumer()
{
    while (TRUE)
    {
        ProducerConsumer.remove();    // call remove procedure of monitor
        consume_item(widget);           // consume the item
    }
}
```

> 📝 **Note:** The condition variable names (`full`/`empty`) here are used purely as *wait-queues* — `wait(full)` means "block until someone signals `full`", not "block because full is true". Read carefully; this naming trips up many students.

### 5.2 Dining Philosophers using a Monitor

Each philosopher has a `state`: `THINKING`, `HUNGRY`, or `EATING`. A philosopher can only move to `EATING` if **neither neighbor is eating**.

```c
Monitor DiningPhilosophers
{
    enum {THINKING, HUNGRY, EATING} state[5];
    condition self[5];

    void pickup(int i)
    {
        state[i] = HUNGRY;
        test(i);                          // try to start eating immediately
        if (state[i] != EATING)
            self[i].wait();                 // else block until signaled
    }

    void putdown(int i)
    {
        state[i] = THINKING;
        test((i + 4) % 5);                // check left neighbor
        test((i + 1) % 5);                  // check right neighbor
    }

    void test(int i)
    {
        if ((state[(i + 4) % 5] != EATING) &&
            (state[i] == HUNGRY) &&
            (state[(i + 1) % 5] != EATING))
        {
            state[i] = EATING;
            self[i].signal();
        }
    }

    void initialization_code()
    {
        for (int i = 0; i < 5; i++)
            state[i] = THINKING;
    }
} // End of Monitor
```

**Why this is deadlock-free:** A philosopher only transitions to `EATING` inside `test()`, which explicitly checks both neighbors aren't eating — so two adjacent philosophers can never eat simultaneously, and the check happens atomically inside the monitor (no race condition), eliminating the circular-wait scenario entirely.

**🧠 Mnemonic:** *"Ask permission before grabbing anything."* Unlike the naive fork-grabbing version, here you declare intent (`HUNGRY`) and the monitor decides if it's safe — no physical fork is ever "held" while waiting.

---

## 6. Concurrency

### 6.1 Precedence Graphs

A **precedence graph** shows the order in which statements *must* execute due to data dependencies (an edge `Si → Sj` means `Si` must complete before `Sj` starts).

**Example:**
```
S1: a = b + c;
S2: d = e * f;
S3: k = a * d;
S4: l = k / 5;
```

Graph:
```
S1   S2
 \   /
  \ /
  S3
   |
  S4
```
`S1` and `S2` are independent (can run concurrently); both must finish before `S3` (which uses `a` and `d`); `S4` must wait for `S3` (uses `k`).

### 6.2 Bernstein's Conditions (for safe concurrent execution)

For any statement `S`, define:
- `R(S)` = set of variables **read** by `S`
- `W(S)` = set of variables **written** by `S`

Two statements `Si` and `Sj` can execute **concurrently** (in any order, or in parallel) **iff all three** hold:

$$
\begin{aligned}
1)\quad & R(S_i) \cap W(S_j) = \varnothing \\
2)\quad & W(S_i) \cap R(S_j) = \varnothing \\
3)\quad & W(S_i) \cap W(S_j) = \varnothing
\end{aligned}
$$

| Condition | Meaning if violated |
|---|---|
| $R(S_i) \cap W(S_j) \neq \varnothing$ | `Si` reads something `Sj` writes → order-dependent result |
| $W(S_i) \cap R(S_j) \neq \varnothing$ | `Sj` reads something `Si` writes → order-dependent result |
| $W(S_i) \cap W(S_j) \neq \varnothing$ | Both write the same variable → last-write-wins race |

**Worked Example:**
```
Si: a = b + c;
Sj: d = b * c;
```
- `R(Si) = {b, c}`, `W(Si) = {a}`
- `R(Sj) = {b, c}`, `W(Sj) = {d}`

Check: `R(Si) ∩ W(Sj) = {b,c} ∩ {d} = ∅` ✅, `W(Si) ∩ R(Sj) = {a} ∩ {b,c} = ∅` ✅, `W(Si) ∩ W(Sj) = {a} ∩ {d} = ∅` ✅

→ All three conditions hold ⇒ `Si` and `Sj` **can safely run concurrently**.

**🧠 Mnemonic:** *"No one should read what another is still writing, and no two should write to the same place."*

### 6.3 Concurrency vs Parallelism

| Aspect | **Concurrency** | **Parallelism** |
|---|---|---|
| Definition | System supports two or more actions **in progress** at the same time | System supports two or more actions **executing simultaneously** |
| Core idea | Dealing with **lots of things at once** (interleaved) | **Doing** lots of things at once (truly simultaneous) |
| Hardware requirement | Can happen on a **single core** (via time-slicing/interleaving) | Requires **multiple cores/processors** |
| Analogy | 2 queues, 1 vending machine (interleaved service) | 2 queues, 2 vending machines (simultaneous service) |
| Execution timeline | Tasks overlap in time but each runs in bursts, alternating | Tasks literally run at the same instant on different execution units |

**Sequential vs Concurrent vs Parallel (timing view):**
- **Sequential:** Process 1 fully finishes → then Process 2 starts (no overlap)
- **Concurrent:** Both processes make progress, execution is *interleaved* on shared resource(s)
- **Parallel:** Both processes literally run *at the same time* on separate resources

**🧠 Mnemonic:** *"Concurrency is about structure (dealing with many things); Parallelism is about execution (doing many things at once)."*

---

## 7. GATE PYQs & Review Questions

### Q1. Which classical IPC problem is mainly concerned with deadlock and starvation among multiple processes sharing limited resources like forks?
- A. Producer-Consumer Problem
- B. Readers-Writers Problem
- **✅ C. Dining Philosophers Problem**
- D. Sleeping Barber Problem

**Why others are wrong:** (A) is about buffer full/empty synchronization, not resource deadlock; (B) is about read/write access control, not shared *physical* resources like forks; (D) is about a bounded waiting-room and balking customers, not deadlock/circular wait.

---

### Q2. In the Producer-Consumer problem, which data structure is most commonly used for managing the buffer between producer and consumer?
- A. Stack
- **✅ B. Queue**
- C. Tree
- D. Hash Table

**Why others are wrong:** The buffer must preserve **FIFO order** (items are consumed in the order produced) via `in`/`out` circular indices — this is exactly queue behavior. A Stack (LIFO) would consume the most-recently-produced item first; Tree/Hash Table don't naturally support ordered circular insert/remove.

---

### Q3. Which synchronization tool is best suited to resolve the Readers-Writers problem while prioritizing readers over writers?
- A. Mutex
- **✅ B. Counting Semaphore** *(used for reader count, combined with a binary semaphore for DB lock)*
- C. Read-Write Lock
- D. Binary Semaphore

**Why others are wrong:** (A)/(D) A single mutex/binary semaphore alone can't track "how many readers are active" — you need a **counting** mechanism (`RC`) plus a binary lock for the DB; (C) Read-Write Lock is a higher-level abstraction, not the fundamental primitive taught in the classical semaphore solution.

---

### Q4. What is the major issue addressed in the Sleeping Barber problem?
- **✅ A. Resource starvation** *(bounded waiting room, customers may leave if full)*
- B. Deadlock avoidance
- C. Process scheduling in multi-core systems
- D. Synchronization between a single producer and consumer

**Why others are wrong:** (B) Deadlock isn't the central issue here — it's a bounded-buffer/synchronization variant with balking; (C) unrelated to multi-core scheduling; (D) it's actually *many* customers (producers) and one barber (consumer), not a single-producer scenario.

---

### Q5. In the Dining Philosophers problem, which approach helps to prevent deadlock?
- A. Philosophers pick both forks simultaneously
- **✅ B. Limiting the number of philosophers allowed to sit at the table** *(e.g., allow at most N−1 to attempt eating)*
- C. Allowing philosophers to eat without thinking
- D. Giving each philosopher only one fork

**Why others are wrong:** (A) "Simultaneously" isn't atomically guaranteed in most implementations and doesn't prevent circular wait by itself unless combined with a global lock; (C) irrelevant to fork acquisition/deadlock; (D) each philosopher NEEDS 2 forks to eat — giving only one guarantees starvation, not a fix.

---

### Q6. Which of the following problems is NOT typically associated with concurrent programming?
- A. Race conditions
- B. Deadlocks
- C. Starvation
- **✅ D. Compilation errors**

**Why others are wrong:** (A), (B), (C) are all classic **runtime** synchronization hazards in concurrent programs; compilation errors are a static/syntax-time issue unrelated to concurrent execution behavior.

---

### Q7 (GATE-style). Producer-Consumer semaphore assignment puzzle
> Consider the following solution to the producer-consumer synchronization problem. The shared buffer size is N. Three semaphores `empty`, `full`, and `mutex` are defined with respective initial values of **0, N, and 1**. Semaphore `empty` denotes the number of available slots in the buffer for the **consumer to read from** (it fills up as producer writes). Semaphore `full` denotes the number of available slots for the **producer to write to** (starts at N, empties as producer writes). Placeholders `P, Q, R, S` can be assigned either `empty` or `full`.

```
Producer:                          Consumer:
do {                                do {
  1. wait(P);                        3. wait(R);
     wait(mutex);                       wait(mutex);
     // Add item to buffer                // Consume item from buffer
     signal(mutex);                        signal(mutex);
  2. signal(Q);                       4. signal(S);
} while (1);                        } while (1);
```

**Which assignment to P, Q, R, S yields a correct solution?**
- A. P: full, Q: full, R: empty, S: empty
- B. P: empty, Q: empty, R: full, S: full
- **✅ C. P: full, Q: empty, R: empty, S: full**
- D. P: empty, Q: full, R: full, S: empty

**Reasoning:** Because the *names are intentionally swapped* from convention here (`full` starts at N — the "available slots to write" semaphore — and `empty` starts at 0 — the "items available to read" semaphore):
- Producer must **wait** on the semaphore that starts at N (available write-slots) → **P = full**
- Producer, after writing, **signals** the semaphore consumers wait on (items available) → **Q = empty**
- Consumer must **wait** on that same "items available" semaphore → **R = empty**
- Consumer, after reading, **signals** the "slots available" semaphore → **S = full**

**Why others are wrong:** (A), (B), (D) all mismatch at least one wait/signal pair against the (swapped) initial-value semantics, which would cause either premature blocking or writing past buffer bounds.

**🧠 Exam tip:** Always trace initial values first — don't assume semaphore *names* match convention; the *initial value* tells you its true role.

---

### Q8 (GATE 2014). Producer-Consumer TRUE/FALSE statement
```c
semaphore n = 0;
semaphore s = 1;

void producer() {
  while(true) {
    produce();
    semWait(s);
    addToBuffer();
    semSignal(s);
    semSignal(n);
  }
}

void consumer() {
  while(true) {
    semWait(s);
    semWait(n);
    removeFromBuffer();
    semSignal(s);
    consume();
  }
}
```
**Which one of the following is TRUE?**
- A. The producer will be able to add an item to the buffer, but the consumer can never consume it.
- B. The consumer will remove no more than one item from the buffer.
- **✅ C. Deadlock occurs if the consumer succeeds in acquiring semaphore `s` when the buffer is empty.**
- D. The starting value for the semaphore `n` must be 1 and not 0 for deadlock-free operation.

**Reasoning:** If the consumer acquires `s` (mutex) first while `n = 0` (buffer empty), it then blocks on `semWait(n)` — but it's **still holding `s`**! The producer needs `s` to add an item and signal `n`, but can never get it → classic deadlock (same root cause as the "wait order matters" warning in Section 1).

**Why others are wrong:** (A) producer CAN add and consumer CAN consume in the non-deadlocking interleavings; (B) consumer can remove multiple items over repeated loop iterations, nothing restricts it to one; (D) `n=0` is actually the *correct* initial value (buffer starts empty) — the real bug is the *order* of `semWait(s)` and `semWait(n)` in the consumer, not the initial value.

---

### Q9 (GATE-style). Readers-Writers fill-in-the-blanks
```
wait (wrt)                              wait (mutex)
writing is performed                      readcount = readcount + 1
signal (wrt)                              if readcount = 1 then S1
                                               S2
                                           reading is performed
                                           S3
                                           readcount = readcount - 1
                                           if readcount = 0 then S4
                                       signal (mutex)
```
**The values of S1, S2, S3, S4 (in that order) are:**
- A. signal(mutex), wait(wrt), signal(wrt), wait(mutex)
- B. signal(wrt), signal(mutex), wait(mutex), wait(wrt)
- **✅ C. wait(wrt), signal(mutex), wait(mutex), signal(wrt)**
- D. signal(mutex), wait(mutex), signal(mutex), wait(mutex)

**Reasoning:** S1 runs when the **first** reader arrives → must **lock out writers** → `wait(wrt)`. S2 must release the `readcount` mutex before the (possibly long) reading operation → `signal(mutex)`. S3 must re-acquire the mutex before decrementing `readcount` → `wait(mutex)`. S4 runs when the **last** reader leaves → must **unlock writers** → `signal(wrt)`. This exactly mirrors the "First Readers-Writers" solution in Section 2.

**Why others are wrong:** All other options place a `signal`/`wait` mismatched with the direction of the lock (e.g., trying to `signal(wrt)` before a writer-block is ever acquired, or double-locking `mutex`), which breaks either mutual exclusion or the reader-count protocol.

---

### Q10 (GATE-style). Dining Philosophers deadlock-avoidance policy
> A solution to the Dining Philosophers Problem which avoids deadlock is:
- A. Ensure that all philosophers pick up the left fork before the right fork
- B. Ensure that all philosophers pick up the right fork before the left fork
- **✅ C. Ensure that one particular philosopher picks up the left fork before the right fork, and all other philosophers pick up the right fork before the left fork**
- D. None of the above

**Reasoning:** This is exactly the **asymmetric solution** from Section 3 — breaking the symmetry (not everyone reaching for the same-side fork) prevents the circular-wait cycle that causes deadlock.

**Why others are wrong:** (A) and (B) are symmetric — if *everyone* follows the same order, all 5 can simultaneously grab their first fork and deadlock exactly as shown in the naive solution; (D) is wrong since C is a valid, well-known solution.

---

### Q11 (GATE-style). Basic Principle of a Monitor
> The basic principle of a monitor is that:
- A. Several resources can only be controlled by a monitor.
- B. Several processes may concurrently execute a procedure of a given monitor.
- **✅ C. Only one process may execute a procedure of a given monitor at any given time.**
- D. It schedules the execution of processes in a Multiprocessor Operating System.

**Reasoning:** This is the **defining property** of monitors — built-in mutual exclusion across ALL procedures of the monitor (see Section 5 intro).

**Why others are wrong:** (A) monitors don't limit how many resources exist, just enforce exclusive procedure access; (B) directly contradicts the core mutual-exclusion guarantee; (D) monitors are a synchronization construct, not a CPU scheduler.

---

### Q12 (GATE-style). Match the Pairs
| Column A | Column B |
|---|---|
| (a) Critical Region | (p) Hoare's Monitor |
| (b) Wait/Signal | (q) Mutual Exclusion |
| (c) Working Set | (r) Principle of Locality |
| (d) Deadlock | (s) Circular Wait |

**✅ Correct matching:** a→q, b→p, c→r, d→s

**Reasoning:**
- **Critical Region ↔ Mutual Exclusion** — a critical region/section is defined precisely to enforce mutual exclusion.
- **Wait/Signal ↔ Hoare's Monitor** — the `wait()`/`signal()` operations on condition variables are Hoare's formalization of monitors.
- **Working Set ↔ Principle of Locality** — the working-set model is built directly on the principle of locality of reference.
- **Deadlock ↔ Circular Wait** — circular wait is one of the 4 necessary conditions for deadlock (and the one most commonly targeted for prevention).

---

### Q13 (GATE 2024 Set 1 — verified). Thread interleaving with two shared variables

> Consider two threads T1 and T2 that update shared variables `a` and `b`. Initially `a = b = 1`. Each statement executes atomically, but context switches can happen at any time between statements.

```
T1                    T2
a = a + 1;            b = 2 * b;
b = b + 1;            a = 2 * a;
```

**Which one of the following lists ALL possible combinations of (a, b) after both threads finish?**
- **✅ A. (a=4, b=4); (a=3, b=3); (a=4, b=3)**
- B. (a=3, b=4); (a=4, b=3); (a=3, b=3)
- C. (a=4, b=4); (a=4, b=3); (a=3, b=4)
- D. (a=2, b=2); (a=2, b=3); (a=3, b=4)

**Worked solution — enumerate all 6 valid interleavings** (order within each thread preserved):

| Order | Steps | Result |
|---|---|---|
| T1-S1, T1-S2, T2-S1, T2-S2 | a=2, b=2, b=2×2=4, a=2×2=4 | (4, 4) |
| T1-S1, T2-S1, T1-S2, T2-S2 | a=2, b=2×1=2, b=2+1=3, a=2×2=4 | (4, 3) |
| T1-S1, T2-S1, T2-S2, T1-S2 | a=2, b=2, a=2×2=4, b=2+1=3 | (4, 3) |
| T2-S1, T1-S1, T1-S2, T2-S2 | b=2×1=2, a=1+1=2, b=2+1=3, a=2×2=4 | (4, 3) |
| T2-S1, T1-S1, T2-S2, T1-S2 | b=2, a=2, a=2×2=4, b=2+1=3 | (4, 3) |
| T2-S1, T2-S2, T1-S1, T1-S2 | b=2×1=2, a=2×1=2, a=2+1=3, b=2+1=3 | (3, 3) |

Distinct outcomes: **{(4,4), (4,3), (3,3)}** → matches **Option A**.

**Why others are wrong:** B, C, D each include at least one (a,b) pair that never actually occurs under any valid interleaving (verify by tracing — e.g., (3,4) or (2,2) never appear above).

---

### Q14 (GATE 2015 Set 1 — verified). Distinct values of shared variable `B`

```c
int B = 2;
BSEM mx = 1;   // (illustrative — original problem has NO locking!)

P1() { C = B - 1;  B = 2 * C; }
P2() { D = 2 * B;  B = D - 1; }
```
> If P1 and P2 execute **concurrently** (no synchronization), the number of distinct values B can take after execution is: **______**

**✅ Answer: 3** (possible final values: **2, 3, 4**)

**Worked solution:**

| Interleaving | Trace | Final B |
|---|---|---|
| P1 fully, then P2 | C=1, B=2; then D=2×2=4, B=4-1=3 | **3** |
| P2 fully, then P1 | D=4, B=3; then C=3-1=2, B=2×2=4 | **4** |
| P1.1, P2.1, P2.2, P1.2 | C=1; D=2×2=4; B=4-1=3; B=2×1=2 | **2** |
| P2.1, P1.1, P1.2, P2.2 | D=4; C=1; B=2×1=2; B=4-1=3 | **3** |
| (other interleavings) | — | 2, 3, or 4 |

Only **{2, 3, 4}** ever occur → **3 distinct values**.

**🧠 Exam tip:** For "number of distinct values" style questions, systematically enumerate valid interleavings (respecting per-thread statement order) rather than guessing — there are only `C(total_statements, statements_of_one_thread)` orderings to check.

---

### Q15 (Practice). Precedence Graph Construction
> Draw the precedence graph for the concurrent program:
```
S1;
Parbegin
    begin S2; S4; end
    begin
        S3;
        Parbegin
            S5;
            begin S6; S8; end
        Parend
    end
    S7;
Parend;
S9;
```
**Approach:** `Parbegin...Parend` marks a fork — all branches inside can run concurrently but must ALL complete before the statement after `Parend`. `Sequential ;` within a branch preserves order.
- `S1` → forks into 3 branches: `{S2→S4}`, `{S3 → (forks into {S5}, {S6→S8})}`, `{S7}`
- All 3 top-level branches must finish before `S9`.

*(Practice exercise — sketch the DAG: S1 fans out to S2, S3, S7; S2→S4; S3 fans out to S5 and S6; S6→S8; S4, S5, S8, S7 all converge into S9.)*

---

### Q16 (Practice). Valid Output Interleavings
```c
void Foo(void) { X; Y; Z; }
void Boo(void) { A; B; }

main() {
    Parbegin
        Foo();
        Boo();
    Parend
}
```
> List out the valid output sequences.

**Approach:** Any interleaving of `{X,Y,Z}` (order preserved) with `{A,B}` (order preserved) is valid. Total valid sequences = $\binom{5}{2} = 10$ (choose 2 positions out of 5 for A,B; the rest are X,Y,Z in order). E.g.: `XYZAB, XYAZB, XYABZ, XAYZB, XAYBZ, XABYZ, AXYZB, AXYBZ, AXBYZ, ABXYZ`.

---

### Q17 (Practice). Final values of `a`, `b` with no synchronization
```c
int a = 0, b = 0;
Parbegin
    begin  1: a = 1;  2: b = b + a;  end
    begin  3: b = 2;  4: a = a + 3;  end
Parend
```
> What are the possible final values of `a` and `b`? Given options: I) a=1,b=2  II) a=1,b=3  III) a=4,b=6

**Approach:** Enumerate all $\binom{4}{2}=6$ interleavings of `{1,2}` and `{3,4}` (each pair order-preserved):

| Order | Trace | (a, b) |
|---|---|---|
| 1,2,3,4 | a=1; b=0+1=1; b=2; a=1+3=4 | (4, 2) |
| 1,3,2,4 | a=1; b=2; b=2+1=3; a=1+3=4 | (4, 3) |
| 1,3,4,2 | a=1; b=2; a=1+3=4; b=2+4=6 | (4, 6) |
| 3,1,2,4 | b=2; a=1; b=2+1=3; a=1+3=4 | (4, 3) |
| 3,1,4,2 | b=2; a=1; a=1+3=4; b=2+4=6 | (4, 6) |
| 3,4,1,2 | b=2; a=0+3=3; a=1; b=2+1=3 | (1, 3) |

Possible pairs: **(4,2), (4,3), (4,6), (1,3)**. Among the given options, **II (a=1,b=3)** and **III (a=4,b=6)** are achievable; **I (a=1,b=2)** never occurs.

---

### Q18 (Practice). Mutex-protected critical sections
```c
int a = 0, b = 20;
BSEM mx = 1;
Parbegin
    begin P(mx); a = a + 1; V(mx); end
    begin P(mx); a = b + 1; V(mx); end
Parend
// Final possible values of a?
```
**Approach:** Since both blocks are protected by the same `mx`, they execute in **some strict order** (no interleaving *within* the critical sections) — only 2 orderings possible:

| Order | Trace | Final `a` |
|---|---|---|
| `a=a+1` first | a = 0+1 = 1; then a = b+1 = 20+1 = 21 | **21** |
| `a=b+1` first | a = 20+1 = 21; then a = a+1 = 21+1 = 22 | **22** |

**✅ Final possible values of `a`: {21, 22}**

---

### Q19 (Practice). Race condition — min/max of a shared counter
```c
int c = 0;
void Foo() {
    int i, n = 10;
    for (i = 1; i <= n; ++i)
        c = c + 1;     // NOT atomic! read-modify-write
}
cobegin
    Foo();
    Foo();
coend
```
> What is the minimum and maximum value of `c`?

- **Maximum = 20**: if the two `Foo()` calls never interleave mid-increment (fully sequential or non-overlapping reads/writes), all 20 increments are counted correctly.
- **Minimum = 2**: in the worst-case interleaving, both threads can repeatedly read the **same stale value** of `c` before either writes back, causing massive "lost updates" — but at least the very last write in each thread's final iteration still increments once more than what was overwritten, bounding the minimum around 2 (extreme race condition).

**🧠 Takeaway:** Unprotected `c = c + 1` is a classic **read-modify-write race** — always protect shared counters with a mutex/semaphore in real code.

---

## 8. Quick Revision Checklist ✅

Use this to self-test before re-reading the full notes:

- [ ] Can you write the Producer-Consumer semaphore solution from memory (Empty, Full, mutex)?
- [ ] Do you know **why** the order of `Wait()` calls matters (deadlock if `mutex` acquired before counting semaphore)?
- [ ] Can you state the First Readers-Writers rule: who locks `db`, and when (first reader in / last reader out)?
- [ ] Can you explain why the naive Dining Philosophers solution deadlocks (circular wait)?
- [ ] Can you name at least 2 solutions to avoid Dining Philosophers deadlock (Tanenbaum's, asymmetric ordering, odd/even rule)?
- [ ] Can you describe the 5 rules of the Sleeping Barber problem?
- [ ] Can you write the monitor-based solution for Producer-Consumer (`enter()`/`remove()` with `count`)?
- [ ] Can you write the monitor-based solution for Dining Philosophers (`state[]`, `test()`, `pickup()`, `putdown()`)?
- [ ] Do you know Bernstein's 3 conditions for safe concurrent execution ($R \cap W$, $W \cap R$, $W \cap W$ all empty)?
- [ ] Can you build a precedence graph from a `Parbegin/Parend` program?
- [ ] Can you clearly distinguish **Concurrency** (dealing with many things, can be single-core) vs **Parallelism** (doing many things, needs multiple cores)?
- [ ] Can you enumerate interleavings systematically to solve "final value of shared variable" GATE questions?
- [ ] Do you remember the monitor's core guarantee: only ONE process executes ANY monitor procedure at a time?

---

*Notes compiled from Lecture 18 — Classical IPC Problems (GeeksforGeeks GATE CS/IT series). GATE PYQ answers cross-verified against official answer keys where applicable (GATE 2014, 2015 Set 1, 2024 Set 1).*
