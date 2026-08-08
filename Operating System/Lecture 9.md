# CPU Scheduling — GATE CS/IT Revision Notes (Lecture 9)
*Source: Principles of Operating Systems — Lecture 9 (GeeksforGeeks GATE)*

## 📑 Topics Covered

| # | Topic |
|---|-------|
| 1 | Need for CPU Scheduling (recap) |
| 2 | FCFS (First-Come, First-Serve) Scheduling |
| 3 | Convoy Effect — Drawback of FCFS |
| 4 | SJF (Shortest Job First) — Non-preemptive |
| 5 | SRTF (Shortest Remaining Time First) — Preemptive SJF |
| 6 | SJF/SRTF Performance, Optimality & Starvation |
| 7 | Practical Limitation of SJF/SRTF |
| 8 | Key Formulas (TAT, WT, RT) |
| 9 | Worked Numericals (Gantt charts) |
| 10 | GATE PYQs & Diagnostic Checks |
| 11 | Quick Revision Checklist |

> **Note:** "Need for CPU Scheduling", "Scheduling Criteria" and "Terms & Concepts" were listed as topics but this lecture's actual walkthrough dives straight into the **algorithms** (FCFS → SJF → SRTF). Criteria/terms (AT, BT, CT, TAT, WT, RT) are picked up implicitly through the formulas and numericals below.

---

## 1. FCFS (First-Come, First-Serve) Scheduling

- **Simplest** CPU scheduling algorithm — the process that requests the CPU **first** gets it **first**.
- Implemented using a simple **FIFO queue** (Tail → Head, first in – first out), just like people queuing at a door.
- **Selection criteria:** Arrival Time (AT)
- **Mode:** Non-preemptive

### 🧠 Mnemonic
Think of FCFS as a **single queue at a door** — whoever joined the line first walks through first, no cutting in line regardless of how long they take inside.

### ⚠️ Drawback: Convoy Effect
- If a **long process** arrives first, all shorter processes behind it must wait unnecessarily long — like a convoy of cars stuck behind one slow truck.
- This **inflates average Waiting Time (WT) and Turnaround Time (TAT)**, even though total CPU work done is the same.

---

## 2. SJF (Shortest Job First) — Non-Preemptive

- **Selection criteria:** Burst Time (BT) — shortest job runs next.
- **Mode:** Non-preemptive (once started, a process runs to completion).
- **Tie-breaking rule:** Arrival Time → Process ID (earlier AT wins; if still tied, lower PID wins).

### 🧠 Mnemonic
SJF = "**shortest customer served first**" at a counter — but once someone starts being served, they finish even if a faster customer walks in.

---

## 3. SRTF (Shortest Remaining Time First) — Preemptive SJF

- **Selection criteria:** Burst Time (BT) — but based on **remaining** burst time.
- **Mode:** Preemptive — a running process is **kicked off the CPU** the moment a new process arrives with a smaller remaining burst time than what's left of the current process.
- This is literally "SJF made preemptive."

---

## 4. Comparison Table — FCFS vs SJF vs SRTF

| Feature | FCFS | SJF (Non-preemptive) | SRTF (Preemptive) |
|---|---|---|---|
| Selection criteria | Arrival Time | Burst Time | Remaining Burst Time |
| Preemptive? | ❌ No | ❌ No | ✅ Yes |
| Optimal for Avg. WT/TAT? | ❌ No | ✅ Yes (among non-preemptive) | ✅ Yes (overall optimal) |
| Suffers from Convoy Effect? | ✅ Yes | Reduced | Minimal |
| Suffers from Starvation? | ❌ No | ✅ Yes (long jobs) | ✅ Yes (long jobs) |
| Practically implementable as-is? | ✅ Yes | ❌ No (needs BT in advance) | ❌ No (needs BT in advance) |
| Throughput | Lower (with long jobs first) | Higher | Highest |

---

## 5. Performance, Optimality & Starvation (SJF/SRTF)

- Both SJF and SRTF **favor shorter processes**, which is why they:
  - **Maximize throughput** (more jobs finish per unit time `T`)
  - **Minimize average Waiting Time and TAT**
- This makes SJF/SRTF the theoretically **optimal scheduling algorithms** for minimizing average waiting time — this is a **benchmark** used to judge other algorithms.
- **Drawback:** **Starvation** — longer processes may wait indefinitely if shorter processes keep arriving.

### ⚠️ Practical Limitation
- SJF/SRTF are **non-implementable in their pure form** in a real OS, because:
  > Burst times of processes are **not known a priori** — the OS cannot know in advance how long a process will run before it actually finishes.
- In practice, burst time is only *predicted* (e.g., via exponential averaging in the Aging/Priority-based approaches covered elsewhere).

---

## 6. Key Formulas

| Formula | Meaning |
|---|---|
| $TAT = CT - AT$ | **Turnaround Time** = Completion Time − Arrival Time (total time spent in the system) |
| $WT = TAT - BT$ | **Waiting Time** = Turnaround Time − Burst Time (time spent waiting in ready queue) |
| $WT = CT - AT - BT$ | Equivalent expanded form of WT |
| $RT = (\text{Time of first CPU allocation}) - AT$ | **Response Time** — time until the process gets the CPU for the *first* time (relevant for preemptive scheduling) |

**Variables:**
- $AT$ = Arrival Time
- $BT$ = Burst Time (CPU service time required)
- $CT$ = Completion Time
- $TAT$ = Turnaround Time
- $WT$ = Waiting Time
- $RT$ = Response Time

---

## 7. Worked Examples

### Example 1 — FCFS & the Convoy Effect

| Process | AT | BT |
|---|---|---|
| P1 | 0 | 20 |
| P2 | 0 | 2 |
| P3 | 0 | 3 |
| P4 | 0 | 1 |
| P5 | 0 | 2 |

**FCFS in arrival order (P1→P2→P3→P4→P5):**

```
CPU: | P1 | P2 | P3 | P4 | P5 |
     0    20   22   25  26   28
```

| Process | CT | TAT = CT−AT | WT = TAT−BT |
|---|---|---|---|
| P1 | 20 | 20 | 0 |
| P2 | 22 | 22 | 20 |
| P3 | 25 | 25 | 22 |
| P4 | 26 | 26 | 25 |
| P5 | 28 | 28 | 26 |

**Average TAT = 121/5 = 24.2** | **Average WT = 93/5 = 18.6** ✅ (matches slide)

**Same processes, better order (P4→P5→P2→P3→P1) — illustrates the Convoy Effect fix:**

```
CPU: | P4 | P5 | P2 | P3 | P1 |
     0    1    3    5    8    28
```

| Process | CT | TAT | WT |
|---|---|---|---|
| P4 | 1 | 1 | 0 |
| P5 | 3 | 3 | 1 |
| P2 | 5 | 5 | 3 |
| P3 | 8 | 8 | 5 |
| P1 | 28 | 28 | 8 |

**Average TAT = 45/5 = 9** | **Average WT = 17/5 = 3.4** ✅ (matches slide)

> 🔑 **Takeaway:** Same processes, same total work — just re-ordering (short jobs first) crashed the average TAT from 24.2 → 9 and average WT from 18.6 → 3.4. This is exactly the **motivation for SJF**.

---

### Example 2 — SJF (Non-preemptive), all arrive at t=0 except one

| Process | AT | BT |
|---|---|---|
| P1 | 0 | 4 |
| P2 | 0 | 5 |
| P3 | 0 | 3 |
| P4 | 1 | 1 |

**Selection criteria: BT | Mode: Non-preemptive | Tie-break: AT → PID**

```
CPU: | P3 | P4 | P1 | P2 |
     0    3    4    8   13
```
- At t=0: P1(4), P2(5), P3(3) available → shortest is **P3**, runs 0→3.
- At t=3: P4 (arrived at t=1, BT=1) waiting → shorter than P1(4)/P2(5) → **P4** runs 3→4.
- At t=4: P1(4) < P2(5) → **P1** runs 4→8.
- At t=8: only **P2** left → runs 8→13.

---

### Example 3 — SJF with staggered arrivals (idle gaps)

| Process | AT | BT |
|---|---|---|
| P1 | 2 | 2 |
| P2 | 6 | 3 |
| P3 | 4 | 4 |
| P4 | 5 | 2 |
| P5 | 3 | 1 |
| P6 | 8 | 1 |

```
CPU: | idle | P1 | P5 | P4 | P2 | P6 | P3 |
     0      2    4    5    7    10   11   15
```
- 0–2: no process has arrived → **idle**
- t=2: only P1 ready → runs 2→4
- t=4: P5(BT1) vs P3(BT4) ready → **P5** runs 4→5
- t=5: P4(BT2) vs P3(BT4) → **P4** runs 5→7
- t=7: P2(BT3) vs P3(BT4) → **P2** runs 7→10
- t=10: P6(BT1) vs P3(BT4) → **P6** runs 10→11
- t=11: only P3 left → runs 11→15

---

### Example 4 — SJF with big idle gaps (SJF ≡ FCFS here)

| Process | AT | BT |
|---|---|---|
| P1 | 15 | 1 |
| P2 | 16 | 2 |
| P3 | 4 | 5 |
| P4 | 10 | 3 |

```
CPU: | idle | P3 | idle | P4 | idle | P1 | P2 |
     0      4    9      10   13     15   16   18
```

> Since only **one process is ever ready at a time**, SJF and FCFS give the **identical** schedule here.

---

### Example 5 — Processes with I/O bursts (CPU burst → I/O → CPU burst)

| Process | AT | 1st CPU Burst | I/O Time | 2nd CPU Burst |
|---|---|---|---|---|
| P1 | 0 | 4 | 10 | 2 |
| P2 | 0 | 2 | 5 | 1 |
| P3 | 1 | 1 | 3 | 2 |
| P4 | 1 | 2 | 1 | 3 |

```
CPU: | P2 | P3 | P4 | P1 | P2 | P3 | P4 | idle | P1 |
     0    2    3    5    9    10   12   15     19   21
```
- First CPU bursts run in arrival order: P2(0-2), P3(2-3), P4(3-5), P1(5-9).
- Each process then goes to I/O (multiple devices → I/O runs in parallel, off-CPU).
- Second CPU bursts are served **as I/O completes and CPU becomes free**:
  - P2's I/O(5) finishes at 7, but CPU busy till 9 → P2 runs 9→10
  - P3's I/O(3) finishes at 6 → runs 10→12
  - P4's I/O(1) finishes at 6 → runs 12→15
  - CPU **idle 15→19** waiting for P1's I/O(10), started at 9, to finish at 19
  - P1 runs 19→21

> 🔑 Illustrates how **I/O-bound processes overlap I/O and CPU usage** — a key idea behind multiprogramming and later feeding into I/O-aware scheduling.

---

### Example 6 — Why Preemption Helps (SRTF vs Non-preemptive SJF)

| Process | AT | BT |
|---|---|---|
| P1 | 0 | 10 |
| P2 | 2 | 6 |

**SRTF (Preemptive):**
```
CPU: | P1 | P2 | P1 |
     0    2    8    16
```
- P1 runs 0→2 (remaining BT = 8). At t=2, P2 arrives with BT=6 < 8 → **preempt** P1.
- P2 runs fully 2→8 (nothing shorter arrives).
- P1 resumes, runs 8→16 (remaining 8 units).

**Non-preemptive SJF (for comparison):**
```
CPU: | P1 | P2 |
     0    10   16
```
- P1 must run to completion (0→10) since it started first; P2 waits till 10, runs 10→16.

| Metric | SRTF | Non-preemptive SJF |
|---|---|---|
| P2's Completion Time | 8 | 16 |
| P2's Waiting Time | 0 | 8 |

> 🔑 **Takeaway:** Preemption (SRTF) lets a **late-arriving short job** jump the queue instead of waiting behind a long job already running — reducing its wait time from 8 to 0.

---

### Example 7 — Full SJF vs SRTF Comparison with all metrics

| Process | AT | BT |
|---|---|---|
| P1 | 4 | 1 |
| P2 | 2 | 3 |
| P3 | 3 | 5 |
| P4 | 1 | 6 |

**SJF (Non-preemptive):**
```
CPU: | idle | P4 | P1 | P2 | P3 |
     0      1    7    8    11    16
```

| Process | CT | TAT | WT | RT |
|---|---|---|---|---|
| P4 | 7 | 6 | 0 | 0 |
| P1 | 8 | 4 | 3 | 3 |
| P2 | 11 | 9 | 6 | 6 |
| P3 | 16 | 13 | 8 | 8 |

**Average TAT = 8** | **Average WT = 4.25** | **Average RT = 4.25**
(RT = WT here because non-preemptive → first allocation = only allocation)

**SRTF (Preemptive):**
```
CPU: | idle | P4 | P2 | P1 | P4 | P3 |
     0      1    2    5    6    11    16
```
- t=1: only P4 ready → runs.
- t=2: P2(BT3) arrives, remaining P4 = 5 → preempt, P2 runs.
- t=4: P1(BT1) arrives; remaining P2 = 1 (tie) → **no preemption on tie**, P2 continues.
- t=5: P2 completes → P1(BT1) shortest → runs 5→6.
- t=6: remaining P3(5) vs remaining P4(5) — **tie** → tie-break by earlier AT → **P4** (AT=1) runs before P3 (AT=3): 6→11.
- t=11: only P3 left → runs 11→16.

| Process | CT | TAT | WT | RT |
|---|---|---|---|---|
| P4 | 11 | 10 | 4 | 0 |
| P2 | 5 | 3 | 0 | 0 |
| P1 | 6 | 2 | 1 | 1 |
| P3 | 16 | 13 | 8 | 8 |

**Average TAT = 7** | **Average WT ≈ 3.25** | **Average RT = 2.25**

> 🔑 SRTF beats SJF on **every** metric here (lower TAT, WT, RT) — consistent with SRTF being the theoretically optimal scheduler.

---

### Example 8 — FCFS with I/O phases (GATE-style, δ = 0 no overhead)

> *Three tasks P1, P2, P3 arrive together (AT=0) with service times 10, 20, 30. Each spends 20% of service time on I/O, 70% on CPU, and the last 10% on I/O before completion. Multiple I/O devices are available.*

| Process | Service Time | I/O₁ (20%) | CPU (70%) | I/O₂ (10%) |
|---|---|---|---|---|
| P1 | 10 | 2 | 7 | 1 |
| P2 | 20 | 4 | 14 | 2 |
| P3 | 30 | 6 | 21 | 3 |

**Reasoning:**
- All 3 processes start I/O₁ **simultaneously** at t=0 (multiple I/O devices → runs in parallel).
- I/O₁ completes: P1 at t=2, P2 at t=4, P3 at t=6 → this is also their **FCFS order for the CPU**.
- CPU idle 0→2 (nothing ready yet), then FCFS on the CPU:

```
CPU: | idle | P1 | P2 | P3 |
     0      2    9    23    44
```
(P1: 2→9 [7 units], P2: 9→23 [14 units], P3: 23→44 [21 units])

- I/O₂ for each process starts right after its CPU burst (parallel device):
  - P1: I/O₂ (1 unit) from 9→10 → **CT₁ = 10**
  - P2: I/O₂ (2 units) from 23→25 → **CT₂ = 25**
  - P3: I/O₂ (3 units) from 44→47 → **CT₃ = 47**

**(i) Average TAT & WT** (AT = 0 for all, so TAT = CT):

| Process | TAT | WT = TAT − ST |
|---|---|---|
| P1 | 10 | 0 |
| P2 | 25 | 5 |
| P3 | 47 | 17 |

**Average TAT = 82/3 ≈ 27.33** | **Average WT = 22/3 ≈ 7.33**

**(ii) CPU Idleness %**
- Total makespan = 47 units. CPU busy = 7 + 14 + 21 = 42 units.
- **CPU idle = 47 − 42 = 5 units → Idle % = 5/47 ≈ 10.6%**

> 💡 **Extension in lecture:** the same problem is redone assuming a **context-switch/dispatch overhead δ = 1 unit** before every scheduler dispatch (IO ↔ CPU transitions). Adding this overhead **pushes the total completion time from 47 → 52**, further increasing idle time and average TAT — illustrating that **context-switch overhead is a real, non-trivial cost** that must be added on top of pure computation/I-O time in scheduling analysis.

---

## 8. GATE PYQs

### PYQ 1 (Numerical Answer Type) — Preemptive SRTF

> Processes with arrival time and CPU burst (ms). Scheduling: **Preemptive SRTF**.

| Process | AT | BT |
|---|---|---|
| P1 | 0 | 10 |
| P2 | 3 | 6 |
| P3 | 7 | 1 |
| P4 | 8 | 3 |

**Gantt chart:**
```
CPU: | P1 | P2 | P3 | P2 | P4 | P1 |
     0    3    7    8    10   13    20
```
- P1 runs 0→3 alone.
- t=3: P2(BT6) arrives, remaining P1=7 > 6 → preempt, P2 runs.
- t=7: P3(BT1) arrives, remaining P2=2 > 1 → preempt, P3 runs 7→8 (completes).
- t=8: P4(BT3) arrives; remaining P2=2 < 3 → P2 resumes, runs 8→10 (completes).
- t=10: P4(3) < remaining P1(7) → P4 runs 10→13.
- t=13: only P1 left, remaining 7 → runs 13→20.

| Process | CT | TAT = CT−AT |
|---|---|---|
| P1 | 20 | 20 |
| P2 | 10 | 7 |
| P3 | 8 | 1 |
| P4 | 13 | 5 |

**Average TAT = (20+7+1+5)/4 = 33/4 = 8.25 ms** ✅

---

### PYQ 2 (MCQ) — SRTF vs Non-preemptive SJF, Average Waiting Time

> Single-processor system, four processes A, B, C, D — (AT, BT): A(0,10), B(2,6), C(4,3), D(6,7).
> Which option gives the correct average waiting times for preemptive **SRTF** and non-preemptive **SJF**?

**Options:**
- A) SRTF = 6, NP-SJF = 7
- **B) SRTF = 6, NP-SJF = 7.5** ✅ **(Correct Answer)**
- C) SRTF = 7, NP-SJF = 7.5
- D) SRTF = 7, NP-SJF = 8.5

**Working — NP-SJF:**
```
CPU: | A | C | B | D |
     0   10   13  19   26
```
WT: A=0, C=6, B=11, D=13 → **Avg WT = 30/4 = 7.5**

**Working — SRTF:**
```
CPU: | A | B | C | B | D | A |
     0   2   4   7    11   18   26
```
WT: A=16, B=3, C=0, D=5 → **Avg WT = 24/4 = 6**

**Why the other options are wrong:**
- **A)** NP-SJF value (7) is wrong — correct non-preemptive WT computation gives 7.5, not 7.
- **C)** SRTF value (7) is wrong — correct preemptive computation gives 6, not 7.
- **D)** Both values (7 and 8.5) are wrong — neither matches the correctly computed WTs (6 and 7.5).

---

## 9. Quick Diagnostic Checks (Concept MCQs)

**Q1. In FCFS scheduling, the average waiting time is...**
- A) Always minimum
- B) Always maximum
- **C) Highly dependent on the process arrival order** ✅
- D) Independent of arrival time

*Why others are wrong:* A and B are false — FCFS gives **neither** the guaranteed minimum (that's SJF's territory) **nor** always the maximum. D is false by definition — FCFS's entire schedule *is* determined by arrival order, so it cannot be "independent" of it.
> If a heavy CPU-bound process arrives first, wait time skyrockets for all subsequent processes (Convoy Effect).

**Q2. Which CPU scheduling algorithms may suffer from Starvation?**
- A) FCFS
- B) SJF (Non-preemptive)
- C) SRTF (Preemptive SJF)
- **D) Both B & C** ✅

*Why others are wrong:* A is wrong — FCFS serves everyone strictly in arrival order, so no process is ever indefinitely skipped. B and C individually are each *partially* correct but incomplete, since **both** algorithms share the same root cause of starvation.
> Any algorithm that strictly prioritizes shortest jobs (or highest static priority) risks leaving long/low-priority jobs waiting indefinitely.

---

## 10. Memory Tricks / Mnemonics Recap

- **FCFS** = a single line at a door (Tail → Head, FIFO) — no cutting in line.
- **Convoy Effect** = a long, slow truck stuck at the front of a highway convoy — everyone behind crawls.
- **SJF** = shortest customer served first at a counter, but once you're being served, you finish (non-preemptive).
- **SRTF** = SJF's "impatient" cousin — will interrupt you mid-service if someone with less remaining work walks in.
- **SJF/SRTF optimality vs practicality** = they're the "perfect exam answer key" — mathematically optimal, but **you can't know the future** (burst times) in a real system, so they can't be directly implemented.

---

## 11. Quick Revision Checklist ✅

- [ ] Can state FCFS's selection criteria (AT) and mode (non-preemptive)
- [ ] Can explain the **Convoy Effect** and why reordering processes reduces avg. TAT/WT
- [ ] Can compute Gantt chart, CT, TAT, WT for FCFS given AT & BT
- [ ] Can state SJF's selection criteria (BT), mode (non-preemptive), and tie-break rule (AT → PID)
- [ ] Can construct SJF Gantt charts handling staggered arrivals + idle gaps
- [ ] Can state SRTF's selection criteria (remaining BT) and that it's preemptive
- [ ] Can trace SRTF preemption points step-by-step and identify tie-break behavior
- [ ] Know why SRTF is always ≤ SJF in avg. WT/TAT (preemption helps late-arriving short jobs)
- [ ] Know that SJF/SRTF are **optimal** (max throughput, min avg WT/TAT) but suffer from **starvation**
- [ ] Know the **practical limitation**: burst times aren't known in advance → SJF/SRTF not directly implementable
- [ ] Know all three formulas: $TAT = CT-AT$, $WT = TAT-BT$, $RT$ = time of first CPU allocation − AT
- [ ] Can solve I/O-overlap FCFS numericals (multiple I/O devices, %-split burst times)
- [ ] Can compute **CPU idle %** = idle time / total makespan
- [ ] Understand impact of adding **context-switch overhead (δ)** on total time/idle time
- [ ] Can solve both GATE PYQs above independently (SRTF avg TAT; SRTF vs NP-SJF avg WT)

---

*Compiled from GeeksforGeeks GATE — Principles of Operating Systems, Lecture 9 (CPU Scheduling Part II).*
