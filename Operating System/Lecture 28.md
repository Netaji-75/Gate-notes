# Virtual Memory — GATE CS/IT Revision Notes (Lecture 28)

> Source: Principles of Operating Systems — Lecture 28 (GeeksforGeeks GATE Series)

## 📑 Table of Contents

| # | Topic |
|---|-------|
| 1 | What is Virtual Memory |
| 2 | Page Replacement Algorithms (FIFO, Optimal, LRU, MRU, LFU/MFU) |
| 3 | LRU Approximation Algorithms (Reference bit, Aging, Second Chance, Enhanced/NRU) |
| 4 | Belady's Anomaly |
| 5 | Thrashing (causes, vicious cycle) |
| 6 | Locality of Reference — Case Studies |
| 7 | Thrashing Control: Working Set Model |
| 8 | Thrashing Control: Page Fault Frequency (PFF) |
| 9 | Disk Structure Basics |
| 10 | Formula Cheat-Sheet |
| 11 | GATE PYQs / Practice Questions |
| 12 | Memory Tricks & Mnemonics |
| 13 | Quick Revision Checklist |

---

## 1. What is Virtual Memory (VM)

- VM is an illusion of a large, contiguous address space given to a process, backed by **physical memory (RAM)** + **disk (swap/backing store)**.
- Only the actively-needed portion of a process resides in physical frames; the rest sits on disk until referenced (**Demand Paging**).
- Enabled by: **Page Table**, **TLB (fast path)**, **Eviction Engine (page replacement algorithm)**, and **Thrashing Watchdogs (WSM/PFF)**.

**Unified mental model (blueprint):**
```
Virtual Space → Translation Pipeline (TLB fast path / Page Table slow path) → Physical RAM (frames)
                                                                                  ↕ Swap In/Out
                                                                                Disk (backing store)
Thrashing Watchdogs (WSM/PFF) monitor fault rate & working-set window, and
trigger the Eviction Engine (LRU/Clock) or process suspension when needed.
```

---

## 2. Page Replacement Algorithms

**Goal (Eviction Dilemma):** Memory is full → to bring in a new page, an old one must be evicted. Choose the page **not needed again for the longest time**. Evicting a **modified/dirty** page costs extra — it must be written back to disk first (double I/O penalty).

### 🔢 Master Reference String (used across FIFO/LRU/MRU/LFU/MFU examples)

```
7, 0, 1, 2, 0, 3, 0, 4, 2, 3, 0, 3, 2, 1, 2, 0, 1, 7, 0, 1
```

### 2.1 FIFO (First-In-First-Out)
- Evicts the **oldest loaded** page, regardless of usage.
- Low implementation overhead, but suffers from **Belady's Anomaly** (see §4).

### 2.2 Optimal (OPT)
- Evicts the page **not needed for the longest time in the future**.
- Gives the **theoretical minimum** number of page faults — used as a benchmark.
- **Impossible to implement in practice** (needs future knowledge / clairvoyance).
- Zero risk of Belady's Anomaly (it's a **stack algorithm**).

### 2.3 LRU (Least Recently Used) — Worked Example

**With 3 frames** → simulate on the master reference string:

| Ref | 7 | 0 | 1 | 2 | 0 | 3 | 0 | 4 | 2 | 3 | 0 | 3 | 2 | 1 | 2 | 0 | 1 | 7 | 0 | 1 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Fault? | F | F | F | F | · | F | · | F | F | F | F | · | · | F | · | F | · | F | · | · |

- Total **page faults = 12**
- Final memory state: **{1, 0, 7}**

**With 4 frames** → same reference string:

| Ref | 7 | 0 | 1 | 2 | 0 | 3 | 0 | 4 | 2 | 3 | 0 | 3 | 2 | 1 | 2 | 0 | 1 | 7 | 0 | 1 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Fault? | F | F | F | F | · | F | · | F | · | · | · | · | · | F | · | · | · | F | · | · |

- Total **page faults = 8**

> ✅ **Key exam insight:** LRU is a **stack algorithm** → increasing frames (3→4) *always* reduced faults here (12 → 8). This monotonic behavior is guaranteed for stack algorithms (LRU, Optimal) — **never** for FIFO.

### 2.4 MRU (Most Recently Used)
- Evicts the page used **most recently** (opposite intuition of LRU) — useful in specific access patterns (e.g., repeated full-table scans).
- On the master reference string with **3 frames**: lecture result → **16 page faults**, final memory state **{7, 0, 2}**.

### 2.5 Counting-Based Algorithms

| Algorithm | Idea | Result (3 frames, master ref. string) |
|---|---|---|
| **LFU** (Least Frequently Used) | Evict page with **smallest reference count** | 13 page faults |
| **MFU** (Most Frequently Used) | Evict page with **largest reference count** | 15 page faults |

Final frame/count snapshot from lecture: Page 2 (count 1), Page 0 (count 2), Page 1 (count 1).

---

## 3. LRU Approximation Algorithms

True LRU needs perfect timestamps/counters — expensive in hardware. Real OSes **approximate** it using a hardware **Reference bit (R)**.

### 3.1 Reference Bit (R) — the basic building block
- `R = 0` → page **not referenced** during the current epoch.
- `R = 1` → page **was referenced** during the current epoch.
- Periodically (epoch boundary), OS clears all R bits and re-checks later.

### 3.2 Additional Reference Bits — "Aging" Algorithm
- Each page keeps an **8-bit history register** (R₁…R₈).
- Every clock tick: **shift the register right**, insert the current R bit at the **MSB**, then clear R.
- To pick a victim: interpret each register as a binary number — the page with the **smallest value** was least recently used → evict it.

**Example (from lecture):**

| Process | Register (R₁…R₈) | Decimal value |
|---|---|---|
| Pᵢ | 1 1 0 1 0 1 1 1 | 215 |
| Pⱼ | 0 0 1 1 1 1 1 1 | 63 |
| Pₖ | 1 0 1 0 1 0 1 1 | 171 |

→ **Pⱼ has the smallest value (63)** → it is the LRU-approximation victim (**Pₓ**).

### 3.3 Second Chance / Clock Algorithm
- Essentially **FIFO + R bit check**.
- Sweep pages in FIFO (load-time / clock-hand) order.
- If the oldest page's `R = 1` → **give it a second chance**: clear R, and move it to the back of the queue (as if just loaded).
- If `R = 0` → evict it immediately.
- **Criterion used:** `(Time-of-Load + R)`

**Example table (page table with f, V/I, P/NP, TOL, R):**

| Page | Frame | V/I | P/NP | TOL (Time of Load) | R (before) | R (after 2nd-chance pass) |
|---|---|---|---|---|---|---|
| 0 (a) | 1 | 1 | 1 | 4 | 1 | 0 (2nd chance given) |
| 1 (c) | 1 | 1 | 1 | 3 | 1 | 0 (2nd chance given) |
| 2 (b) | 1 | 1 | 1 | 1 | 0 | evicted (R was already 0) |
| 3 | 1 | 1 | 0 | — | — | — (invalid entry) |
| 4 (d) | 1 | 1 | 1 | 0 | 1 | 0 → becomes candidate |
| 5 (e) | 1 | 1 | 1 | 2 | 0 | evicted |

### 3.4 Enhanced Second Chance / NRU (Not Recently Used)
- Uses **both R (referenced) and M (modified/dirty) bits** → 4 priority classes.
- **Criterion:** `(R, M)` pair.

| Class | R | M | Meaning | Eviction priority |
|---|---|---|---|---|
| 0 | 0 | 0 | Not referenced, not modified | **Best candidate — evict first** (no I/O cost) |
| 1 | 0 | 1 | Not referenced, but modified | Evict next (needs write-back) |
| 2 | 1 | 0 | Referenced, not modified | Avoid evicting |
| 3 | 1 | 1 | Referenced & modified | **Worst candidate — evict last** (costly + likely needed) |

### 📊 Page Replacement Diagnostic Matrix

| Algorithm | Core Logic | Implementation Overhead | Belady's Anomaly Risk |
|---|---|---|---|
| **FIFO** | Evicts oldest page | Low | **High Risk** |
| **Optimal (OPT)** | Evicts page unneeded for longest future time | Impossible (needs clairvoyance) | Zero Risk |
| **LRU** | Evicts page unused for longest time (past) | High (counters/stacks) | Zero Risk |
| **Second Chance (Clock)** | FIFO + R-bit check, gives a "second chance" | Medium | Moderate Risk |

---

## 4. Belady's Anomaly

> **The Paradox:** Intuitively, "more RAM = fewer page faults." But for **some algorithms (notably FIFO)**, on certain access patterns, **increasing the number of frames can *increase* the number of page faults**.

- **Stack algorithms** (LRU, Optimal) are **immune** to Belady's Anomaly — the set of pages held with `k` frames is always a **subset** of the pages held with `k+1` frames.
- FIFO is **not** a stack algorithm → susceptible.

---

## 5. Thrashing

**Definition:** *Thrashing ≡ the system spends more time paging (swapping data to/from disk) than executing actual user processes.*

### The Vicious Cycle
```
Low CPU utilization
      ↓
OS thinks: "CPU is idle, add more processes!" (increases degree of multiprogramming)
      ↓
More processes competing for the same limited frames
      ↓
Less memory per process → more page faults
      ↓
Even lower CPU utilization → cycle repeats
```

### The Inflection Point of Multiprogramming
- As degree of multiprogramming increases, CPU utilization rises — **up to a point**.
- Beyond that point (the "OS limit"), adding **one more process** starves *all* existing processes of required frames → CPU utilization **collapses** → **Thrashing Zone**.

### Reasons for Thrashing

**Primary causes:**
1. **Lack of frames** (low memory)
2. **High degree of multiprogramming**

**Other contributing factors:**
3. Page replacement policy (poor choice can accelerate thrashing)
4. Page size (too small/large affects fault rate)
5. Program & data structure (poor locality of reference)

---

## 6. Locality of Reference — Case Studies

### Case Study I: Row-major array traversal
Given `integer A[1..128][1..128]`, Page Size = 128 words, Row-Major Order (RMO) storage.

| Code | Access pattern | Page Faults (1 frame available) |
|---|---|---|
| `for i=1..128: for j=1..128: A[j][i] = 1;` (column-major access on row-major storage) | Jumps rows on every access → **poor locality** | **128 × 128 = 16,384 (16K)** |
| `for i=1..128: for j=1..128: A[i][j] = 1;` (row-major access, matches storage) | Sequential within a row → **excellent locality** | **128** |

> 🔑 **Takeaway:** Always access multi-dimensional arrays in the **same order** they are stored (row-major storage → iterate row-by-row) to minimize page faults.

### Structure comparisons (locality-friendliness)

| Comparison | Locality-Friendly (Good for paging) | Locality-Unfriendly (Bad for paging) |
|---|---|---|
| Arrays vs Linked List | **Arrays** (contiguous, sequential access) | **Linked Lists** (nodes scattered, pointer chasing) |
| Linear Search vs Binary Search | **Linear Search** (sequential scan) | **Binary Search** (jumps across memory, poor locality) |

---

## 7. Thrashing Control — Strategy Overview

```
Thrashing Control Strategies
        │
        ├── Prevention → Control the degree of multiprogramming (via Long-Term Scheduler, LTS)
        │
        └── Detection & Recovery → Monitor: Low CPU util. + High degree of MP + High paging-disk util.
                                    → Action: Process Suspension (via Medium-Term Scheduler, MTS)
```

### 7.1 The Working-Set Model (Resolution I)

- **Principle of Locality:** processes move from one *locality* (a group of pages) to another over time.
- **Δ (Working-Set Window):** a fixed number of the *most recent* page references (e.g., 10,000 instructions).
- **WSSᵢ(t)** = Working Set Size of process Pᵢ = number of **unique pages** referenced in the most recent Δ references. It **varies over time**.
  - If Δ too small → doesn't capture the entire locality.
  - If Δ too large → captures several localities at once.
  - If Δ = ∞ → captures the entire program.
- **D = Σ WSSᵢ** = total demand for frames across all `n` processes.
- **Rule:** If `D > m` (available physical frames) → system is heading toward **thrashing** → **suspend a process**.

#### 🧮 Worked Example — Working Set Computation

> **Given:** Page reference string = `c c d b c e c e a d`, **Δ = 4**.
> Initial working set at `t = 0` = `{a, d, e}`, where `a` was referenced at `t=0`, `d` at `t=-1`, `e` at `t=-2`.

Full reference timeline:

| t | -2 | -1 | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Ref | e | d | a | c | c | d | b | c | e | c | e | a | d |

**Step-by-step working sets (window = most recent 4 references) and faults:**

| t | Working Set WS(t) | Size | Fault? |
|---|---|---|---|
| 0 | {e, d, a} | 3 | (initial) |
| 1 | {e, d, a, c} | 4 | **Fault** (c wasn't in WS(0)) |
| 2 | {d, a, c} | 3 | Hit |
| 3 | {a, c, d} | 3 | Hit |
| 4 | {c, d, b} | 3 | **Fault** (b wasn't in WS(3)) |
| 5 | {d, b, c} | 3 | Hit |
| 6 | {d, b, c, e} | 4 | **Fault** (e wasn't in WS(5)) |
| 7 | {b, c, e} | 3 | Hit |
| 8 | {c, e} | 2 | Hit |
| 9 | {c, e, a} | 3 | **Fault** (a wasn't in WS(8)) |
| 10 | {c, e, a, d} | 4 | **Fault** (d wasn't in WS(9)) |

- **Total Page Faults = 5** (at t = 1, 4, 6, 9, 10)
- **Average number of page frames used** = average of WS sizes across t=0..10:
  `(3+4+3+3+3+3+4+3+2+3+4) / 11 = 35 / 11 ≈ 3.18 frames`

### 7.2 Page-Fault Frequency (PFF) Strategy (Resolution II)

- Define an **Upper bound (U)** and **Lower bound (L)** for acceptable page-fault rates.
- If fault rate **> U** → process needs **more frames**; give it more, or **suspend** it if none are available.
- If fault rate **< L** → process is **over-allocated**; take frames away and give them to other processes.
- Goal: keep the **resident set size close to the working set size (W)**.
- **Trigger:** Based purely on live/measured fault rate (reactive, not predictive — unlike WSM which uses reference history).

### Working-Set Model vs PFF — Quick Comparison

| Aspect | Working-Set Model | Page-Fault Frequency (PFF) |
|---|---|---|
| Trigger | Page reference **history** (locality) | Live **fault rate** (hardware counters) |
| Mechanism | Tracks exactly which pages are in window Δ | Adjusts frame allocation using Upper/Lower bounds |
| Pros/Cons | Highly accurate to program behavior, but **complex** (needs timestamping every reference) | Easier to implement, but purely **reactive**, not predictive |

---

## 8. Disk Structure Basics (brief)

- **File System** sits between the OS and raw storage devices (USB disk, hard disks, optical disks); user accesses information as files/directories, OS retrieves/stores it.
- **Hard Disk components:** platter, spindle (rotates the platters), read/write head, arm + arm assembly (actuator), circuit board, power/data ports.
- **Addressing terms:**
  - **Track (t):** a concentric ring on a platter.
  - **Sector (s):** a pie-slice division of a track (smallest addressable unit).
  - **Cylinder (c):** the set of tracks at the same radius across all platters (accessible without moving the arm).
  - **Cluster:** group of sectors treated as one allocation unit by the file system.

---

## 9. Formula Cheat-Sheet

| Formula | Meaning |
|---|---|
| $WSS_i(t) = $ unique pages referenced in most recent $\Delta$ references | Working Set Size of process $i$ at time $t$ |
| $D = \sum_{i=1}^{n} WSS_i$ | Total frame demand across $n$ processes |
| If $D > m$ (available frames) $\Rightarrow$ **Thrashing** | Trigger for suspending a process |
| Number of pages in address space $= \dfrac{\text{Address Space Size}}{\text{Page Size}}$ | Used to compute bits for frame number / page table entries |
| Page Table Size (bytes) $=$ (Number of entries) $\times$ (PTE size in bytes) | Total memory occupied by the page table |
| PTE size $= f + V/I + R + M + \text{Protection} + \text{Other attributes}$ (in bits) | Break-down of a Page Table Entry |

**Worked micro-example (bits breakdown):**
Given: V.A.S = P.A.S = $2^{16}$ Bytes, Page Size = 512 B ($=2^9$), PTE = 32 bits, with 1 bit V/I, 1 bit Reference, 1 bit Modified, 3 bits Protection.

- Frame number bits $f = \log_2(P.A.S / \text{Page Size}) = \log_2(2^{16}/2^9) = \log_2(2^7) = 7$ bits
- Used bits so far $= 7 (f) + 1(V/I) + 1(R) + 1(M) + 3(\text{Protection}) = 13$
- **Other attribute bits** $x = 32 - 13 = \mathbf{19}$ bits

---

## 10. GATE PYQs / Practice Questions

### Q1. FIFO + increasing frame count → effect on page faults?
**Options:** (A) Always decrease (B) Always increase (C) Sometimes increase (D) Never affect
**✅ Correct: (C) Sometimes increase the number of page faults**
*Reasoning:* This is exactly **Belady's Anomaly** — FIFO is not a stack algorithm, so more frames can *sometimes* cause more faults, but not always. (A) and (B) are absolutes, both false since anomaly only appears for *some* patterns; (D) is false since anomaly clearly shows an effect.

---

### Q2. A heavily-used variable initialized very early is removed from memory when?
**Options:** (A) LRU used (B) FIFO used (C) LIFO used (D) None of the above
**✅ Correct: (B) FIFO page replacement algorithm is used**
*Reasoning:* FIFO only looks at **load time**, not usage — so an old-but-frequently-used page can still get evicted purely for being "old." LRU (A) would never evict a heavily-used page (it tracks recency of use, not load time). LIFO (C) is not a standard/valid page replacement policy.

---

### Q3. Belady's Anomaly and Random vs LRU
**S1:** Random page replacement suffers from Belady's anomaly.
**S2:** LRU page replacement suffers from Belady's anomaly.
**Options:** (A) S1 T, S2 T (B) S1 F, S2 T (C) S1 T, S2 F (D) S1 F, S2 F
**✅ Correct: (C) S1 is true, S2 is false**
*Reasoning:* Random replacement is **not a stack algorithm** → can suffer the anomaly (S1 True). LRU **is** a stack algorithm → **immune** to Belady's anomaly (S2 False).

---

### Q4. True/False — Indicate all FALSE statements
- (A) The amount of virtual memory available is limited by the availability of secondary storage. → **True**
- (B) Any implementation of a critical section requires the use of an indivisible micro-instruction, such as test-and-set. → **False** ❌ (monitors/semaphores can implement critical sections without directly requiring test-and-set at the programmer level)
- (C) The use of a monitor ensures that no deadlock will be caused. → **True** (in the sense taught in this course; monitors structure mutual exclusion safely)
- (D) The LRU page replacement policy may cause thrashing for some type of programs. → **True** (even the "best approximation" policy can thrash if the process's working set genuinely exceeds available frames)

**✅ Only (B) is FALSE.**

---

### Q5. Match the Pairs

| List – I | List – II |
|---|---|
| 1. Temporal Locality | a. Virtual Memory |
| 2. Spatial Locality | b. Shared Memory |
| 3. Address Translation | c. Look-ahead Buffer |
| 4. Mutual Exclusion | d. Look-aside Buffer |

**Likely intended answer: (A) a–3, b–4, c–1, d–2**
*Reasoning (memory aid):* **Address Translation ↔ Look-aside Buffer** is the most certain pairing (TLB = *Translation Look-aside Buffer*). **Mutual Exclusion ↔ Shared Memory** makes sense since concurrent access to shared memory needs mutual exclusion.
> ⚠️ Note: the exact arrow-mapping in the handwritten slide was visually ambiguous for the locality pairs — treat this match as the best-supported reconstruction, and re-verify against the original source if this exact question appears in your test bank.

---

### Q6. NRU / FIFO / LRU / Second-Chance — Four Page Frames

| Page | Loaded | Last Ref | R | M |
|---|---|---|---|---|
| 0 | 126 | 279 | 0 | 0 |
| 1 | 230 | 260 | 1 | 0 |
| 2 | 120 | 272 | 1 | 1 |
| 3 | 160 | 280 | 1 | 1 |

- **(A) NRU replaces:** **Page 0** — it's the only page in Class 0 (R=0, M=0), the lowest-priority (best-to-evict) class.
- **(B) FIFO replaces:** **Page 2** — oldest `Loaded` time (120).
- **(C) LRU replaces:** **Page 1** — smallest `Last Ref` time (260), i.e., least recently used.
- **(D) Second Chance replaces:** **Page 0** — checking oldest-loaded first (Page 2, loaded=120): R=1 → give second chance, clear R, move to back. Next-oldest (Page 0, loaded=126): R=0 → **evict Page 0**.

---

### Q7. Page Table sizing
**Given:** Page table has 64 entries, each 11 bits, page size = 512 Bytes. Find size of Logical Address Space (LAS).
**✅ Answer: LAS = 64 × 512 = 32,768 Bytes = 32 KB**
*(Number of entries = number of pages; LAS = number of pages × page size. The 11-bit PTE size is a distractor — irrelevant to computing LAS.)*

---

### Q8. Sharing in a paged memory system is done by:
**Options:** (A) Giving a copy of shared pages to each process (B) Dividing program into procedures/data, sharing only procedures (C) Several page table entries pointing to the same frame in main memory (D) None of the above
**✅ Correct: (C)**
*Reasoning:* True sharing means multiple processes' page tables map **different virtual pages to the same physical frame** — no duplication of data. (A) defeats the purpose of sharing (copies waste memory). (B) is a partial/older technique, not the general mechanism used in paged systems.

---

### Q9. Good/Bad programming structures for demand-paging
| Structure | Verdict | Reasoning |
|---|---|---|
| (a) Stack | **Good** | Access is localized to the top of the stack |
| (b) Hashed symbol table | **Bad** | Hash function scatters accesses randomly → poor locality |
| (c) Sequential search | **Good** | Accesses memory in order, one page at a time |
| (d) Binary search | **Bad** | Jumps across distant memory locations → poor locality |
| (e) Pure code | **Good** | Read-only & shareable, never needs write-back |
| (f) Vector operations | **Bad** | A single operation may touch elements spread across many pages simultaneously |
| (g) Indirection | **Bad** | Pointer chasing can land on an arbitrary, unpredictable page |

---

### Q10. Improving CPU Utilization (Demand Paging)
**Given:** CPU util = 20%, Paging disk = 97.7%, Other I/O = 5% *(clear sign of thrashing — the disk is the bottleneck)*

| Option | Improves CPU Utilization? | Reasoning |
|---|---|---|
| (a) Faster CPU | **No** | Bottleneck is the disk, not CPU speed |
| (b) Bigger paging disk | **No** | Size isn't the bottleneck, speed/contention is |
| (c) Increase degree of multiprogramming | **No — makes it worse** | Already thrashing; adding processes deepens it |
| (d) Decrease degree of multiprogramming | **Yes** | Frees up frames per process, reduces faults |
| (e) Install more main memory | **Yes** | More frames → fewer faults → less disk pressure |
| (f) Faster/multiple hard disks & controllers | **Yes** | Reduces disk queueing/service time for paging |
| (g) Add prepaging | **Possibly (Yes)** | Can reduce faults if working set is predictable |
| (h) Increase page size | **Not clearly helpful / No** | Reduces fault count somewhat but increases internal fragmentation & transfer time per fault |

---

### Q11. D[128][128] Row-Major Array + LRU + 30 Frames
```c
int D[128][128];
for (int i = 0; i < 128; i++)
    for (int j = 0; j < 128; j++)
        D[j][i] *= 10;
```
**Given:** Page frame holds 512 elements = 4 rows/page (since each row has 128 ints). Total pages needed = 128 rows / 4 rows-per-page = **32 pages**. Only **30 physical frames** allocated (< 32 needed) → LRU replacement is forced.

*Reasoning:* The access order `D[j][i]` with `i` as outer loop means for each fixed `i`, the inner loop sweeps `j` from 0 to 127 — touching **all 32 pages sequentially** in one full inner-loop pass. Since the cyclic reuse distance (32 pages) **exceeds available frames (30)**, by the time a page is revisited in the next outer (`i`) iteration, it has already been evicted by LRU. This causes a page fault on **every single page transition**, for **every** outer iteration (128 times), touching 32 distinct pages each time.

**✅ Total Page Faults = 128 × 32 = 4096**

---

### Q12. LRU faults 9 & 11 (with 6 & 4 frames) → What can Optimal give?
**Options:** (A) 9 and 7 (B) 7 and 9 (C) 10 and 12 (D) 6 and 7
**✅ Correct: (B) 7 and 9**
*Reasoning:*
- Optimal (OPT) always gives **≤** faults compared to LRU for the same frame count → OPT(6-frames) ≤ 9, OPT(4-frames) ≤ 11.
- OPT is a **stack algorithm** → faults must be **monotonically non-increasing** as frames increase: OPT(6-frames) ≤ OPT(4-frames).
- (A) 9,7 → violates monotonicity (9 > 7 with more frames — impossible for a stack algorithm).
- (C) 10,12 → violates OPT(6) ≤ LRU(6)=9 (10 > 9, impossible).
- (D) 6,7 → technically doesn't violate the two rules, but (B) is the officially recognized correct answer for this well-known GATE question.

---

### 📝 Practice Problem (unsolved — try yourself!)

> Reference string: `3 5 4 3 5 6 2 5 2 3 4 2 5 4 2 7 4 7 3`, process has **3 frames**.
> For each of the following, show the memory state after each reference and mark fault/hit:
> (A) FIFO (B) LRU (C) Optimal
> (D) R-bit algorithm — ties broken by removing lower page number; R bits cleared every 4 references
> (E) Second Chance — reference bits cleared every 6 references
> (F) LFU

*(No worked solution was shown on-screen for this one — good practice exercise before your exam!)*

---

## 11. Memory Tricks & Mnemonics

- 🧠 **Thrashing, one-liner:** *"Thrashing ≡ more time spent paging than executing user processes."*
- 🧠 **Second Chance = FIFO with a conscience** — "if you were recently used, I'll give you a second chance before kicking you out."
- 🧠 **NRU/Enhanced Second Chance priority order:** *(0,0) cheapest to evict → (1,1) most expensive/valuable to keep* — think of **M (Modified)** as "cost to evict" and **R (Referenced)** as "value to keep."
- 🧠 **Stack algorithms (LRU, Optimal) = Belady-anomaly-proof.** Non-stack (FIFO, Random) = can misbehave with more memory.
- 🧠 **TLB = Translation Look-***aside*** Buffer** → helps recall the Address-Translation ↔ Look-aside-Buffer pairing in match-type questions.
- 🧠 **Working Set Window (Δ) sizing:** too small = misses locality; too large = merges localities; Δ=∞ = whole program.

---

## 12. Quick Revision Checklist

- [ ] Definition & purpose of Virtual Memory
- [ ] FIFO, Optimal, LRU, MRU — core idea of each
- [ ] LRU full worked simulation (3-frame vs 4-frame) & why frame count trend matters
- [ ] LFU vs MFU — counting-based replacement
- [ ] Reference bit (R) — basic idea
- [ ] Aging algorithm — 8-bit shift register, compare as binary numbers
- [ ] Second Chance / Clock algorithm — FIFO + R bit, criterion = TOL + R
- [ ] Enhanced Second Chance / NRU — 4 classes using (R, M)
- [ ] Belady's Anomaly — what it is, who's immune (stack algorithms) vs vulnerable (FIFO)
- [ ] Thrashing — definition, vicious cycle, inflection point of multiprogramming
- [ ] Reasons for thrashing — primary (frames, degree of MP) vs contributing (policy, page size, program structure)
- [ ] Locality of reference case studies — row-major array traversal, arrays vs linked lists, linear vs binary search
- [ ] Thrashing control: Prevention (LTS) vs Detection & Recovery (MTS, process suspension)
- [ ] Working Set Model — Δ, WSSᵢ, D = ΣWSSᵢ, D > m ⟹ thrashing; worked example computation
- [ ] Page Fault Frequency (PFF) strategy — U/L bounds, resident set ≈ W
- [ ] Working Set Model vs PFF — trigger & mechanism differences
- [ ] Disk structure basics — platter, track, sector, cylinder, spindle, arm
- [ ] Page table entry (PTE) bit breakdown & page table size calculation
- [ ] All GATE PYQs above — re-attempt without looking at answers

---

*End of Lecture 28 Notes — Virtual Memory*
