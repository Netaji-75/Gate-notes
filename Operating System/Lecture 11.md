# CPU Scheduling — Lecture 11 (Round Robin, Priority+RR, Multilevel Queue & Feedback Queue)
### GeeksforGeeks GATE CS & IT — Principles of Operating Systems

---

## 📑 Topics Covered

| # | Topic |
|---|-------|
| 1 | Need for CPU Scheduling |
| 2 | Scheduling Criteria (Terms & Concepts) |
| 3 | Round Robin (RR) Scheduling |
| 4 | Performance Analysis of RR (Time Quantum trade-off) |
| 5 | Priority Scheduling + Round Robin (hybrid) |
| 6 | Multilevel Queue (MLQ) Scheduling |
| 7 | Multilevel Feedback Queue (MLFQ) Scheduling |
| 8 | Diagnostic Comparison of All Algorithms |
| 9 | Review MCQs |
| 10 | GATE PYQs & Practice Numericals |

---

## 1. Need for CPU Scheduling — Quick Recap

- Modern OS supports **multiprogramming**: many processes reside in memory, CPU switches among them to maximize utilization.
- **CPU Scheduling** = deciding *which* ready process gets the CPU *next*.
- Goal: maximize CPU utilization & throughput, minimize turnaround/waiting/response time — **not just one metric in isolation** (see Review Q).

---

## 2. Scheduling Criteria — Terms & Concepts

| Metric | Formula | Meaning |
|---|---|---|
| **Turnaround Time (TAT)** | $TAT = CT - AT$ | Total time from arrival to completion |
| **Waiting Time (WT)** | $WT = TAT - BT$ | Time spent waiting in ready queue |
| **Response Time (RT)** | $RT = \text{(time of first CPU burst)} - AT$ | Time until process *first* gets CPU (important for interactive systems) |
| **Context Switching** | — | Overhead of saving/restoring process state; **pure overhead, no useful work done** |

> **CT** = Completion Time, **AT** = Arrival Time, **BT** = Burst Time

---

## 3. Round Robin (RR) Scheduling

### 🧠 Core Idea
- Designed for **Interactive + Time-Sharing systems**.
- Each process gets a fixed **Time Quantum (TQ)** / time slice.
- **Preemptive**: process is preempted when its time quantum **expires** (not based on priority/burst comparison).
- **Selection criterion:** simply *Arrival Time + Time Quantum* — i.e., pure cyclic rotation, no burst-time comparison needed (unlike SJF/SRTF).

> 🔑 **Mnemonic:** RR = "everyone gets a fair turn on the carousel." Preemption trigger = **expiration of time quantum**, nothing else.

### Flow of Execution
1. Pick process from front of Ready Queue.
2. If **Burst Time ≤ TQ** → runs to completion → **Terminate**.
3. Else → runs for exactly TQ → goes to **back of Ready Queue** → repeat.

### ⚖️ The Balancing Act — Choosing Time Quantum

| Quantum Size | Effect |
|---|---|
| **Too large** | RR degenerates into **FCFS** (each process finishes before quantum expires) |
| **Too small** | Excessive **context-switching overhead** dominates → CPU efficiency crashes |
| **Optimal** | Chosen based on system behavior — general rule of thumb: 80% of CPU bursts should be shorter than TQ |

---

## 4. Worked Examples — Round Robin

### 🔹 Example 1 — Basic RR (TQ = 2)

| Process | AT | BT |
|---|---|---|
| P1 | 0 | 4 |
| P2 | 0 | 2 |
| P3 | 0 | 3 |
| P4 | 0 | 6 |

**Gantt Chart:**
```
| P1 | P2 | P3 | P4 | P1 | P3 | P4 | P4 |
0    2    4    6    8   10   11   13   15
```

| Process | CT | TAT (=CT-AT) | WT (=TAT-BT) | RT (first start - AT) |
|---|---|---|---|---|
| P1 | 10 | 10 | 6 | 0 |
| P2 | 4  | 4  | 2 | 2 |
| P3 | 11 | 11 | 8 | 4 |
| P4 | 15 | 15 | 9 | 6 |

**Avg TAT = 10, Avg WT = 6.25, Avg RT = 3**

---

### 🔹 Example 2 — Tie-Breaking Convention Matters! (TQ = 2)

| Process | AT | BT |
|---|---|---|
| P1 | 0 | 4 |
| P2 | 2 | 3 |
| P3 | 4 | 5 |
| P4 | 5 | 1 |

> ⚠️ **Key Insight:** When a **new arrival** and a **preempted process** (quantum just expired) occur at the *same instant*, the order in which they're re-queued changes the entire schedule and the resulting averages! Two valid conventions are shown below.

**Approach I — New arrival queued *before* the preempted process:**
```
| P1 | P2 | P1 | P3 | P2 | P4 | P3 | P3 |
0    2    4    6    8    9   10   12   13
```
Avg TAT = 6.75, Avg WT = 3.5, Avg RT = 1.5

**Approach II — Preempted process re-queued *before* the new arrival:**
```
| P1 | P1 | P2 | P3 | P4 | P2 | P3 | P3 |
0    2    4    6    8    9   10   12   13
```
Avg TAT = 6.25, Avg WT = 3, Avg RT = 1.75

> 💡 **Exam tip:** If a GATE question doesn't specify the tie-breaking rule, check if the answer options match one particular convention — this trips up many students!

---

### 🔹 Example 3 — RR with Staggered Arrivals + CPU Idle Time (TQ = 3)

| Process | AT | BT |
|---|---|---|
| P1 | 2 | 8 |
| P2 | 3 | 5 |
| P3 | 4 | 6 |
| P4 | 6 | 7 |
| P5 | 7 | 2 |

**Gantt Chart:**
```
|idle| P1 | P2 | P3 | P1 | P4 | P5 | P2 | P3 | P1 | P4 | P4 |
0    2    5    8    11   14   17   19   21   24   26   29   30
```
- CPU is **idle from 0–2** since no process has arrived yet.
- Note P5 (BT=2) finishes in a *single* burst since BT < TQ.

---

### 🔹 Time Quantum vs Context-Switch Overhead (Illustrative)

For a single process needing **10 units** of CPU time:

| Quantum Size | Context Switches |
|---|---|
| 12 (> burst) | 0 (never preempted → FCFS-like) |
| 6 | 1 |
| 1 | 9 |

### 🔹 Extreme Case — Very Small Quantum Kills Efficiency

Given: TQ = 0.1, Context-switch overhead **S = 2** (units)

```
| S | P1 | S | P2 | S | P3 | S | P4 |
0   2   2.1  4.1  4.2  6.2  6.3  8.3  8.4
```

$$\text{Efficiency} = \frac{\text{Useful CPU time}}{\text{Total time}} = \frac{0.4}{8.4} = \frac{1}{21} \approx 5\%$$

> Compare: with TQ = 10 for processes of BT 4,5,2,3 (all < TQ), RR **degenerates into FCFS** — zero context switches.

> 📈 **Textbook note (Silberschatz):** A graph of *Average Turnaround Time vs Time Quantum* (for BTs 6,3,1,7) is typically **non-monotonic** — it doesn't always improve as quantum increases or decreases. There is no universal "best" quantum; it must be tuned per workload.

---

## 5. Priority Scheduling + Round Robin (Hybrid)

### Rule
> Run the process with the **highest priority**. Processes with the **same priority** run in **Round Robin** among themselves.

### 🔹 Worked Example (Classic Textbook Problem, TQ = 2)

| Process | Burst Time | Priority (1 = highest) |
|---|---|---|
| P1 | 4 | 3 |
| P2 | 5 | 2 |
| P3 | 8 | 2 |
| P4 | 7 | 1 |
| P5 | 3 | 3 |

**Gantt Chart:**
```
| P4 | P2 | P3 | P2 | P3 | P2 | P3 | P1 | P5 | P1 | P5 |
0    7    9   11   13   15   16   20   22   24   26   27
```

- **P4** (priority 1) runs first, to completion (non-preemptive across priority levels here since it's the sole highest-priority process).
- **P2, P3** (priority 2) round-robin between themselves (TQ=2) until both finish (t=20).
- **P1, P5** (priority 3) round-robin last (t=20 to 27).

---

## 6. Multilevel Queue (MLQ) Scheduling

### Core Idea
- Ready Queue is **permanently partitioned** into multiple queues based on **process type/priority** (e.g., system, interactive, batch, student processes).
- Each queue can use a **different scheduling algorithm**.
- **Absolute priority between queues**: a lower queue runs *only if* all higher-priority queues are empty (strict, no queue-jumping).

### Typical Setup

| Queue | Process Type | Algorithm |
|---|---|---|
| Queue 0 (highest) | System processes | SJF |
| Queue 1 | Interactive processes | Round Robin |
| Queue 2 | Interactive editing processes | Round Robin |
| Queue 3 | Batch processes | FCFS |
| Queue 4 (lowest) | Student / background processes | FCFS |

### Parameters that define an MLQ scheduler
1. Number of queues
2. Scheduling algorithm for each queue
3. Method to decide **which queue** a process enters
4. Scheduling policy **among** the queues (inter-queue policy)

> ⚠️ **Major weakness:** **Starvation** of lower-priority queues if higher queues are never empty. There is **no process movement** between queues in strict MLQ.

---

## 7. Multilevel Feedback Queue (MLFQ) Scheduling

### Core Idea
- Like MLQ, but processes **can move between queues** — this is the key difference.
- A **fluid system**: processes migrate between priority pools based on their *behavior* (CPU-bound vs I/O-bound).

### Parameters defining an MLFQ scheduler
1. Number of queues
2. Scheduling algorithm for each queue
3. Method to **upgrade** a process (move to higher-priority queue)
4. Method to **demote** a process (move to lower-priority queue)
5. Method to decide which queue a new process enters
6. **Aging can be implemented using MLFQ** (prevents starvation!)

### Mechanism

| Direction | Trigger |
|---|---|
| **Demotion** | CPU hogs that exhaust their time quantum without finishing drop to a lower tier (longer quantum, lower priority) |
| **Upgrade (Aging)** | Processes starving in bottom queues are promoted to higher tiers |

### 🔹 Classic 3-Queue Example
- **Q0**: RR, time quantum = 8 ms (highest priority)
- **Q1**: RR, time quantum = 16 ms
- **Q2**: FCFS (lowest priority)

**Flow:** New process enters Q0 → gets 8ms → if unfinished, demoted to Q1 → gets 16 more ms → if still unfinished, demoted to Q2 (FCFS, runs till completion or until higher queues have work).

> 🔑 **Mnemonic:** MLFQ = "the ultimate adaptable scheduler" — short jobs finish fast in the top queue (like SJF behavior), long CPU-bound jobs sink to FCFS at the bottom, and aging keeps everyone fed.

---

## 8. Diagnostic Comparison — All Algorithms

| Algorithm | Approach | Starvation Risk | Best Use Case |
|---|---|---|---|
| FCFS | Non-Preemptive | None | Simple batch systems |
| SJF | Non-Preemptive | High | Predictable burst lengths |
| SRTF | Preemptive | High | Minimal average waiting time |
| Priority | Both | High (without Aging) | Strict hierarchy systems |
| **Round Robin** | Preemptive | None | Time-sharing / Interactive UI |
| **Multilevel Feedback** | Preemptive | Low | Modern Operating Systems |

### 🧭 Mind-Map Summary

- **CPU Scheduling** branches into:
  - **Metrics:** Turnaround Time, Wait Time, Response Time, Context Switching
  - **Algorithms:** FCFS, SJF, Round Robin, Multilevel Feedback
  - **Core Concepts:** CPU vs I/O bound processes, Starvation, Time Quantum

---

## 9. Review MCQs (with Answers)

| # | Question | Correct Answer | Why others are wrong |
|---|---|---|---|
| 1 | In RR, the time quantum should be: | **C. Optimally chosen based on system behavior** | Too large → FCFS; too small → high overhead; equal to BT isn't generalizable |
| 2 | RR is best suited for: | **C. Time-sharing systems** | Batch/real-time/multiprocessing don't need fair round-robin turn-taking as their primary need |
| 3 | If TQ is too small in RR, it leads to: | **C. High context-switching overhead** | Small TQ improves response time but at the cost of overhead, not CPU utilization or throughput |
| 4 | Round Robin algorithm: | **A. Is preemptive** | It's not non-preemptive, doesn't run one process till completion, and does have context switching |
| 5 | In RR, each process: | **C. Is executed in a cyclic order for a fixed time** | Not alphabetical, not one-shot batch, not I/O-based priority |
| 6 | In MLQ, each queue typically has: | **B. A different scheduling algorithm** | Different queues serve different process types (system/interactive/batch), hence different algorithms |
| 7 | In MLQ, processes are classified based on: | **C. Their type or priority** (e.g., system, interactive, batch) | Not memory usage, not CPU burst alone, not submitting user |
| 8 | Process movement between queues in *strict* MLQ: | **B. Only if their priority changes** | In strict MLQ, movement is otherwise not the default behavior (contrast with MLFQ) |
| 9 | In MLQ, the highest priority queue is usually: | **B. System processes queue** | System processes need the fastest response for OS stability |
| 10 | A major disadvantage of MLQ: | **B. Starvation of lower priority queues** | MLQ can run background jobs, uses multiple algorithms, and doesn't *always* context switch |

---

## 10. GATE PYQs & Practice Problems

### 🔸 PYQ 1 — Preemptive Priority with Periodic Processes

> Consider a system with Preemptive Priority scheduling with 3 processes P1, P2, P3 having **infinite instances**. Instances arrive at regular intervals of 3, 7, & 20 ms respectively. Priority = inverse of period. Each instance of P1, P2, P3 consumes 1, 2, 4 ms of CPU respectively. The 1st instance of each process is available at 1 ms. **What is the completion time of the 1st instance of P3?**

**Setup:**

| Process | Period | Priority (1/Period) | Burst per instance | Instance arrivals |
|---|---|---|---|---|
| P1 | 3 | 1/3 (Highest) | 1 ms | 1, 4, 7, 10, 13, ... |
| P2 | 7 | 1/7 | 2 ms | 1, 8, 15, 22, ... |
| P3 | 20 | 1/20 (Lowest) | 4 ms | 1, 21, 41, ... |

**Gantt Chart (preemptive priority):**
```
|idle| P1 | P2 | P1 | P3 | P1 | P2 | P1 | P3 |
0    1    2    4    5    7    8   10   11   13
```
✅ **Answer: Completion time of 1st instance of P3 = 13 ms**

*(P3 gets pushed back repeatedly by higher-priority P1/P2 instances arriving during its execution.)*

---

### 🔸 PYQ 2 — Optimal Non-Preemptive Scheduling Sequence

> The sequence _____ is an optimal non-preemptive scheduling sequence for the jobs below, leaving the CPU idle for ____ unit(s) of time.

| Job | Arrival Time | Burst Time |
|---|---|---|
| 1 | 0.0 | 9 |
| 2 | 0.6 | 5 |
| 3 | 1.0 | 1 |

| Option | Sequence | Idle time |
|---|---|---|
| **A ✅** | {3, 2, 1} | 1 |
| B ❌ | {2, 1, 3} | 0 |
| C ❌ | {3, 2, 1} | 0 |
| D ❌ | {1, 2, 3} | 5 |

✅ **Answer: A** — Sequence: idle(0–1) → J3(1–2) → J2(2–7) → J1(7–16)

> 💡 **Concept:** This illustrates that a **non-preemptive** scheduler *can* sometimes benefit from **briefly staying idle** (rather than immediately dispatching the only available job) to let a shorter job arrive — improving overall average waiting time. This is a classic demonstration that "greedy immediate dispatch" isn't always optimal for non-preemptive scheduling.

---

### 🔸 PYQ 3 — SRTF Context Switches

> Three CPU-bound tasks with execution times 15, 12, and 5 time units arrive at times 0, *t*, and 8 respectively. Using **Shortest Remaining Time First (SRTF)**, what should be the value of *t* to have exactly **4 context switches**? (Ignore switches at time 0 and at the end.)

| Option | Range |
|---|---|
| **A ✅** | 0 < t < 3 |
| B ❌ | t = 0 |
| C ❌ | t ≤ 3 |
| D ❌ | 3 < t < 8 |

✅ **Answer: A (0 < t < 3)**

> **Reasoning sketch:** If *t* > 3, the arriving task at time *t* never has a remaining time short enough to preempt the running task — fewer switches result. Only when *t* < 3 does the second arrival preempt early enough to set off the extra chain of preemptions needed to reach exactly 4 switches. (t = 0 causes the shortest job to run first immediately, reducing switches to 2.)

---

### 🔸 PYQ 4 — I/O + CPU Bursts, Preemptive Priority

> Arrival time, priority, and CPU/I/O burst durations for P1, P2, P3 are given. Each process has CPU burst → I/O burst → CPU burst.

| Process | Arrival Time | Priority | Burst Durations (CPU, I/O, CPU) |
|---|---|---|---|
| P1 | 0 | 2 | 1, 5, 3 |
| P2 | 2 | 3 (lowest) | 3, 3, 1 |
| P3 | 3 | 1 (highest) | 2, 3, 1 |

> The Multi-Programmed OS uses **Preemptive Priority Scheduling**. What are the finish times of P1, P2, P3?

| Option | P1 | P2 | P3 |
|---|---|---|---|
| A | 11 | 15 | 9 |
| **B ✅** | **10** | **15** | **9** |
| C | 11 | 16 | 10 |
| D | 12 | 17 | 11 |

✅ **Answer: B (10, 15, 9)**

---

### 🔸 PYQ 5 — True/False Statement Set on CPU Scheduling

| Statement | Correct? |
|---|---|
| A. The goal is to *only* maximize CPU utilization and minimize throughput | ❌ False — scheduling has *multiple* competing goals, and minimizing throughput is never a goal (we want to *maximize* throughput) |
| B. Turnaround time includes waiting time | ✅ True — $TAT = WT + BT$ |
| C. Implementing preemptive scheduling needs hardware support | ✅ True — requires a timer interrupt |
| D. Round-robin policy can be used even when CPU time required by each process is not known *a priori* | ✅ True — unlike SJF/SRTF, RR needs no burst-time knowledge in advance |

---

### 🔸 PYQ 6 — Preemptive Priority + Round Robin (Practice)

> Processes scheduled using **Preemptive Priority with Round-Robin**. Higher number = higher priority. Idle task $P_{idle}$ has priority 0. Time quantum = 10 units. If preempted by higher priority, process goes to the **end** of the queue.

| Process | Priority | Burst | Arrival |
|---|---|---|---|
| P1 | 40 | 20 | 0 |
| P2 | 30 | 25 | 25 |
| P3 | 30 | 25 | 30 |
| P4 | 35 | 15 | 60 |
| P5 | 5 | 10 | 100 |
| P6 | 10 | 10 | 105 |

**Tasks:** (A) Draw Gantt chart, (B) Turnaround time, (C) Waiting time, (D) CPU utilization.
*(Practice question — apply the RR + Priority rules from Section 5 above to solve.)*

---

### 🔸 PYQ 7 — Dual Processor Non-Preemptive Priority Scheduling

> Two processors M1, M2. Four processes P1(20), P2(16), P3(25), P4(10) — CPU bursts in ms, all arrive at t=0.
> - **M1** priority: P1 > P3 > P2 > P4
> - **M2** priority: P2 > P3 > P4 > P1
>
> A process is scheduled to a free processor only if no higher-priority process (per that processor's order) is waiting. What is the average waiting time?

**Solution:**
- t=0: Both processors free. M1 takes its top choice **P1**; M2 takes its top choice **P2** (no conflict).
- M2 finishes P2 at t=16 → picks next by its priority order among {P3, P4} → **P3** (16 → 41)
- M1 finishes P1 at t=20 → only **P4** left → runs (20 → 30)

| Process | CT | BT | WT = CT − BT |
|---|---|---|---|
| P1 | 20 | 20 | 0 |
| P2 | 16 | 16 | 0 |
| P3 | 41 | 25 | 16 |
| P4 | 30 | 10 | 20 |

✅ **Average Waiting Time = (0+0+16+20)/4 = 9.00 ms**

---

### 🔸 PYQ 8 — Round Robin Completion Time

> Processes A, B, C, D arrive at time 0⁺ with burst times 4, 1, 8, 1 respectively. RR with TQ = 1. Find completion time of process A.

**Gantt Chart:**
```
| A | B | C | D | A | C | A | C | A |
0   1   2   3   4   5   6   7   8   9
```
✅ **Answer: Completion time of A = 9**

---

### 🔸 PYQ 9 — Deriving the RR Guarantee Formula (⭐ Starred / Important)

> Consider a system with **n** processes arriving at time 0⁺ with substantially large burst times. CPU scheduling overhead is **s** seconds, time quantum is **q** seconds. What must **q** be so each process is guaranteed to get its turn on the CPU **exactly** after **t** seconds (in its subsequent run)?

**Derivation:**
```
| S | P1 | S | P2 | S | P3 | ... | S | Pn | S | P1 |
0   s    q  ...                              <---- t ---->
```
- Each full cycle (1 process's execution + its overhead) = $(s + q)$
- To return to the same process, we go through **all n** processes: total cycle time = $n(s+q)$
- Setting this equal to *t*:

$$ n(s + q) = t \implies q = \frac{t - ns}{n} = \frac{t}{n} - s $$

### 🔸 PYQ 10 — Same Concept as a GATE MCQ (Minimum Guarantee)

> n processes share CPU in round-robin. Each switch takes **s** seconds. What quantum **q** minimizes switching overhead while still guaranteeing each process CPU access **at least** every **t** seconds?

| Option | Formula |
|---|---|
| **A ✅** | $q \le \dfrac{t - ns}{n-1}$ |
| B | $q \ge \dfrac{t-ns}{n-1}$ |
| C | $q \le \dfrac{t-ns}{n+1}$ |
| D | $q \ge \dfrac{t-ns}{n+1}$ |

✅ **Answer: A** — Note the denominator is $(n-1)$, not $n$, because this variant asks for a **worst-case guarantee** ("at least every *t* seconds") rather than an exact periodic return — a subtly different (and classic) GATE trap.

---

### 🔸 PYQ 11 — SRTF vs Round Robin with I/O Bursts (TIFR-style)

> P1 needs 12 units CPU + 20 units I/O total — pattern: after every 3 units of CPU, does 5 units of I/O (repeated 4×). P2 needs 15 units CPU, no I/O, arrives just after P1. Compute completion times using **(1) SRTF** and **(2) RR (TQ=4)**.

**Part 1 — SRTF (solved in lecture):**

**Gantt Chart:**
```
| P1 | P2 | P1 | P2 | P1 | P2 | P1 |I/O| (P1 completes)
0    3    8   11   16   19   24   27  32
```
- P1 always preempts P2 when it returns from I/O (its next 3-unit chunk < P2's larger remaining time).
- P2 completes exactly when P1 finishes its 3rd chunk's timing lines up → **P2 completes at t = 24**
- P1 finishes its last CPU chunk at t=27, then does final I/O (27→32) → **P1 completes at t = 32**

**Part 2 — Round Robin (TQ=4):** *(left as practice — apply RR mechanics from Section 3, respecting P1's fixed 3-unit CPU chunks before it blocks for I/O)*

---

### 🔸 PYQ 12 — SJF vs RR: Average Turnaround Time Difference (Practice)

| Process | P1 | P2 | P3 | P4 |
|---|---|---|---|---|
| Burst (ms) | 8 | 7 | 2 | 4 |

> All arrive at t=0. For RR, order = P1,P2,P3,P4, TQ = 4 ms. Find |avg TAT(SJF) − avg TAT(RR)| (2 decimal places).
*(Practice — compute SJF Gantt chart and RR Gantt chart separately, then find average TAT for each.)*

---

### 🔸 PYQ 13 — Round Robin Context Switch Count Puzzle (Practice)

> Four processes P, Q, R, S scheduled via RR (TQ=4), all arrive at t=0. Given: exactly 1 switch S→Q, exactly 1 switch R→Q, exactly 2 switches Q→R, and no switch S→P. Which completion times (P,Q,R,S) are consistent?

| Option | P | Q | R | S |
|---|---|---|---|---|
| A | 4 | 10 | 6 | 2 |
| B | 2 | 9 | 5 | 1 |
| C | 4 | 12 | 5 | 4 |
| D | 3 | 7 | 7 | 3 |

*(Practice — work backward from the context-switch constraints to reconstruct the Gantt chart.)*

---

### 🔸 PYQ 14 — RR with Repeated I/O Requests (Practice)

> 10 processes, all arrive at t=0, each with 20 identical requests. Each request = 20ms CPU + 10ms I/O. Scheduling overhead = 2ms, TQ = 20ms. Find:
> (i) Response time of 1st request of 1st process
> (ii) Response time of 1st request of the last process
> (iii) Response time of subsequent requests of any process

*(Practice — since TQ exactly matches CPU need per request, this becomes a cyclic FCFS-like pattern with overhead 2ms per switch; work out the queue rotation.)*

---

### 🔸 PYQ 15 — RR with Iterative CPU/I/O Loops (Practice)

> Processes A, B, C each run a loop of 100 iterations; each iteration = $t_c$ ms CPU + $t_{io}$ ms I/O.

| Process | $t_c$ | $t_{io}$ |
|---|---|---|
| A | 100 | 500 |
| B | 350 | 500 |
| C | 200 | 500 |

> Started at 0, 5, 10 ms respectively, pure RR with time slice = 50ms. Find the time at which C completes its **first** I/O operation.

*(Practice — track RR rotation with slice=50ms until C accumulates 200ms of CPU time.)*

---

### 🔸 PYQ 16 — Priority Scheduling with Dynamic Aging (Conceptual)

> On arrival, a process gets priority 0. **Running** process priority increases at rate **β**; priority of processes in the **ready queue** increases at rate **α**. By tuning α and β, different scheduling disciplines emerge. What discipline results for:
> 1. **β > α > 0**
> 2. **α < β < 0**

> 💡 **Conceptual takeaway:** When the *running* process's priority grows faster than waiting processes' (β > α > 0), the running process tends to keep winning the priority race → behavior approximates **FCFS** (once running, stays running). When the relationship is reversed or priorities decay, waiting processes can more easily overtake the running one → behavior shifts toward **more frequent preemption (RR-like)**. This models how tunable aging parameters can *simulate* other scheduling policies within a single priority-based framework.

---

### 🔸 PYQ 17 — CPU Efficiency Formula for RR (Conceptual)

> RR with time quantum **Q**, scheduling overhead **S** seconds. Each process runs **T** seconds on average before blocking on I/O. Give a formula for CPU efficiency for:

| Case | Efficiency |
|---|---|
| 1. $Q = \infty$ | $\dfrac{T}{T+S}$ *(quantum never triggers preemption; only 1 switch per I/O block)* |
| 2. $Q > T$ | $\dfrac{T}{T+S}$ *(same as above — process blocks before quantum expires)* |
| 3. $S < Q < T$ | $\dfrac{Q}{Q+S}$ *(quantum expires before natural block — more frequent switching)* |
| 4. $Q = S$ | $\dfrac{S}{S+S} = 50\%$ *(overhead equals useful work each cycle)* |
| 5. $Q \approx 0$ | $\approx 0\%$ *(overhead completely dominates — matches the TQ=0.1 example in Section 4!)* |

---

### 🔸 PYQ 18 — FCFS with Periodic Process Arrivals (Practice)

> CPU executes two process types: **Actuators (A)**, CPU burst = 6s, and **Controllers (C)**, CPU burst = 8s. New A arrives at t=10,20,30,40,50s. New C arrives at t=11,22,33,44,55s. Scheduling = **FCFS**. First A starts running at t=10s. Find average waiting time for all 10 processes (round to 1 decimal).

*(Practice — simulate strict FCFS arrival order and compute waiting time for each of the 10 processes.)*

---

## ✅ Quick Revision Checklist

- [ ] Can define **TAT, WT, RT** and derive one from another
- [ ] Know why **context switching is pure overhead**
- [ ] Round Robin: preemption trigger = **quantum expiry only**
- [ ] Can solve RR Gantt charts including **staggered arrivals** and **idle CPU time**
- [ ] Understand **tie-breaking convention** issue (new arrival vs. requeued process) and how it changes averages
- [ ] Know the **quantum trade-off**: too large → FCFS; too small → high overhead; efficiency = $Q/(Q+S)$ in overhead-dominated regime
- [ ] Can solve **Priority + Round Robin** hybrid scheduling (same-priority processes RR among themselves)
- [ ] MLQ: **fixed** queues, **no movement** between queues, starvation risk for low-priority queues
- [ ] MLFQ: processes **can move** (promote/demote), aging solves starvation, defined by 5-6 parameters
- [ ] Can distinguish MLQ vs MLFQ crisply (movement = the key difference)
- [ ] Know the **Algorithm Diagnostic Matrix** (starvation risk & best use-case per algorithm)
- [ ] Can derive RR's guarantee formula: $q = \frac{t-ns}{n}$ (exact) vs $q \le \frac{t-ns}{n-1}$ (at-least guarantee) — and know *why* the denominators differ
- [ ] Comfortable with **SRTF + I/O burst interleaving** problems (process alternating CPU/I/O chunks)
- [ ] Practiced dual-processor non-preemptive priority scheduling (2 CPUs, 2 independent priority orders)
- [ ] Aware that non-preemptive scheduling can sometimes benefit from **intentional CPU idling** for a better overall schedule

---
*Compiled from GeeksforGeeks GATE CS & IT — Principles of Operating Systems, Lecture 11: CPU Scheduling (Round Robin, Priority+RR, Multilevel Queue Scheduling).*
