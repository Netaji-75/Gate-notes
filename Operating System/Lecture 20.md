# Deadlocks — GATE CS/IT OS Notes (Lecture 20)

> Source: GeeksforGeeks GATE CS&IT — *Principles of Operating Systems*, Lecture 20 (Deadlocks)

---

## 📋 Topics Covered

| # | Topic |
|---|---|
| 1 | Deadlock Concept |
| 2 | Necessary Conditions for Deadlock |
| 3 | Deadlock Handling Strategies (overview) |
| 4 | Deadlock Prevention |
| 5 | Deadlock Avoidance — Resource Allocation Graph (single instance) |
| 6 | Deadlock Avoidance — Banker's Algorithm (multiple instances) |
| 7 | Deadlock Detection — Wait-for Graph (single instance) |
| 8 | Deadlock Detection — Safety-style Algorithm (multiple instances) |
| 9 | Deadlock Recovery (Process Termination / Resource Preemption) |
| 10 | Deadlock Ignorance (Ostrich Algorithm) |
| 11 | Worked Numericals, Review MCQs, GATE PYQs |

---

## 1. Deadlock — Concept

A **deadlock** is a state where a set of processes are each waiting for a resource held by another process **in the same set**, so none of them can ever proceed.

Classic picture:
- Process 1 holds Resource 1, waits for Resource 2.
- Process 2 holds Resource 2, waits for Resource 1.
- Neither can move forward → **deadlock**.

---

## 2. Necessary Conditions for Deadlock

A deadlock can occur **only if all four** of these hold simultaneously (necessary, not individually sufficient):

| Condition | Meaning |
|---|---|
| **Mutual Exclusion** | At least one resource must be held in a non-shareable mode (only one process can use it at a time). |
| **Hold and Wait** | A process holding at least one resource is waiting to acquire additional resources held by other processes. |
| **No Preemption** | Resources cannot be forcibly taken away from a process; only the holder can release them, and only voluntarily. |
| **Circular Wait** | A set of processes {P₀, P₁, …, Pₙ} exists such that P₀ waits for a resource held by P₁, P₁ waits for P₂, …, Pₙ waits for P₀. |

**Mnemonic:** **M-H-N-C** → *"My House Needs Circulation"* (Mutual exclusion, Hold & wait, No preemption, Circular wait).

---

## 3. Deadlock Handling Strategies — Overview

| Strategy | Nature | Idea |
|---|---|---|
| **Prevention** | Proactive | Design the system so a deadlock can *never* occur — kill at least one necessary condition. |
| **Avoidance** | Proactive | Let the deadlock conditions exist, but grant resource requests only if doing so keeps the system in a **safe state**. |
| **Detection & Recovery** | Reactive | Allow deadlocks to happen; periodically run an algorithm to detect them, then recover. |
| **Ignorance (Ostrich Algorithm)** | Inactive | Do nothing — assume deadlocks are rare enough that it's cheaper to just reboot than to handle them (used by many general-purpose OSes, e.g., UNIX). |

**Mnemonic:** Prevention + Avoidance = **Proactive** ; Detection & Recovery = **Reactive** ; Ignorance = **Inactive**.

Other named methods mentioned (not elaborated in this lecture — flagged for further reading):
- **Wound-Wait schemes** (used in timestamp-based deadlock avoidance for distributed systems)
- **Limit resource utilization**

---

## 4. Deadlock Prevention

**Philosophy:** We know the 4 necessary preconditions — so eliminate at least one of them *a priori* (by system design), and deadlock becomes impossible.

| Condition Eliminated | How | Drawback |
|---|---|---|
| **Mutual Exclusion** | Make resources shareable (e.g., read-only files). | Not possible for all resource types (e.g., printers). |
| **Hold and Wait** | Require a process to request **all** resources at once, before execution begins; OR release all held resources before requesting new ones. | ⬇️ Low resource utilization, ⬇️ low concurrency (resources reserved but idle). |
| **No Preemption** | Allow the OS to forcibly take a resource from a waiting process (and roll it back later). | Complex to implement; needs state save/restore. |
| **Circular Wait** | Impose a **linear (total) ordering** on all resource types; a process may only request resources in increasing order of enumeration. | Restrictive; can cause **starvation**. |

### Circular-Wait Prevention — Linear Ordering Rule

> If a process holds resources of type Rⱼ, it may only request resources of type Rₖ where **k > j**. It cannot go back and request Rᵢ where **i ≤ j**.
> Equivalently: any process holding Rᵢ can request Rⱼ, but a process holding Rⱼ cannot request Rᵢ.

**Worked illustration (numbered resources):**

| Resource | ID |
|---|---|
| A | 8 |
| B | 3 |
| C | 10 |
| D | 12 |
| E | 9 |
| F | 15 |
| G | 20 |

A process Pᵢ requests, in order: **A(8) → E(9) → D(12)** — valid, since IDs are strictly increasing (8 → 9 → 12). ✅

If Pᵢ now wants **B(id = 3)**, this is **lower** than the last-requested ID (12) → **not allowed** by the ordering rule. Pᵢ must wait until it releases D, E, A and starts fresh — this avoids circular wait, but can lead to **starvation** if Pᵢ keeps getting deprioritized for the low-numbered resource.

⚠️ **Trade-off to remember:** Prevention kills deadlock but can hurt utilization/concurrency and may cause starvation.

---

## 5. Deadlock Avoidance

**Idea:** Instead of restricting requests outright (like prevention), the OS looks ahead: it grants a request **only if the resulting state is still "safe"** — i.e., there's a guaranteed order in which all processes can still complete.

### Safe vs Unsafe vs Deadlock states

```
┌───────────────────────┐
│        SAFE           │   → guaranteed no deadlock (some order lets all finish)
├───────────────────────┤
│       UNSAFE          │   → deadlock is *possible* but not guaranteed
├───────────────────────┤
│      DEADLOCK          │   → subset of Unsafe; deadlock has occurred
└───────────────────────┘
```

- **Safe State:** There exists a sequence of ALL processes (a "safe sequence") such that each process's remaining resource needs can be satisfied using currently available resources + resources held by processes ahead of it in the sequence.
- **Unsafe State:** No such sequence is guaranteed — the system *might* deadlock (but might not).
- Every **Deadlock state** is Unsafe, but not every Unsafe state leads to Deadlock.

### Deadlock Avoidance — Advantages over Prevention

- No need to preempt & rollback processes (unlike Detection & Recovery).
- **Less restrictive** than Deadlock Prevention.
- OS only grants a request if the grant doesn't move the system into a state that *could* lead to deadlock.

### Avoidance Algorithms — which one to use?

| Resource Type | Algorithm |
|---|---|
| **Single instance per resource type** | Resource-Allocation Graph (RAG) algorithm |
| **Multiple instances per resource type** | ⭐ **Banker's Algorithm** (Dijkstra) |

---

### 5.1 Resource-Allocation Graph (RAG) Scheme — Single Instance

Edge types (they *evolve* over time):

| Edge | Meaning | Notation |
|---|---|---|
| **Claim edge** Pᵢ → Rⱼ | Pᵢ *may* request Rⱼ in the future | dashed line |
| **Request edge** Pᵢ → Rⱼ | Pᵢ has *actually* requested Rⱼ | solid arrow, process→resource |
| **Assignment edge** Rⱼ → Pᵢ | Rⱼ is currently allocated to Pᵢ | solid arrow, resource→process |

**Lifecycle:** `Claim edge → (on request) → Request edge → (on allocation) → Assignment edge → (on release) → back to Claim edge`

> ⚠️ All possible future claims must be declared **a priori** (upfront) for this scheme to work.

**RAG Algorithm:** When Pᵢ requests Rⱼ, the request is granted **only if** converting the request edge to an assignment edge does **not** create a cycle in the resource-allocation graph.

**Worked illustration:**
- At **t₀**: graph is **Safe**.
- At **t₁**: suppose P₂ requests R₂ (claim edge → request edge).
- At **t₂**: granting it converts the graph into an **Unsafe** state (a cycle would form if P₁ later requests R₂ back) — so this request should NOT be granted.

---

### 5.2 Banker's Algorithm — Multiple Instances (⭐ Dijkstra)

**Setup rules:**
- Multiple instances of each resource type exist.
- Each process must declare its **maximum possible claim** for each resource type *a priori*.
- A requesting process may have to **wait** if resources aren't available.
- Once a process gets *all* its resources, it must return them within a **finite** time.

**🏦 Memory Trick (Bank Manager Analogy):**
> Think of the OS as a bank (like SBI). A customer (process) has a sanctioned credit limit (**Max**), has already borrowed some amount (**Allocation**), and may ask for more up to their limit (**Need**). The bank will only approve a new loan (**Request**) if, after giving it, the bank can still guarantee it has enough cash to satisfy the **maximum** possible demand of *every* customer in some order — i.e., the bank stays "solvent" (system stays **Safe**). If granting a loan could bankrupt the bank in a worst case, it's refused — even if the bank has the cash right now.

#### Data Structures (Formulas)

| Structure | Dimension | Meaning |
|---|---|---|
| **n** | scalar | number of processes |
| **m** | scalar | number of resource types |
| **Maximum** | $n \times m$ | $Max[i,j] = k$ → process $P_i$ may need at most $k$ instances of resource $R_j$ |
| **Allocation** | $n \times m$ | $Alloc[i,j] = a$ → $P_i$ currently holds $a$ instances of $R_j$ (note $a \le k$) |
| **Need** | $n \times m$ | $Need[i,j] = b$ → $P_i$ may still need $b$ **more** instances of $R_j$ |
| **Request** | $n \times m$ | $Req[i,j] = c$ → $P_i$ is requesting $c$ instances of $R_j$ **right now** (note $c \le b$) |
| **Total** | $1 \times m$ | $Total[j] = z$ → there are $z$ total instances of $R_j$ in the system |
| **Available** | $1 \times m$ | $Avail[j] = e$ → there are $e$ free instances of $R_j$ right now |

**Key Formulas:**

$$Need[i,j] = Max[i,j] - Allocation[i,j]$$

$$Available[j] = Total[j] - \sum_{i=1}^{n} Allocation[i,j]$$

#### Safety Algorithm (checks if current state is Safe)

```text
1) Let Work and Finish be vectors of length m and n.
   Initialize:
        Work   = Available
        Finish[i] = false, for i = 0, 1, ..., n-1

2) Find an index i such that BOTH:
        (a) Finish[i] == false
        (b) Need_i <= Work        (element-wise)

3) If no such i exists → go to step 4.
   Otherwise:
        Work      = Work + Allocation_i
        Finish[i] = true
        go to step 2

4) If Finish[i] == true for ALL i → system is in a SAFE STATE.
   (The order in which Finish[i] became true is the "safe sequence".)
```

#### Resource-Request Algorithm (checks if a NEW request Request_i can be granted)

```text
Let Request_i be the request vector for process P_i.
If Request_i[j] = k, then P_i wants k instances of resource type R_j.

1) If Request_i <= Need_i        → go to step 2.
   Else                          → ERROR (process exceeded its maximum claim).

2) If Request_i <= Available     → go to step 3.
   Else                          → P_i must WAIT (resources not currently available).

3) Pretend to allocate by modifying the state:
        Available     = Available   - Request_i
        Allocation_i  = Allocation_i + Request_i
        Need_i        = Need_i      - Request_i

   • Run the Safety Algorithm on this tentative state.
   • If SAFE   → grant the request for real.
   • If UNSAFE → P_i must wait, and the OLD state is restored.
```

---

## 6. Worked Examples — Deadlock Avoidance

### Example A — Single Resource Type (n = 5, m = 1)

Given: 5 processes, 1 resource type, **Total R = 25**.

| Pid | Max | Alloc | Need |
|---|---|---|---|
| 1 | 10 | 5 | 5 |
| 2 | 15 | 7 | 8 |
| 3 | 3 | 1 | 2 |
| 4 | 8 | 6 | 2 |
| 5 | 12 | 4 | 8 |

**Step 1 — Available:**
$$Available = Total - \sum Alloc = 25 - (5+7+1+6+4) = 25 - 23 = 2$$

**Step 2 — Apply Safety Algorithm:**

| Step | Chosen Process | Need ≤ Work? | New Work (Work + Alloc) |
|---|---|---|---|
| 1 | P3 | 2 ≤ 2 ✅ | 2 + 1 = 3 |
| 2 | P4 | 2 ≤ 3 ✅ | 3 + 6 = 9 |
| 3 | P5 | 8 ≤ 9 ✅ | 9 + 4 = 13 |
| 4 | P1 | 5 ≤ 13 ✅ | 13 + 5 = 18 |
| 5 | P2 | 8 ≤ 18 ✅ | 18 + 7 = 25 |

✅ **Safe Sequence found:** ⟨P3, P4, P5, P1, P2⟩ → **System is SAFE**.

---

### Example B — Classic Banker's Algorithm (5 processes, 3 resource types)

**Setup:** 5 processes P₀–P₄; resource types A (10 instances), B (5 instances), C (7 instances).

**Snapshot at T₀:**

| Process | Allocation (A B C) | Max (A B C) | Need = Max − Alloc |
|---|---|---|---|
| P0 | 0 1 0 | 7 5 3 | 7 4 3 |
| P1 | 2 0 0 | 3 2 2 | 1 2 2 |
| P2 | 3 0 2 | 9 0 2 | 6 0 0 |
| P3 | 2 1 1 | 2 2 2 | 0 1 1 |
| P4 | 0 0 2 | 4 3 3 | 4 3 1 |

**Available** = Total − ΣAlloc = (10−7, 5−2, 7−5) = **(3, 3, 2)**

**Safety check:**

| Step | Process | Need ≤ Work? | New Work |
|---|---|---|---|
| 1 | P1 | (1,2,2) ≤ (3,3,2) ✅ | (3,3,2)+(2,0,0) = (5,3,2) |
| 2 | P3 | (0,1,1) ≤ (5,3,2) ✅ | (5,3,2)+(2,1,1) = (7,4,3) |
| 3 | P4 | (4,3,1) ≤ (7,4,3) ✅ | (7,4,3)+(0,0,2) = (7,4,5) |
| 4 | P0 | (7,4,3) ≤ (7,4,5) ✅ | (7,4,5)+(0,1,0) = (7,5,5) |
| 5 | P2 | (6,0,0) ≤ (7,5,5) ✅ | (7,5,5)+(3,0,2) = (10,5,7) |

✅ **Safe Sequence:** ⟨P1, P3, P4, P0, P2⟩ → **System is SAFE**.

**Now, requests arrive over time:**

| Time | Request | Check | Result |
|---|---|---|---|
| **T1** | P1 requests (1,0,2) | Req ≤ Need(1,2,2)? ✅. Req ≤ Avail(3,3,2)? ✅. Tentatively: Avail→(2,3,0). Re-run safety → still safe. | ✅ **GRANTED** |
| **T2** | P4 requests (3,3,0) | Req ≤ Need(4,3,1)? ✅. Req ≤ Avail(2,3,0)? A: 3 > 2 ❌ | ❌ **DENIED — P4 must wait** (not enough Available, regardless of Need) |
| **T3** | P0 requests (0,2,0) | Req ≤ Need(7,4,3)? ✅. Req ≤ Avail(2,3,0)? ✅. Tentatively: Avail→(2,1,0). Re-run safety → **no process's Need fits within (2,1,0)** → UNSAFE | ❌ **DENIED — old state restored** |

**Takeaway:** Even if `Request ≤ Available` passes, the request can still be denied if the *resulting* state is unsafe (see T3). Always re-run the Safety Algorithm after a tentative grant.

---

## 7. Deadlock Detection & Recovery

**Idea:** Don't restrict resource requests at all — grant everything, risk deadlock. Periodically run a **detection algorithm** on the current allocation state; if deadlock is found, run a **recovery algorithm**.

| Resource Type | Detection Method |
|---|---|
| **Single instance per resource type** | **Wait-for Graph** + cycle detection |
| **Multiple instances per resource type** | **Detection Algorithm** (Safety-Algorithm-style, but using actual **Request** matrix instead of Need/Max) |

### 7.1 Single-Instance Detection — Wait-for Graph

**Construction:** Take the Resource-Allocation Graph and **collapse/remove the resource nodes** — draw a direct edge Pᵢ → Pⱼ whenever Pᵢ is waiting for a resource currently held by Pⱼ.

**Rule:** A **cycle** in the wait-for graph is both **necessary AND sufficient** for deadlock (only true when each resource type has a single instance!).

**Worked illustration:**
- Original RAG: P₁, P₂, P₃, P₄, P₅ with resources R₁, R₂, R₃, R₄, R₅.
- Collapsed wait-for graph shows: P₁ → P₂ → P₃ → P₄ → P₂ (and) → P₁ ...
- **Cycle found:** P₁ → P₂ → P₃ → P₄ → P₁ → **DEADLOCK confirmed**.
- Method used to find this: run a **cycle-detection algorithm** (e.g., DFS-based) on the wait-for graph.

### 7.2 Multi-Instance Detection — Detection Algorithm

Structurally identical to the Safety Algorithm, **except**:
- Use the actual current **Request** matrix (not Need, since Max claims aren't required for detection).
- `Finish[i]` starts as **true** if `Allocation_i = 0` (a process holding nothing can trivially "finish").

```text
1) Work = Available
   Finish[i] = false, unless Allocation_i == 0 (then true)

2) Find i such that:
       Finish[i] == false  AND  Request_i <= Work

3) If found: Work += Allocation_i ; Finish[i] = true ; repeat step 2
   If none found: go to step 4

4) If Finish[i] == true for all i → NO deadlock.
   Else, every process with Finish[i] == false IS deadlocked.
```

**Important teaching point (from the lecture's worked example):** A system found **Safe** at time T₀ can become **Unsafe/Deadlocked** at time T₁ if a *new* request arrives that cannot be satisfied by the currently free + soon-to-be-released resources — the detection algorithm must be **re-run** whenever the state changes. Always double-check the resulting state after any new request is tentatively granted, not just whether `Request ≤ Available`.

---

## 8. Recovery from Deadlock

Once detected, two broad approaches:

```
Recovery
 ├── Process-based
 │     ├── Kill ALL deadlocked processes  ("Surgeon" approach)
 │     └── Kill ONE at a time             ("Physician" approach)
 └── Resource-based
       ├── Preempt resources from a process
       └── Rollback the preempted process to a safe checkpoint
```

### 8.1 Process Termination

| Approach | Analogy | Pros / Cons |
|---|---|---|
| **Abort ALL deadlocked processes** | 🔪 **Surgeon** — one decisive cut | Simple, guaranteed to break the deadlock. ❌ **Expensive** — computation done by killed processes may be needed by other running processes and is lost. |
| **Abort ONE process at a time** | 🩺 **Physician** — incremental treatment | Less wasteful. ❌ **Overhead** — the detection algorithm must be re-run after each abort to check if deadlock is resolved. |

**Choosing the victim (order to abort):**
- Consider **priority** of the process.
- Prefer aborting a process that is at an **early/initial stage** of execution (less lost work).

Example priority table used in lecture (lower % = "cheaper" to kill in terms of work lost, hence a common victim-selection heuristic):

| Process | P1 | P2 | P3 | P4 |
|---|---|---|---|---|
| % complete | 95% | 80% | 30% | 5% |

*(P4 is the "cheapest" victim — least progress lost.)*

### 8.2 Resource Preemption & Rollback

- Select a resource/process to preempt from, take the resource away, and **rollback** the affected process to some earlier safe checkpoint state.
- ⚠️ **Starvation risk:** if the *same* process is always chosen as the preemption victim (e.g., because it's cheap or low priority), it may **never** make progress → **starvation**. Fix: include the *number of past rollbacks* as a cost factor when selecting victims, so no process is picked indefinitely.

---

## 9. Deadlock Ignorance — the Ostrich Algorithm

- The OS does **nothing** to prevent, avoid, detect, or recover from deadlocks.
- Justification: if deadlocks are rare, it's cheaper to occasionally reboot than to pay the constant overhead of prevention/avoidance/detection.
- Classified as **Inactive** strategy (as opposed to Proactive/Reactive). Used by many general-purpose OSes (e.g., classic UNIX) for most resources.

---

## 10. Formula Cheat-Sheet

| Formula | Use |
|---|---|
| $Need[i,j] = Max[i,j] - Allocation[i,j]$ | Remaining claim of process $i$ on resource $j$ |
| $Available[j] = Total[j] - \sum_i Allocation[i,j]$ | Free instances of resource $j$ |
| Safety check: find $i$ with $Need_i \le Work$ | Core loop of Safety / Detection algorithms |
| **Deadlock-free guarantee (single resource type, $n$ processes, each needs max $m_i$):** $\displaystyle Total \ge \sum_{i=1}^{n}(m_i - 1) + 1 = \sum_i m_i - n + 1$ | Minimum total instances of a resource such that deadlock can **never** occur, even in the worst case (classic "tape drive" style GATE questions) |

---

## 11. Review / Self-Check MCQs

**Q1. Which of the following is NOT a necessary condition for a Deadlock to occur?**
A) Mutual Exclusion  B) Hold and Wait  C) Preemption  D) Circular Wait
✅ **Answer: C.** The actual necessary condition is **"No Preemption"**, not "Preemption" — preemption being *possible* would actually help break deadlocks, not cause them.

**Q2. The Circular Wait condition implies:**
A) A process waits for a resource it already holds
B) A process holds a resource and waits for another held by another process, forming a cycle
C) All processes are in a ready queue
D) All resources are preempted
✅ **Answer: B.** That's the literal definition — a cyclic chain of "holds X, waits for Y".

**Q3. In the Resource Allocation Graph (RAG), a cycle indicates a Deadlock when:**
A) There are multiple resource instances
B) There is only one instance per resource type
C) The graph has more processes than resources
D) All processes are blocked
✅ **Answer: B.** A cycle is necessary **and sufficient** for deadlock only when every resource type has exactly one instance. With multiple instances, a cycle is necessary but **not sufficient**.

**Q4. Which of the following conditions can be eliminated to prevent Deadlock?**
A) Mutual Exclusion  B) Hold and Wait  C) Circular Wait  D) Both B and C
✅ **Answer: D.** In practice, Mutual Exclusion often can't be removed (some resources are inherently non-shareable); Hold-and-Wait and Circular Wait are the ones typically targeted.

**Q5. A system is in a Safe State if:**
A) No deadlock has occurred  B) All resources are allocated  C) There exists a sequence of all processes such that each can finish  D) All processes are waiting
✅ **Answer: C.** This is the exact definition of a safe sequence/safe state.

**Q6. To prevent Hold and Wait, a system can:**
A) Allow only one process in the system  B) Require all resources to be requested at once  C) Preempt all resources  D) Avoid circular wait
✅ **Answer: B.** Requesting everything up-front (or releasing all before requesting more) removes the "hold *some*, wait for more" pattern.

**Q7. Which strategy is used to prevent Circular Wait?**
A) Releasing all resources when deadlock occurs  B) Allocating resources randomly  C) Impose a linear ordering of all resource types  D) Detecting cycles in the RAG
✅ **Answer: C.** Total ordering of resource types + "always request in increasing order" removes the possibility of a cycle.

**Q8. In Deadlock Prevention, Preemption involves:**
A) Killing a process  B) Forcing a process to release resources  C) Allowing a process to hold resources indefinitely  D) Blocking a process forever
✅ **Answer: B.** Preemption (as a *prevention* mechanism against "No Preemption" condition) means the OS can forcibly reclaim a resource.

**Q9. Which of the following is a consequence of requesting all resources at once for Deadlock prevention?**
A) Increased concurrency  B) Better resource utilization  C) Low starvation  D) Low resource utilization
✅ **Answer: D.** Resources sit reserved-but-idle while a process waits to get everything at once → poor utilization and reduced concurrency.

**Q10. A method that ensures the system will never enter an Unsafe State is called:**
A) Deadlock Ignorance  B) Deadlock Detection  C) Deadlock Prevention  D) Deadlock Avoidance
✅ **Answer: D.** Avoidance actively checks safety before granting each request — this is its defining feature (Prevention removes conditions entirely, it doesn't reason about safe/unsafe states).

---

## 12. GATE PYQs (with reasoning)

**PYQ 1. Which of the following is NOT a valid Deadlock Prevention Scheme?**
A) Release all resources before requesting a new resource.
B) Number all resources uniquely and never request a lower-numbered resource than the last one requested.
C) Never request a resource after releasing any resource.
D) Request and be allocated all required resources before execution.

✅ **Answer: B.**
- A, C, D correctly eliminate **Hold-and-Wait** (by forcing "release-before-request" or "all-at-once" patterns).
- B is subtly broken: it only compares against the *last requested* resource, not against **all currently held** resources. A process could still request a resource lower than one it already holds (just not lower than the *most recent* request), which does **not** guarantee an acyclic ordering — circular wait can still form. The correct scheme requires "never request a resource lower-numbered than **any currently held** resource."

**PYQ 2. An OS requires a process to release all resources before requesting another. Which statement is TRUE?**
A) Both starvation and deadlock can occur
B) Starvation can occur but deadlock cannot
C) Starvation cannot occur but deadlock can
D) Neither can occur

✅ **Answer: B.** This policy eliminates **Hold-and-Wait**, so deadlock is impossible. But a process could still be repeatedly denied resources while others keep looping through the release/request cycle → **starvation is still possible**.

**PYQ 3. A computer has six tape drives; n processes compete for them. Each process may need two drives. Max value of n for the system to be guaranteed deadlock-free?**
A) 6  B) 5  C) 4  D) 3

✅ **Answer: B) 5.**
Using $Total \ge \sum(m_i - 1) + 1 = n(2-1)+1 = n+1$. We need $6 \ge n+1 \Rightarrow n \le 5$.
- Options A (6), C (4), D (3): 6 is too many (fails the bound), 4 and 3 are valid but not the *maximum*.

**PYQ 4. An OS has 3 user processes, each requiring 2 units of resource R. Minimum units of R such that no deadlock will ever arise?**
A) 3  B) 5  C) 4  D) 6

✅ **Answer: C) 4.**
$Total \ge n(m-1)+1 = 3(2-1)+1 = 4$.
- A(3): too few — all 3 could each hold 1 and deadlock waiting for the 2nd. B(5), D(6): more than necessary (not the *minimum*).

**PYQ 5. A system has 6 tape drives, n processes compete, each may need 3 tape drives. Max n for guaranteed deadlock-free?**
A) 2  B) 3  C) 4  D) 1

✅ **Answer: A) 2.**
$6 \ge n(3-1)+1 = 2n+1 \Rightarrow n \le 2.5 \Rightarrow n = 2$.
- B, C would violate the bound (n=3 gives $2(3)+1=7 > 6$); D is safe but not maximum.

**PYQ 6. m resources of the same type shared by 3 processes A, B, C with peak demands 3, 4, 6. For what value of m will deadlock NOT occur?**
A) 7  B) 9  C) 10  D) 13  E) 15

✅ **Answer: D) 13** (smallest safe option using the formula).
$m \ge \sum(demand_i - 1) + 1 = (3-1)+(4-1)+(6-1)+1 = 11$. The smallest listed option that satisfies $m \ge 11$ is **13**.
- 7, 9, 10 are all below the safe threshold of 11 → deadlock possible.

**PYQ 7. n processes hold xᵢ instances of resource R (all instances currently occupied); each requests yᵢ more. Exactly two processes p, q have yₚ = y_q = 0. Which is a necessary condition to guarantee the system is NOT approaching deadlock?**
A) $\min(x_p, x_q) < \max_{k \ne p,q} y_k$
B) $x_p + x_q \ge \min_{k \ne p,q} y_k$
C) $\max(x_p, x_q) > 1$
D) $\min(x_p, x_q) > 1$

✅ **Answer: B.**
Since p and q need nothing more, they're guaranteed to finish and release their resources $x_p + x_q$. This combined pool must be able to satisfy at least the **smallest** outstanding request among the rest — enough to let at least one more process proceed and keep the chain going.
- A, C, D use max/min combinations that don't correctly capture "enough freed resources to unblock at least one waiting process."

**PYQ 8. Consider a Resource Allocation Graph with processes P0–P3 and resources r1, r2, r3 (with crossing request/assignment edges). Find if the system is in a deadlock state.**

📝 *Practice exercise* — method to apply:
1. Identify all **assignment edges** (Resource → Process) and **request edges** (Process → Resource).
2. If each resource type has a single instance: collapse into a wait-for graph and look for a cycle → cycle = deadlock.
3. If a resource type has multiple instances: a cycle is necessary but not sufficient — you must additionally verify no process's request can be satisfied by presently available instances (run the Detection Algorithm from §7.2).
*(Redraw the exact graph from the video to trace edges precisely before concluding.)*

**PYQ 9. Banker's Algorithm — 3 processes P0, P1, P2; resources X, Y, Z. Available = (3, 2, 2); system is currently Safe.**

| | Allocation (X Y Z) | Max (X Y Z) | Need (X Y Z) |
|---|---|---|---|
| P0 | 0 0 1 | 8 4 3 | 8 4 2 |
| P1 | 3 2 0 | 6 2 0 | 3 0 0 |
| P2 | 2 1 1 | 3 3 3 | 1 2 2 |

REQ1: P0 requests (0, 0, 2). REQ2: P1 requests (2, 0, 0). Which is TRUE?
A) Only REQ1 can be permitted  B) Only REQ2 can be permitted  C) Both can be permitted  D) Neither can be permitted

✅ **Answer: B) Only REQ2 can be permitted.**

*REQ1 check:* Tentative Avail = (3,0,0); safety trace → P1 can finish (releasing (6,4,0)) but then P2's need (1,2,2) fails on Z (0 available), and P0 also can't finish → **no valid finish order → UNSAFE → REQ1 denied.**

*REQ2 check:* Tentative Avail = (1,2,2); P1 finishes first (need (1,0,0) ≤ (1,2,2)) releasing (5,2,0) → Avail becomes (6,4,2); then P2 finishes (need (1,2,2) ≤ (6,4,2)) releasing (2,1,1) → Avail (8,5,3); then P0 finishes (need (8,4,2) ≤ (8,5,3)) → **all finish → SAFE → REQ2 granted.**

**PYQ 10. n processes hold xᵢ copies of R, request yᵢ more. Exactly 2 processes A, B have request = 0. k free instances exist. Condition to guarantee the system is not approaching deadlock, and total instances of R?**

$$\text{Total instances of } R = \sum_{i=1}^{n} x_i + k$$

**Condition (not approaching deadlock):**
$$x_A + x_B + k \ge \max_{i \ne A,B} y_i$$

Since A and B need nothing more, they're guaranteed to finish; their released resources plus the free pool $k$ must be enough to satisfy at least the **largest** remaining outstanding request, ensuring the chain of completions can continue.

**PYQ 11. A system has 3 programs, each requiring 3 tape units. Minimum tape units so deadlock never arises = ____**

✅ **Answer: 7.**
$Total \ge n(m-1)+1 = 3(3-1)+1 = 7$.

**PYQ 12. A system shares 9 tape drives. Current allocation/max requirement:**

| Process | Current Allocation | Max Requirement | Need |
|---|---|---|---|
| P1 | 3 | 7 | 4 |
| P2 | 1 | 6 | 5 |
| P3 | 3 | 5 | 2 |

Which best describes the current state?
A) Safe, Deadlocked  B) Safe, Not Deadlocked  C) Not Safe, Deadlocked  D) Not Safe, Not Deadlocked

✅ **Answer: B) Safe, Not Deadlocked.**
Total allocated = 3+1+3 = 7 → Available = 9−7 = 2.
- P3 (need 2) ≤ 2 ✅ → finish, Avail = 2+3 = 5.
- P1 (need 4) ≤ 5 ✅ → finish, Avail = 5+3 = 8.
- P2 (need 5) ≤ 8 ✅ → finish.
Safe sequence ⟨P3, P1, P2⟩ exists → Safe. No process is currently *actually* blocked (this is a "Max claims" snapshot, not real pending requests) → Not Deadlocked.

**PYQ 13. Which of the following statements is/are TRUE with respect to deadlocks?**
A) Circular wait is a necessary condition for the formation of deadlock.
B) In a system where each resource has more than one instance, a cycle in its wait-for graph indicates the presence of a deadlock.
C) If the current allocation of resources to processes leads the system to an unsafe state, then deadlock will necessarily occur.
D) In the resource-allocation graph of a system, if every edge is an assignment edge, then the system is not in deadlock state.

✅ **Answer: A and D are TRUE.**
- **A — True:** it's one of the 4 necessary conditions.
- **B — False:** with multiple instances, a cycle is necessary but NOT sufficient for deadlock.
- **C — False:** "Unsafe" only means deadlock is *possible*, not guaranteed — the system could still avoid it depending on future request order.
- **D — True:** if every edge is an assignment edge, no process is currently waiting/requesting anything → no one is blocked → cannot be in deadlock.

**PYQ 14. System with resource types E, F, G; processes P0–P3 with given Allocation & Max matrices; 3 instances of E and 3 of F available.**

| | Allocation (E F G) | Max (E F G) | Need (E F G) |
|---|---|---|---|
| P0 | 1 0 1 | 4 3 1 | 3 3 0 |
| P1 | 1 1 2 | 2 1 4 | 1 0 2 |
| P2 | 1 0 3 | 1 3 3 | 0 3 0 |
| P3 | 2 0 0 | 5 4 1 | 3 4 1 |

📝 *Practice exercise:* Need matrix computed above (assuming G is not a constraining resource here, or treat as given by the full original question). Apply the **Safety Algorithm** from §5.2 with Available = (3, 3, —) to find a safe sequence, or answer whichever specific sub-question (e.g., "max additional G instances P2 could request") was asked in the original slide — re-check the full question text from the video for the exact ask.

**PYQ 15. A multithreaded program P uses x threads and y non-reentrant locks (a thread holding lock *l* cannot re-acquire *l* without releasing it first; blocks if lock unavailable). Minimum x and minimum y together for execution of P to result in a deadlock?**
A) x=1, y=2  B) x=2, y=1  C) x=2, y=2  D) x=1, y=1

✅ **Answer: C) x = 2, y = 2.**
- Deadlock via circular wait needs **at least 2 threads** (one thread alone can't form a cycle with itself under non-reentrant rules in the classic sense) **and at least 2 distinct locks** (classic pattern: T1 holds L1 & wants L2, T2 holds L2 & wants L1). With only 1 lock (y=1), no circular wait is possible regardless of thread count; with 1 thread (x=1), there's no "other" party to wait on.

---

## 13. Mnemonics & Memory Tricks Recap

| Concept | Trick |
|---|---|
| 4 Necessary Conditions | **M-H-N-C**: Mutual exclusion, Hold & wait, No preemption, Circular wait |
| Handling strategy grouping | Prevention + Avoidance = **Proactive**; Detection & Recovery = **Reactive**; Ignorance = **Inactive** |
| Circular Wait Prevention | Numbered resources; only request in **increasing** order — going "backwards" is forbidden (can starve!) |
| Safe / Unsafe / Deadlock | Deadlock ⊂ Unsafe ⊂ All States. Unsafe ≠ Deadlock (just risk) |
| Banker's Algorithm | 🏦 **Bank Manager Analogy** — never sanction a loan that could make the bank unable to meet *someone's* maximum future demand |
| Process Termination | **Surgeon** (abort all — decisive, costly) vs **Physician** (abort one — careful, needs re-diagnosis each time) |
| Victim selection & Starvation | Repeatedly picking the same "cheap" victim → **starvation**; factor in rollback count |
| Ostrich Algorithm | "Bury head in sand" — do nothing, just reboot if it happens |

---

## 14. Quick Revision Checklist ✅

- [ ] Can state all 4 necessary conditions for deadlock (Mutual Exclusion, Hold & Wait, No Preemption, Circular Wait)
- [ ] Can classify the 4 handling strategies as Proactive / Reactive / Inactive
- [ ] Can explain Deadlock Prevention and how each of the 4 conditions is eliminated + trade-offs
- [ ] Understand the linear-ordering trick for preventing circular wait, and why it can starve processes
- [ ] Can define Safe / Unsafe / Deadlock states and their relationship
- [ ] Know when to use RAG (single-instance) vs Banker's Algorithm (multi-instance) for Avoidance
- [ ] Can write out the RAG edge lifecycle: Claim → Request → Assignment → (release) → Claim
- [ ] Know the RAG grant rule: grant only if no cycle is created
- [ ] Can state Banker's data structures: Max, Allocation, Need, Request, Available, Total
- [ ] Know the formulas: $Need = Max - Alloc$; $Available = Total - \sum Alloc$
- [ ] Can execute the Safety Algorithm step by step and find a safe sequence
- [ ] Can execute the Resource-Request Algorithm (3-step check: Need, Available, tentative-grant-and-recheck-safety)
- [ ] Solved: single-resource-type worked example (n=5, m=1)
- [ ] Solved: full Banker's Algorithm worked example (5 processes, 3 resource types) including sequential T1/T2/T3 requests
- [ ] Understand Deadlock Detection differs from Avoidance: uses **Request** matrix, not Need/Max
- [ ] Know Wait-for Graph construction (collapse resource nodes) and that cycle = deadlock **only for single-instance** resources
- [ ] Can apply the multi-instance Detection Algorithm and identify which processes are truly deadlocked
- [ ] Know both Recovery approaches: Process Termination (abort-all vs abort-one) and Resource Preemption + Rollback
- [ ] Understand starvation risk during recovery and its mitigation
- [ ] Know the Ostrich Algorithm and when it's a reasonable choice
- [ ] Memorized the deadlock-free guarantee formula: $Total \ge \sum(m_i - 1) + 1$ for classic "n processes / m resources each" GATE questions
- [ ] Reviewed all MCQs and GATE PYQs above and can justify each correct/incorrect option
