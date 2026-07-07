# Process Synchronization — Lecture 13 (GATE CS/IT — Principles of OS)
*Source: GeeksforGeeks GATE — CS & IT Engineering, Principles of Operating Systems, Lecture 13*

## 📑 Topics Covered in This Lecture

| # | Topic | Covered in this deck? |
|---|-------|------------------------|
| 1 | Need for Synchronization & Race Conditions | ✅ |
| 2 | Types of Synchronization (Competitive / Cooperative) | ✅ |
| 3 | Critical Section (CS) — Terminology | ✅ |
| 4 | Requirements / Conditions of the CS Problem | ✅ |
| 5 | Assumptions of the CS Problem | ✅ |
| 6 | Synchronization Mechanisms — Classification | ✅ |
| 7 | Lock Variable Solution (Software) | ✅ |
| 8 | Strict Alternation | ⚠️ Listed as a topic on the title slide but **not actually explained** in this deck — likely covered in the next lecture |
| 9 | Peterson's Solution | ⚠️ Listed as a topic on the title slide but **not actually explained** in this deck — likely covered in the next lecture |
| 10 | GATE PYQs | ⚠️ A placeholder "GATE PYQ's" slide exists, but no actual questions were shown in this PDF |

> 💡 **Note to self:** This lecture ends right after showing that the *Lock Variable* method fails. Strict Alternation, Peterson's/Dekker's solution, and the GATE PYQs will be in Lecture 14 — check that PDF too.

---

## 1. Why Do We Need Synchronization?

When two or more processes access **shared data/memory** without coordination, three problems can occur:

- **Inconsistency (Incorrectness)** — final value of shared data becomes wrong.
- **Data Loss** — updates from one process get overwritten/lost.
- **Deadlocks** — processes wait on each other forever.

```
Process 1 ---Read---> ┌───────────────┐ <---Write--- Process 2
Process 3 ---Read---> │ Shared Memory │
                       └───────────────┘
```

**Synchronization (Process Coordination)** = the mechanism that controls the order/timing of access to shared resources so these problems don't happen.

---

## 2. Types of Synchronization

| Type | Definition | Example |
|------|------------|---------|
| **Competitive Synchronization** | Two or more processes are said to be in *competitive sync* **iff** they compete/contend for a **shared resource** | Pi and Pj both updating a shared counter `C` |
| **Cooperative Synchronization** | Processes **must get affected by each other** (they cooperate/communicate, not just compete) | **Producer–Consumer** problem using a shared bounded buffer |

**Producer–Consumer setup (conceptual):**
- A shared **buffer** of `N` slots holds data.
- `in` → index where Producer inserts next item.
- `out` → index where Consumer removes next item.
- A shared `count` variable tracks number of filled slots.

---

## 3. The Race Condition — Why Simple Shared Variables Fail

### 3.1 Classic Example: Shared Counter `C`

- Pi executes: `C = C + 1;`
- Pj executes: `C = C - 1;`

At the **machine/assembly level**, each high-level statement is **not atomic** — it breaks into multiple instructions:

**Pi's assembly (increment):**
```
I.   Load  Rp, C     // load C into register Rp
II.  Inc   Rp        // increment register
III. Store C, Rp     // store back to C
```

**Pj's assembly (decrement):**
```
Load  Rc, C     // load C into register Rc
Dec   Rc        // decrement register
Store C, Rc     // store back to C
```

If the CPU switches between Pi and Pj **in the middle** of these instruction sequences (e.g., after `ti: Pi: I, II` runs, then a context switch to `tj: Pj: I, II` happens before Pi's `Store` executes), the final value of `C` becomes **wrong** — e.g., starting at `C = 5`, depending on interleaving you can end up with `C = 4` or `C = 6` instead of the correct `C = 5`.

> ☁️ **Result: Inconsistency** — this is the classic **Race Condition**: the final result of shared data depends on the *timing/order* in which processes are scheduled, which is unpredictable.

### 3.2 Classic Example: Bounded-Buffer Producer–Consumer Code

```c
#define N 100
int Buffer[N];
int Count = 0;

void Producer(void) {
    int itemp, in = 0;
    while (1) {
        itemp = Produce();        // NCS
        while (Count == N);       // CS: wait if buffer full
        Buffer[in] = itemp;       // CS
        in = (in + 1) % N;        // NCS
        Count = Count + 1;        // CS  <-- race condition here
    }
}

void Consumer(void) {
    int itemc, out = 0;
    while (1) {
        while (Count == 0);       // CS: wait if buffer empty
        itemc = Buffer[out];      // CS
        out = (out + 1) % N;      // CS
        Count = Count - 1;        // CS  <-- race condition here
        Process(itemc);
    }
}
```

- Both `Count = Count + 1` (Producer) and `Count = Count - 1` (Consumer) compile down to **Load–Modify–Store** sequences on the shared variable `Count`.
- If these interleave badly (marked "Comp" — compete — in the lecture), `Count` ends up **inconsistent**, same root cause as the Pi/Pj example above.

---

## 4. Key Terminology

| Term | Meaning |
|------|---------|
| **Critical Section (CS)** | The part of a process's code where it accesses **shared resources/variables** |
| **NCS (Non-Critical Section)** | The rest of the code — no shared resource access |
| **Race Condition** | Outcome of concurrent execution depends on the **exact timing/interleaving** of instructions |
| **Pre-emption** | A running process is forcibly removed from the CPU (e.g., by a timer interrupt) so another can run |

**The Critical Section Problem:** Design a protocol so that when process `Pi, Pj, Pk, ...` want to access their critical sections, **at most one is inside `<CS>` at a time** — no matter how they're scheduled.

```
   Pi     Pj     Pk
    \      |      /
     \     |     /
      ▼    ▼    ▼
      ┌──────────┐
      │  <CS>    │   ← only ONE process allowed inside at any instant
      └──────────┘
```

---

## 5. Architecture / Model of a Synchronization Mechanism

Every synchronization mechanism (any algorithm/tool) wraps the Critical Section with an **Entry Section** and an **Exit Section**:

```
Pi, Pj, Pk  ──▶  Entry Section  ──▶  <CS>  ──▶  Exit Section  ──▶  (loop back)
```

- **Entry Section** = "ask permission" (acquire lock)
- **Critical Section** = exclusive execution
- **Exit Section** = "release lock"
- **Remainder Section** = rest of the code (safe, non-critical execution)

> 🧠 **Mnemonic:** Every process must "knock" (Entry) before entering the room (CS), and must "close the door properly" (Exit) on the way out.

---

## 6. Requirements of the CS Problem (Solution Must Satisfy)

| # | Requirement | Meaning | Also known as |
|---|-------------|---------|----------------|
| 1 | **Mutual Exclusion (ME)** | If `Pi` is executing in its CS, **no other process** may execute in its CS at the same time | *Primary* requirement |
| 2 | **Progress** | If no process is in CS and some processes want to enter, only processes **not in their remainder section** get to decide who enters next — and this decision **can't be postponed indefinitely** | Liveness condition |
| 3 | **Bounded Waiting** | There is a **limit** on how many times other processes can enter their CS after a process has requested entry, before that process's request is granted | Prevents starvation |
| 4 | **Architectural Neutrality (Arbitrary Speed)** | No assumption may be made about the relative **speeds of processes** or the **number of CPUs** | Portability requirement |

> ⚠️ **Deadlock ⟹ Violation of Progress.** If processes are stuck waiting on each other forever (deadlock), the Progress condition is automatically violated (no process is making headway even though some want to enter CS).

---

## 7. Assumptions / Ground Rules of the CS Problem & Synchronization Mechanisms

1. **No process can enter CS without executing the Entry Section first.**
2. **A process can get pre-empted (by the CPU/scheduler)** while executing the Entry Section, the CS, or the Exit Section.
3. **A process is logically considered to have "left" the CS only after it completes the Exit Section** (not immediately after finishing the CS code).
4. **A process can enter the CS at any time, and any number of times**, as long as no other process is interested in entering.
5. **A process will enter and exit the CS in a finite amount of time** — i.e., "the CS is flawless" (no infinite loops inside CS).

---

## 8. Classification of Synchronization Mechanisms

```
                      Synchronization Mechanisms
                       /                        \
              Busy-Waiting                 Non-Busy Waiting
              (loop-based)                (if-then-else / blocking)
                       \                        /
        ┌───────────────┼────────────────────┐
        ▼                ▼                    ▼
   Software          Hardware              OS-Based
   ───────           ────────              ────────
   - Lock Variable    - TSL (Test-and-Set)   - Semaphores
   - Strict           - SWAP                 - Monitors
     Alternation       - CAS
   - Peterson's /      - Interrupt
     Dekker's /          Disabling
     Bakery Algo
```

| Category | Busy-Waiting? | Examples |
|----------|----------------|----------|
| **Software** | Yes | Lock Variable, Strict Alternation, Peterson's/Dekker's Algorithm, Bakery Algorithm |
| **Hardware** | Yes | TSL (Test-and-Set-Lock), SWAP, CAS (Compare-and-Swap), Interrupt Disabling |
| **OS-Based** | No (blocking) | Semaphores, Monitors |

> 🧠 **Mnemonic:** *"Busy-waiting = keep looping and checking (like repeatedly knocking on a locked door); Non-busy waiting = go to sleep and get woken up (blocking) when the door opens."*

---

## 9. Lock Variable — Software Solution (Busy-Waiting)

**Characteristics:**
- Busy-waiting
- Software solution
- Multi-process solution

**Idea:** A shared `Lock` variable indicates CS status:

| Lock value | Meaning |
|------------|---------|
| `0` | CS is **free** |
| `1` | CS is **in use** |

**Pseudocode:**
```c
// Entry Section
while (Lock != 0);   // busy-wait until lock is free
Lock = 1;            // acquire the lock

// Critical Section
// ... shared resource access ...

// Exit Section
Lock = 0;             // release the lock
```

### 9.1 Why It Fails — Assembly-Level Analysis

At the machine level, `while (Lock != 0); Lock = 1;` expands into multiple instructions:

```
Assembly Code (Entry Section):
1. Load  Ri, M[Lock]   // Loading lock value into register
2. CMP   Ri, #0        // Compare register value with zero
3. JNZ   step 1         // Jump back to step 1 if condition not satisfied
4. Store M[Lock], #1   // Process marks: "I've entered the CS"
5. CS                   // Critical Section
6. Store #0, Lock      // Exit CS, reset Lock to 0
```

Because steps 1–4 are **not atomic**, a process can be pre-empted *right after* checking `Lock == 0` (step 1–3) but *before* setting `Lock = 1` (step 4). Another process can then also pass the check and enter the CS.

**Trace showing the failure:**

| Time | P1 | P2 |
|------|----|----|
| 1 | executes steps 1, 2, 3 → sees `Lock = 0`, about to set it | — |
| 2 | **pre-empted (Pr)** before step 4 | — |
| 3 | — | executes steps 1, 2, 3, 4, 5 (enters CS!) |
| 4 | resumes, executes steps 4, 5 → **also enters CS!** | (still in CS) |

Both **P1 and P2 end up inside the CS at the same time** → Mutual Exclusion is violated.

### 9.2 Verdict — Does Lock Variable Satisfy the CS Requirements?

| Requirement | Satisfied? |
|--------------|------------|
| **Mutual Exclusion** | ❌ Fails (race condition on the check-then-set sequence, as shown above) |
| **Progress** | ✅ Satisfied |
| **Bounded Waiting** | ❌ Fails |

> ⚠️ **Key takeaway:** A simple shared Lock variable is **not sufficient** for correct mutual exclusion because the "check lock, then set lock" sequence is not atomic at the hardware level. This motivates the need for smarter software algorithms (Strict Alternation, Peterson's/Dekker's) or hardware-atomic instructions (TSL, CAS) — covered in the next lecture.

---

## 10. Memory Tricks / Mnemonics Recap

- **Entry → CS → Exit**, like **knocking → using the room → closing the door**.
- **Busy-waiting vs Non-busy waiting** = *repeatedly knocking* vs *sleeping until woken up*.
- **Deadlock implies violation of Progress** — if nobody can move forward, progress is broken by definition.
- **Lock Variable fails ME** because "check-then-set" is not atomic — a process can be interrupted *between* checking and setting the lock.

---

## ✅ Quick Revision Checklist

- [ ] Can explain why synchronization is needed (Inconsistency, Data Loss, Deadlock)
- [ ] Can distinguish **Competitive** vs **Cooperative** synchronization with examples
- [ ] Can explain the **race condition** using the Pi (`C=C+1`) / Pj (`C=C-1`) assembly example
- [ ] Can explain the race condition in the **Producer–Consumer bounded buffer** code (`Count++`/`Count--`)
- [ ] Know the definitions: **Critical Section, NCS, Race Condition, Pre-emption**
- [ ] Can draw the **Entry → CS → Exit → Remainder** model of a synchronization mechanism
- [ ] Can list and explain all **4 requirements**: Mutual Exclusion, Progress, Bounded Waiting, Architectural Neutrality/Arbitrary Speed
- [ ] Know that **Deadlock ⟹ violates Progress**
- [ ] Can list all **5 assumptions** of the CS problem (entry section mandatory, pre-emption possible in Entry/CS/Exit, "left CS" only after Exit, unrestricted entry when uncontested, finite CS execution time)
- [ ] Can classify sync mechanisms into **Busy-Waiting vs Non-Busy-Waiting**, and further into **Software / Hardware / OS-based**
- [ ] Can write the **Lock Variable** pseudocode and its assembly-level breakdown
- [ ] Can explain **why Lock Variable fails Mutual Exclusion and Bounded Waiting**, but satisfies Progress
- [ ] ⚠️ Remember: Strict Alternation, Peterson's Solution, and GATE PYQs were **not** in this lecture — check Lecture 14

---

*End of Lecture 13 notes. Continue with Lecture 14 for Strict Alternation, Peterson's/Dekker's Solution, and GATE PYQs.*
