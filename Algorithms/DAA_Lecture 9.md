# GATE CS/IT — DAA Lecture 09 Notes
### Divide & Conquer (Recursion Tree, Long Integer Multiplication, Karatsuba, Toom-Cook) + Heap Data Structure

> **Source:** GeeksforGeeks GATE video-lecture slides. The PDF actually contains **two lectures back-to-back**:
> **Part 1** – *Design & Analysis of Algorithms: Divide and Conquer, Lecture 09* and
> **Part 2** – *Design & Analysis of Algorithms: Heap and Set Data Structures, Lecture 9*.
>
> **Scope notes**
> - The Part-1 agenda slide lists *Control Abstraction, Max-Min, Binary Search, Merge Sort*, but the slides that follow actually teach **recursion trees → long integer multiplication → Karatsuba → Toom-Cook**. Notes follow the real content.
> - The Part-2 agenda lists *Heap Operations, Heap Sort, Set Operations, Find and Create*. This PDF only covers **heap operations + heapsort + heap PYQs**. Set operations / Find-Create are not in these slides.
> - Legend: **(lecture)** = answer/derivation visible on the slide. **(derived)** = worked out by me because the slide left it blank. 🆕 = my own clarification, not from the lecture.

---

## Table of Contents

**Part 1 — Divide & Conquer**
1. [Recursion Tree Method](#1-recursion-tree-method)
2. [Long Integer Multiplication (LIM)](#2-long-integer-multiplication-lim)
3. [Karatsuba Algorithm](#3-karatsuba-algorithm)
4. [Toom-Cook Algorithm (multi-way split)](#4-toom-cook-algorithm-multi-way-split)
5. [Part 1 PYQ](#5-gate-pyq--divide--conquer)

**Part 2 — Heaps**
6. [Heap Basics & Array Representation](#6-heap-basics--array-representation)
7. [Heap Creation: Insertion Method vs Heapify](#7-heap-creation-insertion-method-vs-heapify)
8. [Time Complexity of Build-Heap](#8-time-complexity-of-build-heap)
9. [Heap Sort](#9-heap-sort)
10. [Insert / Delete / Inc-Dec Key](#10-insert--delete--incdec-key)
11. [Heap PYQs](#11-gate-pyqs--heaps)

[Memory Tricks](#memory-tricks--mnemonics) · [Quick Revision Checklist](#quick-revision-checklist)

---

# PART 1 — DIVIDE & CONQUER

## 1. Recursion Tree Method

**Use:** visualise a recurrence, compute cost **per level**, then **sum over all levels** to get $T(n)$.

### 1.1 Steps (general recurrence $T(n) = aT(n/b) + f(n)$)

| Step | What to do |
|---|---|
| 1 | Write the recurrence, e.g. $T(n) = 2T(n/2) + n$ |
| 2 | Root cost $= f(n)$ (work done at the top level) |
| 3 | At level $i$: there are $a^i$ sub-problems, each of size $n/b^i$ |
| 4 | Cost at level $i$ $= a^i \cdot f(n/b^i)$ — sum over all levels |
| 5 | Tree depth $= \log_b n$; sum of all level costs $= T(n)$ |

**Formulas**
- $a^i$ = number of nodes at level $i$ ($a$ = number of sub-problems per call)
- $n/b^i$ = size of each sub-problem at level $i$ ($b$ = shrink factor)
- Last level $k$: $\dfrac{n}{b^k}=1 \Rightarrow k=\log_b n$

### 1.2 Merge Sort recurrence (warm-up)

$$T(n)=\begin{cases} c & n=1\\ 2T(n/2)+cn & n>1\end{cases}$$

- Every level costs $cn$; there are $\log n + 1$ levels.
- **Total** $= cn\log n + cn = \Theta(n\log n)$.

### 1.3 Worked examples

#### Example 1 — Balanced: $T(n)=2T(n/2)+n$

| Level | Nodes | Size each | Cost of level |
|---|---|---|---|
| 0 | 1 | $n$ | $n$ |
| 1 | 2 | $n/2$ | $n$ |
| 2 | 4 | $n/4$ | $n$ |
| … | … | … | $n$ |
| $k$ | $2^k$ | $n/2^k=1$ | $2^k\cdot T(1)=n$ |

- $\dfrac{n}{2^k}=1 \Rightarrow n=2^k \Rightarrow k=\log_2 n$
- $T(n)= n(k+1) = n(\log n + 1) = \mathbf{O(n\log n)}$ (in fact $\Theta$)

#### Example 2 — Unbalanced: $T(n)=T(n/3)+T(2n/3)+n$

Branches shrink at different rates → tree is **not full**, so find **two bounds**:

| Bound | Which path | Depth $k$ | Result |
|---|---|---|---|
| **Lower** ($T(n)\ge$ LHS) | **shortest** path (always ÷3): $n/3^k=1$ | $k=\log_3 n$ | $T(n)\ge n(\log_3 n+1)\Rightarrow \Omega(n\log n)$ |
| **Upper** ($T(n)\le$ RHS) | **longest** path (always ×2/3): $n(2/3)^k=1\Rightarrow(3/2)^k=n$ | $k=\log_{3/2} n$ | $T(n)\le n(\log_{3/2} n+1)\Rightarrow O(n\log n)$ |

Both bounds match ⇒ $T(n)=\mathbf{\Theta(n\log n)}$.

#### Example 3 — $T(n)=T(n/5)+T(4n/5)+n$

- Lower: shortest path $k=\log_5 n$ ⇒ $T(n)\ge n(\log_5 n+1)$ ⇒ $\Omega(n\log n)$
- Upper: longest path $k=\log_{5/4} n$ ⇒ $T(n)\le n(\log_{5/4} n+1)$ ⇒ $O(n\log n)$
- ⇒ $\Theta(n\log n)$

#### Generalised form (**remember this**)

$$T(n)=T(\alpha n)+T((1-\alpha)n)+O(n),\quad 0<\alpha<1\;\;\Rightarrow\;\; T(n)=\Theta(n\log n)$$

(Splits that add up to exactly $n$ keep every level's cost $=n$.)

#### Example 5 — Fibonacci-type: $T(n)=T(n-1)+T(n-2)+c,\; T(1)=1,\,T(0)=0$

Level costs: $1\cdot c,\; 2c,\; 4c,\dots$ (number of nodes doubles) → cost at level $i$ is $2^i c$.

| Bound | Path | Depth | Sum of level costs | Result |
|---|---|---|---|---|
| **Upper** | **longest** path (always $n-1$): $n-k=0$ | $k=n$ | $\sum_{i=0}^{n}2^i=2^{n+1}-1$ | $T(n)=O(2^n)$ |
| **Lower** | **shortest** path (always $n-2$): $n-2k=0$ | $k=n/2$ | $\sum_{i=0}^{n/2}2^i=2^{n/2+1}-1$ | $T(n)=\Omega(2^{n/2})$ |

Hence $T(n)=O(2^n)$ and $\Omega(2^{n/2})$, but **$T(n)\ne\Theta(2^n)$** — the bounds are not within a constant factor.

**Why $2^n$ and $2^{n/2}$ are different classes (asymptotic comparison slide):**
- $f(n)=n$ vs $g(n)=2n$ (or $n/2$) → same order (constant factor).
- $f(n)=2^n$ vs $g(n)=2^{n/2}$: write $2^n = 2^{n/2}\cdot 2^{n/2}$, cancel one $2^{n/2}$ ⇒ compare $2^{n/2}$ vs $1$ ⇒ ratio grows unboundedly ⇒ **LHS > RHS**, not $\Theta$ of each other.
- Lesson: *halving the exponent changes the class; halving the base argument ($n\to n/2$) does not.*

#### Example 6 — (CLRS) $T(n)=3T(n/4)+n^2$

- Depth: $n/4^k=1\Rightarrow k=\log_4 n$
- Level $i$ cost $=\left(\tfrac{3}{16}\right)^i n^2$ (3 children each of size $n/4$ → $3\cdot (n/4)^2 = \tfrac{3}{16}n^2$)
- Leaves: $3^k\,T(1)=3^{\log_4 n}=n^{\log_4 3}$ → $\Theta(n^{\log_4 3})$

$$T(n)=n^2\sum_{i=0}^{k-1}\left(\tfrac{3}{16}\right)^i+3^k T(1)\;\le\; n^2\cdot\frac{1}{1-\frac{3}{16}}+n^{\log_4 3}\;=\;c_1 n^2+n^{\log_4 3}$$

using the geometric series $\sum_{i=0}^{\infty}x^i=\dfrac{1}{1-x}$ for $|x|<1$.

⇒ $T(n)=\mathbf{\Theta(n^2)}$ (root dominates; $\log_4 3<2$).

#### Example 7 — (MIT 6.046) $T(n)=T(n/4)+T(n/2)+n^2$

| Level | Cost |
|---|---|
| 0 | $n^2$ |
| 1 | $\tfrac{5}{16}n^2$ (since $(n/4)^2+(n/2)^2$) |
| 2 | $\tfrac{25}{256}n^2=(\tfrac{5}{16})^2n^2$ |

Total $=n^2\left(1+\tfrac{5}{16}+(\tfrac{5}{16})^2+\cdots\right)=\Theta(n^2)$ — **geometric series, decreasing ratio**.

### 1.4 Quick rule — what the level costs do

| Level cost pattern | Total dominated by | Example |
|---|---|---|
| Constant every level | $f(n)\times$ #levels | $2T(n/2)+n \to n\log n$ |
| **Decreasing** geometric (ratio $<1$) | **Root** → $\Theta(f(n))$ | $3T(n/4)+n^2 \to n^2$ |
| Increasing geometric (ratio $>1$) | **Leaves** | $4T(n/2)+n \to n^2$ |

### 1.5 Practice recurrences from the lecture (left unsolved on the slide)

| # | Recurrence | Answer **(derived)** | Reason |
|---|---|---|---|
| 1 | $T(n)=T(n/2)+T(n/4)+n$ | $\Theta(n)$ | ratio $\tfrac12+\tfrac14=\tfrac34<1$ → decreasing geometric, root dominates |
| 2 | $T(n)=T(n/2)+T(n/4)+n^2$ | $\Theta(n^2)$ | ratio $\tfrac14+\tfrac1{16}=\tfrac5{16}<1$ |
| 3 | $T(n)=T(n/3)+T(2n/3)+n^2$ | $\Theta(n^2)$ | ratio $\tfrac19+\tfrac49=\tfrac59<1$ |

---

## 2. Long Integer Multiplication (LIM)  ★ *(starred as important in lecture)*

### 2.1 The problem

- For machine-sized integers (`int x;` = 4 B = 32 bits, bounded min/max): $u\pm v = O(1)$ and $u*v=O(1)$ — **only because the numbers fit in a hardware word**.
- For *long* ($n$-digit, $n$ large) integers, how to represent?

| Option | Idea | Verdict |
|---|---|---|
| **H/W solution** | Increase the size of the hardware data type | ❌ **Not viable** |
| **S/W solution** | Store digits in **integer arrays** | ✅ Used |

Setup: $u, v$ are $n$-digit numbers stored in arrays $A[1..n]$, e.g. $u=4586392$ → `A = [4,5,8,6,3,9,2]` (indices 1–7).

### 2.2 Basic operations on array-stored numbers

| Operation | Method | Time |
|---|---|---|
| $u\pm v$ | `for i ← 1 to n: c[i] = u[i] ± v[i]` (carry handling ignored) | $O(n)$ |
| $u*v$ | **School method** (non-D&C): nested `for i`, `for j` over $n$ digits | $O(n^2)$ |

### 2.3 Divide & Conquer LIM

Let $u,v$ be $n$-digit integers, $m=\lfloor n/2\rfloor$. Split each into two $\tfrac n2$-digit halves:

$$u = w\cdot 10^m + x,\qquad v = y\cdot 10^m + z$$

| Symbol | Meaning |
|---|---|
| $w = u\,/\,10^m$ | high half of $u$ (integer division) |
| $x = u\,\%\,10^m$ | low half of $u$ |
| $y = v\,/\,10^m$ | high half of $v$ |
| $z = v\,\%\,10^m$ | low half of $v$ |

**Example:** $u=1234=12\times10^2+34$, $v=5678=56\times10^2+78$ (here $m=2$).

$$u\cdot v=(w\,10^m+x)(y\,10^m+z)=\underbrace{wy}_{}\,10^{2m}+(wz+xy)\,10^m+xz\qquad\text{…(1)}$$

- **4 multiplications** of $\tfrac n2$-digit numbers ($wy,\;wz,\;xy,\;xz$) + additions/shifts $O(n)$:

$$T(n)=\begin{cases}c & n=1\\ 4T(n/2)+bn & n>1\end{cases}\;\;\Rightarrow\;\; \mathbf{O(n^2)}\;\;(\log_2 4=2)$$

> ⚠️ Plain D&C gives **no improvement** over the school method. The gain comes from reducing 4 → 3 multiplications (Karatsuba).

---

## 3. Karatsuba Algorithm

- Discovered by **Anatoly Karatsuba in 1960**, published **1962**.
- A D&C algorithm that reduces the multiplication of two $n$-digit numbers to **3 multiplications of $\tfrac n2$-digit numbers** (instead of 4) → at most $n^{\log_2 3}\approx n^{1.58}$ single-digit multiplications (vs $n^2$ for the traditional method).
- **Karatsuba = Toom-2.**

### 3.1 The trick

From (1), name the two "outer" products:

$$P_{r1}=w\cdot y,\qquad P_{r2}=x\cdot z$$

Let $t$ be an $\tfrac n2$-digit-ish integer: $t=P_{r3}=(w+x)(y+z)=wy+(wz+xy)+xz$

$$\Rightarrow\; wz+xy = P_{r3}-(P_{r1}+P_{r2})$$

$$\boxed{\,u\cdot v = P_{r1}\cdot10^{2m}+\big(P_{r3}-(P_{r1}+P_{r2})\big)\cdot10^{m}+P_{r2}\,}\qquad\text{…(2)}$$

Only **three** recursive multiplications: $P_{r1},P_{r2},P_{r3}$.

```text
// Karatsuba (B = 10^m, u = u1*B + u0, v = v1*B + v0)
m1 = u1 * v1                  // = Pr1
m2 = u0 * v0                  // = Pr2
m3 = (u1 + u0) * (v1 + v0)    // = Pr3
// uv = m1*B^2 + (m3 - m1 - m2)*B + m2
```

$$T(n)=3T(n/2)+bn\;\Rightarrow\;T(n)=O\!\left(n^{\log_2 3}\right)=O(n^{1.58})$$

> 🆕 Minor detail: $(w+x)$ and $(y+z)$ can carry into one extra digit; this doesn't change the asymptotic result.

### 3.2 Classical vs Karatsuba

| | Classical LIM (D&C) | Karatsuba |
|---|---|---|
| Sub-multiplications | **4** ($wy, wz, xy, xz$) | **3** ($P_{r1},P_{r2},P_{r3}$) |
| Recurrence | $T(n)=4T(n/2)+O(n)$ | $T(n)=3T(n/2)+O(n)$ |
| Complexity | $O(n^2)$ | $O(n^{\log_2 3})\approx O(n^{1.585})$ |

---

## 4. Toom-Cook Algorithm (multi-way split)

- Named after **Andrei Toom** (b. 1942, introduced the low-complexity algorithm) and **Stephen Cook** (b. 1939, cleaned up its description).
- Multiplication algorithm for large integers: split each number into **$k$ parts**, do operations on parts; larger $k$ lets you **combine many sub-multiplications** → lower complexity. Sub-multiplications are again done **recursively** with Toom-Cook.
- **Toom-3 ≠ Toom-Cook**: Toom-3 is just *one instance* ($k=3$) of Toom-Cook. (People wrongly use the names interchangeably.)
- **Karatsuba is Toom-2** (split in two; reduces 4 mults → 3).
- **Toom-3** reduces **9 multiplications to 5** → $\Theta(n^{\log 5/\log 3})\approx\Theta(n^{1.46})$.
- General **Toom-$k$**: $\Theta\!\big(c(k)\,n^{e}\big)$ with $e=\dfrac{\log(2k-1)}{\log k}$; $n^e$ = time on sub-multiplications, $c(k)$ = time on additions & multiplication by small constants.

### 4.1 Split-by-split comparison

**3-way split** (3 parts of size $n/3$):

| Method | Recurrence | Complexity |
|---|---|---|
| Plain D&C | $9T(n/3)+O(n)$ | $O(n^2)$ |
| Karatsuba-style (AK) | $8T(n/3)+O(n)$ | $O(n^{\log_3 8})$ |
| **Toom-Cook (TC)** | $5T(n/3)+bn$ | $O(n^{\log_3 5})=O(n^{1.46})$ |

- For a generic $x\,T(n/3)+O(n)$ → $O(n^{\log_3 x})$. To beat Karatsuba we need $\log_3 x<1.58 \Rightarrow \mathbf{x=5}$.

**4-way split** (4 parts of size $n/4$):

| Method | Recurrence | Complexity |
|---|---|---|
| Plain D&C | $16T(n/4)+O(n)$ | $O(n^2)$ |
| AK (optimised) | $15T(n/4)+O(n)$ | $O(n^{1.95})$ |
| **TC (optimised)** | $x\,T(n/4)+O(n)$, need $\log_4 x<1.46\Rightarrow x=7$ | $7T(n/4)+O(n)=O(n^{1.40})$ |

**$K$-way split (general):**

| Method | Recurrence | Complexity |
|---|---|---|
| Plain D&C | $K^2\,T(n/K)+O(n)$ | $O(n^2)$ |
| AK | $(K^2-1)\,T(n/K)+O(n)$ | $O\!\big(n^{\log_K(K^2-1)}\big)$ |
| **Toom-Cook** | $(2K-1)\,T(n/K)+O(n)$ | $\mathbf{O\!\big(n^{\log_K(2K-1)}\big)}$ |

### 4.2 Master comparison of multiplication algorithms

| Algorithm | Split | Sub-mults | Complexity | Exponent |
|---|---|---|---|---|
| Traditional (school) | — | $n^2$ digit mults | $O(n^2)$ | 2 |
| **Karatsuba (Toom-2)** | 2 | 3 | $O(n^{\log_2 3})$ | ≈ 1.585 |
| **Toom-3** | 3 | 5 (instead of 9) | $O(n^{\log_3 5})$ | ≈ 1.465 |
| **Toom-$K$** | $K$ | $2K-1$ | $O(n^{\log_K(2K-1)})$ | → 1 as $K\to\infty$ 🆕 |

---

## 5. GATE PYQ — Divide & Conquer

**Q.** Consider the recurrence
$$T(n)=\begin{cases}T(n/2)+T(2n/5)+7n & n>0\\ 1 & n=0\end{cases}$$
Which one of the following is correct?

| Option | Statement | Verdict |
|---|---|---|
| A | $T(n)=\Theta(n^{5/2})$ | ❌ Far too large — work *shrinks* every level, it doesn't blow up |
| B | $T(n)=\Theta(n\log n)$ | ❌ $n\log n$ needs sub-problem sizes summing to exactly $n$ (balanced); here they sum to $<n$ |
| **C** | **$T(n)=\Theta(n)$** | ✅ **(derived)** |
| D | $T(n)=\Theta((\log n)^{5/2})$ | ❌ Sub-linear, but the root alone costs $7n$, so $T(n)=\Omega(n)$ |

**Reason:** Level ratio $=\tfrac12+\tfrac25=\tfrac{9}{10}<1$ → decreasing geometric series → total $\le 7n\cdot\dfrac{1}{1-0.9}=70n$ and $\ge 7n$ ⇒ $\Theta(n)$.

---

# PART 2 — HEAPS (Heap & Set Data Structures, Lecture 9)

## 6. Heap Basics & Array Representation

**Heap operations covered:** create · insert · delete · inc/dec key.

**Definition:** A **Heap** is a **Complete Binary Tree (CBT)** in which the value at each node is **at least as large as the values at its children** (Max-Heap). A **Min-Heap** is the mirror image (each node ≤ children).

- **CBT:** all levels full except possibly the last, which is filled **left to right**. Nodes are numbered in **level order** $1\ldots n$ ($n$ = number of nodes). Because of this, a CBT is stored in an array with **no gaps**.

| Min-Heap example | Max-Heap example |
|---|---|
| root 10; level 1: 15, 30; level 2: 40, 50, 100, 40 | root 100; level 1: 40, 50; level 2: 10, 15, 50, 40 |

### Index formulas (1-indexed — used for the algorithms)

| Quantity | Formula |
|---|---|
| Parent of node $i$ | $\lfloor i/2\rfloor$ |
| Left child | $2i$ |
| Right child | $2i+1$ |

Example (root = index 1): array `[_, 6, 4, 5, 3, 2, 0, 1]` (index 0 unused) → node 4 (value 3, index 4) is left child of index 2; node index 5 is right child of index 2, etc.

> ⚠️ **If the array is 0-indexed** (as in some GATE PYQs, root at index 0): parent $=\lfloor (i-1)/2\rfloor$, children $=2i+1,\;2i+2$, last internal node $=\lfloor (n-2)/2\rfloor$ 🆕.

---

## 7. Heap Creation: Insertion Method vs Heapify

Two ways to create a heap from $n$ elements:

| | **Insertion method** | **Heapify (Build-Heap)** |
|---|---|---|
| Idea | Insert elements one by one (top-down) | Fix subtrees bottom-up |
| Direction | Elements **sift up** | Elements **sift down** (ADJUST) |
| Start | $i=2$ to $n$ | $i=\lfloor n/2\rfloor$ down to 1 |
| Best case | $O(n)$ — input in **descending** order (for max-heap) | $O(n)$ always |
| Worst case | $O(n\log n)$ — input in **ascending** order | $O(n)$ |

### 7.1 INSERT (max-heap, sift-up)

```text
Procedure INSERT(A, n)
    integer i, j, n
    j ← n;  i ← ⌊n/2⌋;  item ← A(n)
    while (i > 0 and A(i) < item) do
        A(j) ← A(i)             // move the parent down
        j ← i;  i ← ⌊i/2⌋
    repeat
    A(j) ← item                 // a place for A(n) is found
end INSERT

// Build a heap by repeated insertion:
for i ← 2 to n do
    call INSERT(A, i)
repeat
```

**Example — inserting 90 as the 7th item** into heap `80 | 45 70 | 40 35 50` :
90 appended as child of 70 → swap with 70 → swap with 80 → root = 90.
Result: `90 | 45 80 | 40 35 50 70`.

**Example — forming a heap from (40, 80, 35, 90, 45, 50, 70) by insertion:**

| Insert | Tree after insertion (level order) |
|---|---|
| 40 | `40` |
| 80 | `80 40` (80 swaps above 40) |
| 35 | `80 40 35` |
| 90 | 90 lands under 40 → swaps with 40, then 80 → `90 80 35 40` |
| 45 | `90 80 35 40 45` |
| 50 | under 35 → swap with 35 → `90 80 50 40 45 35` |
| 70 | under 50 → swap with 50 → `90 80 70 40 45 35 50` |

Final array: **`[90, 80, 70, 40, 45, 35, 50]`**.

### 7.2 ADJUST (sift-down) and HEAPIFY (Build-Heap)

```text
procedure ADJUST(A, i, n)
    integer i, j, n
    j ← 2*i;  item ← A(i)
    while j ≤ n do
        if (j < n and A(j) < A(j+1)) then
            j ← j + 1                    // j points to the larger child
        end if
        if (item ≥ A(j)) then
            exit                          // a position for item is found
        else
            A(⌊j/2⌋) ← A(j)               // move the larger child up a level
            j ← 2*j
        end if
    repeat
    A(⌊j/2⌋) ← item
end ADJUST

procedure HEAPIFY(A, n)     /* Build-Heap */
    // Readjust the elements in A(1:n) to form a heap
    integer n, i
    for i ← ⌊n/2⌋ to 1 by −1 do
        call ADJUST(A, i, n)
    repeat
end HEAPIFY
```
> Fixes applied: slide had `j ≤ n and A(j) < A(j+1)` → should be `j < n` (needs a right sibling); `J ← 2*j` → `j ← 2*j`; "HEAP1FY" → "HEAPIFY".

**Worked example — HEAPIFY on (100, 119, 118, 171, 112, 151, 132)** (indices 1–7, start at $i=\lfloor 7/2\rfloor=3$):

| Step | Action | Array (index 1→7) |
|---|---|---|
| (i) initial | — | `100 119 118 171 112 151 132` |
| (ii) ADJUST($i=3$) | 118 vs children 151, 132 → swap with larger (151) | `100 119 151 171 112 118 132` |
| (iii) ADJUST($i=2$) | 119 vs children 171, 112 → swap with 171 | `100 171 151 119 112 118 132` |
| (iv) ADJUST($i=1$) | 100 vs 171, 151 → swap with 171; then 100 vs 119, 112 → swap with 119 | **`171 119 151 100 112 118 132`** ✅ |

---

## 8. Time Complexity of Build-Heap

Let the tree have $k$ levels with $n=2^k$ (levels numbered $1\ldots k$ from the top).

- Max # nodes at level $i$ $=2^{i-1}$
- Max # of level moves/comparisons when a node at level $i$ is adjusted $=(k-i)$
- Total over level $i$: $(k-i)\cdot 2^{i-1}$

$$T(n)=\sum_{i=1}^{k}(k-i)\,2^{\,i-1}=\tfrac12\Big[k\sum_{i=1}^{k}2^i-\sum_{i=1}^{k}i\,2^i\Big]$$

Using $\sum_{i=1}^{k}2^i=2^{k+1}-2$ and $\sum_{i=1}^{k}i2^i=(k-1)2^{k+1}+2$:

$$=\tfrac12\Big[k(2^{k+1}-2)-\big((k-1)2^{k+1}+2\big)\Big]=\tfrac12\big[2^{k+1}-2k-2\big]=2^k-k-1$$

$$=n-\log n-1=\boxed{O(n)}$$

> **Key insight:** most nodes sit near the bottom and move very few levels — so Build-Heap is **linear**, even though one ADJUST can cost $O(\log n)$.

---

## 9. Heap Sort

```text
procedure HEAPSORT(A, n)
    // A(1:n) contains n elements to be sorted
    1. call HEAPIFY(A, n)                        // builds Max-Heap, O(n)
    2. for i ← n to 2 by −1 do
           t ← A(i);  A(i) ← A(1);  A(1) ← t     // swap root (max) to the end
           call ADJUST(A, 1, i−1)                 // restore heap on remaining i−1 items, O(log n)
       repeat
end HEAPSORT
```

- Step 1: $O(n)$. Step 2: $(n-1)$ delete-max operations, each $O(\log n)$.
- $T(n)=O(n)+O(n\log n)=\mathbf{O(n\log n)}$.
- After each swap the largest element is fixed at its final position $i$ (sorted part grows from the right; heap part is positions $1..i-1$).
- Slide illustration: heap `112 | 151 132 | 119 130 100` with `171` already placed at the end.

---

## 10. Insert / Delete / Inc-Dec Key

| Operation | Best case | Worst case | Note |
|---|---|---|---|
| **Insert** one element (into existing heap) | $O(1)$ (new element already ≤ parent) | $O(\log n)$ (sifts up to the root) | Tree path length $=\log n$ |
| **Delete** (root) | — | $O(\log n)$ | Swap with last, ADJUST from root |
| **Increase / Decrease key** | — | $O(\log n)$ | Sift up (increase in max-heap) or sift down |
| **Build-Heap** (all $n$) | $O(n)$ | $O(n)$ | Heapify |
| **Insert $n$ elements one-by-one** | $O(n)$ | $O(n\log n)$ | Insertion method |

**Useful structure fact:** about **$n/2$ of the nodes are leaves** (the whole bottom level of a CBT holds roughly half the nodes).

---

## 11. GATE PYQs — Heaps

> Array questions below use 1-indexed positions unless stated.

### Q1. Which array representation is a valid binary Max-Heap?

| Opt | Array | Valid? | Why |
|---|---|---|---|
| A | `<25, 12, 16, 13, 10, 8, 14>` | ❌ | Node 12 (idx 2) has child 13 > 12 |
| **B** | **`<25, 14, 16, 13, 10, 8, 12>`** | ✅ | Every parent ≥ its children **(derived)** |
| C | `<25, 14, 13, 16, 10, 8, 12>` | ❌ | Node 14 (idx 2) has child 16 > 14 |
| D | `<25, 14, 12, 13, 10, 8, 16>` | ❌ | Node 12 (idx 3) has child 16 > 12 |

### Q2. Which one is a valid **3-ary** Maximum Heap array?
(3-ary, 1-indexed: children of $i$ are $3i-1,\;3i,\;3i+1$; parent of $i$ is $\lfloor (i+1)/3\rfloor$ — i.e. children of 1 are 2,3,4; children of 2 are 5,6,7.)

| Opt | Array | Valid? | Why |
|---|---|---|---|
| A | `<1, 3, 5, 6, 8, 9>` | ❌ | Root is the smallest |
| B | `<9, 6, 3, 1, 8, 5>` | ❌ | Node 6 (idx 2) has child 8 > 6 |
| C | `<9, 3, 6, 8, 5, 1>` | ❌ | Node 3 (idx 2) has child 5 > 3 |
| **D** | **`<9, 5, 6, 8, 3, 1>`** | ✅ | Root 9 ≥ 5,6,8; node 5 ≥ its children 3,1 **(derived)** |

### Q3. In a Binary Max-Heap with $n$ elements, the smallest element can be found in time of ______.
**Answer: $O(n)$ (lecture)** — the minimum is somewhere among the ~$n/2$ **leaves**, and there is no ordering among leaves, so all must be scanned. (Also equals the cost of converting to a Min-Heap via Build-Heap, $O(n)$.)

### Q4. Binary heap with $n$ elements; insert $n$ more elements (**not necessarily one after another**). Total time?

| Opt | Answer | Verdict |
|---|---|---|
| A | $O(n^2)$ | ❌ Far too loose |
| B | $n\log n$ | ❌ That's the cost of inserting one-by-one; not the best |
| **C** | **$n$** | ✅ **(lecture)** — append to get $2n$ elements, run **Heapify** in $O(n)$ |
| D | $n^2\log n$ | ❌ Far too loose |

### Q5. Binary Heap in an array with the smallest at the root; the **7th smallest** element can be found in time complexity of ______.
**Answer: $O(1)$ (lecture)** — the 7th smallest must lie within the top 7 levels ⇒ at most $2^7-1=127$ nodes to examine — a **constant independent of $n$**.

### Q6. Binary Heap in an array with $n$ elements; insert an element. If a **Binary Search** is performed along the path from the newly inserted element to the root, the number of comparisons is of the order of ______.
**Answer: $O(\log\log n)$ (lecture)** — the root-to-leaf path has $\log n$ elements (sorted along the path); binary search over $\log n$ items takes $\log(\log n)$ comparisons. (Note: the *moves* are still $O(\log n)$.)

### Q7. The approximate number of elements that can be sorted in $O(\log n)$ time using Heap Sort is ______.
Options: (a) $O(1)$ (b) $O(\log n)$ (c) $O(\sqrt n)$ (d) $O\!\left(\dfrac{\log n}{\log\log n}\right)$

**Answer: (d) (lecture)**. Sorting $m$ elements takes $m\log m$. Put $m=\dfrac{\log n}{\log\log n}$:

$$m\log m=\frac{\log n}{\log\log n}\big[\log\log n-\log\log\log n\big]=\log n-\frac{\log n}{\log\log n}\log\log\log n=O(\log n)$$

- (a) ✘ trivially too small; (b) $m=\log n$ ⇒ $\log n\cdot\log\log n$, exceeds $O(\log n)$; (c) $m=\sqrt n$ ⇒ $\sqrt n\log n$, way over.

### Q8. Given $\lceil\log n\rceil$ sorted lists, each having $\lfloor n/\log n\rfloor$ elements. Time to merge them into a single sorted list using a Heap is ______.
**Answer: $O(n\log\log n)$ (derived)** — total $\approx n$ elements; keep a **min-heap of size $k=\log n$** (one head per list); each of the $n$ extract-min + insert steps costs $O(\log k)=O(\log\log n)$.

### Q9. Minimum number of interchanges needed to convert the array `89, 19, 40, 17, 12, 10, 2, 5, 7, 11, 6, 9, 70` into a Max-Heap is

| Opt | Value | Verdict |
|---|---|---|
| A | 0 | ❌ Not already a heap — 70 (idx 13) > its parent 10 (idx 6) |
| B | 1 | ❌ One swap (10↔70) fixes idx 6 but then 70 > 40 (idx 3) |
| **C** | **2** | ✅ **(derived)** swap 10↔70, then 40↔70 |
| D | 3 | ❌ Not minimal |

**Check:** Only violation is at the right-most branch: idx 13 (70) under idx 6 (10) under idx 3 (40). Two swaps move 70 up to idx 3; 89 at root is still ≥ 70. Everything else already satisfies the heap property.

### Q10. An array of size $n$ is converted to a Heap by adjusting the heaps rooted at each internal node, starting at node $\lfloor (n-1)/2\rfloor$ and going up to the root (index 0), in order $\lfloor (n-1)/2\rfloor,\lfloor (n-3)/2\rfloor,\dots,0$. The time required is

| Opt | Answer | Verdict |
|---|---|---|
| A | $O(\log n)$ | ❌ Can't even touch all internal nodes |
| **B** | **$O(n)$** | ✅ This is Build-Heap / Heapify **(derived)** |
| C | $O(n\log\log n)$ | ❌ Valid upper bound but not tight |
| D | $O(n\log n)$ | ❌ Valid upper bound but not tight (that's the insertion method's worst case) |

### Q11. Array $X$ of $n$ distinct integers is a CBT (first index 0). If **only the root** violates the heap property, the best asymptotic algorithm to convert it into a heap runs in

| Opt | Answer | Verdict |
|---|---|---|
| A | $O(n)$ | ❌ Not best — one sift-down suffices |
| **B** | **$O(\log n)$** | ✅ A single ADJUST from the root: ≤ tree height **(derived)** |
| C | $O(n\log n)$ | ❌ Way too slow |
| D | $O(n\log\log n)$ | ❌ Way too slow |

### Q12. CBT whose **left and right subtrees of the root are Max-Heaps**. The **lower bound** for the number of operations to convert the tree to a heap is

| Opt | Answer | Verdict |
|---|---|---|
| **A** | **$\Omega(\log n)$** | ✅ **(derived)** worst case: root must sink along a root-to-leaf path of length $\log n$ |
| B | $\Omega(n)$ | ❌ False as a lower bound — an $O(\log n)$ algorithm exists |
| C | $\Omega(n\log n)$ | ❌ Same reason |
| D | $\Omega(n^2)$ | ❌ Same reason |

### Q13. Max-Heap array `40, 30, 20, 10, 15, 16, 17, 8, 4` (indices 1–9). Insert **35**. The new Heap is

| Opt | Array | Verdict |
|---|---|---|
| A | `40,30,20,10,15,16,17,8,4,35` | ❌ Just appended — 35 > parent 15 violates heap property |
| **B** | **`40,35,20,10,30,16,17,8,4,15`** | ✅ **(derived)** |
| C | `40,30,20,10,35,16,17,8,4,15` | ❌ Stopped after one swap — 35 still > parent 30 |
| D | `40,35,20,10,15,16,17,8,4,30` | ❌ Wrong element ended up at the end |

**Trace:** 35 placed at idx 10 (parent idx 5 = 15) → swap → idx 5 (parent idx 2 = 30) → swap → idx 2 (parent idx 1 = 40 ≥ 35) → stop.

### Q14. The number of possible **Min-Heaps** containing each value from $\{1,2,3,4,5,6,7\}$ exactly once is ______. ★
**Answer: 80 (derived)** — a 7-node CBT has root + two 3-node subtrees.
- Root must be 1.
- Choose which 3 of the remaining 6 values go to the left subtree: $\binom{6}{3}=20$.
- A 3-node Min-Heap on 3 given values: smallest is root, the other two can be arranged in $2$ ways ⇒ $h(3)=2$.
- Total $=\binom63\cdot h(3)\cdot h(3)=20\cdot2\cdot2=\mathbf{80}$.

### Q15. Consider the statements:
I. The smallest element in a Max-Heap is always at a leaf node.
II. The second largest element in a Max-Heap is always a child of the root node.
III. A Max-Heap can be constructed from a BST in $O(n)$ time.
IV. A BST can be constructed from a Max-Heap in $O(n)$ time.

| Opt | Statements | Verdict |
|---|---|---|
| A | I, III and IV | ❌ IV is false |
| B | II, III and IV | ❌ IV is false |
| **C** | **I, II and III** | ✅ **(lecture: I ✔, II ✔, III ✔, IV ✘)** |
| D | I, II and IV | ❌ IV is false |

**Why:** (I) any non-leaf has a smaller child below it, so the minimum can't be internal. (II) the 2nd largest must be a child of the max (root). (III) BST in-order traversal gives a sorted array in $O(n)$ (or just run Build-Heap, $O(n)$). (IV) building a BST from a heap needs comparison-based ordering — $\Omega(n\log n)$.

### Q16. $H$ is a Binary **Min-Heap** with $n$ elements as an Array. Worst-case time of an optimal algorithm to find the **maximum** element in $H$?

| Opt | Answer | Verdict |
|---|---|---|
| A | $\Theta(\log n)$ | ❌ Max isn't on any single path |
| B | $\Theta(n\log n)$ | ❌ Needless overhead |
| **C** | **$\Theta(n)$** | ✅ **(derived)** max is one of the ~$n/2$ leaves; no order among them ⇒ scan all leaves |
| D | $\Theta(1)$ | ❌ That's for the **minimum** (root) |

---

# Memory Tricks / Mnemonics

- **"Karatsuba = Toom-2; Toom-3 is just one instance of Toom-Cook."** *(lecture)* — don't use "Toom-3" and "Toom-Cook" interchangeably.
- **Mult-count ladder:** 4 → 3 (Karatsuba), 9 → 5 (Toom-3), generally $K^2 \to 2K-1$ (Toom-$K$). *(lecture)*
- **Exponent ladder:** 2 → 1.585 → 1.465 (→ 1.40 for a 4-way split with 7 mults). *(lecture)*
- **Unbalanced recursion tree → sandwich it:** *shortest path* gives the **lower** bound ($\Omega$), *longest path* gives the **upper** bound ($O$); if both give $n\log n$ ⇒ $\Theta$. *(lecture)*
- **Generalised form:** $T(\alpha n)+T((1-\alpha)n)+O(n)=\Theta(n\log n)$ — "sizes add to $n$ ⇒ every level costs $n$". *(lecture)*
- **Decreasing geometric ratio ⇒ root dominates** (e.g., $\tfrac3{16}$, $\tfrac5{16}$, $\tfrac9{10}$). 🆕
- **Heap parent/child:** parent $\lfloor i/2\rfloor$, children $2i,\,2i+1$ (1-indexed). *(lecture)*
- **Heapify runs bottom-up from $\lfloor n/2\rfloor$ to 1; insertion builds top-down** — bottom-up is linear because most nodes are near the leaves. 🆕
- **Insertion-method extremes:** *descending* input → $O(n)$ (best), *ascending* input → $O(n\log n)$ (worst) for a Max-Heap. *(lecture)*
- **Half of a heap's nodes are leaves ⇒ min of a Max-Heap / max of a Min-Heap costs $\Theta(n)$.** *(lecture)*
- **Heapsort = Heapify $O(n)$ + $(n-1)\times$ Delete $O(\log n)$.** *(lecture)*

---

# Quick Revision Checklist

**Recursion Tree Method**
- [ ] 5 steps: recurrence → root cost → $a^i$ nodes of size $n/b^i$ → level cost $a^if(n/b^i)$ → depth $\log_b n$, sum levels
- [ ] Merge-sort tree: $\log n+1$ levels × $cn$ = $cn\log n+cn$
- [ ] $2T(n/2)+n\to\Theta(n\log n)$
- [ ] Unbalanced trees $T(n/3)+T(2n/3)+n$, $T(n/5)+T(4n/5)+n$: use shortest path ($\Omega$) & longest path ($O$) ⇒ $\Theta(n\log n)$
- [ ] Generalised form with $0<\alpha<1$ ⇒ $\Theta(n\log n)$
- [ ] $T(n)=T(n-1)+T(n-2)+c$: $O(2^n)$, $\Omega(2^{n/2})$, **not** $\Theta(2^n)$; why $2^n\ne\Theta(2^{n/2})$
- [ ] $3T(n/4)+n^2$ (CLRS) ⇒ $\Theta(n^2)$; geometric series $\tfrac1{1-x}$; leaves $n^{\log_4 3}$
- [ ] $T(n/4)+T(n/2)+n^2$ ⇒ $\Theta(n^2)$ (ratio $\tfrac5{16}$)
- [ ] Practice: $T(n/2)+T(n/4)+n\to\Theta(n)$; with $n^2\to\Theta(n^2)$; $T(n/3)+T(2n/3)+n^2\to\Theta(n^2)$
- [ ] PYQ $T(n/2)+T(2n/5)+7n\to\Theta(n)$

**Long Integer Multiplication**
- [ ] Why $u\pm v$, $u*v$ are $O(1)$ only for word-size ints; H/W fix not viable; S/W array representation
- [ ] Array add $O(n)$; school multiply $O(n^2)$
- [ ] Split $u=w10^m+x$, $v=y10^m+z$, $m=\lfloor n/2\rfloor$; $w,x,y,z$ via `/` and `%`
- [ ] $uv=wy10^{2m}+(wz+xy)10^m+xz$; $T(n)=4T(n/2)+bn\to O(n^2)$

**Karatsuba**
- [ ] History: 1960 / published 1962 (Anatoly Karatsuba)
- [ ] $P_{r1}=wy$, $P_{r2}=xz$, $P_{r3}=(w+x)(y+z)$; middle term $=P_{r3}-(P_{r1}+P_{r2})$
- [ ] $T(n)=3T(n/2)+bn\to O(n^{\log_2 3})\approx n^{1.58}$

**Toom-Cook**
- [ ] Toom (b.1942) & Cook (b.1939); Toom-3 is $k=3$ instance; Karatsuba = Toom-2
- [ ] Toom-3: 9 → 5 mults, $n^{\log_3 5}\approx n^{1.46}$
- [ ] Toom-$k$: $(2k-1)T(n/k)$ → $n^{\log_k(2k-1)}$
- [ ] 3-way table: 9T→$n^2$, 8T→$n^{\log_3 8}$, 5T→$n^{1.46}$ ($x=5$)
- [ ] 4-way table: 16T→$n^2$, 15T→$n^{1.95}$, 7T→$n^{1.40}$ ($x=7$)
- [ ] $K$-way: $K^2$, $K^2-1$, $2K-1$ sub-multiplications

**Heaps**
- [ ] Definition: CBT + (max-heap) parent ≥ children; min-heap mirror
- [ ] Level-order numbering; array indices (1-indexed: $\lfloor i/2\rfloor$, $2i$, $2i+1$; 0-indexed variants)
- [ ] Two creation methods: insertion vs heapify
- [ ] INSERT pseudocode (sift-up); example: inserting 90 as 7th element; heap from (40,80,35,90,45,50,70)
- [ ] Insertion method: best $O(n)$ (descending input), worst $O(n\log n)$ (ascending input)
- [ ] ADJUST (sift-down) & HEAPIFY pseudocode; trace on (100,119,118,171,112,151,132)
- [ ] Build-Heap complexity derivation $\sum (k-i)2^{i-1}=2^k-k-1=O(n)$
- [ ] HEAPSORT: heapify $O(n)$ + $(n-1)$ delete $O(\log n)$ ⇒ $O(n\log n)$
- [ ] Insert: best $O(1)$, worst $O(\log n)$; Inc/Dec key: $O(\log n)$; Delete $O(\log n)$
- [ ] Half the nodes are leaves ($n/2$)
- [ ] PYQs: valid max-heap array · valid 3-ary max-heap · min of max-heap $O(n)$ · insert $n$ more $O(n)$ · 7th smallest $O(1)$ · binary search on path $O(\log\log n)$ · sort $\tfrac{\log n}{\log\log n}$ elements in $O(\log n)$ · merge $\log n$ lists $O(n\log\log n)$ · min interchanges = 2 · heapify loop $O(n)$ · only-root violation $O(\log n)$ · root-subtrees-heaps $\Omega(\log n)$ · insert 35 trace · # min-heaps of 7 values = 80 · BST↔heap statements (I, II, III) · max of min-heap $\Theta(n)$

**Not in this PDF (still to cover):** Set operations, Find and Create (Union-Find), Control Abstraction, Max-Min, Binary Search, Merge Sort (D&C agenda items).
