# Divide and Conquer — GATE DAA Lecture 06 Revision Notes

> Source: GeeksforGeeks GATE — *Design & Analysis of Algorithms*, Lecture 06 (Divide and Conquer).
> Note: this lecture is **DAA**, not Operating Systems.

## Topics Covered

1. [Control Abstraction & Recurrences](#1-control-abstraction--recurrences)
2. [Types of D&C Recurrences (Symmetric vs Asymmetric)](#2-types-of-dc-recurrences)
3. [Max-Min Problem](#3-max-min-problem)
4. [Binary Search](#4-binary-search)
5. [Merge Sort](#5-merge-sort)
6. [Quick Revision Checklist](#6-quick-revision-checklist)

---

## 1. Control Abstraction & Recurrences

**Divide and Conquer (D&C)** = split the problem into smaller subproblems of the same kind, solve each recursively, then **combine** the answers. Three ingredients: **SMALL** (is it small enough?), **DIVIDE**, **COMBINE**.

### 1.1 Control abstraction (two-way split on `A[p..q]`)

```text
Procedure DANDC(p, q)
    global n, A(1:n); integer m, p, q;        // 1 ≤ p ≤ q ≤ n
    if SMALL(p, q)
        then return G(p, q)                   // solve directly
    else
        m ← DIVIDE(p, q)                      // p ≤ m < q
        return COMBINE( DANDC(p, m), DANDC(m + 1, q) )
    endif
end DANDC
```

- Size of the current problem: $n = q - p + 1$
- Subproblems: `A[p..m]` and `A[m+1..q]`

### 1.2 General form (k subproblems)

```text
Algorithm DAndC(P)
{
    if Small(P) then return S(P);
    else
    {
        Divide P into smaller instances P1, P2, ..., Pk, k >= 1;
        Apply DAndC to each of these subproblems;
        return Combine(DAndC(P1), DAndC(P2), ..., DAndC(Pk));
    }
}
```

### 1.3 Recurrence relations

**Two equal halves:**

$$
T(n)=\begin{cases} g(n), & n \text{ small}\\ 2T\!\left(\frac{n}{2}\right)+f(n), & \text{otherwise}\end{cases}
$$

**General k subproblems (sizes $n_1,\dots,n_k$):**

$$
T(n)=\begin{cases} g(n), & n \text{ small}\\ T(n_1)+T(n_2)+\dots+T(n_k)+f(n), & \text{otherwise}\end{cases}
$$

| Symbol | Meaning |
|---|---|
| $T(n)$ | Time of DANDC on $n$ inputs |
| $g(n)$ | Time to solve **small** inputs directly (SMALL + G) |
| $f(n)$ | Time for **DIVIDE + COMBINE** (plus SMALL check) at one level |
| $n_i$ | Size of the $i$-th subproblem |

---

## 2. Types of D&C Recurrences

| | **Symmetric** | **Asymmetric** |
|---|---|---|
| Subproblem sizes | All **equal** | **Unequal** |
| General form | $T(n)=a\,T(n/b)+f(n)$, $a\ge1,\ b>1$ | $T(n)=T(\alpha n)+T((1-\alpha)n)+f(n)$ etc. |
| Meaning of params | $a$ = **number of subproblems**, $b$ = **size factor** of each | $\alpha,\beta,\gamma$ = fractions of $n$ |
| Solved using | **Master method** | **Recursion tree** |
| Classic example | Merge sort, Binary search (single branch) | **Quicksort** |

### Symmetric examples
1. $T(n)=2T(n/2)+f(n)$
2. $T(n)=3T(n/3)+f(n)$
3. $T(n)=4T(n/4)+f(n)$
4. $T(n)=4T(n/2)+f(n)$ — e.g., a $4\times4$ matrix split into **four** $2\times2$ blocks (note: 4 subproblems, each of size $n/2$)

### Asymmetric examples
- **Case 1** (two unequal parts — Quicksort style; use recursion tree):
  - $T(n)=T(n/3)+T(2n/3)+f(n)$
  - $T(n)=T(n/5)+T(4n/5)+f(n)$
  - General: $T(n)=T(\alpha n)+T((1-\alpha)n)+f(n)$
- **Case 2** (parts don't sum to $n$ / more than two parts; use recursion tree):
  - $T(n)=T(n/2)+T(n/4)+f(n)$
  - $T(n)=T(n/2)+T(n/4)+T(n/8)+f(n)$
  - General: $T(n)=T(\alpha n)+T(\beta n)+T(\gamma n)+f(n)$

---

## 3. Max-Min Problem

**Goal:** find the **maximum and minimum** of an array of $n$ elements using the fewest *element comparisons*.

Example array: `5092, 99, 90, 6, 158, 295` → Max = 5092, Min = 6.

### 3.1 Straight (naive) approach

```text
Algorithm STRAIT-MAXMIN(A, n, max, min)
{
    1. max ← min ← A[1];
    2. for i ← 2 to n
       {
           if (A[i] > max)  max ← A[i];  endif
           if (A[i] < min)  min ← A[i];  endif
       }
}
```

- **Total element comparisons = $2(n-1)$** (two per element, for elements 2..n)

### 3.2 Straight Max-Min V2 (use `else`)

```text
Algorithm STRAIT-MAXMIN-V2(A, n, max, min)
{
    1. max ← min ← A[1];
    2. for i ← 2 to n
       {
           if (A[i] > max)      max ← A[i];
           else if (A[i] < min) min ← A[i];
           endif
       }
}
```

| Case | Input order | Comparisons |
|---|---|---|
| **Best** | **Increasing** (1st test always true, 2nd never runs) | $n-1$ |
| **Worst** | **Decreasing** (1st always fails, 2nd always runs) | $2(n-1)$ |
| **Average** | 1st comparison fails for ~½ of the elements | $(n-1)+\dfrac{n}{2}=\dfrac{3n}{2}-1$ |

- **Space:** $O(1)$

### 3.3 Divide & Conquer Max-Min

**Idea:** split array into two halves, find (max, min) of each half recursively, then combine: `max = larger of two maxes`, `min = smaller of two mins`.

**Small(P) cases:**
- 1 element ($i=j$): `max = min = a[i]`
- 2 elements ($i=j-1$): **1 comparison** decides max & min

```text
Algorithm MaxMin(i, j, max, min)
// a[1:n] is a global array. 1 <= i <= j <= n.
// Sets max and min to the largest and smallest values in a[i..j].
{
    if (i = j) then max := min := a[i];                 // Small(P): 1 element
    else if (i = j - 1) then                            // Small(P): 2 elements
    {
        if (a[i] < a[j]) then
        {
            max := a[j]; min := a[i];
        }
        else
        {
            max := a[i]; min := a[j];
        }
    }
    else
    {   // P is not small: divide into subproblems
        mid := floor((i + j) / 2);                      // find where to split
        MaxMin(i, mid, max, min);                       // solve subproblems
        MaxMin(mid + 1, j, max1, min1);
        // Combine
        if (max < max1) then max := max1;
        if (min > min1) then min := min1;
    }
}
```

> OCR fix: the first slide version had `Max := :=a[i]` → corrected to `max := min := a[i]`.

#### Worked example (n = 9)

`A = [34, 15, -2, 0, 15, 25, 45, 63, -1]` (indices 1–9)

```text
(1,9)  → mid=5
├── (1,5) → mid=3
│   ├── (1,3) → mid=2
│   │   ├── (1,2): max=34, min=15
│   │   └── (3,3): max=-2, min=-2
│   │   ⇒ (1,3): max=34, min=-2
│   └── (4,5): max=15, min=0
│   ⇒ (1,5): max=34, min=-2
└── (6,9) → mid=7
    ├── (6,7): max=45, min=25
    └── (8,9): max=63, min=-1
    ⇒ (6,9): max=63, min=-1
⇒ (1,9): max=63, min=-2     ← final answer
```

Second small example (n = 6): `33 11 44 | 55 66 22` → pair-level (min,max): (33,33), (11,44), (55,55), (22,66) → (11,44), (22,66) → final **(min=11, max=66)**.

#### Performance of D&C Max-Min

**Space (run-time stack):** depth of recursion tree = $\log n$ → **Space = $O(\log n)$** (the stack grows as DandC(1,9) → (1,5) → (1,3) → (1,2) ...).

**Comparison count recurrence** ($T(n)$ = number of element comparisons):

$$
T(n)=\begin{cases}0, & n=1\\ 1, & n=2\\ 2T\!\left(\frac{n}{2}\right)+2, & n>2\end{cases}
$$

(the "+2" = one comparison for max-combine, one for min-combine)

**Solving by repeated substitution:**

$$
\begin{aligned}
T(n) &= 2T(n/2)+2\\
&= 4T(n/4)+4+2\\
&= 2^k\,T\!\left(\tfrac{n}{2^k}\right)+\sum_{i=1}^{k}2^i
\end{aligned}
$$

Stop when $\dfrac{n}{2^k}=2 \Rightarrow n=2^{k+1}\Rightarrow k=\log_2 n-1$.

$$
T(n)=\frac{n}{2}\cdot T(2)+\frac{2(2^k-1)}{2-1}=\frac{n}{2}+2^{k+1}-2=\frac{n}{2}+n-2=\boxed{\frac{3n}{2}-2}
$$

This holds for **all cases of input** (best = worst = average).

### 3.4 Comparison table: Naive vs D&C (element comparisons)

| Input case | Non-D&C (straight) | D&C |
|---|---|---|
| **Increasing** (best) | $n-1$ ✅ (better) | $\frac{3n}{2}-2$ |
| **Decreasing** (worst) | $2(n-1)$ | $\frac{3n}{2}-2$ ✅ |
| **Average** | $\frac{3n}{2}-1$ | $\frac{3n}{2}-2$ ✅ |

- **Time complexity: $O(n)$** for both; D&C wins on worst/average comparisons but uses **$O(\log n)$ stack** vs $O(1)$.

### 3.5 Non-recursive pairing trick (from the 100-number problem)

For $n$ numbers (n even): compare elements **in pairs** → $n/2$ comparisons → gives $n/2$ "winners" (max-candidates) and $n/2$ "losers" (min-candidates). Then find max among winners ($n/2-1$) and min among losers ($n/2-1$).

$$
\text{Total}=\frac{n}{2}+\left(\frac{n}{2}-1\right)+\left(\frac{n}{2}-1\right)=\frac{3n}{2}-2
$$

### 3.6 GATE PYQs — Max-Min

**Q1.** An array of $n$ numbers is given, where $n$ is even. The max as well as the min of these $n$ numbers needs to be determined. Which is **TRUE** about the number of comparisons needed?

| Option | Statement | |
|---|---|---|
| A | At least $2n-c$ comparisons, for some constant $c$, are needed | ❌ |
| **B** | **At most $1.5n-2$ comparisons are needed** | ✅ **Correct** |
| C | At least $n\log_2 n$ comparisons are needed | ❌ |
| D | None of the above | ❌ |

- **Why B:** the pairing / D&C method achieves $\frac{3n}{2}-2$.
- **A wrong:** $2n-c$ would be a lower bound above $1.5n-2$, but $1.5n-2$ is achievable.
- **C wrong:** $n\log n$ is far more than needed; the problem is solvable in $O(n)$.
- **D wrong:** B is true.

**Q2.** Let $P$ be an array of $n$ integers. Let $t$ be the lowest upper bound on the number of comparisons required to find min and max in an arbitrary array of $n$ elements. Which is correct?

| Option | Statement | |
|---|---|---|
| A | $t > 2n-2$ | ❌ |
| B | $t > 3\lceil n/2\rceil$ and $t \le 2n-2$ | ❌ |
| **C** | **$t > n$ and $t \le 3\lceil n/2\rceil$** | ✅ **Correct** |
| D | $t > \lceil\log_2 n\rceil$ and $t \le n$ | ❌ |

- **Why C:** the best-known bound is $3\lceil n/2\rceil-2$, which is $>n$ (for $n$ large enough) and $\le 3\lceil n/2\rceil$.
- **A wrong:** $t$ is only about $1.5n$, not above $2n-2$.
- **B wrong:** $t = 3\lceil n/2\rceil-2$ is **less than** $3\lceil n/2\rceil$, so "$t>3\lceil n/2\rceil$" fails.
- **D wrong:** $t\le n$ fails; you need roughly $1.5n$ comparisons.

**Q3.** The minimum number of comparisons required to find the min and max of **100 numbers** is ______.

**Answer: 148**

| Step | Comparisons |
|---|---|
| Pair up 100 numbers → 50 pairs → 50 "max-list" + 50 "min-list" | 50 |
| Max of the 50-element max-list | 49 |
| Min of the 50-element min-list | 49 |
| **Total** | $50+49+49=\mathbf{148}$ |

Check with the formula: $\frac{3\cdot100}{2}-2=148$ ✔

---

## 4. Binary Search

**Pre-condition:** array must be **sorted** (non-decreasing) and allow **random access** (array, not linked list).

**Idea:** compare key $x$ with the middle element; discard the half that can't contain $x$. Only **one** subproblem is solved (no real COMBINE step).

### 4.1 Trace example — Find `75`

Array (index 0–9): `1, 3, 12, 14, 23, 34, 55, 65, 75, 78`

| Iteration | low–high | mid | arr[mid] | Decision |
|---|---|---|---|---|
| 1 | 0 – 9 | 4 | 23 | $23<75$ → go right (low = 5) |
| 2 | 5 – 9 | 7 | 65 | $65<75$ → go right (low = 8) |
| 3 | 8 – 9 | 8 | **75** | `arr[mid] == x` → **found at index 8** |

### 4.2 Iterative Binary Search

```text
Algorithm BinSearch(a, n, x)
// Given array a[1:n] in non-decreasing order, n >= 0,
// determine whether x is present; if so return j such that x = a[j]; else return 0.
{
    low := 1; high := n;
    while (low <= high) do
    {
        mid := floor((low + high) / 2);
        if (x < a[mid])       then high := mid - 1;
        else if (x > a[mid])  then low  := mid + 1;
        else return mid;
    }
    return 0;
}
```

### 4.3 Recursive Binary Search

(`i` = lower index, `l` = upper index)

```text
Algorithm BinSrch(a, i, l, x)
// Given a[i:l] in non-decreasing order, 1 <= i <= l,
// determine whether x is present; if so return j with x = a[j]; else return 0.
{
    if (l = i) then                       // Small(P): one element
    {
        if (x = a[i]) then return i;
        else return 0;
    }
    else
    {   // Reduce P into a smaller subproblem
        mid := floor((i + l) / 2);
        if (x = a[mid]) then return mid;
        else if (x < a[mid]) then
            return BinSrch(a, i, mid - 1, x);
        else
            return BinSrch(a, mid + 1, l, x);
    }
}
```

> OCR/typo fixes: `BinSearch`/`BinSrch` naming made consistent; missing `)` in `if (x = a[mid])` added.

### 4.4 D&C representation of Binary Search

Instance $I=\langle n,\ a_1\dots a_n,\ x\rangle$, with $k=\lfloor\frac{1+n}{2}\rfloor$ (middle):

| Condition | Subproblem |
|---|---|
| $x<a_k$ | $I_1=\langle k-1,\ a_1\dots a_{k-1},\ x\rangle$ (left part) |
| $x=a_k$ | $I_2=\langle 1,\ a_k,\ x\rangle$ (found — stop) |
| $x>a_k$ | $I_3=\langle n-k,\ a_{k+1}\dots a_n,\ x\rangle$ (right part) |

Only **one** of $I_1, I_3$ is explored each step.

### 4.5 Binary decision tree (n = 7)

```text
            4
         /     \
        2       6
       / \     / \
      1   3   5   7
```

- Height $h=\log_2(n+1)$ → worst-case comparisons ∝ height → **$O(\log n)$**.

### 4.6 Performance

$$
T(n)=\begin{cases}c, & n=1\\ T\!\left(\frac{n}{2}\right)+a, & n>1\end{cases}
\ \Rightarrow\ \boxed{T(n)=O(\log n)}
$$

($a$ = constant work for one comparison + mid computation.)

### 4.7 Linear (sequential) search — average case formula

$$
A(n)=\sum_{i=1}^{n}p_i\cdot t_i
$$

($p_i$ = probability that the key is at position $i$; $t_i$ = comparisons needed for position $i$.) For a **successful** search with equally likely positions:

$$
A(n)=\frac{1+2+3+\dots+n}{n}=\frac{n(n+1)}{2n}=\frac{n+1}{2}
$$

### 4.8 GATE PYQs — Binary Search / Searching

**Q1.** The average number of key comparisons in a **successful sequential search** in a list of length $n$ is:

| Option | Value | |
|---|---|---|
| A | $\log n$ | ❌ |
| B | $(n-1)/2$ | ❌ |
| C | $n/2$ | ❌ |
| **D** | **$(n+1)/2$** | ✅ **Correct** (derived above) |

- A is binary search's behaviour, not sequential. B and C are off by a constant because position $i$ costs $i$ comparisons (the sum $1..n$ gives $(n+1)/2$).

**Q2.** The recurrence arising for the complexity of Binary Search is:

| Option | Recurrence | |
|---|---|---|
| A | $T(n)=2T(n/2)+k$ | ❌ |
| **B** | **$T(n)=T(n/2)+k$, $k$ constant** | ✅ **Correct** |
| C | $T(n)=T(n/2)+\log n$ | ❌ |
| D | $T(n)=T(n/2)+n$ | ❌ |

- A: that would search **both** halves. C/D: work per step is $O(1)$ (one comparison), not $\log n$ or $n$.

**Q3.** Let $F(n)$ be the maximum number of comparisons while searching a sorted array of size $n$ using Binary Search. Which ONE is TRUE?

| Option | Statement | |
|---|---|---|
| **A** | **$F(n)=F(\lfloor n/2\rfloor)+1$** | ✅ **Correct** |
| B | $F(n)=F(\lfloor n/2\rfloor)+F(\lfloor n/2\rfloor)$ | ❌ |
| C | $F(n)=F(\lfloor n/2\rfloor)$ | ❌ |
| D | $F(n)=F(n-1)+1$ | ❌ |

- B: searches both halves (that's not binary search). C: ignores the comparison made at mid. D: describes linear search.

**Q4.** For which input does Binary Search take $O(\log n)$ in the worst case?

| Option | Input | |
|---|---|---|
| A | Array of $n$ integers in any order | ❌ |
| **B** | **Array of $n$ integers in increasing order** | ✅ **Correct** |
| C | Linked list of $n$ integers in any order | ❌ |
| D | Linked list of $n$ integers in increasing order | ❌ |

- A, C: not sorted → halving is invalid. C, D: linked list has no random access → reaching `mid` costs $O(n)$, destroying the $\log n$ bound.

---

## 5. Merge Sort

- A **Divide-and-Conquer** sorting algorithm, **invented by John von Neumann in 1945**.
- **Divide:** split array into two halves (mid).
- **Conquer:** recursively sort both halves.
- **Combine:** **merge** the two sorted halves into one sorted array.
- Splitting continues until each subarray has **1 element** (trivially sorted).

### Worked example

Input: `7 3 2 16 24 4 11 9`

```text
Divide:
[7 3 2 16 24 4 11 9]
[7 3 2 16]        [24 4 11 9]
[7 3] [2 16]      [24 4] [11 9]
[7][3] [2][16]    [24][4] [11][9]

Merge (conquer/combine):
[3 7] [2 16]      [4 24] [9 11]
[2 3 7 16]        [4 9 11 24]
[2 3 4 7 9 11 16 24]   ← sorted
```

Recurrence (symmetric type, from Section 1): $T(n)=2T(n/2)+f(n)$ with $f(n)$ = merge cost.

> *Addition (not in the slides, standard result):* merge cost is $\Theta(n)$ → $T(n)=2T(n/2)+n$ → **$O(n\log n)$** in all cases; needs $O(n)$ extra space.

---

## Memory Tricks

The lecture has no explicit mnemonic, so these are quick aids (my additions):

- **D-C-C** → **D**ivide, **C**onquer, **C**ombine.
- **Symmetric = Master method, Asymmetric = Recursion tree.**
- **"3n/2 − 2"** → compare in **pairs** (n/2), then n/2 − 1 each for max-list and min-list.
- **Binary Search = one branch only** → $T(n/2)+c$ (not $2T(n/2)$).

---

## 6. Quick Revision Checklist

- [ ] D&C control abstraction: SMALL, G, DIVIDE, COMBINE, and $n=q-p+1$
- [ ] General DAndC(P) with $k$ subproblems
- [ ] Recurrences: $T(n)=g(n)$ vs $2T(n/2)+f(n)$ and the $k$-subproblem form
- [ ] Meaning of $g(n)$ and $f(n)$
- [ ] Symmetric recurrence $aT(n/b)+f(n)$: meaning of $a$, $b$; Master method
- [ ] Asymmetric recurrences (Quicksort-style, Case 1 and Case 2); solved by recursion tree
- [ ] Straight Max-Min: $2(n-1)$ comparisons
- [ ] Max-Min V2 with `else`: best $n-1$ (increasing), worst $2(n-1)$ (decreasing), average $\frac{3n}{2}-1$; space $O(1)$
- [ ] D&C Max-Min algorithm (cases for 1 and 2 elements; combine step)
- [ ] D&C Max-Min trace on a 9-element array; stack space $O(\log n)$
- [ ] Recurrence $T(n)=2T(n/2)+2$, $T(2)=1$ → $\frac{3n}{2}-2$ (all input cases)
- [ ] Naive vs D&C comparison table; time $O(n)$
- [ ] Pairing method; answer 148 for $n=100$
- [ ] PYQs: Max-Min (1.5n − 2; $3\lceil n/2\rceil$ bounds; 100 numbers → 148)
- [ ] Binary search: sorted + random access required; trace on key 75
- [ ] Iterative and recursive BinSearch code
- [ ] D&C view: instances $I_1, I_2, I_3$ with $k=\frac{1+n}{2}$
- [ ] Decision tree height $\log(n+1)$; $T(n)=T(n/2)+c=O(\log n)$
- [ ] Sequential search average: $(n+1)/2$; formula $A(n)=\sum p_i t_i$
- [ ] PYQs: BS recurrence, $F(n)=F(\lfloor n/2\rfloor)+1$, linked list vs array
- [ ] Merge sort: divide → sort halves → merge; von Neumann, 1945; worked 8-element example
