# Memory Management — Lecture 21 (GATE CS/IT — Principles of Operating Systems)

> Source: GeeksforGeeks GATE CS&IT — OS Lecture 21 (Memory Management III + Deadlock Review)

## 📌 Topics Covered (Table of Contents)

| # | Topic |
|---|---|
| 1 | Memory Hierarchy & Memory Subsystem |
| 2 | Abstract View of Memory (Address/Word/Word-size) |
| 3 | Addressing Formulas & Worked Numericals |
| 4 | Memory Hardware — RAM/ROM Chip Interface (Address bus, Data bus, CS, RD, WR) |
| 5 | CPU–Memory Interaction (MAR, MDR, Registers, Bus) |
| 6 | Multi-Chip Memory & Address Decoding |
| 7 | Static vs Dynamic Loading/Linking |
| 8 | Deadlocks — Concept Review (Necessary conditions, Prevention, Avoidance) |
| 9 | GATE PYQs — Memory Addressing Numericals |
| 10 | GATE PYQs — Deadlock (Prevention, Avoidance, Banker's Algorithm, RAG) |
| 11 | Quick Revision Checklist |

---

## 1. Memory Hierarchy & Memory Subsystem

Memory is organized as a **pyramid** — as you go down, capacity increases but speed decreases and cost/bit decreases.

```
Level 0  → CPU Registers        (fastest, smallest, priciest)
Level 1  → Cache Memory (SRAM)
Level 2  → Main Memory (DRAM)
Level 3  → Magnetic Disk (Disk Storage)
Level 4  → Optical Disk
Level 5  → Magnetic Tape        (slowest, largest, cheapest)
```

### Detailed characteristics

| Level | Volatility | Use case | Speed / Cost |
|---|---|---|---|
| Processor Registers | Volatile | Near-instantaneous use | Very fast, very expensive |
| Processor Cache | Volatile | Immediate term use | Very fast, expensive |
| RAM (Random Access Memory) | Volatile | Very short term use | Fast, affordable |
| Flash / USB memory | Non-volatile | Short–longer term use | Slower, fairly cheap |
| Hard Disk Drives | Non-volatile | Long term use | Slower, very cheap |
| Tape Backup Media | Volatile* (per slide) | Very long term use | Very slow, very cheap |

**Simplified access flow:**
`CPU ⇄ Register ⇄ Cache Memory ⇄ Main Memory ⇄ Secondary Memory`

> 💡 **Memory trick:** As you move down the pyramid — **Speed ↓, Cost/bit ↓, Capacity ↑**.

---

## 2. Abstract View of Memory

Memory can be visualized as a **linear 1-dimensional array of words**.

- Each location has a unique **address** (0 to N−1).
- Words are also called **locations / cells**.
- A word can hold either an **instruction** or **data**.
- **Word size (m)** = fixed length of each word, measured in bits.

```
Address    Memory
  0     [ ......... ]
  1     [ ......... ]
  2     [ ......... ]
  3     [ ......... ]
  .           .
  .           .
 N-1    [ ......... ]
```

---

## 3. Addressing Formulas & Worked Numericals

### 🔑 Core Formulas

$$n = \text{Address bits} = \lceil \log_2 N \rceil$$

$$N = \text{Number of words} = 2^n$$

$$m = \text{Word size / word length (bits)}$$

**Where:**
- `n` = number of **address lines** required
- `N` = total number of **addressable words/locations**
- `m` = number of **bits per word** (word size), also = number of **data lines**

### Powers of 2 — Quick Reference Table

| Power | Value | Approx |
|---|---|---|
| $2^5$ | 32 | — |
| $2^6$ | 64 | — |
| $2^7$ | 128 | — |
| $2^8$ | 256 | — |
| $2^9$ | 512 | — |
| $2^{10}$ | 1024 | ≈ $10^3$ = **1K** |
| $2^{20}$ | — | ≈ $10^6$ = **1M** |
| $2^{30}$ | — | ≈ $10^9$ = **1G** |
| $2^{40}$ | — | ≈ $10^{12}$ = **1T** |

### Worked Examples — Finding `n` given `N`

| # | Given | Working | Answer |
|---|---|---|---|
| 1 | N = 8 W | $n = \log_2 8 = 3$ | **n = 3 bits** |
| 2 | N = 256 W | $n = \log_2 256 = 8$ | **n = 8 bits** |
| 3 | N = 16 GB | $n = 4 + 30 = 34$ (4 bits for 16, 30 bits for G) | **n = 34 bits** |
| 4 | N = 256 MW | — | **n = 28 bits** |
| 5 | n = 39 bits | reverse: N = $2^{39}$ | **N = 512 GW** |
| 6 | N = 500 GB | (500 ≈ $2^9$, +30 for G) | **n = 39 bits** |
| 7 | N = 4096 W = 4×1024 ≈ $2^{12}$ | $n=12$ | **n = 12 bits** |
| 8 | N = logX Bytes | — | **n = log(logX) bits** |

**3-bit example (N=8) address encoding:**
```
000 → 0     100 → 4
001 → 1     101 → 5
010 → 2     110 → 6
011 → 3     111 → 7
```

### Worked Example — Full System Sizing

**Given:** `n = 27 bits` (address), `m = 16 bits` (word size), 1 Word = 2 Bytes

$$N_W = 2^{27} = 128\,MW$$
$$N_B = 128M \times 2B = 256\,MB$$
$$N_{Bits} = 256M \times 8\,bits = 2^8 \times 2^{20} \times 2^3 = 2^{31} = 2\,Gbit$$

### Worked Example — Word Size Given in Bits

**Given:** `N = 512T bits` total memory; divided into words of size **64 bits (8 Bytes)**

$$N_W = \frac{512T\,bits}{64\,bits} = \frac{2^{49}}{2^6} = 2^{43} = 8\,TW$$
$$N_B = 8TW \times 8B = 64\,TB$$

> 💡 **Memory trick:** Always convert everything to powers of 2 first — GATE numericals almost always reduce to clean exponents.

---

## 4. Memory Hardware — RAM/ROM Chip Interface

### CPU-level block diagram
```
        Memory
       ↑      ↑
  Control   ALU ← Accumulator
   Unit       ↑↓
              Input / Output
```
`Control Unit` and `ALU` together form the **Processor**.

### Address Bus / Data Bus relationship

- **Address Bus** = `n` bits → selects **1 of N = 2ⁿ** memory locations (unidirectional, CPU → Memory)
- **Data Bus** = `m` bits → carries the word to/from memory (bidirectional)
- Relationship: **N ∝ m** is WRONG intuition to have — actually **N depends on n** (address lines), **m** is just the word width.

```
Address Bus (n-bit) →  ┌─────────┐
      Read       →     │   RAM   │  ⇄ Data Bus (m-bit)
      Write      →     │  (N×m)  │
   Chip Select   →     └─────────┘
```

### Example: 128×8 RAM chip

- `128 × 8` RAM → 128 locations, each 8 bits wide → **128 Bytes**
- Needs a **7-bit address line** (since $2^7 = 128$) → `AD7`
- Needs an **8-bit data bus**
- Control signals: `CS1`, `CS2` (chip select), `RD` (read), `WR` (write)

### RAM Chip Truth Table (Control Logic)

| CS1 | CS2 (active-low) | RD | WR | Memory Function | State of Data Bus |
|---|---|---|---|---|---|
| 0 | 0 | × | × | Inhibit | High-impedance |
| 0 | 1 | × | × | Inhibit | High-impedance |
| 1 | 0 | 0 | 0 | Inhibit | High-impedance |
| 1 | 0 | 0 | 1 | **Write** | Input data to RAM |
| 1 | 0 | 1 | × | **Read** | Output data from RAM |
| 1 | 1 | × | × | Inhibit | High-impedance |

> 💡 The chip is **only active** when `CS1=1` and `CS2=0` (i.e., $\overline{CS2}=1$). Read/Write then decide direction.

### ROM Chip Example: 512×8 ROM
- 9-bit address (`AD9`), since $2^9 = 512$
- 8-bit data bus (read-only, no `WR` pin)

### Address-Space Partitioning across Multiple Chips

Given 4× `RAM (128×8)` + 1× `ROM (512×8)` sharing an 11-bit address bus (bits 10–1):

| Component | Hex Address Range | Bit 10 | Bit 9 | Bit 8 | Bits 7–5(x) | Bits 4-1(x) |
|---|---|---|---|---|---|---|
| RAM 1 | 0000–007F | 0 | 0 | 0 | x | x |
| RAM 2 | 0080–00FF | 0 | 0 | 1 | x | x |
| RAM 3 | 0100–017F | 0 | 1 | 0 | x | x |
| RAM 4 | 0180–01FF | 0 | 1 | 1 | x | x |
| ROM | 0200–03FF | 1 | x | x | x | x |

A **decoder** (using the higher-order address bits, e.g., bits 10, 9, 8) generates the `CS` signals for each chip — this is how the CPU's single address bus talks to **multiple** memory chips without conflict.

---

## 5. CPU–Memory Interaction (Internal Registers)

```
                Main Memory
                ↑      ↕      ↕
              MAR      MDR    Control
              (addr)  (data)
   ┌──────────────────────────────┐
   │  PC   R0                     │
   │  IR   R1     ALU             │
   │       ...                    │
   │       Rn-1                   │
   │        N general-purpose     │
   │           registers          │
   └──────────────────────────────┘
                 CPU
```

| Register | Full form | Role |
|---|---|---|
| **MAR** | Memory Address Register | Holds the address to be accessed in memory |
| **MDR** | Memory Data Register | Holds the data being read/written |
| **PC** | Program Counter | Holds address of next instruction |
| **IR** | Instruction Register | Holds the currently executing instruction |
| **R0…Rn-1** | General Purpose Registers | Temporary operand/result storage |

> 💡 **Memory trick:** MAR talks to the **address bus**, MDR talks to the **data bus** — "Address goes out through MAR, Data goes in/out through MDR."

---

## 6. Buses connecting CPU, Memory, and I/O

```
 CPU  ⇄  Memory ⋯ Memory  ⇄  I/O ⋯ I/O
   │________Control Lines_________│
   │________Address Lines_________│  → collectively called the "BUS"
   │__________Data Lines___________│
```

All devices (CPU, multiple memory chips, multiple I/O devices) share these **3 sets of lines**: Control, Address, Data.

---

## 7. Static vs Dynamic Loading / Linking

| Aspect | **Static** | **Dynamic** |
|---|---|---|
| Memory utilization | Inefficient utilization of memory | Efficient space utilization |
| Execution speed | Faster execution (everything pre-loaded) | Program execution becomes slower (loaded on demand) |

### Illustrative Example
Program on disk = 60 KB total, broken into functions:
```
main() { 10 KB
   if(cond)
      f();     // f() is 20 KB, calls g()
}

f() { 20 KB
   g();        // g() is 5 KB, calls h()
}

g() { 5 KB
   h();        // h() is 25 KB, calls scanf()
}

h() { 25 KB
   scanf();
}
```
- **Static loading**: All 60 KB loaded into memory upfront regardless of whether `if(cond)` is true → wastes memory if branch not taken, but no runtime loading delay.
- **Dynamic loading**: Only `main()` (10 KB) loaded initially; `f()`, `g()`, `h()` loaded **only when actually called** → saves memory but adds loading overhead during execution.

> 💡 **Memory trick:** Static = "load everything now, run fast later." Dynamic = "load only what's needed, save space but pay a runtime loading cost."

---

## 8. Deadlocks — Concept Review

### 4 Necessary Conditions for Deadlock (Coffman conditions)
1. **Mutual Exclusion** — resource held in non-sharable mode
2. **Hold and Wait** — process holds a resource while waiting for another
3. **No Preemption** — resource cannot be forcibly taken away
4. **Circular Wait** — a cycle of processes each waiting for a resource held by the next

> ⚠️ **"Preemption" is NOT a necessary condition** — it is the *absence* of preemption (No Preemption) that is required for deadlock. If preemption is allowed, deadlock is prevented.

### Circular Wait
> A process holds a resource and waits for another resource which is held by another process, **forming a cycle** — NOT just "waiting for a resource it already holds."

### Resource Allocation Graph (RAG) & Cycles
- A cycle in RAG ⟹ **deadlock only if each resource type has exactly ONE instance**.
- If a resource type has **multiple instances**, a cycle is **necessary but not sufficient** for deadlock.

### Prevention Strategies (breaking one of the 4 conditions)

| Condition | How to Eliminate |
|---|---|
| Hold and Wait | Require all resources to be requested **at once** (before execution) |
| Circular Wait | Impose a **linear/total ordering** of all resource types; request resources in increasing order |
| No Preemption | **Force** a process to release resources (preemption) |
| Mutual Exclusion | Cannot typically be eliminated for non-sharable resources |

> ⚠️ **Mutual Exclusion and Circular Wait can theoretically both be targeted, but the standard testable pair for prevention is Hold-and-Wait + Circular Wait ("Both B and C").**

- **Requesting all resources at once** (to prevent hold-and-wait) ⟹ leads to **low resource utilization** (resources reserved but idle) — this is the trade-off cost.
- **Preemption** (as a prevention technique) = **forcing a process to release resources** it's holding (not killing it, not blocking it forever).

### Safe State
A system is in a **Safe State** if there **exists a sequence** of all processes such that each process can get its maximum resource need satisfied (finish) using currently available + resources released by processes ahead of it in the sequence.

### Deadlock Avoidance
A method that **ensures the system never enters an Unsafe State** = **Deadlock Avoidance** (e.g., Banker's Algorithm) — distinct from Prevention (which eliminates one of 4 conditions structurally) and Detection (which lets deadlock happen, then detects+recovers).

> 💡 **Memory trick:**
> - **Prevention** = don't let the *conditions* for deadlock ever arise.
> - **Avoidance** = let conditions arise, but never grant a request that leads to an *unsafe* state.
> - **Detection & Recovery** = let it happen, then find & fix it.

---

## 9. GATE PYQs — Memory Addressing Numericals

| # | Question | Options | ✅ Answer | Reasoning |
|---|---|---|---|---|
| 1 | 16-bit address bus → max addressable memory size? | A) 64 KB B) 16 KB C) 128 KB D) 32 KB | **A) 64 KB** | $2^{16} = 65536\,B = 64KB$; others miscompute the exponent-to-KB conversion. |
| 2 | Address lines needed to access 1 MB memory? | A) 10 B) 16 C) 20 D) 24 | **C) 20** | $1MB = 2^{20}B$, so 20 address lines needed. |
| 3 | Byte-addressable memory, 12 address lines → memory capacity? | A) 1 KB B) 2 KB C) 4 KB D) 8 KB | **C) 4 KB** | $2^{12} = 4096\,B = 4KB$. |
| 4 | Address lines needed to address $2^n$ locations? | A) n B) 2n C) n/2 D) $n^2$ | **A) n** | By definition, $N=2^n$ needs exactly $n$ address lines. |
| 5 | Memory has 1024 words, each 8 bits → size in bytes? | A) 1 KB B) 2 KB C) 512 B D) 8 KB | **A) 1 KB** | $1024\,words \times 1\,Byte/word$ (since 8 bits = 1 Byte) $= 1024B = 1KB$. |
| 6 | 32-bit address bus → max addressable memory space? | A) $2^{32}$ bytes B) 4 GB C) Both A and B D) 32 GB | **C) Both A and B** | $2^{32}\,bytes = 4\,GB$ — both expressions describe the same quantity. |
| 7 | Memory system is 4K×8 organization → address lines required? | A) 10 B) 12 C) 13 D) 14 | **B) 12** | $4K = 2^{12}$, so 12 address lines (N) and 8 data lines (m). |
| 8 | Effective memory size of a system is determined by? | A) Data bus width only B) Address bus width only C) Processor speed/config D) Control bus width | **B) Address bus width only** | Address bus width (n) fixes $N=2^n$, the number of addressable locations — this determines the *size*, data bus only fixes word width. |
| 9 | Memory capacity = words × bits/word. Address & data lines for 4K×16? | A) 10 addr, 16 data B) 11 addr, 8 data C) 12 addr, 16 data D) 12 addr, 12 data | **C) 12 address, 16 data lines** | $4K=2^{12}$ locations → 12 address lines; 16 bits/word → 16 data lines. |
| 10 | 4 GB max memory, word-addressable, word = 2 bytes → size of address bus? | A) At least 28 bits B) At least 2 bytes C) At least 31 bits D) Minimum 4 bytes | **C) At least 31 bits** | $N_W = \dfrac{4GB}{2B} = 2GW = 2^{31}$ words ⟹ need 31 address lines. |

---

## 10. GATE PYQs — Deadlock Concepts & Numericals

### Conceptual PYQs

**Q1. Which of the following is NOT a valid Deadlock Prevention scheme?**
- A) Release all resources before requesting a new resource. ✔ Valid
- B) Number all resources uniquely and never request a lower-numbered resource than the last one requested. ✔ Valid
- C) **Never request a resource after releasing any resource.** ❌ **NOT valid**
- D) Request and be allocated all required resources before execution. ✔ Valid

✅ **Answer: C**
**Reasoning:** (A), (B), (D) are all legitimate hold-and-wait/circular-wait prevention schemes. (C) is nonsensical/not a recognized valid scheme — it doesn't structurally prevent any of the 4 conditions.

---

**Q2. OS requires a process to release all resources before requesting another resource. Which is TRUE?**
- A) Both starvation and deadlock can occur.
- **B) Starvation can occur but deadlock cannot occur.** ✅
- C) Starvation cannot occur but deadlock can occur.
- D) Neither starvation nor deadlock can occur.

**Reasoning:** This policy breaks **Hold-and-Wait**, so deadlock (which needs all 4 conditions) **cannot** occur. But a process could still be repeatedly denied resources by others (starvation) — so starvation is still possible.

---

**Q3. 6 Tape Drives, n processes competing, each process needs (up to) 2 drives. Max value of n for guaranteed deadlock-free system?**

Using **Max Claim / Deadlock-avoidance formula:**
$$n_{max} = \frac{TotalDrives - NoOfProcesses}{PerProcessMax - 1} \quad \text{style reasoning (Min–Max)}$$

Working shown: $TD = 6$, each process needs 2 drives max.
- Min(n) to **cause** deadlock = 6 (if each of 6 processes holds 1 drive and waits for the 2nd)
- **Max(n) for guaranteed deadlock freedom = 5**

✅ **Answer: B) 5**
Trial: $P_1..P_5$ each hold 1 drive (5 drives used, 1 free) — the free drive lets at least one process finish and release, cascading to completion — always safe.

---

**Q4. 3 user processes, each requiring 2 units of resource R. Minimum units of R such that no deadlock ever arises?**

$$Max(R)_{deadlock} = 3 \quad (\text{if each process holds 1 and waits for another → deadlock})$$
$$Min(R)_{deadlock\ free} = 4$$

✅ **Answer: C) 4**
With 4 units: even if each of the 3 processes holds 1 unit (using 3 units), 1 unit remains free, letting some process complete → deadlock-free.

---

**Q5. 6 Tape Drives, n processes, each process may need 3 tape drives. Max n for guaranteed deadlock-free system?**

$$TD = 6,\quad P_i \to 3(TD)\ \text{max each}$$

Trial with $n=2$: $P_1$–2, $P_2$–2 (using formula pattern from lecture) → deadlock-free guaranteed.

✅ **Answer: A) 2**

---

**Q6. m resources of same type, shared by 3 processes A, B, C with peak demands 3, 4, 6. For what value(s) of m will deadlock NOT occur?**

Using the standard deadlock-avoidance bound:
$$m \geq \sum (\text{peak}_i - 1) + 1 = (3-1)+(4-1)+(6-1)+1 = 2+3+5+1 = 11$$

So deadlock will **not** occur for **m ≥ 11**.

Options: A)7 B)9 C)10 D)13 E)15

✅ **Answer: D) 13 and E) 15** (both ≥ 11; multi-select style question)
**Reasoning:** 7, 9, 10 are all < 11 → deadlock possible. 13 and 15 are ≥ 11 → guaranteed deadlock-free.

---

**Q7. Snapshot of a system with n processes; process $i$ holds $x_i$ instances of resource R (all instances currently occupied); process $i$ additionally requests $y_i$ more instances. Exactly two processes $p, q$ have $y_p = y_q = 0$. Which is a necessary condition to guarantee the system is NOT approaching deadlock?**

- A) $\min(x_p, x_q) < \max_{k \neq p,q} y_k$
- **B) $x_p + x_q \geq \min_{k \neq p,q} y_k$** ✅
- C) $\max(x_p, x_q) > 1$
- D) $\min(x_p, x_q) > 1$

**Reasoning (derivation):**
| PID | Alloc (R) | Req (R) | Avail (R) |
|---|---|---|---|
| $P_1$ | $x_1$ | $y_1$ | 0 |
| ⋮ | ⋮ | ⋮ | ⋮ |
| $P_p$ | $x_p$ | 0 | — |
| $P_q$ | $x_q$ | 0 | — |
| ⋮ | ⋮ | ⋮ | ⋮ |
| $P_n$ | $x_n$ | $y_n$ | — |

- If $x_p + x_q \geq \min_{k\neq p,q} y_k$ → the freed resources from $p,q$ finishing can satisfy at least one other waiting process → **deadlock-free**.
- If $x_p + x_q < \min_{k\neq p,q} y_k$ → no other process can proceed even after $p,q$ finish → **deadlock**.

---

**Q8. RAG-based question:** Given a Resource Allocation Graph with resources $r_1, r_2, r_3$ and processes $P_0$–$P_3$, determine if system is deadlocked.

Constructed table:

| PID | Alloc ($r_1,r_2,r_3$) | Req ($r_1,r_2,r_3$) | Avail ($r_1,r_2,r_3$) |
|---|---|---|---|
| $P_0$ | 1 0 1 | 0 1 1 | 0 0 1 |
| $P_1$ | 1 1 0 | 1 0 0 | |
| $P_2$ | 0 1 0 | 0 0 1 | |
| $P_3$ | 0 1 0 | 1 2 0 | |

✅ **Result: System is SAFE (not deadlocked).**
**Safe sequence: ⟨P2, P0, P1, P3⟩**

---

**Q9. Banker's Algorithm — 3 resource types X, Y, Z, processes P0, P1, P2. System currently in a Safe State. Available: X=3, Y=2, Z=2.**

| Process | Alloc (X,Y,Z) | Max (X,Y,Z) | Need (X,Y,Z) |
|---|---|---|---|
| P0 | 0,0,1 | 8,4,3 | 8,4,2 |
| P1 | 3,2,0 | 6,2,0 | 3,0,0 |
| P2 | 2,1,1 | 3,3,3 | 1,2,2 |

**Requests:**
- **REQ1:** P0 requests (0,0,2)
- **REQ2:** P1 requests (2,0,0)

- REQ1: needs (0,0,2) — Need[P0]=(8,4,2) ✓ ≥ req; Avail(3,2,2) ≥ (0,0,2) ✓ → tentatively grant → new Avail=(3,2,0) → check safety → **fails** (leads to unsafe state) ❌
- REQ2: needs (2,0,0) — Need[P1]=(3,0,0) ✓ ≥ req; Avail(3,2,2) ≥ (2,0,0) ✓ → grant → new Avail=(1,2,2) → **remains safe** ✅

✅ **Answer: B) Only REQ2 can be permitted.**

---

**Q10. [Homework — not solved live]** System with n processes $\langle P_1,\dots,P_n\rangle$, each allocated $x_i$ and requesting $y_i$ copies of resource R. Exactly 2 processes A, B have zero request. k instances of R are free. Find condition for "not approaching deadlock" (min request satisfiable) and total instances of R.
> 📝 *Left as self-practice in the lecture.*

---

**Q11. A system has 3 programs, each requiring 3 tape units. Minimum tape units such that deadlock never arises?**

$$P_1: 2,\ P_2: 2,\ P_3: 2 + 1 = 7$$

✅ **Answer: 7**
(Standard formula: $\sum(\text{max}_i - 1) + 1 = (3-1)\times3 + 1 = 7$.)

---

**Q12. System shares 9 tape drives. Current allocation & max requirement:**

| Process | Current Allocation | Max Requirement |
|---|---|---|
| P1 | 3 | 7 |
| P2 | 1 | 6 |
| P3 | 3 | 5 |

Total allocated = 3+1+3 = 7, Available = 9−7 = 2.

- **Deadlocked?** Check if any process's remaining need ≤ available (2): P3 needs 5−3=2 ≤ 2 → P3 can finish → releases 3 → available becomes 5 → P1 needs 7−3=4 ≤5 → finishes → etc. So **not deadlocked**.
- **Safe?** A completion sequence exists (P3 → P1 → P2 or similar) → **Safe**.

✅ **Answer: B) Safe, Not Deadlocked**

---

**Q13. Which of the following statements is/are TRUE about deadlocks?**
- **A) Circular wait is a necessary condition for the formation of deadlock.** ✅ TRUE
- B) In a system where each resource has more than one instance, a cycle in wait-for graph indicates deadlock. ❌ FALSE (only *necessary*, not sufficient, with multiple instances)
- C) If current allocation leads system to unsafe state, deadlock will necessarily occur. ❌ FALSE (unsafe ≠ deadlock; deadlock is *possible*, not guaranteed)
- **D) In RAG, if every edge is an assignment edge, system is NOT in deadlock state.** ✅ TRUE

✅ **Answer: A and D**

---

**Q14. [Homework — not solved live]** 3 resource types E, F, G; 4 processes P0–P3; Allocation & Max matrices given; 3 instances of E and 3 instances of F available.
> 📝 *Left as self-practice — apply Banker's safety algorithm to check safe state / answer sub-question.*

---

**Q15. A multithreaded program P executes with x threads and y non-reentrant locks. If a thread can't acquire a lock, it blocks. Minimum (x, y) together for which execution of P CAN result in a deadlock?**

- A) x=1, y=2
- B) x=2, y=1
- **C) x=2, y=2** ✅
- D) x=1, y=1

**Reasoning:** Deadlock (circular wait) needs **at least 2 threads** (x≥2) each holding one lock and waiting for the other's lock — needs **at least 2 locks** (y≥2) too. With only 1 thread or 1 lock, no circular wait is possible (a single non-reentrant lock held+re-requested by the same thread is a self-deadlock scenario too, but the minimal *classic* answer here is x=2, y=2).

---

## 11. Quick Revision Checklist ✅

**Memory Hierarchy & Basics**
- [ ] Memory hierarchy pyramid (Registers → Cache → RAM → Disk → Optical → Tape)
- [ ] Volatility, speed, and cost trade-offs at each level
- [ ] CPU ⇄ Register ⇄ Cache ⇄ Main Memory ⇄ Secondary Memory flow

**Addressing**
- [ ] Abstract view: memory = linear array of words
- [ ] Formula: $n = \lceil \log_2 N \rceil$, $N = 2^n$
- [ ] Difference between address bits (n), word size (m), number of words (N)
- [ ] Converting between bytes/KB/MB/GB/TB using powers of 2
- [ ] Solving "given N find n" and "given n find N" style numericals

**Memory Hardware**
- [ ] RAM/ROM chip pin interface: Address bus, Data bus, Read, Write, Chip Select
- [ ] RAM truth table (CS1/CS2/RD/WR → Read/Write/Inhibit/High-Z)
- [ ] Multi-chip memory systems & address decoding using a decoder
- [ ] CPU internal registers: MAR (address), MDR (data), PC, IR, general-purpose registers
- [ ] Bus structure: Control lines, Address lines, Data lines shared among CPU/Memory/I/O

**Loading/Linking**
- [ ] Static vs Dynamic loading trade-offs (memory efficiency vs execution speed)

**Deadlocks**
- [ ] 4 necessary conditions: Mutual Exclusion, Hold & Wait, No Preemption, Circular Wait
- [ ] Preemption is NOT a necessary condition (No Preemption is)
- [ ] Circular wait = cycle of processes holding+waiting resources
- [ ] RAG cycle ⟹ deadlock only when single instance per resource type
- [ ] Prevention: eliminate Hold-and-Wait and/or Circular Wait (most common exam target)
- [ ] Cost of "request all resources at once": low resource utilization
- [ ] Preemption (as prevention) = forcibly releasing resources from a process
- [ ] Safe State definition (sequence exists such that all processes can finish)
- [ ] Deadlock Avoidance = never enter unsafe state (e.g., Banker's Algorithm)
- [ ] Min/Max resource-count numericals (tape drives, generic resource R) — formula: $\sum(\text{max}_i - 1) + 1$
- [ ] Banker's Algorithm safety check for granting requests (check Need vs Available, then re-verify safety)
- [ ] RAG-based deadlock detection via Allocation/Request/Available tables + safe sequence
- [ ] Deadlock in multithreaded programs (min threads/locks for circular wait)

---

*Notes compiled from GeeksforGeeks GATE CS&IT — Principles of OS, Lecture 21 (Memory Management III).*
