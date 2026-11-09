# Greedy Method (+ Disjoint Sets) — GATE Revision Notes
**Subject:** Design & Analysis of Algorithms (DAA) · **Source:** GeeksforGeeks GATE, Lecture 10

> ⚠️ This lecture is **DAA (Greedy Method)**, not OS. The one OS link: Job Sequencing treats the *machine as the CPU*, all jobs arrive at t = 0, and every burst time is 1 unit.

---

## 📑 What this lecture covers

1. [Sets & Disjoint-Set Operations (Union & Find)](#1-sets--disjoint-set-operations-union--find)
2. [Greedy Strategy — Core Idea & Terminology](#2-greedy-strategy--core-idea--terminology)
3. [Control Abstraction (General Method)](#3-control-abstraction-general-method)
4. [Fractional Knapsack Problem](#4-fractional-knapsack-problem)
5. [Job Sequencing with Deadlines (JSD)](#5-job-sequencing-with-deadlines-jsd)
6. [Optimal Merge Patterns](#6-optimal-merge-patterns)
7. [Huffman Coding](#7-huffman-coding)
8. [GATE PYQs](#8-gate-pyqs)
9. [Quick Revision Checklist](#9-quick-revision-checklist)
10. [Corrections to Lecture Slides](#10-corrections-to-lecture-slides)

---

## 1. Sets & Disjoint-Set Operations (Union & Find)

### Setup
- Elements of the sets are the numbers $1, 2, \dots, n$ (in practice, indices into a symbol table).
- Sets are **pairwise disjoint**: for $S_i, S_j$ with $i \ne j$, no element is in both.
- Example ($n = 10$):
  - $S_1 = \{1,7,8,9\}$, $S_2 = \{2,5,10\}$, $S_3 = \{3,4,6\}$
- **Representation:** each set is a **tree**; edges point child → parent; the **root = representative of the set**.
  - Roots here: $S_1 \to 1$, $S_2 \to 5$, $S_3 \to 3$.
  - A "Set name → Pointer" table points each set name to its root (this is the *logical* representation).

### The two operations

| Operation | Meaning | Example |
|---|---|---|
| **Disjoint Set Union** $(S_i \cup S_j)$ | Merge two disjoint sets; $S_i, S_j$ stop existing independently and are replaced by the union | $S_1 \cup S_2 = \{1,7,8,9,2,5,10\}$ |
| **Find(i)** | Return the **representative (root)** of the set containing element $i$ | 4 is in $S_3$; 9 is in $S_1$ |

- **Union of two trees can be done either way:** make root 5 a child of root 1, **or** make root 1 a child of root 5 — both are valid representations of $S_1 \cup S_2$.

### Array (parent-pointer) representation
$P[i]$ = parent of $i$; **root has $P = -1$**.

| i | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
|---|---|---|---|---|---|---|---|---|---|---|
| **P** | −1 | 5 | −1 | 3 | −1 | 3 | 1 | 1 | 1 | 5 |

### Algorithms
```text
Algorithm Union(i, j)          // i, j are roots
{
    p[i] := j;                 // make j the parent of i
}

Algorithm Find(i)
{
    while (p[i] >= 0) do i := p[i];
    return i;
}
```

| | Time |
|---|---|
| **Union** | $O(1)$ |
| **Find** (simple) | $O(h)$, where $h$ = **height of the tree** (worst case a chain → $O(n)$) |
| **Find with Collapsing Rule** | Lecture notes it as $O(1)$ *(strictly: amortised / nearly constant after repeated finds)* |

### Collapsing Rule (path compression) — worked example
Tree on the board (root 5):

```
5 ── 8 ── 1, 2 ── 7
  ├─ 9 ── 4 ── 6
  └─ 3 ── 10
```

| i | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
|---|---|---|---|---|---|---|---|---|---|----|
| **P** | 8 | 8 | 5 | 9 | −1 | 4 | 2 | 5 | 5 | 3 |

- `Find(6)` walks **6 → 4 → 9 → 5** (root).
- **Collapsing rule:** after finding the root, make every node on the path (6, 4, 9) point **directly to the root 5** → future `Find(6)` is one hop.

> 🧠 **Remember:** Union = just re-point one root (O(1)). Find = climb to the root (O(height)). Collapsing flattens the tree so later finds are fast.
> *(Context: this structure is what powers Kruskal's MST later.)*

---

## 2. Greedy Strategy — Core Idea & Terminology

### Definition
**Greedy Method** = an algorithm design strategy for problems whose solution can be viewed as the result of a **sequence of decisions**.
- Decisions are made **step by step**.
- At every step, out of the available options, it **greedily selects the option that satisfies the constraints and the objective** of the problem.
- **Core principle:** take the *locally optimal* choice (best RIGHT NOW) **without reconsidering past decisions**.

### How it works (4 ideas)

| # | Idea | Meaning |
|---|---|---|
| 1 | **Stages** | Works in stages; one input considered per stage; decision = include / exclude |
| 2 | **Selection procedure** | Inputs are considered in an order fixed by a selection rule (e.g. sort by ratio) |
| 3 | **Feasibility** | If adding the input keeps the partial solution feasible → add it; otherwise skip it — **never revisited** |
| 4 | **Optimality** | Goal = a feasible solution that max/minimises the objective function |

- Most problems have **$n$ inputs** and need a **subset** that satisfies some constraints.
- **Feasible solution** = any subset that satisfies the constraints.
- **Optimal solution** = feasible solution that **minimises or maximises** the objective function.

### Types of problems

| Decision problem | Optimization problem |
|---|---|
| Output is **T/F, Yes/No, 1/0** | Output is the **Min/Max value** of some criterion |

### Terminology (board)

| # | Term | Meaning |
|---|---|---|
| 1 | Problem definition | What is to be solved |
| 2 | **Constraints** | **Explicit** and **Implicit** (see below) |
| 3 | **Solution space** | All possible ways of organising the inputs, based on **explicit** constraints |
| 4 | **Feasible solution** | A solution that satisfies the **implicit** constraints |
| 5 | **Objective function** | The Min/Max criterion |
| 6 | **Optimal solution** | Feasible solution that optimises the objective function |

| Explicit constraints | Implicit constraints |
|---|---|
| **Boundary** conditions on each variable | **Local conditions** = **feasibility criteria** |
| Define the solution space | Pick feasible solutions out of the space |

**Example — n-Queens (n = 4):** solution = tuple $\langle x_1,\dots,x_n\rangle$, where $x[i]$ = column position of queen $Q_i$.
- Explicit: $1 \le x[i] \le n$
- Implicit: no two queens attack each other (same column / diagonal)
- Solution space (permutation view) = $4! = 24$; one valid solution: $\langle 2,4,1,3\rangle$.

### Applications of greedy (slide)
Scheduling · Shortest Path · Huffman Coding · Minimum Spanning Tree.
*(Board illustration: from a source S, at each node many outgoing options exist; greedy commits to the locally best edge toward D.)*

### Two paradigms of greedy problems

| Subset Paradigm | Ordering Paradigm |
|---|---|
| Greedy **selects a subset** of the choices that satisfies all constraints | Greedy produces an **arrangement / order** of elements to get the best result |
| **Knapsack**, **Job Sequencing with Deadlines** | **Minimum Spanning Tree** (Prim/Kruskal), **Optimal Merge Pattern** |

---

## 3. Control Abstraction (General Method)

```text
Procedure GREEDY(A, n)
    // A[1:n] contains the n inputs
    solution := {}                      // initialise solution to empty
    for i := 1 to n do
        x := SELECT(A)                  // pick best remaining input (and remove it)
        if FEASIBLE(solution, x) then   // can x be included?
            solution := UNION(solution, x)
        endif
    repeat
    return (solution)
end GREEDY
```

| Function | Role |
|---|---|
| **SELECT** | Selects an input from `A[]` and removes it; value assigned to `x` |
| **FEASIBLE** | Boolean function — can `x` be included in the solution vector? |
| **UNION** | Combines `x` with the solution **and updates the objective function** |

- **General complexity:** dominated by the SELECT step (usually a **sort**) → $O(n \log n)$.

---

## 4. Fractional Knapsack Problem

### Problem
- Bag (knapsack) of capacity $M$; $n$ objects, each with **weight** $w_i$ and **profit** $p_i$.
- Decision variable $x_i$ = fraction of object $i$ taken (contributes $w_i x_i$ weight, $p_i x_i$ profit).
- If $\sum w_i > M$, not everything fits → must decide each $x_i$.

$$\text{Maximise } \sum_{i=1}^{n} p_i x_i \quad \text{(objective function)}$$
$$\text{subject to } \sum_{i=1}^{n} w_i x_i \le M \quad \text{(implicit constraint)}$$
$$0 \le x_i \le 1 \quad \text{(explicit constraint)}$$

| Symbol | Meaning |
|---|---|
| $M$ | Knapsack capacity |
| $n$ | Number of objects |
| $w_i, p_i$ | Weight and profit of object $i$ |
| $x_i$ | Fraction of object $i$ chosen |

- Solution space = **infinite** (fractional / real values).

### Three greedy strategies compared
Instance: $n=3,\ M=20,\ (w_1,w_2,w_3)=(18,15,10),\ (p_1,p_2,p_3)=(25,24,15)$

| Strategy | Order | $x$ vector | Σ w·x | Σ p·x |
|---|---|---|---|---|
| **I. Greedy on max profit** | 1, 2, 3 | $(1,\ 2/15,\ 0)$ | 20 | $25 + \frac{2}{15}\cdot 24 = $ **28.2** |
| **II. Greedy on min weight** | 3, 2, 1 | $(0,\ 2/3,\ 1)$ | 20 | $15 + \frac{2}{3}\cdot 24 = $ **31** |
| **III. Greedy on max $p/w$** ✅ | 2, 3, 1 | $(0,\ 1,\ 1/2)$ | 20 | $24 + \frac12 \cdot 15 = $ **31.5** |

Ratios for III: $p_1/w_1 = 25/18 = 1.38$, $p_2/w_2 = 24/15 = 1.6$, $p_3/w_3 = 15/10 = 1.5$.

> 🧠 **Remember:** profit alone or weight alone fails. **Profit per unit weight ($p_i/w_i$)** is the optimal greedy criterion for the *fractional* knapsack. (Board: "$w_i \to p_i$, so $1 \to p_i/w_i$".)

### Algorithm
$$\text{Order objects so that } \frac{p_i}{w_i} \ge \frac{p_{i+1}}{w_{i+1}}$$

```text
Procedure GREEDY_KNAPSACK(P, W, M, X, n)
// P(1:n), W(1:n) = profits, weights, ordered so that P(i)/W(i) >= P(i+1)/W(i+1)
// M = knapsack size, X(1:n) = solution vector
real P(1:n), W(1:n), X(1:n), M, cu;
integer i, n;
X := 0                          // initialise solution to zero
cu := M                         // cu = remaining knapsack capacity
for i := 1 to n do
    if W(i) > cu then exit endif
    X(i) := 1
    cu := cu - W(i)
repeat
if i <= n then X(i) := cu / W(i) endif    // fraction of the last object
end GREEDY_KNAPSACK
```
- **Time:** $O(n)$ for the loop (plus $O(n\log n)$ if sorting by ratio is included).
- Fractional knapsack **always yields the globally optimal solution**.

### Worked example (also GATE-style, see PYQ 1)
$n=7,\ M=15$; $p=(10,5,15,7,6,18,3)$; $w=(2,3,5,7,1,4,1)$

**Step 1 — ratios $p/w$:** $(5.0,\ 1.67,\ 3.0,\ 1.0,\ 6.0,\ 4.5,\ 3.0)$

**Step 2 — sort descending:** Item5 (6.0), Item1 (5.0), Item6 (4.5), Item3 (3.0), Item7 (3.0), Item2 (1.67), Item4 (1.0)

**Step 3 — fill (M = 15):**

| Item | w | Remaining cap before | Take | Remaining after |
|---|---|---|---|---|
| 5 | 1 | 15 | full ($x_5=1$) | 14 |
| 1 | 2 | 14 | full ($x_1=1$) | 12 |
| 6 | 4 | 12 | full ($x_6=1$) | 8 |
| 3 | 5 | 8 | full ($x_3=1$) | 3 |
| 7 | 1 | 3 | full ($x_7=1$) | 2 |
| 2 | 3 | 2 | **partial** $x_2 = 2/3$ | 0 |
| 4 | 7 | 0 | skip ($x_4=0$) | 0 |

**Solution:** $\langle x_1,\dots,x_7\rangle = \langle 1,\ \tfrac23,\ 1,\ 0,\ 1,\ 1,\ 1\rangle$

**Profit** $= 6+10+18+15+3+5\cdot\tfrac23 = 52 + 3.33 = \mathbf{55.33}$

---

## 5. Job Sequencing with Deadlines (JSD)

### Problem rules
- $n$ jobs on **one machine (CPU)**; each job takes **unit time** (burst time = 1); arrival time = 0 for all.
- Job $i$ has **deadline** $d_i \ge 0$ and **profit** $p_i \ge 0$.
- Profit $p_i$ is earned **iff job $i$ completes by its deadline**.
- Only **one job at a time** on the machine.
- **Feasible solution:** a subset $J$ of jobs such that every job in $J$ finishes by its deadline.
- **Optimal solution:** feasible subset with **maximum total profit**.
- Goal: *select a subset of the $n$ jobs that can all finish within their deadlines and generates maximum profit.*

> Slot $t$ = time interval $[t-1, t]$. A job with deadline $d$ may be placed in any slot $\le d$.

### Greedy approach
1. **Sort jobs by profit (descending).**
2. For each job in that order: if it can be scheduled by its deadline (i.e. $J \cup \{i\}$ stays feasible) → **add it**.
3. Return $J$.

*Practical technique used in all examples:* put the job in the **latest free slot $\le d_i$**; if none is free → reject.

```text
Algorithm GreedyJob(d, J, n)
// J is a set of jobs that can be completed by their deadlines
{
    J := {1};
    for i := 2 to n do                              // n-1 iterations
    {
        if (all jobs in J ∪ {i} can be completed by their deadlines)   // O(n) check
            then J := J ∪ {i};
    }
}
```
- **Time:** $(n-1)$ iterations × $O(n)$ feasibility check = $\mathbf{O(n^2)}$.
- Assumes jobs are pre-sorted so $p_1 \ge p_2 \ge \dots \ge p_n$.

### Worked example A — board (n = 4)
$(d_1..d_4) = (2,1,2,1)$; $(p_1..p_4) = (105, 23, 12, 46)$
- $|J|=0$ → profit 0. $|J|=3$ impossible (max deadline = 2, so only 2 slots).
- $|J|=2$ options:
  - $\{J_1,J_2\}$ ✓ (order $J_2, J_1$)
  - $\{J_1,J_3\}$ ✓
  - $\{J_2,J_4\}$ ✗ (both need slot 1)
- Best: $\{J_1, J_4\}$ → schedule **$J_4$ in slot 1, $J_1$ in slot 2** → profit $105+46 = 151$.

### Worked example B — board (n = 7)
$(d_1..d_7) = (4,3,2,3,4,2,2)$; $(p_1..p_7) = (120,64,90,32,45,12,55)$

Sorted by profit: J1(120,d4), J3(90,d2), J2(64,d3), J7(55,d2), J5(45,d4), J4(32,d3), J6(12,d2)

| Job | Profit | Deadline | Slot given | Result |
|---|---|---|---|---|
| J1 | 120 | 4 | 4 | ✅ |
| J3 | 90 | 2 | 2 | ✅ |
| J2 | 64 | 3 | 3 | ✅ |
| J7 | 55 | 2 | 1 (slot 2 taken) | ✅ |
| J5 | 45 | 4 | none (1–4 full) | ❌ |
| J4 | 32 | 3 | none | ❌ |
| J6 | 12 | 2 | none | ❌ |

**CPU schedule:** `| J7 | J3 | J2 | J1 |` at 0–1–2–3–4 → **Profit = 55+90+64+120 = 329**

### Worked example C — slide (n = 7)

| Job | J1 | J2 | J3 | J4 | J5 | J6 | J7 |
|---|---|---|---|---|---|---|---|
| Profit | 35 | 30 | 25 | 20 | 15 | 12 | 5 |
| Deadline | 3 | 4 | 4 | 2 | 3 | 1 | 2 |

| Job | Profit | Deadline | Selected? | Slot |
|---|---|---|---|---|
| J1 | 35 | 3 | ✅ | 3 |
| J2 | 30 | 4 | ✅ | 4 |
| J3 | 25 | 4 | ✅ | 2 |
| J4 | 20 | 2 | ✅ | 1 |
| J5 | 15 | 3 | ❌ | — |
| J6 | 12 | 1 | ❌ | — |
| J7 | 5 | 2 | ❌ | — |

**Max profit = 35+30+25+20 = 110** · **Sequence: J4 → J3 → J1 → J2** (slots 1,2,3,4)

---

## 6. Optimal Merge Patterns

### Background — 2-way merging
- Merging sorted files of sizes $n$ and $m$ into one sorted file costs **$(n+m)$ record movements**.
  - e.g. $F_1(3,5,8,10)$ and $F_2(2,4,7,9)$ → $F(2,3,4,\dots)$ : $n+m$ moves.
- Merging **many** files pairwise: the **order of merges** changes the total cost → find the cheapest order.

### Key insight (greedy)
**Always merge the two SMALLEST files first.** Cost of a merge = sum of the two sizes. Represent the merge order as a **binary tree**.

$$\text{Total record movements} = \sum_{i=1}^{n} d_i \cdot q_i$$

| Symbol | Meaning |
|---|---|
| $q_i$ | Size (records) of file $i$ (leaf weight) |
| $d_i$ | **Depth** of leaf $i$ in the merge tree |
| (Equivalent) | Sum of all **internal node** weights |

### Algorithm (tree method)
```text
treenode = record {
    treenode *lchild; treenode *rchild;
    integer weight;
};

Algorithm Tree(n)
// list is a global list of n single-node binary trees
{
    for i := 1 to n-1 do
    {
        pt := new treenode;                         // get a new tree node
        (pt -> lchild) := Least(list);              // merge two trees with
        (pt -> rchild) := Least(list);              // smallest lengths
        (pt -> weight) := ((pt -> lchild) -> weight)
                        + ((pt -> rchild) -> weight);
        Insert(list, pt);
    }
    return Least(list);        // tree left in list is the merge tree
}
```

### Time complexity — data structure matters

| `list` implemented as | Cost | Total |
|---|---|---|
| **Linear (unsorted) list** | `Least` = $O(n)$, called $\sim n$ times | $O(n^2)$ |
| **Min-heap** | build $O(n)$ + $(n-1)$ × $O(\log n)$ | $O(n \log n)$ |

### Worked example 1 — 3 files (board)
$F_1=12,\ F_2=20,\ F_3=5$ — compare merge orders:

| Order | Merges | Total record movements |
|---|---|---|
| $F_1+F_2$, then $+F_3$ | 32, 37 | $32+37 = 69$ |
| $F_3+F_2$, then $+F_1$ | 25, 37 | $25+37 = 62$ |
| $F_3+F_1$, then $+F_2$ ✅ | 17, 37 | $17+37 = \mathbf{54}$ *(computed — screenshot cut off)* |

→ Merge the **two smallest** first ($5+12$) = optimal.

### Worked example 2 — 7 files (board)
$\langle F_1..F_7\rangle = \langle 12, 8, 4, 18, 25, 2, 10\rangle$ → sorted: 2, 4, 8, 10, 12, 18, 25

| Step | Merge | New node |
|---|---|---|
| 1 | 2 + 4 | 6 |
| 2 | 6 + 8 | 14 |
| 3 | 10 + 12 | 22 |
| 4 | 14 + 18 | 32 |
| 5 | 22 + 25 | 47 |
| 6 | 32 + 47 | 79 |

**Total = 6+14+22+32+47+79 = 200** (options were a) 79 b) 213 c) **200** ✅)

### Worked example 3 — 5 files (board)
$\langle 25,15,5,10,30\rangle$: merge 5+10 = 15 → 15+15 = 30 → 25+30 = 55 → 30+55 = 85.

$$\sum d_i q_i = 2\cdot25 + 2\cdot15 + 3\cdot5 + 3\cdot10 + 2\cdot30 = 185$$

### Worked example 4 — slide (GATE Q): $f_1=20, f_2=30, f_3=10, f_4=5, f_5=30$

| Step | Merge | Node | Cost |
|---|---|---|---|
| 1 | $f_4(5)+f_3(10)$ | A = 15 | 15 |
| 2 | $A(15)+f_1(20)$ | B = 35 | 35 |
| 3 | $f_2(30)+f_5(30)$ | C = 60 | 60 |
| 4 | $B(35)+C(60)$ | root = 95 | 95 |

**Total = 15+35+60+95 = 205**

> 🧠 **Link:** Optimal Merge Pattern and **Huffman coding use the SAME greedy structure** — always merge the two smallest. Huffman = OMP applied to character frequencies.

---

## 7. Huffman Coding

**Data encoding & data compression technique.** Invented by **David Huffman** (MIT, 1952, as a graduate-student term paper). Used in fax machines, modems, computer networks, HDTV, ZIP/JPEG/MP3 — nearly everywhere digital data is compressed.

### Idea
- Fixed-length codes (e.g. ASCII, BCD) waste bits. Huffman assigns **variable-length codes: frequent symbols get shorter codes**.
- Codes are **prefix-free**: *no code is a prefix of another* → decoding is unambiguous.
- *(Board side note: BCD is a fixed radix-10 code, $r=10$. Example alphabet: $26+26+10+40 = 102$ characters → fixed-length would need $\lceil\log_2 102\rceil = 7$ bits each.)*

### Algorithm steps
1. Count frequencies / probabilities of all characters.
2. Build a **min-heap** (priority queue) of leaf nodes.
3. While heap has > 1 node: **extract the 2 nodes with lowest frequency**.
4. Create a new internal node with freq = **sum** of the two.
5. Insert it back into the heap.
6. Remaining node = **root** of the Huffman tree.
7. Assign **0/1** to left/right branches; read each code **from root to leaf**.

*(Same loop as `Algorithm Tree(n)` from Optimal Merge.)* **Time: $O(n \log n)$.**

### Formulas

$$L_{avg} = \sum_{i=1}^{n} p_i \,\ell_i \;\;\text{(bits/symbol)} \qquad \text{Total bits} = N \cdot L_{avg} = \sum f_i\,\ell_i$$

| Symbol | Meaning |
|---|---|
| $p_i$ / $f_i$ | Probability / frequency of symbol $i$ |
| $\ell_i$ (= $d_i$) | Code length = depth of leaf $i$ |
| $N$ | Number of characters in the message |

### Example 1 — symbols A–I (slide + diagram)

| Symbol | Prob | Code | Length | $p \times \ell$ |
|---|---|---|---|---|
| A | .36 | `1` | 1 | .36 |
| B | .15 | `011` | 3 | .45 |
| C | .13 | `010` | 3 | .39 |
| D | .11 | `0011` | 4 | .44 |
| E | .09 | `0010` | 4 | .36 |
| F | .07 | `0001` | 4 | .28 |
| G | .05 | `00001` | 5 | .25 |
| H | .03 | `000001` | 6 | .18 |
| I | .01 | `000000` | 6 | .06 |

**Average code length = 2.77 bits/symbol** (vs 4 bits fixed-length for 9 symbols).
Merge order: .01+.03=.04 → .04+.05=.09 → .07+.09=.16 → .09+.11=.20 → .13+.15=.28 → .16+.20=.36 → .28+.36=.64 → .36+.64=1.0

### Example 2 — frequencies on a tree (slide)
$a{=}2,\ e{=}4,\ h{=}8,\ g{=}16,\ f{=}11,\ c{=}5,\ j{=}6,\ d{=}21,\ b{=}10,\ i{=}8$ (total 91)

Merges: $a{+}e=6$ → $6{+}h=14$ → $14{+}g=30$ ; $c{+}j=11$ → $f{+}11=22$ ; $30{+}22=52$ ; $b{+}i=18$ → $d{+}18=39$ ; $52{+}39=91$.

Codes *(derived, 0 = left)*: d=`10`, g=`001`, f=`010`, b=`110`, i=`111`, h=`0001`, c=`0110`, j=`0111`, a=`00000`, e=`00001` → total = **283 bits**.

### Example 3 — board (5 symbols) + encode/decode
$L=\{a,b,c,d,e\}$ with probabilities $(0.23,\ 0.05,\ 0.45,\ 0.18,\ 0.09)$

Merges: $b{+}e = 0.14$ → $0.14{+}d = 0.32$ → $a{+}0.32 = 0.55$ → $c{+}0.55 = 1.0$

| Symbol | Code | Length |
|---|---|---|
| a | `10` | 2 |
| b | `1100` | 4 |
| c | `0` | 1 |
| d | `111` | 3 |
| e | `1101` | 4 |

$$L_{avg} = 2(0.23)+4(0.05)+1(0.45)+3(0.18)+4(0.09) = \mathbf{2.01\ bits/char}$$

- **Encode** `cccaaccdca` → `000 10 10 00 111 0 10` = `000101000111010` → **15 bits** (uniform 3-bit code would need **30 bits**).
- **Decode** `1000101111010…` → `10|0|0|10|111|10|10` → **`accadaa`**.
- **Decoding rule:** read bits from the root until you hit a leaf; output the symbol; restart at root.

---

## 8. GATE PYQs

### PYQ 1 — Fractional Knapsack
**Q.** Find an optimal solution to the Knapsack instance: $n=7,\ M=15$; $(p_1..p_7) = (10,5,15,7,6,18,3)$; $(w_1..w_7) = (2,3,5,7,1,4,1)$.

✅ **Answer:** $\langle x_1..x_7\rangle = \langle 1,\ 2/3,\ 1,\ 0,\ 1,\ 1,\ 1\rangle$, **profit ≈ 55.33** (board writes 55.3).
*Reasoning:* sort by $p/w$ and fill greedily (full worked example in Section 4).

---

### PYQ 2 — 0/1 Knapsack: greedy vs optimal
**Q.** Items (one unit each), capacity 11 kg, **no splitting**:

| Item | Weight (kg) | Value (₹) | $v/w$ |
|---|---|---|---|
| 1 | 10 | 60 | 6 |
| 2 | 7 | 28 | 4 |
| 3 | 4 | 20 | 5 |
| 4 | 2 | 24 | 12 |

Greedy sorts by value/weight (descending) and packs from the first item. Find $V_{opt} - V_{greedy}$.

✅ **Answer: 16**
- **Greedy:** order 4 (12), 1 (6), 3 (5), 2 (4). Take item 4 (w=2, remaining 9); item 1 (w=10) doesn't fit; take item 3 (w=4, remaining 5); item 2 (w=7) doesn't fit → $V_{greedy} = 24+20 = 44$.
- **Optimal:** item 1 alone = 60 (other combos: 2+3 = 48, 2+4 = 52, 3+4 = 44) → $V_{opt} = 60$.
- $60 - 44 = 16$.

> 🧠 **Takeaway:** the $p/w$ greedy is optimal only for **fractional** knapsack, **not 0/1** knapsack.

---

### PYQ 3 — Job Sequencing (2 parts)
**Q.** 9 tasks, unit time each, one at a time; profit earned if finished by deadline.

| Task | T1 | T2 | T3 | T4 | T5 | T6 | T7 | T8 | T9 |
|---|---|---|---|---|---|---|---|---|---|
| Profit | 15 | 20 | 30 | 18 | 18 | 10 | 23 | 16 | 25 |
| Deadline | 7 | 2 | 5 | 3 | 4 | 5 | 2 | 7 | 3 |

**Greedy schedule (sort by profit, latest free slot):**

| Slot | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
|---|---|---|---|---|---|---|---|
| Task | T2 | T7 | T9 | T5 | T3 | T1 | T8 |

Rejected: **T4** (slots 1–3 already full) and **T6** (slots 1–5 already full).

**(a) Are all tasks completed in the max-profit schedule?**
- (A) All tasks are completed ❌ — max deadline is 7, so only 7 slots exist for 9 tasks.
- (B) T1 and T6 left out ❌ — then T4 would stay in, but T2, T7, T9, T4 all have deadline ≤ 3 and need 4 slots in 3.
- (C) T1 and T8 left out ❌ — both have deadline 7 and free slots 6, 7 exist, so dropping them loses profit.
- **(D) T4 and T6 are left out ✅**

**(b) Maximum profit?**
- **(A) 147 ✅** = 20+23+25+18+30+15+16
- (B) 165 ❌ and (C) 167 ❌ — would require dropping ≤ 10 (or ≤ 8) worth of profit, but two tasks *must* be dropped (9 tasks, 7 slots) and the actual drop is 18+10 = 28.
- (D) 175 ❌ — equals the sum of **all** profits (infeasible).

---

### PYQ 4 — Optimal Merge Pattern
**Q.** Comparisons needed to merge sorted files (pairwise, optimal merge pattern):

| File | $f_1$ | $f_2$ | $f_3$ | $f_4$ | $f_5$ |
|---|---|---|---|---|---|
| Records | 20 | 30 | 10 | 5 | 30 |

Options: (A) 270 (B) 210 (C) 205 (D) 201

✅ **Answer: (C) 205** (merges: 15, 35, 60, 95 — see Section 6).
- (A) 270 ❌ — that's merging in the given order: 50+60+65+95 (not optimal).
- (B) 210 ❌ — larger than the optimum; the greedy result is the minimum.
- (D) 201 ❌ — lower than the proven minimum, so impossible.

---

### PYQ 5 — Huffman decoding (Fibonacci frequencies)
**Q.** $a{:}1,\ b{:}1,\ c{:}2,\ d{:}3,\ e{:}5,\ f{:}8,\ g{:}13,\ h{:}21$. Which character sequence matches the Huffman code `110111100111010`?

Options: (A) fdheg (B) ecgdf (C) dchfg (D) fehdg

*(No answer was marked on the slide — derived below.)*

**Build the tree** (merges: 1+1=2, 2+2=4, 4+3=7, 7+5=12, 12+8=20, 20+13=33, 33+21=54) → a **skewed tree**. Giving the heavier child `0`:

| h | g | f | e | d | c | b | a |
|---|---|---|---|---|---|---|---|
| `0` | `10` | `110` | `1110` | `11110` | `111110` | `1111110` | `1111111` |

**Decode:** `110` `11110` `0` `1110` `10` → **f d h e g**

✅ **Answer: (A) fdheg**
- (B) ecgdf ❌ — starts with e = `1110`, but string starts `1101…`.
- (C) dchfg ❌ — starts with d = `11110`, but string starts `11011…`.
- (D) fehdg ❌ — after f (`110`) next bits are `1111`, which doesn't match e = `1110`.

> ⚠️ Assumes the standard convention "heavier child = 0". Also sanity check: higher frequency ⇒ shorter code (g: 2 bits, f: 3, e: 4, d: 5).

---

### PYQ 6 — Expected length of Huffman-encoded message
**Q.** $P{=}0.22,\ Q{=}0.34,\ R{=}0.17,\ S{=}0.19,\ T{=}0.08$. Expected length (bits) of a 100-character message?

**Merges:** $T{+}R = 0.25$ → $S{+}P = 0.41$ → $0.25{+}Q = 0.59$ → $0.41{+}0.59 = 1$

| Symbol | P | Q | R | S | T |
|---|---|---|---|---|---|
| Length | 2 | 2 | 3 | 2 | 3 |

$L_{avg} = 0.22(2)+0.34(2)+0.17(3)+0.19(2)+0.08(3) = 2.25$ bits/char

✅ **Answer: 225 bits** ($100 \times 2.25$)

---

### PYQ 7 — Prefix-free code for `abbccddeee`
**Q.** Assign binary codes such that (1) no code is a prefix of another; (2) for letters with equal frequency, the one earlier in dictionary order gets a code length ≤ the other's. Find the minimum length of the encoded string.

Options: (A) 25 (B) 23 (C) 21 (D) 30

**Frequencies:** a=1, b=2, c=2, d=2, e=3 (total 10).
Huffman merges: $1{+}2=3$ → $2{+}2=4$ → $3{+}3=6$ → $4{+}6=10$; **cost = 3+4+6+10 = 23**.
Tie-breaking so rule 2 holds: lengths a=3, b=2, c=2, d=3, e=2 → $1(3)+2(2)+2(2)+2(3)+3(2) = 23$ ✓ (b ≤ c ≤ d in length).

✅ **Answer: (B) 23**
- (A) 25 ❌ and (D) 30 ❌ — not minimal (30 = fixed 3-bit code).
- (C) 21 ❌ — below the Huffman optimum, impossible.

---

### PYQ 8 — Which frequencies give this Huffman tree? (select all)
**Tree:** perfectly **balanced** — root → two internal nodes; **A, B siblings** (left), **C, D siblings** (right); all leaves at depth 2.

Options:
- A. $p_A{=}0.4,\ p_B{=}0.3,\ p_C{=}0.2,\ p_D{=}0.1$
- B. $p_A{=}0.35,\ p_B{=}0.25,\ p_C{=}0.2,\ p_D{=}0.2$
- C. $p_A=p_B=p_C=p_D=0.25$
- D. $p_A{=}0.2,\ p_B{=}0.35,\ p_C{=}0.2,\ p_D{=}0.25$

*(No answer marked on slide — derived.)*

**Condition:** the two smallest must be siblings and merge first; then the other two must merge together (needs smallest-pair sum ≥ the larger of the remaining two).

✅ **Answer: B and C**
- A ❌ — merge D+C = 0.3; then 0.3 ties with B and merges, leaving A (0.4) at depth 1 → skewed tree.
- **B ✅** — merges C+D (0.2+0.2), then B+A (0.25+0.35) → matches.
- **C ✅** — all equal; ties can be broken to get A–B and C–D pairs.
- D ❌ — two smallest are A and C (0.2 each) → they'd be siblings, but the tree has A with B.

---

## 9. Quick Revision Checklist

**Disjoint sets**
- [ ] Why sets are *pairwise disjoint*; set = tree; root = representative
- [ ] Union and Find definitions; tree vs pointer-table vs **array** ($P=-1$ for root) representation
- [ ] `Union(i,j): p[i]:=j` → $O(1)$; `Find(i)` → $O(h)$
- [ ] **Collapsing rule** (path compression) & the 6→4→9→5 example

**Greedy basics**
- [ ] Definition: sequence of decisions, locally optimal, never revisited
- [ ] 4 ideas: stages, selection procedure, feasibility, optimality
- [ ] Decision vs optimization problems
- [ ] Terminology: explicit vs implicit constraints, solution space, feasible, objective function, optimal
- [ ] n-Queens example (explicit $1\le x[i]\le n$, implicit = no attacks, $4!$)
- [ ] Control abstraction: `SELECT`, `FEASIBLE`, `UNION`; complexity $O(n\log n)$
- [ ] **Subset** paradigm (Knapsack, JSD) vs **Ordering** paradigm (MST, Optimal Merge)
- [ ] Applications: scheduling, shortest path, Huffman, MST

**Knapsack**
- [ ] Objective $\max\sum p_ix_i$, constraint $\sum w_ix_i\le M$, $0\le x_i\le1$
- [ ] Profit-greedy vs weight-greedy vs **$p/w$-greedy** (28.2 / 31 / **31.5**)
- [ ] `GREEDY_KNAPSACK` procedure; $O(n)$ after sorting
- [ ] $n=7, M=15$ example → 55.33
- [ ] Greedy fails for **0/1** knapsack (PYQ 2: 44 vs 60)

**Job Sequencing**
- [ ] Rules: unit time, one machine, profit only if by deadline
- [ ] Sort by profit ↓, place in latest free slot ≤ deadline
- [ ] `GreedyJob` algorithm → $O(n^2)$
- [ ] Examples A, B (329), C (110: J4→J3→J1→J2)
- [ ] 9-task PYQ → 147, T4 & T6 left out

**Optimal Merge Pattern**
- [ ] 2-way merge cost = $n+m$
- [ ] Merge two smallest first; total $=\sum d_iq_i$ = sum of internal nodes
- [ ] `Algorithm Tree(n)`; $O(n^2)$ linear list vs $O(n\log n)$ min-heap
- [ ] Examples: 54 (3 files), 200 (7 files), 185 (5 files), **205** (GATE)
- [ ] Link with Huffman

**Huffman**
- [ ] Purpose: variable-length, prefix-free compression; inventor & uses
- [ ] 7 algorithm steps; $O(n\log n)$
- [ ] $L_{avg}=\sum p_i\ell_i$; total bits $=N\cdot L_{avg}$
- [ ] A–I example → 2.77 bits/symbol
- [ ] 5-symbol example → 2.01 bits/char; encode (15 bits vs 30) and decode (`accadaa`)
- [ ] Decoding by walking the tree
- [ ] PYQs: Fibonacci decode (fdheg), expected length 225, `abbccddeee` → 23, which-frequencies-give-tree (B, C)

---

## 10. Corrections to Lecture Slides

Things I fixed/flagged while converting — double-check these against your own source:

1. **Huffman table slide (A–I):** codes shown for F, G (`00011`, `000010`) don't match the diagram or the stated average of 2.77. Correct lengths are F = 4, G = 5 (`0001`, `00001`), which give exactly 2.77. The diagram's "001" for F is also a typo for `0001`.
2. **Knapsack board (greedy on profit):** board shows 28.3; $25 + \frac{2}{15}\times24 = 28.2$.
3. **Optimal Merge:** the GATE question says "comparisons", but the accepted answer (205) is computed as the sum of merged-file sizes (record movements).
4. **`Union(i,j)`** as written assumes $i$ and $j$ are **roots**; calling it on non-root elements breaks the structure.
5. **Collapsing Find "O(1)"** is amortised/near-constant, not worst-case for the first find.
6. **PYQ 5 and PYQ 8** had no marked answer on the slides — answers above are derived by me.
7. **Optimal Merge example 3 (board):** the last merge order was partly cut off in the screenshot; 54 is computed.
