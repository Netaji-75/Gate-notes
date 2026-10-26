# Design & Analysis of Algorithms — Lecture 02: Analysis of Algorithms & Asymptotic Notations

> **Source:** GeeksforGeeks GATE – DAA, *I. Analysis of Algorithms*, Lecture 02 (slides + whiteboard).
> **Note:** This lecture is **Algorithms (DAA)**, not OS. Filed here as part of your GATE CS/IT notes collection.
> **Code in lecture:** none (only algorithm names as examples), so no code blocks below.

---

## Table of Contents

1. [Why analyse algorithms? Apriori analysis](#1-why-analyse-algorithms-apriori-analysis)
2. [Best / Average / Worst case](#2-best--average--worst-case)
3. [Asymptotic notations — why and what](#3-asymptotic-notations--why-and-what)
4. [Big-Oh (O) — upper bound](#4-big-oh-o--upper-bound)
5. [Big-Omega (Ω) — lower bound](#5-big-omega-ω--lower-bound)
6. [Big-Theta (Θ) — tight bound](#6-big-theta-θ--tight-bound)
7. [Small-oh (o) and Small-omega (ω)](#7-small-oh-o-and-small-omega-ω)
8. [Rate of growth & ordering of functions](#8-rate-of-growth--ordering-of-functions)
9. [Analogy with real numbers + Venn view](#9-analogy-with-real-numbers--venn-view)
10. [Space complexity](#10-space-complexity)
11. [Math toolkit: logs, exponentials, series](#11-math-toolkit-logs-exponentials-series)
12. [Properties of Big-Oh](#12-properties-of-big-oh)
13. [Relational properties (reflexive, symmetric, transitive, transpose, trichotomy)](#13-relational-properties-of-asymptotic-notations)
14. [Worked examples: finding Θ of sums/products](#14-worked-examples-finding-θ-of-sums-and-products)
15. [Factorial, Stirling & log n!](#15-factorial-stirlings-approximation--log-n)
16. [GATE-style questions](#16-gate-style-questions)
17. [Memory tricks](#17-memory-tricks--analogies)
18. [Quick revision checklist](#18-quick-revision-checklist)

---

## 1. Why analyse algorithms? Apriori analysis

- **Apriori analysis** = analyse the algorithm **before** running it, as a function of input size $n$. It is machine-independent.
- Output is the **time function $T(n)$**, bounded two ways:
  - **Upper bound (UB)** → worst case → **O**
  - **Lower bound (LB)** → best case → **Ω**
- **Use 1:** Find $T(n)$ of one algorithm.
- **Use 2:** Compare two algorithms. Given $T_1(n)$ and $T_2(n)$, if $T_1(n) = O(T_2(n))$ then Algo 1 grows no faster than Algo 2.

---

## 2. Best / Average / Worst case

For a **fixed input size $n$**, there can be different **input classes** $I_1, I_2, \dots, I_k$ (e.g., increasing order, decreasing order, random…). Each class has a running time $t_i$ and occurrence probability $p_i$.

| Case | Definition | Curve on graph |
|---|---|---|
| **Worst case** $W(n)$ | **Max** steps over any instance of size $n$ | Through the **highest** point of each column |
| **Best case** $B(n)$ | **Min** steps over any instance of size $n$ | Through the **lowest** point of each column |
| **Average case** $A(n)$ | **Average** steps over all instances of size $n$ | Between the two |

**Formula (average case):**

$$A(n) = \sum_{i=1}^{k} p_i \cdot t_i$$

- $k$ = number of input classes, $p_i$ = probability of class $i$ (must sum to 1), $t_i$ = time taken on class $i$.

**Always:**

$$B(n) \le A(n) \le W(n)$$

### Four possible relationships

| # | Relation | Example algorithm |
|---|---|---|
| 1 | $B(n) = A(n) = W(n)$ | **Merge Sort** |
| 2 | $B(n) = A(n) < W(n)$ | **Quick Sort** |
| 3 | $B(n) < A(n) = W(n)$ | **Linear Search** |
| 4 | $B(n) < A(n) < W(n)$ | *(left blank in lecture — all three differ)* |

> ⚠️ **Exam caution:** The summary slide maps *Best → Ω, Average → Θ, Worst → O* and says worst case is used most for reliability guarantees. This is the common shorthand, but remember: **O, Ω, Θ are bounds on functions, not cases**. You can state an O-bound, Ω-bound or Θ-bound for *any* of B(n), A(n), W(n).

**Linear search illustration (typed slide):** found at index 0 → best case $\Omega(1)$; expected ≈ $n/2$ comparisons → average $\Theta(n)$; not found → worst $O(n)$.

---

## 3. Asymptotic notations — why and what

- We **don't** measure exact time (2 ms vs 3 ms) — it depends on hardware, compiler, etc. We study **growth behaviour as $n \to \infty$**.
- Five notations: **O, Ω, Θ, o, ω** (the "main three": Big-O, Omega, Theta).
- **Origin:** Big-O is part of the **Bachmann–Landau notation** family (Paul Bachmann, Edmund Landau). The letter O stands for ***Ordnung*** (German: "order of approximation").

### Five main roles

| # | Role | Key point |
|---|---|---|
| 1 | **Describe rate of growth** | O(n) linear, O(n²) quadratic, O(log n) logarithmic → shows scalability |
| 2 | **Compare algorithms** | Machine-independent. Algo A = $100n$ vs Algo B = $n^2$ → A better for large $n$ (since $100n < n^2$ for $n > 100$) |
| 3 | **Ignore constants & lower-order terms** | Keep only the dominant term: $T(n) = 5n^2 + 3n + 10 \Rightarrow O(n^2)$ |
| 4 | **Give bounds** | O = upper, Ω = lower, Θ = tight |
| 5 | **Help in algorithm design** | Pick efficient approaches; avoid poorly scaling solutions |

### Worked comparison: $n^2$ vs $2^n$ (whiteboard)

| $n$ | $n^2$ | $2^n$ |
|---|---|---|
| 1 | 1 | 2 |
| 2 | 4 | 4 |
| 3 | 9 | 8 |
| **4** | **16** | **16** ← equal (crossover) |
| 5 | 25 | 32 |
| 6 | 36 | 64 |

For $n \ge 5$, $2^n$ stays larger → asymptotically $n^2 < 2^n$.

---

## 4. Big-Oh (O) — upper bound

**Definition:**

$$f(n) = O(g(n)) \iff \exists\, c, k > 0 \ \text{such that}\ f(n) \le c \cdot g(n)\ \ \forall\, n \ge k$$

- $c$ = positive constant multiplier; $k$ (a.k.a. $n_0$) = threshold after which the inequality holds. Both must be **fixed** and **independent of $n$**.
- CLRS set form: $O(g(n)) = \{ f(n) : \exists\, c, n_0 > 0,\ 0 \le f(n) \le c\,g(n)\ \forall n \ge n_0 \}$
- Read as "f of n is big-oh of g of n". $O(g(n))$ is a **set of functions**, so $f(n) \in O(g(n))$ (the "=" is a notational convention).

### Worked examples

| Ex | Function | Reasoning | Result |
|---|---|---|---|
| 1 | $f(n) = n^2 + n + 1$ | $n^2+n+1 < n^2+n^2+n^2 = 3n^2$ (c = 3, g = n²) | $O(n^2)$ ✓ |
| 1 (cont.) | same | $n^2+n+1 < 5n^3$ for $n>1$ | also $O(n^3)$, $O(n^4)$, … ✓ |
| 2 | $f(n) = n$ | $n \le 2n$, $n>1$ | $O(n)$ ✓, $O(n\log n)$ ✓, $O(\sqrt n)$ ✗ |
| 3 | $f=3n$, $g=n+\log n$ | each is within a constant of the other | (a) $f=O(g)$ ✓ (b) $g=O(f)$ ✓ |
| 4 | $f(n) = 2^{100}$ (constant) | $2^{100} < 4^{100}\cdot 1$ (c = 4¹⁰⁰, g = 1) | $O(1)$ ✓ |
| 4′ | $f(n) = 10$ | $10 < 20 \cdot 1$ | $O(1)$ ✓ |
| 5 | $f=\log n$, $g=2^{10}$ (constant) | $\log n$ grows, constant doesn't | (a) $f=O(g)$ ✗ (b) $g=O(f)$ ✓ |

**Rule of thumb (boxed in lecture):**

> A function with **lesser or equal** rate of growth $= O(\text{function with equal or higher rate of growth})$.

### Set picture of $O(n^2)$

| Function | In $O(n^2)$? |
|---|---|
| $2^{100}$ (constant) | ✓ |
| $2n$ | ✓ |
| $1000\, n\log n$ | ✓ |
| $100 n^2$ | ✓ |
| $3n^3$ | ✗ |

---

## 5. Big-Omega (Ω) — lower bound

$$f(n) = \Omega(g(n)) \iff \exists\, c, k > 0 \ \text{such that}\ f(n) \ge c \cdot g(n)\ \ \forall\, n \ge k$$

CLRS: $\Omega(g(n)) = \{ f(n) : \exists\, c, n_0 > 0,\ f(n) \ge c\,g(n) \ge 0\ \forall n \ge n_0 \}$

### Worked example: $f(n) = n^2 + n + 1$

| Claim | Reason | Valid? |
|---|---|---|
| $\Omega(n^2)$ | $f(n) \ge 1\cdot n^2$, $n>1$ | ✓ |
| $\Omega(n)$ | $f(n) \ge 2n$, $n>1$ | ✓ |
| $\Omega(n\log n)$ | $f(n) \ge c_1\, n\log n$ | ✓ |
| $\Omega(n^5)$ | $f(n) \ge c\,n^5$ fails for large $n$ | ✗ ($f \ne \Omega(n^5)$) |

### Set picture

| Set | Members ✓ | Non-members ✗ |
|---|---|---|
| $\Omega(n^2)$ | $10n^2$, $n^3$, $n^2\log n$ | $n\log n$ |
| $\Omega(1)$ | $\log n$, $n$, $100$ | $1/n$ |

---

## 6. Big-Theta (Θ) — tight bound

$$f(n) = \Theta(g(n)) \iff f(n) = O(g(n)) \ \text{AND}\ f(n) = \Omega(g(n))$$

Equivalently, $\exists\, c_1, c_2, n_0 > 0$ such that

$$c_2 \cdot g(n) \;\le\; f(n) \;\le\; c_1 \cdot g(n) \quad \forall n \ge n_0$$

- $g(n)$ is both an upper and lower bound (up to constants) → **exact growth rate**.
- **Example:** $1\cdot n^2 \le 1 + n + n^2 \le 3\cdot n^2$ ⇒ $\Theta(n^2)$.
- Typed-slide example: $f(n) = 3n+5 \Rightarrow O(n),\ \Omega(n),\ \Theta(n)$.

### Summary table (ASN)

| Notation | Meaning | Condition | Example $f=3n+5$ |
|---|---|---|---|
| **O** (Big-Oh) | Upper bound | $f(n) \le c\,g(n)$, $n\ge n_0$ | $O(n)$ |
| **Ω** (Big-Omega) | Lower bound | $f(n) \ge c\,g(n)$, $n\ge n_0$ | $\Omega(n)$ |
| **Θ** (Big-Theta) | Tight bound | $c_1 g(n) \le f(n) \le c_2 g(n)$ | $\Theta(n)$ |

---

## 7. Small-oh (o) and Small-omega (ω)

| Notation | Meaning | Condition | Think |
|---|---|---|---|
| **o(g)** | **Proper (strict)** upper bound — $g$ strictly greater than $f$ | $f(n) < c\cdot g(n)$ for **all** $c>0$, $n \ge k$ | $<$ |
| **ω(g)** | **Proper (strict)** lower bound — $g$ strictly smaller than $f$ | $f(n) > c\cdot g(n)$ for **all** $c>0$, $n \ge k$ | $>$ |

> Big-O / Ω need **some** $c$; small-o / ω need **every** $c > 0$.

### Examples with $f(n) = n^2$

| Claim | Valid? | | Claim | Valid? |
|---|---|---|---|---|
| $O(n^2)$ | ✓ | | $\Omega(n^2)$ | ✓ |
| $O(n^3)$ | ✓ | | $\Omega(n)$ | ✓ |
| $o(n^2)$ | ✗ (same rate) | | $\Omega(n\log n)$ | ✓ |
| $o(n^3)$ | ✓ | | $\omega(n^2)$ | ✗ |
| | | | $\omega(n)$ | ✓ |
| | | | $\omega(n\sqrt n)$ | ✓ |

---

## 8. Rate of growth & ordering of functions

**Increasing order of growth:**

$$\text{Constant} < \log < \text{Linear} < \text{Quadratic} < \text{Cubic} < \dots < 2^n < 3^n < 4^n < \dots < n^n$$

$$O(1) < O(\log n) < O(n) < O(n\log n) < O(n^2) < O(2^n)$$

On the chart (low → high): $1,\ \log_2 n,\ \sqrt n,\ n,\ n\log_2 n,\ n^2,\ 2^n,\ n!$

| Class | Name | Value at $n=100$ | Typical algorithm |
|---|---|---|---|
| $O(1)$ | Constant | 1 | Array access |
| $O(\log n)$ | Logarithmic | 7 | Binary search |
| $O(n)$ | Linear | 100 | Linear search |
| $O(n\log n)$ | Linearithmic | 664 | Merge sort |
| $O(n^2)$ | Quadratic | 10,000 | Bubble sort |
| $O(2^n)$ | Exponential | $1.27\times10^{30}$ | Subset problems |

---

## 9. Analogy with real numbers + Venn view

Treat functions $f, g$ like real numbers $a, b$:

| Notation | Real-number analogue |
|---|---|
| $f = O(g)$ | $a \le b$ |
| $f = \Omega(g)$ | $a \ge b$ |
| $f = \Theta(g)$ | $a = b$ |
| $f = o(g)$ | $a < b$ |
| $f = \omega(g)$ | $a > b$ |

**Venn picture:** $O(g)$ and $\Omega(g)$ are overlapping ellipses; their **intersection is $\Theta(g)$**; $o(g)$ sits inside $O(g)$ (not touching Θ); $\omega(g)$ sits inside $\Omega(g)$.

**Supremum / infimum picture (whiteboard):** among all upper-bound functions of $f(n)$, the tightest (least) one is the **supremum**; among all lower-bound functions, the tightest (greatest) one is the **infimum**.

---

## 10. Space complexity

- **Space** = number of memory cells (words) needed to carry out the computation, **excluding space that holds the input**. i.e., only the **work space**.
- All order-of-growth and asymptotic-bound definitions for time **carry over** to space.
- Work space can't exceed running time (writing each cell takes ≥ constant time):

$$S(n) = O(T(n))$$

| Algorithm | Space complexity | Why |
|---|---|---|
| Linear Search | $\Theta(1)$ | One cell for result + loop variables |
| Binary Search, Selection Sort, Insertion Sort | $\Theta(1)$ | Same reason (in-place) |
| MERGE (merging two sorted arrays) | $\Theta(n)$ | Auxiliary storage equal to input size $n$ |

---

## 11. Math toolkit: logs, exponentials, series

### Logarithm properties

| # | Property |
|---|---|
| 1 | $\log_c(ab) = \log_c a + \log_c b$ |
| 2 | $\log_b a^n = n\log_b a$ |
| 3 | $\log_b a = \dfrac{\log_c a}{\log_c b}$ (change of base) |
| 4 | $\log_b \dfrac1a = -\log_b a$ |
| 5 | $\log_b a = \dfrac{1}{\log_a b}$ |
| 6 | $a^{\log_b c} = c^{\log_b a}$ |
| 7 | $a = b^{\log_b a}$ |
| + | $\log\dfrac ab = \log a - \log b$ (handwritten) |

### Exponentials (for real $a>0$)

$a^0 = 1,\quad a^1 = a,\quad a^{-1} = 1/a,\quad (a^m)^n = a^{mn} = (a^n)^m,\quad a^m\cdot a^n = a^{m+n}$

### Geometric sum (first term $a$, ratio $r$, $n$ terms)

| Case | Formula |
|---|---|
| $r = 1$ | $S_n = n\cdot a$ |
| $\lvert r\rvert < 1$ | $S_n = \dfrac{a(1-r^n)}{1-r}$ |
| $\lvert r\rvert > 1$ | $S_n = \dfrac{a(r^n-1)}{r-1}$ |
| Infinite, $\lvert r\rvert<1$ | $S_\infty = \dfrac{a}{1-r}$ |
| Infinite, $\lvert r\rvert>1$ | Diverges — no sum |

Whiteboard checks: $\sum_{i=1}^{n}2^i = \dfrac{2(2^n-1)}{2-1} = 2^{n+1}-2$; $\ \sum_{i=1}^{n}\left(\tfrac12\right)^i = 1 - \tfrac{1}{2^n}$.

### Standard series

| Series | Formula |
|---|---|
| Arithmetic | $\displaystyle\sum_{k=1}^{n}k = \frac{n(n+1)}{2}$ |
| Geometric | $\displaystyle\sum_{k=0}^{n}x^k = \frac{x^{n+1}-1}{x-1}\ (x\ne1)$ |
| Harmonic | $\displaystyle\sum_{k=1}^{n}\frac1k \approx \log n$ |

Integral rule used for approximations: $\displaystyle\int x^n\,dx = \frac{x^{n+1}}{n+1}$

---

## 12. Properties of Big-Oh

Let $d, e, f, g$ map non-negative integers to non-negative reals.

| # | Property | Statement |
|---|---|---|
| 1 | **Scalar multiplication** | $d=O(f) \Rightarrow a\cdot d = O(f)$ for constant $a>0$ (e.g., $100n - O(n)$ is still $O(n)$) |
| 2 | **Sum rule** | $d=O(f),\ e=O(g) \Rightarrow d+e = O(\max(f,g))$ |
| 3 | **Product rule** | $d=O(f),\ e=O(g) \Rightarrow d\cdot e = O(f\cdot g)$ |
| 4 | **Transitivity** | $d=O(f),\ f=O(g) \Rightarrow d=O(g)$ |
| 5 | **Polynomial** | Degree-$d$ polynomial is $O(n^d)$; note $(n+k)^d \sim n^d$ |
| 6 | **Exponential dominates** | $n^x = O(a^n)$ for fixed $x>0,\ a>1$ |
| 7 | **Log power** | $\log n^x = x\log n = O(\log n)$ for fixed $x>0$ |
| 8 | **Log vs polynomial** | $\log^x n = O(n^y)$ for fixed $x>0,\ y>0$ (e.g., $(\log n)^9 = O(n)$) |

**Example (props 2 & 3):** $d(n)=n^2,\ e(n)=n^3$ → $d+e = O(n^3)$, $d\cdot e = O(n^5)$.

**Trick for $(\log n)^9$ vs $n$:** take log of both → $9\log\log n$ vs $\log n$. Since $\log\log n < \log n$ asymptotically, $n$ wins.

---

## 13. Relational properties of asymptotic notations

### Reflexive / Symmetric / Transitive / Transpose symmetry

| Property | O | Ω | Θ | o | ω |
|---|---|---|---|---|---|
| **Reflexive** | ✓ | ✓ | ✓ | ✗ | ✗ |
| **Symmetric** | ✗ | ✗ | ✓ | ✗ | ✗ |
| **Transitive** | ✓ | ✓ | ✓ | ✓ | ✓ |
| **Transpose symmetry** | $f=O(g)\Leftrightarrow g=\Omega(f)$ | (same pair) | — (not applicable; Θ is symmetric) | $f=o(g)\Leftrightarrow g=\omega(f)$ | (same pair) |

(Compare with reals: $a\le b \Leftrightarrow b\ge a$.)

### Trichotomy

- **Real numbers:** for any $a, b$, exactly one of $a<b$, $a>b$, $a=b$ holds.
- **Functions:** one *might expect*: $f<g$ → O, o; $f>g$ → Ω, ω; $f=g$ → Θ. Examples where it works:
  1. $f=\log n,\ g=n$ → $f<g$
  2. $f=\log n,\ g=1/n$ → $f>g$
  3. $f=10n,\ g=2n+100$ → $f=g$ (Θ)
- **BUT** asymptotic notations **need NOT satisfy trichotomy.** Counter-example: $f(n)=n,\ g(n)=n^{1+\sin n}$. Exponent oscillates between 0 and 2, so $g$ swings between $n^0$ and $n^2$; neither $f=O(g)$ nor $f=\Omega(g)$ holds.

> **Exam point:** *Any two functions are not always comparable* using ASNs.

---

## 14. Worked examples: finding Θ of sums and products

| # | $f(n)$ | Working | Result |
|---|---|---|---|
| 1 | $n + \log n$ | $O(n)$ and $\Omega(n)$ | $\Theta(n)$ |
| 2 | $(n+k)^m,\ (k,m)>0$ | $O(n^m)$ and $\Omega(n^m)$ | $\Theta(n^m)$ |
| 3 | $n + 2^{100} + n^2$ | dominant term $n^2$ | $\Theta(n^2)$ |
| 4 | $\sum_{i=1}^n 1$ | $= n$ | $\Theta(n)$ |
| 5 | $\sum_{i=1}^n i$ | $= \frac{n(n+1)}{2}$ | $\Theta(n^2)$ |
| 6 | $\sum_{i=1}^n i^2$ | $= \frac{n(n+1)(2n+1)}{6}$ | $\Theta(n^3)$ |
| 7 | $\sum_{i=1}^n i^3$ | $= \left[\frac{n(n+1)}{2}\right]^2$ | $\Theta(n^4)$ |
| 8 | $\sum_{i=1}^n 2^i$ | $= \frac{2(2^n-1)}{2-1} = 2^{n+1}-2$ | $\Theta(2^n)$ |
| 9 | $\sum_{i=1}^n i\cdot 2^i$ | $= (n-1)\,2^{n+1}+2$ | $\Theta(n\cdot 2^n)$ |
| 10 | $\prod_{i=1}^n 1$ | $= 1^n = 1$ | $\Theta(1)$ |
| 11 | $\prod_{i=1}^n i$ | $= n!$ | see [§15](#15-factorial-stirlings-approximation--log-n) |
| 12 | $\sum_{i=1}^n (1/2)^i$ | $= 1 - \frac{1}{2^n}$ (geometric, $r<1$) | $O(1)$ (i.e., $\Theta(1)$) |
| 13 | $\sum_{x=1}^n \frac1x$ | $\approx \int_1^n \frac1x dx = [\log x]_1^n$ | $\Theta(\log n)$ |
| 14 | $\sum_{i=1}^n i^{-1/2}$ | $\approx \int i^{-1/2}\,di$ | $\Theta(\sqrt n)$ |
| 15 | $\sum_{i=1}^n \sqrt i$ | $\approx \int_1^n i^{1/2}di = \left[\frac{i^{3/2}}{3/2}\right]_1^n$ | $\Theta(n^{3/2}) = \Theta(n\sqrt n)$ |
| 16 | $n^{1/\log_2 n}$ | $1/\log_2 n = \log_n 2$ ⇒ $n^{\log_n 2} = 2$ | $2 = O(1)$ ✓ |

---

## 15. Factorial, Stirling's approximation & $\log n!$

### Bounds on $n!$

- $n! = n(n-1)\cdots 2\cdot1 < n\cdot n\cdots n = n^n$ ⇒ $\;n! = O(n^n)$ ✓
- Can we say $n! = \Omega(n^n)$? **No** — $n! < n^n$, so $n! \ne \Omega(n^n)$. (In fact $n! = o(n^n)$.)
- Valid lower bounds: $n! = \Omega(n)$, $\Omega(n^2)$, and $\Omega(2^n)$.
  - Reason for $\Omega(2^n)$: $n! = 1\cdot2\cdot3\cdots n \ge 2\cdot2\cdots2$ ($n-1$ factors) $= 2^{n-1}$.
  - ⚠️ *Correction:* the whiteboard writes the inequality as "$<2\cdot2\cdots2 < 2^n$", but the direction should be **$n! \ge 2^{n-1}$** (so $n!$ grows **faster** than $2^n$) — that's what gives $\Omega(2^n)$.

### Stirling's approximation

$$n! \simeq \sqrt{2\pi n}\left(\frac{n}{e}\right)^n$$

- $n!$ is strictly less than $n^n$: $n^n > \sqrt{2\pi n}\,(n/e)^n \Rightarrow e^n > \sqrt{2\pi}\sqrt n = c\sqrt n$ ✓ (true for all large $n$).

### $f(n) = \sum_{x=1}^{n}\log x = \log n!$ → $\Theta(n\log n)$

**Approach I (Stirling):**

$$\log n! = \log\sqrt{2\pi} + \tfrac12\log n + n\log n - n\log e = \Theta(n\log n)$$

(the $n\log n$ term dominates).

**Approach II (integral):**

$$\sum_{i=1}^n \log i \approx \int_1^n \log x\,dx = [x\log x - x + c] = n\log n - n + c = \Theta(n\log n)$$

---

## 16. GATE-style questions

### Q1. Worst-case $W(n)$ and average-case $A(n)$ running times on input size $n$. Which is **ALWAYS TRUE**?

| Option | Statement | Verdict | Reason |
|---|---|---|---|
| A | $A(n) = \Omega(W(n))$ | ✗ | Needs $A$ ≥ $W$ order; false when $A$ is asymptotically smaller (e.g., Quick Sort: $A=n\log n$, $W=n^2$) |
| B | $A(n) = \Theta(W(n))$ | ✗ | Same counter-example — $A$ and $W$ can differ in order |
| **C** | **$A(n) = O(W(n))$** | **✓** | $A(n) \le W(n)$ always |
| D | $A(n) = o(W(n))$ | ✗ | Fails when $A = W$ in order (e.g., Merge Sort, Linear Search) |

Also noted: $A(n) = \omega(W(n))$ is **never true** (since $A \le W$).

### Q2. State True / False

| # | Statement | Answer | Reason |
|---|---|---|---|
| 1 | $100\,n\log n = O(n\log n)$ | **True** | Constant factor ignored |
| 2 | $2^{n+1} = O(2^n)$ | **True** | $2^{n+1} = 2\cdot2^n$ |
| 3 | $2^{2n} = O(2^n)$ | **False** | $2^{2n} = (2^2)^n = 4^n > 2^n$ |
| 4 | $0<x<y \Rightarrow n^x = O(n^y)$ | **True** | Higher exponent dominates |
| 5 | $(n+k)^m \ne \Theta(n^m)$, $(k,m)>0$ | **False** | It **is** $\Theta(n^m)$ |
| 6 | $\sqrt{\log n} = O(\log\log n)$ | **False** | $\sqrt{\log n}$ grows faster than $\log\log n$ |
| 7 | $\log n$ is $\Omega(1/n)$ | **True** | $\log n \ge c\cdot\frac1n$ for large $n$ |
| 8 | $2^{n^2}$ is $O(n!)$ | **False** | $n! \le n^n$; compare logs: $n^2$ vs $n\log n$ → $2^{n^2}$ grows faster |
| 9 | $n^2$ is $O(2^{2\log n})$ | **True** | $2^{2\log_2 n} = 2^{\log_2 n^2} = n^2$ |
| 10 | $a^n \ne O(n^x)$, $a>1,\ x>0$ | **True** | Exponential beats polynomial |
| 11 | $2^{\log_2 n^2}$ is $O(n^2)$ | **True** | Equals $n^2$ |

### Q3. $f(n) = \sum_{i=1}^{n} i^3 = x$. Choices for $x$ is/are:

Since $\sum i^3 = \left[\frac{n(n+1)}{2}\right]^2 = \Theta(n^4)$:

| Statement | Verdict | Reason |
|---|---|---|
| I. $\Theta(n^4)$ | ✓ | Exact growth |
| II. $\Theta(n^5)$ | ✗ | $n^5$ is too large to be a lower bound |
| III. $O(n^5)$ | ✓ | $n^4 \le n^5$ → valid upper bound |
| IV. $\Omega(n^3)$ | ✓ | $n^4 \ge n^3$ → valid lower bound |

| Option | Set | Correct? |
|---|---|---|
| A | I, II and IV | ✗ (II wrong) |
| B | I and III | ✗ (misses IV) |
| C | I and IV | ✗ (misses III) |
| **D** | **I, III and IV** | **✓** |

### Q4. $f(n), g(n)$ positive functions; $f(n) = O(g(n))$. Then:

| Part | Statement | Answer | Reason / counter-example |
|---|---|---|---|
| a | Is $f(n)$ always $O\big((f(n))^2\big)$? | **No** ✗ | $f(n)=\frac1n$: $\frac1n > \frac1{n^2}$, so $f \ne O(f^2)$ (squaring shrinks functions < 1) |
| b | Is $2^{f(n)} = O(2^{g(n)})$? | **No** ✗ | $f=2n,\ g=n$: $f=O(g)$ but $2^{2n}=4^n \ne O(2^n)$ (constant factors become exponents) |

---

## 17. Memory tricks & analogies

- **Multistoried building (bounds):** Imagine $f(n)$ sits on floor 10. Any floor **above** (11, 12, 13…) is an **upper bound**; any **below** (9, 8, 7…) is a **lower bound**. The **lowest upper floor = supremum**; the **highest lower floor = infimum**. Θ means you've pinned down the floor tightly from both sides.
- **Real-number analogy:** O ≈ **≤**, Ω ≈ **≥**, Θ ≈ **=**, o ≈ **<**, ω ≈ **>**. (Remember the *small* letters are the *strict* ones.)
- **Venn trick:** **Θ = O ∩ Ω**.
- **Scalar rule:** constants never matter in Big-O ($100n = O(n)$).
- **Growth ladder:** *Constant → Log → Linear → Quad → Cubic → … → Exponential ($2^n<3^n<\dots$) → $n^n$*.
- **Exponent-in-constant trap:** $f=O(g)$ does **not** imply $2^f = O(2^g)$ (see Q4b) — multiplying the exponent changes the base.
- **Trichotomy trap:** reals are always comparable; functions like $n$ and $n^{1+\sin n}$ are **not**.

---

## 18. Quick revision checklist

**Basics & analysis**
- [ ] Apriori analysis — what it is; UB ↔ worst ↔ O, LB ↔ best ↔ Ω
- [ ] Comparing two algorithms via $T_1(n) = O(T_2(n))$
- [ ] Best / Average / Worst case definitions and graph
- [ ] $A(n) = \sum p_i t_i$ and $B \le A \le W$
- [ ] Four B/A/W relationships with examples (Merge, Quick, Linear Search)
- [ ] Caution: O/Ω/Θ are bounds, not cases

**Asymptotic notations**
- [ ] Why asymptotics (hardware-independent, large $n$, ignore constants)
- [ ] Five roles of ASN; $100n$ vs $n^2$; $n^2$ vs $2^n$ crossover table (n = 4)
- [ ] Origin of "O" (*Ordnung*, Bachmann–Landau)
- [ ] Big-Oh definition + examples (Ex 1–5) + set view of $O(n^2)$
- [ ] Big-Omega definition + examples + sets $\Omega(n^2)$, $\Omega(1)$
- [ ] Big-Theta definition ($O \cap \Omega$) + $1\cdot n^2 \le n^2+n+1 \le 3n^2$
- [ ] Small-o and small-ω definitions ("for all $c>0$") + $f=n^2$ examples
- [ ] Real-number analogy (≤, ≥, =, <, >) and Venn diagram
- [ ] Supremum/infimum & building analogy

**Growth rates**
- [ ] Order: $1<\log n<\sqrt n<n<n\log n<n^2<n^3<2^n<3^n<n^n$
- [ ] Values at $n=100$: 1, 7, 100, 664, 10,000, $1.27\times10^{30}$

**Space complexity**
- [ ] Definition (work space, excludes input); $S(n)=O(T(n))$
- [ ] Linear/Binary search, Selection/Insertion sort → $\Theta(1)$; Merge → $\Theta(n)$

**Math toolkit**
- [ ] 7 log properties (+ $\log\frac ab$)
- [ ] Exponent rules
- [ ] Geometric sum (finite: $r<1$, $r>1$, $r=1$; infinite: $\frac{a}{1-r}$)
- [ ] Arithmetic, geometric, harmonic series

**Properties**
- [ ] Big-Oh properties: scalar, sum (max), product, transitivity, polynomial, exp dominates, log power, log vs poly
- [ ] Reflexive / symmetric / transitive / transpose-symmetry table
- [ ] Trichotomy for reals vs functions; counter-example $n$ vs $n^{1+\sin n}$

**Worked Θ examples**
- [ ] $\sum i,\ \sum i^2,\ \sum i^3 \to \Theta(n^2),\Theta(n^3),\Theta(n^4)$
- [ ] $\sum 2^i \to \Theta(2^n)$; $\sum i2^i \to \Theta(n2^n)$
- [ ] $\prod 1 \to \Theta(1)$; $\sum (1/2)^i \to O(1)$; $\sum 1/x \to \Theta(\log n)$
- [ ] $\sum i^{-1/2} \to \sqrt n$; $\sum\sqrt i \to n\sqrt n$; $n^{1/\log_2 n} = 2$

**Factorial**
- [ ] $n! = O(n^n)$, $n! \ne \Omega(n^n)$, $n! = \Omega(2^n)$
- [ ] Stirling's approximation and $e^n > c\sqrt n$ check
- [ ] $\log n! = \Theta(n\log n)$ (two approaches)

**Questions**
- [ ] "Always true" for $A(n)$ vs $W(n)$ → $A = O(W)$
- [ ] 11 True/False items
- [ ] $\sum i^3$ question → option D
- [ ] $f=O(g)$ ⇏ $f=O(f^2)$ and ⇏ $2^f=O(2^g)$
