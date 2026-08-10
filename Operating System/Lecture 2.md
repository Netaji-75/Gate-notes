# Operating Systems — Lecture 02: Introduction & Background
*GATE CS/IT Revision Notes*

---

## 📑 Table of Contents

| # | Topic |
|---|-------|
| 1 | What is an OS? (Definitions & Views) |
| 2 | Interface Between User, Programmer & Hardware |
| 3 | Layered View of a Computer System |
| 4 | Computer System Organization (Bus, CPU, Memory, I/O) |
| 5 | Von Neumann Architecture & Basic CPU Registers |
| 6 | Machine/Instruction Cycle |
| 7 | Instruction Format & Addressing Modes |
| 8 | Classification of Instruction Set |
| 9 | Control Unit — Functions & Types |
| 10 | Von Neumann vs Harvard Architecture |
| 11 | Memory Hierarchy |
| 12 | OS as Resource Manager / Control Program |
| 13 | Goals/Functions of an OS |
| 14 | Operating System Services |
| 15 | Command Line Interpreter (CLI) & GUI |
| 16 | Types of Operating Systems (preview) |
| 17 | Uniprogrammed vs Multiprogrammed Systems (preview) |

---

## 1. What is an OS? (Definitions & Views)

There is **no single universally accepted definition** of an OS — it depends on who you ask.

- **Commercial/Approximate view:** "Everything a vendor ships when you order an operating system" — but this varies wildly across vendors.
- **Strict/Technical view:** "The **one program running at all times** on the computer" → this is the **kernel**.
- **Everything else** on the system is either:
  - A **system program** — ships with the OS but is *not* part of the kernel.
  - An **application program** — any program not associated with the OS at all.
- Modern general-purpose/mobile OSes also ship **middleware** — software frameworks providing extra services to app developers (databases, multimedia, graphics APIs, etc.)

### 🧠 Mental Model — "Core to Crust"
Think of the system like a planet:
- **Core (Kernel):** the central, always-running program.
- **Mantle (System Programs + Middleware):** ships with OS, sits just outside the kernel.
- **Crust (Application Programs):** everything else running on the surface.

### 🧠 Mnemonic — "OS acts like a Government"
- Sets up **control programs**.
- Provides a **set of utilities to simplify application development**.
- **Acts like a government**: doesn't produce useful work itself, but creates an environment in which other programs can do useful work efficiently.

---

## 2. Interface Between User, Programmer & Hardware

The OS is fundamentally an **interface between the user/programmer and the hardware (H/W)**.

- **User program (L₁ level):** language is essentially unrestricted — Σ(a...z), i.e., high-level/human-readable instructions.
- **Kernel/OS (L₂ level):** sits between user and hardware.
- **Hardware:** only understands Σ(0,1) — binary.

**Example — compilation view:**
```
a = b + c;
```
compiles down to machine instructions:
```
Ld  R1, b
Ld  R2, c
Add R1, R2
St  a, R1
```

This ties into the **Stored Program Concept**: instructions (I₁, I₂, I₃ ... I₁₀₀) and data both live in memory; execution proceeds sequentially (**sequential flow**) using a Program Counter (PC) that points to the next instruction to fetch.

- **IP (Instruction Pointer)** → feeds into **CPU (Control Unit + ALU)** → produces **OP (Output)**, all coordinated via **Memory**.
- This is loosely tied to concepts like the **JVM**, which also implements a memory + fetch-execute abstraction on top of real hardware.

---

## 3. Layered View of a Computer System

```
 ┌────────────┐
 │   Users     │
 └────────────┘
        ↕
 ┌───────────────┬─────────┐
 │ Command Interp.│  API   │   ← L1 (Interface Layer)
 └───────────────┴─────────┘
        ↕
 ┌────────────────────┐
 │  OS Kernel / Core   │        ← L2 (Hardware Interface)
 └────────────────────┘
        ↕
 ┌────────────────────┐
 │      Hardware        │
 └────────────────────┘
```

- **Interface = API / SCI (System Call Interface)**
- Users interact through:
  1. **Command Interpreter** → further split into:
     - **GUI (Desktop)** — icon/mouse driven.
     - **Textual/Shell** — `$` prompt driven.
  2. **API** — used directly by programs/processes (P₁) to talk to the kernel.

### Full Abstraction Stack (most → least abstract)
| Layer | Mode |
|---|---|
| Users | User Mode |
| System Programs / Application Programs / User Programs | User Mode |
| Library Routines | User Mode |
| **System Calls** *(boundary line)* | — |
| Operating System | Kernel Mode |
| Computer Hardware | Kernel Mode |

> 🧠 **Remember:** Everything above the System Call line runs in **User Mode**; everything below (OS + Hardware) runs in **Kernel Mode**.

---

## 4. Computer System Organization (Bus, CPU, Memory, I/O)

- One or more **CPUs** and multiple **device controllers** connect through a **common bus**, which provides shared access to memory.
- This enables **concurrent execution** — but CPUs and devices are constantly **competing for memory cycles**.

```
CPU ── Disk Controller ── USB Controller ── Graphics Adapter
              │                  │                 │
           Disks         Mouse/Keyboard/Printer   Monitor
              └──────────────┬──────────────────────┘
                          Memory (shared, via common bus)
```

**Bus lines are typically split into 3 channels:**
- **Control** bus
- **Address** bus
- **Data** bus

> 🧠 **Think About This:** If multiple components (CPU, devices) simultaneously fight for the same memory/compute resources, **without a conductor (the OS), the instruments simply make noise.** This is exactly why an OS/resource-manager is needed.

---

## 5. Von Neumann Architecture & Basic CPU Registers

### Von Neumann Basic Structure
```
        Memory
       ↑↓    ↑↓
 Control Unit ←→ ALU (Accumulator) ←→ Input/Output
        (together = Processor)
```

### Von Neumann Architecture (detailed)
- **Memory** holds both **Programs and Data** (Data Register + Address Register).
- **CPU** contains: Operand A, Operand B, ALU, Registers (Temporary Memory), Result.
- **Control Unit + Instruction Register** manage the **Program Counter (PC)**.
- Data & Address Bus + Control Bus connect Memory ↔ CPU, driven by a **Clock**.
- Input Devices → Memory → Output Devices.

### Basic Computer Registers (Example: 16-bit CPU, 4096-word memory)
| Register | Bits | Purpose |
|---|---|---|
| PC (Program Counter) | 0–11 | Holds address of next instruction |
| AR (Address Register) | 0–11 | Holds memory address |
| IR (Instruction Register) | 0–15 | Holds the current instruction |
| TR (Temporary Register) | 0–15 | Temporary storage during execution |
| DR (Data Register) | 0–15 | Holds data fetched from memory |
| AC (Accumulator) | 0–15 | Holds intermediate arithmetic results |
| INPR / OUTR | 0–7 | Input/Output registers |

> **Definition:** A **processor register (CPU register)** is one of a small set of data holding places that are part of the CPU. A register may hold an instruction, a storage address, or any kind of data (bit sequence or characters). Some instructions specify registers as part of the instruction itself.

### CPU internal datapath (generic)
```
Main Memory ↔ MDR (Memory Data Register) ↔ ALU ↔ CU (Control Unit)
Main Memory ↔ MAR (Memory Address Register)
CPU also has: PC, IR, and general registers R0, R1, R2 ... Rn
```

---

## 6. Machine/Instruction Cycle

### Machine Cycle (4 Steps)
| Step | Action |
|---|---|
| 1 | **Fetch** instruction from memory |
| 2 | **Decode** instructions into commands (Control Unit) |
| 3 | **Execute** commands (ALU) |
| 4 | **Store** results back in memory |

### Instruction Cycle in Microprocessor (cyclic view)
```
Fetch Instruction → Decode Instruction → Execute the Instruction → Read address from memory → (repeat)
```

---

## 7. Instruction Format & Addressing Modes

A machine instruction is typically split into 3 parts:

```
| Addressing Mode | Opcode | Operand (Address/Data) |
      Part-1         Part-2         Part-3
```

**Example instruction format (16-bit):**
```
Bit:   15    14        12 11              0
     ┌────┬─────────────┬──────────────────┐
     │ I  │   Opcode     │      Address      │
     └────┴─────────────┴──────────────────┘
```

- **First 12 bits (0–11):** specify an address.
- **Next 3 bits (12–14):** specify the opcode.
- **Leftmost bit (15) = I:** specifies addressing mode.
  - **I = 0** → **Direct address**
  - **I = 1** → **Indirect address**

---

## 8. Classification of Instruction Set

There are **5 categories** of instructions:

1. **Data Transfer Instructions**
2. **Arithmetic Instructions**
3. **Logical Instructions**
4. **Branching Instructions**
5. **Control Instructions**

---

## 9. Control Unit — Functions & Types

### Block Diagram
```
Instruction Register → Control Unit → Control signals within CPU
Flags → Control Unit
Clock → Control Unit
Control Unit ↔ Control Bus (signals to/from bus)
```

### Functions of the Control Unit
- Coordinates the sequence of data movements into, out of, and between a processor's sub-units.
- Interprets instructions.
- Controls data flow inside the processor.
- Receives external instructions/commands and converts them into a sequence of control signals.
- Controls execution units (ALU, data buffers, registers) within the CPU.
- Handles multiple tasks: fetching, decoding, execution handling, and storing results.

### Types of Control Unit
| Type | Description |
|---|---|
| **Hardwired** | Fixed logic circuits implement control signals |
| **Micro-programmable** | Control signals generated via microcode/microprogram |

---

## 10. Von Neumann vs Harvard Architecture

| Feature | **Von Neumann Model** | **Harvard Model** |
|---|---|---|
| Memory | **Single shared memory** for both instructions & data | **Separate memories** for instructions and data |
| Structure | Input → CPU (Control Unit + ALU + Memory Unit) → Output | Instruction Memory ↔ Control Unit ↔ Data Memory; Control Unit also connects to ALU & I/O |
| Bottleneck | Can suffer from the **"Von Neumann bottleneck"** (single bus for instructions & data) | Avoids this bottleneck — parallel access to instruction & data memory |
| Typical Use | General-purpose computers | DSPs, microcontrollers |

---

## 11. Memory Hierarchy

Arranged from **fastest/smallest/most expensive** (top) to **slowest/largest/cheapest** (bottom):

```
        Registers          ← Smaller, Faster  (Primary Storage)
          Cache
       Main Memory
   ─────────────────────  (Volatile ↔ Non-volatile boundary)
    Non-volatile Memory
      Hard-disk Drives     (Secondary Storage)
        Optical Disk
       Magnetic Tapes      ← Larger, Slower  (Tertiary Storage)
```

### Characteristics of Various Storage Types

| Level | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| **Name** | Registers | Cache | Main memory | Solid-state disk | Magnetic disk |
| **Typical size** | <1 KB | <16 MB | <64 GB | <1 TB | <10 TB |
| **Implementation tech.** | Custom memory, multi-port CMOS | On/off-chip CMOS SRAM | CMOS SRAM | Flash memory | Magnetic disk |
| **Access time (ns)** | 0.25–0.5 | 0.5–25 | 80–250 | 25,000–50,000 | 5,000,000 |
| **Bandwidth (MB/sec)** | 20,000–100,000 | 5,000–10,000 | 1,000–5,000 | 500 | 20–150 |
| **Managed by** | Compiler | Hardware | Operating system | Operating system | Operating system |
| **Backed by** | Cache | Main memory | Disk | Disk | Disk or tape |

> 📝 **Note:** Movement between levels of the storage hierarchy can be either **explicit** or **implicit**.

> 🧠 **Trend to remember:** As you go down the hierarchy → **Storage capacity increases, but access speed decreases.**

---

## 12. OS as Resource Manager / Control Program

The OS acts as a **Resource Manager**, managing two broad categories:

```
              Resource Manager
                 /          \
              H/W            S/W
                          <Files; Semaphores; Monitors; ...>
```

- **User / App view:** `User → Application → Operating System → Hardware` (and back up).
- **Simplified stack:** `User → Application or Programs → Operating System (contains Kernel) → Hardware (CPU, Memory, Devices)`

---

## 13. Goals/Functions of an OS

**6 primary goals** (mnemonic-friendly list):

1. **Convenience**
2. **Efficiency**
3. **Reliability**
4. **Robustness**
5. **Portability**
6. **Scalability**

> 🧠 **Grouping tip:** Convenience & Efficiency are often grouped together (P/S/A,B,C,D style groupings were sketched by the instructor as a way to categorize these goals — treat Convenience+Efficiency as "user-facing goals" and Reliability+Robustness as "system-integrity goals", with Portability+Scalability as "architectural goals").

### What Operating Systems Do (perspective-dependent)
| Perspective | What they care about |
|---|---|
| **Users** (PCs) | Convenience, ease of use, good performance — **don't care about resource utilization** |
| **Shared systems** (mainframe/minicomputer) | Must keep *all* users happy → OS acts as **resource allocator + control program** for efficient HW use |
| **Workstation users** | Dedicated resources, but frequently use shared resources from a **server** |
| **Mobile devices** (smartphones/tablets) | Resource-poor; optimized for **usability + battery life**; use touch/voice interfaces |
| **Embedded systems** (in devices/automobiles) | Little/no user interface; run primarily **without user intervention** |

### Functions of Operating System (5-part wheel)
1. **Processor Management**
2. **Memory Management**
3. **Security**
4. **File Management**
5. **Error Detection**

---

## 14. Operating System Services

OS services split into two broad sets:

### A) Services helpful directly to the **User**
| Service | Description |
|---|---|
| **User Interface (UI)** | CLI, GUI, touch-screen, batch — varies by OS |
| **Program execution** | Load a program into memory, run it, end execution (normally or with error) |
| **I/O operations** | Running programs may need I/O via a file or I/O device |
| **File-system manipulation** | Read/write/create/delete files & directories, search, list info, manage permissions |
| **Communications** | Processes exchange info — same computer (shared memory) or across network (message passing/packets moved by OS) |
| **Error detection** | OS constantly watches for errors in CPU/memory HW, I/O devices, or user programs; takes corrective action; debugging facilities help users/programmers |

### B) Services for **efficient system operation** (resource sharing)
| Service | Description |
|---|---|
| **Resource allocation** | Allocates resources (CPU cycles, main memory, file storage, I/O devices) among multiple concurrent users/jobs |
| **Logging (Accounting)** | Tracks which users use how much and what kind of resources |
| **Protection & Security** | **Protection** = controlled access to system resources; **Security** = defending system from outsiders (user authentication + defending I/O devices from invalid access) |

### Full Service Wheel (as taught)
Memory Management • Device Management • I/O Management • Networking • Communication Management • Secondary Storage Management • Command Interpretation • Error Detection • Job Accounting • Security • File Management

### Layered Service View
```
User & Other System Programs
   → GUI / Touch Screen / Command Line  (User Interfaces)
   → System Calls
      → Program Execution | I/O Operations | File Systems |
         Communication | Resource Allocation | Accounting |
         Error Detection | Protection & Security
   → Operating System
   → Hardware
```

---

## 15. Command Line Interpreter (CLI) & GUI

### Command Line Interpreter (Shell)
- **CLI allows direct command entry.**
- Sometimes implemented **in the kernel**, sometimes as a **systems program**.
- Sometimes **multiple flavors** implemented — called **shells** (e.g., Bourne shell).
- Primarily **fetches a command from the user and executes it**.
- Commands may be:
  - **Built-in** to the shell, OR
  - **Just names of external programs.**
- 🧠 **Key insight:** If commands are just program names (not built-in), **adding new features doesn't require modifying the shell itself** — just add a new program.

*(Example shown: Bourne Shell session — `uptime`, `df -kh`, `ps aux | sort -nrk 3,3 | head -n 5`, `ls -l` — standard Unix command-line usage, no numerical problem to solve here.)*

### GUI (Graphical User Interface)
- **User-friendly desktop metaphor** — usually via mouse, keyboard, monitor.
- Icons represent files, programs, actions, etc.
- Mouse actions over interface objects → info, options, execute function, open folder.
- **Invented at Xerox PARC.**
- Many systems now combine **both CLI + GUI**:
  | OS | GUI | CLI |
  |---|---|---|
  | Microsoft Windows | GUI-first | "Command" shell available |
  | Apple macOS | "Aqua" GUI | UNIX kernel underneath, shells available |
  | Unix/Linux | CLI-first | Optional GUI (CDE, KDE, GNOME) |

### Touchscreen Interfaces
- Needed because mouse is not possible/desired on touch devices.
- **Actions/selection based on gestures.**
- **Virtual keyboard** for text entry.
- **Voice commands** also supported.

---

## 16. Types of Operating Systems (preview — detailed in next lecture)

Mentioned categories (to be covered in depth later):
- Network Operating System
- Distributed Operating System
- Time-Sharing Operating System
- Batch Operating System
- Real-Time Operating System
- Multiprogramming Operating System
- Multiprocessing Operating System
- Mobile Operating System

---

## 17. Uniprogrammed vs Multiprogrammed Systems (preview)

```
             Disk Technology
              /            \
   Uniprogrammed (U.P)   Multiprogrammed (M.P)
```
*(This is introduced as a lead-in to the next topic — full comparison expected in the following lecture on Types of OS.)*

---

## 📌 Formulas in This Lecture

| Formula | Meaning |
|---|---|
| $L_1 = \Sigma(a \dots z)$ | Language at the User Program layer — human-readable symbol set (alphabetic instructions) |
| $L_2 = \Sigma(0, 1)$ | Language at the Hardware layer — binary symbol set |

> These aren't numeric formulas but a way of expressing the **level of abstraction** in terms of the "alphabet" each layer operates in — the OS/Kernel is the translator between $L_1$ and $L_2$.

---

## 💻 Code Snippets

**Example: High-level to machine-level translation**
```c
a = b + c;
```
Compiles to (assembly-like pseudocode):
```asm
Ld  R1, b      ; Load value of b into register R1
Ld  R2, c      ; Load value of c into register R2
Add R1, R2     ; R1 = R1 + R2
St  a, R1      ; Store result from R1 into variable a
```

**Example: Bourne Shell commands (from terminal demo)**
```bash
uptime
df -kh
ps aux | sort -nrk 3,3 | head -n 5
ls -l /usr/lpp/mmfs/bin/mmfsd
```

---

## 🎯 GATE PYQs

*No explicit GATE PYQs or MCQs were included in this particular lecture (it is a foundational/background lecture). Numerical/PYQ-heavy content is expected in later lectures on CPU Scheduling, Process Sync, Memory Management, etc.*

---

## 🧠 Memory Tricks / Mnemonics Recap

- **OS = Government analogy** — doesn't do productive work itself, but creates the environment for others to do productive work; acts as a control program + set of utilities.
- **Core → Mantle → Crust** — Kernel is the innermost, always-running core; System Programs/Middleware form the mantle; Application Programs are the outer crust.
- **"Without a conductor, the instruments simply make noise"** — memory device for *why* an OS/resource manager is essential when multiple components compete for shared resources (CPU, memory, bus).
- **If shell commands = program names → no shell modification needed to add features** — remember this as the reason CLIs are extensible.
- **Memory hierarchy trend:** ↓ down the pyramid = ↑ capacity, ↓ speed.
- **User Mode vs Kernel Mode split** happens exactly at the **System Call** boundary.

---

## ✅ Quick Revision Checklist

- [ ] Can define OS from both the *commercial* and *technical/kernel* viewpoints
- [ ] Know the difference between **kernel**, **system program**, **application program**, and **middleware**
- [ ] Can explain the **Core-to-Crust** model
- [ ] Understand the **Interface** concept (User/Programmer ↔ OS ↔ Hardware) and $L_1$ vs $L_2$ language levels
- [ ] Can trace `a = b + c;` down to Load/Add/Store instructions
- [ ] Understand the **Stored Program Concept** and sequential instruction flow via PC
- [ ] Know the **layered abstraction stack**: Users → Programs → Library Routines → System Calls → OS (Kernel Mode) → Hardware
- [ ] Can draw the **Computer System Organization** diagram: CPU/Memory/I-O devices via common bus (Control, Address, Data lines)
- [ ] Can explain **Von Neumann Architecture** and label CPU registers: PC, AR, IR, TR, DR, AC, INPR, OUTR, MAR, MDR
- [ ] Know the **4-step Machine Cycle**: Fetch → Decode → Execute → Store
- [ ] Understand **Instruction Format**: Addressing mode (I bit) + Opcode + Operand/Address; I=0 direct, I=1 indirect
- [ ] Can list the **5 categories of instructions**: Data Transfer, Arithmetic, Logical, Branching, Control
- [ ] Know the **functions and types of Control Unit** (Hardwired vs Micro-programmable)
- [ ] Can compare **Von Neumann vs Harvard architecture** (shared vs separate instruction/data memory)
- [ ] Can draw the **Memory Hierarchy pyramid** and recall access time/bandwidth trends across levels
- [ ] Understand OS as a **Resource Manager** (H/W resources + S/W resources like files, semaphores, monitors)
- [ ] Can list the **6 goals of an OS**: Convenience, Efficiency, Reliability, Robustness, Portability, Scalability
- [ ] Know how OS goals differ by **user perspective**: PC users, shared/mainframe systems, workstations, mobile devices, embedded systems
- [ ] Can list the **5 core OS functions**: Processor Mgmt, Memory Mgmt, Security, File Mgmt, Error Detection
- [ ] Can classify **OS Services** into "user-helpful" vs "system-efficiency" categories
- [ ] Know all **user-helpful services**: UI, Program execution, I/O ops, File-system manipulation, Communications, Error detection
- [ ] Know all **system-efficiency services**: Resource allocation, Logging/Accounting, Protection & Security
- [ ] Understand the difference between **Protection** (controlling access) and **Security** (defending against internal/external attacks)
- [ ] Know **user ID vs group ID vs privilege escalation**
- [ ] Understand **CLI (shell)** — built-in vs external program commands, and why the latter is more extensible
- [ ] Understand **GUI** — desktop metaphor, invented at Xerox PARC, examples across Windows/macOS/Linux
- [ ] Know **touchscreen interface** basics — gesture-based, virtual keyboard, voice commands
- [ ] Have a preview list of **OS types** to study next: Network, Distributed, Time-Sharing, Batch, Real-Time, Multiprogramming, Multiprocessing, Mobile
- [ ] Know the **Uniprogrammed vs Multiprogrammed** split is the next topic (disk technology driven)

---

*Notes compiled from: CS & IT Engineering — Principles of Operating Systems, Lecture 02 (GeeksforGeeks GATE).*
