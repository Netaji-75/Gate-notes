# OS Lecture 3: Introduction & Background — Revision Notes

> **Source:** GATE CS&IT Engineering — Principles of Operating Systems, Lecture 03 (GeeksforGeeks GATE)

---

## 📑 Topics Covered (Table of Contents)

| # | Topic |
|---|-------|
| 1 | What is an OS — Uniprogrammed vs Multiprogrammed systems |
| 2 | Types of Operating Systems (Batch, Multiprogrammed, Time-Sharing, Real-Time, Network, Distributed, Mobile) |
| 3 | Preemptive vs Non-Preemptive Scheduling |
| 4 | Popular OS Examples (Windows, macOS, Linux, iOS, Android, Chrome OS, FreeBSD, Ubuntu) |
| 5 | Computing Environments (Traditional, Mobile, Client-Server, Peer-to-Peer, Cloud, Real-Time) |
| 6 | Cloud Computing Models (Public/Private/Hybrid, SaaS/PaaS/IaaS) |
| 7 | Free & Open-Source Operating Systems |
| 8 | Architectural Requirements for Multiprogrammed OS (DMA, MMU, Dual-Mode CPU) |
| 9 | Kernel Mode vs User Mode + Mode Switching (System Calls) |
| 10 | Kernel Data Structures (Linked Lists, BST, Hash Map, Bitmap) |
| 11 | Review Questions / GATE-style MCQs |

---

## 1. What is an Operating System? — The Core Idea

An **Operating System (OS)** is system software that manages hardware resources and provides an environment for users to execute programs conveniently and efficiently.

### 🧠 Foundational Concept: Disk Technology → Programming Models

```
Disk Technology
       │
   ┌───┴────┐
Uniprogrammed OS   Multiprogrammed OS
   (U.Pr)              (M.Pr)
```

### Uniprogrammed OS (Single Job at a time)
- **Memory layout:** OS resides in **System Area (S-Area)**; only **one process (P₁)** occupies the **User Area (U-Area)**.
- Disk holds queued jobs: `P1, P2, P3, P4...`
- When P₁ needs I/O, the CPU sits **idle** waiting for the I/O Device (IOD) to finish.
- **Consequences:**
  - Idleness of CPU
  - Poor CPU utilization
  - Low **throughput**
  - Classic example: **DOS**

### Multiprogrammed OS (Multiple jobs in memory)
- Memory holds **multiple processes** (P₁, P₂, P₃, P₄) simultaneously in the User Area.
- While one process (say P₁) waits on I/O, CPU switches to execute another ready process (P₂).
- **Goal:** Maximize CPU utilization → maximize **throughput**.

> 💡 **Mnemonic:** Uniprogrammed = "one guest in the house, host waits idle when guest steps out." Multiprogrammed = "host immediately entertains another guest while the first one is away."

---

## 2. Preemptive vs Non-Preemptive Scheduling

Multiprogrammed OS scheduling is classified based on **when** the CPU can be taken away from a running process.

| Aspect | Non-Preemptive (N.Pr) | Preemptive (Pr) |
|---|---|---|
| CPU given up | **Voluntarily released** by the process | Can be **forcibly taken away** |
| Responsiveness | **Poor** interactive/responsive behavior | **Improved** interactive/responsiveness |
| Decision triggers | Completion, I/O requirement, System Call | Completion, I/O requirement, System Call, **+ Time, Priority, etc.** |
| Example OS | Windows 3.0 | Unix/Linux, Windows, Mac (modern OS family) |

> 💡 **Key takeaway:** Preemptive scheduling adds **time-slice** and **priority** as additional triggers for context switch — this is *why* modern OSes feel more responsive.

---

## 3. Types of Operating Systems

### 3.1 Batch OS
- Processes a **batch of similar jobs** with **no user interaction**.
- Jobs (Job1, Job2...Jobn) → OS groups into batches → sent to CPU sequentially.
- **Advantage:** Efficient use of CPU/memory/storage — no manual intervention needed.
- **Best for:** Payroll processing, report generation, large-scale repetitive data processing.
- **Limitation:** Not suitable when immediate feedback/real-time processing is needed.
- **Examples:** IBM's OS/360, Unisys' MCP.

### 3.2 Multiprogrammed OS
- Multiple programs reside in main memory **simultaneously**; CPU attention switches between them.
- Achieved via **time-slicing** (e.g., **Round-Robin** scheduling) — CPU time divided into **quantum/time slices**.
- When one program waits on I/O, another utilizes the CPU/memory → improves overall efficiency.
- **Examples:** IBM's OS/360, Unix.

**Memory Diagram concept:**
```
Secondary Storage ⇄ Main Memory [Supervisor | Job A | Job B | Job C(waiting) ] ⇄ CPU
```
- E.g., "Job B in execution" while Job C waits for CPU time — this is the essence of multiprogramming.

### 3.3 Time-Sharing (Multitasking) OS
- Allows **multiple users** to share a single computer **simultaneously**, each via a virtual terminal/remote login.
- CPU switches rapidly between user tasks → illusion of exclusive control for each user.
- A **time-sharing schedule** ensures fair CPU allocation.
- **Security concern:** risk of unauthorized access to another user's terminal → mitigated via user accounts & permissions.
- **Used in:** Universities, research institutions, corporate networks.
- **Examples:** Unix, Linux, Windows Server.

### 3.4 Real-Time OS (RTOS)
- Prioritizes **timely task execution** — often in microseconds/milliseconds.
- Functions on **deterministic behavior** to guarantee critical tasks get priority.
- Uses scheduling like **Rate-Monotonic Scheduling** and **Earliest Deadline First (EDF)**.
- Extra-efficient **interrupt handling** with minimal delay.
- Synchronization via **semaphores, mutexes, message queues**.
- **Characteristics (5 pillars):** Consistency, Reliability, Predictability, Performance, Scalability.
- **Used in:** Industrial automation, robotics, avionics, medical devices, telecommunications.
- **Examples:** VxWorks, QNX.

### 3.5 Network OS (NOS)
- **Formula/Concept:** `NOS = Core OS + Network Protocols (e.g., TCP/IP)`
- Manages/coordinates network resources — enables multiple computers to communicate & share data.
- Provides: file sharing, printer sharing, network security.
- Built-in services: **DNS** (name resolution), **DHCP** (auto IP assignment), **NTP** (time sync).
- Security: encryption, access control, authentication (biometrics, smart cards).
- **Examples:** Novell NetWare, Windows Server (Active Directory).

### 3.6 Distributed OS
- Runs on **multiple interconnected computers** working together as a **single system**.
- Key property: **Transparency** — user sees one cohesive system though resources are physically distributed.
- Individual computers = **nodes/hosts**, each with own processor/memory/storage.
- Provides **inter-process communication (IPC)** mechanisms for data exchange & synchronization across nodes.
- **Biggest advantage:** Operates despite failures — via **redundancy, replication, fault-tolerant protocols**.
- **Used in:** Cloud computing, large-scale data processing, distributed databases, distributed AI.
- **Examples:** Amoeba, DCOM (Distributed Common Object Model).

### 3.7 Mobile OS
- Powers smartphones, tablets, and mobile devices.
- Supports **touch-based interaction**, gestures, native app-store integration.
- **Resource optimization:** manages background processes to save battery while enabling notifications, music, sync.
- **Security priority:** device encryption, app permissions, biometric auth, remote device management.
- **Examples:** Android, iOS, Windows Mobile.

---

## 4. Popular Operating Systems — Quick Reference

| OS | Category | Key Strength |
|---|---|---|
| **Microsoft Windows** | Traditional/Desktop | User-friendly, huge software compatibility |
| **macOS** | Traditional/Desktop | Seamless Apple hardware integration, sleek UI |
| **Linux** | Open/Server | Open-source, flexible, huge distro variety |
| **iOS** | Mobile | Secure, tightly controlled, Apple ecosystem |
| **Android** | Mobile | Open-source, ubiquitous |
| **Chrome OS** | Specialized | Lightweight, web/cloud-focused (Chromebooks) |
| **FreeBSD** | Open/Server | Unix-like, extreme stability, backbone of cloud servers |
| **Ubuntu** | Open/Server (Linux distro) | User-friendly Linux distro, big community |

### 🧠 System Synthesis Matrix (from slides)

| OS Category | Primary Focus | Key Strength | Classic Examples |
|---|---|---|---|
| Traditional | General purpose computing | Software compatibility | Windows, macOS |
| Mobile | Portability & touch | Battery & resource optimization | iOS, Android |
| Distributed | High availability | Fault tolerance & redundancy | Cloud systems |
| Open/Server | Customization & control | Stability & security | Linux, FreeBSD, Ubuntu |

---

## 5. Computing Environments

| Environment | Description |
|---|---|
| **Traditional / Personal Computing** | Stand-alone general-purpose machines; blurred today by Internet interconnectivity. Uses **portals** (web access to internal systems), **thin clients** (network computers acting as web terminals), **firewalls** for home protection. |
| **Mobile Environment** | Handheld smartphones/tablets. Extra OS features: GPS, gyroscope. Enables new app types (e.g., **augmented reality**). Connectivity via **IEEE 802.11 wireless** or cellular data. Leaders: **Apple iOS**, **Google Android**. |
| **Client-Server Computing** | Dumb terminals replaced by smart PCs acting as **clients**; **servers** respond to client requests. Two sub-types: **Compute-server system** (interface for service requests, e.g. DB) and **File-server system** (interface to store/retrieve files). |
| **Peer-to-Peer (P2P) Computing** | Another distributed-system model. **Does NOT distinguish clients & servers** — all nodes are **peers**, each may act as client, server, or both. Node joins network via a **central lookup service** OR **broadcast + discovery protocol**. Examples: **Napster, Gnutella, VoIP (Skype)**. |
| **Cloud Environment** | Delivers computing/storage/apps as a service over network. Logical extension of **virtualization**. E.g., **Amazon EC2** — thousands of servers, millions of VMs, pay-per-usage. |
| **Real-Time Systems** | Strict timing constraints, deterministic performance (see RTOS above). |

> 💡 **Mnemonic — Traditional vs Mobile vs Client-Server:** Think of it as an evolution: *Stand-alone box → Connected box (Traditional with Internet) → Pocket box (Mobile) → Box that talks to a Big Box (Client-Server) → No boss box, everyone's equal (P2P) → Boxes that rent out their guts over the internet (Cloud).*

---

## 6. Cloud Computing — Deployment & Service Models

**Cloud computing environments** = Traditional OSes + VMMs (Virtual Machine Monitors) + Cloud management tools.
- Requires **firewalls** (Internet connectivity security).
- **Load balancers** spread traffic across multiple applications/servers.

### Deployment Models

| Type | Description |
|---|---|
| **Public Cloud** | Available via Internet to anyone willing to pay |
| **Private Cloud** | Run by a company for its own internal use |
| **Hybrid Cloud** | Combination of public + private cloud components |

### Service Models

| Model | Full Form | What it Provides | Example |
|---|---|---|---|
| **SaaS** | Software as a Service | One or more applications available via Internet | Word processor (e.g., Google Docs) |
| **PaaS** | Platform as a Service | Ready software stack for application deployment | Database server platform |
| **IaaS** | Infrastructure as a Service | Servers/storage available over Internet | Storage for backup use |

**Cloud architecture flow (from slide diagram):**
```
Internet → Customer Requests → Cloud Customer Interface
                                        │
                                    Firewall
                                        │
                                  Load Balancer
                                        │
        ┌───────────────┬───────────────┬─────────────────────┐
   Virtual Machines  Virtual Machines  Storage   Cloud Management Services
      (Servers)         (Servers)
```

---

## 7. Free and Open-Source Operating Systems

- OS made available in **source-code format**, not just binary (unlike **closed-source / proprietary** software).
- A counter-movement to **copy protection** and **Digital Rights Management (DRM)**.
- Started by the **Free Software Foundation (FSF)** — uses the **"copyleft" GNU Public License (GPL)**.
- ⚠️ **Note:** "Free software" and "open-source software" are two **different philosophies**, championed by different groups (see FSF's own writing on this distinction).
- **Examples:** GNU/Linux, BSD UNIX (including the core of macOS/Mac OS X).
- Can be explored using **Virtual Machine Monitors (VMM)** like:
  - **VMware Player** (free on Windows)
  - **VirtualBox** (open-source & free, cross-platform) — used to run guest OSes for exploration.

---

## 8. Architectural Requirements for a Typical Multiprogrammed OS

For hardware to support multiprogramming, **3 key architectural features** are needed:

### 8.1 I/O (Secondary Storage) → **DMA Support**
- **DMA = Direct Memory Access**
- Allows I/O devices (D) to transfer data directly to/from **Memory** without CPU intervention on every byte.
- While DMA transfer happens, CPU is free to execute another process (Pⱼ) instead of Pᵢ (which triggered the I/O).

```
        Memory
          ▲
          │ DMA
          │
   CPU → Device (D)
   (Pⱼ)     ▲
   (Pᵢ waiting)
```

### 8.2 Memory → **Address Translation Support**
- Needs an **MMU (Memory Management Unit)** to translate:
  - **Logical Address (LA) / Virtual Address (VA)** — generated by CPU/process
  - → into → **Physical Address (PA)** — actual location in memory

```
CPU (Pᵢ) → generates LA/VA → MMU → translates → PA → Memory
```
- **MMU = Memory Management Unit**, essential for supporting multiple processes safely in separate memory regions.

### 8.3 CPU → **Dual Mode Operation**
See detailed section below (Section 9).

---

## 9. Kernel Mode vs User Mode (Dual-Mode CPU Operation)

The CPU operates in **two modes** to protect the OS and hardware from misbehaving user programs.

```
            Dual Mode Operation
                   │
        ┌──────────┴───────────┐
    User Mode (UM)         Kernel Mode (KM)
   (Non-Privileged)    (Privileged/System/Supervisory Mode)
```

### Comparison Table: User Mode vs Kernel Mode

| Feature | User Mode | Kernel Mode |
|---|---|---|
| Runs | User applications | OS routines |
| Hardware access | **No / Limited** access to hardware | **Complete** access to hardware |
| Preemption | **Preemptive** (Non-Atomic) — can be interrupted mid-execution | **Non-Preemptive** (Atomic) — completes without interruption |
| Mode bit (PSW) | **1 (UM)** | **0 (KM)** |
| Switching mechanism | Uses **System Call Interface (SCI)** / **API** to request kernel services | — |

- The **Mode bit** lives in the **PSW (Program Status Word)**: `0 = Kernel Mode`, `1 = User Mode`.
- Transition between modes happens via the **System Call Interface (SCI) / API**.

### 🧠 Mode Switching Mechanics (from whiteboard walkthrough)

**Example program flow:**
```c
main() {
    int a, b, c;
    a = 1;          // instruction 1
    b = 5;          // instruction 2
    c = a + b;      // instruction 3
    f(c);           // instruction 4 → function call
}

f(k) {
    printf("%d", k);   // library/pre-defined function call
}
```

**Function Types:**
```
        Functions
           │
   ┌───────┴───────┐
User-Defined     Pre-Defined (Library functions, e.g. .lib files, stdlib)
```

**How a Pre-Defined (e.g., `printf`) library call transitions modes:**

1. `f(c)` is called → this is a **user-defined function** → stays in **User Mode**.
2. Inside `f()`, `printf()` is called → this is a **pre-defined/library function**.
3. `printf()` internally issues a **System Call (SVC / Software Interrupt)** — this is what actually crosses into **Kernel Mode**.
4. Flow:
   - `BSA` (Branch/Boundary to System Area) → `SVC` (Supervisor Call) → triggers a **software interrupt** → handled by the **ISR (Interrupt Service Routine)**.
   - The **Dispatch Table** maps the system call number to the **address of the actual system call routine** (e.g., entry `i` in the dispatch table points to address `x` of the syscall code).
   - The syscall routine executes in Kernel Mode, then `RET` (returns) back to User Mode, resuming right after the call.

```
  User Mode                    │  Kernel Mode
  main() → f(c) → printf() ────┼──→ SVC → ISR → Dispatch Table[i] → syscall code @ addr x → RET
                                │        (software interrupt / trap)
```

> 💡 **Mnemonic:** Think of the **Dispatch Table** as a phone directory — the syscall number is like a "contact name," and the table looks up the actual "phone number" (memory address) of the kernel routine to call.

> 💡 **Mnemonic — SCI/API boundary:** User Mode = "the guest area of a hotel" (limited access). Kernel Mode = "staff-only backstage area" (full access to everything). The System Call is like ringing the reception bell to request staff to do something on your behalf — you (the user process) can't just walk backstage yourself.

---

## 10. Kernel Data Structures

The OS kernel internally relies on classic data structures to manage processes, memory, files, etc.

| Data Structure | Description |
|---|---|
| **Singly Linked List** | Each node points to the next; last node → `null`. |
| **Doubly Linked List** | Each node has pointers to both next and previous nodes; first/last node ends in `null`. |
| **Circular Linked List** | Like singly linked list, but the last node points back to the first — forms a loop, no `null` termination. |
| **Binary Search Tree (BST)** | Property: `left ≤ right` (left subtree values ≤ node ≤ right subtree). <br>• Search performance: **O(n)** (worst case, skewed tree) <br>• **Balanced BST**: **O(log n)** |
| **Hash Map** | Uses a **hash_function(key)** to compute an index into an array (the hash map), storing the **value** at that index. Enables near O(1) average lookup. |
| **Bitmap** | A string of **n binary digits** representing the status of **n items** (e.g., free/used blocks in memory or disk). |

**Linux Kernel implementations:** defined in header files —
- `<linux/list.h>` (linked lists)
- `<linux/kfifo.h>` (FIFO queue)
- `<linux/rbtree.h>` (red-black tree, a self-balancing BST)

---

## 11. GATE-Style Review Questions (with Answers)

### Q1. Which of the following is **NOT** a function of an Operating System?
- A) Memory management
- B) **Compiler design** ✅
- C) File system management
- D) Database Management

**Answer: B — Compiler design**
> Reasoning: Memory management, file system management are core OS functions. Compiler design is a separate systems-software domain, not an OS responsibility. (Note: Database Management is *also* not strictly an OS function, but the marked correct answer per the lecture is **Compiler design**.)

---

### Q2. What is the primary purpose of an Operating System?
- A) **To provide an environment in which a user can execute programs conveniently** ✅
- B) To compile user programs
- C) To provide a platform for database applications only
- D) To convert high-level code to machine code

**Answer: A**
> Reasoning: (B) is the job of a compiler, not the OS. (C) is too narrow — OS supports all types of applications. (D) is again compiler/assembler work, not OS.

---

### Q3. Which type of Operating System allows Multiple Users to use the system simultaneously?
- A) Real-time OS
- B) Single-user OS
- C) **Multi-user OS** ✅
- D) Embedded OS

**Answer: C**
> Reasoning: Real-time OS focuses on deadline-critical tasks, not multi-user sharing. Single-user OS supports only one user session at a time. Embedded OS runs on dedicated embedded devices, typically single-purpose.

---

### Q4. What is a Kernel in an Operating System?
- A) A hardware component of the OS
- B) The shell that interacts with the user
- C) **The core part of the OS that manages system resources** ✅
- D) A type of device driver

**Answer: C**
> Reasoning: The kernel is pure software (not hardware — rules out A). The shell is the user-interaction layer, distinct from the kernel (rules out B). A device driver is a component the kernel manages, not the kernel itself (rules out D).

---

### Q5. In which type of Operating System is response time critical?
- A) Time-sharing OS
- B) Batch OS
- C) Distributed OS
- D) **Real-time OS** ✅

**Answer: D**
> Reasoning: Time-sharing OS cares about fairness/turnaround, not strict deadlines. Batch OS has no interactivity requirement at all. Distributed OS focuses on transparency & fault tolerance, not timing guarantees. Real-time OS is *defined* by meeting strict timing constraints.

---

### Q6. Which of the following best describes the role of an Operating System in a Computer System?
- A) It provides networking capabilities to applications.
- B) It translates high-level language to machine code.
- C) **It acts as an interface between hardware and user applications.** ✅
- D) It provides permanent storage for user data.

**Answer: C**
> Reasoning: (A) networking is just one OS service, not its overall role. (B) is a compiler's job. (D) permanent storage is provided by the file system, which is only one part of the OS — not its overall defining role.

---

### Q7. Which of the following is/are **NOT** a valid service provided by an Operating System?
- A) Program Execution
- B) **Word Processing** ✅
- C) Protection and Security
- D) **Code optimization** ✅

**Answer: B and D**
> Reasoning: Program Execution and Protection & Security are classic OS services. Word Processing is an **application-level** service (e.g., MS Word), not an OS service. Code optimization is a **compiler** task, not an OS task.

---

### Q8. Consider a system where the OS supports Multiprogramming. Which of the following is essential for Multiprogramming?
- A) Multi-Core CPU
- B) **Larger Memory to accommodate multiple Programs** ✅
- C) Dual Mode CPU
- D) Virtual memory

**Answer: B**
> Reasoning: Multiprogramming just needs *multiple programs resident in memory at once* — this requires sufficient memory, not necessarily multiple cores (A single core can multiplex via time-sharing). Dual Mode CPU (C) is needed for protection in general OSes, but isn't the *specific* requirement being tested for enabling multiprogramming itself. Virtual memory (D) is a memory management technique, not a strict multiprogramming prerequisite.

> ⚠️ Note: Slide markings show C (Dual Mode CPU) circled in one slide and B in the "final knowledge check" slide — **B (Larger Memory)** is the version marked correct in the later validation slide; treat this as the authoritative answer, but review both options carefully since dual-mode CPU is also commonly cited as a multiprogramming requirement in textbooks.

---

### Q9. Which of the following statements is/are TRUE regarding Kernel Mode and User Mode?
- A) Kernel mode is used for running application-level programs. ❌ **FALSE**
- B) **Kernel mode allows direct access to Hardware Devices.** ✅ **TRUE**
- C) In User Mode, the CPU can execute all instructions. ❌ **FALSE**
- D) **Switching from User Mode to Kernel mode incurs significant overhead.** ✅ **TRUE**

**Answer: B and D**
> Reasoning: (A) is false — application-level programs run in **User Mode**; Kernel Mode is reserved for the core OS to access hardware directly. (C) is false — in User Mode, the CPU can only execute a **restricted (non-privileged)** instruction set, not all instructions. (B) and (D) are true: kernel mode gives full hardware access, and the mode-switch (via trap/interrupt/dispatch table lookup) does carry real overhead.

---

### Q10. In a Time-Sharing Operating System, which Scheduling Algorithm is commonly used?
- A) First-Come First-Serve
- B) Shortest Job First
- C) **Round Robin** ✅
- D) Priority Scheduling

**Answer: C**
> Reasoning: FCFS and SJF don't guarantee fair, timely response for multiple interactive users (can cause long waits). Priority Scheduling can starve low-priority users. **Round Robin**, with its fixed time-slice/quantum, ensures every user gets fair, periodic CPU access — matching the goals of time-sharing systems.

---

## 🧠 Memory Tricks & Mnemonics — Quick Recap

- **Uniprogrammed vs Multiprogrammed:** "One guest vs. multiple guests in the house — host (CPU) stays busy only when there's more than one guest to attend to."
- **Preemptive vs Non-Preemptive:** Preemptive = "the boss can interrupt you anytime (time/priority based)"; Non-preemptive = "you finish your task, THEN hand over control voluntarily."
- **User Mode vs Kernel Mode:** "Hotel guest area (User Mode, limited access) vs. staff-only backstage (Kernel Mode, full access)." System calls = "ringing the reception bell."
- **Dispatch Table:** "A phone directory that maps a syscall number to the actual address (phone number) of the kernel routine."
- **NOS formula:** `NOS = Core OS + Network Protocols (TCP/IP)`
- **DMA:** Lets I/O talk to memory directly — CPU doesn't babysit every byte transfer.
- **MMU:** The "translator" between what a process *thinks* its address is (Logical/Virtual) and where it *actually* lives (Physical).

---

## ✅ Quick Revision Checklist

Use this to self-test before moving to the next lecture:

- [ ] Can explain the difference between **Uniprogrammed** and **Multiprogrammed** OS with a memory diagram.
- [ ] Can list and briefly describe all **7 types of OS**: Batch, Multiprogrammed, Time-Sharing, Real-Time, Network, Distributed, Mobile.
- [ ] Can differentiate **Preemptive vs Non-Preemptive** scheduling and name their respective decision triggers.
- [ ] Can name **RTOS's 5 key characteristics**: Consistency, Reliability, Predictability, Performance, Scalability.
- [ ] Can state the **NOS formula**: Core OS + Network Protocols.
- [ ] Can explain **Distributed OS's** key property: **Transparency**, and how it achieves fault tolerance.
- [ ] Can list popular OS examples across categories (Windows, macOS, Linux, iOS, Android, Chrome OS, FreeBSD, Ubuntu).
- [ ] Can describe all **6 Computing Environments**: Traditional, Mobile, Client-Server, P2P, Cloud, Real-Time.
- [ ] Can differentiate **Compute-server vs File-server** systems.
- [ ] Can explain **P2P** — no client/server distinction, discovery via central lookup or broadcast.
- [ ] Can list **Cloud deployment models** (Public/Private/Hybrid) and **service models** (SaaS/PaaS/IaaS) with examples.
- [ ] Can explain **Free vs Open-Source** software as two different philosophies (FSF/GPL vs OSI).
- [ ] Can state the **3 architectural requirements** for Multiprogrammed OS: DMA (I/O), MMU/Address Translation (Memory), Dual-Mode CPU.
- [ ] Can explain **DMA** and why it frees up the CPU.
- [ ] Can explain **MMU**'s role: Logical/Virtual Address → Physical Address translation.
- [ ] Can differentiate **User Mode vs Kernel Mode** (access level, preemptibility, mode bit value in PSW).
- [ ] Can trace the **mode-switching flow**: user function call → library function (e.g., printf) → System Call (SVC/software interrupt) → ISR → Dispatch Table lookup → syscall execution → RET back to user mode.
- [ ] Can list basic **Kernel Data Structures**: Singly/Doubly/Circular Linked Lists, BST (O(n) vs O(log n) balanced), Hash Map, Bitmap.
- [ ] Can recall relevant **Linux kernel header files**: `<linux/list.h>`, `<linux/kfifo.h>`, `<linux/rbtree.h>`.
- [ ] Attempted all **10 review MCQs** and understood why each wrong option is wrong.

---

*Notes compiled from GeeksforGeeks GATE CS&IT — Principles of Operating Systems, Lecture 03.*
