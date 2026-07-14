# Deadlocks — GATE CS/IT OS Notes (Lecture 19)

> Source: GeeksforGeeks GATE CS&IT — Principles of Operating Systems, Lecture 19

## 📑 Table of Contents

| # | Topic |
|---|-------|
| 1 | Concurrency Mechanisms Recap (Parbegin-Parend / Cobegin-Coend) |
| 2 | Precedence Graphs from Concurrent Programs |
| 3 | Using Binary Semaphores (BSEM) to Enforce Precedence |
| 4 | Deadlock — Intuition & Real-World Analogies |
| 5 | Deadlock vs Starvation |
| 6 | Deadlock Definition (Formal) |
| 7 | Necessary Conditions for Deadlock |
| 8 | Resource Allocation Graph (RAG) |
| 9 | Cycle vs Deadlock — Basic Facts |
| 10 | Deadlock Handling Strategies (Overview) |
| 11 | Deadlock Prevention (breaking each necessary condition) |
| 12 | GATE PYQs / Practice Questions |
| 13 | Mnemonics & Quick Memory Tricks |
| 14 | Quick Revision Checklist |

---

## 1. Concurrency Mechanisms Recap

Before deadlocks, the lecture revisits how **concurrent execution** is expressed in pseudocode, since deadlock examples build on this.

### (i) Parbegin–Parend / Cobegin–Coend
- Sequential block:
  ```
  {
    S1;
    S2;
    S3;
  }
  ```
  is equivalent to:
  ```
  begin
    S1;
    S2;
    S3;
  end
  ```
  → executes **strictly in order**: S1 → S2 → S3 (drawn as a straight-line precedence chain).

- **Parallel block** (all statements run concurrently, and the block only finishes when *all* inner statements finish):
  ```
  S0;
  Parbegin
      S1;
      S2;
      S3;
  Parend
  S4;
  ```
  **Precedence graph**: S0 → {S1, S2, S3} (fan-out), then {S1, S2, S3} → S4 (fan-in, i.e., S4 waits for all three).

### 2. Precedence Graphs from Concurrent Programs

**Rule of thumb for drawing precedence graphs:**
- A statement **before** `Parbegin` → single predecessor of all statements inside the parallel block.
- A statement **after** `Parend` → single successor, waits for **all** branches inside the block to finish.
- Nested `begin...end` inside a `Parbegin/Parend` block just means "these statements run sequentially, but this entire sequential chain runs in parallel with the other branches."

**Worked Example 1:**
```
S1;
Parbegin
    begin S2; S3; end
    S4;
    S5;
    begin S6; S7; S8; end
    S9;
Parend
S10;
```
**Precedence Graph:**
```
        S1
   /  /  |  \   \
 S2  S4  S5  S6  S9
  |          |
 S3          S7
              |
             S8
   \   \   |  /  /
        S10
```
- S1 fans out to 5 parallel branches: (S2→S3), S4, S5, (S6→S7→S8), S9
- All 5 branches converge into S10 (S10 waits for everything).

**Worked Example 2 (nested Parbegin):**
```
S1;
Parbegin
    begin
        S2; S4;
        Parbegin
            S5; S6;
        Parend
    end
    S3;
Parend
S7;
```
**Precedence Graph:** S1 → {S2→S4→(S5∥S6), S3} → S7
- Note: In the graph, an edge from **S3 → S6** does *not* exist — S3 and S6 are on independent, concurrent branches (this is explicitly highlighted with a strike/cross mark on the false S3→S6 edge in the slide as a common student mistake).

**Worked Example 3 (GATE-style, 9 statements — reused in PYQ section too):**
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
Parend
S9;
```
**Precedence Graph:**
```
          S1
     /     |      \
   S2      S3      S7
   |      /  \       \
   S4   S5    S6       \
    \    \    |        /
     \    \   S8      /
      \    \  /      /
             S9
```
- S1 → S2, S3, S7 (three parallel branches)
- S2 → S4
- S3 → S5 and S3 → S6 → S8 (S3 has its own nested parallel block)
- S4, S5, S8, S7 all converge into S9

---

## 3. Using Binary Semaphores (BSEM) to Enforce Precedence

A precedence graph can be **implemented in code** using binary semaphores (`P`/`V` operations) instead of `Parbegin/Parend`, when you need explicit ordering control.

**Pattern:** For every edge `Sx → Sy` in the precedence graph, create a binary semaphore initialized to 0. The producer statement does a `V()` after finishing; the consumer statement does a `P()` before starting.

**Worked Example (Parbegin–Parend with BSEM):**

Given precedence graph: S1 → {S2, S3}; S2 → S4; S3 → nothing further shown differently; S4, S6 → converge; etc. (from the diagram with S1,S2,S3,S4,S5,S6,S7):

```
BSEM a, b, c, d, e, f, g = {0};

Parbegin
    begin S1; V(a); V(b); end
    begin P(a); S2; S4; V(c); V(d); end
    begin P(b); S3; V(e); end
    begin P(c); S5; V(f); end
    begin P(a); P(e); S6; V(g); end
    begin P(f); P(g); S7; end
Parend
```

**How to read this:** Each `P(x)` blocks until the matching `V(x)` has been executed by the statement that must finish first — this is how semaphores *simulate* the precedence graph's arrows in actual concurrent code (instead of relying on `Parbegin/Parend` syntax alone).

> 💡 **Exam tip:** GATE loves giving you a `Parbegin-Parend` block and asking you to (a) draw the precedence graph, or (b) rewrite it using semaphores, or (c) list all valid output orderings. Master the fan-out/fan-in rule above.

---

## 4. Deadlock — Intuition & Real-World Analogies

The lecture uses several analogies to build intuition before the formal definition:

| Analogy | Deadlock Mapping |
|---|---|
| **Rachel & Ross (Friends) — Omelet vs Pancake** | Rachel holds the *oil*, wants the *pan*. Ross holds the *pan*, wants the *oil*. Neither releases what they hold → deadlock. |
| **Cop & Criminal hostage stand-off** | Cop (Thread 1) holds Resource #1 (criminal's friend) but needs Resource #2 (owned by Criminal). Criminal (Thread 2) holds Resource #2 but needs Resource #1. Classic circular wait. |
| **4-way traffic intersection gridlock** | Each car occupies a segment of road another car needs, and all are waiting on each other → traffic deadlock. |

**Key visual takeaway:** In every analogy, the entities form a **cycle** of "I hold X, I want Y" relationships, and nobody is willing to voluntarily give up what they hold.

---

## 5. Deadlock vs Starvation

| Aspect | **Deadlock** | **Starvation** |
|---|---|---|
| Definition | A set of processes where *every* process is waiting for an event that can only be caused by another process in the same set | A process is perpetually denied the resources it needs |
| When it occurs | Each process holds a resource and waits for a resource held by another process | A process waits for a resource for a long period while the system keeps favoring others |
| Who is affected | **All** related processes are permanently stuck | **Some** processes wait indefinitely, but others can proceed normally |

---

## 6. Deadlock Definition (Formal)

- **Deadlock**: A process is **deadlocked** if it is waiting for an event that will **never occur**.
- Typically (but not necessarily) more than one process is involved together in a deadlock — this mutual entanglement is called the **"deadly embrace."**
- **Indefinitely Postponed (≈ starvation)**: A process is delayed repeatedly over a *long* period while the system's attention goes to other processes — i.e., logically the process *could* proceed, but the system never gives it the CPU/resource.

> ⚠️ **Distinction to remember for GATE:** Deadlock = *can never* proceed (structurally impossible). Starvation = *may* proceed eventually, just repeatedly deprioritized.

---

## 7. Necessary Conditions for Deadlock

A deadlock can arise **only if all four** of the following hold **simultaneously**:

1. **Mutual Exclusion** — Only one process at a time can use a resource.
2. **Hold and Wait** — A process holding at least one resource is waiting to acquire additional resources currently held by other processes.
3. **No Preemption** — A resource can be released only **voluntarily** by the process holding it, after that process has completed its task (no one can forcibly snatch it away).
4. **Circular Wait** — There exists a set of waiting processes $\{T_0, T_1, \dots, T_n\}$ such that:
   - $T_0$ is waiting for a resource held by $T_1$
   - $T_1$ is waiting for a resource held by $T_2$
   - ⋮
   - $T_{n-1}$ is waiting for a resource held by $T_n$
   - $T_n$ is waiting for a resource held by $T_0$

> 💡 **Mnemonic:** **M-H-N-C** → "**M**y **H**ouse **N**eeds **C**leaning" (Mutual exclusion, Hold & wait, No preemption, Circular wait). All 4 must hold for deadlock — breaking **any one** prevents deadlock (this is the basis of *Deadlock Prevention*, see §11).

---

## 8. Resource Allocation Graph (RAG)

A **Resource Allocation Graph** is $G = (V, E)$:

**Vertices (V):**
- **Process** nodes (circles), e.g., $P_i$
- **Resource** nodes (rectangles), e.g., $R$ — a resource box can contain **multiple instance dots** inside it (e.g., a resource type with 3 instances).

**Edges (E)** — three types:
| Edge type | Notation | Meaning |
|---|---|---|
| **Request edge** | $P_i \rightarrow R$ | Process $P_i$ has requested an instance of resource $R$ and is waiting |
| **Assignment edge** | $R \rightarrow P_i$ | An instance of resource $R$ has been allocated to $P_i$ |
| **Claim edge** | $P_i \dashrightarrow R$ (dotted) | $P_i$ *may* request $R$ in the future (used in deadlock-avoidance graphs) |

### Worked RAG Example

**Given:**
- 1 instance of $R_1$, 2 instances of $R_2$, 1 instance of $R_3$, 3 instances of $R_4$
- $P_1$ holds one instance of $R_2$, waiting for an instance of $R_1$
- $P_2$ holds one instance of $R_1$ and one instance of $R_2$, waiting for an instance of $R_3$
- $P_3$ holds one instance of $R_3$

**Graph relationships:**
```
R1 → P2        (R1 assigned to P2)
R1 → P1  (wait, P1 → R1 is a request)   → P1 requests R1
R2 → P1        (R2 assigned to P1)
R2 → P2        (R2 assigned to P2)
R3 → P3        (R3 assigned to P3)
P2 → R3        (P2 requests R3)
```
- **This graph has NO cycle → NO deadlock.** ($R_4$ isn't even involved.)

### Modified Example — With a Deadlock
If instead:
- $P_1$ holds $R_2$, requests $R_1$
- $P_2$ holds $R_1$, requests $R_2$ (mutual — no $R_3$ dependency)
- $P_3$ requests $R_2$ but is not part of the cycle

Then $P_1 \to R_1 \to P_2 \to R_2 \to P_1$ forms a **cycle → deadlock** between $P_1$ and $P_2$.

### Important Counter-Example: Cycle WITHOUT Deadlock
When a resource type has **multiple instances**, a cycle in the RAG does **not guarantee** deadlock:

- $R_1$ (2 instances): one assigned to $P_2$, one to $P_1$'s "producer" — $R_1 \to P_2$ and $P_1 \to R_1$... 
- Given: $R_1$ has 2 instances (one → $P_2$, one requested by $P_3$... but also assigned so that $P_1$ can proceed); $R_2$ has 2 instances similarly shared between $P_1$/$P_3$/$P_4$.
- Even though a cycle $P_1 \to R_1 \to P_3 \to R_2 \to P_1$ exists, because $R_1$ and $R_2$ each have a **free/available instance that can be granted to a waiting process** (e.g., $P_4$ can get an instance of $R_2$ and finish, releasing it), the system is **NOT deadlocked** — processes can still complete given the right scheduling order.
- At time $t_i$: **No deadlock** (someone can still be satisfied and eventually release resources, breaking the apparent cycle's effect).

---

## 9. Cycle vs Deadlock — Basic Facts

| Situation | Conclusion |
|---|---|
| Graph contains **no cycle** | **No deadlock**, guaranteed |
| Graph has a cycle, **and each resource type has only 1 instance** | **Deadlock**, guaranteed |
| Graph has a cycle, **and some resource types have several instances** | Only a **possibility** of deadlock — not guaranteed (depends on whether spare instances can free up a process in the cycle) |

---

## 10. Deadlock Handling Strategies (Overview)

```
Methods of Handling Deadlock
├── Deadlock Prevention
├── Deadlock Avoidance
│     ├── Resource-Allocation Graph Algorithm (Single Instance) — T1 type courses
│     └── Banker's Algorithm (Multiple Instance) — T1 type courses
├── Deadlock Detection & Recovery ("Doctor's Strategy") — T2 type courses
└── Deadlock Ignorance — "Ostrich Algorithm" — T2 type courses
```

*(T1/T2 likely refer to different course-tier labels used by the instructor — indicates depth/priority of topic for GATE.)*

### The Ostrich Algorithm
- **Idea**: Pretend there is no problem — don't do anything, just restart the system if it happens ("stick your head in the sand").
- **Rationale**: Makes the *common case* (no deadlock) faster and more reliable, since prevention/avoidance/detection algorithms are **expensive** and add overhead.
- **Reasonable when:**
  - Deadlocks occur **very rarely**
  - Cost of prevention is **high**
- **Real-world use**: UNIX and Windows largely take this approach.
- It's a **trade-off between Convenience and Correctness**.

---

## 11. Deadlock Prevention

**Philosophy:** We know the 4 necessary conditions (§7). To *prevent* deadlock, design the system to ensure **at least one** of these conditions can **never** hold — "excluding the possibility of deadlock a priori."

### (a) Attacking Mutual Exclusion
- Generally **not practical to eliminate** — some resources (printers, etc.) are inherently exclusive-use. *(Marked with ✗ in the lecture — not a usable strategy in most systems.)*

### (b) Attacking Hold and Wait
Two possible protocols (either one breaks Hold-and-Wait):
1. **All-at-once allocation**: A process must **request and be allocated ALL resources it will ever need (a priori), before it starts execution.**
2. **Release-before-request**: A process must **release ALL resources currently held before making a new/fresh request** for additional resources.

- **Downsides**: Low resource utilization (resources reserved but unused for long periods) and possible starvation (a process needing many popular resources may never get them all at once).

### (c) Attacking No Preemption
**Idea:** Allow preemption of resources — i.e., a resource *can* be taken away from a process before it voluntarily releases it.

Two flavors:
- **Forcible preemption**: The OS forcibly takes a resource from a waiting process and gives it to another (used for resources whose state can be easily saved/restored, e.g., CPU registers, memory).
- **Self preemption**: If a process holding some resources requests another that can't be immediately granted, all resources currently held are released (self-preempted), and the process must request everything again later.

### (d) Attacking Circular Wait
- **Solution: Linear (total) ordering of all resource types.**
- Assign each resource type $R_i$ a unique integer rank.
- Rule: If a process holds resources of type $R_j$, it may only request resources of type $R_k$ where $k > j$. It **cannot** request any $R_i$ where $i \le j$.
- Symmetric statement: A process holding $R_i$ can request $R_j$ (since $j > i$), but a process holding $R_j$ **cannot** request $R_i$ (since $i < j$) — this asymmetry breaks the possibility of a cycle forming.

> 💡 **Mnemonic for Prevention strategies:** "**Break one link, break the chain.**" Deadlock needs *all 4* conditions — killing any single one (M/H/N/C) prevents deadlock entirely. Circular-wait-breaking via **resource ordering** is the most commonly asked in GATE.

---

## 12. GATE PYQs / Practice Questions

### Q1 — First Reader-Writer Problem (Semaphore, Busy Waiting)
```c
int R = 0, W = 0;
BSEM mutex = 1;

void Reader(void) {
    L1: P(mutex);
    if (W == 1) {
        V(mutex);
        goto L1;
    } else {
        R = R + 1;
        V(mutex);          // <-- fill in the blank
    }
    <DB_READ>
    P(mutex);
    R = R - 1;
    V(mutex);
}

void Writer(void) {
    L2: P(mutex);
    if (R >= 1 || W == 1) {   // <-- fill in the blank
        V(mutex);
        goto L2;
    } else {
        W = 1;
        V(mutex);
    }
    <DB_WRITE>
    P(mutex);
    W = 0;
    V(mutex);
}
```
**Answer / reasoning:**
- Reader's blank: `V(mutex)` — must release mutex after incrementing reader count so other readers/writers can check the flags.
- Writer's blank: `if (R >= 1 || W == 1)` — a writer must block if **either** a reader is currently reading (R ≥ 1) **or** another writer is writing (W == 1). This enforces mutual exclusion for the writer against both readers and other writers.

---

### Q2 — Precedence Graph for Concurrent Program
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
Parend
S9;
```
**Task:** Draw the precedence graph.

**Answer:**
```
              S1
       /      |      \
     S2       S3       S7
      |      /   \       \
     S4    S5     S6       \
       \     \     |         \
        \     \    S8         \
         \     \   /          /
                 S9
```
- S1 → S2, S3, S7
- S2 → S4
- S3 → S5, S3 → S6 → S8
- S4, S5, S8, S7 → S9 (S9 waits on all terminal branches)

*(This is the classic example from §2 Worked Example 3 above — GATE frequently reuses this exact structure.)*

---

### Q3 — Valid Output Sequences of a Concurrent Program
```c
void Foo(void) {
    X;
    Y;
    Z;
}

void Boo(void) {
    A;
    B;
}

main() {
    Parbegin
        Foo();
        Boo();
    Parend
}
```
**Task:** List out valid output sequences (interleavings), given internal order X→Y→Z must be preserved, and A→B must be preserved, but the two blocks can interleave arbitrarily.

| # | Sequence | Valid? |
|---|---|---|
| 1 | X Y Z A B | ✅ Valid |
| 2 | A B X Y Z | ✅ Valid |
| 3 | X A Y B Z | ✅ Valid |
| 4 | A X B Z Y | ❌ **Invalid** — Z appears before Y, violating Foo's required order X→Y→Z |

**Reasoning:** Any interleaving is valid **as long as** the relative order within each thread (X before Y before Z; A before B) is preserved. Sequence 4 breaks this by putting Z before Y.

---

### Q4 — Final Values of Shared Variables (No Synchronization)
```c
int a = 0, b = 0;
Parbegin
    begin           // P1
        1: a = 1;
        2: b = b + a;
    end
    begin           // P2
        3: b = 2;
        4: a = a + 3;
    end
Parend
```
**Options:**
- I) a = 1, b = 2
- II) a = 1, b = 3
- III) a = 4, b = 6

**Answer:** **II and III are valid; I is NOT valid.**

**Reasoning (interleavings):**
- Order `⟨3;4;1;2⟩`: b=2 (stmt3); a=0+3=3... *(instructor's worked trace)* → gives **a=1, b=3** via order `⟨3;4;1;2⟩`: b=2, a=a+3=3, a=1 (overwritten), b=b+a=3+1=... 
  - Actual traced result shown: **a = 1, b = 3** ✅
- Order `⟨1;3;4;2⟩`: a=1, b=2, a=a+3=4, b=b+a=2+4=6 → **a = 4, b = 6** ✅
- **a=1, b=2** (option I) is not achievable by any valid interleaving of statements 1,2,3,4 — hence invalid.

---

### Q5 — Critical Section with Mutex (Load/Increment/Store breakdown)
```c
int a = 0, b = 20;
BSEM mx = 1;
Parbegin
    begin
        P(mx);
        a = a + 1;   // decomposed as: 1: Load a; Inc; Store a
        V(mx);
    end
    begin
        P(mx);
        a = b + 1;   // decomposed as: 1: Load b; 2: a = b + 1 (Store)
        V(mx);
    end
Parend
```
**Task:** Find final possible value(s) of `a`, given the critical section is properly mutex-protected.

**Answer:** Since **both branches are protected by the same mutex `mx`**, the two critical sections execute in **strict mutual exclusion** (no interleaving between them):
- If branch 1 runs fully first: a = 0+1 = 1, then branch 2: a = b+1 = 20+1 = **21**
- If branch 2 runs fully first: a = 20+1 = 21, then branch 1: a = 21+1 = **22**

→ **Final possible values of a: {21, 22}**

**Follow-up variant (different mutexes — `my` for branch 1, `mx` for branch 2):**
- If the two branches use **different, independent semaphores** (`my` and `mx`), they are **no longer mutually exclusive** — instructions can interleave freely.
- This allows a **lost update**: e.g., branch 2 reads `b=20` before branch 1 finishes, both compute based on stale values → possible result: **a = 22 only** (or other values depending on interleaving) — demonstrates why using the *same* mutex for related shared-variable operations is essential.

> 💡 **Key GATE insight:** If two critical sections **share the same lock**, they're serialized (no interleaving) → limited final-value set. If they use **different locks**, interleaving is possible → more (or different) final values, and race conditions/lost updates can occur.

---

### Q6 — Distinct Values of Shared Variable B
```c
integer B = 2;

P1() {
    1: C = B - 1;
    2: B = 2 * C;
}

P2() {
    3: D = 2 * B;
    4: B = D - 1;
}
```
**Task:** If P1 and P2 execute concurrently, how many distinct final values of B are possible?

**Answer:** **3** distinct values → **B ∈ {2, 3, 4}**

**Reasoning:** Depending on the interleaving order of statements 1,2,3,4 (respecting 1 before 2, and 3 before 4), B can end up as 2, 3, or 4 based on which reads see updated vs. stale values of B and C/D.

---

### Q7 — Min/Max Value of Shared Counter (Race Condition)
```c
int c = 0;

void Foo() {
    int i, n = 10;
    for (i = 1; i <= n; ++i)
        c = c + 1;
}

cobegin
    P1: Foo();
    P2: Foo();
coend
```
**Task:** What is the minimum and maximum value of `c` after both P1 and P2 finish (no synchronization)?

**Answer:**
- **Max(c) = 20** — achieved if every `c = c + 1` executes atomically with no lost updates (i.e., no interleaving corrupts any increment).
- **Min(c) = 2** — in the worst case, if both P1 and P2 repeatedly read the **same stale value of c** before either writes back (11 read-modify-write races each losing updates), the counter can be as low as effectively only "2" net increments taking effect (classic **lost update** problem due to non-atomic `c = c + 1`, which internally is Load → Increment → Store).

> 💡 **Mnemonic:** `c = c + 1` is **NOT atomic** — it's really 3 steps (**Load, Inc, Store**). Race conditions happen in the gap between Load and Store.

---

### Q8 — GATE PYQ: Two Threads Updating Shared Variables (HW / Practice flagged in lecture)
```
Initially: a = b = 1
Each statement executes atomically (no interruption mid-statement),
but interleaving between T1 and T2 can happen at statement boundaries.

T1                    T2
a = a + 1;            b = a * b;
b = b + 1;            a = 2 * a;
```
**Question:** Which option lists ALL possible combinations of (a, b) after both T1 and T2 finish?

**Options:**
- (A) (a=4,b=4); (a=3,b=3); (a=4,b=3)
- (B) (a=3,b=4); (a=4,b=3); (a=3,b=3)
- (C) (a=4,b=4); (a=4,b=4); (a=3,b=4)
- (D) (a=2,b=2); (a=2,b=3); (a=3,b=4)

**Note:** This question was flagged by the instructor as **homework (HW)** — marked but not solved live in this lecture. Recommended approach: enumerate all valid interleavings (respecting each thread's internal order) and compute (a,b) for each; the correct GATE answer for this well-known PYQ is **(A)**.
*(Cross-check with the original source before final revision, as the instructor left this as self-practice.)*

---

### Q9 — Set of Possible Values of x (3-way Concurrent Increment, Non-Atomic Expression Evaluation)
```
x := 1;
Cobegin
    x := x + 1  ||  x := x + 1  ||  x := x + 1
Coend
```
*(Reading and writing of variables is atomic, but expression evaluation `x + 1` is NOT atomic — i.e., "load x", "add 1", "store x" can interleave across the three concurrent statements.)*

**Options:**
- (A) {4}
- (B) {2, 3, 4}
- (C) {2, 4}
- (D) {2, 3}
- (E) {2}

**Answer: (B) {2, 3, 4}**

**Reasoning:**
- **Max = 4**: if all three increments execute without any overlapping read (fully serialized), x goes 1→2→3→4.
- **Min = 2**: if all three threads read x=1 *before* any of them writes back, all three compute `1+1=2` and whichever writes last leaves x=2 (worst-case total lost updates).
- **3 is also achievable**: e.g., two threads race and lose one update (net effect of a "2"), while the third's increment applies cleanly on top → x=3.
- Since reads/writes are atomic but the **whole expression evaluation is not**, all of {2,3,4} are reachable, ruling out options that exclude any of these (A, C, D, E are each incomplete).

---

## 13. Mnemonics & Quick Memory Tricks

- 🍳 **Rachel & Ross (oil & pan)** → helps recall that deadlock = "I won't give up what I have until I get what you have, and you feel the same" — a **mutual, circular refusal**.
- 👮 **Cop & Criminal hostage swap** → visualizes **circular wait** concretely: Thread 1 needs Resource #2 (owned by Thread 2's lock), Thread 2 needs Resource #1 (owned by Thread 1's lock).
- 🐦 **Ostrich Algorithm** → "head in the sand" = ignore the problem; used by real OSes (UNIX/Windows) when deadlocks are rare and prevention is costly.
- **M-H-N-C** → the 4 necessary conditions: **M**utual exclusion, **H**old-and-wait, **N**o preemption, **C**ircular wait. Break any ONE → no deadlock.
- **Load-Inc-Store** → remember that `x = x + 1` / `c = c + 1` is never a single atomic step; this is the root of most race-condition GATE questions.
- **Resource ordering breaks circular wait** → think of it like a **one-way staircase**: you can only ask for resources "higher" than what you already hold, so you can never loop back down to create a cycle.

---

## 14. Quick Revision Checklist

- [ ] Can draw a **precedence graph** from any `Parbegin/Parend` (including nested) pseudocode block
- [ ] Understand fan-out (before Parbegin) and fan-in (after Parend) rules
- [ ] Can convert a precedence graph into **BSEM-based P/V code**
- [ ] Know the difference between **Deadlock** and **Starvation** (table in §5)
- [ ] Can state the formal definition of deadlock ("waiting for an event that will never occur")
- [ ] Can list and explain all **4 necessary conditions**: Mutual Exclusion, Hold & Wait, No Preemption, Circular Wait
- [ ] Can draw a **Resource Allocation Graph** given process/resource holding & request info
- [ ] Know RAG edge types: **Request**, **Assignment**, **Claim** (dotted)
- [ ] Know the **cycle vs deadlock** rule: single-instance cycle = deadlock; multi-instance cycle = only *possible* deadlock
- [ ] Know all 4 (or 5) **deadlock handling strategies**: Prevention, Avoidance (RAG algo / Banker's algo), Detection & Recovery, Ignorance (Ostrich)
- [ ] Understand **why** Ostrich Algorithm is used in practice (cost vs rarity trade-off)
- [ ] Can explain **Deadlock Prevention** techniques for each of the 4 conditions (especially: all-at-once vs release-before-request for Hold & Wait; forcible vs self preemption; resource ordering for circular wait)
- [ ] Can trace concurrent program interleavings to find **all possible final variable values** (race condition analysis)
- [ ] Remember: `x = x + 1` is **Load → Increment → Store**, not atomic, unless protected by the **same** mutex/semaphore
- [ ] Practiced Reader-Writer semaphore skeleton (First Readers-Writers problem with busy waiting)

---

*Part of my GATE CS/IT — Operating Systems revision notes collection.*
