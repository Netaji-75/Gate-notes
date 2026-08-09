# Lecture 30 — File System & Disk Scheduling (GATE CS/IT Revision Notes)

> Source: GeeksforGeeks GATE CS&IT — Principles of Operating Systems, Lecture 30
> Parts covered: **IV. File System** and **IV. Disk Scheduling**

---

## 📑 Table of Contents

| # | Topic |
|---|-------|
| 1 | Physical & Logical Disk Structure |
| 2 | File System Implementation — Allocation Methods (Contiguous, Linked, Indexed) |
| 3 | Unix i-node Structure (Direct/Indirect/Double/Triple Indirect Blocks) |
| 4 | FAT (File Allocation Table) — DOS/Windows |
| 5 | Free (Disk) Space Management — Bitmap, Linked List, Grouping, Counting |
| 6 | Anatomy of a "Save" — full write path |
| 7 | Space vs Speed Trade-off (Block Size) |
| 8 | GATE PYQs — File System |
| 9 | Disk Scheduling — Basics (Seek Time, Rotational Latency, Bandwidth) |
| 10 | Disk Scheduling Algorithms — FCFS, SSTF, SCAN, LOOK, C-SCAN, C-LOOK |
| 11 | Fully Worked Example (all 6 algorithms, same queue) |
| 12 | GATE PYQs — Disk Scheduling |
| 13 | Spooling vs Buffering |
| 14 | Mnemonics / Memory Tricks |
| 15 | Quick Revision Checklist |

---

## 1. Physical & Logical Disk Structure

- **Physical structure**: platters, tracks, sectors, cylinders, read/write heads, actuator arm.
- **Logical structure**: the OS presents the disk as a **linear array of logical blocks** (abstracting away physical geometry) — this is what the file system allocates to files.
- **File System Interface** = the set of operations (open, read, write, seek, close, directory ops) exposed to users/programs.
- **File System Implementation** = how the OS internally realizes these operations using on-disk data structures (allocation tables, i-nodes, free lists, etc.)

---

## 2. File System Implementation — Allocation Methods

The core problem (**"Layer 4: The Allocation Challenge"**): *How do we efficiently pack variable-sized files into fixed-size hardware blocks?*

| Method | How it works | Pros | Cons |
|---|---|---|---|
| **Contiguous Allocation** | File occupies a contiguous run of blocks | Fast sequential & random access, simple | **External fragmentation**; file growth is hard |
| **Linked Allocation** | Each block stores a pointer to the next block of the file | No external fragmentation; easy to grow file | Slow random access (must traverse); pointer overhead per block; reliability risk (broken chain) |
| **Indexed Allocation** | A separate **index block** stores pointers to all data blocks of the file | No external fragmentation; fast random access | Overhead of index block(s); for very large files → need multi-level indexing (this is exactly what Unix i-nodes solve) |

> ⭐ **Key GATE fact:** Only **Linked** and **Indexed** allocation avoid **external fragmentation** (Contiguous does not).

---

## 3. Unix i-node Structure

### 3.1 Concept
Every file is represented by an **i-node** containing:
- General attributes: **type, size, owner, permissions, date/time, link count**
- **Block Info**: pointers to data blocks

The Unix i-node is a **hybrid indexed allocation** scheme — balances efficiency for small files with scalability for massive files:

```
i-node ──► 10 (or 12) Direct Block Addresses (DBA)  → straight to Data
        ──► 1 Single-Indirect DBA   → Index block → Data
        ──► 1 Double-Indirect DBA   → Index block → Index blocks → Data
        ──► 1 Triple-Indirect DBA   → Index block → Index → Index → Data
```

- **Direct blocks**: fast access for small files (no extra disk read).
- **Single indirect**: 1 extra disk read to get the index block, then data.
- **Double indirect**: 2 extra disk reads.
- **Triple indirect**: 3 extra disk reads — needed because more index blocks may exist than can be addressed by a 32-bit file pointer.

### 3.2 Master Formula — Maximum File Size

$$
\text{Max File Size} = (n_d \times \text{DBS}) + (k \times \text{DBS}) + (k^2 \times \text{DBS}) + (k^3 \times \text{DBS})
$$

Where:
- $n_d$ = number of **direct** block addresses in the i-node
- $k = \dfrac{\text{DBS}}{\text{DBA}}$ = number of block-addresses that fit inside **one** block (fan-out of an index block)
- $\text{DBS}$ = Disk Block Size
- $\text{DBA}$ = Disk Block **Address** size (in bytes/bits)
- Terms present depend on how many indirection levels the i-node has (single/double/triple)

> **Trick:** Always compute $k$ = (block size)/(address size) first — it's the fan-out factor multiplying at each indirection level.

### 3.3 Worked Examples

**Example A** — 10 direct + single + double indirect DBAs, DBA = 16 bits, DBS = 1 KB

$$
k = \frac{1\text{KB}}{16\text{ bits}} = \frac{1024\text{ B}}{2\text{ B}} = 512
$$

$$
\text{Max Size} = 10(1\text{KB}) + 512(1\text{KB}) + 512^2(1\text{KB})
$$
$$
= 10\text{KB} + 512\text{KB} + 262144\text{KB} = 522\text{KB} + 256\text{MB} \approx 256.5\text{ MB}
$$

**Example B** — 8 direct + single/double/triple indirect, DBS = 1 KB, each block holds 128 DBAs

- (i) Max File Size:
$$
8(1KB) + 128(1KB) + 128^2(1KB) + 128^3(1KB) \approx 2\text{ GB} \;(\approx 2^{31}\text{ bytes})
$$
- (ii) Size of Disk Block Address: $1\text{KB} / 128 = 8\text{ B} = \mathbf{64\text{ bits}}$
- (iii) Is 2 GB possible on the given disk? Given disk = $2^{64} \times 1\text{KB} = 2^{74}\text{ bytes}$ (astronomically large) → **Yes**, 2 GB easily fits.

**Example C** — 12 direct, 1 single-indirect, 1 double-indirect; DBS = 4 KB; DBA = 32-bit (4 B)

$$
k = \frac{4\text{KB}}{4\text{B}} = 1024 = 1\text{K}
$$
$$
\text{Max Size} = 12(4KB) + 1024(4KB) + 1024^2(4KB) = 48KB + 4MB + 4GB \approx \mathbf{4.0\ GB}
$$

**Example D** — 8 direct, 1 indirect, 1 doubly-indirect; DBS = 128 B; DBA = 8 B → GATE MCQ

$$
k = \frac{128\text{B}}{8\text{B}} = 16
$$
$$
\text{Max Size} = 8(128B) + 16(128B) + 16^2(128B) = 1024B + 2048B + 32768B = \mathbf{35\ KB}
$$
✅ **Answer: (B) 35 K Bytes**

**Example E** — index table stores 128 DBAs, DBS = 4 KB. If file ≤ 128 blocks → direct addressing. If file > 128 blocks → 128 entries point to next-level index blocks, each holding 256 addresses.

$$
\text{Max File Size} = 128 \times 256 \times 4\text{KB} = 32768 \times 4\text{KB} = \mathbf{128\ MB}
$$

---

## 4. FAT (File Allocation Table) — DOS/Windows

### 4.1 Structure
- **Directory Entry**: `File Name | General Attributes | First DBA`
- **FAT table**: one entry per disk block; entry = DBA of the **next** block in the file's chain (like a linked list, but the "pointers" live in one central table instead of inside each data block).
- A value of **−1** in the FAT marks **end of file (EOF)**.

```
Directory: KK.C ──► First DBA = 8
FAT[8]  = y
FAT[y]  = 2
FAT[2]  = x
FAT[x]  = -1   (EOF)
```
So file `KK.C` occupies blocks: **8 → y → 2 → x**

### 4.2 FAT Versions

| Version | Year | Limit |
|---|---|---|
| FAT12 | 1980 | small (12-bit entries) |
| FAT16 | 1984 | ~20 MB |
| FAT32 | 1996 | max ~16 TB |

### 4.3 Worked Example — Max File Size in a FAT System

FAT-based FS, each FAT entry overhead = 4 bytes, disk = $100\times10^6$ bytes, data block size = $10^3$ bytes.

$$
N (\text{total blocks}) = \frac{100\times10^6}{10^3} = 100\times10^3 \text{ blocks}
$$
$$
\text{FAT Size} = N \times 4\text{B} = 100\times10^3\times4 = 400\times10^3\text{ B} = 0.4\times10^6\text{ B}
$$
$$
\text{Max File Size} = \text{Disk Size} - \text{FAT Size} = 100\times10^6 - 0.4\times10^6 = \mathbf{99.6\times10^6\text{ bytes}}
$$

### 4.4 Worked Example — One-Level Directory FAT

Disk Block Size = 4 KB.
- Block 0 = Boot Control Block
- Block 1 = FAT, **10-bit** entry per data block
- Blocks 2,3 = Directory, **32-bit** entry per file
- Block 4 onwards = Data blocks

**(a) Max number of files:**
$$
\text{No. of entries} = \frac{\text{Directory Size}}{\text{Entry Size}} = \frac{8\text{KB}}{4\text{B}} = \mathbf{2048\ (2K)\ files}
$$

**(b) Max possible file size:**
DBA = 10 bits ⟹ $N = 2^{10} = 1024$ total addressable blocks.
Blocks 0–3 (boot + FAT + 2 directory blocks) are reserved, so usable data blocks = $1024 - 4 = 1020$.
$$
\text{Max File Size} = 1020 \times 4\text{KB} = \mathbf{4080\ KB}
$$

---

## 5. Free (Disk) Space Management

Four classic techniques:

### 5.1 Bit Vector / Bitmap
- One **bit per block**: `0 = free`, `1 = in use` (or vice-versa depending on convention — GfG slide used 0=allocated,1=free in one diagram and 0=free,1=allocated in another; **always check the question's convention!**).
- Very efficient for finding **contiguous** free blocks using bitwise hardware ops.
- **Flaw**: needs the whole map held in contiguous memory.

**Formula:**
$$
\text{Bitmap size (bits)} = N \text{ (total number of blocks)}
$$

**Worked Example:** Disk = 20 MB, DBA = 16 bit, DBS = 1 KB → $N = 20K$ blocks (free)
$$
\text{Bitmap size} = 20\text{K bits} = \frac{20000}{8192} \approx 2.5 \text{–} 3 \text{ blocks (of 1KB = 8Kbits)}
$$

### 5.2 Free (Linked) List
- Each free block stores a pointer to the next free block — **zero extra space**, but slow to traverse (no random access to "give me the nth free block").
- Variant — **"Free List of addresses"**: a block stores a small array of addresses of other free blocks (a hybrid, faster than pure block-to-block linking).

**Worked Example:** 1 block → 512 (addresses, when DBA=16bit, DBS=1KB), N=20K free blocks
$$
\text{Blocks needed to store the free list} = \frac{20\text{K}}{512} \approx \mathbf{40\ blocks}
$$

### 5.3 Grouping
- The first free block stores addresses of several other free blocks, the **last** of which points to a block holding addresses of the *next* group, and so on — a compromise between bitmap and pure linked list.

### 5.4 Counting Method
- Exploits the fact that **contiguous free blocks are common** — store `(First Free DBA, Count of contiguous free blocks)` pairs instead of individual addresses. Much more compact when free space is not fragmented.

| First Free DBA | # of Free Contiguous Blocks |
|---|---|
| 5 | 30 |
| 65 | 108 |
| 250 | 350 |
| 850 | 20 |
| 1200 | 200 |

> Typical GATE trick question: given `F_new = 10 blocks` freed somewhere, determine whether it merges as **First-Fit (FF)**, **Best-Fit (BF)**, or **Next-Fit (NF)** into the free list/table.

### 5.5 Comparison Table

| Technique | Space overhead | Random access to free block | Best when |
|---|---|---|---|
| **Bitmap** | Fixed (1 bit/block) regardless of free-block count | Easy (bit index = block #) | Contiguous-block search needed |
| **Linked List** | Zero extra space | Slow (traverse) | Space is scarce |
| **Grouping** | Small | Medium | Balance of both |
| **Counting** | Very small (if disk mostly has large contiguous free runs) | Medium | Free space is mostly contiguous |

**Relation (Free List vs Bitmap space):** Disk has `B` blocks, `F` free, DBA = `D` bits.
$$
\text{Free list uses less space than bitmap when: } \quad F \times D < B
$$
(since bitmap always costs exactly $B$ bits, but a free list costs $F \times D$ bits — one D-bit address per free block)

Also: **Relation between B and D** (an address must be able to represent every block):
$$
B \le 2^D \quad \Longrightarrow \quad \log_2 B \le D
$$

---

## 6. Anatomy of a "Save" (Synthesis — how all pieces fit together)

When an application issues a **Save** system call for a file:

1. **User Request** — App issues a "Save" system call for e.g. a 2 MB file.
2. **Housekeeping** — OS checks the **Bit Vector** to find available, unallocated blocks.
3. **File Creation** — OS creates a **Directory Entry** and instantiates an **i-node** (metadata).
4. **Allocation** — OS uses **Indexed Allocation** to assign the free blocks to the file.
5. **Physical Write** — The actuator arm moves the read/write head to the correct cylinder/sector to alter magnetic states.

---

## 7. Space vs Speed Trade-off (Block Size)

**Every file system balances throughput against space utilization — there is no perfect block size.**

| | Larger Block Size | Smaller Block Size |
|---|---|---|
| **Disk Throughput** | ✅ Better (fewer seeks) | ❌ Worse (more seeks) |
| **Space Utilization** | ❌ Worse — **internal fragmentation** (e.g., a 1 KB file wastes 3 KB inside a 4 KB block) | ✅ Better — near-zero internal fragmentation |
| **Indexing overhead** | Lower | Higher (more blocks to track) |

> Ext4, APFS, NTFS are simply different blueprints optimized for different physical media and workloads — not "better" or "worse" universally.

**GATE MCQ:** *Using a larger block size in a fixed-block-size FS leads to…* → ✅ **(A) Better Disk Throughput but Poorer Disk Space Utilization.**

---

## 8. GATE PYQs — File System

| # | Question | Options | Answer | Why others are wrong |
|---|---|---|---|---|
| 1 | Data blocks of a **very large file** in Unix FS are allocated using… | (A) Contiguous (B) Linked (C) Indexed (D) An extension of indexed allocation | **(D)** | Large files need multi-level (double/triple) indirect indexing — an *extension* of plain single-level indexed allocation, not plain (C) which only covers moderate-size files |
| 2 | Linear-List directory: which op **necessarily** needs a full scan? | (A) Open existing file (B) Create new file (C) Rename existing file (D) Delete existing file | **(B) Creation of a new file** | Creating a file requires scanning the *entire* list to confirm the name doesn't already exist (uniqueness check); open/rename/delete can often stop as soon as the match is found |
| 3 | Which allocation scheme(s) allow **no External Fragmentation**? I. Contiguous II. Linked III. Indexed | (A) I & III (B) II only (C) III only (D) II & III | **(D) II and III only** | Contiguous allocation inherently suffers external fragmentation since it needs a *single contiguous run* of blocks |
| 4 | Max file size, I-node with 8 direct, 1 indirect, 1 doubly-indirect, DBS = 128 B, DBA = 8 B | (A) 3 KB (B) 35 KB (C) 280 KB (D) Depends on disk size | **(B) 35 K Bytes** | See worked Example D above: $8(128)+16(128)+16^2(128)=35\text{KB}$ |
| 5 | Data blocks of a very large Unix file are allocated using | (A) Contiguous (B) Linked (C) Indexed (D) An extension of indexed allocation | **(D)** | Same reasoning as #1 |
| 6 | Larger block size in fixed-block FS leads to… | (A) Better throughput, poorer space util (B) Better both (C) Poorer throughput, better space util (D) Poorer both | **(A)** | Larger blocks → fewer seeks (↑ throughput) but more internal fragmentation (↓ space efficiency); B & D are physically inconsistent trade-offs |
| 7 | FAT-based FS, entry overhead 4 B, disk = $100\times10^6$ B, block size $=10^3$ B → max file size in units of $10^6$ B | — | **99.6** | See §4.3 — subtract FAT-table overhead from raw disk size |
| 8 | For a page trace, LRU gives 9 & 11 faults for 6-page & 4-page memory resp. What may Optimal give? | (A) 9,7 (B) 7,9 (C) 10,12 (D) 6,7 | **(B) 7 and 9** | Optimal always faults ≤ LRU for the *same* memory size, **and** more memory ⟹ optimal faults never increase (no Belady anomaly in Optimal). Only (B) satisfies $7\le9$, $9\le11$, and $7\le9$ (monotonic) simultaneously |
| 9 | Sharing in a paged memory system is done by… | (A) Copy of shared pages per process (B) Split into procedures/data, share only procedures (C) Several page table entries → same frame (D) None | **(C)** | Sharing = multiple processes' page tables map different virtual pages to the **same physical frame**; (A) defeats the purpose of sharing (duplicates memory) |
| 10 | Good/Bad techniques for demand paging: stack, hashed symbol table, sequential search, binary search, pure code, vector ops, indirection | — | See below | — |

**Q10 detail (classic Silberschatz "good/bad for demand paging" question):**

| Technique | Good/Bad | Reason |
|---|---|---|
| Stack | ✅ Good | High locality — top of stack accessed repeatedly |
| Hashed symbol table | ❌ Bad | Random-looking access pattern → poor locality → many faults |
| Sequential search | ✅ Good | Accesses memory in order → excellent locality |
| Binary search | ❌ Bad | Jumps across the whole array → poor locality |
| Pure (reentrant) code | ✅ Good | Never modified → can be shared across processes; small working set |
| Vector operations | ❌ Bad | Touches an entire large array once each pass → poor locality, many page faults |
| Indirection | ❌ Bad | Pointer chasing can land anywhere in memory → unpredictable, poor locality |

**Bonus PYQ (CPU utilization diagnosis)** — Demand paging system: CPU util 20%, Paging disk 97.7%, Other I/O 5%. For each fix, does it likely **improve** CPU utilization?

| Action | Effect |
|---|---|
| (a) Faster CPU | ❌ No — CPU is idle 80% of the time already; it's not the bottleneck |
| (b) Bigger paging disk | ❌ No — size ≠ speed |
| (c) Increase multiprogramming | ❌ No (likely worse) — disk already ~98% busy; more processes ⟹ more paging ⟹ thrashing |
| (d) Decrease multiprogramming | ✅ Yes — fewer processes competing for memory ⟹ less paging |
| (e) Install more main memory | ✅ Yes — fewer page faults |
| (f) Faster/parallel disks | ✅ Yes — disk is the actual bottleneck (97.7%) |
| (g) Add prepaging | ✅ Possibly — reduces reactive fault-driven I/O |
| (h) Increase page size | ⚠️ Possibly, up to a point — beyond that, internal fragmentation + wasted I/O hurts |

---

## 9. Disk Scheduling — Basics

The OS must schedule pending I/O requests on a disk queue for efficiency, just like CPU scheduling schedules processes.

```
Ready Queue → CPU  (Short-Term / CPU Scheduling)
Disk Queue  → Disk (Disk Scheduling)
```

**Access Time = Seek Time + Rotational Latency**

| Term | Meaning |
|---|---|
| **Seek time** | Time for the R/W head to move to the correct **cylinder** — the dominant, most-optimized cost |
| **Rotational latency** | Additional wait for the desired **sector** to rotate under the head |
| **Disk bandwidth** | Total bytes transferred ÷ total time (first request → last transfer completion) |

> **Goal of all disk scheduling algorithms: minimize total seek time / head movement.**

---

## 10. Disk Scheduling Algorithms

| Algorithm | Strategy | Notes |
|---|---|---|
| **FCFS** | Service requests strictly in arrival order | Simple, fair, but can cause **very long total seek distance** (no optimization) |
| **SSTF** (Shortest Seek Time First) | Always service the request **nearest** to current head position | a.k.a. "Nearest Track Next"; minimizes *immediate* seek but can **starve** far-away requests |
| **SCAN** (Elevator algorithm) | Head sweeps in one direction servicing requests, goes **all the way to the disk's edge (0 or max)**, then reverses | Like an elevator — services all requests in current direction before turning around |
| **LOOK** | Same as SCAN, but reverses direction as soon as **no more requests exist** ahead — doesn't go all the way to the disk edge | More efficient version of SCAN |
| **C-SCAN** (Circular SCAN) | Services in one direction to the edge, then **jumps back to the opposite edge (0)** without servicing on the return trip, and starts a fresh sweep | Gives more **uniform wait time** across the disk (avoids bias toward middle cylinders) |
| **C-LOOK** | Same as C-SCAN, but jumps back only to the **lowest pending request** (not all the way to 0) | Most efficient overall in typical benchmarks |

### Direction convention used below
- SCAN/LOOK/C-SCAN/C-LOOK examples assume the head is initially **moving toward larger cylinder numbers (right)**.

---

## 11. Fully Worked Example — All 6 Algorithms

**Given:** Disk queue = `98, 183, 37, 122, 14, 124, 65, 67`
Head starts at cylinder **53**. Disk range: **0 – 199**.

### 11.1 FCFS
Service strictly in given order:
$$
53\to98\to183\to37\to122\to14\to124\to65\to67
$$
$$
|98-53|+|183-98|+|37-183|+|122-37|+|14-122|+|124-14|+|65-124|+|67-65|
$$
$$
=45+85+146+85+108+110+59+2 = \mathbf{640}
$$

### 11.2 SSTF
Always jump to the nearest unserviced request:
$$
53\to65\to67\to37\to14\to98\to122\to124\to183
$$
$$
12+2+30+23+84+24+2+59 = \mathbf{236}
$$

### 11.3 SCAN (Elevator)
Moving right first, go **all the way to disk end (199)**, then reverse to the last request on the left (14):
$$
53\to65\to67\to98\to122\to124\to183\to199\ (\text{edge})\to37\to14
$$
$$
(199-53)+(199-14) = 146+185 = \mathbf{331}
$$

### 11.4 LOOK
Same as SCAN but **stops at the last request**, not the disk edge:
$$
53\to65\to67\to98\to122\to124\to183\ (\text{last req.})\to37\to14
$$
$$
(183-53)+(183-14) = 130+169 = \mathbf{299}
$$

### 11.5 C-SCAN
Sweep right to the disk end (199), **jump to 0**, sweep right again up to the last request:
$$
53\to183\to199\ (\text{edge})\to0\ (\text{jump})\to37
$$
$$
(199-53)+(199-0)+(37-0) = 146+199+37 = \mathbf{382}
$$

### 11.6 C-LOOK
Sweep right to the last request (183), **jump directly to the lowest pending request (14)**, sweep right again:
$$
53\to183\ (\text{last req.})\to14\ (\text{jump})\to37
$$
$$
(183-53)+(183-14)+(37-14) = 130+169+23 = \mathbf{322}
$$

### 11.7 Summary Table

| Algorithm | Total Head Movement (seeks) |
|---|---|
| FCFS | 640 |
| SSTF | 236 |
| SCAN | 331 |
| LOOK | 299 |
| C-SCAN | 382 |
| C-LOOK | 322 |

> **Observation:** SSTF gave the least total movement here, but SSTF can starve requests far from the current head. LOOK/C-LOOK are usually preferred in practice for a good balance of efficiency + fairness.

---

## 12. GATE PYQs — Disk Scheduling

### PYQ 1 — C-LOOK head movement (GATE-style)
> Requests: `47, 38, 121, 191, 87, 11, 92, 10`. Head at **63**, moving toward **larger** cylinder numbers. Cylinders 0–199. Find total head movement using **C-LOOK**.

**Solution:**
Sorted requests: `10, 11, 38, 47, 87, 92, 121, 191`
From 63, moving right: service `87 → 92 → 121 → 191`, then **jump directly to the lowest request (10)** (jump itself isn't counted as head movement in C-LOOK), then continue right: `10 → 11 → 38 → 47`.

$$
(191-63) + (47-10) = 128 + 37 = \mathbf{165}
$$

✅ **Total head movement = 165 cylinders**

### PYQ 2 — FCFS vs Closest-Cylinder-Next (SSTF)
> Requests arrive for cylinders `10, 22, 20, 2, 40, 6, 38` (in that order) while head is at cylinder **20**. Seek time = **6 ms/cylinder**. Compute total seek time for (a) FCFS and (b) Closest cylinder next (SSTF).

**(a) FCFS:**
$$
20\to10\to22\to20\to2\to40\to6\to38
$$
$$
10+12+2+18+38+34+32 = 146 \text{ cylinders}
$$
$$
\text{Time} = 146\times6\text{ms} = \mathbf{876\ ms}
$$

**(b) SSTF (Closest cylinder next):**
$$
20\to20\to22\to10\to6\to2\to38\to40
$$
$$
0+2+12+4+4+36+2 = 60 \text{ cylinders}
$$
$$
\text{Time} = 60\times6\text{ms} = \mathbf{360\ ms}
$$

### PYQ 3 — SSTF: position of a specific request in service order
> Disk has 201 cylinders (0–200), head at cylinder **100**. Queue: `30, 85, 90, 100, 105, 110, 135, 145`. Using SSTF, after how many requests is cylinder **90** serviced?

**Solution (SSTF order):**
$$
100 \to 105 \to 110 \to 90 \to \dots
$$
(100 is distance 0 → serviced first; 105 is next-nearest at distance 5; 110 next at distance 5; then 90 is nearest remaining at distance 20)

✅ **Cylinder 90 is serviced after 3 requests** (100, 105, 110 serviced before it).

### PYQ 4 — SSTF vs SCAN, additional distance traveled
> Requests (track numbers): `45, 20, 90, 10, 50, 60, 80, 25, 70` for a disk with 100 tracks (0–99). Head starts at track **50**. SCAN moves toward 100 first. Find the extra distance SSTF traverses compared to SCAN.

**SSTF path:** $50\to45\to60\to70\to80\to90\to25\to20\to10$
$$
5+15+10+10+10+65+5+10 = 130
$$

**SCAN path (toward 100 first, then to disk edge 0):** $50\to99\ (\text{edge})\to0\ (\text{edge})$
$$
(99-50)+(99-0) = 49+99 = 148
$$

$$
\text{Difference} = 148 - 130 = \mathbf{18\ \text{tracks}}
$$
(SSTF traverses **18 tracks less** than SCAN in this example — i.e., SSTF is more efficient here, though it can starve far requests in general.)

> ⚠️ Exact numeric answer can shift slightly depending on whether the exam defines SCAN as reaching the **physical disk boundary** or the **last pending request** in that direction — always re-read the question's SCAN definition carefully.

### PYQ 5 — FCFS→SSTF benchmark improvement
> OS loads/executes one sequential process at a time using FCFS disk scheduling. If replaced by SSTF (vendor claims 50% better *benchmark* results), what's the expected real I/O performance improvement for user programs?

✅ **Answer: No improvement (0%).** Since the OS runs a **single sequential process at a time**, there is only ever **one pending disk request** in the queue — SSTF (and any other scheduling algorithm) has nothing to optimize over, because there's no queue of multiple requests to reorder. Disk-scheduling algorithms only help when **multiple concurrent requests** are pending.

### PYQ 6 — SSTF ordering / statement verification
> Requests `(P,155), (Q,85), (R,110), (S,30), (T,115)`; head at cylinder **100**; scheduler = SSTF. Which statement is **FALSE**?
>
> (A) R is serviced before P (B) T is serviced before P (C) Q is serviced after S, but before T (D) The head reverses direction between servicing Q and P

**SSTF order:** from 100 → nearest is R(110, dist 10) → T(115, dist 5 from 110) → P(155, dist 40 from 115) → Q(85, dist 70 from 155) → S(30, dist 55 from 85)
Order: **R → T → P → Q → S**

- (A) R before P → **True**
- (B) T before P → **True**
- (C) Q serviced *after S*? — **False!** Q is serviced *before* S in this order (Q comes 4th, S comes 5th) — so **(C) is FALSE**
- (D) Head reverses between Q(85) and P(155)? — direction was decreasing (155→85 is leftward after being at 115→155 rightward)... checking: R(110)→T(115) is rightward; T(115)→P(155) is rightward; P(155)→Q(85) is leftward — **reversal happens between P and Q**, not "between Q and P" as stated loosely, but this describes the same transition, so (D) is essentially **True**.

✅ **Answer: (C)** — "Q is serviced after S, but before T" is FALSE (Q is actually serviced *before* S, not after).

---

## 13. Spooling vs Buffering

| | **Spooling** | **Buffering** |
|---|---|---|
| **Purpose** | Overlap the I/O of **one job** with the **execution of a different job** | Overlap the I/O of **one job** with the **execution of the same job** |
| **Storage location** | Secondary storage (disk) — the "Spool" | A small area of **RAM** (the "Buffer") |
| **Example** | Print spooler — printing job A's output while job B computes | Reading ahead into a buffer while the current process consumes earlier data |

---

## 14. Mnemonics / Memory Tricks

- 🛗 **SCAN = "Elevator Algorithm"** — think of a lift that goes all the way to the top/bottom floor even if no one there wants off, then reverses.
- 👀 **LOOK** = SCAN, but the elevator **peeks ("looks") ahead** first — if no one's waiting further up, it doesn't bother going all the way, it just turns around early.
- 🔄 **C- prefix (C-SCAN / C-LOOK) = "Circular"** — treats the disk like a **circular queue**: after reaching one end, it jumps back to the *start* and sweeps in the *same* direction again (never reverses direction) — this gives **fairer, more uniform wait times** across all cylinders (SCAN/LOOK bias toward the middle cylinders since they get "visited" from both directions).
- 📇 **i-node "fan-out" trick:** $k = \dfrac{\text{Block Size}}{\text{Address Size}}$ — this is the multiplier at every indirection level (single = $k$, double = $k^2$, triple = $k^3$).
- 🗂️ **FAT = "linked list, but pointers live in a separate table"** instead of inside each data block (unlike pure linked allocation).
- 🧮 **Counting method for free space** exploits: *"free blocks tend to come in contiguous runs"* — store (start, count) pairs instead of every single address.

---

## 15. Quick Revision Checklist ✅

**File System**
- [ ] Physical vs Logical disk structure
- [ ] Contiguous vs Linked vs Indexed allocation — pros/cons, external fragmentation (only Contiguous has it)
- [ ] Unix i-node: direct, single/double/triple indirect blocks
- [ ] Master formula: Max File Size using $k = \text{DBS}/\text{DBA}$
- [ ] Can compute Max Disk Block Address size from "N addresses per block"
- [ ] FAT structure: directory entry (first DBA) + FAT chain + EOF marker (−1)
- [ ] FAT12 / FAT16 / FAT32 — rough size limits & years
- [ ] Free space management: Bitmap, Linked List, Grouping, Counting — pros/cons of each
- [ ] Formula: bitmap size = N bits; free-list-vs-bitmap crossover: $F \times D < B$
- [ ] Block size trade-off: bigger block → better throughput, worse space utilization (internal fragmentation)
- [ ] "Anatomy of a Save" — 5-step pipeline (request → housekeeping/bitmap check → i-node/dir entry creation → indexed allocation → physical write)

**Disk Scheduling**
- [ ] Access time = Seek time + Rotational latency; Disk bandwidth formula
- [ ] FCFS — simple, no optimization, can be very inefficient
- [ ] SSTF — nearest request first, can starve far requests
- [ ] SCAN vs LOOK — SCAN always reaches physical disk boundary; LOOK stops at last request
- [ ] C-SCAN vs C-LOOK — circular versions; jump back without servicing; more uniform cylinder-wait fairness
- [ ] Be able to redo the "98,183,37,122,14,124,65,67 from head=53" example for **any** algorithm
- [ ] Spooling (disk, cross-job) vs Buffering (RAM, same-job)

---

*Compiled from GeeksforGeeks GATE OS Lecture 30 (File System + Disk Scheduling) — for personal GATE revision use.*
