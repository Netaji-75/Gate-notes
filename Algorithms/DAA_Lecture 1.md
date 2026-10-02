# Design & Analysis of Algorithms — Lecture 01: Analysis of Algorithms

> **Source:** GeeksforGeeks GATE – DAA, Lecture 01 (Analysis of Algorithms)
> **Exam relevance:** GATE CS/IT · GATE DA · TIFR · ISRO/BARC/NIC · semester exams · placements

---

## Table of Contents

1. [Course Roadmap & Logistics](#1-course-roadmap--logistics)
2. [Algorithm Concept](#2-algorithm-concept)
3. [Algorithm Lifecycle](#3-algorithm-lifecycle)
4. [Need for Analysis](#4-need-for-analysis)
5. [Methodology: Aposteriori vs Apriori](#5-methodology-aposteriori-vs-apriori)
6. [Apriori Framework & RAM Model](#6-apriori-framework--ram-model)
7. [Worked Example: Step-Count Method](#7-worked-example-step-count-method)
8. [Rate / Order of Growth](#8-rate--order-of-growth)
9. [Comparing Functions (Examples)](#9-comparing-functions-examples)
10. [Input Size](#10-input-size)
11. [GATE PYQs](#11-gate-pyqs)
12. [Memory Tricks](#12-memory-tricks--mnemonics)
13. [Quick Revision Checklist](#13-quick-revision-checklist)

---

## 1. Course Roadmap & Logistics

### Weightage & usefulness
| Exam | Marks from DAA |
|---|---|
| GATE CS/IT | **~8–10 marks** (≈ 55–60 hrs of study suggested) |
| GATE DA | **~4–6 marks** |

Also useful for: **TIFR, ISRO/BARC/NIC, semester exams, placements.**

### Prerequisites
- **Programming language** (C / C++ / Java): reading `for i ← 1 to n`, `if (a < b) c = c + 1;`
- **Basics of Data Structures** (college level)
- **Maths background:** logarithms, functions, summation series, exponents, differentiation & integration rules
  - Be comfortable comparing: **n vs log n**, **√n vs log n**

### Text books
1. *Introduction to Algorithms* — Cormen et al. (**CLRS**)
2. *Fundamentals of Algorithms* — **Horowitz & Sahni**

### Full course schedule

| # | Unit | Topics |
|---|---|---|
| 1 | **Analysis of Algorithms** | Concept & lifecycle · Need · Methodology & types · Asymptotic Notations (ASN) · Framework for recursive algorithms · Apriori analysis of non-recursive algorithms · Loop complexities · Space complexity · Mathematical background |
| 2 | **Divide & Conquer** | General method · Max-Min · Merge Sort · Binary Search · Quick Sort · Matrix Multiplication · Long Integer Multiplication (LIM) · Master Method · Recursion Tree |
| 3 | **Greedy Method** | General method · Knapsack · Job Sequencing with Deadlines · Optimal Merge Patterns (Huffman Coding) · MST (Prim's, Kruskal's) · Dijkstra's shortest paths |
| 4 | **Dynamic Programming** | The method · DP vs Greedy vs D&C · Multistage graphs · TSP · Binary (0/1) Knapsack · All-Pairs Shortest Paths · Bellman-Ford · LCS · Matrix Chain Multiplication · Sum of Subsets · Reliable System Design · Optimal Cost BST |
| 5 | **Graph Algorithms** | Representation · Traversals · **DFS** (undirected connected, disjoint graphs/DFT, directed graphs & edge types, DAG) · **BFS** (FIFO, LIFO, LC) · Parenthesization Theorem |
| 6 | **Heap Algorithms** | Create, Insert, Delete, Modify · Heapsort |
| 7 | **Sets** | Representations · Operations |
| 8 | **Sorting** | Terminology · Bubble, Selection, Insertion, Radix |
| 9 | **Backtracking & Branch and Bound** | Overview |

---

## 2. Algorithm Concept

> *"Algorithms are the threads that tie together most of the subfields of Computer Science."* — **Donald Knuth**

### 2.1 Origin of the word
- Named after **Muhammad ibn Musa al-Khwarizmi** — Persian mathematician, "father of algebra", born ~**AD 780**, director of the **House of Wisdom** (Baghdad).
- Latinised name **"Algoritmi"** → the term **algorithm**.
- **Algebra** ← *al-jabr* in the title of his book (~**AD 820**) *al-Kitab al-Mukhtasar fi Hisab al-Jabr wal-Muqabalah*.

### 2.2 Definitions

| Term | Definition |
|---|---|
| **Informal** | A well-defined computational procedure that takes input(s), produces output(s), runs in finite time, and transforms input → output through clear steps |
| **Effective method (procedure)** | A procedure reducing the solution of a class of problems to a series of **rote steps** which, if followed to the letter, is bound to: (a) always give *some* answer rather than none, (b) always give the **right** answer, never a wrong one, (c) finish in a **finite** number of steps, (d) work for **all instances** of the problem class |
| **Algorithm (formal)** | An **effective method** expressed as a **finite list of well-defined instructions** for calculating a **function** |

Lecture's board definition: *a finite set of steps/statements to solve a given problem (function)*; each step consists of one or more **basic/fundamental operations** (e.g., `a + b *`), which must satisfy **definiteness** and **effectiveness**.

- May accept **0 or more inputs**
- Must produce **at least one output**
- Model: `Inputs → [ Abstract machine | Processing logic ] → Outputs`

### 2.3 Correct vs Incorrect algorithm
- **Correct:** for **every problem instance** (= *test case input*), it **halts** (finishes in finite time) **and** outputs the correct solution. A correct algorithm *solves* the problem.
- **Incorrect:** may **never halt** on some inputs, **or** halts with a **wrong answer**.

### 2.4 Properties of an algorithm — **FDIOE**

| Letter | Property | Meaning |
|---|---|---|
| **F** | **Finiteness** | Must terminate after a finite number of steps (no infinite loops) |
| **D** | **Definiteness** | Each step precisely and unambiguously defined for every case |
| **I** | **Input** | **Zero or more** inputs from a specified set of objects |
| **O** | **Output** | **One or more** outputs with a specified relation to inputs |
| **E** | **Effectiveness** | Every operation is basic enough to be done exactly and in finite length |

### 2.5 Problem vs Algorithm vs Program
- One **problem** → **many algorithms** (algorithm 1, 2, … k)
- One **algorithm** → **many programs** (implementations in different languages)

```
Problem ──► Algorithm 1
        ──► Algorithm 2
        ──► Algorithm k ──► Program 1, Program 2, … Program n
```

### 2.6 Ways to express an algorithm

| Method | ✅ Pros | ❌ Cons |
|---|---|---|
| **Natural language** | Easy to write | Verbose, **ambiguous** |
| **Flowchart** | Visual; avoids most ambiguity; largely standardized | Hard to modify without specialized tools |
| **Pseudo-code** *(preferred; the lecture uses SPARK-style pseudo-code)* | Avoids most ambiguity; language-agnostic, precise | **No standard syntax** (resembles programming languages vaguely) |
| **Programming language** | Directly executable | Too many **low-level details** unnecessary for high-level understanding |

### 2.7 Common elements of algorithms
1. **Acquire data (input)** — read values from an external source (e.g., coefficients of a polynomial)
2. **Computation** — arithmetic, comparisons, logical tests (`+ − * && <`)
3. **Selection** — choose among 2+ courses of action (`if`)
4. **Iteration** — repeat instructions a fixed number of times or until a condition holds (loops)
5. **Report results (output)** — show results / request more data from the user

### 2.8 Real-world problems solved by algorithms

| Domain | Problem | Technique hinted |
|---|---|---|
| Human Genome Project | ~30,000 genes, ~3 billion base pairs; storing & analysing data | **Dynamic Programming** |
| Internet | Finding good **routes** for data; **search engine** for pages | Shortest paths / graph algorithms |
| E-commerce | Privacy of credit cards, passwords; **public-key cryptography, digital signatures** | **Numerical algorithms & number theory** |
| File compression | LZW compression (repeating character sequences) | **Huffman Coding** |
| Resource allocation | Oil-well placement for profit; campaign ad spending; airline crew assignment; ISP resource placement | **Linear programming** |
| Road map | Shortest route between two intersections (too many routes to brute-force) | Shortest-path algorithms |
| Medicine | Classify a tumor as cancerous vs benign by similarity to known images | Similarity / classification |

---

## 3. Algorithm Lifecycle

| Step | Phase | Note |
|---|---|---|
| 1 | **Problem definition** | |
| 2 | **Requirements (SRS)** | + **constraints** |
| 3 | **Design / Logic** | ⬅ **covered in DAA** |
| 4 | Algorithm development | |
| 5 | **Validation** | |
| 6 | **Analysis** | ⬅ **covered in DAA** |
| 7 | **Implementation** | |
| 8 | **Testing & Debugging** | |

> **DAA = Design (step 3) + Analysis (step 6).**

---

## 4. Need for Analysis

**What & Why:**
1. **Resource consumption** — how much resource an algorithm uses
2. **Performance comparison** — for a problem **P** with candidate algorithms **A₁, A₂, A₃**, pick the **efficient** (effective) solution

**Metrics of performance (resources):** **Time** and **Space**.

Analysis = how resource needs (time & space) **scale with increasing input size**.

### Proposed definitions of "efficiency"

| # | Definition | Verdict |
|---|---|---|
| 1 | Efficient if, when implemented, it runs **quickly** on real input instances | Vague ("quickly"), environment-dependent |
| 2 | Efficient if it achieves **qualitatively better worst-case** performance (analytically) than **brute-force** search | Better |
| 3 | Efficient if it has a **polynomial running time** | ✅ **Adopted** (lecture boxes this; relates to *Time*) |

---

## 5. Methodology: Aposteriori vs Apriori

**How do we measure time? → Time complexity.**

### 5.1 Why raw timing fails
A statement like `x ← y + z;` takes different time depending on the **environment/platform**:
- **H/W:** CPU, memory, I/O
- **S/W:** OS, programming language (C / C++ / Java)

**Analogy (travel X → Y):** time depends on the mode — Walk ≈ 1 hr, Cycle ≈ 40, Bike ≈ 15, Car ≈ 10 (min), Bus/Metro, Plane… The *distance* is the same; the *platform* changes the time. So raw time ≠ a fair comparison.

### 5.2 Comparison table

| Feature | **Aposteriori analysis** (Experimental) | **Apriori analysis** (Analytical) |
|---|---|---|
| Approach | Implement, run, measure | Mathematical analysis of high-level description |
| Platform dependence | **Dependent** on H/W & S/W | **Independent** of H/W & S/W |
| Inputs considered | Only a **limited set** of test inputs (must be representative) | **All possible inputs** of any size |
| Needs implementation? | **Yes** — must implement & execute | **No** — works on pseudo-code/high-level description |
| Comparing 2 algorithms | Only fair if run on the **same** H/W & S/W | Fair, environment-independent |
| Output | Actual running time (secs) | Function **f(n)** of input size |
| Scaling behaviour | Not always clear | Clear (order of growth) |

**Conclusion:** experimentation is useful but **not sufficient** → we need an **analytic framework** that
- takes **all possible inputs** into account,
- evaluates **relative efficiency** of any two algorithms independent of H/W & S/W,
- works on a **high-level description** without implementing/running.

---

## 6. Apriori Framework & RAM Model

### 6.1 Components of the Apriori analysis framework
1. A **language** for describing algorithms (pseudo-code)
2. A **computational model** that algorithms execute within → **RAM Model** ⭐
3. A **metric** for measuring running time
4. An **approach** for characterizing running times (incl. **recursive** algorithms)

### 6.2 RAM (Random Access Machine) model
- **Memory:** sequence of instructions/cells (I₁, I₂, I₃ … Iₙ) + a **CPU**
- **Each fundamental/basic operation takes constant time = 1 unit**

### 6.3 Goal of the methodology
Associate with each algorithm a function **f(n)** that characterizes its running time in terms of **input size n**.
- Typical functions: **n**, **n²**
- "Algorithm A runs in time proportional to n" ⇒ actual time never exceeds **c·n** (c depends on H/W & S/W)
- If A is ∝ n and B is ∝ n² → **prefer A** (n grows at a smaller rate than n²)

---

## 7. Worked Example: Step-Count Method

**Setup:** time under the RAM model; input size = `n`; every basic operation = 1 unit.

```text
Algorithm Test(int n)
{
1.  x ← y + z;                       // 2 units  (add + assign)
2.  for i ← 1 to n                   // loop
        c ← c + 1;
3.  for i ← 1 to n                   // outer loop
        for j ← 1 to n               // inner loop
            k ← k * 5;
}
```

### Statement-wise count

| Stmt | Frequency (order of magnitude) | Units (step count) | Breakdown |
|---|---|---|---|
| 1 | 1 | **2** | add + assign |
| 2 | n | **4n + 2** | `i←1` (1) + test `(n+1)` + increment `n` + body `c←c+1` = 2n |
| 3 | n² | **4n² + 4n + 2** | outer: `1 + (n+1) + n`; inner init: `n`; inner test: `n(n+1)`; inner increment: `n·n`; body `k←k*5`: `2·n·n` |

### Total time

$$T(n) = 2 + (4n+2) + (4n^2+4n+2) = 4n^2 + 8n + 6 \text{ units}$$

**Order of magnitude** = *frequency / number of times the fundamental operation in a statement executes.* Here frequencies: 1, n, n² → **n² + n + 1**.

### Asymptotic simplification (ASN)
Drop lower-order terms (`8n`, `6`) and constant factor (`4`):

$$T(n) = 4n^2 + 8n + 6 \;\Rightarrow\; \boxed{O(n^2)}$$

---

## 8. Rate / Order of Growth

> It is the **rate of growth (order of growth)** of running time that really interests us.

- **Order of growth** = approximation of how long a program takes as **input size increases**.
- Focus on operations that grow **proportionally with input size**; **ignore lower-order terms and constant factors** (fixed operations).
- Described using **asymptotic notations (ASN)** — e.g., **Big-Θ, Big-O**.

### 8.1 Two families of time functions

| | **Polynomial** | **Exponential** |
|---|---|---|
| Form | $n^x,\; x \ge 0$ → $n^0, n^1, n^2, n^3 \dots$ | $a^n,\; a > 1$ → $2^n, 3^n, 4^n \dots n^n$ |
| Growth | **Grows slow**, lower rate of growth | **Grows fast**, higher rate of growth |
| Time | Less time | More time |
| Algorithm behaviour | Algorithm runs **fast** | Algorithm runs **slow** |

### 8.2 Categories of functions (increasing rate of growth →)

| # | Category | T(n) | Group |
|---|---|---|---|
| 1 | **Constant** | $T(n) = c$ (e.g., f(0)=f(1)=f(2)=…=10) | Polynomial group (as drawn in lecture) |
| 2 | **Logarithmic** | $T(n) = \log_2 n$ | ↑ |
| 3 | **Linear** | $T(n) = n$ | ↑ |
| 4 | **Quadratic** | $T(n) = n^2$ | ↑ |
| 5 | **Cubic** | $T(n) = n^3$ | ↑ |
| 6 | **Exponential** | $T(n) = 2^n$ | Exponential |

**Order:** $c < \log n < n < n^2 < n^3 < 2^n$

### 8.3 Formula & example: Sum of Squares

$$1^2 + 2^2 + \dots + n^2 = \sum_{i=1}^{n} i^2 = \frac{n(n+1)(2n+1)}{6}$$

- `n` — number of terms; `i` — loop index/term.

```text
Algorithm SSQ(int n)                  // Version 1: loop  →  O(n)
{
    int i, sum;
    sum = 0;
    for i ← 1 to n
        sum = sum + i * i;
    return (sum);
}

Algorithm SSQ_V2(int n)               // Version 2: formula  →  O(1)
{
    return ((n * (n + 1) * (2 * n + 1)) / 6);
}
```

| Version | Time | Complexity |
|---|---|---|
| SSQ (loop) | grows with n | **O(n)** |
| SSQ_V2 (formula) | constant (**7** operations) | **O(1)** ✅ better |

> **Takeaway:** the *same problem* can have algorithms with very different complexities → analysis picks the best.

---

## 9. Comparing Functions (Examples)

### Example 1: f(n) = 3n, g(n) = 2n + 10

| n | f(n) | g(n) |
|---|---|---|
| 2 | 6 | 14 |
| 4 | 12 | 18 |
| 6 | 18 | 22 |
| 8 | 24 | 26 |
| *Increase per step (Δn = 2)* | **+6** | **+4** |

- g starts higher (constant 10), but **f grows faster per step**. Both are **linear → same order O(n)**. *(Extra: they are equal at n = 10 and f > g for n > 10.)*

### Example 2: f(n) = 2n, g(n) = n² + 1

| n | f(n) | g(n) |
|---|---|---|
| 1 | 2 | 2 |
| 2 | 4 | 5 |
| 3 | 6 | 10 |
| 4 | 8 | 17 |
| 5 | 10 | 26 |
| *Increase per step* | **+2, +2, +2, +2** (constant) | **+3, +5, +7, +9** (keeps increasing) |

- f increases by a **constant** → linear. g's increments themselves grow → **quadratic**, so **g grows faster** than f.

---

## 10. Input Size

**Input size (n)** = measure of how large the input is = amount of data processed; the variable that decides how running time/space grows. *How it's measured depends on the problem:*

| Input type | Input size n | Example |
|---|---|---|
| **Arrays / lists** | Number of elements | Search in array of 100 elements → n = 100 |
| **Strings** | Length of the string | 50-character string → n = 50 |
| **Files** | Number of bytes or records | — |
| **Graphs** | Number of **vertices V** and/or **edges E** | Complexity often written **O(V + E)** |
| **Numbers** | **Number of digits** (**not** the numeric value) | Number 1000 → input size = **4 digits**, not 1000 |

> ⚠️ **GATE trap:** for numeric inputs (e.g., primality test, factorial of a big number), input size is the **number of digits/bits**, not the value.

---

## 11. GATE PYQs

**No GATE PYQs or knowledge-check questions appear in this lecture.**

Self-check questions (original, *not* PYQs) based on the lecture:

**Q1.** Which of the following is **NOT** a required property of an algorithm?
(A) Finiteness (B) Definiteness (C) At least one input (D) At least one output
**Answer: (C)** — an algorithm may have **zero or more** inputs, but **must** have ≥ 1 output. (A, B, D are required properties.)

**Q2.** Aposteriori analysis is *not* preferred for comparing algorithms because:
(A) It considers all inputs (B) It is hardware/software independent (C) It depends on the platform and tested inputs (D) It needs no implementation
**Answer: (C)** — (A), (B), (D) are properties of *apriori* analysis.

**Q3.** Simplify T(n) = 4n² + 8n + 6 in asymptotic terms.
(A) O(n) (B) O(n²) (C) O(n³) (D) O(2ⁿ)
**Answer: (B)** — drop lower-order terms & constants. (A) underestimates; (C), (D) are loose/wrong tight bounds for the dominant n² term.

**Q4.** Input size for testing whether the number 1000 is prime is:
(A) 1000 (B) 4 (C) 10 (D) 1
**Answer: (B)** — number of digits (4), not the numeric value.

---

## 12. Memory Tricks & Mnemonics

- 🔑 **FDIOE** → **F**inite, **D**efinite, **I**nput, **O**utput, **E**ffective (the 5 properties).
- **Travel analogy:** going X → Y by walk/cycle/bike/car/plane — same distance, different platform ⇒ different time. This is why **raw running time (aposteriori) is unreliable**.
- **Problem → many algorithms → many programs** (one-to-many at each level).
- **"Dominant term wins":** for large n, drop constants and lower-order terms (4n² + 8n + 6 → n²).
- **Lifecycle:** DAA covers only the **boxed** phases — **Design (3)** and **Analysis (6)**.
- *(Extra aid)* **Apriori = "before" running** (analysis from the description); **Aposteriori = "after" running** (measure from experiments).

---

## 13. Quick Revision Checklist

**Course & logistics**
- [ ] GATE weightage (CS/IT ≈ 8–10, DA ≈ 4–6) and other exams that use DAA
- [ ] Prerequisites: programming, basic DS, maths (logs, series, exponents, calculus rules)
- [ ] Text books: CLRS, Horowitz & Sahni
- [ ] The 9 units of the course

**Algorithm concept**
- [ ] Origin of the word (al-Khwarizmi → "Algoritmi"; *al-jabr* → algebra)
- [ ] Definition of effective method vs algorithm
- [ ] Algorithm = finite steps, basic operations, ≥0 inputs, ≥1 output
- [ ] Correct vs incorrect algorithm (halts + correct for every instance)
- [ ] FDIOE properties
- [ ] Problem → algorithms → programs
- [ ] Ways to express: natural language, flowchart, pseudo-code, programming language (pros/cons)
- [ ] Common elements: input, computation, selection, iteration, output
- [ ] Real-world applications (Genome–DP, Huffman, crypto–number theory, LP, shortest route)

**Lifecycle & need**
- [ ] 8 lifecycle steps; DAA = Design + Analysis
- [ ] Need: resource consumption + performance comparison; metrics = time & space
- [ ] Three efficiency definitions; polynomial time is adopted

**Methodology**
- [ ] Why platform (H/W, S/W) affects raw time; travel analogy
- [ ] Aposteriori vs Apriori comparison table
- [ ] Limitations of experimentation (3 points)
- [ ] 4 components of apriori framework; RAM model (each basic op = 1 unit)
- [ ] Goal: f(n) in terms of input size n

**Step-count & growth**
- [ ] Step-count of the Test algorithm → T(n) = 4n² + 8n + 6 → O(n²)
- [ ] Order of magnitude = frequency of execution of the fundamental operation
- [ ] Order/rate of growth; drop lower-order terms & constants
- [ ] Polynomial vs exponential functions
- [ ] Ordering: c < log n < n < n² < n³ < 2ⁿ
- [ ] Sum of squares formula; SSQ O(n) vs SSQ_V2 O(1)
- [ ] Function comparison examples (3n vs 2n+10; 2n vs n²+1)

**Input size**
- [ ] Array / string / file / graph (V, E, O(V+E)) / number (digits, not value)

**Coming next:** Asymptotic notations (Big-O, Big-Θ, …), recursive-algorithm framework, loop complexities, space complexity.
