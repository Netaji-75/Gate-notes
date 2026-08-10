# Memory Management — Lecture 22 (GATE CS/IT — Principles of Operating Systems)

> Source: GeeksforGeeks GATE CS&IT — Principles of Operating Systems, Lecture 22 (Memory Management, Part III)

## 📑 Table of Contents

| # | Topic |
|---|-------|
| 1 | Memory Hierarchy / Memory Subsystem |
| 2 | Linking — Static vs Dynamic |
| 3 | Object File vs Executable File (.obj vs .exe) |
| 4 | Address Binding (Compile / Load / Run Time) |
| 5 | Multistep Processing of a User Program |
| 6 | Design of a Memory Manager — Goals & Functions |
| 7 | Memory Management Techniques (Contiguous vs Non-Contiguous) |
| 8 | Overlays |
| 9 | Partitions — Fixed (MFT) vs Variable (MVT) |
| 10 | Base & Limit Registers (Protection) |
| 11 | Relocation Register (Address Translation) |
| 12 | Allocation Policies (First/Best/Worst/Next Fit) |
| 13 | Fragmentation (Internal vs External) |
| 14 | Compaction |
| 15 | Worked Examples |
| 16 | GATE PYQs / Review MCQs |
| 17 | Mnemonics & Quick Tips |
| 18 | Quick Revision Checklist |

---

## 1. Memory Hierarchy / Memory Subsystem

The memory hierarchy arranges storage devices by **speed, cost, and capacity** — as you go down the pyramid, capacity increases but speed decreases (and cost per bit decreases).

```
Level 0 → CPU Registers            (fastest, smallest, most expensive)
Level 1 → Cache Memory (SRAM)
Level 2 → Main Memory (DRAM)
Level 3 → Magnetic Disk (Disk Storage)
Level 4 → Optical Disk
Level 5 → Magnetic Tape            (slowest, largest, cheapest)
```

**Key property:** *Increase in cost per bit* ↑ as you go **up**; *Increase in capacity & access time* ↑ as you go **down**.

### Detailed characteristics

| Level | Volatility | Use Duration | Example | Speed/Cost |
|---|---|---|---|---|
| Registers | Volatile | Near-instantaneous use | Processor registers | Very fast, very expensive |
| Cache | Volatile | Immediate-term use | Processor cache | Very fast, expensive |
| RAM | Volatile | Very short-term use | Random Access Memory | Fast, affordable |
| Flash/USB | Non-volatile | Short-to-longer term use | Flash, USB memory | Slower, fairly cheap |
| HDD | Non-volatile | Long-term use | Hard disk drives | Slower, very cheap |
| Tape | Volatile* | Very long-term use | Tape backup media | Very slow, very cheap |

*(as shown on slide — tape is typically non-volatile in reality, but marked this way in the lecture)*

---

## 2. Linking — Static vs Dynamic

**Linking** = the process of **resolving external references** (variables/functions) between separately compiled files.

### Example walkthrough
```c
// kk.c
extern int x;
main() {
    foo();
    scanf(...);   // BSA (Base+something Address) of scanf resolved here
}
```
```c
// jk.c
int x;   // definition of the external variable declared in kk.c
```

- `kk.c` **declares** `extern int x`; `jk.c` **defines** `int x`. The linker resolves the reference between the two object files.
- Function calls like `foo()` and library calls like `scanf()` also need their addresses resolved by the linker before/at load.

### Types of Linking

| Type | When Resolved | Notes |
|---|---|---|
| **Static Linking** | Before run-time (at compile/link time) | Library code is copied into the final executable |
| **Dynamic Linking** | At run time | Library code stays external (DLL); only a *stub* is linked in |

---

## 3. Object File vs Executable File (`.obj` vs `.exe`)

- Compiling `main.c` → produces `main.obj` (object file) which contains a **stub** for `scanf()` instead of the full function body.
- At link/load time, this stub is resolved either:
  - **Statically**: actual `scanf()` code (~10 KB) is copied into `main.exe`.
  - **Dynamically**: stub points to a **DLL** (Dynamically Linked Library) which is loaded/shared at runtime.
- **Reusability** is the key benefit of dynamic linking — multiple programs can share one copy of the library code in memory instead of each `.exe` embedding its own copy.

---

## 4. Address Binding

**Address Binding** = the **association of instructions and data units to memory locations (addresses)**.

### Example
```c
int z;
f() {
    int a;          // local, has limited "extent" (lifetime)
    static int a;    
    {
        int y;
    }
}
int e;
e = 5;
```
- Variables have an **extent** (lifetime/scope) — e.g., `int y` only exists inside its inner block.
- Binding maps a variable (like `e`) to a **physical memory address** (e.g., address holding value `5`).

**Illustration:** For `int a, b, c; c = a + b;`, the compiler generates instructions like:
```
I1: LD  R1, a
I2: LD  R2, b
I3: ADD R1, R2
I4: ST  c, R1
```
Each variable (`a`, `b`, `c`) must be bound to a memory location before these instructions can execute.

### Binding Types

```
Binding
 ├── Binding Type
 │      ├── Static
 │      └── Dynamic
 └── Binding Time  ⭐ (Most Important)
        ├── Compile Time
        ├── Load Time
        └── Run Time
```

### Value Binding vs Address Binding (illustrated with `int x = 5;`)

| Binding | When | What Happens |
|---|---|---|
| **Address Binding (Load Time)** | At load time | Variable `x` is assigned a memory location (e.g., address `2000`) |
| **Value Binding (Run Time)** | At run time | The actual value `5` is stored into that location via an instruction like `STORE x, #5` |

> 🔑 **Key takeaway:** The *address* of `x` may be fixed at load time, but the *value* is bound only when the instruction actually executes (run time).

---

## 5. Multistep Processing of a User Program

A source program goes through several stages before it becomes a running process:

```
Source Program
     ↓ (Compiler)
Object File  ←── Other Object Files
     ↓ (Linker)
Executable File
     ↓ (Loader)
Program in Memory (Process)  ←── Dynamically Linked Libraries
```

| Stage | Time Phase |
|---|---|
| Source → Compiler → Object File | **Compile Time** |
| Object File → Linker → Executable File | **Load Time** (linking happens here or later) |
| Executable File → Loader → Program in Memory | **Load Time** |
| Program in Memory executing | **Execution Time** (this "Program in Memory" = a **Process**) |

### Full picture (with linking types mapped to binding times)

| Linking Type | Occurs At |
|---|---|
| **Static Linking** | Compile time → produces load module directly |
| **Load-time Dynamic Linking** | Load time — system library linked in when program is loaded |
| **Run-time Dynamic Linking** | Run time — dynamically loaded system library linked in *during* execution |

```
source program
     ↓ compiler/assembler        } compile time
object module  ←─ other object modules (static linking)
     ↓ linkage editor
load module                       } load time
     ↓ loader  ←─ system library (load-time dynamic linking)
in-memory binary memory image     } execution time (run time)
     ↑
dynamically loaded system library (run-time dynamic linking)
```

---

## 6. Design of a Memory Manager — Goals & Functions

**Functions of a Memory Manager:**
1. Allocation
2. Protection
3. Address Translation
4. Free Space Management
5. Deallocation

**Goals:**
- Effective utilization of memory (reduce fragmentation)
- Manage execution of **overlays** — running larger programs in smaller memory areas (leads to the concept of **Virtual Memory**)

---

## 7. Memory Management Techniques (Overview)

```
Techniques
   ├── Contiguous Allocation (CA)  → "Centralized"
   │       ├── Overlays
   │       └── Partitions
   │              ├── Fixed
   │              └── Variable
   │                    └── Buddy System
   │
   └── Non-Contiguous Allocation (NCA) → "Distributed"
           ├── Paging
           ├── Segmentation
           └── Segmentation with Paging
```

- **Contiguous**: entire process occupies **one continuous block** of memory.
- **Non-Contiguous**: process is split into blocks (e.g., pages) that may be scattered across memory (e.g., across memory modules M1, M2, M3...).

---

## 8. Overlays

**Concept:** Split a large program into pieces that are **not needed at the same time**, and swap them in/out of a smaller reserved memory region. Used classically for a **2-Pass Assembler**.

### Worked Example — 2-Pass Assembler Overlay

Given components and their sizes:

| Component | Size |
|---|---|
| Pass 1 | 70 KB |
| Pass 2 | 80 KB |
| Symbol Table | 30 KB |
| Overlay Driver | 10 KB |
| Common Routines | 20 KB |
| **Total (without overlay)** | **210 KB** |

Available Memory = **150 KB** (Pass 1 and Pass 2 never run simultaneously, so they can *share* the same overlay region.)

**Memory layout with overlay:**
```
┌───────────────────┐
│  Symbol Table       │  30 KB
├───────────────────┤
│  Common Routines    │  20 KB
├───────────────────┤
│  Overlay Driver      │  10 KB
├───────────────────┤
│  Pass 1  / Pass 2    │  max(70, 80) = 80 KB  → "shared overlay area"
└───────────────────┘
```
Total memory needed = 30 + 20 + 10 + 80 = **140 KB** ✅ (fits within 150 KB, versus 210 KB without overlays)

> 💡 The overlay region size = **max of the sizes of components that are never needed simultaneously**, not their sum.

---

## 9. Partitions — Fixed vs Variable

```
Partitions
  ├── Fixed (MFT — Multiprogramming with Fixed Number of Tasks)   → Static
  └── Variable (MVT — Multiprogramming with Variable Number of Tasks) → Dynamic
```

### 9.1 Fixed Partitions (MFT)

- Memory is divided into a **fixed number of partitions** at system generation/boot time.
- **1 Partition = 1 Program** (each partition can hold exactly one process).
- Example layout: OS (S-Area) + partitions of size 250 KB, 30 KB, 120 KB, 180 KB for processes.
- A **bitmap**/**bit vector** (0/1 per partition) tracks free/occupied partitions.
- **Best suited for:** multiprogramming systems with **fixed, known tasks**.

### 9.2 Variable Partitions (MVT / Dynamic Partitioning)

- Partitions are created **dynamically**, exactly the size needed by each incoming process (no wasted space per partition at allocation time).
- Example: Memory holds processes with requirement sizes `85KB, 34KB, 102KB, 350KB, 185KB, ...` allocated as `P2(34KB), P3(102KB), P5(185KB)`, etc., leaving irregular holes.

### Performance Comparison: Fixed vs Variable Partitions

| Criterion | Fixed Partitions (MFT) | Variable Partitions (MVT) |
|---|---|---|
| Internal Fragmentation | ✅ Present | ❌ Absent |
| External Fragmentation | ❌ Absent | ✅ Present |
| Degree of Multiprogramming | Limited to number of partitions | Flexible |
| Max Process Size | Limited to max partition size | Flexible |
| Allocation Policy | Best Fit (less relevant since sizes fixed) | Worst Fit (preferred, leaves usable leftover holes) |

---

## 10. Base & Limit Registers (Protection)

Used to **protect** memory — ensure a process only accesses its own allocated space.

### Formula / Condition
$$ \text{base} \le a \le \text{base} + \text{limit} $$

Where:
- $a$ = the memory address being accessed by the CPU
- **base** = starting address of the process's memory partition
- **limit** = size of the partition (so `base + limit` = ending boundary)

**Hardware check:**
```
CPU generates address `a`
   if a >= base AND a < base + limit → allowed, access memory
   else → trap to OS: "illegal addressing error"
```

- Only the **OS (kernel mode)** can set the base and limit registers.

### Numeric Example
| OS | Process | Base | Limit |
|---|---|---|---|
| 0–256000 | process | 300040 | 30040 |
| | process | 420940 | 120900 |
| | process | 880000 | ... |

Check: `Base ≤ M < base + limit` for absolute address `M`.

---

## 11. Relocation Register (Address Translation)

Translates a **logical address** (generated by CPU) into a **physical address** using a **relocation register** (essentially acts like the base register) and a **limit register**.

### Formula
$$ \text{Physical Address} = \text{Logical Address} + \text{Relocation Register (Base)} $$

**Check condition:** Logical address must be `< limit register`, else → **trap: addressing error**.

### Numeric Example
- Limit register = **1001** (max allowed logical address range)
- Relocation register (base) = **2000**
- CPU generates logical address = **150** (offset)
- Since `150 < 1001` → valid
- Physical address = `150 + 2000` = **2150**

```
CPU → logical address (offset)
        ↓
   is offset < limit? ── no → trap: addressing error
        ↓ yes
   physical address = offset + relocation register
        ↓
      memory
```

---

## 12. Allocation Policies ⭐

Given a list of free holes and a process's memory requirement, decide **where** to place it.

| Policy | Rule |
|---|---|
| **First Fit (FF)** | Allocate the **first** free hole that is big enough |
| **Best Fit (BF)** | Allocate the **smallest** free hole that is big enough |
| **Worst Fit (WF)** | Allocate the **largest** free hole available |
| **Next Fit (NF)** | Like First Fit, but search starts from the **location of the last allocation** (not from the beginning) |

### Worked Example
Free hole list (top to bottom): `180 KB (used ✗), 50 KB (used ✗), 30 KB (used ✗), 250 KB (used ✗), 100 KB, 20 KB`

Request: **Process needs 15 KB**

- **First Fit** → scans from top, picks first hole ≥ 15 KB.
- **Best Fit** → scans all holes, picks the smallest one that's ≥ 15 KB.
- **Worst Fit** → picks the **250 KB** hole (largest available).
- **Next Fit** → resumes search from the **Last Allocated (L.A.)** pointer position onward, wrapping around if needed.

---

## 13. Fragmentation

### Internal Fragmentation
- Occurs in **Fixed Partitioning**.
- Wasted space **inside** an allocated partition (partition is bigger than the process needs).

### External Fragmentation
- Occurs in **Variable Partitioning**.
- Total free memory is enough, but it's **scattered** in small non-contiguous holes, so no single hole is big enough for a new process.

### Comparison

| Criterion | Fixed Partition | Variable Partition |
|---|---|---|
| Internal Fragmentation | ✅ | ❌ |
| External Fragmentation | ❌ | ✅ |
| Degree of Multiprogramming | Limited | Flexible |
| Max Process Size | Limited | Flexible |
| Best Allocation Policy | Best Fit | Worst Fit |

### Worked Example — External Fragmentation buildup
Main Memory: `OS | P1(2MB) | P2(6MB) | P3(3MB) | P4(4MB) | P5(6MB)`

| Event | Resulting State |
|---|---|
| P2 completes | Hole of 6MB appears |
| P6 (3MB) arrives | Placed in part of the 6MB hole → 3MB hole remains |
| P7 (2MB) arrives | Placed in remaining hole → 1MB hole remains |
| P4 (4MB) completes | New 4MB hole appears |
| P8 (5MB) arrives | ❌ **Cannot load P8** — no single hole ≥ 5MB, even though total free space may be ≥ 5MB (holes: 1MB + 4MB, non-contiguous) |

> This is a classic illustration of **external fragmentation**: enough total free memory exists, but not enough *contiguous* memory.

---

## 14. Compaction

**Compaction** ("defragmentation") = moving all allocated processes together to one end of memory, merging all free holes into a single large contiguous block.

### Example
- Memory = 120 KB total; holes of 30 KB, 50 KB (between P1 and P2), and 100 KB scattered around P1, P2.
- After compaction → P1 and P2 pushed together, single free hole of **150 KB** created at the end (combined memory becomes usable as one big block).

### Downsides of Compaction
1. **Time-consuming overhead** — copying large amounts of memory takes CPU time.
2. Requires **run-time address binding** — since processes physically move, their addresses must be relocatable at run time (dynamic relocation via base/relocation register).

### The Other Fix for External Fragmentation: Non-Contiguous Allocation
Instead of compaction, external fragmentation can also be solved by switching to **Non-Contiguous Allocation** techniques like **Paging** (covered in the next lecture).

---

## 15. Worked Examples (RAM Chip Sizing — Hardware Numericals)

### Example A
**Q:** How many `32K × 1` RAM chips are needed to provide a memory capacity of `256K Bytes`?

- Total bits needed = `256K × 8` bits (since 1 Byte = 8 bits) = `2048K` bits
- Each chip provides `32K × 1` = `32K` bits
- Number of chips = `2048K / 32K` = **64**

✅ **Answer: 64** (Option C)

### Example B (GATE 2021 style)
**Q:** A RAM chip has a capacity of 1024 words of 8 bits each (`1K × 8`). How many `2×4` decoders **with enable line** are needed to construct a `16K × 16` RAM from `1K × 8` RAM chips?

**Reasoning:**
- To build `16K` words from `1K`-word chips → need `16K / 1K = 16` chip-rows (row expansion for address space).
- To build `16`-bit width from `8`-bit chips → need `16 / 8 = 2` chips per row (column expansion for word width).
- Total chips = `16 × 2 = 32` chips.
- A `2×4` decoder produces 4 outputs from 2 select lines → to select among **16 rows**, we need decoders that together produce 16 enable outputs.
  - One `2×4` decoder (with enable) gives 4 outputs.
  - To get 16 outputs using a two-level decoder tree: 1 top-level decoder (selecting among 4 groups) + 4 second-level decoders (4 outputs each) = **5 decoders total**.

✅ **Answer: 5** (Option B)

---

## 16. GATE PYQs / Review MCQs

### Q1. Which of the following is true about Address Binding?
- A. Logical addresses are generated by the loader
- B. Physical addresses are generated by the user program
- C. Logical addresses are generated by the CPU during execution
- D. **It is an association of Instructions and Data Units to Memory Locations** ✅

**Why others are wrong:**
- A: Logical addresses are generated by the **CPU**, not the loader.
- B: Physical addresses are generated by the **hardware/MMU**, not the user program.
- C: This describes logical addresses correctly but is **not the definition of address binding itself** — D is the actual definition being tested.

---

### Q2. Which of the following correctly describes the difference between Static and Dynamic Linking?
- A. **Static linking occurs before run-time execution, dynamic linking at run time** ✅
- B. Static linking includes libraries in the executable, dynamic linking does not
- C. Dynamic linking increases memory usage compared to static linking
- D. Static linking uses run time libraries

**Why others are wrong:**
- B: True in spirit but **not the core distinguishing definition** tested (timing is the fundamental difference; B is a *consequence*, not the definition).
- C: Opposite is true — **dynamic linking reduces** memory usage via shared libraries.
- D: Backwards — static linking uses **compile-time**, not run-time, libraries.

---

### Q3. In Contiguous Memory allocation, each Process is allocated:
- A. Multiple non-contiguous blocks of memory
- B. **A single contiguous block of memory** ✅
- C. Memory pages from different frames
- D. Segmented memory

**Why others are wrong:**
- A, C, D all describe **non-contiguous** allocation schemes (paging/segmentation), which is the opposite of what's asked.

---

### Q4. How many 32K × 1 RAM chips are needed to provide a memory capacity of 256K Bytes?
- A. 8
- B. 32
- C. **64** ✅
- D. 128

**Why others are wrong:** Computed as `(256K × 8 bits) / (32K × 1 bit)` = 64. Options A, B, D result from miscounting bit-width or byte/bit conversion.

---

### Q5. [GATE 2021] A RAM chip has a capacity of 1024 words of 8 bits each (1K × 8). The number of 2×4 decoders with enable line needed to construct a 16K × 16 RAM from 1K × 8 RAM is:
- A. 4
- B. **5** ✅
- C. 6
- D. 7

**Why others are wrong:** Requires a 2-level decoder tree to select among 16 (1K-word) blocks: 1 master decoder + 4 sub-decoders = 5. Fewer decoders (4) can't address all 16K rows; more (6,7) overcounts the required selection lines.

---

### Q6. Linker is given object modules for a set of programs that were compiled separately. What information needs to be included in an object module? *(MSQ — multiple correct)*
- A. **Object code** ✅
- B. **Relocation bits** ✅
- C. **Names and locations of all external symbols defined in the object module** ✅
- D. Absolute addresses of internal symbols ❌

**Why D is wrong:** Internal symbols don't need **absolute** addresses stored in the object module — actual absolute addresses are only fixed at link/load time (that's the whole point of relocation bits); the object module only needs *relative* addressing info.

---

### Q7. An advantage of Dynamic Linking is that:
- A. **The segments that are not used in a run need not be linked into the process address space** ✅
- B. It reduces execution time overhead
- C. Debugging is simplified because programs are modular
- D. The linker need not construct the known segment table

**Why others are wrong:**
- B: Dynamic linking actually **adds** some run-time overhead (resolving links at execution time), not reduces it.
- C: Modularity isn't really about debugging ease in this context — it's a distractor.
- D: The linker still needs segment table info; this isn't a real benefit of dynamic linking.

---

### Q8. Dynamic linking can cause security concerns because:
- A. Security is dynamic
- B. **The path for searching dynamic linking is not known till runtime** ✅
- C. Linking is insecure
- D. Cryptographic procedures are not available for dynamic linking

**Why others are wrong:** A, C, D are vague/false generalizations with no technical basis — the real security issue is that the **search path for the dynamic library is resolved only at runtime**, which can be exploited (e.g., DLL hijacking).

---

### Q9. Overlay Tree Sizing Problem
Given an overlay tree:
```
                ROOT (2 KB)
             /      |      \
         A(4KB)   B(6KB)   C(8KB)
        /    \        \        \
    D(6KB)  E(8KB)   F(2KB)   G(4KB)
```
**Q: What is the size of the partition (physical memory) required to Load & Run this program?**

- A. 12 KB
- B. **14 KB** ✅
- C. 10 KB
- D. 8 KB

**Rule:** Partition size = **Maximum{ sum of sizes along any single Root-to-Leaf path }** (since only one path's worth of overlay modules needs to be resident at a time).

**Calculation:**
- Path ROOT→A→E = `2 + 4 + 8 = 14 KB` ← **maximum path**
- Path ROOT→A→D = `2 + 4 + 6 = 12 KB`
- Path ROOT→B→F = `2 + 6 + 2 = 10 KB`
- Path ROOT→C→G = `2 + 8 + 4 = 14 KB` (ties with A→E)

**Why others are wrong:** They correspond to shorter/smaller paths (A→D, B→F, or C alone), not the **maximum** root-to-leaf path, which is what determines the minimum resident partition size needed.

> Note: Total program size (sum of *all* nodes) = 2+4+6+8+6+8+2+4 = **40 KB**, but you **don't** need all of it resident simultaneously — only the max single-path size (14 KB), which is the whole point of overlays.

---

## 17. Mnemonics & Quick Tips

- 🧠 **Base ≤ Address < Base + Limit** — think of it as a fence: the base is the "gate" you enter through, and base+limit is the "far wall" you can't cross.
- 🧠 **Overlay size = MAX of mutually-exclusive-in-time components**, not their sum — "you don't carry your winter coat and swimsuit in the same suitcase pocket at once."
- 🧠 **Fixed partition ↔ Internal fragmentation** / **Variable partition ↔ External fragmentation** — remember: "Fixed walls waste space *inside* the room (Internal); Variable rooms leave scraps *between* rooms (External)."
- 🧠 **Best Fit for Fixed partitions, Worst Fit for Variable partitions** — counter-intuitive pairing worth memorizing directly.
- 🧠 **Compaction = "run-time address binding is a MUST"** — because you're literally moving processes around in physical memory, so addresses can't be fixed earlier.
- 🧠 **Overlay Tree partition size = longest root-to-leaf path**, not the total tree size — "you only need to pack for the longest single journey, not for every possible destination."

---

## 18. Quick Revision Checklist ✅

- [ ] Memory hierarchy levels (Registers → Cache → RAM → Disk → Optical → Tape) and their volatility/speed/cost tradeoffs
- [ ] Linking definition; static vs dynamic linking
- [ ] `.obj` vs `.exe`; role of stubs and DLLs; reusability benefit of dynamic linking
- [ ] Address Binding definition (association of instructions/data to memory locations)
- [ ] Binding **types**: static vs dynamic
- [ ] Binding **times**: compile time, load time, run time (⭐ most important classification)
- [ ] Value binding vs Address binding distinction
- [ ] Multistep processing pipeline: source → compiler → object → linker → executable → loader → memory image (process)
- [ ] Mapping of linking types to load/run time (static linking, load-time dynamic linking, run-time dynamic linking)
- [ ] Memory Manager: 5 functions (allocation, protection, address translation, free space mgmt, deallocation) + 2 goals (utilization, managing overlays)
- [ ] Contiguous vs Non-Contiguous memory allocation techniques
- [ ] Overlays concept + worked numerical (2-pass assembler example)
- [ ] Fixed Partitions (MFT) vs Variable Partitions (MVT) — definitions and comparison table
- [ ] Base & Limit register protection mechanism + formula
- [ ] Relocation register address translation + formula + numeric example
- [ ] Allocation policies: First Fit, Best Fit, Worst Fit, Next Fit — definitions + worked example
- [ ] Internal vs External Fragmentation — definitions, causes, comparison table
- [ ] Worked example showing external fragmentation buildup over time (process allocation/completion sequence)
- [ ] Compaction — definition, benefit, and its two downsides (time overhead + requires run-time binding)
- [ ] RAM chip sizing numericals (bits vs bytes conversion; decoder-based chip selection, GATE 2021 style)
- [ ] All GATE PYQs/MCQs above — can you answer each without looking at the notes?

---

*End of Lecture 22 Notes — Memory Management (Part III). Next up: Paging & Non-Contiguous Allocation.*
