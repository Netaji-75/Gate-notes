# Process Synchronization — Lecture 12 (IPC + Round Robin Numericals)
### GATE CS/IT | Principles of Operating Systems

> Source: GeeksforGeeks GATE CS&IT Engineering — Process Synchronization, Lecture 12.
> Note: The slide deck's stated agenda (Requirements of CS Problem, Lock Variable, Strict Alternation,
> Peterson Solution) is **not actually covered in this particular file** — the real content is
> **Inter-Process Communication (IPC)** followed by a long set of **Round Robin scheduling numericals**
> (these are recap/PYQ-practice slides). This note documents exactly what's in the deck.

---

## 📑 Table of Contents

| # | Topic |
|---|-------|
| 1 | What is Inter-Process Communication (IPC) |
| 2 | Independent vs Cooperating Processes |
| 3 | Why Processes Cooperate — 4 Drivers of IPC |
| 4 | Communication Models — Shared Memory vs Message Passing |
| 5 | Pipes (as an IPC mechanism) |
| 6 | IPC Methods & UNIX IPC Mechanisms |
| 7 | Process Synchronization = Process Coordination (intro) |
| 8 | Formula Bank (Round Robin) |
| 9 | Worked Examples (RR Gantt-chart numericals) |
| 10 | GATE PYQs (with answers/reasoning where solved in lecture) |
| 11 | Memory Tricks |
| 12 | Quick Revision Checklist |

---

## 1. What is Inter-Process Communication (IPC)

- IPC is the **mechanism provided by the OS** that lets processes communicate with each other.
- It covers two things:
  - One process **notifying another** that an event has occurred.
  - **Transferring data** from one process to another.
- **Intra-process communication** (inside a single process, e.g. `main()` calling `f(a,b)` which shares
  global variables) does **not** need IPC — normal function calls / globals are enough.
- **Inter-process communication** is needed the moment two *separate* processes (P1 with thread t1,
  P2 with thread t2) need to talk — they need some **shared medium** (e.g., a pipe) between an event
  `e1` in P1 and an event `e2` in P2.

---

## 2. Independent vs Cooperating Processes

| Aspect | Independent Process | Cooperating Process |
|---|---|---|
| Effect on others | Cannot affect or be affected by other processes | Can affect / be affected by other processes |
| Data | Executes in isolation | Shares data and state |
| Needs IPC? | ❌ No | ✅ Yes — needs mechanisms to communicate, exchange data, and synchronize |
| Scope | Single machine | May run on one or more computers connected by a network |

---

## 3. Why Processes Cooperate — 4 Drivers of IPC

1. **Information Sharing** — multiple users/processes need access to the same piece of information.
2. **Computation Speedup** — breaking a task into subtasks that run in parallel.
3. **Modularity** — dividing system functions into separate processes or threads.
4. **Convenience** — lets a single user work on many tasks simultaneously.

> 🧠 **Mnemonic**: **I S M C** → *Information, Speedup, Modularity, Convenience.*

---

## 4. Communication Models — Shared Memory vs Message Passing

| Feature | (a) Shared Memory | (b) Message Passing |
|---|---|---|
| How it works | Process A and Process B both read/write to a common **shared memory** region | Process A and B exchange data via a **message queue** (`m0, m1, m2 ... mn`) maintained by the kernel |
| Kernel involvement | Kernel sets up the shared region once; **not involved in every read/write** | Kernel is involved in **every send/receive** (moves messages through the queue) |
| Speed | Generally **faster** after setup (no kernel call per access) | Generally **slower** (system call overhead per message) |
| Synchronization | Programmer must handle synchronization (race conditions possible) | Kernel handles delivery order; simpler to reason about |
| Best for | Large/frequent data transfer between processes on the same machine | Smaller amounts of data, or processes across a network |

**Two models of IPC (summary):**
1. Shared Memory
2. Message Passing

---

## 5. Pipes (as an IPC mechanism)

- A **pipe** connects two processes P1 and P2 using a pair of file descriptors.
- Each process gets **two ends**: `pfd[0]` (read end) and `pfd[1]` (write end).
- Diagram: `P1 --(pfd[0], pfd[1])--> P2` and `P2 --(pfd[0], pfd[1])--> P1` (bidirectional via the shared pipe).
- Pipes are one of the oldest UNIX IPC mechanisms — simple, but typically **unidirectional** per pipe
  instance (need two pipes for full duplex) and work best for **related processes** (e.g., parent-child).

---

## 6. IPC Methods & UNIX IPC Mechanisms

**Methods in IPC (general list):**
1. Shared Memory
2. Message Passing
3. Pipes
4. Sockets
5. Semaphores
6. Message Queues

**UNIX-specific IPC mechanisms** (different UNIX versions introduced different ones):
- Pipes
- Message queues
- Semaphores
- Shared memory
- Sockets
- RPC (Remote Procedure Call)

> 🧠 **Mnemonic**: Think of these as **"the toolbox"** — Shared Memory & Message Passing are the two
> *models*; everything else (pipes, sockets, semaphores, message queues, RPC) are *concrete tools* that
> implement one of those two models.

---

## 7. Process Synchronization = Process Coordination

- The lecture explicitly frames **Process Synchronization** as **Process Coordination** — i.e., once
  cooperating processes share data (via IPC), they need to **coordinate access** to that shared data/state
  to avoid inconsistency. This sets up the motivation for the *Critical Section Problem*, locks, and
  Peterson's Solution (covered in a **different/later lecture**, not in this file).

---

## 8. Formula Bank (Round Robin)

### 8.1 Time Quantum for a Guaranteed Turn (n processes, scheduling overhead `s`, quantum `q`)

If `n` processes are running back-to-back in Round Robin (each gets exactly one time quantum `q`
per round, plus a context-switch overhead `s`), and a process must get its next turn on the CPU
exactly after `t` seconds:

$$t - ns = (n-1)\,q \quad\Longrightarrow\quad q = \frac{t - ns}{n-1}$$

**Variables:**
- `n` = number of processes
- `s` = scheduling/context-switch overhead (seconds)
- `q` = time quantum (seconds)
- `t` = time after which the *same* process is guaranteed to get the CPU again

**Three flavors of this formula (depending on the wording of the question):**

| Condition | Guarantee |
|---|---|
| $q = \dfrac{t-ns}{n-1}$ | Process gets CPU turn **exactly after** `t` seconds |
| $q \le \dfrac{t-ns}{n-1}$ | Process gets CPU turn **at least once within** `t` seconds |
| $q \ge \dfrac{t-ns}{n-1}$ | Process gets CPU turn **at least every** `t` seconds |

> 🧠 **Mnemonic**: "≤ → within (can happen sooner too)", "≥ → every (must not exceed t)", "= → exactly".

### 8.2 CPU Efficiency for Round Robin with I/O blocking

System: RR scheduling, time quantum `Q`, scheduling overhead `S`, each process runs on average `T`
seconds of CPU before blocking on I/O.

$$\eta = \frac{Q}{S+Q} \quad \text{(general working formula, } S < Q < T\text{)}$$

| Condition | CPU Efficiency `η` |
|---|---|
| $Q = \infty$ | $\eta = \dfrac{T}{S+T}$ |
| $Q > T$ | $\eta = 1$ (100%) |
| $S < Q < T$ | $\eta = \dfrac{Q}{S+Q}$ |
| $Q = S$ | $\eta = \dfrac{1}{2} = 50\%$ |
| $Q \approx 0$ | $\eta \approx 0$ |

**Variables:**
- `Q` = time quantum
- `S` = CPU scheduling overhead per context switch
- `T` = average CPU burst before a process blocks on I/O
- `η` (eta) = CPU efficiency/utilization

> 🧠 **Intuition**: Efficiency is just *"useful work" / "useful work + overhead"*. If the quantum is huge
> (`Q > T`), the process finishes its burst before getting preempted, so there's effectively **no
> RR overhead** → 100% efficient. If the quantum is tiny (`Q ≈ 0`), you spend almost all your time
> context-switching → efficiency collapses to 0.

---

## 9. Worked Examples (RR Numericals)

### Example 1 — Deriving the Time-Quantum Formula
**Setup:** `n` processes arrive at time `0⁺` with very large burst times. Scheduling overhead = `s`
seconds, time quantum = `q`. Ready queue order: `P1, P2, P3, P4, P5 → P1 …`

Gantt chart pattern: `s | P1 | s | P2 | s | P3 | s | P4 | s | P5 | s | P1 …`

Between two consecutive visits of `P1`, there are `(n-1)` other processes served, each contributing
`(s + q)`, plus one more `s` before P1 restarts:

$$t - ns = (n-1)q \implies q = \frac{t-ns}{n-1}$$

**Answer:** $q = \dfrac{t-ns}{n-1}$ (this process is guaranteed the CPU **exactly** after `t` seconds).

---

### Example 2 — MCQ Version of the Above (At Least Every t Seconds)
**Q:** Consider `n` processes sharing the CPU in round-robin. Each process switch takes `s` seconds.
What must the quantum `q` be so that switching overhead is minimized **but** each process is
guaranteed its turn **at least every** `t` seconds?

**Options:**
- A. $q \le \dfrac{t-ns}{n-1}$
- B. $q \ge \dfrac{t-ns}{n-1}$ ✅ **(Correct)**
- C. $q \le \dfrac{t-ns}{n+1}$
- D. $q \ge \dfrac{t-ns}{n+1}$

**Reasoning:**
- ✅ **B is correct** — "at least every `t` seconds" means the quantum must be **large enough** that the
  round-trip time never exceeds `t`; since $t - ns = (n-1)q$ at the boundary, any $q \ge \frac{t-ns}{n-1}$
  keeps the round-trip time ≤ `t`.
- ❌ A is wrong sign — `≤` would only guarantee the turn *sometimes sooner*, not *every* `t` seconds reliably.
- ❌ C, D wrong — denominator should be `(n-1)` (number of *other* processes served in between), not `(n+1)`.

---

### Example 3 — SJF vs RR: Average Turnaround Time Difference
**Setup:** 4 processes, all arrive at t = 0.

| Process | P1 | P2 | P3 | P4 |
|---|---|---|---|---|
| Burst time (ms) | 8 | 7 | 2 | 4 |

RR order: `P1, P2, P3, P4` (as scheduled), time quantum = 4 ms. **Question:** Find $|TAT_{avg}(SJF) - TAT_{avg}(RR)|$, rounded to 2 decimals.

*(Slide poses the question; detailed Gantt-chart solving for this one is not shown in the deck excerpt — attempt via standard SJF and RR Gantt charts as revision practice.)*

---

### Example 4 — RR Context-Switch Count Puzzle
**Setup:** Processes `P, Q, R, S` scheduled by Round Robin, quantum = 4, all arrive at `t = 0`, order
`P, Q, R, S`. Given constraints:
- Exactly **1** context switch from `S → Q`
- Exactly **1** context switch from `R → Q`
- Exactly **2** context switches from `Q → R`
- **0** context switches from `S → P`

**Reconstructed CPU timeline:** `P | Q | R | S | Q | R | Q`

**Question:** Which option's burst times are **NOT** possible?

| Option | P | Q | R | S |
|---|---|---|---|---|
| A | 4 | 10 | 6 | 2 |
| B | 2 | 9 | 5 | 1 |
| C | 4 | 12 | 5 | 4 |
| D | 3 | 7 | 7 | 3 |

**Answer:** **D** is the one that does **not** fit the reconstructed CPU sequence/transition constraints
(A, B, C are consistent with the `S→Q:1, R→Q:1, Q→R:2, S→P:0` pattern; D breaks it).

---

### Example 5 — Response Time in RR with I/O (10 processes, identical requests)
**Setup:** 10 processes, all arrive at `t = 0`. Each process has 20 identical *requests*; each request
consumes **20 ms CPU**, then **10 ms I/O**, then the next request starts. Scheduling overhead = 2 ms,
time quantum = 20 ms.

| Sub-question | Answer |
|---|---|
| (i) Response time of 1st request of the **1st** process (P1) | **22 ms** (= overhead 2 + CPU burst 20) |
| (ii) Response time of 1st request of the **last** process (P10) | **220 ms** (= 10 × (2+20)) |
| (iii) Response time of a **subsequent** request of any process | Depends on I/O overlap with other processes' CPU turns — worked via the general pattern below |

**Response Time — definition used:** the difference between the time a process/request is
**admitted** and the time it takes to **produce the response (result)**.

**General derivation shown for a smaller variant (S = 2 ms, TQ = 10 ms, 10 processes, 1 request each with 10 ms I/O):**

$$RT(P_{1,1}) = 10(2+10) + (2+10) = 132\ \text{ms}$$
$$RT(P_{10,1}) = 10(2+10) + 10(2+10) = 240\ \text{ms}$$

*(Pattern: `k × (s+q)` to complete one full round for all processes, plus extra rounds if the process's I/O hasn't finished by the time its turn comes around again.)*

---

### Example 6 — Three Processes, CPU + I/O Loop (First I/O completion time)
**Setup:** Processes A, B, C — each executes a **loop of 100 iterations**; each iteration does `tc` ms
of CPU work then `t_io` ms of I/O (system has enough I/O devices, negligible scheduling overhead).

| Process | $t_c$ (ms) | $t_{io}$ (ms) |
|---|---|---|
| A | 100 | 500 |
| B | 350 | 500 |
| C | 200 | 500 |

Start times: A at 0 ms, B at 5 ms, C at 10 ms. Pure time-sharing (Round Robin), **time slice = 50 ms**.

**Question:** At what time does process **C** complete its **first I/O operation**?

**Gantt chart (CPU):**
```
CPU:  A | B | C | A | B | C | B | C | B | C
time: 0   50  100 150 200 250 300 350 400 450 500
```
- C needs `tc = 200 ms` of CPU total for its first iteration → across 4 slices of 50 ms each
  (at 100–150, 250–300, 350... in the RR rotation) C accumulates 200 ms of CPU by **t = 500 ms**.
- C then does I/O for 500 ms → finishes I/O at **500 + 500 = 1000 ms**.

**Answer: 1000 ms**

---

### Example 7 — Dynamic-Priority Preemptive Scheduling (FCFS/LCFS as special cases)
**Setup:** Preemptive priority scheduling with **dynamically changing priorities**. A process is
assigned priority **0** on arrival. The **running** process's priority increases at rate **β**;
processes waiting in the **ready queue** have priority increasing at rate **α**.

**Question:** What scheduling discipline results for each condition?

| Condition | Resulting Discipline |
|---|---|
| $\beta > \alpha > 0$ | **FCFS** — running process's priority grows faster than waiting ones, so once running, a process is never preempted; new arrivals (priority 0) never overtake it → effectively runs to completion in arrival order |
| $\alpha < \beta < 0$ | **LCFS** (Last Come First Served) — both priorities decay (negative growth), but the *ready-queue* priority decays faster (more negative) than the running one, so the **most recently arrived** (priority still near 0) process keeps preempting → last process in gets served first |

> 🧠 **Mnemonic**: "**Runner grows faster → stays running → FCFS.** **Waiter decays faster → newest
> arrival wins → LCFS.**"

---

## 10. GATE PYQs (as listed in the deck)

> The deck's "GATE PYQ's" section is effectively **Examples 1–8 above** plus the additional problems
> below, which are posed in the slides as PYQ-style practice but **not fully solved on-screen** in this
> particular lecture segment — solve them as self-test practice.

### PYQ A — FCFS Mixed Process Types
Two process types on one CPU: **Actuators (A)** — CPU burst 6 s; **Controllers (C)** — CPU burst 8 s.
- Type A arrives at t = 10, 20, 30, 40, 50 s
- Type C arrives at t = 11, 22, 33, 44, 55 s
- FCFS scheduling; first process (A) starts running at t = 10 s.

**Find:** Average waiting time (rounded to 1 decimal) for all 10 processes.
*(Not solved in the visible slide — reconstruct the Gantt chart from arrival times to solve.)*

### PYQ B — Preemptive Priority + Round Robin
6 processes with the following data (quantum = 10 units; idle task $P_{idle}$ has priority 0):

| Process | Priority | Burst | Arrival |
|---|---|---|---|
| P1 | 40 | 20 | 0 |
| P2 | 30 | 25 | 25 |
| P3 | 30 | 25 | 30 |
| P4 | 35 | 15 | 60 |
| P5 | 5 | 10 | 100 |
| P6 | 10 | 10 | 105 |

**Find:** (A) Gantt chart, (B) turnaround time per process, (C) waiting time per process, (D) CPU utilization.
*(Higher number = higher priority; preempted process goes to the back of the queue.)*

### PYQ C — Two Processors, Non-Preemptive Priority Scheduling
Processors $M_1$, $M_2$; processes P1–P4 with bursts 20, 16, 25, 10 ms (all arrive together).
- $M_1$ priority order: P1 > P3 > P2 > P4
- $M_2$ priority order: P2 > P3 > P4 > P1
- A process goes to whichever free processor allows the highest-priority eligible process to run; ignore context-switch time.

**Options:** (A) 9.00 (B) 8.75 (C) 6.50 (D) 7.50

*(Answer not marked in the visible slide — work it out by simulating both processors in parallel, always assigning the highest-priority *available* process to a free processor.)*

---

## 11. Memory Tricks / Mnemonics (collected)

| Trick | Meaning |
|---|---|
| **I S M C** | Information sharing, Speedup, Modularity, Convenience — the 4 reasons processes cooperate |
| **≤ → within, ≥ → every, = → exactly** | How to read the three variants of the RR time-quantum-guarantee formula |
| **η = useful work / (useful work + overhead)** | Core intuition for CPU efficiency formulas in RR |
| **Runner grows faster → FCFS; Waiter decays faster → LCFS** | Dynamic-priority scheduling special cases |
| Shared Memory = "same room"; Message Passing = "mailboxes with a queue" | Quick way to distinguish the two IPC models |

---

## 12. Quick Revision Checklist ✅

- [ ] Definition of IPC — mechanism for processes to notify events / transfer data
- [ ] Intra-process vs Inter-process communication — when is IPC actually needed?
- [ ] Independent vs Cooperating processes (table)
- [ ] 4 drivers of cooperation: Information sharing, Computation speedup, Modularity, Convenience
- [ ] Two IPC models: Shared Memory vs Message Passing (+ role of kernel in each)
- [ ] Pipes: `pfd[0]` / `pfd[1]`, connects two related processes
- [ ] 6 IPC methods: shared memory, message passing, pipes, sockets, semaphores, message queues
- [ ] UNIX IPC mechanisms: pipes, message queues, semaphores, shared memory, sockets, RPC
- [ ] Process Synchronization ≡ Process Coordination (framing for next lecture on Critical Section, Peterson's Solution)
- [ ] RR time-quantum formula: $q = \frac{t-ns}{n-1}$ and its ≤ / ≥ variants
- [ ] RR CPU-efficiency formula: $\eta = \frac{Q}{S+Q}$ and its boundary cases ($Q>T$, $Q=S$, $Q\to0$)
- [ ] Can reconstruct a Gantt chart from context-switch-count constraints (Example 4 style)
- [ ] Can compute response time for repeated CPU+I/O requests across many RR processes (Example 5 style)
- [ ] Can compute "time of first I/O completion" for looped CPU/IO processes under RR (Example 6 style)
- [ ] Dynamic-priority preemptive scheduling → FCFS ($\beta>\alpha>0$) vs LCFS ($\alpha<\beta<0$)
- [ ] Practice PYQs: FCFS mixed-arrival waiting time, preemptive-priority+RR Gantt chart, dual-processor non-preemptive priority scheduling

---

*Still pending from the stated agenda (cover in the next lecture's notes): Requirements of the
Critical Section Problem, Lock Variable, Strict Alternation, Peterson's Solution.*
