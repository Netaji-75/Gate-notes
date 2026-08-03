# Memory Management — Lecture 25 (Paging, Multilevel Paging, Inverted/Hashed Paging, Segmentation)
**GATE CS/IT — Principles of Operating Systems | Revision Notes**

> Source: GeeksforGeeks GATE CS&IT Engineering, "Principles of Operating Systems", Lecture 25.

---

## 📑 Table of Contents

| # | Topic |
|---|-------|
| 1 | Memory Hierarchy (quick recap) |
| 2 | Memory Management Techniques — Overview |
| 3 | Paging Fundamentals & Page Table Size (PTS) Formula |
| 4 | Page Size Trade-off (Internal Fragmentation vs PTS) |
| 5 | Optimal Page Size Derivation |
| 6 | Inverted Page Table |
| 7 | Hashed Page Table |
| 8 | Multilevel Paging (why, concept, 2-level/3-level translation) |
| 9 | Multilevel Paging — Advantages, Disadvantages, Performance |
| 10 | Segmentation & Segmented Paging |
| 11 | GATE PYQs & Practice Questions (fully worked) |
| 12 | Mnemonics / Remember-This Tips |
| 13 | Bonus Spillover: Synchronization Primitives (Binary vs Split Binary Semaphore) |
| 14 | Quick Revision Checklist |

---

## 1. Memory Hierarchy (Quick Recap)

| Level | Component | Trend |
|-------|-----------|-------|
| 0 | CPU Registers | ↑ Cost/bit, ↑ Speed, ↓ Capacity (top of pyramid) |
| 1 | Cache Memory (SRAM) | |
| 2 | Main Memory (DRAM) | |
| 3 | Magnetic Disk | |
| 4 | Optical Disk | |
| 5 | Magnetic Tape | ↓ Cost/bit, ↓ Speed, ↑ Capacity (bottom of pyramid) |

**Rule of thumb:** As you go **down** the hierarchy → capacity increases, but cost-per-bit and access time trends reverse (cheaper per bit, slower access).

---

## 2. Memory Management Techniques — Overview

```
Memory Management Techniques
├── Contiguous Allocation
└── Non-Contiguous Allocation
    ├── Paging
    ├── Multilevel Paging
    ├── Inverted Paging
    ├── Segmentation
    └── Segmented Paging
```

This lecture focuses on the **Non-Contiguous** branch.

---

## 3. Paging Fundamentals & Page Table Size (PTS) Formula

**Paging** = a memory management scheme that eliminates the need for contiguous allocation of physical memory. Logical Address Space (LAS) is split into equal-sized **pages**; Physical Address Space (PAS) is split into equal-sized **frames**.

### Key Formula

$$
\text{PTS} = N \times e \qquad \text{where } N = \frac{\text{LAS}}{\text{PS}}
$$

- **PTS** = Page Table Size
- **N** = number of pages/entries in the page table
- **e** = size of one Page Table Entry (PTE), in bytes
- **LAS** = size of Logical Address Space
- **PS** = Page Size

**Important relation:** $\text{PTS} \propto N \propto \dfrac{1}{\text{PS}}$
→ **Increasing page size decreases page table size** (fewer, bigger pages ⇒ fewer entries needed).

**Worked mini-example:**
Given LA = 32 bits, PS = 4 KB ⇒ PTS = 1M entries (words). If PTE size *e* = 4 Bytes ⇒ **PTS = 4 MB**.

---

## 4. Page Size Trade-off (Internal Fragmentation vs PTS)

Increasing page size shrinks the page table **but** increases internal fragmentation. Classic worked example:

**Given:** Program size = 1026 Bytes. Compare two page sizes:

| Page Size | # Pages Required | Internal Fragmentation (IF) |
|---|---|---|
| Small (S) = 2 B | 513 pages | 0 B |
| Large (L) = 1024 B | 2 pages | 1022 B |

**Takeaway:** Small page size → tiny/no internal fragmentation but huge page table. Large page size → tiny page table but large internal fragmentation. This tension motivates the "optimal page size" formula below.

---

## 5. Optimal Page Size Derivation

**Given:**
- LAS = $S$ bytes
- PTE size = $e$ bytes
- Page size = $P$ bytes (the unknown we want to optimize)

**Goal:** Choose $P$ such that **total overhead** (PTS + average Internal Fragmentation) is **minimized**.

$$
\text{PTS} = \frac{S}{P}\cdot e \qquad \text{Average IF} = \frac{P}{2}
$$

$$
\text{Total Overhead} = \frac{Se}{P} + \frac{P}{2}
$$

Differentiate w.r.t. $P$ and set to zero:

$$
\frac{d}{dP}\left(\frac{Se}{P}+\frac{P}{2}\right)=0 \;\Rightarrow\; -\frac{Se}{P^2}+\frac{1}{2}=0
$$

$$
\boxed{P = \sqrt{2Se}}
$$

📌 **This is a very commonly asked GATE formula — memorize it as-is.**

---

## 6. Inverted Page Table

### Concept
Normally, **each process** has its own page table with one entry **per virtual page**. This wastes memory if the address space is huge (mostly-unused).

**Inverted Page Table (IPT) — a.k.a. Reverse Paging:**
- There is only **ONE** page table for the **entire system** (shared across processes), not one per process.
- It has **one entry per physical frame** (not per virtual page).
- Each entry stores `⟨PID, Page#⟩` — i.e., *which process's which page* currently occupies that frame.

### Address Translation Flow

```
Logical Address: [pid | p | d]
        ↓
Search Page Table for matching (pid, p)  →  gives index i (the frame number)
        ↓
Physical Address = [i | d]
        ↓
Physical Memory
```

Since the table must be **searched by content** (linear search through `⟨pid, p⟩` pairs) rather than direct indexing, this is inherently slower than simple paging — hence **Hashed Paging** (Section 7) is used to speed up the search.

### Formula & Worked Example

**Given:** LA = 36 bits, PA = 28 bits, Page Size = 4 KB, PTE size = 32 bits (4 B).

**(a) Conventional (single-level) Page Table Size:**
$$
\text{PTS} = N\times e = \frac{2^{36}}{2^{12}}\times 4B = 2^{24}\times 4B = 2^{26}\,B = \mathbf{64\ MB}
$$

**(b) Inverted Page Table Size:**
Inverted PT has one entry **per physical frame**, so:
$$
M' = \frac{\text{PAS}}{\text{PS}} = \frac{2^{28}}{2^{12}} = 2^{16}\ \text{frames}
$$
$$
\text{Inverted PTS} = M' \times e = 2^{16}\times 4B = 2^{18}\,B = \mathbf{256\ KB}
$$

✅ **Huge saving:** 64 MB → 256 KB. This is *the* reason Inverted Page Tables exist — they scale with **physical memory size**, not virtual address space size (so they don't blow up even for 64-bit virtual addresses).

⚠️ **Limitation:** Only practical for **smaller physical memories**, since search time grows with table size (mitigated using hashing, see below).

---

## 7. Hashed Page Table

### Concept
To speed up the "search by content" problem of Inverted Paging, use a **hash function** to jump near the right bucket instead of a full linear scan.

```
Logical Address: [p | d]
        ↓
p → hash function f(p) → index i (bucket in Hash Table, size N)
        ↓
Search chain at bucket i for matching page# → get frame r
        ↓
Physical Address = [r | d]
```

- **Collisions** (two different keys hashing to same bucket) are resolved via **Chaining** (linked list at each bucket).
  - Example: keys 10 and 20 with hash = `key % 10` → both give `f(10) = 0` and `f(20) = 0` → collide → chained together.

### Formula

$$
\text{Size of Hashed Page Table} = C \times N
$$

- **C** = number of entries (buckets) in the Hash Table
- **N** = average size of the linked list (chain) per bucket

---

## 8. Multilevel Paging

### Why Do We Need It?

Single-level (simple) paging page tables can become **too large** to feasibly keep in memory for **every process**.

**Motivating numeric example:**
- 32-bit virtual address, 4 KB pages
- Number of pages = $2^{32}/2^{12} = 2^{20}$
- A single-level page table needs $2^{20}$ entries
- If each entry = 4 B → Size = $2^{20}\times 4B = \mathbf{4\ MB\ per\ process}$

→ 4 MB *per process*, resident continuously, is unacceptable at scale. **Solution: "Page the page table."**

### Concept

Instead of one giant flat table, split the virtual address into **multiple parts**, each indexing a **level** of a page-table hierarchy:

```
Virtual/Logical Address → [Outer Page # | Inner Page # | Offset]
```

- **Outer Page #** → indexes the **Outer/first-level Page Table** (a.k.a. Page Directory)
- **Inner Page #** → indexes the **Second-level Page Table**
- **Offset** → byte location within the final page

**Translation path:** `Virtual Address → Page Directory → Page Table → Physical Frame`

### 2-Level Translation Diagram

```
LA: [p1 | p2 | d]
p1 → outer-page table entry → points to a "page of the page table"
p2 → indexes into that page-of-page-table → gives frame #
d  → offset within that frame
Physical Address = [frame # | d]
```

### Worked Example (Space Savings)

**Given:** LA = 32 bits, PS = 4 KB. A process only uses:
- **Text/Code:** 2 contiguous pages `⟨P0, P1⟩`
- **Data:** 2 contiguous pages `⟨Px, Py⟩`
- **Stack:** 1 page (last page of VAS)
- *(Heap: not used in this example)*

Since VAS = 4 GB and each Outer Page Table entry covers a large chunk (via 1K inner-table entries), only a **few inner (2nd-level) tables** are actually needed:

- 1 Outer Page Table (OPT) = 1K entries → $1K \times 4B = \mathbf{4\ KB}$
- Only 3 inner-table "chunks" are actually touched (1 for code, 1 for data, 1 for stack) = $3 \times 1K \times 4B = \mathbf{12\ KB}$
- **Total actual overhead = 4 KB + 12 KB = 16 KB**

Compare this **16 KB** (multilevel, on-demand) to the **4 MB** required by a flat single-level table — this is the core benefit of multilevel paging: *unused regions of the address space cost nothing.*

### 3-Level Paging (Extension)

Same idea, one more level of indirection:

```
LA: [2nd Outer Page | Outer Page | Inner Page | Offset]
```
Used when address spaces are even larger (e.g., 64-bit systems commonly use 4–5 levels).

---

## 9. Multilevel Paging — Advantages, Disadvantages, Performance

### Advantages vs Disadvantages

| Advantages | Disadvantages |
|---|---|
| Reduces memory required for page tables | Increased memory access time (multiple lookups) |
| Page tables created only **on demand** | 2-level paging → 3 total memory accesses (2 for page tables + 1 for data) |
| Handles **sparse address spaces** efficiently | n-level paging → **n + 1** memory accesses |
| Scalable to large address spaces | Can slow the system down significantly **without a TLB** |
| Hardware-friendly (Intel, ARM support it natively) | |

### Main Objectives (Summary)
1. **Reduce Page Table Memory Overhead** — a single-level table wastes space even for unused regions.
2. **Support Sparse Address Spaces** — no entries kept for never-touched regions.
3. **Make Large Page Tables Manageable** — break one big table into directory/subdirectory/table levels.
4. **Fit Page Tables into Pages** — each table at each level is sized to exactly fit into one page frame → simplifies allocation & swapping.

### Performance Formula

For an *n*-level page table, average access needs *n+1* memory references (n table walks + 1 data access) — **unless a TLB (Translation Lookaside Buffer) is used** to cache recent translations.

**Effective Access Time (with page faults possible):**
$$
\text{EAT} = (1-p)\times t_{mem} + p \times t_{pf}
$$
- $p$ = probability of a page fault
- $t_{mem}$ = normal memory access time (e.g., 200 ns)
- $t_{pf}$ = average page fault service time (e.g., 8 ms)

### Real-World Examples

| OS/Hardware | Levels Used |
|---|---|
| Linux (x86-64) | 4-level (or 5-level on newer systems) |
| Windows | Hierarchical (multi-level) page tables |
| macOS (XNU kernel) | Multi-level |
| x86 (32-bit) | Traditionally 2-level |
| x86-64 | 4-level: PML4 → Page Directory Pointer Table → Page Directory → Page Table |
| Older/simpler systems | Single-level paging, segmentation+paging, or inverted page tables (e.g., some IBM designs) |

---

## 10. Segmentation & Segmented Paging

**Segmentation:** Divides the logical address space into **variable-sized, logically meaningful units** (segments) — e.g., code segment, data segment, stack segment — each with a `⟨Base, Length⟩` pair in a **Segment Table**.

**Address translation:**
$$
\text{Physical Address} = \text{Base}[\text{segment}] + \text{offset} \quad \text{(valid only if } \text{offset} < \text{Length})
$$

**Segmented Paging:** Combines both — each **segment** is itself paged (broken into fixed-size pages), giving benefits of both schemes (logical grouping + no external fragmentation).

**Why does the Segment Table itself sometimes need a Page Table?**
→ Because the **Segment Table can itself be too large to fit into a single page frame** — so it must be paged like any other large table. *(See PYQ in Section 11.)*

---

## 11. GATE PYQs & Practice Questions (Fully Worked)

### Q1. Does the program fit in the address space?
**Given:** Paging system, Address Space = 65,536 Bytes (64 KB), Page Size = 4096 Bytes (4 KB).
Program sections: Text = 32,768 B, Data = 16,386 B, Stack = 15,870 B. *(A page holds only ONE section's data — no mixing.)*

**Work:**
| Section | Size (B) | Pages Needed (rounded up) |
|---|---|---|
| Text | 32,768 | 32768/4096 = **8** |
| Data | 16,386 | ⌈16386/4096⌉ = **5** |
| Stack | 15,870 | ⌈15870/4096⌉ = **4** |
| **Total needed** | | **17 pages** |

Pages available in address space = 65536 / 4096 = **16 pages**.

**(a) Does it fit? → ❌ No** (needs 17 pages, only 16 available).

**(b) Max page size such that it fits?**
> ⚠️ *Note: the lecture's handwritten working for this sub-part is only partially legible in the source slide (shows intermediate values like D≈2049, S≈1984). The core method is: try increasing P and recompute* $\lceil \text{Text}/P\rceil + \lceil \text{Data}/P\rceil + \lceil \text{Stack}/P\rceil \le \text{AddressSpace}/P$ *and solve for the largest valid power-of-2 (or given) page size. Recommend re-deriving this part from the original video if you need the exact numeric answer — the method above is the exam-safe takeaway.*

---

### Q2. Conventional vs Inverted Page Table — number of entries
**Given:** 32-bit logical address, 4 KB page size, physical memory up to 512 MB.

**(a) Conventional single-level page table entries:**
$$
N = \frac{2^{32}}{2^{12}} = 2^{20} = \mathbf{1\ M\ entries}
$$

**(b) Inverted page table entries** (= number of physical frames):
$$
M' = \frac{2^{29}}{2^{12}} = 2^{17} = \mathbf{128\ K\ entries}
$$
*(512 MB = $2^{29}$ bytes)*

**Bonus — approximate size in bytes:** if each entry (page# + a few flag bits) ≈ 3 B:
$$
\text{Conventional PTS} \approx 1M \times 3B = \mathbf{3\ MB}
$$

---

### Q3. Which statement is FALSE? (Classic GATE question)

| Option | Statement | Verdict |
|---|---|---|
| A | The TLB performs an associative (parallel) search on all valid entries using the page number of the incoming VA. | ✅ True |
| B | If a VA has a TLB hit, but the subsequent search for the word is a cache miss, the word will *always* be present in main memory. | ✅ True — a TLB entry only exists for pages currently resident in memory |
| **C** | **Memory access time using a given inverted page table is always the same for all incoming virtual addresses.** | ❌ **FALSE — this is the answer.** Access time varies because the IPT is searched (linearly or via hashing/chaining), so lookup time depends on collisions/search length. |
| D | In a hashed page table system, if two distinct VAs V1 and V2 hash to the same value, the memory access time for these will *not* be the same. | ✅ True — collisions cause chain traversal, so access times differ from the no-collision case |

**Answer: C**

---

### Q4. Two-level Paging — page size, VAS size, and translation overhead
**Given:** 2-level paging. Top 9 bits → outer PT index. Next 7 bits → inner PT index. VA size = 28 bits. PTE = 32 bits at both levels.

**(i) Page size & number of pages in VAS:**
- Offset bits = 28 − 9 − 7 = **12 bits** → Page size = $2^{12}$ = **4 KB**
- Number of pages in VAS = $2^{28}/2^{12} = 2^{16}$ = **65,536 pages**

**(ii) Space overhead to translate ONE instruction/data unit:**
- Outer PT: $2^9$ entries × 4 B = 512 × 4 = **2 KB** (whole outer table must be resident)
- One Inner PT (only one is walked per access): $2^7$ entries × 4 B = 128 × 4 = **512 B**
- **Total overhead = 2 KB + 512 B = 2.5 KB (2560 Bytes)**

---

### Q5. Three-level Paging — finding uniform Page Size
**Given:** 3-level paging, uniform page size at all levels, VA = 46 bits, PTE = 32 bits (4 B) at every level. Find Page Size $P$ (power of 2) such that the **Outer Page Table exactly fits in one frame**.

**Solve:** Let offset = $p$ bits ⇒ Page size $P = 2^p$.
Each table has $P/4$ entries ⇒ index bits per level = $p-2$ (same at all 3 levels, since uniform).

$$
3(p-2) + p = 46 \;\Rightarrow\; 4p - 6 = 46 \;\Rightarrow\; p = 13
$$

**Page Size = $2^{13}$ = 8 KB.**

**Virtual Address Format:**

| Level-1 index | Level-2 index | Level-3 index | Offset |
|---|---|---|---|
| 11 bits | 11 bits | 11 bits | 13 bits |

*(Check: 11+11+11+13 = 46 ✅)*

---

### Q6. Finding number of levels $L$
**Given:** 57-bit VA, multi-level tree-structured page tables with $L$ levels, Page size = 4 KB, PTE = 8 Bytes at all levels.

**Solve:**
- Offset = $\log_2(4KB) = 12$ bits ⇒ remaining index bits = $57-12=45$
- Entries per table = PageSize/PTE = $4096/8 = 512 = 2^9$ ⇒ 9 index bits per level
- $L \times 9 = 45 \Rightarrow \mathbf{L = 5}$

---

### Q7. Minimum page table memory for a process (3-level, 39-bit VA)
**Given:** VA format = `[L1: 9b][L2: 9b][L3: 9b][Offset: 12b]`, Page size = 4 KB, PTE = 8 B at every level. Process P uses 2 GB virtual memory (contiguous from 0), mapped to 2 GB physical memory. Find **minimum** total page-table memory required across all levels.

**Solve (each table has $2^9=512$ entries, size $512\times 8B = 4KB$):**

- Pages actually used = $2^{31}/2^{12} = 2^{19}$ pages
- **Level-3 tables needed** (each covers 512 pages = 2 MB): $2^{19}/2^9 = 2^{10} = 1024$ tables → $1024\times 4KB = \mathbf{4096\ KB\ (4\ MB)}$
- **Level-2 tables needed** (each L2 table covers 512 L3 tables = 1 GB): $2GB/1GB = 2$ tables → $2\times 4KB = \mathbf{8\ KB}$
- **Level-1 (root) table:** always exactly **1** table → $\mathbf{4\ KB}$

$$
\text{Total} = 4096 + 8 + 4 = \mathbf{4108\ KB}
$$

---

### Q8. Demand-paged Outer Directory — find X + Y
**Given:** 32-bit system, 4 KB pages, PTE = 4 B, 2-level page table. OS allocates the **outer page directory** at process creation (always 1 page = 1024 entries). OS uses **demand paging** for inner page tables (an inner-table page is allocated only if it has ≥1 valid entry). A process accesses **2000 unique pages** (none swapped out). X = minimum, Y = maximum number of *page-table pages* (across both levels) after execution. Find X + Y.

**Solve:**
- Each inner table has $4KB/4B = 1024$ entries → covers 1024 pages.
- **Minimum (X):** pack all 2000 pages into as few inner tables as possible → $\lceil 2000/1024 \rceil = 2$ inner tables. Total = 1 (outer) + 2 (inner) = **3**
- **Maximum (Y):** spread the 2000 pages across as many *distinct* inner tables (groups) as possible — capped by the outer table's own 1024-entry limit → max **1024** inner tables can be activated. Total = 1 (outer) + 1024 (inner) = **1025**

$$
X + Y = 3 + 1025 = \mathbf{1028}
$$

---

### Q9. Finding PTE size $x$ (3-level demand paging)
**Given:** 32-bit VA, 3-level page tables, page size = 2 KB, PTE = $x$ bytes. Maximum total size of page table (across all levels, fully populated worst case) = 8210 KB. Find $x$.

**Method:**
- Offset bits = $\log_2(2KB) = 11$ ⇒ remaining index bits = $32-11=21$ ⇒ assuming equal split across 3 levels → 7 bits/level ⇒ 128 entries/table.
- Worst-case table counts: L1 = 1, L2 = 128, L3 = $128\times128=16384$ → total tables = 16513, each of size $128x$ bytes.
- $\text{Total} = 16513 \times 128 \times x$ bytes, set equal to $8210\times1024$ bytes and solve for $x$.

> ⚠️ *Note: solving this equation gives $x \approx 4$ Bytes (nearest clean value), but the exact numbers don't divide perfectly — worth re-checking bit-split assumptions against the original problem statement if this appears verbatim on your exam. Use the **method** shown here regardless of the exact number.*

---

### Q10. Finding $b$ (2-level hierarchical paging)
**Given:** Logical Address Space = $2^{32}$ bytes, 2-level paging, page size = 4096 B, VA = `[b-bit outer index | inner offset | page offset]`, inner PT entry = 8 B, uniform page size.

**Solve:**
- Page offset (low bits) = $\log_2(4096) = 12$ bits
- Inner PT entries per page = $4096/8 = 512 = 2^9$ ⇒ inner index = 9 bits
- Remaining (outer index) $b = 32 - 12 - 9 = \mathbf{11}$

---

### Q11. Bits needed to address next-level table/frame (3-level, 36-bit PA)
**Given:** 3-level paging. VA = 32 bits, PA = 36 bits, Page frame = 4 KB, PTE = 4 B. Bit split: L1 = bits 30-31, L2 = bits 21-29, L3 = bits 12-20, Offset = bits 0-11.

**Q:** Bits needed *within a PTE* to address the next-level table (or final frame) at L1, L2, L3 respectively?

**Options:** A) 20,20,20  B) **24,24,24**  C) 24,24,20  D) 25,25,24

**Reasoning:** Regardless of level, a PTE must store a **physical frame number** (of the next-level table, or of the final data frame). Since PA = 36 bits and offset = 12 bits, frame number = $36-12 = 24$ bits — **same at every level** (all tables/pages are aligned to page-size frames in physical memory).

**Answer: B) 24, 24 and 24**

---

### Q12. Effective Access Time with TLB + Cache (2-level paging)
**Given:** 2-level page tables (both in main memory), VA = PA = 32 bits, top 10 bits → L1 index, next 10 bits → L2 index, low 12 bits → offset, PTE = 4 B at both levels. TLB hit rate = 96%. Physically-addressed cache hit rate = 90%. Main memory access = 10 ns, cache access = 1 ns, TLB access = 1 ns. No page faults occur.

**Options:** A) 1.5 ns  B) 2 ns  C) 3 ns  D) **4 ns** *(nearest 0.5 ns)*

**Solve:**
- Effective data-access time (via cache): $0.9\times1 + 0.1\times10 = 1.9$ ns
- **TLB hit (96%):** $1\,(\text{TLB}) + 1.9\,(\text{data}) = 2.9$ ns
- **TLB miss (4%):** walk L1 (10 ns) + walk L2 (10 ns) + data access (1.9 ns) + initial TLB check (1 ns) = 22.9 ns
- **Average** = $0.96\times2.9 + 0.04\times22.9 = 2.784 + 0.916 = 3.7$ ns → nearest option ≈ **4 ns**

**Answer: D**

---

### Q13. Memory for Page Tables — Sparse Process (Classic GATE 2014)
**Given:** A process has only: 2 contiguous **code** pages starting at VA `0x00000000`, 2 contiguous **data** pages starting at VA `0x00400000`, and 1 **stack** page starting at VA `0xFFFFF000`. (Assume 32-bit VA, 4 KB pages, 2-level paging with 10-10-12 split, PTE = 4 B ⇒ each page table = 4 KB.)

**Options:** A) 8 KB  B) 12 KB  C) **16 KB**  D) 20 KB

**Solve:** Each **outer-level index** covers a 4 MB region (1024 inner entries × 4 KB).
- Code (`0x000000`) → outer index **0**
- Data (`0x400000` = $2^{22}$ exactly) → outer index **1**
- Stack (`0xFFFFF000`, near top of 4 GB) → outer index **1023**

All three fall in **different** outer-index regions → need **3 separate inner (2nd-level) page tables**, each 4 KB = **12 KB**. Plus the single **outer page table** itself = **4 KB**.

$$
\text{Total} = 12KB + 4KB = \mathbf{16\ KB}
$$

**Answer: C**

---

### Q14. Segment Table — Physical Address Lookup

| Segment | Base | Length |
|---|---|---|
| 0 | 1219 | 600 |
| 1 | 3300 | 14 |
| 2 | 90 | 100 |
| 3 | 2327 | 580 |
| 4 | 1952 | 96 |

**Given logical addresses (segment, offset):**

| Query | Offset vs Length | Result |
|---|---|---|
| A. (0, 4302) | 4302 > 600 | ❌ Invalid (out of bounds / addressing error) |
| B. (1, 15) | 15 > 14 | ❌ Invalid (out of bounds) |
| C. (2, 50) | 50 < 100 ✅ | **PA = 90 + 50 = 140** |
| D. (3, 400) | 400 < 580 ✅ | **PA = 2327 + 400 = 2727** |
| E. (4, 112) | 112 > 96 | ❌ Invalid (out of bounds) |

**Only C (140) and D (2727) are valid** physical addresses; A, B, E trigger a segmentation fault/addressing error.

---

### Q15. Segmented-Paging — finding Page Size of a Segment
**Given:** VAS = PAS = $2^{16}$ Bytes, VAS divided into **8 equal non-overlapping segments**, paging applied within each segment, PTE size = 16 bits. Find page size $P$ (power of 2) such that the segment's page table exactly fits in **one frame**.

**Solve:**
- Segment size = $2^{16}/8 = 2^{13} = 8192$ Bytes
- Let $P = 2^p$. Entries needed = Segment Size / $P$ = $2^{13-p}$
- Page table size = entries × PTE size = $2^{13-p} \times 2B$ (16 bits = 2 B), must equal $P = 2^p$:

$$
2^{13-p}\times 2 = 2^p \Rightarrow 2^{14-p}=2^p \Rightarrow 14-p=p \Rightarrow p=7
$$

**Page Size = $2^7$ = 128 Bytes**

---

### Q16. Segmented Paging — Full System Sizing
**Given:** 256-entry page table per segment, page size of segment = 8 KB, VAS supports 2K segments, PTE = 16 bits, Segment Table entry = 32 bits.

**(a) Size of VAS:**
$$
\text{Segment size} = 256 \times 8KB = 2MB \quad\Rightarrow\quad \text{VAS} = 2048 \times 2MB = \mathbf{4\ GB\ (2^{32}\ B)}
$$

**(b) Address Translation Space Overhead:**
- Segment Table = $2048 \times 4B = 8192B = 8$ KB
- Total Page Tables = $2048 \times (256\times2B) = 2048\times512B = 1{,}048{,}576B = 1$ MB
- **Total overhead = 8 KB + 1 MB = 1032 KB**

**(c) Levels of memory access for translation:**
Segment Table access → Page Table access → Data access = **3 memory accesses**

---

### Q17. Why must the Segment Table have its own Page Table? (Classic GATE)

**Options:**
- A. **The Segment Table is often too large to fit in one page** ✅ *(Correct)*
- B. Each segment is spread over a number of pages
- C. Segment tables point to page table and not to physical locations of segment
- D. The processor's description base register points to a Page Table

**Answer: A**
**Why others are wrong:** B describes segmented-paging in general, not a reason the *segment table itself* needs paging. C describes a mechanism/consequence, not the root cause. D is an implementation detail specific to certain architectures, not a general reason.

---

## 12. Mnemonics / Remember-This Tips

- **"Page the page tables"** → the core one-liner for why multilevel paging exists.
- **PTS ∝ 1/PS** → bigger pages = smaller page table, but more internal fragmentation. Trade-off, balanced by $P=\sqrt{2Se}$.
- **Inverted PT = "one entry per frame, not per page"** → scales with *physical* memory, not virtual — huge win for 64-bit systems.
- **Hashed Paging = "Inverted paging with a shortcut"** — hashing avoids a full linear scan of the inverted table.
- Frame-number field in a PTE is **always (PA − offset) bits wide, at every level** — this is why Q11 gives the same 24 bits at all 3 levels.
- **Segment Table itself may need a page table** — because it, too, can be "too big for one page."

---

## 13. Bonus Spillover: Synchronization Primitives (appears at end of this lecture's recording — likely lead-in to next lecture)

### Binary Semaphore vs Split Binary Semaphore

| Binary Semaphore | Split Binary Semaphore |
|---|---|
| Single semaphore | Collection of binary semaphores |
| Each semaphore independent | Semaphores are related |
| Multiple semaphores may simultaneously have value 1 | Exactly **one** semaphore has value 1 at any time |
| Used directly for mutual exclusion | Used to implement monitors and condition synchronization |
| Simple primitive | Structured synchronization technique |

**Why "Split"?** Imagine one binary semaphore holding `Permission = 1`. Now **split** that single permission/token across, say, three semaphores: `mutex`, `condition1`, `condition2` — only **one of them** is allowed to be `1` at any moment (e.g., `mutex=1, condition1=0, condition2=0` OR `mutex=0, condition1=1, condition2=0`, etc.).

> 🧠 **Mnemonic:** *"The single execution token is split across multiple binary semaphores, but there is still only one token in the system."*

### PYQ (unmarked in source — reasoning provided, verify against your source before exam)
**Consider the class of Synchronization Primitives. Which of the following is FALSE?**
- A. Test-and-set primitives are as powerful as Semaphores.
- B. There are various synchronization problems implementable using an array of semaphores but not by binary semaphores.
- C. Split binary semaphores and binary semaphores are equivalent.
- D. All statements A to C are False.

*No answer was marked in the source lecture for this question — treat it as a discussion/practice prompt rather than a confirmed-answer PYQ. Cross-check against the original source before relying on it for exam prep.*

---

## 14. Quick Revision Checklist ✅

Self-test — can you explain each of these without looking back?

- [ ] Memory hierarchy order (Registers → Cache → Main Memory → Disk → Optical → Tape) and cost/speed/capacity trends
- [ ] Contiguous vs Non-contiguous allocation techniques (list all 5 non-contiguous types)
- [ ] PTS formula: $\text{PTS}=N\times e$, and $N=\text{LAS}/\text{PS}$
- [ ] Trade-off: increasing page size ↓ PTS but ↑ internal fragmentation
- [ ] Optimal Page Size formula: $P=\sqrt{2Se}$ — derive it, don't just memorize
- [ ] Inverted Page Table: structure (`⟨pid, page⟩` per frame), address translation via search, size formula $M'\times e$
- [ ] Why Inverted PT scales better than conventional PT for large VAS
- [ ] Hashed Paging: hash function, collision handling via chaining, size formula $C\times N$
- [ ] Why Multilevel Paging exists (single-level table can be MBs per process)
- [ ] 2-level and 3-level address translation diagrams (outer table → inner table → offset)
- [ ] Multilevel Paging advantages (on-demand, sparse-space friendly) vs disadvantages (n+1 memory accesses, needs TLB)
- [ ] Effective Access Time formula with page-fault probability
- [ ] Segmentation basics: `⟨Base, Length⟩`, bounds checking
- [ ] Segmented Paging: why the Segment Table may itself need a Page Table
- [ ] All worked numericals (page fit/no-fit, PTE bit-widths, min/max page table sizes, TLB+cache effective access time)
- [ ] Bonus: Binary Semaphore vs Split Binary Semaphore differences

---

*End of Lecture 25 Notes — Memory Management (Paging deep-dive).*
