# Design & Analysis of Algorithms — Lecture 05
## Analysis of Algorithms: Loop Analysis + Intro to Divide & Conquer

> **Source:** GeeksforGeeks GATE – DAA Lecture 05 (slides + whiteboard).
> **Subject:** DAA (not OS) — filed here as part of the GATE CS/IT notes collection.
> **Note on scope:** The agenda slide lists *Asymptotic Notations, Recursive Algorithms, Loop Analysis, Space Complexity*, but this PDF actually covers: one recurrence PYQ, **loop analysis (bulk of the lecture)**, and the **start of Divide & Conquer**. Space complexity and asymptotic-notation theory are **not** in this PDF.

---

## Table of Contents

1. [Recurrence warm-up (GATE 2014)](#1-recurrence-warm-up-gate-2014)
2. [Loops: types & anatomy](#2-loops-types--anatomy)
3. [Loop-analysis toolkit (rules + formulas)](#3-loop-analysis-toolkit)
4. [Worked examples 1–25 (loop running times)](#4-worked-examples)
5. [Best-case vs worst-case with a conditional (Ex. 26)](#5-best-case-vs-worst-case-example-26)
6. [GATE PYQs on loops](#6-gate-pyqs-on-loops)
7. [Homework (Ex. 30)](#7-homework-ex-30)
8. [Divide & Conquer: general method & control abstraction](#8-divide--conquer)
9. [Memory tricks / patterns](#9-memory-tricks--patterns)
10. [Quick revision checklist](#10-quick-revision-checklist)

---

## 1. Recurrence warm-up (GATE 2014)

**Q. (GATE 2014, 1M)** Solution of $T(n) = 2T\left(\frac{n}{2}\right) + \log n$, with $T(1)=1$?

| Option | Answer | Verdict |
|---|---|---|
| A | $\Theta(n)$ | ✅ **Correct** |
| B | $\Theta(n \log n)$ | ❌ This is the answer only if $f(n)=n$; here $f(n)=\log n$ is far smaller. |
| C | $\Theta(n^2)$ | ❌ Way too large; total work can't exceed the leaf work $\sim n$ plus a small overhead. |
| D | $\Theta(\log n)$ | ❌ Ignores that there are 2 subproblems per level (leaves alone give $n$). |

### Method shown: repeated substitution (expansion)

$$T(n) = 2^k\,T\!\left(\frac{n}{2^k}\right) + \sum_{i=0}^{k-1} 2^i \log\frac{n}{2^i}$$

Stop when $n/2^k = 1 \Rightarrow 2^k = n,\; k=\log_2 n$:

$$T(n) = n\,T(1) + \log n\sum_{i=0}^{k-1}2^i \;-\; \sum_{i=0}^{k-1} i\cdot 2^i$$

- $\sum_{i=0}^{k-1} 2^i = 2^k - 1 = n-1$
- $\sum_{i=0}^{k-1} i\cdot 2^i = (k-2)2^k + 2$

$$T(n) = cn + (n-1)\log n - \big[(\log n - 2)\,n + 2\big] = (c+2)\,n - \log n - 2 = \Theta(n)$$

**Quick check via Master Theorem (case 1):** $n^{\log_2 2}=n$ and $f(n)=\log n = O(n^{1-\varepsilon})$ ⇒ $T(n)=\Theta(n)$.

### Practice recurrences given on the board (all with $T(1)=1$)

The lecture only wrote these down; **solutions below are my own working, not from the video** — verify before relying on them.

| # | Recurrence | Hint | Answer (my working) |
|---|---|---|---|
| 1 | $T(n)=2T(\sqrt n)+\dfrac{\log n}{\log\log n}$ | Put $n=2^m$, $S(m)=2S(m/2)+\frac{m}{\log m}$ | $\Theta(\log n\cdot\log\log\log n)$ |
| 2 | $T(n)=2T(n/2)+\dfrac{n}{\log n}$ | Level $i$ costs $\frac{n}{\log n - i}$ → harmonic sum | $\Theta(n\log\log n)$ |
| 3 | $T(n)=\sqrt2\,T(n/2)+\log n$ | $n^{\log_2\sqrt2}=n^{1/2}$; $f(n)$ polynomially smaller | $\Theta(\sqrt n)$ |

---

## 2. Loops: types & anatomy

### 2.1 Loop constructs
| Construct | Form |
|---|---|
| **for** | `for i ← 1 to n` |
| **while** | `while (cond) { ... }` — test *before* body |
| **do … while** | `do { ... } while (cond);` — test *after* body |
| **repeat … until** | `repeat { ... } until (cond);` |
| **goto** | `x: ...; goto x;` (label + jump back) |

### 2.2 Structural types of loops (flowchart slide)
**Simple loop** · **Nested loop** · **Concatenated (sequential) loop** · **Unstructured loop** (overlapping / jumping, e.g., via `goto`).

### 2.3 Nested loops — Independent vs Dependent

| Feature | **Independent nested** | **Dependent nested** |
|---|---|---|
| Inner bound depends on outer variable? | ❌ No | ✅ Yes |
| Example | `for i←1 to n`<br>`  for j←1 to n` | `for i←1 to n`<br>`  for j←i to n` |
| Total iterations | outer × inner = $n\cdot n = n^2$ | must **sum** over $i$: $n + (n-1)+\dots+1 = \frac{n(n+1)}{2}$ |

> **Sequential (concatenated) loops** → costs **add** (dominant term wins). **Nested independent loops** → costs **multiply**.

### 2.4 Anatomy of a `for` loop
`for (Init; Condition; Update) { S; }`

1. **Init** – executed once
2. **Comparison** – executed (iterations + 1) times
3. **Statement (body)** – executed per iteration
4. **Update** – executed per iteration

For `for i←1 to n`: the loop ends when **i = n+1**; the body runs **n** times, the condition is checked **n+1** times.

**`break` effect:**
- `for i←1 to n { break; }` → body runs **once** → $O(1)$
- `for i←1 to n { for j←1 to n { break; } }` → inner loop costs 1 each time → **$O(n)$** total

---

## 3. Loop-analysis toolkit

**Steps:** (1) find the loop variable's sequence → (2) solve for the last term to get number of iterations $k$ → (3) total iterations $= k+1$ → (4) multiply / sum with other loops → (5) drop constants and lower-order terms.

### Update-pattern cheat-sheet

| Update | Values taken by $i$ | Iterations | Complexity |
|---|---|---|---|
| $i = i+1$ | $1,2,\dots,n$ | $n$ | $O(n)$ |
| $i = i + a$ ($a\ge1$ const) | $1,\,1+a,\,1+2a,\dots$ | $\approx n/a$ | $O(n)$ |
| $i = i \times a$ ($a>1$) | $1,a,a^2,\dots,a^k$ | $\log_a n$ | $O(\log n)$ |
| $i = i / 2$ (from $n$) | $n,\frac n2,\frac n4,\dots,\frac{n}{2^k}$ | $\log_2 n$ | $O(\log n)$ |
| $i = i \times i$ (from 2) | $2^{2^0},2^{2^1},\dots,2^{2^k}$ | $\log\log n$ | $O(\log\log n)$ |
| $i = \sqrt{i}$ (from $n$) | $n^{1/2^0},n^{1/2^1},\dots,n^{1/2^k}$ | $\log\log n$ | $O(\log\log n)$ |
| Condition $i\cdot i\le n$, $i{+}{+}$ | $1,\dots,\sqrt n$ | $\sqrt n$ | $O(\sqrt n)$ |
| $k \mathrel{+}= i$, $i{+}{+}$ (triangular) | $k=1,3,6,\dots,\frac{j(j+1)}2$ | $\approx\sqrt{2n}$ | $O(\sqrt n)$ |

### Key formulas

| Formula | Meaning of variables |
|---|---|
| $\displaystyle \sum_{i=1}^{n} i = \frac{n(n+1)}{2}$ | $n$ = number of terms; used for dependent loops ($j$ from $i$ to $n$) |
| $\displaystyle S=\frac{a\,(1-r^{m})}{1-r}$ | Geometric series: $a$ = first term, $r$ = common ratio, $m$ = number of terms |
| $\displaystyle \sum_{i=0}^{k}\frac{1}{2^i}=2-\frac{1}{2^k}$ | Halving series; with $n=2^k$: $n\sum\frac1{2^i}=2n-1$ |
| $\displaystyle \sum_{x=1}^{n}\frac1x = H_n \approx \log n$ | Harmonic series → gives $n\log n$ for loops with step $j=j+i$ |
| $\displaystyle \frac{j(j+1)}{2}=n \Rightarrow j\approx\sqrt{2n}$ | Triangular-number loop; $j$ = number of iterations |
| $2^{2^k}=n \Rightarrow k=\log\log n$ | Squaring loop; $k$ = number of squarings |
| $n^{1/2^k}=2 \Rightarrow k=\log\log n$ | Square-root loop |
| $\displaystyle \sum_{i=1}^n O(n) = O(n)\cdot\sum_{i=1}^n 1 = O(n^2)$ | Summation view of a nested loop |

---

## 4. Worked examples

> Convention: `c = c + 1` is the unit-cost statement; we count how many times it runs.

### Ex. 1 — single loop
```
for i ← 1 to n
    c = c + 1;
```
$n$ iterations → **$O(n)$**

### Ex. 2 — independent nested, inner bound $n/2$
```
for i ← 1 to n          // n
    for j ← 1 to n/2    // n/2
        c = c + 1;
```
$n\cdot\frac n2=\frac{n^2}{2}$ → **$O(n^2)$**

### Ex. 2b — inner loop runs a constant number of times
```
for i ← 1 to n                        // n
    for j ← n/2 to n by n/2           // j = n/2, n → 2 iterations
        c = c + 1;
```
$n\cdot 2 = 2n$ → **$O(n)$**

### Ex. 3 — `break` in innermost loop
```
for i ← 1 to n            // n
    for j ← 1 to n/4      // n/4
        for k ← 1 to n    // runs only once because of break
            break;
```
$n\cdot\frac n4\cdot1=\frac{n^2}{4}$ → **$O(n^2)$**

### Ex. 4 — innermost loop with constant trip count
```c
for (i = 1; i <= n; ++i)                    // n
    for (j = 1; j <= n; ++j)                // n
        for (k = n/2; k <= n; k += n/2)     // k = n/2, n → 2
            c = c + 1;
```
$n\cdot n\cdot 2 = 2n^2$ → **$O(n^2)$**

### Ex. 5 — doubling `while`
```c
i = 1;
while (i <= n) { i = i * 2; }
```
$i = 1\cdot2^0,\,1\cdot2^1,\dots,1\cdot2^k$; last term $2^k=n\Rightarrow k=\log_2 n$.
Iterations $=k+1$ → **$O(\log n)$**
(Equivalent `for (i=1; i<=n; i=i*2)`.)

### Ex. 6 — halving `while`
```c
i = n;
while (i > 0) { i = i / 2; }
```
$i=\frac n{2^0},\frac n{2^1},\dots,\frac n{2^k}$; $\frac n{2^k}=1\Rightarrow k=\log_2 n$.
Total iterations $=k+1$ → **$O(\log n)$**

### Ex. 8 — linear × logarithmic
```c
for (i = 1; i <= n; ++i)            // n
    for (j = 1; j < n; j = 2*j)     // log n
        c = c + 1;
```
→ **$O(n\log n)$**

### Ex. 9 — bound hidden inside a variable
```c
m = pow(2, n);                          // m = 2^n
for (i = 1; i <= n; ++i)                // n
    for (j = 1; j <= m; j = 2*j)        // log m = log 2^n = n
        c = c + 1;
```
$n\cdot\log m = n\cdot\log_2 2^n = n\cdot n$ → **$O(n^2)$**

### Ex. 11 — summation view
$$f(n)=\sum_{i=1}^{n}O(n)=\sum_{i=1}^{n}\left(\sum_{j=1}^{n}O(1)\right)=O(n)\cdot\sum_{i=1}^{n}1 = O(n^2)$$
- A single `for i←1 to n: c=c+1` is $\sum_{i=1}^n O(1)=O(n)$.
- Wrapping it in another loop over $n$ gives $O(n^2)$.

### Ex. 12 — nested `while` loops, both logarithmic
```c
i = n;
while (i > 0) {              // log n
    j = 1;
    while (j <= n) {         // log n
        j = 2 * j;
    }
    i = i / 2;
}
```
→ $\log n\cdot\log n=$ **$O(\log^2 n)$**

### Ex. 13 — step 1
```c
for (i = 1; i <= n; i = i + 1)  c = c + 1;
```
→ **$O(n)$**  (total iterations $=k+1=O(k)$)

### Ex. 14 — step 2
```c
for (i = 1; i <= n; i = i + 2)  c = c + 1;
```
$i = 1+2\cdot0,\,1+2\cdot1,\dots,1+2k=n\Rightarrow k=\frac{n-1}{2}\approx\frac n2$ → **$O(n)$**

### Ex. 15 — step 5, and the general rule
```c
for (i = 1; i <= n; i = i + 5)  c = c + 1;
```
$k=n/5$ → **$O(n)$**.
**General:** `i = i + a` with constant $a\ge1$ → $k=n/a=O(n)$.

### ⭐ Starred example — harmonic loop
```c
for (i = 1; i <= n; ++i)
    for (j = 1; j <= n; j = j + i)
        c = c + 1;
```
For $i=1,2,3,\dots,n$ the inner loop runs $n,\ \frac n2,\ \frac n3,\dots,\frac nn$ times:

$$n+\frac n2+\frac n3+\dots+\frac nn = n\sum_{x=1}^{n}\frac1x = n\log n \;\Rightarrow\; \mathbf{O(n\log n)}$$

### Ex. 16–18 — multiplicative steps
```c
for (i = 1; i <= n; i = i * 2)  c = c + 1;   // Ex.16
for (i = 1; i <= n; i = i * 3)  c = c + 1;   // Ex.17
for (i = 1; i <= n; i = i * 5)  c = c + 1;   // Ex.18
```
$i=1\cdot a^0,1\cdot a^1,\dots,1\cdot a^k$ ⇒ $k=\log_a n$ ⇒ **$O(\log n)$** for any constant $a$.
(Ex.16 → $\log_2 n$, Ex.17 → $\log_3 n$, Ex.18 → $\log_5 n$.)

### Ex. 19 — squaring
```c
for (i = 2; i <= n; i = i * i)  c = c + 1;
```
| $n$ | values of $i$ | as powers |
|---|---|---|
| 4 | 2, 4 | $2^{2^0},2^{2^1}$ |
| 16 | 2, 4, 16 | $2^{2^0},2^{2^1},2^{2^2}$ |
| general | … | $2^{2^0},\dots,2^{2^k}$ |

$n=2^{2^k}\Rightarrow 2^k=\log n\Rightarrow k=\log\log n$; total iterations $=k+1$ → **$O(\log\log n)$**

### Ex. 20 — square root
```c
for (i = n; i >= 2; i = sqrt(i))  c = c + 1;
```
$i=n^{1/2^0},n^{1/2^1},\dots,n^{1/2^k}$; stop when $n^{1/2^k}=2\Rightarrow k=\log\log n$.
Iterations $=k+1$ → **$O(\log\log n)$**

### Ex. 21 — `i * i <= n`
```c
for (i = 1; i * i <= n; ++i)  c = c + 1;
```
$i^2=n\Rightarrow i=\sqrt n$ → **$O(\sqrt n)$**

### Unnumbered — triangular-sum `while`
```c
k = 1; i = 1;
while (k <= n) {
    i = i + 1;
    k = k + i;
}
```
| $i$ | 1 | 2 | 3 | … | $j$ |
|---|---|---|---|---|---|
| $k$ | 1 | 1+2 | 1+2+3 | … | $1+2+\dots+j$ |

Stop when $\frac{j(j+1)}{2}=n\Rightarrow j^2+j=2n\Rightarrow j\approx\sqrt n$. Time $=O(j)=$ **$O(\sqrt n)$**

### Unnumbered — dependent nested loop (value vs time)
```c
c = 0;
for (i = 1; i <= n; ++i)
    for (j = i; j <= n; ++j)
        c = c + 1;
```
Inner runs $n, n-1,\dots,1$ times:
$$\text{val}(c)=1+2+\dots+n=\frac{n(n+1)}{2},\qquad \text{Time}=\mathbf{O(n^2)}$$

### Ex. 22 — three loops inside one (value of `q` vs time)
```c
int fun(int n)
{
    int i, j, k, p, q = 0;
    for (i = 1; i <= n; ++i)
    {
        p = 0;                                  // a) O(1)
        for (j = n; j > 1; j = j / 2)           // b) log n  → p = log n
            ++p;
        for (k = 1; k < p; k = k * 2)           // c) log p = log log n
            ++q;
    }
    return q;
}
```
| Part | Cost per outer iteration | × $n$ |
|---|---|---|
| a) `p = 0` | 1 | $n$ |
| b) halving loop | $\log n$ (and $p=\log n$) | $n\log n$ |
| c) doubling loop up to $p$ | $\log p=\log\log n$ | $n\log\log n$ |

- **Time** $= n + n\log n + n\log\log n = \mathbf{O(n\log n)}$
- **Value returned** $\text{val}(q)=\mathbf{O(n\log\log n)}$

> ⚠️ **Time ≠ value.** Time counts all loop work; `q` only counts part (c).

### Ex. 23 — sequential loops inside an outer loop
```c
for (i = 1; i <= n; ++i)
{
    j = 1;                         // a) 1      → n   total
    while (j <= n) j = 2 * j;      // b) log n  → n log n total
    for (k = 1; k <= n; ++k)       // c) n      → n²  total
        c = c + 1;
}
```
$T(n)=n+n\log n+n^2=\mathbf{O(n^2)}$

### Ex. 24 — three-level mix
```c
for (i = 2; i <= n; i = i * i)           // log log n
    for (j = 1; j <= n; ++j)             // ┐ together: n log n
        for (k = 1; k <= n; k = k + j)   // ┘ (harmonic: n/j)
            c = c + 1;
```
Middle + inner = $\sum_{j=1}^n \frac nj = n\log n$; times outer $\log\log n$:
$$\mathbf{O(n\log n\cdot\log\log n)}$$

### Ex. 25 — number of `*` printed ($n=2^{2^k}$)
```c
n = 2^(2^k);
for (i = 1; i <= n; ++i)
{
    j = 2;
    while (j <= n)
    {
        j = j * j;
        printf("*");
    }
}
```
| $k$ | $n$ | $j$ values | stars per outer iteration |
|---|---|---|---|
| 1 | 4 | 2, 4 | 2 |
| 2 | 16 | 2, 4, 16 | 3 |
| $k$ | $2^{2^k}$ | $2^{2^0},\dots,2^{2^k}$ | $k+1$ |

$$\text{Total '*'}=n(k+1)=n(\log\log n+1)$$

---

## 5. Best-case vs worst-case: Example 26

```c
A[1..n]  : array of binary;
f(m) = Θ(m);          // cost of f is linear in its argument
count : integer;
count = 1;
for i = 1 to n
{
    if (A[i] == 1) count++;
    else {
        f(count);
        count = 1;        // ← present in version (a), struck out in version (b)
    }
}
```

### Version (a) — with `count = 1` reset
Every `f(count)` call is on the run length just ended, so total $f$-cost ≈ sum of run lengths ≤ $n$.
**Best case = Worst case = $O(n)$.**

### Version (b) — `count = 1` removed (struck out on the slide)
`count` never resets, so `f(count)` can be expensive on every `0`.

| Case | Input pattern | Cost |
|---|---|---|
| I | all $A[i]=1$ | $O(n)$ |
| II | all $A[i]=0$ (`count` stays 1, `f(1)=O(1)`) | $O(n)$ |
| III | $A[i]=1$ for $i=1..n-1$, $A[n]=0$ | $(n-1)+(n-1)+1 = O(n)$ |
| IV | first $n/2$ are 1, last $n/2$ are 0 | $\frac n2+\frac n2\cdot\frac n2 = \mathbf{O(n^2)}$ |

- **Best case (B.C.) = $O(n)$**
- **Worst case (W.C.) = $O(n^2)$** (Case IV: `count`≈$n/2$ and `f` is called $n/2$ times)

*(The slide leaves "B.C. / W.C." blank for version (b); the table above is the whiteboard's case analysis.)*

---

## 6. GATE PYQs on loops

### PYQ-1 — Return value of `unknown(n)`
```c
int unknown(int n)
{
    int i, j, k = 0;
    for (i = n/2; i <= n; i++)
        for (j = 2; j <= n; j = j * 2)
            k = k + n/2;
    return (k);
}
```
The return value is ______.

| Option | Answer | Verdict |
|---|---|---|
| A | $\Theta(n^2)$ | ❌ Misses the $\log n$ from the doubling inner loop. |
| B | $\Theta(n^2\log n)$ | ✅ **Correct** |
| C | $\Theta(n^3)$ | ❌ Treats `k += n/2` as adding $n$ times, not as one add of size $n/2$. |
| D | $\Theta(n^3\log n)$ | ❌ Same overcount as C plus the $\log n$. |

**Why B:** outer runs $\approx n/2$ times, inner $\log n$ times, each adds $n/2$:
$\frac n2\cdot\log n\cdot\frac n2=\frac{n^2\log n}{4}=\Theta(n^2\log n)$.

---

### PYQ-2 — GATE 2023 CS (2M, **MSQ**) — Function_1 vs Function_2
```
Function_1                         Function_2
  while n > 1 do                     for i = 1 to 100 * n do
      for i = 1 to n do                  x = x + 1;
          x = x + 1;                 end for
      end for
      n = ⌊n/2⌋;
  end while
```
Let $f_1(n), f_2(n)$ = number of times `x = x + 1` executes. Which is/are TRUE?

| Option | Statement | Verdict |
|---|---|---|
| A | $f_1(n)\in\Theta(f_2(n))$ | ✅ **True** — both are $\Theta(n)$ |
| B | $f_1(n)\in o(f_2(n))$ | ❌ little-o needs strictly smaller growth; they grow at the same rate |
| C | $f_1(n)\in\omega(f_2(n))$ | ❌ little-omega needs strictly larger growth; not the case |
| D | $f_1(n)\in O(n)$ | ✅ **True** — $f_1\approx 2n$ |

**Counting $f_1$:** $n+\frac n2+\frac n4+\dots+\frac n{2^k}$ with $n=2^k$:

$$f_1 = n\sum_{i=0}^{k}\frac1{2^i}=n\left(2-\frac1{2^k}\right)=2n-1=\Theta(n),\qquad f_2=100n=\Theta(n)$$

> ⚠️ The board simplification ends with "$n-1$"; the correct geometric sum is $2n-1$ (the $\frac1{1-r}=2$ factor was dropped). **The Θ(n) conclusion is unaffected.**

**Correct answers: A and D.**

---

### PYQ-3 — `Is_Prime(n)`
```c
int Is_Prime(int n)
{
    int i;
    for (i = 2; i <= sqrt(n); i++)
        if (n % i == 0)
        {
            printf("Not Prime.\n");
            return 0;
        }
    return 1;
}
```
Let $T(n)$ = number of times the `for` loop executes on input $n$. Which is true?

| Option | Statement | Verdict |
|---|---|---|
| A | $T(n)=O(\sqrt n)$ and $T(n)=\Omega(\sqrt n)$ | ❌ Best case is 1 iteration (e.g., even $n$ exits at $i=2$), so $\Omega(\sqrt n)$ fails. |
| B | $T(n)=O(\sqrt n)$ and $T(n)=\Omega(1)$ | ✅ **Correct** |
| C | $T(n)=O(n)$ and $T(n)=\Omega(\sqrt n)$ | ❌ $\Omega(\sqrt n)$ fails for the same reason. |
| D | None of these | ❌ B is valid. |

**Reasoning:** **Min = 1** iteration (n divisible by 2) · **Max = $\sqrt n$** iterations (n prime) ⇒ $\Omega(1)$ and $O(\sqrt n)$.

---

### PYQ-4 — Time complexity of `fun` in Θ notation
```c
int fun(int n)
{
    int i, j;
    for (i = 1; i <= n; i++) {
        for (j = 1; j < n; j += i) {
            printf("%d %d", i, j);
        }
    }
}
```

| Option | Answer | Verdict |
|---|---|---|
| A | $\Theta(n\sqrt n)$ | ❌ No $\sqrt n$ structure; sum is harmonic. |
| B | $\Theta(n^2)$ | ❌ Would need the inner loop to run $n$ times for every $i$; it runs only $\approx n/i$. |
| C | $\Theta(n\log n)$ | ✅ **Correct** |
| D | $\Theta(n^2\log n)$ | ❌ Overcounts by a factor of $n$. |

**Why C:** inner loop runs $\approx\frac ni$ times ⇒ $\sum_{i=1}^n\frac ni=n H_n=\Theta(n\log n)$.

---

### PYQ-5 — Value of `j` after the loop
```c
for (i = n, j = 0; i > 0; i /= 2, j += i);
```
Let $\text{val}(j)$ = value stored in $j$ after termination. Which is true?

| Option | Statement | Verdict |
|---|---|---|
| A | $\text{val}(j)=\Theta(\log n)$ | ❌ $\log n$ is the **number of iterations** (time), not the value accumulated. |
| B | $\text{val}(j)=\Theta(\sqrt n)$ | ❌ Nothing here produces a square root; $j$ is a halving geometric sum. |
| C | $\text{val}(j)=\Theta(n)$ | ✅ **Correct** |
| D | $\text{val}(j)=\Theta(n\log n)$ | ❌ Would need $n$ added $\log n$ times; terms here shrink geometrically. |

**Why C:** $j = n+\frac n2+\frac n4+\dots+\frac n{2^k}\approx 2n=\Theta(n)$ (strictly, because `i /= 2` executes before `j += i`, the sum is $\frac n2+\frac n4+\dots\approx n-1$ — still $\Theta(n)$).
**Time $=\Theta(\log n)$ but value $=\Theta(n)$.**

---

## 7. Homework (Ex. 30)

**Q.** Count the number of `*` printed:
```
for i ← 1 to (n − 1)
    for j ← (i + 1) to n
        for k ← 1 to j
            print("*");
```
*(Left as H/W in the lecture; solution below is mine.)*

For fixed $j$, the number of $i$ values is $j-1$ ($i=1..j-1$), and each prints $j$ stars:

$$\sum_{j=2}^{n} j(j-1)=\sum_{j=1}^{n}j^2-\sum_{j=1}^{n}j=\frac{n(n+1)(2n+1)}{6}-\frac{n(n+1)}{2}=\frac{(n-1)\,n\,(n+1)}{3}=\frac{n^3-n}{3}=\Theta(n^3)$$

Sanity check $n=2$: only $i=1,j=2$ → 2 stars; formula gives $\frac{1\cdot2\cdot3}{3}=2$ ✓.

---

## 8. Divide & Conquer

### 8.1 Idea (3-step strategy)
> **Recursively break a problem into smaller sub-problems, solve them independently, then combine the results.**

| Step | What happens |
|---|---|
| **1. DIVIDE** | Split problem $P$ into $k$ smaller sub-problems $P_1,\dots,P_k$ (each strictly smaller than $P$). |
| **2. CONQUER** | Recursively solve each sub-problem with D&C. If `Small(P)` is true → solve **directly** (base case). |
| **3. COMBINE** | Merge sub-solutions to get the answer for $P$. |

**Whiteboard illustration:** array $A=\langle 8,10,2,4\,|\,12,18,20,6\rangle$ is halved into $L_1, L_2$, then into pairs; each pair is solved directly ($l_1, l_2, \dots$), then results are combined upward ($l_3=$ combine($l_1,l_2$), $l_4$ likewise, and finally $L$ = combine($l_3,l_4$)).

### 8.2 General algorithm
```c
Algorithm DAndC(P)
{
    if (Small(P)) return S(P);          // base case
    else {
        Divide P into smaller instances P1, P2, ..., Pk;   // k ≥ 1
        Apply DAndC to each sub-problem;
        return Combine(DAndC(P1), DAndC(P2), ..., DAndC(Pk));
    }
}
```
*(One slide writes $k>1$, the other $k\ge1$ — for true "divide" you split into at least 2 parts; $k=1$ is a degenerate "decrease-and-conquer" case.)*

### 8.3 Control abstraction (array form)
```
Procedure DANDC(p, q)
    global n, A(1:n); integer m, p, q;       // 1 ≤ p ≤ q ≤ n
    if SMALL(p, q)
        then return G(p, q)
    else
        m ← DIVIDE(p, q)                      // p ≤ m < q
        return COMBINE( DANDC(p, m), DANDC(m + 1, q) )
    endif
end DANDC
```
- Current sub-array: $A[p..q]$, **size** $n=q-p+1$.
- Split point $m$ → sub-problems $A(p:m)$ and $A(m+1:q)$.
- **`COMBINE` is optional** (e.g., binary search needs no combine step).
- `SMALL(p,q)` = base-case test; `G(p,q)` = direct solution for small input.

### 8.4 Recurrence for D&C

If the two sub-problems are roughly equal in size:

$$T(n)=\begin{cases} g(n), & n \text{ small}\\[4pt] 2\,T\!\left(\dfrac n2\right)+f(n), & \text{otherwise}\end{cases}$$

General form with $k$ sub-problems of sizes $n_1,\dots,n_k$:

$$T(n)=T(n_1)+T(n_2)+\dots+T(n_k)+f(n)$$

| Symbol | Meaning |
|---|---|
| $T(n)$ | Time of DANDC on $n$ inputs |
| $g(n)$ | Time to solve a **small** instance directly |
| $f(n)$ | Time for **DIVIDE + COMBINE** |
| $n_1..n_k$ | Sizes of the $k$ sub-problems after division |

---

## 9. Memory tricks / patterns

The lecture has no verbal analogies on the slides, but the board repeatedly uses these patterns — worth memorising:

- **"Multiply nested, add sequential."** Independent nested loops multiply; back-to-back loops add (then keep the dominant term).
- **Dependent loop? → write the iteration counts as a series** ($n, n-1, \dots, 1$) and sum.
- **Last term trick:** write the loop variable's sequence, set the **last term = n**, solve for $k$, then **iterations = k + 1**.
- **Multiply by $a$ → $\log_a n$** · **Add $a$ → $n/a$** · **Square → $\log\log n$** · **Square-root → $\log\log n$** · **$i\cdot i\le n$ → $\sqrt n$.**
- **Step $j = j + i$ with $i=1..n$ → harmonic → $n\log n$** (the ⭐ example).
- **Condition runs once more than the body:** body $n$ times ⇒ condition $n+1$ times.
- **`break` in a loop ⇒ that loop costs 1.**
- **Constant trip count** (e.g., `k = n/2; k <= n; k += n/2`) ⇒ factor of **2**, not $n$.
- **Time ≠ value.** Always ask: is the question about *running time* or the *value of a variable*? (PYQ-5: time $\log n$, value $n$; Ex. 22: time $n\log n$, value $n\log\log n$.)
- **D&C = Divide → Conquer → Combine**; base case = `Small(P)`.

---

## 10. Quick revision checklist

**Recurrences**
- [ ] Solve $T(n)=2T(n/2)+\log n$ by substitution → $\Theta(n)$ (and by Master Theorem)
- [ ] $\sum 2^i = 2^k-1$ and $\sum i\,2^i=(k-2)2^k+2$
- [ ] Attempt the 3 practice recurrences ($\sqrt n$ recursion, $n/\log n$, $\sqrt2\,T(n/2)$)

**Loop basics**
- [ ] 5 loop constructs: for, while, do-while, repeat-until, goto
- [ ] 4 structural types: simple, nested, concatenated, unstructured
- [ ] Independent vs dependent nested loops
- [ ] Four parts of a `for`: init, comparison, statement, update (condition runs one extra time)
- [ ] Effect of `break` on iteration count

**Loop patterns**
- [ ] $i{+}{+}$ → $O(n)$; $i{+}{=}a$ → $n/a$ → $O(n)$
- [ ] $i{*}{=}a$ → $\log_a n$; $i{/}{=}2$ → $\log n$
- [ ] $i=i\cdot i$ and $i=\sqrt i$ → $\log\log n$
- [ ] $i\cdot i\le n$ → $\sqrt n$
- [ ] Triangular `k += i` loop → $\sqrt n$
- [ ] Dependent loop $j=i..n$ → $\frac{n(n+1)}2$ → $O(n^2)$
- [ ] Harmonic loop $j{+}{=}i$ → $n\log n$
- [ ] Loop bound inside a variable ($m=2^n$) → $\log m = n$
- [ ] Nested logs → $\log^2 n$; $n\log n\cdot\log\log n$ (Ex. 24)
- [ ] Sequential loops inside outer loop (Ex. 23) → sum, keep dominant
- [ ] Time vs returned value (Ex. 22, PYQ-5)
- [ ] Stars printed with $n=2^{2^k}$ → $n(\log\log n+1)$
- [ ] Best vs worst case with and without `count = 1` reset (Ex. 26)

**GATE PYQs**
- [ ] 2014: $T(n)=2T(n/2)+\log n$ → $\Theta(n)$
- [ ] `unknown(n)` → $\Theta(n^2\log n)$
- [ ] 2023 MSQ: $f_1\in\Theta(f_2)$ and $f_1\in O(n)$ (A, D); $2n-1$ sum
- [ ] `Is_Prime`: $O(\sqrt n)$ and $\Omega(1)$ (option B)
- [ ] `j += i` harmonic loop → $\Theta(n\log n)$
- [ ] `i /= 2, j += i` → $\text{val}(j)=\Theta(n)$

**Homework**
- [ ] Triple-loop star count $=\frac{n^3-n}{3}=\Theta(n^3)$

**Divide & Conquer**
- [ ] 3 steps: Divide, Conquer, Combine
- [ ] `DAndC(P)` algorithm with `Small(P)`, `S(P)`
- [ ] Control abstraction `DANDC(p,q)`: `SMALL`, `G`, `DIVIDE`, `COMBINE` (optional), size $n=q-p+1$
- [ ] Recurrence $T(n)=g(n)$ / $2T(n/2)+f(n)$ and meaning of $g$, $f$, $n_1..n_k$

**Not in this PDF (revise from other lectures)**
- [ ] Asymptotic notation theory · [ ] Recursive-algorithm analysis (Master theorem, recursion tree) · [ ] Space complexity
