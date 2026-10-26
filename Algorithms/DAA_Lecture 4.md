# Design & Analysis of Algorithms — Lecture 04
## Analysis of Recursive Algorithms (Recurrence Relations & Back-Substitution)

> **Source:** GeeksforGeeks GATE — *Design & Analysis of Algorithms*, Lecture 04 (Analysis of Algorithms), instructor Khaleel Khan.
> **Note:** The lecture's agenda slide lists *Asymptotic Notations, Recursive Algorithms, Loop Analysis, Space Complexity*, but the slides actually cover **recursive algorithm analysis** (time + a bit of space) and GATE PYQs on recurrences. Items marked 🔹 are my additions (not in the lecture).

---

## 📑 Table of Contents

1. [Method: Back-Substitution (Iteration)](#1-method-back-substitution-iteration)
2. [Framework: Linear recursion `What(n)` with different `B(n)`](#2-framework-linear-recursion-whatn-with-different-bn)
3. [Branching recursion: `Do_It(n)` → exponential](#3-branching-recursion-do_itn--exponential)
4. [Square-root recursions → log log n](#4-square-root-recursions--log-log-n)
5. [Divide-by-2 recursions `Recur(n)`](#5-divide-by-2-recursions-recurn)
6. [Tricky recurrence: $T(n)=\sqrt n\,T(\sqrt n)+n$](#6-tricky-recurrence-tn--sqrtn-tsqrtn--n)
7. [Flowchart question (best vs worst case)](#7-flowchart-question-best-vs-worst-case)
8. [Code-reading question: `abc(char *s)`](#8-code-reading-question-abcchar-s)
9. [Master Cheat-Table of all results](#9-master-cheat-table)
10. [GATE PYQs](#10-gate-pyqs)
11. [Memory tricks](#11-memory-tricks)
12. [Quick Revision Checklist](#12-quick-revision-checklist)

---

## 1. Method: Back-Substitution (Iteration)

**Steps (used in every example in this lecture):**

1. Write the recurrence from the code: **base case** (constant $c$) + **recursive case**.
2. Combine constants: all non-recursive $O(1)$ statements ($a$, $b$, …) merge into one constant, e.g. $a + b = d$.
3. **Expand** repeatedly: $T(n) \to T(n-1) \to T(n-2) \dots$ and spot the pattern after $k$ steps.
4. Write the general **$k$-th step** form.
5. Find $k$ by equating the argument to the **base case** (e.g. $n-k=1$, $\frac{n}{2^k}=1$, $n^{1/2^k}=2$).
6. Substitute $k$ back → closed form → keep the **dominant term** → Big-O.

**Dominant-term rule (from the lecture's side notes):**

| Expression | Order |
|---|---|
| $a + n$ | $O(n)$ |
| $n^2 + n + a$ | $O(n^2)$ |

---

## 2. Framework: Linear recursion `What(n)` with different `B(n)`

```c
Algorithm What(n)
{
    if (n == 1) return;
    else
    {
        What(n - 1);
        B(n);          // cost depends on the case
    }
}
```

The recurrence is $T(n) = T(n-1) + \text{cost of } B(n)$. Result depends on how expensive $B(n)$ is:

| Case | $B(n)$ | Recurrence | Result |
|---|---|---|---|
| **I** | $O(1)$ | $T(n) = T(n-1) + c$ | $O(n)$ |
| **II** | $O(n)$ | $T(n) = T(n-1) + (a + n)$ | $O(n^2)$ |
| **III** ⭐ | $O(1/n)$ | $T(n) = T(n-1) + \left(\frac{1}{n} + a\right)$ | $O(n)$ |

### Case III in detail (⭐ starred in lecture)

$$T(n) = \begin{cases} c, & n = 1 \\ T(n-1) + \dfrac{1}{n} + a, & n > 1 \end{cases}$$

Expansion:

- $T(n) = T(n-1) + \frac{1}{n} + a$
- $T(n-1) = T(n-2) + \frac{1}{n-1} + a$
- $T(n) = T(n-2) + \frac{1}{n-1} + \frac{1}{n} + 2a$
- … $T(n) = T(n-k) + \frac{1}{n-(k-1)} + \dots + \frac{1}{n} + ka$

Set $n - k = 1$ → $k = n-1$:

$$T(n) = T(1) + \frac12 + \frac13 + \dots + \frac1n + a(n-1) = \sum_{x=1}^{n}\frac{1}{x} + an - a$$

$$\boxed{T(n) = \log n + an - a = O(n)}$$

- **Key fact:** harmonic sum $\sum_{x=1}^{n} \frac1x \approx \log n$ (🔹 more precisely $\ln n + O(1)$, i.e. $\Theta(\log n)$).
- The linear term $an$ dominates → **$O(n)$**.

**Space (🔹 derived):** depth of recursion is $n$ → stack space $O(n)$.

---

## 3. Branching recursion: `Do_It(n)` → exponential

```c
Algorithm Do_It(n)
{
    if (n == 1) return;
    else
        return (Do_It(n - 1) + Do_It(n - 1));   // two calls, v1 + v2
}
```

**Recurrence:**

$$T(n) = \begin{cases} c, & n = 1 \\ a + T(n-1) + T(n-1) + b, & n > 1 \end{cases} \;\Rightarrow\; T(n) = 2T(n-1) + d$$

**Back-substitution:**

| Step | Form |
|---|---|
| ① | $T(n) = 2T(n-1) + d$ |
| ② | $T(n-1) = 2T(n-2) + d$ |
| ③ | $T(n) = 4T(n-2) + 3d$ |
| ④ | $T(n) = 8T(n-3) + 7d$ |
| general | $T(n) = 2^k\,T(n-k) + (2^k - 1)\,d$ |

Base case: $n - k = 1 \Rightarrow k = n-1$

$$T(n) = 2^{n-1}T(1) + (2^{n-1} - 1)d = 2^{n-1}c + 2^{n-1}d - d = \frac{c\cdot 2^n}{2} + \frac{d\cdot 2^n}{2} - d$$

$$\boxed{T(n) = O(2^n)}$$

**Space (🔹 derived):** max recursion depth $n-1$ → $O(n)$ (even though time is exponential).

---

## 4. Square-root recursions → log log n

### 4a. Single call: `A(√n)`

```c
Algorithm A(n)
{
    if (n == 2) return;
    else
        return A(sqrt(n));
}
```

$$T(n) = \begin{cases} c, & n = 2 \\ a + T(\sqrt n) + b, & n > 2 \end{cases} \;\Rightarrow\; T(n) = T(n^{1/2}) + d$$

Back-substitution:

- $T(n) = T(n^{1/2}) + d$
- $T(n^{1/2}) = T(n^{1/4}) + d$
- $T(n) = T(n^{1/4}) + 2d = T(n^{1/2^2}) + 2d = T(n^{1/2^3}) + 3d$
- general: $T(n) = T\!\left(n^{1/2^k}\right) + k\,d$

**Finding $k$:** stop when $n^{1/2^k} = 2$

$$\frac{1}{2^k}\log_2 n = \log_2 2 = 1 \;\Rightarrow\; 2^k = \log_2 n \;\Rightarrow\; k = \log_2\log_2 n$$

$$T(n) = T(2) + d\log\log n = c + d\log\log n \;\Rightarrow\; \boxed{T(n) = O(\log\log n)}$$

**Space:** $O(\log\log n)$ (recursion depth = $k$).

### 4b. Two calls: `A(√n) + A(√n)`

```c
Algorithm A(n)
{
    if (n == 2) return;
    else
        return (A(sqrt(n)) + A(sqrt(n)));
}
```

$$T(n) = \begin{cases} c, & n = 2 \\ a + 2T(\sqrt n) + b, & n > 2 \end{cases} \;\Rightarrow\; T(n) = 2T(\sqrt n) + d$$

Back-substitution:

- $T(n) = 2T(n^{1/2}) + d$
- $T(n) = 4\,T(n^{1/4}) + 3d$
- general: $T(n) = 2^k\,T\!\left(n^{1/2^k}\right) + (2^k - 1)\,d$

With $n^{1/2^k} = 2 \Rightarrow 2^k = \log n,\; k = \log\log n$:

$$T(n) = 2^k\,T(2) + (2^k - 1)d = c\log n + d\log n - d$$

$$\boxed{T(n) = O(\log n)}$$

> **Side identity (from lecture, yellow note):** $2^{\log_2 \log_2 n} = \log_2 n$ — this is why $2^k$ becomes $\log n$.

**Space:** $O(\log\log n)$ (depth is still $k$).

### 4a vs 4b comparison

| | 4a: `A(√n)` | 4b: `A(√n) + A(√n)` |
|---|---|---|
| Recurrence | $T(n) = T(\sqrt n) + d$ | $T(n) = 2T(\sqrt n) + d$ |
| Number of levels $k$ | $\log\log n$ | $\log\log n$ |
| Work per level | constant | doubles each level |
| Total time | $O(\log\log n)$ | $O(\log n)$ |
| Space | $O(\log\log n)$ | $O(\log\log n)$ |

---

## 5. Divide-by-2 recursions `Recur(n)`

For all of these: $\dfrac{n}{2^k} = 1 \Rightarrow n = 2^k \Rightarrow k = \log_2 n$.

### 5a. One recursive call: `Recur(n/2); B(n);`

```c
Algorithm Recur(n)
{
    if (n == 1) return;
    else
    {
        Recur(n / 2);
        B(n);
    }
}
```

**Space (both cases):** $O(\log n)$ (depth $= k = \log n$).

| Case | $B(n)$ | Recurrence | Result |
|---|---|---|---|
| **I** | $O(1)$ | $T(n) = T(n/2) + a$ | $O(\log n)$ |
| **II** | $O(n)$ | $T(n) = T(n/2) + (n + a) \approx T(n/2) + n$ | $O(n)$ |

**Case I working:**

$$T(n) = T(n/2) + a = T(n/4) + 2a = T(n/8) + 3a = \dots = T\!\left(\tfrac{n}{2^k}\right) + ka = T(1) + a\log n = c + a\log n$$

$$\boxed{O(\log n)}$$

**Case II working:**

$$T(n) = T(n/2) + n,\quad T(n/2) = T(n/4) + \tfrac n2$$

$$T(n) = T(n/4) + \tfrac n2 + n = T(n/8) + \tfrac n4 + \tfrac n2 + n = \dots = T\!\left(\tfrac{n}{2^k}\right) + n\sum_{i=0}^{k-1}\frac{1}{2^i}$$

Geometric series: $\;\sum_{i=0}^{k-1}\frac{1}{2^i} = \dfrac{1 - (1/2)^k}{1 - 1/2} = 2\left(1 - \frac{1}{2^k}\right)$

$$T(n) = c + 2n\left(1 - \tfrac{1}{2^k}\right) = c + 2n - 2 = \boxed{O(n)}$$

> ⚠️ **Lecture slip:** the handwritten steps drop the factor 2 from dividing by $\frac12$ and write $c + n - 1$. The **final answer $O(n)$ is unaffected**, but the exact closed form is $c + 2n - 2$.

**Geometric series formula (from lecture):**

$$S = \frac{a(1 - r^n)}{1 - r}, \qquad \sum_{i=1}^{n}\frac{1}{2^i} = 1 - \frac{1}{2^n}$$

- $a$ = first term, $r$ = common ratio, $n$ = number of terms.

### 5b. Two recursive calls: `Recur(n/2); Recur(n/2); B(n);`

```c
Algorithm Recur(n)
{
    if (n == 1) return;
    else
    {
        Recur(n / 2);
        Recur(n / 2);
        B(n);
    }
}
```

**Space (both cases):** $O(k) = O(\log n)$.

| Case | $B(n)$ | Recurrence | General form after $k$ steps | Result |
|---|---|---|---|---|
| **I** | $O(1)$ | $T(n) = 2T(n/2) + a$ | $2^k\,T(n/2^k) + (2^k - 1)a$ | $O(n)$ |
| **II** | $O(n)$ | $T(n) = 2T(n/2) + (n + a)$ | $2^k\,T(n/2^k) + k\cdot n$ | $O(n\log n)$ |

**Case I working** ($n = 2^k$):

- $T(n) = 2T(n/2) + a$
- $= 4T(n/4) + 3a$
- $= 2^2 T(n/2^2) + (2^2 - 1)a$
- $= 2^k T(n/2^k) + (2^k - 1)a = n\cdot c + a\cdot n - a = \boxed{O(n)}$

**Case II working:**

- $T(n) = 2T(n/2) + n$
- $T(n/2) = 2T(n/4) + n/2$
- $T(n) = 2[2T(n/4) + n/2] + n = 4T(n/4) + 2n$
- $= 8T(n/8) + 3n = 2^3\,T(n/2^3) + 3n$
- $= 2^k\,T(n/2^k) + k\,n = n\,T(1) + n\log n = cn + n\log n = \boxed{O(n\log n)}$

### 5 — Side-by-side summary (🔹 comparison)

| | 1 call `T(n/2)` | 2 calls `2T(n/2)` |
|---|---|---|
| $B(n) = O(1)$ | $O(\log n)$ | $O(n)$ |
| $B(n) = O(n)$ | $O(n)$ | $O(n\log n)$ |
| Space | $O(\log n)$ | $O(\log n)$ |

---

## 6. Tricky recurrence: $T(n) = \sqrt n\, T(\sqrt n) + n$

$$T(n) = \begin{cases} 2, & n = 2 \\ \sqrt n \cdot T(\sqrt n) + n, & n > 2 \end{cases}$$

**Back-substitution:**

- ① $T(n) = n^{1/2}\,T(n^{1/2}) + n$
- ② $T(n^{1/2}) = n^{1/4}\,T(n^{1/4}) + n^{1/2}$
- ③ $T(n) = n^{1/2}\left[n^{1/4}T(n^{1/4}) + n^{1/2}\right] + n = n^{3/4}\,T(n^{1/4}) + 2n$
- $= n^{1 - \frac{1}{2^2}}\,T(n^{1/2^2}) + 2n$
- $= n^{1 - \frac{1}{2^3}}\,T(n^{1/2^3}) + 3n$
- general: $T(n) = n^{1 - \frac{1}{2^k}}\;T\!\left(n^{1/2^k}\right) + k\,n$

**Base case:** $n^{1/2^k} = 2 \Rightarrow 2^k = \log n \Rightarrow k = \log\log n$

$$T(n) = \frac{n}{n^{1/2^k}}\cdot T(2) + k\,n = \frac{n}{2}\cdot 2 + n\log\log n$$

$$\boxed{T(n) = n + n\log\log n = O(n\log\log n)}$$

---

## 7. Flowchart question (best vs worst case)

**Question:** The flowchart represents recursive algorithm $A(n)$. All statements except the recursive calls are $O(1)$. Find the **best-case and worst-case** time.

**How to read it:** count the number of `A(n/2)` calls on each path.

| Case | Path | # of `A(n/2)` calls | Recurrence | Result |
|---|---|---|---|---|
| **Best** | First decision = *yes* → exits after the 1st call + 1 more call | **2** | $T(n) = 2T(n/2) + c$ | $O(n)$ ✅ |
| **Worst** | Never exits early → goes through all blocks | **8** | $T(n) = 8T(n/2) + c$ | $O(n^3)$ ✅ |

**Answer:** Best case $= O(n)$, Worst case $= O(n^3)$.

🔹 Check via Master theorem: $n^{\log_2 8} = n^3$ dominates $f(n) = c$; $n^{\log_2 2} = n$ dominates $f(n) = c$.

---

## 8. Code-reading question: `abc(char *s)`

```c
void abc(char *s)
{
    if (*s != '\0')
    {
        printf("%c", *s);
        abc(s + 1);
        abc(s + 1);
    }
}
```

**Question:** If `s` points to a string of length $n$, what is the time complexity of `abc`?

- `s` holds the address (e.g. 2000) of the first char; `s+1` points to the rest, i.e. a string of length $n-1$.
- Each call: 1 `printf` (constant) + **2** recursive calls on size $n-1$.

$$T(n) = \begin{cases} c, & n = 1 \\ a + b + T(n-1) + T(n-1), & n > 1 \end{cases} \;\Rightarrow\; T(n) = 2T(n-1) + a$$

$$\boxed{T(n) = O(2^n)}$$

> Same shape as `Do_It(n)` in Section 3.

---

## 9. Master cheat-table

| # | Algorithm / Recurrence | Time | Space |
|---|---|---|---|
| 1-I | $T(n) = T(n-1) + c$ | $O(n)$ | 🔹 $O(n)$ |
| 1-II | $T(n) = T(n-1) + (a+n)$ | $O(n^2)$ | 🔹 $O(n)$ |
| 1-III | $T(n) = T(n-1) + \frac1n + a$ | $O(n)$ | 🔹 $O(n)$ |
| 2 | $T(n) = 2T(n-1) + d$ (`Do_It`) | $O(2^n)$ | 🔹 $O(n)$ |
| 3 | $T(n) = T(\sqrt n) + d$ | $O(\log\log n)$ | $O(\log\log n)$ |
| 4 | $T(n) = 2T(\sqrt n) + d$ | $O(\log n)$ | $O(\log\log n)$ |
| 5a-I | $T(n) = T(n/2) + a$ | $O(\log n)$ | $O(\log n)$ |
| 5a-II | $T(n) = T(n/2) + n$ | $O(n)$ | $O(\log n)$ |
| 5b-I | $T(n) = 2T(n/2) + a$ | $O(n)$ | $O(\log n)$ |
| 5b-II | $T(n) = 2T(n/2) + n$ | $O(n\log n)$ | $O(\log n)$ |
| 6 | $T(n) = \sqrt n\,T(\sqrt n) + n$ | $O(n\log\log n)$ | — |
| 7 | Flowchart (2 vs 8 calls of $T(n/2)$) | Best $O(n)$, Worst $O(n^3)$ | — |
| 8 | `abc(s)`: $T(n) = 2T(n-1) + a$ | $O(2^n)$ | — |

**Useful formulas used in this lecture**

| Formula | Meaning |
|---|---|
| $\sum_{x=1}^{n}\frac1x \approx \log n$ | Harmonic series |
| $S = \dfrac{a(1-r^n)}{1-r}$ | Geometric series ($a$ = 1st term, $r$ = ratio, $n$ = terms) |
| $2^{\log_2\log_2 n} = \log_2 n$ | Used in sqrt-recursion with 2 calls |
| $n^{1/2^k} = 2 \Rightarrow k = \log_2\log_2 n$ | Depth of $\sqrt{}$ recursion |
| $\frac{n}{2^k} = 1 \Rightarrow k = \log_2 n$ | Depth of halving recursion |
| $\log\sqrt n = \frac12\log n$ | Used in the MSQ |

---

## 10. GATE PYQs

### 🅷/W 1 — GATE 2014 (1 Mark)

**Q.** Which one of the following correctly determines the solution of the recurrence relation with $T(1) = 1$?

$$T(n) = 2T\!\left(\frac n2\right) + \log n$$

| Option | Answer |
|---|---|
| **A** | $\Theta(n)$ ✅ |
| B | $\Theta(n\log n)$ |
| C | $\Theta(n^2)$ |
| D | $\Theta(\log n)$ |

> 🔹 *Left as homework in the lecture (no answer shown); solved here via the Master theorem.*

- **Why A:** $a=2,\,b=2$ → $n^{\log_b a} = n$. $f(n) = \log n = O(n^{1-\epsilon})$ → Master Case 1 → $\Theta(n)$.
- **Why not B:** $n\log n$ needs $f(n) = \Theta(n)$ (Case 2); here $f(n)$ is only $\log n$.
- **Why not C:** $n^2$ is far bigger than anything this recurrence generates (only $n$ leaves, each with tiny extra work).
- **Why not D:** $\log n$ is too small — the $n$ leaves alone already contribute $\Theta(n)$.

---

### 🅷/W 2 — Recurrence with floor of square root

**Q.** $T(1) = 1$, and
$$T(n+1) = T(n) + \left\lfloor\sqrt{n+1}\,\right\rfloor \quad \text{for all } n \ge 1.$$
The value of $T(m^2)$ for $m \ge 1$ is:

| Option | Expression |
|---|---|
| A | $\dfrac m6(21m - 39) + 4$ |
| **B** | $\dfrac m6\left(4m^2 - 3m + 5\right)$ ✅ |
| C | $\dfrac m2\left(3m^{2.5} - 11m + 20\right) - 5$ |
| D | $\dfrac m6\left(5m^3 - 34m^2 + 137m - 104\right) - \dfrac56$ |

> 🔹 *Left as homework in the lecture; solved here.*

**Solution:** $T(m^2) = \sum_{j=1}^{m^2}\lfloor\sqrt j\rfloor$. Values $j \in [i^2,\,(i+1)^2 - 1]$ give $\lfloor\sqrt j\rfloor = i$ and there are $2i+1$ of them, so

$$T(m^2) = \sum_{i=1}^{m-1} i(2i+1) + m = \frac{(m-1)m(2m-1)}{3} + \frac{(m-1)m}{2} + m = \frac m6\left(4m^2 - 3m + 5\right)$$

**Quick check:** $T(1)=1,\ T(4)=5,\ T(9)=16,\ T(16)=38$; B gives $1, 5, 16, 38$ ✔.

- **Why not A:** only degree 2 in $m$ (grows too slowly); fails at $m=4$ (gives 34, not 38).
- **Why not C:** contains $m\cdot m^{2.5} = m^{3.5}$ — non-integer power, impossible for a sum of integer polynomial terms.
- **Why not D:** degree 4 ($\approx \frac56 m^4$) — too fast; true growth is $\approx \frac23 m^3$.

---

### GATE 2020 (1 Mark)

**Q.** For parameters $a$ and $b$, both of which are $\omega(1)$: $T(n) = T(n^{1/a}) + 1$, and $T(b) = 1$. Then $T(n)$ is:

| Option | Answer |
|---|---|
| A | $\Theta(\log_2\log_2 n)$ |
| **B** | $\Theta(\log_a\log_b n)$ ✅ |
| C | $\Theta(\log_b\log_a n)$ |
| D | $\Theta(\log_{ab} n)$ |

**Working (from lecture):**

- $T(n) = T(n^{1/a}) + 1 = T(n^{1/a^2}) + 2 = T(n^{1/a^3}) + 3 = \dots = T(n^{1/a^k}) + k$
- Stop when $n^{1/a^k} = b$:
  $$\frac{1}{a^k}\log_2 n = \log_2 b \;\Rightarrow\; a^k = \log_b n \;\Rightarrow\; k = \log_a\log_b n$$
- $T(n) = T(b) + \log_a\log_b n = 1 + \log_a\log_b n = \Theta(\log_a\log_b n)$

**Why the others are wrong:**
- **A:** hard-codes base 2 and ignores the parameters $a, b$ (the base of the log changes the order when $a$, $b$ are $\omega(1)$).
- **C:** swaps $a$ and $b$ — the *inner* log must use $b$ (stopping value), the *outer* uses $a$ (the root index).
- **D:** a single log; the recursion shrinks $n$ by roots, so it gives a $\log\log$, not a $\log$.

---

### MSQ — Recurrence with $n = 2^{2k}$

**Q (MSQ).** When $n = 2^{2k}$ for some $k \ge 0$, the recurrence
$$T(n) = \sqrt2\,T(n/2) + \sqrt n,\quad T(1) = 1$$
evaluates to:

| Option | Expression | Lecture's mark |
|---|---|---|
| **A** | $\sqrt n\,(\log n + 1)$ | ✅ |
| **B** | $\sqrt n\,\log n$ | ✅ |
| **C** | $\sqrt n\,\log\sqrt n$ | ✅ |
| D | $n\log\sqrt n$ | ❌ |

**Working:**

- ① $T(n) = 2^{1/2}\,T(n/2) + n^{1/2}$
- ② $T(n/2) = 2^{1/2}\,T(n/4) + (n/2)^{1/2}$
- ③ $T(n) = 2\,T(n/2^2) + 2\sqrt n$
- Next: $2\sqrt2\,T(n/8) + 3\sqrt n = 2^{3/2}\,T(n/2^3) + 3\sqrt n$
- general: $T(n) = 2^{k/2}\,T(n/2^k) + k\sqrt n$, with $n = 2^k$ (so $2^{k/2} = \sqrt n$, $k = \log n$)

$$T(n) = \sqrt n\cdot T(1) + \sqrt n\log n = \sqrt n\,(1 + \log n)$$

- **Exact value** = **A**. 🔹 *(Strictly, only A is the exact closed form.)*
- The lecture ticks **B and C as well** because all three are the **same order**: $\log\sqrt n = \tfrac12\log n$, so $\sqrt n\log\sqrt n = \tfrac{\sqrt n}{2}\log n$, and all of A, B, C are $O(\sqrt n\log n)$.
- **Why D is wrong:** $n\log\sqrt n$ has a factor $n$, not $\sqrt n$ — it grows much faster than the true $\sqrt n\log n$.

---

## 11. Memory tricks

*The lecture has no explicit mnemonics; these 🔹 are my own aids built from its patterns.*

- 🔹 **"One call = add, two calls = double."** `T(n−1)+…` → polynomial; `2T(n−1)+…` → $2^n$.
- 🔹 **Halving depth is $\log n$; square-rooting depth is $\log\log n$.** (Halving the *number*, vs halving the *number of bits*.)
- 🔹 **$2^k$ trick for $\sqrt{}$ recursions:** $2^k = \log n$ → so $2^{\text{depth}} = \log n$, hence two-call version gives $O(\log n)$.
- 🔹 **Halving + $B(n)=O(1)$:** one call → $\log n$; two calls → $n$ (leaf count). **$B(n)=O(n)$:** one call → $n$; two calls → $n\log n$ (merge-sort shape).
- 🔹 **Space = recursion depth** (for plain recursion without extra arrays): $n$-depth → $O(n)$, halving → $O(\log n)$, sqrt → $O(\log\log n)$.
- 🔹 **Always stop at the base case, not at 0:** set the argument equal to the base value (1 or 2) to find $k$.
- 🔹 **$O(1/n)$ is still $\Omega$(constant) overall:** the harmonic sum adds only $\log n$; the $+a$ per level gives the linear $an$.

---

## 12. Quick Revision Checklist

- [ ] Steps of **back-substitution** (expand → general $k$ → solve for $k$ → substitute → dominant term)
- [ ] Merging constants ($a + b = d$) and keeping the dominant term ($a+n \to n$, $n^2+n+a \to n^2$)
- [ ] `What(n)`: $T(n-1) + B(n)$ for $B = O(1),\ O(n),\ O(1/n)$ → $O(n),\ O(n^2),\ O(n)$
- [ ] Harmonic sum $\sum 1/x \approx \log n$ (Case III)
- [ ] `Do_It(n)` = $2T(n-1)+d$ → $O(2^n)$; general form $2^kT(n-k)+(2^k-1)d$
- [ ] `abc(s)` string recursion → $2T(n-1) + a$ → $O(2^n)$
- [ ] $T(n) = T(\sqrt n) + d$ → $O(\log\log n)$; derive $k = \log\log n$
- [ ] $T(n) = 2T(\sqrt n) + d$ → $O(\log n)$; identity $2^{\log\log n} = \log n$
- [ ] `Recur(n/2)` once with $B = O(1)$ → $O(\log n)$; with $B = O(n)$ → $O(n)$
- [ ] `Recur(n/2)` twice with $B = O(1)$ → $O(n)$; with $B = O(n)$ → $O(n\log n)$
- [ ] Geometric series formula $\frac{a(1-r^n)}{1-r}$ and $\sum \frac{1}{2^i} = 1 - \frac{1}{2^n}$
- [ ] $T(n) = \sqrt n\,T(\sqrt n) + n$ → $O(n\log\log n)$
- [ ] Flowchart: count recursive calls on best/worst path → $2T(n/2)$ → $O(n)$; $8T(n/2)$ → $O(n^3)$
- [ ] Space complexity = recursion depth: $O(n)$, $O(\log n)$, $O(\log\log n)$ cases
- [ ] GATE 2014: $2T(n/2)+\log n$ → $\Theta(n)$ (H/W 1)
- [ ] GATE (H/W 2): $T(m^2) = \frac m6(4m^2-3m+5)$
- [ ] GATE 2020: $T(n)=T(n^{1/a})+1,\ T(b)=1$ → $\Theta(\log_a\log_b n)$
- [ ] MSQ: $\sqrt2\,T(n/2)+\sqrt n$ with $n=2^{2k}$ → $\sqrt n(\log n+1)$ ≈ $O(\sqrt n\log n)$; options A, B, C (D wrong)
