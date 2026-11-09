# Divide and Conquer (Part 2): Merge Sort, Quick Sort & Matrix Multiplication — GATE DAA Lecture 07 Notes

> Source: GeeksforGeeks GATE — *Design & Analysis of Algorithms*, Lecture 07 (57 slides, typed + handwritten).
> Continues Lecture 06 (control abstraction, Max-Min, Binary Search). Note: this is **DAA**, not OS.
> Marks used below: ✅ = answer marked/derived on the slide, ⚠️ = discrepancy or addition by me (not on the slides).

## Topics Covered

1. [Merge Sort](#1-merge-sort) — merging, algorithms, time/space, call counts, top-down vs bottom-up
2. [Merge Sort PYQs](#2-gate-pyqs--merge-sort)
3. [Quick Sort](#3-quick-sort) — Hoare partition, Lomuto/CLRS partition, performance, pivot choice
4. [Quick Sort PYQs](#4-gate-pyqs--quick-sort)
5. [Matrix Multiplication (D&C)](#5-matrix-multiplication-divide--conquer)
6. [Memory Tricks](#memory-tricks)
7. [Quick Revision Checklist](#7-quick-revision-checklist)

---

## 1. Merge Sort

- **Divide-and-Conquer** sort, invented by **John von Neumann (1945)**.
- **Divide:** split at `mid` → **Conquer:** sort both halves recursively → **Combine:** **merge** two sorted halves.
- Base case: **one element** (already sorted). All real work happens in the **merge** (the combine step).

### 1.1 Worked example (top-down)

Input `7 3 2 16 24 4 11 9`

```text
Divide:
[7 3 2 16 | 24 4 11 9]
[7 3 | 2 16]   [24 4 | 11 9]
[7][3] [2][16] [24][4] [11][9]

Merge:
[3 7] [2 16]   [4 24] [9 11]
[2 3 7 16]     [4 9 11 24]
[2 3 4 7 9 11 16 24]
```

Second example (top-down): `18 12 16 22 5 24 9` → `5 9 12 16 18 22 24`
(⚠️ the slide's last row prints `28` instead of `24`, a typo.)

### 1.2 Merging process (conquer / combine) — 2-pointer method

Merge two **sorted** lists `L1` (size $n$) and `L2` (size $m$), with $n \le m$. Keep pointer `i` on `L1`, `j` on `L2`; always copy the smaller of `L1[i]`, `L2[j]` to the output, advance that pointer.

Example: `L1 = ⟨4, 8, 12, 16, 20⟩`, `L2 = ⟨3, 7, 15, 25, 40, 50⟩` → `L = ⟨3, 4, 7, 8, 12, 15, 16, 20, 25, 40, 50⟩`

| Case | Comparisons | When |
|---|---|---|
| **Minimum** | $n$ (the smaller list's size) | every element of `L1` < first element of `L2` (e.g., `⟨4,5,8,9⟩` vs `⟨10,15,20,25,40,50⟩`) |
| **Maximum** | $n+m-1$ | elements interleave, so both lists run out almost together |

$$
n \;\le\; \text{comparisons} \;\le\; n+m-1 \;\Rightarrow\; O(n+m)
$$

### 1.3 Algorithms (code)

```text
Algorithm MergeSort(low, high)
// a[low:high] is a global array to be sorted.
// Small(P) is true if there is only one element to sort.
{
    if (low < high) then                 // more than one element
    {
        // Divide P into subproblems: find where to split the set
        mid := floor((low + high) / 2);
        // Solve the subproblems
        MergeSort(low, mid);
        MergeSort(mid + 1, high);
        // Combine the solutions
        Merge(low, mid, high);
    }
}
```

```text
Algorithm Merge(low, mid, high)
// a[low:high] is a global array containing two sorted subsets,
// in a[low:mid] and in a[mid+1:high]. Merge them into one sorted
// set in a[low:high]. b[] is an auxiliary global array.
{
    h := low; i := low; j := mid + 1;
    while ((h <= mid) and (j <= high)) do
    {
        if (a[h] <= a[j]) then
        {
            b[i] := a[h]; h := h + 1;
        }
        else
        {
            b[i] := a[j]; j := j + 1;
        }
        i := i + 1;
    }
    if (h > mid) then                    // left half exhausted
        for k := j to high do
        {
            b[i] := a[k]; i := i + 1;
        }
    else                                 // right half exhausted
        for k := h to mid do
        {
            b[i] := a[k]; i := i + 1;
        }
    for k := low to high do a[k] := b[k];   // copy back
}
```

> OCR fixes: `allow : mid]` → `a[low : mid]`; `b[i] = a[h]` → `b[i] := a[h]`; stray `i: = i+1` → `i := i + 1`.

**Divide / merge order on a 10-element example** `A = 179 285 310 351 652 | 254 423 450 520 861` (indices 1–10):

```text
Divide (mid shown):  (1,10)→5   (1,5)→3   (1,3)→2   (1,2)→1   (4,5)→4
                     (6,10)→8   (6,8)→7   (6,7)→6   (9,10)→9
Merge calls (low, mid, high), in order:
  (1,1,2) (1,2,3) (4,4,5) (1,3,5)  (6,6,7) (6,7,8) (9,9,10) (6,8,10)  (1,5,10)
Result B: 179 254 285 310 351 423 450 520 652 861
```

### 1.4 Performance of Merge Sort

**Time complexity** — $T(n)$ = time of `MergeSort(n)`:

$$
T(n)=\begin{cases} c, & n=1\\ 2T\!\left(\frac{n}{2}\right)+bn, & n>1\end{cases}
$$

($c$ = constant for 1 element; $bn$ = merge cost, linear in $n$.)

Expanding: $T(n)=2^kT(n/2^k)+k\,bn$. Stop at $n/2^k=1 \Rightarrow k=\log_2 n$:

$$
T(n)=nc+bn\log_2 n=\boxed{\Theta(n\log n)}
$$

**Space:** auxiliary array $b[\,]$ ($n$) + recursion stack ($\log n$) → $n+\log n=\Theta(n)$.

**Number of calls** (handwritten derivation):

| Quantity | Recurrence | Result |
|---|---|---|
| Calls to `MergeSort(n)` | $T(n)=2T(n/2)+1,\ T(1)=1$ → $2^kT(n/2^k)+(2^k-1)$ with $2^k=n$ | $n+n-1=\mathbf{2n-1}$ |
| Calls to `Merge(n)` | (total `MergeSort` calls) − (calls to `MergeSort(1)`) = $(2n-1)-n$ | $\mathbf{n-1}$ |

### 1.5 Top-Down vs Bottom-Up (2-way) Merge Sort

| Feature | **Top-Down (Recursive)** | **Bottom-Up (Iterative, 2-way)** |
|---|---|---|
| Technique | Recursion; split until 1 element | Iterative **passes**; no recursion |
| Merge pattern | Recursive halves | Merge pairs of size 1, then 2, then 4… |
| Extra space | Call stack $O(\log n)$ + aux array | $O(1)$ call-stack overhead (+ aux array) |
| Time | $O(n\log n)$ | $O(n\log n)$ (same) |
| Recurrence | $T(n)=2T(n/2)+n$ | — |
| Ease | More intuitive, easier to code | Also called **2-way merge sort** |

> **GATE TIP (slide):** 2-way merge sort does $\lceil\log_2 n\rceil$ passes; each pass costs $O(n)$ comparisons → $O(n\log n)$ total.

**Bottom-up trace** — `A = 310 285 179 652 351 423 861 254` ($n=2^3$):

```text
Pass 1 (size-1 runs → size 2): [285 310] [179 652] [351 423] [254 861]
Pass 2 (size 2 → size 4):      [179 285 310 652] [254 351 423 861]
Pass 3 (size 4 → size 8):      [179 254 285 310 351 423 652 861]  ← sorted
```

For $n=2^k$: **number of passes $=\log_2 n$**, each pass processes all $n$ elements → $\Theta(n\log n)$.

⚠️ *Addition (the slide poses "total comparisons in min & max cases for 2-way merge sort" but shows only the setup):* with the merge bounds of §1.2 applied to equal-sized lists,
- **Min** ≈ $\dfrac{n}{2}\log_2 n$ (each merge of two size-$s$ lists costs $s$),
- **Max** $= n\log_2 n-(n-1)$ (each merge costs $2s-1$).

---

## 2. GATE PYQs — Merge Sort

**Q1.** For merging two sorted lists of sizes $m$ and $n$ into a sorted list of size $m+n$, we require comparisons of:

| Option | | |
|---|---|---|
| A | $O(m)$ | ❌ only accounts for one list; worst case needs $m+n-1$ |
| B | $O(n)$ | ❌ same reason |
| **C** | **$O(m+n)$** | ✅ **Correct** |
| D | $O(\log m+\log n)$ | ❌ far too small; every element must be placed |

**Q2** ⭐ Merge Sort takes **30 s to sort 64 elements** in the worst case. Approximately how many elements can be sorted in **6 minutes**?

**Answer: 512**

| Step | Work |
|---|---|
| Model | time ∝ $n\log n$; for 64: $64\log_2 64 = 64\times6=384$ units ↔ 30 s |
| 1 unit | $30/384$ s |
| 360 s | $360\times384/30=4608$ units $=n\log_2 n$ |
| Solve | $n=2^k$: $2^k\cdot k=4608 \Rightarrow k=9 \Rightarrow n=2^9=\mathbf{512}$ ✅ |

Why not 768? The naive proportion $64\cdot\frac{360}{30}=768$ treats time as **linear** in $n$, but Merge Sort is $n\log n$.

**Q3.** Straight two-way Merge Sort on `20, 47, 15, 8, 9, 4, 40, 30, 12, 17`; order after the **second pass**?

| Option | Sequence | |
|---|---|---|
| A | 8, 9, 15, 20, 47, 4, 12, 17, 30, 40 | ❌ not size-4 runs formed from adjacent pairs |
| **B** | **8, 15, 20, 47, 4, 9, 30, 40, 12, 17** | ✅ **Correct** |
| C | 15, 20, 47, 4, 8, 9, 12, 30, 40, 17 | ❌ runs don't match adjacent-pair merging |
| D | 4, 8, 9, 15, 20, 47, 12, 17, 30, 40 | ❌ merges across the wrong run boundaries |

Working:
- Pass 1 → `(20 47)(8 15)(4 9)(30 40)(12 17)`
- Pass 2 (merge adjacent pairs) → `(8 15 20 47)(4 9 30 40)(12 17)` = option B

---

## 3. Quick Sort

- **Partition-exchange sort** — pick a **pivot** $x$; rearrange so **left ≤ pivot ≤ right**; sort both sides recursively.
- **No combine step** — after partitioning, the array is already in the right arrangement.
- Invented by **Sir C. A. R. (Tony) Hoare**. When implemented well, it can be about **2–3× faster** than Merge Sort and Heapsort (slide).

```text
[  elements < x  ]  x  [  elements > x  ]
                  PIVOT
```

### 3.1 Hoare version (first element as pivot)

```text
Algorithm QuickSort(p, q)
// Sorts a[p..q] of global array a[1:n]. a[n+1] is defined and must be >= all
// elements in a[1:n]  (sentinel, e.g. ∞ / 9999).
{
    if (p < q) then                      // more than one element
    {
        j := Partition(a, p, q + 1);     // j = final position of the pivot
        QuickSort(p, j - 1);
        QuickSort(j + 1, q);
        // no need to combine solutions
    }
}
```

```text
Algorithm Partition(a, m, p)
// Rearranges a[m..p-1] around t = a[m]; a[p] is set to ∞ (sentinel).
// After completion a[q] = t, a[k] <= t for m <= k < q, a[k] >= t for q < k < p.
{
    v := a[m]; i := m; j := p;
    repeat
    {
        repeat i := i + 1; until (a[i] >= v);
        repeat j := j - 1; until (a[j] <= v);
        if (i < j) then Interchange(a, i, j);
    } until (i >= j);
    a[m] := a[j]; a[j] := v;
    return j;
}

Algorithm Interchange(a, i, j)
{
    p := a[i];
    a[i] := a[j]; a[j] := p;
}
```

- **Partition time: $O(n)$** (one pass of `i` from the left and `j` from the right).
- ⚠️ One typed slide shows `until (a[i] > v)`; the standard (and other slides') version is `a[i] >= v`.

**Worked trace** — `A[1..9] = 65 70 75 80 85 60 55 50 45`, `a[10] = ∞`, pivot $v=65$:

| Step | `i` stops at | `j` stops at | Action | Array |
|---|---|---|---|---|
| 1 | 2 (70) | 9 (45) | swap | 65 **45** 75 80 85 60 55 50 **70** |
| 2 | 3 (75) | 8 (50) | swap | 65 45 **50** 80 85 60 55 **75** 70 |
| 3 | 4 (80) | 7 (55) | swap | 65 45 50 **55** 85 60 **80** 75 70 |
| 4 | 5 (85) | 6 (60) | swap | 65 45 50 55 **60 85** 80 75 70 |
| 5 | 6 (85) | 5 (60) | $i \ge j$ → stop | — |
| Final | — | — | `a[m] := a[j]; a[j] := v` | **60 45 50 55 \[65\] 85 80 75 70** |

Pivot 65 lands at **position 5**; left `{60,45,50,55}` all < 65, right `{85,80,75,70}` all > 65. Final `(i, j) = (6, 5)`.

### 3.2 Lomuto partition & CLRS version (last element as pivot)

**Lomuto idea (slide):**
- Pivot = **last element**.
- Scan from the left; whenever an element is **< (≤) pivot**, **increment the current index** and **swap** that element into it.
- Finally **swap pivot with element at (index + 1)**; return that index.
- The pivot is then at its **final sorted position** → **excluded** from the recursive calls.

```text
Algorithm QuickSort(A, p, r)
{
    if (p < r)
    {
        q := Partition(A, p, r);
        QuickSort(A, p, q - 1);
        QuickSort(A, q + 1, r);          // pivot A[q] excluded
    }
}

Algorithm Partition(A, p, r)
{
    x := A[r]; i := p - 1;
    for j = p to (r - 1)
    {
        if (A[j] <= x)
        {
            i := i + 1;
            swap(A[i], A[j]);
        }
    }
    swap(A[i + 1], A[r]);
    return (i + 1);
}
```

**Trace** `A = 2 8 7 1 3 5 6 4` (pivot $x=4$): the `swap`s happen at $j=1$ (self), $j=4$, $j=5$ → after the final swap: **`2 1 3 [4] 7 5 6 8`**, returns pivot index 4.

Second picture on the slide (pivot = last element `3`): `9 -3 5 2 6 8 -6 1 3` → left (≤3) `-3 2 -6 1`, `3`, right (≥3) `8 5 9 6` → recurse → sorted `-6 -3 1 2 3 5 6 8 9`.

### 3.3 Hoare vs Lomuto

| Feature | **Hoare (Sahni version)** | **Lomuto / CLRS** |
|---|---|---|
| Pivot choice | **First** element | **Last** element |
| Scanning | Two pointers `i` (→) and `j` (←) | One scan pointer `j`, one boundary `i` |
| Recursion | `QuickSort(p, j-1)`, `QuickSort(j+1, q)` | `QuickSort(p, q-1)`, `QuickSort(q+1, r)` |
| Best / Avg time | $O(n\log n)$ | $O(n\log n)$ |
| Worst time | $O(n^2)$ | $O(n^2)$ |
| Space (stack) | $O(\log n)$ | $O(\log n)$ |
| Stable? | **No** | **No** |
| In-place? | **Yes** | **Yes** |

> Slide key: *"Hoare's pivot is NOT at its final sorted position after partition (include it in next recursion). Lomuto's IS final (exclude it)."*
> ⚠️ That key describes the original two-pointer Hoare scheme. In *this lecture's* Sahni-style `Partition` the pivot **is** put at `a[j]` and excluded (`j-1`, `j+1`), exactly as shown in §3.1.

### 3.4 Performance of Quick Sort

Partition costs $O(n)$. Time depends on how **balanced** the split is.

| Case | Input / split | Recurrence | Result |
|---|---|---|---|
| **Best** | Pivot splits into **$n/2$ + $n/2$** (e.g., unsorted/random-like) | $T(n)=O(n)+2T(n/2)$ | $O(n\log n)$ |
| **Worst** | Split **$n-1$ and $0$** — already **sorted** (asc/desc) with first/last pivot | $T(n)=O(n)+T(n-1)$ | $O(n^2)$ |
| **Average** | Random splits | — | $O(n\log n)$ (Sahni) |

**Asymmetric recurrence:** any constant-ratio split $T(n)=T(\alpha n)+T((1-\alpha)n)+O(n)$ still gives $O(n\log n)$ (solved by the recursion tree; e.g., $T(n/5)+T(4n/5)+n$).

Illustrative worst-case inputs from the slide: `⟨5,10,12,18,23⟩` (ascending) and `⟨30,20,18,16,12,5⟩` (descending), both with first-element pivot.

### 3.5 Pivot ranking

| Pivot strategy | Outcome |
|---|---|
| **First element** on an already-sorted array | **Worst case $O(n^2)$** — splits into $n-1$ and $0$ every time |
| **Median** as pivot (found in $O(n)$) | **Best case $O(n\log n)$** — perfect 50/50 split |
| **Random** pivot (default in practice) | **Expected $O(n\log n)$** — probabilistically balanced |

---

## 4. GATE PYQs — Quick Sort

**Q1.** Quick Sort where the **central element** of the array is always chosen as pivot. Worst-case complexity?

**Answer: $O(n^2)$** ✅ (handwritten on the slide)
- The array can be arranged so the middle element is always the minimum/maximum → split $(n-1,\,0)$ every time. Choosing by *position* does not guarantee a balanced split.
- ⚠️ The later summary slide ("Quick Sort GATE Practice Questions", Q1) says "$O(n\log n)$ — always balanced split", which contradicts the worked slide. Go with **$O(n^2)$**.

**Q2.** Median of an array can be found in $O(n)$. If the median is the pivot, worst-case complexity of Quick Sort?

**Answer: $\Theta(n\log n)$** ✅

$$T(n)=O(n)_{\text{find median}}+O(n)_{\text{partition}}+2T(n/2)=2T(n/2)+O(n)$$

**Q3.** The **$(n/4)$-th smallest** element is chosen as pivot using an $O(n)$ algorithm. Time complexity?

**Answer: $O(n\log n)$** ✅

$$T(n)=O(n)+O(n)+T(n/4)+T(3n/4)$$

General form $T(n)=O(n)+O(n)+T(\alpha n)+T((1-\alpha)n)=O(n\log n)$ for any constant $\alpha$.

**Q4.** A procedure finds a pivot that splits the list into two sub-lists each with **at least one-fifth** of the elements. $T(n)$ = number of comparisons for $n$ elements. Then:

| Option | | |
|---|---|---|
| A | $T(n)\le 2T(n/5)+n$ | ❌ sub-list sizes are $n/5$ and up to $4n/5$, not both $n/5$ |
| **B** | **$T(n)\le T(n/5)+T(4n/5)+n$** | ✅ **Correct** (worst split = 1:4, plus $n$ for partition) |
| C | $T(n)\le 2T(4n/5)+n$ | ❌ not the exact recurrence for a 1:4 split (assumes both parts are $4n/5$) |
| D | $T(n)\le 2T(n/2)+n$ | ❌ assumes a perfect half split |

**Q5.** $P$ = Quick Sort program (ascending). $t_1$, $t_2$ = times for inputs `[1 2 3 4 5]` and `[5 4 3 2 1]`. Which holds?

| Option | | |
|---|---|---|
| **A** | **$t_1=t_2$** | ✅ **Correct** — both are sorted (asc / desc) → same worst-case $n-1,\,0$ splits |
| B | $t_1>t_2$ | ❌ no input is worse than the other here |
| C | $t_1<t_2$ | ❌ same reason |
| D | $t_1=t_2=5\log 5$ | ❌ $n\log n$ is the *best* case; these are worst-case inputs |

**Q6.** First element as pivot. $t_1$, $t_2$ = comparisons for `{1,2,3,4,5}` and `{4,1,5,3,2}`. Which holds?

| Option | | |
|---|---|---|
| A | $t_1=5$ | ❌ sorted input costs $4+3+2+1=10$ |
| B | $t_1<t_2$ | ❌ $t_1=10$ is larger than $t_2$ |
| **C** | **$t_1>t_2$** | ✅ **Correct** |
| D | $t_1=t_2$ | ❌ $t_2$ is smaller (see below) |

Working (each partition of size $k$ ≈ $k-1$ comparisons): $t_1=4+3+2+1=10$. For `{4,1,5,3,2}`: pivot 4 → `{1,3,2}` and `{5}` (4 comps), then pivot 1 (2 comps), then `{3,2}` (1 comp) → $t_2=7$.

**Q7.** Quick-sort (first element pivot) on (i) `1, 2, 3, …, n` and (ii) `n, n−1, …, 2, 1`. $C_1$, $C_2$ = comparisons. Then:

| Option | | |
|---|---|---|
| A | $C_1<C_2$ | ❌ |
| B | $C_1>C_2$ | ❌ |
| **C** | **$C_1=C_2$** | ✅ **Correct** — both are worst-case ($O(n^2)$) with the same split pattern |
| D | Cannot say for arbitrary $n$ | ❌ it holds for every $n$ |

**Q8.** 25 distinct elements, pivot chosen **uniformly at random**. Probability that the pivot lands in the **worst possible location** in the first partition (2 d.p.)?

**Answer: $2/25=\mathbf{0.08}$** ✅
Worst locations = the **first** or **last** sorted position (pivot is the min or max) → 2 favourable out of 25.

**Q9.** In-place Quick Sort with **last element as pivot** on `60 70 80 90 100`. Minimum number of swaps?

*(No answer is shown on the slide — working is mine.)* ⚠️
- The array is already sorted; the pivot is the maximum, so every element satisfies `A[j] <= x`.
- With the CLRS code (§3.2) every comparison triggers `swap(A[i], A[j])` with $i=j$ (a self-swap), plus the final `swap(A[i+1], A[r])`.
- Counting each `swap(...)` call: sizes 5, 4, 3, 2 → $5+4+3+2=14$. Counting only swaps that actually move data: **0**. State which convention the question intends before answering.

---

## 5. Matrix Multiplication (Divide & Conquer)

*(Slide title: "Strassen's Matrix Multiplication"; the slides actually cover the standard D&C method and its $O(n^3)$ cost. The 7-multiplication Strassen trick is **not** shown.)*

### 5.1 Basic matrix operations ($A_{n\times n},\,B_{n\times n},\,C_{n\times n}$)

| Operation | Code idea | Time |
|---|---|---|
| **Add / subtract** $A\pm B=C$ | `for i←1 to n, for j←1 to n: C[i,j] = A[i,j] ± B[i,j]` | $O(n^2)$ |
| **Multiply** $A\times B=C$ (school method, non-D&C) | `for i, for j: C[i,j]=0; for k: C[i,j] += A[i,k]*B[k,j]` | $O(n^3)$ |

$$
C_{i,j}=\sum_{k=1}^{n}A_{i,k}\,B_{k,j}
$$

### 5.2 D&C approach (`DandC-MATMULT`, $n\times n$)

Split each matrix into four $\frac n2\times\frac n2$ blocks:

$$
\begin{aligned}
C_{11}&=A_{11}B_{11}+A_{12}B_{21} &\text{(1)}\\
C_{12}&=A_{11}B_{12}+A_{12}B_{22} &\text{(2)}\\
C_{21}&=A_{21}B_{11}+A_{22}B_{21} &\text{(3)}\\
C_{22}&=A_{21}B_{12}+A_{22}B_{22} &\text{(4)}
\end{aligned}
$$

- **8** multiplications of size $\frac n2$ and **4** additions of size $\frac n2$ ($\Rightarrow$ $O(n^2)$ combine work).
- Recursion shrinks $n\times n \to \frac n2\times\frac n2 \to \dots \to 2\times2$ / $1\times1$ (**Small**).

$$
T(n)=\begin{cases} c, & n\le 2\\ 8\,T\!\left(\frac{n}{2}\right)+bn^2, & n>2\end{cases}
\;\Rightarrow\; \boxed{O(n^3)}
$$

($8$ = number of subproblems, $n/2$ = size of each, $bn^2$ = cost of the block additions.) **No improvement** over the school method — this is the baseline Strassen improves on.

⚠️ *Addition (standard result, not on the slides):* Strassen reduces 8 multiplications to 7, giving $T(n)=7T(n/2)+O(n^2)=O(n^{\log_2 7})\approx O(n^{2.81})$.

---

## Memory Tricks

- **Merge Sort = hard merge, easy divide; Quick Sort = hard partition, easy combine (none).**
- **"Quick Sort is NOT always quick — it's all about the pivot."** First element on sorted input = disaster; median = GOAT; random = safe default. (Slide: *"L. Ratio. Every. Single. Pass."* for the $n-1$ / $0$ split.)
- **Hoare vs Lomuto:** Hoare = *two pointers from both ends*, Lomuto = *one scanner + boundary*; Lomuto's pivot is final ⇒ **exclude** it from recursion.
- **Calls to MergeSort = $2n-1$**, calls to Merge = $n-1$ (a binary tree with $n$ leaves has $n-1$ internal nodes).
- **2-way merge sort:** $\log_2 n$ passes × $n$ work per pass.
- **Merging bounds:** *min = smaller list's size ($n$), max = $n+m-1$.*
- **Time-scaling questions:** never scale linearly for $n\log n$ — compute total *units* first (the "30 s for 64" question).

---

## 7. Quick Revision Checklist

**Merge Sort**
- [ ] Divide / conquer / combine; invented by von Neumann (1945)
- [ ] Merge of two sorted lists: 2-pointer method
- [ ] Merge comparison bounds: min $n$, max $n+m-1$ ($O(n+m)$)
- [ ] `MergeSort(low, high)` and `Merge(low, mid, high)` code (aux array `b`, copy-back loop)
- [ ] Divide tree + order of merge calls for 10 elements
- [ ] $T(n)=2T(n/2)+bn \Rightarrow \Theta(n\log n)$ (substitution)
- [ ] Space $n+\log n=\Theta(n)$
- [ ] Calls: MergeSort $=2n-1$, Merge $=n-1$
- [ ] Top-down vs bottom-up table; $\lceil\log_2 n\rceil$ passes; bottom-up trace
- [ ] PYQs: merging $O(m+n)$; 30 s/64 elements → 512; 2nd-pass sequence (B)

**Quick Sort**
- [ ] Partition-exchange idea; no combine step
- [ ] Hoare `QuickSort` / `Partition` / `Interchange`; sentinel $a[n+1]=\infty$; partition $O(n)$
- [ ] Hoare trace on `65 70 75 80 85 60 55 50 45`
- [ ] Lomuto / CLRS partition (last-element pivot) and trace on `2 8 7 1 3 5 6 4`
- [ ] Hoare vs Lomuto comparison table (stable? no; in-place? yes; space $O(\log n)$)
- [ ] Best $T=O(n)+2T(n/2)$; worst $T=O(n)+T(n-1)$; average $O(n\log n)$
- [ ] Asymmetric split $T(\alpha n)+T((1-\alpha)n)+n=O(n\log n)$
- [ ] Pivot ranking: first (sorted) ≻ worst, median = best, random = expected best
- [ ] PYQs: central pivot $O(n^2)$; median pivot $\Theta(n\log n)$; $n/4$ pivot $O(n\log n)$; 1/5 split recurrence (B); $t_1=t_2$ for sorted asc/desc; $t_1>t_2$ for `{1..5}` vs `{4,1,5,3,2}`; $C_1=C_2$; probability $2/25=0.08$; swaps on sorted input

**Matrix Multiplication**
- [ ] Addition $O(n^2)$; school multiplication $O(n^3)$
- [ ] Block formulas $C_{11},C_{12},C_{21},C_{22}$
- [ ] $T(n)=8T(n/2)+bn^2=O(n^3)$
- [ ] (Extra) Strassen: 7 multiplications → $O(n^{\log_2 7})$
