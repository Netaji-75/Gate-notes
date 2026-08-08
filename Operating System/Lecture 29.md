# File System — GATE CS/IT OS Notes (Lecture 29)

> Source: Principles of Operating Systems — Lecture 29, "File System" (GeeksforGeeks GATE CS&IT series)

## 📑 Topics Covered

| # | Topic |
|---|-------|
| 1 | Physical Disk Structure (platters, tracks, sectors, cylinders) |
| 2 | Logical Disk Structure (partitioning & formatting) |
| 3 | File System Interface (file concept, attributes, operations, directory structures) |
| 4 | File System Implementation (layered architecture, allocation methods) |
| 5 | File Allocation Methods (Contiguous, Linked, Indexed) |
| 6 | Numerical formulas & worked examples |
| 7 | GATE PYQs & practice questions |

---

## 1. The Big Picture — "The Grand Illusion"

The **File System** is the **visible part of the OS** — it is the invisible engine that translates human-friendly logical concepts into physical hardware constraints.

| Layer | What it looks like | Description |
|---|---|---|
| **User Illusion** | Neatly organized folders & files | Abstract, hierarchical, human-readable |
| **File System** | The invisible engine | Maps logical concepts → physical hardware in milliseconds |
| **Hardware Reality** | Raw magnetic sectors / flash cells | Chaotic, bound by strict physical geometry |

**System stack (CPU → Disk):**

```
CPU → Application → File System Software → Disk Manager → Device Driver → Interface (SATA/IDE/SCSI) → Disk
```

- **File System & Device Management flow:** `Operating System ↔ File System ↔ Files`
- A **user** accesses information as files & directories; the **OS** is responsible for actually retrieving/storing that information on physical media (disks, USB, CD, etc.)

---

## 2. Physical Disk Structure

### Key Components

| Component | Description |
|---|---|
| **Spindle** | Central rotational axis; typically 3600+ RPM |
| **Platter** | The magnetic physical storage medium (disk itself) |
| **Actuator Arm & Read/Write Head** | Mechanical apparatus that moves to read/alter magnetic states on the spinning platter |

### Disk Geometry (Hierarchy)

```
Disk → Platters → Surface → Tracks → Sectors/Blocks
```

| Term | Meaning |
|---|---|
| **Track** | A single concentric circular ring on one platter |
| **Sector** | Smallest physical storage unit on a track (historically **512 bytes**) |
| **Cluster** | A grouping of adjacent sectors treated as a single unit by the OS |
| **Cylinder** | A vertical stack of identical tracks across multiple platters (same track # on all platters) |

### 💡 Disk I/O Time Formula

$$
\text{Disk I/O Time (per sector)} = \text{Seek Time (ST)} + \text{Rotational Latency (LT)} + \text{Transfer Time (TT)}
$$

| Symbol | Meaning |
|---|---|
| **ST** (Seek Time) | Time for the R/W head (actuator arm) to move to the correct track |
| **LT** (Rotational Latency) | Time for the platter to rotate so the desired sector is under the head |
| **TT** (Transfer Time) | Time to actually read/write the data once positioned |

---

## 3. Logical Disk Structure (Formatting)

### Workflow: Raw Disk → Usable File System

```
Hard Drive → Partition → File System (e.g. APFS/NTFS) → Directory → Files
```

### Three-Stage Formatting Process

| Stage | Action |
|---|---|
| **1. Low-Level Formatting** | Maps out physical tracks & sectors |
| **2. Partitioning** | Divides disk into isolated logical "districts" (volumes) |
| **3. Logical Formatting** | Installs the actual File System architecture (metadata structures) |

### Partition Types

- **Primary (Bootable)** — can contain an OS that boots
- **Extended (Non-bootable)** — data-only, no OS boot capability

### Boot Sequence (3 steps)

1. **POST** (Power-On Self-Test)
2. **BIOS** (loads boot device info)
3. **Bootstrap** (loads OS from the Boot Loader)

### MBR (Master Boot Record)

- Located at the very start of the disk (before Partition 1)
- Contains:
  - **Partition Table**
  - **Boot Loader**

### Anatomy of a Single Partition (Blueprint)

```
MBR → Partition Boot Sector → File System Area → Data Area
```

Breaking down the **File System Area** further (whiteboard notation):

| Block | Full Name | Contains |
|---|---|---|
| **BCB** | Boot Control Block | Partition Boot Sector; in UFS called the "Boot Block" |
| **PCB** | Partition Control Block | UFS → **Super Block**; NTFS/FAT → **Master File Table (MFT)** |
| **Directory Structure** | — | Metadata mapping filenames to blocks |
| **Data Blocks (DB)** | — | Actual file content |

> **SA** = System Area (BCB + PCB + Directory Structure); **UA** = User Area (Data Blocks)

### Disk Layout with Multiple Partitions

```
[MBR][1st Partition Boot Sector][FS Area of P1][Data Area of P1] | [FS Area of P2][Data Area of P2]
```

---

## 4. The File System Landscape (OS ↔ File System mapping)

| OS Environment | Active Standard | Legacy/Alternative | Key Engineering Features |
|---|---|---|---|
| **Windows** | **NTFS** | FAT32 / exFAT | Journaling, file encryption, exFAT optimized for removable flash |
| **macOS** | **APFS** | HFS+ | Optimized for SSD/Flash, instant snapshots, space-sharing |
| **Linux** | **Ext4** | Ext3 | Robust journaling, high performance, large volume support |
| **Android** | **F2FS** | ext4 | Flash-Friendly File System, optimized for mobile flash storage |

**Details:**
- **NTFS** (New Technology File System) — Windows NT onward; supports permissions, encryption, large files.
- **FAT32** — older, widely compatible; used for USB drives/memory cards for cross-platform compatibility.
- **exFAT** — Microsoft, 2006; optimized for flash memory (USB/SD cards).
- **HFS+** — predecessor to APFS; supports journaling but lacks modern SSD optimizations.
- **APFS** (2017) — optimized for SSD/Flash; supports encryption, snapshots, space-sharing.
- **Ext4** — default on most Linux distros; large file/volume support, journaling.
- **F2FS** — designed for flash storage common in phones/tablets.

---

## 5. File System Interface

### 5.1 File Concept

> **File** = a collection of logically related records of an entity.

- A **File is an Abstract Data Type (ADT)**: `<Definition, Structure, Operations, Attributes>`
- Internal structure can be:
  - **Flat** (just bytes — e.g., Unix-style)
  - **Records** → can be organized **hierarchically**

### 5.2 File Attributes

A file's attributes vary by OS but typically include:

| Attribute | Description |
|---|---|
| **Name** | Symbolic file name — the only human-readable info |
| **Identifier** | Unique non-human-readable numeric tag within the FS |
| **Type** | Needed for systems supporting multiple file types |
| **Location** | Pointer to device & block address |
| **Size** | Current size (bytes/words/blocks) + possibly max allowed size |
| **Protection** | Access-control info (who can read/write/execute) |
| **Time, date, user ID** | Creation, last modification, last use — useful for security/monitoring |

📌 **Where are attributes stored?**

| OS | Storage Structure |
|---|---|
| **UNIX/Linux** | **I-Node** |
| **Windows/DOS** | **Directory Entry** |

Both are generically called a **File Control Block (FCB)**.

### 5.3 File Operations

**Six core operations:**

1. **Create**
2. **Open**
3. **Read / Write / Modify / Truncate**
4. **Copy / Move / Rename / Concatenate**
5. **Close**
6. **Delete**

Triggered via two paths:
- **Commands** (interactive users)
- **System Calls** (programmers)

**Detailed semantics:**

| Operation | What happens |
|---|---|
| **Create** | 2 steps: (1) find/allocate space in FS, (2) make a directory entry for the new file |
| **Write** | System call specifies filename + data; OS searches directory for file location; **write pointer** tracks where next write occurs and updates after each write |
| **Read** | System call specifies filename + destination memory location; directory searched; **read pointer** tracks next read location |
| **Reposition (Seek)** | Directory searched; **current-file-position pointer** moved to a given value — does **not** require actual I/O |
| **Delete** | Directory searched for file; associated space released for reuse; directory entry erased |
| **Truncate** | Erases file *contents* but **keeps attributes** — resets length to 0, releases file space, but doesn't force delete+recreate |

### 5.4 File Types (by Extension)

| File Type | Usual Extension | Function |
|---|---|---|
| Executable | exe, com, bin, none | Ready-to-run machine-language program |
| Object | obj, o | Compiled, machine language, not linked |
| Source code | c, cc, java, pas, asm, a | Source code in various languages |
| Batch | bat, sh | Commands to the command interpreter |
| Text | txt, doc | Textual data, documents |
| Word processor | wp, tex, rtf, doc | Various word processor formats |
| Library | lib, a, so, dll | Libraries of routines for programmers |
| Print/view | ps, pdf, jpg | ASCII/binary formatted for printing/viewing |
| Archive | arc, zip, tar | Related files grouped (often compressed) |
| Multimedia | mpeg, mov, rm, mp3, avi | Binary audio/A-V file |

### 5.5 Directory Structures — Evolution

The two fundamental problems any directory design must solve:
- **Naming problem** — avoiding filename collisions
- **Grouping problem** — logically organizing related files

| Structure | Concept | Flaw / Benefit |
|---|---|---|
| **1. Single-Level** | One master directory for **all** users | ❌ Massive naming collisions; **no grouping** |
| **2. Two-Level** | Separate **User File Directory** per user, under a **Master File Directory** | ✅ Efficient searching, same filename allowed across users ❌ Still **no internal grouping** capability |
| **3. Tree-Structured** | Root → Branches → Leaves (modern standard) | ✅ Enables absolute/relative **path names**, deep grouping |
| **4. General Graph** | Allows **cycles** and **links** (shortcuts); one file can exist in multiple directories simultaneously | ✅ Maximum flexibility ❌ Needs cycle-detection; complicates deletion |

> Handwritten note: General Graph Directory structure is used to implement **file sharing**, formally modeled as a **DAG (Directed Acyclic Graph)**.

**Two-Level Directory — path name example:**
```
Master File Directory: User1 | User2 | User3 | User4
User File Directory:   cat,bo,a,test | a,data | a,test | x,data,a
```

---

## 6. File System Implementation

### 6.1 Layered File System Architecture

```
Application Programs
        ↓
Logical File System
        ↓
File-Organization Module
        ↓
Basic File System
        ↓
I/O Control
        ↓
Devices
```

| Layer | Responsibility |
|---|---|
| **Logical File System** | Manages **metadata** (everything except actual file content). Maintains file structure via **File Control Blocks (FCB)** — called an **I-node** in UNIX. Provides symbolic-name → info mapping to the layer below. |
| **File-Organization Module** | Knows about files & their **logical blocks**, as well as **physical blocks**. Translates logical block addresses → physical block addresses (based on allocation method used). Includes the **free-space manager** (tracks unallocated blocks). |
| **Basic File System** | Issues **generic commands** to the appropriate device driver to read/write physical blocks (e.g., "drive 1, cylinder 73, track 2, sector 10"). Manages memory buffers/caches for FS, directory, and data blocks. |
| **I/O Control** | Consists of **device drivers** + **interrupt handlers**. Device driver = translator: input is high-level ("retrieve block 123"), output is low-level hardware-specific instructions. Uses **DMA** (Direct Memory Access) for efficient transfer. |
| **Devices** | Physical hardware (disks, SSDs, etc.) |

### 6.2 Full FS Architecture (alternate view)

```
User Application → Logical File System → Virtual File System → Physical File System → [Partition 1 | Partition 2 | Partition 3]
```

### 6.3 Logical File Attributes (example: `data_report.pdf`)

| Attribute | Value/Meaning |
|---|---|
| Identifier | Unique non-human numerical tag |
| Name | Symbolic human-readable string |
| Location | Pointer to physical device + block address |
| Size | Current bytes + maximum limit |
| Protection | Access control (R/W/X) |
| Timestamps | Creation, modification, last use |

**Operation pipeline (logical file):**
```
Create → Write → Read → Seek → Truncate → Delete
```

---

## 7. File Allocation Methods ⭐ (High-weight GATE topic)

> **Allocation Method** = how disk blocks are allocated to files. Disk blocks are numbered sequentially `0..n`; mapping to actual track/sector is handled at a lower level.

```
File Allocation Methods
   ├── Contiguous Allocation
   └── Non-Contiguous Allocation
          ├── Linked List Allocation
          └── Indexed Allocation
```

### 7.1 Contiguous Allocation

**Concept:** File occupies a set of *strictly contiguous* blocks. Directory only needs **starting block address + length**.

| Directory Entry | Start (First DBA) | Length (Size) |
|---|---|---|
| count | 0 | 2 |
| f | 6 | 2 |
| tr | 14 | 3 |
| mail | 19 | 6 |
| list | 28 | 4 |

✅ **Advantage:** Maximum performance for sequential read/write — minimal seek time, R/W head moves linearly.

❌ **Flaw:** Severe **external fragmentation** — as files are created/deleted, unusable gaps form; file growth is strictly limited.

**Performance issue summary:**

| Issue | Contiguous Allocation |
|---|---|
| Internal Fragmentation | Yes |
| External Fragmentation | **Yes** |
| Increasing existing file size | **Inflexible** |
| Type of access supported | Sequential / Direct (**faster** access) |

---

### 7.2 Linked (List) Allocation

**Concept:** File = a **linked list of scattered disk blocks**. Directory holds pointer to **first and last block**; each block points to the next.

```
directory: File | start | end
           Jeep  |  9    | 25
```

✅ **Advantage:** Completely **eliminates external fragmentation**. Files can grow indefinitely as long as free blocks exist.

❌ **Flaw:** Disastrous for **random access** — to reach block 100, OS must traverse blocks 1 through 99. Pointers consume extra disk space overhead.

**Performance issue summary:**

| Issue | Linked Allocation |
|---|---|
| Internal Fragmentation | Yes |
| External Fragmentation | **Never** |
| Increasing existing file size | **Flexible** |
| Type of access supported | **Only Sequential** (slow) |
| Links/pointers consume disk space | Yes |
| Vulnerability of pointers | A single corrupted pointer breaks the whole chain |

---

### 7.3 Indexed Allocation

**Concept:** Each file is associated with an **Index Block**, which contains information about all the (scattered) data blocks in use by the file. Directory just stores the **index block number**.

```
directory: File | index block
           Jeep  |    19
Index Block[19] = {16, 1, 10, 25, -1, -1, -1, ...}
```

✅ **Advantage:** The "gold standard" for **rapid random access**, without external fragmentation.

❌ **Flaw:** **Pointer overhead** — even a tiny file needs an entire dedicated Index Block.

### Allocation Comparison Table

| Criterion | Contiguous | Linked | Indexed |
|---|---|---|---|
| External Fragmentation | Yes (severe) | Never | Never |
| Sequential Access | Excellent (fastest) | Good (only mode supported) | Good |
| Random/Direct Access | Good (if size known) | Poor (very slow) | **Excellent** |
| File Growth | Inflexible | Flexible | Flexible |
| Overhead | None (just start+length) | 1 pointer/block | 1 index block/file |
| Space wasted | External frag gaps | Pointer storage | Index block (esp. for tiny files) |

---

## 8. Key Formulas & Worked Numericals

### 8.1 Max File Size vs Max Disk Size (using Disk Block Address)

$$
\text{Max. Possible Disk/File Size} = 2^{\text{DBA}} \times \text{DBS}
$$

| Symbol | Meaning |
|---|---|
| **DBA** | Disk Block Address size, in **bits** (number of bits used to address a block) |
| **DBS** | Disk Block Size, in **bytes** |

> ⚠️ Constraint: **Actual given disk size ≤ Maximum possible disk size**

**Worked Example 1:**
- Given: `DBA = 16 bits`, `DBS = 1 KB`
- $2^{16} \times 1\text{KB} = 65536 \text{ KB} = 64\text{ MB}$
- ✅ **Max possible disk size = 64 MB**

### 8.2 Number of Blocks Needed for a File (with Internal Fragmentation)

$$
N = \frac{\text{File Size}}{\text{DBS}}
$$

If `N` isn't an integer, round **up**, and compute Internal Fragmentation (**I.F.**) = (blocks allocated × DBS) − File Size.

**Worked Example 2 (Q1):**
- `DBS = 512 B`, `File = 1 MB`
- $N = \dfrac{1\text{MB}}{512\text{B}} = \dfrac{2^{20}}{2^{9}} = 2^{11} = 2048$ blocks
- **Number of blocks = 2048**, **Internal Fragmentation = 0** (divides evenly)

**Worked Example 3 (Q2):**
- `DBS = 2 KB`, `File Size = 55 KB`
- $N = \dfrac{55\text{KB}}{2\text{KB}} = 27.5 \rightarrow$ round up to **28 blocks**
- **Internal Fragmentation = (28 × 2KB) − 55KB = 56KB − 55KB = 1 KB**

### 8.3 Indexed Allocation — Max File Size with ONE Index Block

$$
\text{Max File Size (1 Index Block)} = \left(\frac{\text{DBS}}{\text{DBA (in bytes)}}\right) \times \text{DBS}
$$

i.e., **Number of entries (N) in one Index Block** = $\dfrac{\text{DBS}}{\text{DBA}}$ (DBA here in bytes, not bits), then:

$$
\text{Max File Size} = N \times \text{DBS}
$$

**Worked Example 4:**
- `DBA = 16 bits = 2 Bytes`, `DBS = 1 KB`
- $N = \dfrac{1\text{KB}}{2\text{B}} = 512 \text{ entries}$
- **Max File Size = 512 × 1KB = 512 KB**

**Worked Example 5 (Q2 on slide):**
- `DBA = 32 bits = 4 Bytes`, `DBS = 8 KB`
- $N = \dfrac{8\text{KB}}{4\text{B}} = 2K$ entries
- **Max Possible File Size (with 1 Index Block) = 2K × 8KB = 16 MB**

**Worked Example 6 (Q3 on slide):**
- Given: a block of size 2 KB can store **512 entries** of data block addresses
- $\Rightarrow \text{DBA} = \dfrac{2\text{KB}}{512} = 4 \text{ Bytes} = 32\text{ bits}$
- **File Size = 512 × 2KB = 1 MB**

### 8.4 General Relation (Index Block)

$$
\boxed{\text{DBA} = \frac{\text{DBS}}{N}}
$$

Where **N** = number of entries (Disk Block Addresses) that fit in one Index Block.

---

## 9. GATE PYQ Bank (with reasoning)

> ⚠️ Note: Several PYQ slides show only the **question stem** without visible marked answers/options in the source PDF. These are transcribed below exactly as shown; solve as practice using the formulas above. Where the correct option was legible, it is marked.

### Q1. Paging — Page Fault Trace (Belady's Anomaly)
**Q:** For a certain page trace starting with no page in memory, a demand-paged memory system operated under the LRU replacement policy results in 9 and 11 page faults when the primary memory is of 6 and 4 pages, respectively. When the same page trace is operated under the optimal policy, the number of page faults may be:

- A. 9 and 7
- B. 7 and 9
- C. 10 and 12
- D. 6 and 7

**Reasoning:** Optimal replacement is *provably* the best possible — it must always produce **fewer or equal** page faults than LRU for the *same* memory size, and reducing memory size can never *decrease* page faults (no Belady's-anomaly-immune guarantee needed here, optimal is always monotonic — more frames ⇒ fewer or equal faults). So for 6 frames, optimal faults ≤ 9; for 4 frames, optimal faults ≤ 11, **and** optimal(6 frames) ≤ optimal(4 frames).
- ✅ **Correct: D (6 and 7)** — both values are ≤ the corresponding LRU values, and 6 ≤ 7 (monotonic w.r.t. frame count).
- ❌ A, B: 9 is not less than LRU's 9 (should be strictly less generally, and ordering with 7 is wrong pairing); B has smaller value paired with fewer frames, contradicting monotonicity direction.
- ❌ C: both values exceed the corresponding LRU values — impossible since optimal can't be worse than LRU.

*(Note: this is a Paging/Memory Management PYQ referenced within the lecture — included here as it appeared in the source slides.)*

### Q2. Sharing in Paged Memory
**Q:** Sharing in a paged memory system is done by:
- A. Giving a copy of the shared pages to each process
- B. Dividing the program into procedures and data and allowing only the procedures to be shared
- C. **Several page table entries pointing to the same frame in the main memory** ✅
- D. None of the above

**Reasoning:** Page-level sharing means multiple processes' page tables map different logical pages to the **same physical frame**.
- ❌ A: Giving copies defeats the purpose of *sharing* (that's duplication, not sharing).
- ❌ B: Restricting sharing to only procedures is an implementation detail, not the general mechanism.
- ❌ D: C is correct, so "none" is wrong.

### Q3. Demand-Paging — Good/Bad Programming Structures
**Q:** Which of the following programming techniques/structures are "good" vs "bad" for a demand-paged environment? Explain.
(a) Stack (b) Hashed symbol table (c) Sequential search (d) Binary search (e) Pure code (f) Vector operations (g) Indirection

**Reasoning (general GATE theory answer):**
- **Good (locality-friendly):** Stack (sequential push/pop = good spatial locality), Sequential search (accesses memory in order), Pure code (read-only, reused across processes, no need to write back), Binary search *can* be less good since it jumps around, but touches few pages overall for large arrays — often considered acceptable.
- **Bad (poor locality):** Hashed symbol tables (random access pattern — hash spreads keys across many pages), Vector operations on huge/sparse vectors (large strides = poor locality), Indirection (each pointer dereference can hit a different page → frequent faults).

### Q4. Demand-Paging — CPU Utilization Tuning
**Q:** Given CPU utilization = 20%, Paging disk utilization = 97.7%, Other I/O devices = 5%. For each of the following, say whether it will (or is likely to) improve CPU utilization:
(a) Faster CPU (b) Bigger paging disk (c) ↑ degree of multiprogramming (d) ↓ degree of multiprogramming (e) More main memory (f) Faster/multiple hard disks (g) Prepaging (h) Increase page size

**Reasoning:** The system is clearly **disk-I/O bound** (paging disk at 97.7% = the bottleneck), not CPU-bound.
- ❌ (a) Faster CPU — won't help; CPU isn't the bottleneck.
- ❌ (b) Bigger paging disk — size ≠ speed; doesn't reduce I/O wait.
- ❌ (c) ↑ multiprogramming — will make thrashing **worse** (more processes competing for pages → more disk I/O).
- ✅ (d) ↓ multiprogramming — reduces thrashing, frees up disk I/O contention → **improves CPU utilization**.
- ✅ (e) More main memory — reduces page faults directly → **improves CPU utilization**.
- ✅ (f) Faster/multiple disks with multiple controllers — directly attacks the disk bottleneck → **improves CPU utilization**.
- ✅ (g) Prepaging — can reduce the number of individual page faults by loading pages proactively → **may help**.
- ⚠️ (h) Increase page size — mixed: may reduce page-fault *count* but wastes memory (internal fragmentation) and each fault takes longer to service; not a reliable fix.

### Q5. Disk Specifications — Numerical
**Q:** Given: Platters = 16, Tracks/Surface = 512, Sectors/Track = 2048, Sector offset = 12 bits, Avg Seek Time = 30 ms, Disk RPM = 3600. Calculate:
(A) Unformatted Capacity of Disk
(B) IO-Time/Sector
(C) Data Transfer Rate
(D) Sector Address

**Approach (formulas to apply):**
- Sector size = $2^{12} = 4096$ bytes (from 12-bit offset)
- Surfaces = 16 platters × 2 = 32 (if double-sided) — **check whether question means 16 surfaces or 16 platters**
- Capacity = Surfaces × Tracks/Surface × Sectors/Track × Sector Size
- Rotational Latency = $\dfrac{60}{\text{RPM}} \times \dfrac{1}{2}$ seconds (average = half rotation)
- IO-Time/Sector = Seek Time + Rotational Latency + Transfer Time (per formula in §2)
- Data Transfer Rate = Sector Size / Transfer Time per sector
- Sector Address = bits for (Surface # + Track # + Sector #)

### Q6. Loading a Program from Disk (with page distribution)
**Q:** How long does it take to load a 64 KB program from a disk with Avg Seek time = 30 ms, Rotation time = 20 ms, Track Size = 32 KB, Page Size = 4 KB? Pages distributed randomly. What % time is saved if 50% of pages are contiguous?

**Approach:** Random pages each incur full seek+rotation+transfer; contiguous pages amortize seek+rotation across the whole run, saving time relative to per-page overhead.

### Q7. Cylinders on Disk — Numerical
**Q:** 512 GB disk, 32 storage surfaces, 4096 sectors/track, 1024 bytes/sector. Find number of cylinders.

**Formula:**
$$
\text{Cylinders} = \frac{\text{Total Disk Size}}{\text{Surfaces} \times \text{Sectors/Track} \times \text{Bytes/Sector}}
$$

### Q8. Linear List Directory — Full Scan Requirement
**Q:** Consider a Linear-List directory implementation. Which operation(s) *necessarily* require a full scan of directory "Foo" for successful completion?
- A. Opening an existing file in Foo
- B. **Creation of a new file in Foo** ✅
- C. **Renaming of an existing file in Foo** ✅
- D. Deletion of an existing file from Foo

**Reasoning:**
- ❌ A (Open): Search can stop as soon as the file is found — **no full scan required** on success.
- ✅ B (Create): Must scan the **entire** list first to confirm the name doesn't already exist (avoid duplicates) before adding — full scan is mandatory.
- ✅ C (Rename): Must scan the **entire** list to ensure the *new* name isn't already taken by another file — full scan mandatory.
- ❌ D (Delete): Search stops once the matching entry is found — no full scan needed.

### Q9. Allocation Scheme with No External Fragmentation
**Q:** Which allocation scheme(s) can be used if **no External Fragmentation** is allowed?
I. Contiguous II. Linked III. Indexed
- A. I and III only
- B. II only
- C. III only
- D. **II and III only** ✅

**Reasoning:** Both Linked and Indexed allocation scatter blocks anywhere on disk (no need for contiguous free space) → **never** cause external fragmentation.
- ❌ Contiguous requires a single contiguous run of blocks → **does** suffer external fragmentation.
- So any option including "I" (Contiguous) is wrong → eliminates A.
- B and C are each individually true but incomplete — D is the most complete correct set.

### Q10. UNIX I-Node — Max File Size
**Q:** UNIX I-Node with 8 direct DBAs + 3 indirect DBAs (Single, Double, Triple). Disk Block Size = 1 KB, each block holds 128 DBAs. Calculate:
(i) Max File Size with this I-Node structure
(ii) Size of Disk Block Address
(iii) Is this file size possible over the given disk?

**Approach:**
- DBA size: Since 1 block (1KB) holds 128 addresses → DBA size = 1KB / 128 = **8 Bytes = 64 bits**
- Max File Size = [8 direct + 128 (single-indirect) + 128² (double-indirect) + 128³ (triple-indirect)] blocks × 1KB
- Whether this is "possible" depends on comparing against **Max Possible Disk Size** = $2^{\text{DBA bits}} \times \text{DBS}$ — if DBA=64 bits, max disk size is astronomically larger than any real disk, so realistically the file size is limited by **actual disk size**, not by the I-node structure.

### Q11. Multi-level Index Table — Max File Size
**Q:** File system stores 128 DBAs in the index table of the Directory. Disk Block Size = 4 KB. If file size < 128 blocks, addresses act as **direct** data block addresses. If file size > 128 blocks, the 128 addresses point to **next-level Index Blocks**, each containing 256 data block addresses. What is Max File Size?

**Approach:**
$$
\text{Max File Size} = 128 \times 256 \times \text{DBS} = 128 \times 256 \times 4\text{KB}
$$
(since each of the 128 top-level entries now points to an index block holding 256 addresses, each addressing one 4KB data block)

### Q12. I-Node with Direct + Single + Double Indirect — Max File Size in GB
**Q:** I-Node has 12 direct, 1 single-indirect, 1 double-indirect pointer. Disk block size = 4 KB, disk block address = 32 bits (4 Bytes). Find max possible file size in GB (1 decimal place).

**Approach:**
- Entries per index block = $\dfrac{4\text{KB}}{4\text{B}} = 1024$
- Max File Size (blocks) = 12 (direct) + 1024 (single-indirect) + 1024² (double-indirect)
- Max File Size = (12 + 1024 + 1,048,576) × 4KB → convert to GB

### Q13. File Descriptor — Max Possible File Size
**Q:** 300 GB Disk. File descriptor: 8 Direct + 1 Indirect + 1 Doubly-Indirect Block Address. Disk Block size = 128 Bytes, Disk Block Address size = 8 Bytes. Max possible file size:
- A. 3 K Bytes
- B. 35 K Bytes
- C. 280 K Bytes
- D. Dependent on the size of the disk

**Approach:**
- Entries per index block = $\dfrac{128\text{B}}{8\text{B}} = 16$
- Max File Size (blocks) = 8 (direct) + 16 (single-indirect) + 16² (double-indirect) = 8 + 16 + 256 = 280 blocks
- Max File Size = 280 × 128 B = **35,840 Bytes ≈ 35 KB**
- ✅ **Correct: B (35 K Bytes)**
- ❌ A: too small — undercounts the indirect blocks' contribution.
- ❌ C: 280 is the **block count**, not the size in KB (forgot to multiply by block size correctly, or confused blocks with KB).
- ❌ D: The file size is fully determined by the I-node structure & block/address sizes — **independent of actual disk size** (as long as disk ≥ this size).

### Q11 (dup ref). Data Blocks of a Very Large File in UNIX FS
**Q:** The Data Blocks of a very large file in the Unix File System are allocated using:
- A. Contiguous allocation
- B. Linked allocation
- C. Indexed allocation
- D. **An extension of indexed allocation** ✅

**Reasoning:** UNIX I-nodes use direct blocks for small files, then **single/double/triple indirect blocks** for large files — this is a **multi-level (extended) indexed allocation**, not plain single-level indexing.
- ❌ A, B: UNIX doesn't use contiguous or pure linked allocation for data blocks.
- ❌ C: Plain (single-level) indexed allocation alone can't scale to very large files — UNIX *extends* it with indirect blocks.

### Q12 (dup ref). Larger Block Size in Fixed-Block File System
**Q:** Using a larger block size in a fixed block-size file system leads to:
- A. **Better Disk Throughput but Poorer Disk Space Utilization** ✅
- B. Better throughput AND better utilization
- C. Poorer throughput but better utilization
- D. Poorer throughput AND poorer utilization

**Reasoning:** Larger blocks mean fewer, bigger I/O transfers per file (fewer seeks needed relative to data volume) → **better throughput**. But larger blocks increase **internal fragmentation** (the last block of most files is only partially used) → **worse space utilization**.
- ❌ B: Utilization does **not** improve with larger blocks — trade-off runs the opposite way.
- ❌ C, D: Throughput actually **improves**, not worsens, with larger blocks (fewer per-block overheads).

### Q13 (dup ref). FAT-based File System — Max File Size
**Q:** FAT-based file system; each FAT entry overhead = 4 bytes. Disk = $100 \times 10^6$ bytes, data block size = $10^3$ bytes (note: likely a typo for "10³" i.e. 1000 bytes in the source). Find max file size (in units of $10^6$ bytes).

**Approach:**
- Number of blocks on disk = Disk Size / Block Size = $\dfrac{100\times10^6}{10^3} = 10^5$ blocks
- FAT needs one entry per block; a file can use **at most all data blocks** (chained via FAT) minus reserved system blocks
- Max File Size ≈ (Total blocks) × Block Size (in the ideal case where a single file uses the whole disk)

### Q14. One-Level Directory File System — Layout Numerical
**Q:** One-level directory FS on disk with Block Size = 4 KB:
- Block 0: Boot Control Block
- Block 1: FAT — one 10-bit entry per Data Block (= next Data Block Address in the file's chain)
- Blocks 2,3: Directory — 32-bit entry per File
- Block 4 onwards: Data blocks

(a) Max possible number of files?
(b) Max possible file size in bytes?

**Approach:**
- (a) Number of directory entries possible = (2 blocks × 4KB) / (32 bits = 4 Bytes per entry) = $\dfrac{8192}{4}$ = 2048 files max
- (b) FAT entry = 10 bits → can address $2^{10} = 1024$ distinct data blocks → Max File Size = 1024 × 4KB = 4 MB (assuming a file could use all data blocks in the chain)

### Q15. Free-Space Management — General Relation
**Q:** Disk has 'B' Blocks, 'F' free. Disk Block Address = 'D' bits, Disk Block Size = 'X' Bytes.
(A) Calculate: (i) Given Disk Size (ii) Max Possible Disk Size (iii) Relation between B & D
(B) When does a Free List use less space than a Bit Map?

**Approach:**
- (i) Given Disk Size = $B \times X$ bytes
- (ii) Max Possible Disk Size = $2^{D} \times X$ bytes
- (iii) Relation: $B \leq 2^{D}$ (must be addressable)
- (B) **Free List uses less space than Bit Map when the disk is nearly full** (few free blocks, F is small) — because Free List cost ∝ F × D bits, while Bit Map cost is always B bits (fixed, regardless of how many are free). Free List wins when $F \times D < B$.

### Q16. Free Space Bit-Map — Hex Trace
**Q:** Bit-map starts as `1000 0000 0000 0000` (block 0 = Root Directory). After File A (6 blocks) is written: `1111 1110 0000 0000`. Show the Bit-Map after:
- (A) File B written, using 5 blocks
- (B) File A is deleted
- (C) File C written, using 8 blocks
- (D) File B is deleted

**Approach:** System always allocates the **lowest-numbered free blocks first**.
- Start: `1111 1110 0000 0000` (bits 0–6 used, rest free)
- **(A) File B (5 blocks)** → next 5 free bits (7–11) turn to 1 → `1111 1111 1110 0000`
- **(B) File A deleted** (frees blocks 1–6, i.e., bits 1–6, since bit 0 is Root Dir and stays 1) → `1000 0001 1110 0000`
- **(C) File C (8 blocks)** → allocate lowest free bits first: bits 1–6 (6 bits, freed by A) + bits 12–13 (2 more needed) → `1111 1111 1111 1000`
- **(D) File B deleted** (frees bits 7–11, which were allocated to B) → `1111 1110 0001 1000`

*(Convert each 4-bit nibble group to hex as needed for final "HEX Code" answer format.)*

### Q17. Disk Cache — Miss Rate vs Cache Size
**Q:** In-memory cache for disk blocks. Cache hit latency = 1 ms, disk latency = 10 ms. Cost of checking cache membership ≈ 0. Cache sizes available in multiples of 10 MB. Given a Miss-Rate vs Cache-Size graph (starts ~80% at 10MB, decreasing to ~15% at 80MB). Find the **smallest cache size** for avg. read latency < 6 ms.

**Formula:**
$$
\text{Avg. Latency} = (1-\text{MissRate}) \times 1\text{ms} + \text{MissRate} \times (1\text{ms} + 10\text{ms})
$$
(simplify to: $\text{Avg. Latency} = 1\text{ms} + \text{MissRate} \times 10\text{ms}$)

Set $< 6$ ms → $\text{MissRate} < 0.5$ (50%) → read off the graph the smallest cache size where miss rate drops below 50% (appears to be around **30 MB**, where miss rate ≈ 41%).

### Q18. Worst-Case Disk Space for Page Storage
**Q:** Disk space for Page storage relates to Max number of Processes 'N', Bytes in Virtual Address Space 'B', Bytes in RAM 'R'. Give expression for worst-case Disk Space required.

**Formula:**
$$
\text{Worst-case Disk Space} = N \times B - R
$$

**Reasoning:** In the absolute worst case, every one of the N processes could have its **entire virtual address space** (B bytes) resident on disk (none of it cached in physical RAM at once for that accounting), but since RAM (R bytes) is holding *some* pages for *some* process at any instant, the disk only needs to hold the swapped-out remainder: total virtual memory demand across all processes ($N \times B$) minus what's currently held in RAM ($R$).

---

## 10. Memory Tricks / Mnemonics 🧠

- **"File System is the visible part of the OS"** — everything else (schedulers, memory managers) works invisibly in the background; the FS is what the *user* directly interacts with.
- **Disk hierarchy mnemonic:** *Disk → Platters → Surface → Tracks → Sectors/Blocks* (top-down zoom-in, like Russian nesting dolls).
- **"The Grand Illusion"** — Users see neat folders; the FS is the invisible translator engine; hardware is chaotic raw magnetic sectors. Remember these as **3 layers of abstraction**.
- **FCB analogy:** Think of the FCB as a file's "ID card" — UNIX calls this ID card an **I-node**; Windows/DOS calls it a **Directory Entry**.
- **Allocation trade-off mnemonic:**
  - **Contiguous** = "a Book on a shelf" → fast to read sequentially, but hard to grow (no room to expand pages without breaking the shelf order).
  - **Linked** = "a Treasure Hunt" → follow the clue (pointer) to the next block; flexible but slow to jump ahead.
  - **Indexed** = "a Table of Contents" → one lookup gets you the page number for any chapter (random access), but you always need that ToC page even for a 1-page pamphlet (overhead).
- **DBA vs DBS relation:** Remember $\text{Max Disk/File Size} = 2^{\text{DBA}} \times \text{DBS}$ — "**how many addresses you can make, times how big each block is**."

---

## 11. Quick Revision Checklist ✅

Use this to self-test before re-reading the full notes:

- [ ] Can you draw the CPU → App → FS Software → Disk Manager → Device Driver → Disk stack?
- [ ] Can you list the disk geometry hierarchy: Disk → Platter → Surface → Track → Sector?
- [ ] Can you state the Disk I/O Time formula and define ST, LT, TT?
- [ ] Do you know the 3-step formatting process (Low-level format → Partition → Logical format)?
- [ ] Can you explain what's inside MBR (Partition Table + Boot Loader)?
- [ ] Can you name the 3 boot steps (POST → BIOS → Bootstrap)?
- [ ] Do you know the BCB / PCB / Directory Structure / Data Block layout of a partition?
- [ ] Can you match each OS to its default file system (Windows→NTFS, macOS→APFS, Linux→Ext4, Android→F2FS)?
- [ ] Can you define a File as an ADT and list its 7 standard attributes?
- [ ] Do you know FCB = I-node (UNIX) = Directory Entry (Windows/DOS)?
- [ ] Can you list all 6 file operations and explain read/write/seek pointers?
- [ ] Can you compare all 4 directory structures (Single/Two-Level/Tree/General Graph) with their flaws?
- [ ] Can you draw the 5-layer File System implementation stack (Logical FS → File-Org Module → Basic FS → I/O Control → Devices)?
- [ ] Can you compare **Contiguous vs Linked vs Indexed** allocation on: external frag, sequential vs random access, flexibility, overhead?
- [ ] Can you derive: Max File/Disk Size = $2^{\text{DBA}} \times \text{DBS}$?
- [ ] Can you compute number of blocks needed + internal fragmentation for a given file size & block size?
- [ ] Can you compute Max File Size for Indexed Allocation with 1 index block: $N = \text{DBS}/\text{DBA}$, then Max Size $= N \times \text{DBS}$?
- [ ] Can you extend this to multi-level (double/triple) indexed allocation as used in UNIX I-nodes?
- [ ] Do you remember: Free List is cheaper than Bit Map only when disk is **nearly full** (few free blocks)?
- [ ] Can you solve a free-space bit-map trace question (alloc lowest-numbered free block first)?

---

*End of notes — Lecture 29: File System.*
