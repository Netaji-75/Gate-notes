# Process Synchronization — Lecture 14 (GATE CS/IT Revision Notes)

> Source: GeeksforGeeks GATE CS & IT — Principles of Operating Systems, Lecture 14 (Process Synchronization III)

## 📑 Topics Covered

| # | Topic |
|---|-------|
| 1 | Requirements of the Critical Section (CS) Problem |
| 2 | Lock Variable (Software Solution — flawed) |
| 3 | Strict Alternation (2-Process Software Solution — flawed) |
| 4 | Peterson's Solution (Software Solution — correct for 2 processes) |
| 5 | Peterson's Solution on Modern Architecture (instruction reordering problem) |
| 6 | Memory Barrier |
| 7 | Dekker's Algorithm |
| 8 | Review Questions / Quick Checks |
| 9 | GATE PYQs |

---

## 1. Critical Section Problem — Requirements

The **critical section (CS)** is the part of a program where a process accesses **shared resources** (shared variables, files, data structures, devices, etc.).

Any valid solution to the CS problem **must** satisfy 3 conditions:

| Requirement | Meaning |
|---|---|
| **Mutual Exclusion** | No two processes can execute in their critical sections at the same time. |
| **Progress** | If no process is in CS, and some processes want to enter, the decision of who enters next cannot be postponed indefinitely (only processes not in remainder section participate in this decision). |
| **Bounded Waiting** | There must be a limit on the number of times other processes are allowed to enter CS after a process has requested entry and before that request is granted (prevents starvation). |

> ⚠️ **Note:** *Deadlock-freedom* and *Preemption* are **NOT** part of the 3 official CS requirements — a common GATE distractor.

**General structure of any CS solution:**
```
Entry Section     -> code to request permission to enter CS
Critical Section  -> actual access to shared resource
Exit Section      -> code to release / signal permission
Remainder Section -> rest of the code (doesn't touch shared resource)
```

---

## 2. Software Solution Attempt #1 — Lock Variable

**Idea:** Use a shared variable `Lock`. `0` = unlocked, `1` = locked.

```
Entry Section:
    while (Lock != 0);   // busy wait until lock is free
    Lock = 1;            // acquire lock

Critical Section

Exit Section:
    Lock = 0;            // release lock
```

**Equivalent Assembly (why it fails):**
```
1. Load  Ri, M[Lock]     // Load lock value into register
2. CMP   Ri, #0          // Compare register value with 0
3. JNZ   step1           // Jump back to step 1 if not equal (busy wait)
4. Store M[Lock], #1     // Mark lock as taken -> enters CS
5. CS
6. Store #0, Lock        // Exit CS, release lock
```

**Why it fails (Mutual Exclusion violation):**
Steps 1–4 are **not atomic**. A process can be preempted right after step 3 (check passed) but before step 4 (store). Another process can then also pass its check and enter the CS simultaneously → **Mutual Exclusion is violated**.

🧠 **Mnemonic:** *"Check-then-act" without atomicity always breaks mutual exclusion — this is the core problem behind almost every naive synchronization attempt.*

---

## 3. Software Solution Attempt #2 — Strict Alternation

**Idea:** Use a shared variable `turn` that dictates **whose turn** it is to enter CS — strictly alternating between two processes.

- `turn == 0` → P0's turn to access CS
- `turn == 1` → P1's turn to access CS

```c
int turn = 0;   // or rand(0,1) initially

void Process(int i) {
    int j = NOT(i);        // the other process id
    while (1) {
        Non_CS();
        while (turn != i); // (b) wait for your turn — busy wait
        <CS>                // (c)
        turn = j;           // (d) hand over turn to the other process
    }
}
```

**Satisfies:**
- ✅ **Mutual Exclusion** — only the process whose turn it is can proceed past the `while`.
- ✅ **Bounded Waiting** — strictly alternates, so a process waits *at most* one turn of the other process.

**Fails:**
- ❌ **Progress** — Because processes are **forced to strictly alternate**, if P0 finishes CS and is not interested in re-entering for a long time (busy in remainder section) but P1 wants to enter again immediately, P1 **cannot** enter until it becomes its turn — even though P0 doesn't want the CS. This violates Progress (a process not in remainder section is forced to wait for a process that isn't even requesting CS).

🧠 **Mnemonic:** *"Strict Alternation = 100% turn-based" → great for Mutual Exclusion + Bounded Waiting, but terrible for Progress because you can't skip your "opponent's" turn even if they don't want it.*

| Property | Lock Variable | Strict Alternation |
|---|---|---|
| Mutual Exclusion | ❌ (not atomic) | ✅ |
| Progress | N/A (fails ME itself) | ❌ |
| Bounded Waiting | N/A | ✅ |
| Busy Waiting? | Yes | Yes |
| Works for how many processes | — | Only 2 |

---

## 4. Peterson's Solution (1981) — Correct Software Solution (2 Processes)

**Category:** Software solution, busy-waiting, **2-process solution only** = `(Lock Variable + Strict Alternation)` combined cleverly.

Peterson's solution combines:
- A `flag[]` array — **intent** to enter CS (`flag[i] = true/false`)
- A `turn` variable — **tiebreaker** deciding whose turn it is when both want CS

### Code

```c
// Shared variables
boolean flag[2] = {false, false};
int turn;

void Process(int i) {
    int j = 1 - i;              // the other process
    while (true) {
        flag[i] = true;          // 1. show interest in entering CS
        turn = j;                // 2. politely give turn to the other process
        while (flag[j] && turn == j);  // 3. wait only if other wants CS AND it's their turn
        
        /* critical section */   // 4.
        
        flag[i] = false;          // 5. leave CS, show no interest
        /* remainder section */
    }
}
```

**Original textbook (Tanenbaum-style) version:**
```c
#define FALSE 0
#define TRUE  1
#define N     2                    /* number of processes */

int turn;                          /* whose turn is it? */
int interested[N];                 /* all values initially 0 (FALSE) */

void enter_region(int process) {   /* process is 0 or 1 */
    int other;                     /* number of the other process */
    other = 1 - process;           /* the opposite of process */
    interested[process] = TRUE;    /* show that you are interested */
    turn = process;                /* set flag */
    while (turn == process && interested[other] == TRUE) /* null stmt */ ;
}

void leave_region(int process) {   /* process: who is leaving */
    interested[process] = FALSE;   /* indicate departure from critical region */
}
```

**Alternate presentation (using `waiting[]`):**
```c
// Shared variables
int turn = 0;
boolean waiting[2] = {FALSE, FALSE};

// Process 0                       // Process 1
do {                                do {
  waiting[0] = TRUE;                  waiting[1] = TRUE;
  turn = 0;                           turn = 1;
  while (turn == 0 &&                 while (turn == 1 &&
         waiting[1] == TRUE) {              waiting[0] == TRUE) {
     no_op;                              no_op;
  }                                    }
  critical section                    critical section
  waiting[0] = FALSE;                  waiting[1] = FALSE;
  remainder section                    remainder section
} while (TRUE);                     } while (TRUE);
```

### Line-by-line meaning (`flag`/`turn` version)

| Line | Meaning |
|---|---|
| `flag[i] = true;` | Process-i wants to be in the critical section next |
| `turn = j;` | It's currently Process-j's turn (i politely offers turn to j) |
| `while (flag[j] && turn == j);` | Wait only while: (1) process-j **also** wants CS, **and** (2) it's process-j's turn |
| `flag[i] = false;` | Process-i doesn't want CS anymore (in remainder section) |

### Why all 3 properties hold

| Requirement | Why it holds |
|---|---|
| **Mutual Exclusion** | If both `flag[i]` and `flag[j]` are true, only one value of `turn` can exist at a time — whichever process's turn it ISN'T gets blocked in the `while`. Only one process can pass the check. |
| **Progress** | If a process isn't interested (`flag[j] == false`), the waiting condition is false, so the other process enters immediately — no needless blocking. |
| **Bounded Waiting** | After releasing CS, `turn` is set to give the other process priority — a process waits at most one turn of the other process before entering. |

### Worked Trace Example (from the handwritten slide)

Shared: `flag[0]=F, flag[1]=F, turn=` (undefined initially)

| Step | Process | Action | flag[0] | flag[1] | turn |
|---|---|---|---|---|---|
| 1 | P0 (i=0,j=1) | executes line 1: `flag[0]=true` | T | F | — |
| 2 | P1 (i=1,j=0) | executes lines 1,2,3 → `flag[1]=true; turn=0; while(flag[0] && turn==0)` → **true**, P1 waits | T | T | 0 |
| 3 | P0 (i=0,j=1) | executes lines 2,3 → `turn=1; while(flag[1] && turn==1)` → **true**, P0 also waits?? |

> ⚠️ Note: This is exactly why the **order of "turn=i" AND "while" check** matters — the slide walks through interleaving to show that whichever process executes `turn = j` **last** ends up losing its own turn and waiting, guaranteeing the other enters CS first. Follow the code precisely: the *last* process to set `turn` is the one that ends up blocked, so the correct entry is always resolved to exactly one process. Eventually P1 enters CS (as shown: `i=1, j=0: 3 <CS>`).

🧠 **Mnemonic:** *Peterson = "I'll go first, but I'll let YOU go first" (`turn = j`) — it's software courtesy that guarantees fairness.*

---

## 5. Peterson's Solution on Modern Architecture — ⚠️ IT MAY NOT WORK!

> **Key exam point:** Peterson's Solution is only guaranteed correct on the **theoretical/simple sequential-consistency model** of computing. On **modern architectures** (with compiler optimizations, out-of-order CPUs, caches), it is **NOT guaranteed to work.**

**Reason:** To improve performance, **processors and/or compilers may reorder operations that have no data dependencies.**

- For **single-threaded** programs → reordering is safe (result is always the same because there's no observer of intermediate order).
- For **multi-threaded** programs → reordering **can produce inconsistent/unexpected results**, because another thread MAY observe the intermediate (reordered) state.

### Example: Independent Variable Reordering

```c
// Shared data between two threads
boolean flag = false;
int x = 0;

// Thread 1 performs:
while (!flag);
print x;

// Thread 2 performs:
S1: x = 100;
S2: flag = true;
```

**Expected output:** `100` (because Thread 1 waits for `flag` to become true, which is supposed to happen only *after* `x = 100`).

**But:** since `flag` and `x` are independent variables (no dependency), the compiler/CPU may reorder Thread 2's instructions as:
```c
flag = true;   // reordered to execute FIRST
x = 100;       // reordered to execute SECOND
```
**If this reordering occurs → output may be `0`!** (Thread 1 sees `flag == true` and reads `x` *before* it's actually set to 100.)

### Applying this to Peterson's Solution

```
process0                 turn = 0   --(reordered)-->  flag[0] = true  --> CS
process1     turn = 1, flag[1] = true  -->  CS
```

If the CPU reorders `turn = i` and `flag[i] = true` for either process, **both processes can end up in the CS at the same time** → **violates Mutual Exclusion!**

🧠 **Mnemonic:** *"No dependency = free to reorder" — this is the root cause of ALL lock-free algorithm failures on modern hardware.*

---

## 6. Memory Barrier

**Definition:** When a **memory barrier** instruction is performed, the system ensures that **all loads and stores are completed before any subsequent load or store operations are performed.**

- Even if instructions were reordered by compiler/CPU, a memory barrier **forces completion and visibility** of all prior stores to memory before any future load/store proceeds.
- Memory barriers are considered **very low-level operations** and are typically **only used by kernel developers** writing specialized code to ensure mutual exclusion.

### Fixing the Reordering Example with Memory Barriers

```c
// Shared data
boolean flag = false;
int x = 0;

// Thread 1
while (!flag)
    memory_barrier();
print x;

// Thread 2
x = 100;
memory_barrier();
flag = true;
```

| Barrier | Guarantee |
|---|---|
| Thread 1's `memory_barrier()` | Ensures `flag` is loaded **before** `x` is loaded — no premature reads. |
| Thread 2's `memory_barrier()` | Ensures the assignment to `x` completes **before** the assignment to `flag` — no premature "signal". |

🧠 **Mnemonic:** *Memory Barrier = "finish everything behind me before you start anything ahead of me."*

---

## 7. Dekker's Algorithm

Another classic **2-process software solution**, historically the **first correct solution** to the CS problem (predates Peterson's). Uses both a `flag[]` array (intent) **and** a `turn` variable, but with more complex logic that *backs off* if it's not your turn (avoiding pure busy-deadlock).

### Code (as shown in lecture)

```c
void Dekkers_Algorithm(int i) {
    int j = !(i);
    while (1) {
        Non_CS();
        flag[i] = TRUE;
        while (flag[j] == TRUE) {
            if (turn == j) {
                flag[i] = FALSE;
                while (turn == j);   // busy wait until it becomes your turn
                flag[i] = TRUE;
            }
        }
        <CS>
        flag[i] = FALSE;
        turn = j;                   // hand over turn on exit
    }
}
```

### Cleaner textbook form

```c
// flag[] is boolean array; turn is an integer
flag[0] = false;
flag[1] = false;
turn = 0;   // or 1

// P0:
flag[0] = true;
while (flag[1] == true) {
    if (turn != 0) {
        flag[0] = false;
        while (turn != 0) {
            // busy wait
        }
        flag[0] = true;
    }
}
// critical section
turn = 1;
flag[0] = false;
// remainder section

// P1:  (symmetric, roles of 0 and 1 swapped)
flag[1] = true;
while (flag[0] == true) {
    if (turn != 1) {
        flag[1] = false;
        while (turn != 1) {
            // busy wait
        }
        flag[1] = true;
    }
}
// critical section
turn = 0;
flag[1] = false;
// remainder section
```

**Key idea:** If both processes want to enter (`flag[i]=flag[j]=true`), the one whose turn it ISN'T temporarily withdraws its interest (`flag[i]=false`) to let the other proceed, then re-asserts interest once it becomes its turn — avoiding a livelock where both keep flipping flags forever.

| Feature | Peterson's Solution | Dekker's Algorithm |
|---|---|---|
| Year | 1981 | Older (1965, first correct solution) |
| Complexity | Simpler | More complex (nested while + backoff) |
| Processes supported | 2 | 2 |
| Uses `flag[]` + `turn` | ✅ | ✅ |
| Mutual Exclusion / Progress / Bounded Waiting | ✅ all satisfied (on simple architecture) | ✅ all satisfied (on simple architecture) |
| Vulnerable to instruction reordering on modern HW | ✅ Yes | ✅ Yes (same root cause) |

---

## 📌 Quick Recap Table — All Software Solutions

| Solution | Mutual Exclusion | Progress | Bounded Waiting | Busy Waiting | Works for N processes? |
|---|---|---|---|---|---|
| Lock Variable | ❌ | — | — | ✅ | No |
| Strict Alternation | ✅ | ❌ | ✅ | ✅ | Only 2 |
| Peterson's Solution | ✅ (on simple arch.) | ✅ | ✅ | ✅ | Only 2 |
| Dekker's Algorithm | ✅ (on simple arch.) | ✅ | ✅ | ✅ | Only 2 |

---

## ✅ Review / Quick-Check Questions (from lecture)

**Q1. Why is disabling interrupts not a recommended synchronization technique on Multiprocessor systems?**
- A. It can't efficiently disable interrupts on all processors ✅ **(Correct)**
- B. It increases context switching — ❌ not the real reason; irrelevant to the multiprocessor issue.
- C. It violates mutual exclusion — ❌ disabling interrupts on a single CPU *does* achieve ME on that CPU; the issue is scaling to multiple CPUs.
- D. It causes immediate system crashes — ❌ overstated/false.

**Q2. Which Synchronization Technique may lead to Busy-Waiting?**
- A. Binary Semaphore — ❌ (blocking semaphores put processes to sleep, no spinning)
- B. Mutex using spinlock ✅ **(Correct)** — spinlocks force a process to loop continuously ("spin") while waiting for a lock, wasting CPU cycles.
- C. Bakery Algorithm — not chosen here (software solution can busy-wait too, but the lecture marks B as the intended answer for "spinlock" specifically).
- D. Peterson Solution — also busy-waits, but B is the direct/expected answer in this quick-check.

---

## 🎯 GATE PYQs

### PYQ 1 — I/O Device Mutual Exclusion using `Busy` flag

> Several concurrent processes are attempting to share an I/O device. In an attempt to achieve mutual exclusion, each process is given the following structure (`Busy` is a shared Boolean variable):
> ```
> <code unrelated to device use>     // Non-CS
> Repeat
>     until Busy = FALSE;             // Entry
>     Busy = TRUE;
> <code to access shared device>      // CS
> Busy = FALSE;                       // Exit
> <code unrelated to device use>
> ```
> Which of the following is/are true of this approach?
> I. It provides a reasonable solution to the problem of guaranteeing Mutual Exclusion.
> II. It may consume substantial CPU time accessing the Busy variable.
> III. It will fail to guarantee Mutual Exclusion.

- A. I only
- B. **II & III only ✅ (Correct)**
- C. III only
- D. I & II

**Reasoning:**
- **I is FALSE** — this is exactly the non-atomic "check-then-set" Lock Variable flaw (see Section 2): the check (`Repeat until Busy=FALSE`) and set (`Busy=TRUE`) are not atomic, so two processes can interleave and both enter CS.
- **II is TRUE** — the `Repeat...until` loop is a busy-wait loop, consuming CPU cycles while polling `Busy`.
- **III is TRUE** — follows directly from I being false: Mutual Exclusion CAN be violated due to non-atomic check+set.

---

### PYQ 2 — `critical_flag` routine (Get_Exclusive_Access)

> Processes P1 and P2 use `critical_flag` in the following routine to achieve mutual exclusion. Assume `critical_flag` is initialized to `FALSE`.
> ```c
> Get_Exclusive_Access() {
>     if (Critical_Flag == FALSE) {
>         Critical_Flag = TRUE;
>         Critical_Region();
>         Critical_Flag = FALSE;
>     }
> }
> ```
> Consider the statements:
> (i) It is possible for both P1 and P2 to access `critical_region` concurrently.
> (ii) This may lead to a Deadlock.

- A. (i) is false and (ii) is true
- B. Both (i) and (ii) are false
- C. **(i) is true and (ii) is false ✅ (Correct)**
- D. Both (i) and (ii) are true

**Reasoning:**
- **(i) is TRUE** — the check (`if Critical_Flag==FALSE`) and the set (`Critical_Flag=TRUE`) are two separate, non-atomic steps; a context switch between them lets both P1 and P2 pass the check and enter `Critical_Region()` together → Mutual Exclusion violated.
- **(ii) is FALSE** — there's no scenario of mutual/circular waiting here (no process holds a resource while waiting for another it can never get) — this is a Mutual Exclusion failure, **not** a Deadlock. A is wrong because (i) is actually true, not false. B and D are wrong for the same two reasons above.

---

### PYQ 3 — Strict Alternation Two-Process Solution

> ```
> Process 0                          Process 1
> Entry: loop while (turn == 1);     Entry: loop while (turn == 0);
> (critical section)                 (critical section)
> Exit: turn = 1;                    Exit: turn = 0;
> ```
> The shared variable `turn` is initialized to zero. Which one of the following is TRUE?

- A. This is a correct two-process synchronization solution.
- B. This solution violates mutual exclusion requirement.
- C. **This solution violates progress requirement. ✅ (Correct)**
- D. This solution violates bounded wait requirement.

**Reasoning:** This is the classic **100% Strict Alternation** scheme (Section 3). It **guarantees** Mutual Exclusion (B wrong) and Bounded Waiting (D wrong) since access strictly alternates — but it **fails Progress**: if Process 0 is not interested in re-entering CS but Process 1 wants to enter again, Process 1 must still wait for `turn` to flip to 1 by Process 0, even though Process 0 isn't using/requesting the CS. A is wrong since it's not fully correct.

---

### PYQ 4 — S1/S2 Shared Boolean Method (P1 vs P2)

> ```
> Method used by P1            Method used by P2
> while (S1==S2);              while (S1!=S2);
> Critical Section              Critical Section
> S1 = S2;                      S2 = not(S1);
> ```
> Initial values of shared Booleans S1 and S2 are randomly assigned. Which one of the following statements describes the properties achieved?

- A. **Mutual exclusion but not Progress ✅ (Correct)**
- B. Progress but not Mutual Exclusion
- C. Neither Mutual Exclusion nor Progress
- D. Both Mutual Exclusion and Progress

**Reasoning (trace):**
- If `S1 == S2` (both T or both F): P1's `while(S1==S2)` blocks → only **P2** can enter CS.
- If `S1 != S2`: P2's `while(S1!=S2)` blocks → only **P1** can enter CS.
- So **exactly one process** can pass at any time → **Mutual Exclusion holds ✅**.
- But **Progress fails ❌**: Trace — start `S1=T, S2=T` → P2 enters CS → sets `S2 = not(S1) = F` → now `S1=T, S2=F` → P1 enters CS → sets `S1 = S2 = F` → now `S1=F, S2=F` (both equal again) → **only P2 can proceed now, even if P1 wants to enter again and P2 doesn't** → a process can be forced to wait even when the other isn't interested → Progress violated.
- B, C, D are wrong because Mutual Exclusion **does** hold; only Progress fails.

---

### PYQ 5 — `wants1`/`wants2` Solution

> ```c
> /* P1 */                          /* P2 */
> while (TRUE) {                    while (TRUE) {
>   wants1 = TRUE;                    wants2 = TRUE;
>   while (wants2 == TRUE);           while (wants1 == TRUE);
>   /* Critical Section */            /* Critical Section */
>   wants1 = FALSE;                   wants2 = FALSE;
> }                                  }
> /* Remainder section */            /* Remainder section */
> ```

- A. It does not ensure Mutual Exclusion.
- B. It does not ensure Bounded Waiting.
- C. It requires that processes enter the critical section in Strict Alternation.
- D. It does not prevent Deadlocks, but ensures Mutual Exclusion.

**Reasoning:** This is a **naive "both raise flag first" attempt** with **no turn/tiebreaker variable**. If both processes execute `wants1=TRUE` and `wants2=TRUE` almost simultaneously, **both** `while` conditions become true → **both processes wait forever** → **classic Deadlock**, but crucially **Mutual Exclusion is never actually violated** (no one enters CS at all in that case) — matches **option D**: it fails to prevent deadlock, but whenever a process *does* enter, mutual exclusion is intact. (A is wrong — ME does hold; B is a secondary effect not the primary flaw; C is wrong — there's no explicit alternation variable like `turn`, so it's not "strict alternation".)

*(Note: this question's answer key context favors D based on the "lock without tie-breaker → deadlock, not ME-violation" pattern.)*

---

### PYQ 6 — `varP`/`varQ` Process X and Y

> ```c
> // Process X                         // Process Y
> while (true) {                       while (true) {
>   varP = true;                         varQ = true;
>   while (varQ == true) {               while (varP == true) {
>     /* critical section */               /* critical section */
>     varP = false;                         varQ = false;
>   }                                      }
> }                                       }
> ```
> `varP`, `varQ` are shared, both initialized to `false`. Which statement is TRUE?

- A. The proposed solution prevents deadlock but fails to guarantee Mutual Exclusion.
- B. The proposed solution guarantees Mutual Exclusion but fails to prevent Deadlock.
- C. The proposed solution guarantees Mutual Exclusion and prevents Deadlock.
- D. The proposed solution fails to prevent Deadlock and fails to guarantee Mutual Exclusion.

**Reasoning:** This is structurally identical to the **Lock Variable flaw pattern**: `varP = true` then immediately check `varQ` — no atomic combination, and no tiebreaker/`turn` variable. If both X and Y set their own flag (`varP=true`, `varQ=true`) before checking the other's flag, **both** end up looping forever waiting on the other's flag → **Deadlock is possible**. Since both never actually enter the CS simultaneously in the deadlock case, and outside of deadlock only one can pass at a time, **Mutual Exclusion still holds** → matches option **B** (fails to prevent deadlock, but guarantees ME). *(A and C wrongly claim deadlock is prevented; D wrongly claims ME fails.)*

---

### PYQ 7 — `Fetch_And_Add` Busy-Wait Lock

> `Fetch_And_Add(X, i)` is an atomic Read-Modify-Write instruction: reads memory location `X`, increments it by value `i`, and returns the **old** value of `X`. `L` is shared unsigned integer initialized to `0` (0 = lock available, non-zero = lock unavailable).
> ```c
> AcquireLock(L) {
>     while (Fetch_And_Add(L, 1))
>         L = 1;
> }
> ReleaseLock(L) {
>     L = 0;
> }
> ```
> This implementation:

- A. Fails as L can overflow
- B. Fails as L can take on a non-zero value when the lock is actually available
- C. **Works correctly but may starve some processes ✅ (Correct)**
- D. Works correctly without starvation

**Reasoning:** Every call to `Fetch_And_Add(L,1)` **atomically** increments `L` and returns the old value — so exactly one process gets the old value `0` (lock was free) and enters; all others get a non-zero old value and loop (setting `L=1` each failed attempt keeps it locked/non-zero so mutual exclusion holds). This correctly provides mutual exclusion (rules out A, B) — but since there's **no ordering/queueing mechanism** (no FIFO, no ticket order), a process could **repeatedly lose the race** to newer arrivals → **starvation is possible**, so it "works correctly" (ME holds) **but may starve** some processes → **C** is correct, **D** is wrong because starvation isn't prevented.

---

### PYQ 8 — `test-and-set` based `enter_CS`/`leave_CS`

> ```c
> void enter_CS(X) {
>     while (test-and-set(X));
> }
> void leave_CS(X) {
>     X = 0;
> }
> ```
> `X` is a memory location initialized to 0. Consider:
> I. The above solution to CS problem is deadlock-free.
> II. The solution is starvation-free.
> III. The processes enter CS in FIFO order.
> IV. More than one process can enter CS at the same time.

- A. **I only ✅ (Correct)**
- B. I and II
- C. II and III
- D. IV only

**Reasoning:** `test-and-set(X)` is an **atomic hardware instruction** — it sets `X` to 1 and returns the old value in one indivisible step, so it correctly guarantees Mutual Exclusion (rules out IV) and is **Deadlock-free** — someone always successfully acquires the lock eventually when it's released (**I is TRUE**). However, it provides **no fairness guarantee**:
- **II is FALSE** — a process could keep losing the "race" to test-and-set repeatedly if unlucky with scheduling → **starvation possible**.
- **III is FALSE** — there's no queue/ticket mechanism, so entry order is **not FIFO**; whichever process's `test-and-set` executes first (by chance/scheduler) wins, regardless of request order.
- **IV is FALSE** — atomicity of test-and-set strictly prevents more than one process succeeding at once.

---

## 🧠 Memory Tricks / Mnemonics Recap

- **Lock Variable failure:** "Check-then-act" without atomicity always breaks mutual exclusion.
- **Strict Alternation:** 100% turn-based → guarantees ME + Bounded Wait, but breaks Progress (can't skip an uninterested opponent's turn).
- **Peterson's Solution:** "I'll go first, but I'll let YOU go first" (`turn = j`) — software courtesy guaranteeing fairness.
- **Reordering bug:** "No dependency = free to reorder" — root cause of all lock-free algorithm failures on modern hardware.
- **Memory Barrier:** "Finish everything behind me before you start anything ahead of me."
- **Test-and-set / Fetch-and-Add:** Atomic HW instructions fix Mutual Exclusion but never automatically fix **starvation** or **FIFO ordering** — those need extra mechanisms (e.g., ticket/queue-based locks).

---

## ✅ Quick Revision Checklist

- [ ] State the 3 requirements of the CS problem (Mutual Exclusion, Progress, Bounded Waiting) — and know Deadlock-freedom/Preemption are **not** official requirements.
- [ ] Explain why the **Lock Variable** solution fails Mutual Exclusion (non-atomic check+set).
- [ ] Explain **Strict Alternation**: satisfies ME + Bounded Wait, fails Progress — and WHY (forced turn-taking).
- [ ] Write **Peterson's Solution** code from memory (`flag[]` + `turn`), and explain how each of the 3 properties is satisfied.
- [ ] Know the 3 different textbook presentations of Peterson's Solution (flag/turn, interested[], waiting[]) — they're all the same idea.
- [ ] Understand why **Peterson's Solution may fail on modern architectures** due to **instruction reordering** by compiler/CPU (no data dependency between `turn=i` and `flag[i]=true`).
- [ ] Explain the **Memory Barrier** fix — forces completion/visibility of stores before subsequent loads/stores.
- [ ] Recall **Dekker's Algorithm** structure — `flag[]` + `turn`, with a "back off if not your turn" nested loop to avoid livelock.
- [ ] Be able to spot the **"check-then-set without atomicity"** pattern in any pseudocode (classic GATE trap: `Busy`, `Critical_Flag`, `varP/varQ`, `wants1/wants2` — all same underlying flaw family).
- [ ] Distinguish **Mutual Exclusion violation** vs **Deadlock** vs **Starvation** vs **Progress violation** vs **Bounded Waiting violation** — GATE loves mixing these up in options.
- [ ] Know that **atomic hardware instructions** (`test-and-set`, `Fetch_And_Add`) guarantee Mutual Exclusion + Deadlock-freedom by default, but do **NOT** guarantee starvation-freedom or FIFO order unless explicitly designed for it.
- [ ] Practice tracing shared-variable interleavings (S1/S2, wants1/wants2, varP/varQ style questions) step by step — this is the most common GATE question pattern for this topic.

---
*Notes compiled from GeeksforGeeks GATE CS & IT — Process Synchronization, Lecture 14.*
