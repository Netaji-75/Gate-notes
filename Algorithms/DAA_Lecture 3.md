# Design & Analysis of Algorithms: Analysis of Algorithms (Lecture 03)

> **Source:** GeeksforGeeks GATE, DAA Lecture 03. **Exam:** GATE CS/IT.
> **Legend:** ✔ **(lecture)** = answer marked/worked in the lecture. ✔ **(solved)** = slide had no answer, so I worked it out. 🔹 = my added tip, not from the lecture.

---

## Table of Contents

1. [Space Complexity](#1-space-complexity)
2. [Asymptotic Notations: Quick Reference](#2-asymptotic-notations-quick-reference)
3. [True/False Drill (Apriori Analysis)](#3-truefalse-drill-apriori-analysis)
4. [Comparing & Ordering Functions](#4-comparing--ordering-functions)
5. [GATE PYQs on Asymptotics](#5-gate-pyqs-on-asymptotics)
6. [Loop / Non-Recursive Analysis: Leader in an Array](#6-loop--non-recursive-analysis-leader-in-an-array)
7. [Binary Tree Height Problems](#7-binary-tree-height-problems)
8. [Data-Structure Operation Cost Problems (Queue, DLL, Heap)](#8-data-structure-operation-cost-problems)
9. [Recursive Algorithms: Framework & Recurrences](#9-recursive-algorithms-framework--recurrences)
10. [Memory Tricks](#10-memory-tricks--mnemonics)
11. [Quick Revision Checklist](#11-quick-revision-checklist)

**Lecture agenda:** Asymptotic Notations → Recursive Algorithms → Loop Analysis → Space Complexity.

---

## 1. Space Complexity

### Core idea
A **program (algorithm)** = **Instructions** + **Data (Input)**.
- Space for **instructions** is **constant**, so ignore it.
- Space for **input data** is **excluded** by definition.

> **Space used by an algorithm** = number of **memory cells (words)** needed to carry out the computational steps for one instance of the problem, **excluding the space that holds the input**.
> In other words, it is only the **Work Space** (auxiliary space).

### Key results
$$S(n) = O(T(n))$$

| Symbol | Meaning |
|---|---|
| $T(n)$ | time complexity of the algorithm |
| $S(n)$ | space complexity of the algorithm |

- **Why?** Writing into each memory cell takes at least constant time, so work space can never exceed running time.
- All **order-of-growth / asymptotic-bound definitions** for time ($f(n)$, O, Ω, Θ) **carry over** to space.

### Worked examples

| # | Algorithm | What it does | Time | Space | Reason for space |
|---|---|---|---|---|---|
| 1 | `SWAP(x, y)` | swap via temp `t` | O(1) | **O(1)** | only one extra variable `t` |
| 2 | `LS(A, n, x)` (Linear Search) | scan array for `x` | O(n) | **O(1)** | only `i` (and result) used |
| 3 | `SumSq(A, n)` | `B[i] = A[i]*A[i]` | O(n) | **O(n)** | **extra array `B[1..n]`** is work space |
| 4 | `MERGE` (two sorted arrays) | merge into aux array | Θ(n) | **Θ(n)** | aux storage of size exactly n |
| 5 | Binary Search, Selection Sort, Insertion Sort | | n/a | **Θ(1)** | in-place, only loop variables |

**Code (from the lecture, cleaned up):**

```c
// 1) SWAP: Time O(1), Space O(1)
Algorithm SWAP(int x, int y)
{
    int t;
    t = x;
    x = y;
    y = t;
}
```

```c
// 2) Linear Search: Time O(n), Space O(1)
Algorithm LS(A, n, x)
{
    int n, i, A[n];
    for i <- 1 to n
        if (x == A[i])
            return(i);
    return(0);
}
```

```c
// 3) SumSq: Time O(n), Space O(n)  (array B is extra)
Algorithm SumSq(A, n)
{
    int n, A[1..n], B[1..n], i;
    for i <- 1 to n
        B[i] = A[i] * A[i];
}
```

> **Rule of thumb:** Count only the *extra* memory (new arrays, recursion stack). Input arrays and their size don't count.

---

## 2. Asymptotic Notations: Quick Reference

| Notation | Meaning | Intuition |
|---|---|---|
| $f = O(g)$ | $f$ grows **no faster** than $g$ | upper bound, $f \le g$ |
| $f = \Omega(g)$ | $f$ grows **at least as fast** as $g$ | lower bound, $f \ge g$ |
| $f = \Theta(g)$ | both O and Ω | tight bound, $f \approx g$ |
| $f = o(g)$ | $f$ grows **strictly slower** than $g$ | $f < g$ |

**Facts used in the lecture:**
- Base of a log is **trivial** in asymptotics: $\log_2 n = \Theta(\log_{10} n)$ (they differ by a constant factor).
- Base of an *exponent* is **not** trivial: $2^n \neq \Theta(3^n)$.
- Exponent of a polynomial is **not** trivial: $n^2 \neq \Theta(n^5)$.
- $\log_b a = \dfrac{\log_c a}{\log_c b}$ (change of base).

---

## 3. True/False Drill (Apriori Analysis)

*(Slides gave no answers. All solved by me.)*

| # | Statement | T/F | Reason |
|---|---|---|---|
| 1 | $100\,n\log n = O(n\log n)$ | **True** | constants are dropped |
| 2 | $2^{n+1} = O(2^n)$ | **True** | $2^{n+1} = 2\cdot 2^n$, constant factor |
| 3 | $2^{2n} = O(2^n)$ | **False** | $2^{2n} = (2^n)^2 = 4^n$, not a constant multiple of $2^n$ |
| 4 | $0<x<y \Rightarrow n^x = O(n^y)$ | **True** | smaller power ≤ bigger power |
| 5 | $(n+k)^m \neq \Theta(n^m)$, $k,m>0$ | **False** | it **is** $\Theta(n^m)$, so "≠" is wrong |
| 6 | $\sqrt{\log n} = O(\log\log n)$ | **False** | let $L=\log n$: $\sqrt{L}$ grows faster than $\log L$ |
| 7 | $\log n$ is $\Omega(1/n)$ | **True** | $1/n \to 0$, $\log n$ grows |
| 8 | $2^{n^2}$ is $O(n!)$ | **False** | $n! \approx 2^{n\log n} \ll 2^{n^2}$ |
| 9 | $n^2$ is $O(2^{2\log n})$ | **True** | $2^{2\log_2 n} = n^2$ |
| 10 | $a^n \neq O(n^x)$, $a>1, x>0$ | **True** | exponential is never bounded by a polynomial |
| 11 | $2^{\log_2 n^2}$ is $O(n^2)$ | **True** | $2^{\log_2 n^2} = n^2$ |

> *Slide 8 was cut off in the PDF ("O(n!…"). I assumed $O(n!)$.*

---

## 4. Comparing & Ordering Functions

### 4.1 Standard growth hierarchy

$$1 < \log\log n < \sqrt{\log n} < \log n < (\log n)^k < \sqrt{n} < n < n\log n < n^2 < n^3 < n^{\log n} < 2^n < 4^n < n! < n^n < 2^{2^n}$$

**Lecture's shorthand:** label each function **P** (Polynomial) or **E** (Exponential). **P < E** always.

### 4.2 Technique: factor out the common part

**Example:** $f(n) = n^2\log n$ vs $g(n) = n\log^{10} n$

- $f = (n\log n)\cdot n$
- $g = (n\log n)\cdot(\log n)^9$
- Compare **$n$ vs $(\log n)^9$**. Take logs: $\log n$ vs $9\log\log n$, and $\log n$ is bigger.
- ∴ $f > g$, so **$g = O(f)$** and **$f = \Omega(g)$**.

### 4.3 Technique: take log of both sides (for exponentials)

For $f_1 = 10^n$, $f_2 = n^{\log n}$, $f_3 = (\sqrt{n})^n$:

| Function | $\log f$ | Order |
|---|---|---|
| $f_1 = 10^n$ | $n\log 10 = \Theta(n)$ | middle |
| $f_2 = n^{\log n}$ | $(\log n)^2$ | smallest |
| $f_3 = (\sqrt n)^n$ | $\tfrac{n}{2}\log n$ | largest |

→ $f_2 < f_1 < f_3$

### 4.4 Worked orderings

**(a)** $2^n;\ n^{3/2};\ n\log n;\ n^{\log n}$ ($f_1..f_4$) ✔ **(lecture)**

$$f_3 < f_2 < f_4 < f_1 \quad\Longleftrightarrow\quad n\log n < n^{3/2} < n^{\log n} < 2^n$$

- Polynomials first: $n^{1.5} = n\sqrt n$ vs $n\log n$ → $\sqrt n > \log n$, so $n\log n < n^{3/2}$.
- $n^{\log n}$ vs $2^n$: log both → $(\log n)^2$ vs $n$ → $n$ wins → $2^n$ is bigger.

**(b)** $\log n;\ \sqrt n;\ n;\ n\log n;\ \sqrt{\log n};\ n^2$ ✔ **(lecture)**

$$\sqrt{\log n} < \log n < \sqrt n < n < n\log n < n^2$$

**(c)** $n^{1/3};\ e^n;\ n^{7/4};\ n\log^9 n;\ 1.001^n$ ($f_1..f_5$) ✔ **(lecture)**

$$f_1 < f_4 < f_3 < f_5 < f_2 \quad\Longleftrightarrow\quad n^{1/3} < n\log^9 n < n^{7/4} < 1.001^n < e^n$$

- $n^{7/4}$ vs $n\log^9 n$ → $n^{3/4}$ vs $\log^9 n$ → polynomial wins.
- Any exponential with base > 1 beats every polynomial.

**(d)** $\log_2 n;\ \log_{10} n;\ \log_2\log_2 n;\ (\log_2\log_2 n)^{10}$ ($f_1..f_4$) ✔ **(lecture)**

$$f_3 < f_4 < f_1 \sim f_2$$

($f_1 \sim f_2$ = same order, since log bases differ by a constant.)

**(e)** $2^{2^n};\ n!;\ 4^n;\ 2^n$ ✔ **(solved)**

$$2^n < 4^n < n! < 2^{2^n}$$

**(f)** $2^{\log n};\ (\log n)^2;\ \sqrt{\log n};\ \log\log n$ ✔ **(solved)**

$$\log\log n < \sqrt{\log n} < (\log n)^2 < 2^{\log n} = n$$

### 4.5 Log-base value vs order

> **In terms of value**, $\log_2 n > \log_{10} n$, because $\log_{10} n = \dfrac{\log_2 n}{\log_2 10} = \dfrac{\log_2 n}{3.33}$.
> **In terms of asymptotic order**, they are the **same**: $\Theta(\log n)$.

### 4.6 Piecewise-function trap ⭐ (starred in lecture)

$$f(n)=\begin{cases} n^3, & 0<n\le 10{,}000\\ n, & n>10{,}000\end{cases}\qquad g(n)=\begin{cases} n, & 0<n\le 100\\ n^3, & n>100\end{cases}$$

- Asymptotics only care about **large n** (beyond some $n_0$).
- For $n > 100$: $g(n) = n^3 \ge f(n)$ (equal till 10,000, then $n^3 > n$).
- ∴ **$f(n) = O(g(n))$** ✔ with $n_0 = 100$. The reverse **$g = O(f)$ is false**.

---

## 5. GATE PYQs on Asymptotics

### PYQ 1: Sum of cubes
$f(n) = \sum_{i=1}^{n} i^3 = x$. Choices for $x$:
**I.** $\Theta(n^4)$ **II.** $\Theta(n^5)$ **III.** $O(n^5)$ **IV.** $\Omega(n^3)$

- (A) I, II and IV
- (B) I and III
- (C) I and IV
- **(D) I, III and IV** ✔ **(solved)**

**Why:** $\sum i^3 = \left(\frac{n(n+1)}{2}\right)^2 = \Theta(n^4)$. So I ✔; II ✗ (too big to be tight); III ✔ (loose upper bound); IV ✔ (loose lower bound).
- (A) ✗ includes II. (B) ✗ misses IV. (C) ✗ misses III.

---

### PYQ 2: Polynomial vs exponential base
$f(n) = 2^n$; $g(n) = n^n$

- **(A) $f = O(g)$** ✔ **(lecture)**
- (B) $f = \Omega(g)$
- (C) $f = \Theta(g)$
- (D) None of these

**Why wrong:** $n^n$ grows strictly faster than $2^n$, so $f$ is **not** Ω(g) and **not** Θ(g). (D) ✗ because (A) holds.

---

### PYQ 3: Log bases ⭐
$f(n) = \log_2 n$; $g(n) = \log_{10} n$

- (A) $f = O(g)$
- (B) $f = \Omega(g)$
- **(C) $f = \Theta(g)$** ✔ **(lecture)**
- (D) None of these

**Why:** base is trivial in ASN, both are $\Theta(\log n)$. (A) and (B) are *also technically true* (Θ implies both), but (C) is the tightest and intended answer. (D) ✗.

---

### PYQ 4: Log in the exponent
$f(n) = n^{\log_2 n}$; $g(n) = n^{\log_{10} n}$

- (A) $f = O(g)$
- **(B) $f = \Omega(g)$** ✔ **(lecture)**
- (C) $f = \Theta(g)$
- (D) None of these

**Why:** $\log_2 n \approx 3.33\log_{10} n$, so $f = g^{3.33}$, a **polynomially larger** function. (A) ✗ f outgrows g. (C) ✗ not same order. (D) ✗ (B) is true.
**Lecture analogy:** $2$ vs $5$ → $\Theta$ (constants). But $n^2$ vs $n^5$ → only $O$. A constant in the **exponent** changes the order.

---

### PYQ 5: Order of 5 functions
Arrange in increasing order of asymptotic complexity: $10,\ \sqrt n,\ n,\ \log_2 n,\ \dfrac{100}{n}$

- (A) $\log_2 n,\ \tfrac{100}{n},\ 10,\ \sqrt n,\ n$
- **(B) $\tfrac{100}{n},\ 10,\ \log_2 n,\ \sqrt n,\ n$** ✔ **(lecture)**
- (C) $10,\ \tfrac{100}{n},\ \sqrt n,\ \log_2 n,\ n$
- (D) $\tfrac{100}{n},\ \log_2 n,\ 10,\ \sqrt n,\ n$

**Why:** $100/n \to 0$ (smallest) < constant 10 < $\log n$ < $\sqrt n$ < $n$.
(A) ✗ puts $\log n$ before $100/n$ (which decays to 0). (C) ✗ puts 10 before $100/n$ and $\sqrt n$ before $\log n$. (D) ✗ puts $\log n$ before the constant 10.

---

### PYQ 6: Three exponentials
$f_1 = 10^n$, $f_2 = n^{\log n}$, $f_3 = (\sqrt n)^n$. Increasing order of growth?

- (A) $f_1, f_2, f_3$
- (B) $f_3, f_2, f_1$
- (C) $f_2, f_3, f_1$
- **(D) $f_2, f_1, f_3$** ✔ **(lecture)**

**Why:** log of each: $(\log n)^2 < n\log 10 < \tfrac n2\log n$. (A) ✗ wrongly puts $n^{\log n}$ above $10^n$. (B) ✗ exactly reversed. (C) ✗ puts $(\sqrt n)^n$ below $10^n$, but once $\sqrt n > 10$ it overtakes.

---

### PYQ 7: Relations among f, g, h ⭐
Given: $f = O(g)$, $g \ne O(f)$ **⟹ $f < g$**; and $g = O(h)$, $h = O(g)$ **⟹ $g = \Theta(h)$**. So **$f < g = h$**.

Which of the following is **TRUE**? *(lecture marks the false one as D)*

- **(A)** $f(n)$ is $O(h(n))$ ✔ true
- **(B)** $f(n) + h(n)$ is $O(g(n) + h(n))$ ✔ true
- **(C)** $h(n) \ne O(f(n))$ ✔ true
- **(D)** $f(n)\cdot g(n) \ne O(g(n)\cdot h(n))$ ✗ **FALSE**

**Why D is false:** $f<g$ and $g=h$ give $f\cdot g < g\cdot h$, so $f g = O(gh)$. The "≠" claim is wrong.
*Note: the slide is annotated "FALSE"/"are", so the intended answer is the one **false** statement, **D**.*

---

### PYQ 8: GATE 2015 (2M) ⭐
$f(n) = n$ and $g(n) = n^{(1+\sin n)}$, $n$ a positive integer. Which is/are correct?
**I:** $f(n) = O(g(n))$ **II:** $f(n) = \Omega(g(n))$

- (A) Only I
- (B) Only II
- (C) Both I and II
- **(D) Neither I nor II** ✔ **(lecture)**

**Why:** $\sin n$ oscillates in $[-1,1]$, so $g$ swings between about $n^0 = 1$ and $n^2$ **infinitely often**.
- When $\sin n \approx 1$: $g \approx n^2 > f$, so $f = \Omega(g)$ fails.
- When $\sin n \approx -1$: $g \approx 1 < f$, so $f = O(g)$ fails.
- No single constant works for all large n → neither holds.

---

### PYQ 9: GATE 2023 CS (1M, MSQ)
$f(n) = n$ and $g(n) = n^2$. Which are true?

- **(A) $f \in O(g)$** ✔ **(lecture)**
- (B) $f \in \Omega(g)$
- **(C) $f \in o(g)$** ✔ **(lecture)**
- (D) $f \in \Theta(g)$

**Why wrong:** $f < g$ strictly. (B) ✗ f is not at least as fast as g. (D) ✗ not tight (different powers).

---

### PYQ 10: Which is TRUE for all positive functions f(n)? ⭐

- **(A) $f(n^2) = \Theta(f(n)^2)$, when $f(n)$ is a polynomial** ✔ **(lecture)**
- (B) $f(n^2) = o(f(n)^2)$
- (C) $f(n^2) = O(f(n)^2)$, when $f(n)$ is an exponential function
- (D) $f(n^2) = \Omega(f(n)^2)$

**Why:**
- (A) ✔ e.g. $f = n^3$: $f(n^2) = n^6$ and $f(n)^2 = n^6$ → equal order.
- (B) ✗ same example: $n^6 \ne o(n^6)$.
- (C) ✗ $f = 2^n$: $f(n^2) = 2^{n^2}$ but $f(n)^2 = 4^n$; $2^{n^2} \neq O(4^n)$.
- (D) ✗ $f = \log n$: $f(n^2) = 2\log n$ but $f(n)^2 = (\log n)^2$; $2\log n \neq \Omega(\log^2 n)$.

---

### PYQ 11: Package A vs Package B ⭐ (numerical)
DB has $10^x$ records. Package **A** takes $10\,n\log n$; package **B** takes $0.0001\,n^2$ for $n$ records. Find the **smallest integer $x$** for which A outperforms B.

**Setup:** $n = 10^x$, logs are base 10 so $\log_{10} n = x$.

**Trial values:**

| x | n | Time A = 10·n·log n | Time B = 10⁻⁴·n² | Winner |
|---|---|---|---|---|
| 1 | 10 | $10\cdot10\cdot1 = 100$ | $10^{-4}\cdot10^2 = 10^{-2}$ | B |
| 2 | 100 | $10\cdot100\cdot2 = 2000$ | $10^{-4}\cdot10^4 = 1$ | B |

**General solution:** need $T(A) < T(B)$

$$10\cdot 10^x \cdot x \;<\; 10^{-4}\cdot 10^{2x}$$

$$x \;<\; \frac{10^x}{10^5}$$

| x | $10^x/10^5$ | $x <$ that? |
|---|---|---|
| 5 | 1 | $5<1$ ✗ |
| 6 | 10 | $6<10$ ✔ |

**Answer: x = 6** ✔ **(lecture)**

---

## 6. Loop / Non-Recursive Analysis: Leader in an Array

**Problem:** An element is a **Leader** if it is **greater than all elements to its right**. Find the time complexity of the **most efficient** algorithm to print all leaders in an array of size $n$.

**Example:** `A = [35, 6, 20, 15, 5, 10, 8]` (indices 1..7). Leaders marked in the lecture: **35, 20, 15, 10**.
🔹 *By definition the last element is vacuously a leader. The lecture's code loops over `i = 1..n-1` only, so add `print(A[n])` if your question counts it.*

### Version 1: Brute force

```c
Algorithm LEADER_V1(A, n)
{
    int n, A[n], i, j;
    for i <- 1 to (n-1) {
        for j <- (i+1) to n {
            if (A[i] < A[j])
                break;
        }
        if (j == (n+1))
            print(A[i]);
    }
}
```

| Case | Input order | Work | Time |
|---|---|---|---|
| **Best** | Increasing | inner loop breaks immediately each time | **O(n)** |
| **Worst** | Decreasing | $(n-1)+(n-2)+\dots+1 = \dfrac{n(n-1)}{2}$ | **O(n²)** |

**Space:** O(1)

### Version 2: Linear scan from Right → Left (optimal)

```c
Algorithm LEADER_V2(A, n)
{
    int n, A[n], i, L;
    L <- A[n];                       // current max seen from the right
    for i <- (n-1) downto 1 {
        if (A[i] > L) {
            print(A[i]);
            L <- A[i];               // update running max
        }
    }
}
```

- Single pass → **Time O(n)** (always), **Space O(1)**.
- **Answer:** **Θ(n)** (O(n)) ✔ **(solved)**; the slide left the blank empty.

| | V1 (Brute force) | V2 (Right-to-left scan) |
|---|---|---|
| Idea | compare each element to all on its right | keep running max from the right |
| Best case | O(n) | O(n) |
| Worst case | **O(n²)** | **O(n)** |
| Space | O(1) | O(1) |

---

## 7. Binary Tree Height Problems

### Q (TIFR): Tree where level $i$ has exactly $i$ nodes
Root at level 1, level $i$ has exactly $i$ nodes. Height $h$ of such a tree with $n$ nodes is of order ___?

**General bounds for any binary tree with n nodes:**
$$\underbrace{\log n}_{\text{lower bound (LB)}} \;\le\; h \;\le\; \underbrace{n}_{\text{upper bound (UB)}}$$

**Solution:**

$$n = \sum_{i=1}^{h} i = \frac{h(h+1)}{2} \;\Rightarrow\; h^2 + h = 2n \;\Rightarrow\; h^2 \sim n \;\Rightarrow\; h \sim \sqrt n$$

**Answer: $O(\sqrt n)$** ✔ **(lecture)**

### Companion: Complete (full) binary tree
- **Max nodes at level $i$ = $2^{i-1}$.**

$$n = \sum_{i=1}^{h} 2^{i-1} = \tfrac12\sum_{i=1}^{h}2^i = \tfrac12\left(2^{h+1}-2\right) = 2^h - 1$$

$$n = 2^h - 1 \;\Rightarrow\; h = \log_2(n+1) = O(\log n)$$

- **Skewed tree** (one node per level, e.g. $h=4$): $h = n$.

| Tree shape | Nodes at level i | Height |
|---|---|---|
| Skewed (chain) | 1 | $h = n$ (worst, UB) |
| "Level i has i nodes" | $i$ | $h = O(\sqrt n)$ |
| Complete / perfect | $2^{i-1}$ | $h = \log_2(n+1) = O(\log n)$ (best, LB) |

---

## 8. Data-Structure Operation Cost Problems

### 8.1 MultiQueue: amortized analysis
`k` is a global parameter. Operations: EnQueue, DeQueue, and:

```c
MultiQueue(Q)
{
    m = k;
    while ((Q is not empty) and (m > 0))
    {
        DeQueue(Q);
        m = m - 1;
    }
}
```

**Q:** Worst-case time of a sequence of **n** queue operations on an initially empty queue?

- **(A) O(n)** ✔ **(lecture)**
- (B) O(n + k)
- (C) O(n·k)
- (D) O(n²)

**Why:** Each element is enqueued once and dequeued at most once, so the **total number of dequeues ≤ total enqueues ≤ n**, regardless of how many MultiQueue calls are made. Total work is O(n).
(B), (C), (D) are valid-but-loose upper bounds; (A) is the tightest.

---

### 8.2 Array-based queue
A queue is implemented using an array so that EnQueue/DeQueue are performed **efficiently**. Which is correct for a queue of size n?

- **(A) Both operations can be performed in O(1)** ✔ **(lecture)**
- (B) Worst complexity of both operations will be Ω(n)
- (C) Worst-case time of both operations will be Ω(log n)
- (D) At least one operation can be performed in O(1) and the worst-case time for the other will be Ω(n)

**Why:** A **circular array** with `front` and `rear` pointers gives O(1) enqueue and dequeue. (B), (C), (D) all claim some operation must cost Ω(n) or Ω(log n), which is false for the efficient implementation.

---

### 8.3 Sorted doubly linked list: total cost
$N$ items in a **sorted DLL**. For delete and decrease-key a **pointer to the record** is provided. Algorithm performs in order: **Θ(N) delete, O(log N) insert, O(log N) find, Θ(N) decrease-key** operations. Total time?

- (A) $O(\log^2 N)$
- (B) $O(N)$
- **(C) $O(N^2)$** ✔ **(lecture)**
- (D) $\Theta(N^2\log N)$

**Cost per operation in a sorted DLL (N items):**

| Operation | # of ops | Cost each | Subtotal |
|---|---|---|---|
| Delete (pointer given) | Θ(N) | O(1) | O(N) |
| Insert (must find position) | O(log N) | O(N) | O(N log N) |
| Find (linear traversal) | O(log N) | O(N) | O(N log N) |
| Decrease-key (re-position in sorted order) | Θ(N) | O(N) | **O(N²)** |

$$T(N) = O(N) + N\log N + N\log N + N\cdot N = O(N^2)$$

**Why wrong:** (A), (B) ignore the N×N decrease-key term. (D) is too loose, since the dominant term is $N^2$, not $N^2\log N$.

---

### 8.4 Best data structure for a mixed workload ⭐
Algorithm performs $(\log N)^{1/2}$ **find**, $N$ **insert**, $(\log N)^{1/2}$ **delete**, $(\log N)^{1/2}$ **decrease-key** operations (pointers given for delete/decrease-key). Best total asymptotic complexity?

- **(A) Unsorted array** ✔ **(solved)**
- (B) Min-heap
- (C) Sorted array
- (D) Sorted doubly linked list

| Data structure | Insert (×N) | Find (×√log N) | Delete (ptr) | Dec-key (ptr) | **Total** |
|---|---|---|---|---|---|
| **Unsorted array** | O(1) → **N** | O(N) → **N√log N** | O(1) | O(1) | **O(N√log N)** ✔ |
| Min-heap | O(log N) → N log N | O(N) → N√log N | O(log N) | O(log N) | O(N log N) |
| Sorted array | O(N) → N² | O(log N) | O(N) | O(N) | O(N²) |
| Sorted DLL | O(N) → N² | O(N) | O(1) | O(N) | O(N²) |

**Why:** the **N inserts dominate**, so choose the structure with O(1) insert → unsorted array.
(B) pays $\log N$ per insert; (C) and (D) pay $N$ per insert.

---

## 9. Recursive Algorithms: Framework & Recurrences

### 9.1 Why recurrences?
- Iterative algorithms → count loop iterations.
- **Recursive algorithms → need more sophisticated techniques: define & solve a *recurrence relation*.**

### 9.2 Anatomy of a recursive algorithm

| # | Component | Meaning |
|---|---|---|
| 1 | **Base case** | stopping condition, returns without recursing |
| 2 | **Recursive case** | calls itself on a smaller input |
| 3 | **Call stack** | stack of **activation records** (one per active call) |
| 4 | **Recursion depth** | max number of nested calls (decides **space**) |
| 5 | **Tail recursion** | recursive call is the last statement |

**Flow:** `Initial Call → [Work + (Recursion | Base Case)] → Return`.

### 9.3 Time-complexity framework

```c
Algorithm What(n)
{
    if (n = 1) return;          // base case
    else {                       // n > 1
        What(n - 1);             // recursive call
        B(n);                    // extra work
    }
}
```

Let $T(n)$ = time complexity of `What(n)`:

$$T(n) = \begin{cases} c, & n = 1\\ a + T(n-1) + O(B(n)), & n > 1\end{cases}$$

| Symbol | Meaning |
|---|---|
| $c$ | constant time for the base case |
| $a$ | constant overhead of the call/comparison |
| $T(n-1)$ | cost of the recursive call |
| $B(n)$ | cost of the non-recursive work done at each level |

### 9.4 Method: Back Substitution
Repeatedly substitute the recurrence into itself, find the pattern after $k$ steps, then pick $k$ so that the argument hits the base case ($n - k = 1 \Rightarrow k = n-1$).

#### Case 1: $B(n) = O(1)$

$$T(n) = T(n-1) + d, \quad d > 0$$

| Step | Expression |
|---|---|
| ① | $T(n) = T(n-1) + d$ |
| ② | $T(n-1) = T(n-2) + d$ |
| ③ | $T(n) = T(n-2) + 2d$ |
| ④ | $T(n) = T(n-3) + 3d$ |
| ⑤ | $T(n) = T(n-k) + kd$ |

Set $n - k = 1 \Rightarrow k = n-1$:

$$T(n) = T(1) + (n-1)d = c + dn - d \;\Rightarrow\; \boxed{T(n) = \Theta(n)}$$

(both $O(n)$ and $\Omega(n)$ ⇒ $\Theta(n)$)

**Space:** recursion depth is $k = n-1$ activation records ⇒ **O(n)** ✔ **(lecture)**

#### Case 2: $B(n) = O(n)$

$$T(n) = \begin{cases} c, & n=1\\ a + T(n-1) + n, & n>1\end{cases}$$

| Step | Expression |
|---|---|
| ① | $T(n) = T(n-1) + n + a$ |
| ② | $T(n-1) = T(n-2) + (n-1) + a$ |
| ③ | $T(n) = T(n-2) + (n-1) + n + 2a$ |
| ④ | $T(n) = T(n-3) + (n-2) + (n-1) + n + 3a$ |
| ⑤ | $T(n) = T(n-k) + [n-(k-1)] + \dots + n + ka$ |

With $k = n-1$ (and taking $c = 1$):

$$T(n) = T(1) + (2+3+\dots+n) + a(n-1) = \frac{n(n+1)}{2} + an - a \;\Rightarrow\; \boxed{T(n) = O(n^2)}$$

### 9.5 Summary: `What(n)` recurrence

| $B(n)$ | Recurrence | Time | Space (stack) |
|---|---|---|---|
| $O(1)$ | $T(n) = T(n-1) + d$ | $\Theta(n)$ | $O(n)$ |
| $O(n)$ | $T(n) = T(n-1) + n + a$ | $O(n^2)$ | $O(n)$ |

---

## 10. Memory Tricks & Mnemonics

- **Program = Instructions + Data (Input).** Instruction space is a *constant*, and input is *excluded*, so only **work space** counts.
- **"Space ≤ Time":** you can't touch more memory cells than you have time to write, so **$S(n) = O(T(n))$**.
- **P < E:** **P**olynomial always loses to **E**xponential. Label every function P or E first, then only compare within each group.
- **Log base is trivial, exponent/base of power is not:**
  - $\log_2 n \sim \log_{10} n$ ✔ (differ by constant 3.33)
  - $n^2 \ne \Theta(n^5)$, $2^n \ne \Theta(3^n)$
- **Value vs order:** $\log_2 n$ is *numerically* bigger (×3.33) but *asymptotically* equal to $\log_{10} n$.
- **"Factor out the common part"** when comparing (e.g., $n\log n$ common in $n^2\log n$ vs $n\log^{10}n$), then compare the leftovers.
- **Exponentials → take log both sides** before comparing.
- **Piecewise functions:** ignore small-$n$ behaviour; only the tail after $n_0$ matters.
- **Recursion = Activation Record on the Call Stack**, so space ≈ recursion depth.
- **Back-substitution recipe:** expand → spot the pattern at step $k$ → set $n-k=1$ → simplify.
- **Leader trick:** scan **right → left** keeping a running max (Linear Search R→L), turning O(n²) into O(n).
- **Level-sum trick for tree height:** *i nodes at level i* ⇒ $h\sim\sqrt n$; *$2^{i-1}$ nodes* ⇒ $h\sim\log n$; *1 node* ⇒ $h = n$.
- 🔹 **Added tip:** For mixed-operation DS questions, find the **dominant operation count** (the biggest multiplier) and pick the DS that makes *that* operation cheapest.

---

## 11. Quick Revision Checklist

**Space complexity**
- [ ] Definition: work space only, excluding input
- [ ] Program = instructions (constant) + data (input)
- [ ] $S(n) = O(T(n))$ and why
- [ ] Space of SWAP, Linear Search (O(1)); SumSq with array B (O(n)); MERGE (Θ(n))
- [ ] Binary Search / Selection Sort / Insertion Sort → Θ(1) space

**Asymptotics**
- [ ] Meaning of O, Ω, Θ, o
- [ ] Log base trivial; exponent / power base not trivial
- [ ] All 11 True/False items (esp. $2^{2n}\ne O(2^n)$, $\sqrt{\log n}\ne O(\log\log n)$, $(n+k)^m=\Theta(n^m)$)
- [ ] Factor-out technique ($n^2\log n$ vs $n\log^{10}n$)
- [ ] Log-both-sides technique ($10^n$, $n^{\log n}$, $(\sqrt n)^n$)
- [ ] Piecewise function question (f = O(g) from $n_0 = 100$)
- [ ] P < E and the full growth hierarchy
- [ ] Orderings (a)–(f) in section 4.4

**GATE PYQs**
- [ ] $\sum i^3$ → Θ(n⁴) (I, III, IV)
- [ ] $2^n$ vs $n^n$
- [ ] $\log_2 n$ vs $\log_{10} n$ → Θ
- [ ] $n^{\log_2 n}$ vs $n^{\log_{10} n}$ → Ω only
- [ ] Order of $10, \sqrt n, n, \log_2 n, 100/n$
- [ ] $10^n$, $n^{\log n}$, $(\sqrt n)^n$ order
- [ ] f < g = h relations (D false)
- [ ] GATE 2015: $n$ vs $n^{1+\sin n}$ → neither
- [ ] GATE 2023 MSQ: $n$ vs $n^2$ → O and o
- [ ] $f(n^2)$ vs $f(n)^2$ for polynomial / exponential / log
- [ ] Package A vs B → x = 6

**Loop analysis**
- [ ] Leader problem: brute force best/worst cases (O(n) / O(n²))
- [ ] Optimal right-to-left scan → O(n), O(1) space

**Trees & data structures**
- [ ] Height bounds: log n ≤ h ≤ n
- [ ] Level-i-has-i-nodes → $O(\sqrt n)$
- [ ] Complete binary tree → $h=\log_2(n+1)$
- [ ] MultiQueue amortized → O(n)
- [ ] Array (circular) queue → O(1) both ops
- [ ] Sorted DLL combined ops → O(N²)
- [ ] Best DS for N inserts + few others → Unsorted array

**Recursion**
- [ ] Why recurrences are needed
- [ ] 5 components: base case, recursive case, call stack (activation record), depth, tail recursion
- [ ] Framework: $T(n)=c$ for $n=1$; $a+T(n-1)+O(B(n))$ for $n>1$
- [ ] Back substitution steps
- [ ] $B(n)=O(1)$ → Θ(n) time, O(n) space
- [ ] $B(n)=O(n)$ → O(n²) time

---

*Next in series: solving recurrences (Master Theorem, recursion tree) and loop-analysis patterns.*
