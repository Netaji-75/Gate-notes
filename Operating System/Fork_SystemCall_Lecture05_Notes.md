# Fork System Call — GATE CS/IT OS Notes (Lecture 05)

> Source: GeeksforGeeks GATE CS&IT — *Principles of Operating Systems*, Lecture 05: **Fork System Call**

## 📑 Table of Contents

| # | Topic |
|---|-------|
| 1 | System Calls — Quick Intro |
| 2 | `fork()` Basics & Return Value Rules |
| 3 | Fork Puzzle Framework (unconditional / loops / short-circuit) |
| 4 | Worked Challenges (Level 1 → Ultimate Boss) |
| 5 | `exec()` and `wait()` — How Fork Combines With Them |
| 6 | Formula Cheat-Sheet |
| 7 | GATE PYQs & Practice Questions (with answers) |
| 8 | Memory Tricks / Mnemonics |
| 9 | Quick Revision Checklist |

---

## 1. System Calls — Quick Intro

A **system call** is the programmatic interface through which a user program requests a service from the OS kernel (file I/O, process creation, communication, etc.). `fork()`, `exec()`, and `wait()` are the three system calls this lecture focuses on, since together they explain **process creation** in UNIX/Linux.

---

## 2. `fork()` Basics & Return Value Rules

`fork()` creates a **new process** (the *child*) that is an (almost) exact duplicate of the calling process (the *parent*) — same code, same variable values, same open files — but a **separate memory address space** from that point onward.

### Return value semantics (the golden rule)

| Return value | Meaning | Who receives it |
|---|---|---|
| `< 0` (typically `-1`) | **Fork failed** — no child created | Calling process |
| `== 0` | This is the **child** process | Child |
| `> 0` | This is the **parent** process; value = **child's PID** | Parent |

**General system-call convention:** negative return ⇒ failure, `≥ 0` ⇒ success. `fork()` is a special case of this general rule.

### Minimal template

```c
#include <sys/types.h>
#include <stdio.h>
#include <unistd.h>

int main() {
    pid_t pid;
    pid = fork();

    if (pid < 0) {                 /* error occurred */
        fprintf(stderr, "Fork Failed");
        return 1;
    } else if (pid == 0) {         /* child process */
        execlp("/bin/ls", "ls", NULL);
    } else {                       /* parent process */
        wait(NULL);                /* parent waits for child to finish */
        printf("Child Complete");
    }
    return 0;
}
```

**Key fact:** after `fork()`, parent and child have **independent copies** of all variables. A change made by one is **never** visible to the other — even with a "regular" (non copy-on-write) fork, because the memory copy is logically made *at the instant of the fork() call*.

---

## 3. Fork Puzzle Framework

GATE loves asking "how many times is X printed?" / "how many processes are created?" for snippets full of `fork()`. The lecture builds up a 3-step framework:

### Step 1 — Unconditional forks: the `2^n` rule
Every **unconditional** `fork()` call doubles the number of currently-existing processes (because *every* live process independently executes that same line of code).

$$\text{Total processes (incl. original)} = 2^n \qquad \text{New (child) processes} = 2^n - 1$$

where `n` = number of *unconditional* `fork()` calls each process executes.

### Step 2 — Short-circuit logical operators prune the tree
`&&` and `||` in C are **short-circuit**: the right-hand operand is evaluated **only when the outcome isn't already decided**.

| Operator | Parent path (`fork()` returns PID → **truthy**) | Child path (`fork()` returns `0` → **falsy**) |
|---|---|---|
| **`&&`** (AND) | Returns TRUE → **must** evaluate RHS → **forks again** | Returns FALSE → **short-circuits** → RHS **skipped**, no extra fork |
| **`\|\|`** (OR) | Returns TRUE → **short-circuits** → RHS **skipped**, no extra fork | Returns FALSE → **must** evaluate RHS → **forks again** |

> **Rule of thumb:** `&&` prunes forking on the **child (false)** branch. `||` prunes forking on the **parent (true)** branch.

### Step 3 — Loops multiply
Treat `for`/`while` loops containing `fork()` as **iterative multipliers** — feed the process count from one iteration into the next (see Formula Cheat-Sheet §6 for the exact patterns).

---

## 4. Worked Challenges

### Level 1 — The Unconditional Cascade
```c
int main() {
    fork();
    fork();
    fork();
    return 0;
}
```
**Q: Total processes created (incl. original)?**
Three unconditional forks → $2^3 = 8$.
✅ **Answer: 8**

---

### Level 2 — Interleaved Execution
```c
int main() {
    fork();
    printf("A");
    fork();
    printf("B");
}
```
**Q: How many times is `'A'` printed?**
- After 1st `fork()`: 2 processes exist → each independently executes `printf("A")` once.
✅ **Answer: 2** (times `A` prints). *(Bonus: `B` prints 4 times, since by then 4 processes exist.)*

---

### Level 3 — Asymmetric Execution
```c
int main() {
    if (fork() == 0)
        printf("Child\n");
    else
        printf("Parent\n");
}
```
**Q: Which is correct?**
- Child (`fork()==0`) → prints `"Child"`.
- Parent (`fork()>0`) → prints `"Parent"`.
✅ **Answer: C — Both print, exactly once each.**

---

### Level 4 — The Loop Multiplier
```c
int main() {
    for (int i = 0; i < 2; i++)
        fork();
    return 0;
}
```
Two unconditional forks (once per iteration, applied by every live process) → $2^2 = 4$.
✅ **Answer: 4**

---

### Bonus — Empty-Body `if` Trap
```c
main() {
    int i, n;
    for (i = 1; i <= n; ++i)
        if (fork() == 0);     // ⚠ trailing semicolon = EMPTY if-body!
    printf("*");              // executes unconditionally, once per process, AFTER the loop
}
```
Because of the stray `;`, the `if` does nothing — `fork()` still runs unconditionally every iteration, so this behaves exactly like `n` unconditional forks in a loop.

| Quantity | Formula | Value |
|---|---|---|
| Total processes | $2^n$ | — |
| **New** processes created | $2^n - 1$ | — |
| Times `'*'` printed | $2^n$ (every process reaches the line once) | **(c) $2^n$** ✅ |

> ⚠️ **OCR/typo watch:** a stray semicolon after an `if(...)` silently turns the next line into an unconditional statement — a favorite GATE trap.

---

### Level 5 — The Logical AND Trap
```c
int main() {
    if (fork() && fork())
        printf("X\n");
}
```
Trace using the **`&&` matrix**: the first `fork()` splits into a *truthy parent* branch (which must evaluate the 2nd `fork()`) and a *falsy child* branch (which short-circuits and never calls the 2nd `fork()`).

**Process tree:** Original process → (2nd fork) → 2 more → **3 total processes** in the whole tree.
📌 **Marked answer: B — 3** *(times `'X'` reaches print in this trace — treat this as the tree-size count; use the `&&`/`||` matrix above to re-derive process membership rather than memorizing the number).*

---

### Level 6 — The Logical OR Alternate Escape
```c
int main() {
    if (fork() || fork())
        fork();
    printf("Hi\n");      // OUTSIDE the if — runs unconditionally for every surviving process
}
```
Full trace (using the `||` matrix, `if`-body only conditionally adds **one more** fork, but `printf` is unconditional so every process created reaches it):

| Fork call | New split | Condition true? | Extra fork inside `if`? |
|---|---|---|---|
| F1 (root `fork()`) | Root(T), K1(F) | Root=T (short-circuits `\|\|`) | Root executes if-body → F3 |
| F3 (Root's if-body fork) | Root, K3 | — | both fall through to `printf` |
| F2 (K1 evaluates RHS since K1 is F) | K1(T), K2(F) | K1=T → executes if-body → F4 | K2=F → skips if-body |
| F4 (K1's if-body fork) | K1, K4 | — | both fall through to `printf` |

**Total processes reaching `printf("Hi")`:** Root, K3, K1, K4, K2 = **5**
✅ **Answer: C — 5**

---

### Boss Level — Mixed Operator Logic
```c
int main() {
    fork() || fork() && fork();   // operator precedence: && binds tighter → fork() || (fork() && fork())
    printf("OS\n");               // unconditional — no `if` at all
}
```
Since `&&` has higher precedence than `||`, this parses as `f1 || (f2 && f3)`, evaluated purely for its **side effects** (the boolean result is discarded), then `printf` runs unconditionally for every resulting process.

| Branch | Processes created |
|---|---|
| `f1` truthy (Root) | `\|\|` short-circuits → stops here → **1 process (Root)** |
| `f1` falsy (K1) | must evaluate `(f2 && f3)` → K1 calls `f2` → splits K1(T)/K2(F); K1(T) must call `f3` → splits K1/K3; K2(F) short-circuits, no `f3` → **3 processes (K1, K2, K3)** |

**Total = 1 + 3 = 4 processes**, all print `"OS"`.
✅ **Answer: B — 4**

---

### Ultimate Boss — The Sequential Cascade Trap
```c
int main() {
    fork();                 // Stmt 1: unconditional
    fork() && fork();       // Stmt 2: bare expression, result discarded
    fork();                 // Stmt 3: unconditional
    printf("A\n");          // unconditional, reached by everyone
}
```
Since each statement is **sequential** (not gated by `if`), apply a per-statement **multiplier** to the running process count:

| Statement | Effect on process count | Multiplier | Reasoning |
|---|---|---|---|
| `fork();` | doubles | **×2** | plain unconditional fork |
| `fork() && fork();` | triples (not ×4!) | **×3** | `&&` prunes 1 of the 4 potential branches (child/false branch never calls 2nd fork) — see Level 5 logic |
| `fork();` | doubles | **×2** | plain unconditional fork |

$$\text{Total processes} = 1 \times 2 \times 3 \times 2 = 12$$

All 12 processes execute the unconditional `printf("A\n")`.
✅ **Answer: D — 12**

---

### Synthesis — The Fork Master Framework

| Step | Rule |
|---|---|
| **1. Count Unconditionals** | Apply the $2^n$ rule for every isolated `fork()` — each is a ×2 multiplier |
| **2. Trace Short-Circuits** | Use the AND/OR matrix. `&&` prunes the **child (false)** path; `\|\|` prunes the **parent (true)** path → effectively a **×3 multiplier** (not ×4) per short-circuited pair |
| **3. Multiply by Loops** | Loops feed the output process count of one iteration as the input to the next — see §6 formulas |

---

## 5. `exec()` and `wait()` — How Fork Combines With Them

| System Call | Purpose |
|---|---|
| `fork()` | Creates a **new process** (child), duplicating the caller |
| `exec()` family (e.g. `execlp()`) | **Replaces** the calling process's memory image with a **new program**. Does *not* create a new process — same PID, brand-new code. |
| `wait()` | Called by the parent to **block until a child terminates** |

```
parent ──► pid = fork() ──► parent (pid>0) ──► wait() ──► parent resumes
                    └──────► child (pid=0) ──► exec() ──► exit()
```

### `exec()` syntax
```c
execlp("PathName", "ProgName", "Parameters", NULL);

// Example — runs the equivalent of shell command: $ ls
execlp("/bin/ls", "ls", NULL);
```

> 🧠 **Key insight:** If `execlp()` **succeeds**, it **never returns** — every line of code *after* it in the child is simply never reached. Code after `exec()` runs **only if `exec()` fails**.

### Worked example combining fork + exec
```c
main() {
    int a = 5;
    a += 10;
    printf("%d", a);              // prints 15 — BEFORE the fork, only original process runs this
    if (fork() == 0) {
        execlp("/bin/ls", "ls", NULL);   // child's image is replaced — output = file listing
        a *= 5;                          // ⚠ NEVER executed if execlp succeeds
        printf("%d", a);                 // ⚠ NEVER executed
    } else {
        a += 100;
        printf("%d", a);          // parent prints 115 (15 + 100)
    }
}
```
**Output:** `15` (before fork) → `115` (parent) → *directory listing* (child, via `ls`, NOT any number).

---

## 6. Formula Cheat-Sheet

| # | Pattern | Formula | Notes |
|---|---|---|---|
| 1 | `n` sequential/looped **unconditional** `fork()` calls | $\text{Total} = 2^n$, $\text{New} = 2^n - 1$ | Base rule |
| 2 | `for(i=1;i<=n;++i) if(fork()==0) print();` | $\text{Prints} = 2^n - 1$ | Only the **child** of each fork call prints — 1 print per fork() call executed anywhere in the tree |
| 3 | `for(i=1;i<=n;++i){ fork(); print(); }` | $\text{Prints} = 2^{n+1} - 2$ | print is **unconditional** and comes **after** fork → all processes existing *after* the fork print, each iteration |
| 4 | `for(i=1;i<=n;++i){ print(); fork(); }` | $\text{Prints} = 2^n - 1$ | print comes **before** fork → only processes existing *before* that iteration's fork print |
| 5 | `while(x>0){ fork(); print(); wait(NULL); x--; }`, starting $x=k$ | $\text{Prints} = \sum_{i=1}^{k} 2^i = 2^{k+1}-2$ | `wait()` doesn't prevent the child from also looping — both parent & child re-enter the loop independently |
| 6 | Recursive: `f(n){ if(n>0){ fork(); f(n-1); } }` | $P(0)=1,\; P(n) = 2\cdot P(n-1) \Rightarrow P(n) = 2^n$ | $P(n)$ = total processes incl. original; **new** processes = $2^n - 1$ |
| 7 | `&&` short-circuit pair (`fork() && fork();`, side-effect only) | Multiplier = **×3** (not ×4) | Child/false branch of 1st fork never calls the 2nd |
| 8 | `\|\|` short-circuit pair (`fork() \|\| fork();`) | Multiplier = **×3** (not ×4) | Parent/true branch of 1st fork never calls the 2nd |

**Variable legend:** $n$ / $k$ = number of loop iterations or unconditional fork calls; $P(n)$ = total live processes after $n$ levels of recursive forking.

---

## 7. GATE PYQs & Practice Questions

### Q1 — Fork in a loop, gated print
```c
main() {
    int i, n;
    for (i = 1; i <= n; ++i)
        if (fork() == 0)
            print("GfG");
}
```
**Number of times `"GfG"` is printed?**
✅ **Answer: $2^n - 1$** — one print per `fork()` call, contributed only by the child of that call (Formula #2).

---

### Q2 — Fork, then print (unconditional, inside loop)
```c
main() {
    int k, n;
    for (k = 1; k <= n; ++k) {
        fork();
        print("GfG");
    }
}
```
✅ **Answer: $2^{n+1} - 2$** (Formula #3).

---

### Q3 — Print, then fork (order reversed)
```c
main() {
    int k, n;
    for (k = 1; k <= n; ++k) {
        print("GfG");
        fork();
    }
}
```
✅ **Answer: $2^n - 1$** (Formula #4) — same numeric answer as Q1, but for a different structural reason (print happens *before* the doubling each round).

---

### Q4 — Parent/Child value & address tracking (classic GATE 2014-style)
```c
main() {
    int a;
    if (fork() == 0) {
        a = a + 5;
        printf("%d, %p\n", a, &a);   // u, v  → wait, actually child prints x, y
    } else {
        a = a - 5;
        printf("%d, %p\n", a, &a);   // u, v printed by PARENT
    }
}
```
Let `u, v` = values printed by **parent**; `x, y` = values printed by **child**.

| Option | Statement |
|---|---|
| A | $u = x+10$ and $v = y$ |
| B | $u = x+10$ and $v \neq y$ |
| **C** ✅ | $u + 10 = x$ and $v = y$ |
| D | $u + 10 = x$ and $v \neq y$ |

**Why C:** child does `a+5`, parent (else branch) does `a-5` → child's value is always 10 more than parent's ⇒ $x = u+10$. Both processes have an **identical virtual address** for `a` (fork() preserves the virtual address layout) ⇒ $v = y$, even though the physical memory pages differ.
**Why others wrong:** A/B reverse the +10 direction; B/D wrongly claim addresses differ (virtual addresses are always equal post-fork).

---

### Q5 — Nested fork with local variables (multi-level tree)
```c
main() {
    int i = 1, j = 2, k = 3;
    i += j += ++k;
    print(i, j, k);
    if (fork() == 0) {
        int d;
        ++i; ++j; --k;
        print(i, j, k);
        if (fork() == 0) {
            d = i + j + k;
            print(i, j, k, d);
        }
    } else {
        --i; --j;
        k = i + j;
        d = i + j + k;
        print(i, j, k, d);
    }
}
```
This illustrates a **3-process chain**: Parent (P) → Child 1 (C1, from the first `fork()`) → Child 2 (C2, nested `fork()` reachable **only** from inside C1's branch). Each process holds its **own independent copies** of `i, j, k, d` from the moment its `fork()` executed.
**Takeaway for GATE:** count nested forks as a **linear chain**, not `2^n` — a fork nested *inside only one branch* of a previous fork does **not** double the whole tree, it only extends that single branch.

---

### Q6 — Fork inside a conditional loop
```c
#include <unistd.h>
int main() {
    int i;
    for (i = 0; i < 10; i++)
        if (i % 2 == 0) fork();
}
```
**Total number of child processes created?**
`i%2==0` is true at `i = 0,2,4,6,8` → **5 effective unconditional fork points** (every surviving process re-executes each of these 5 points as it loops through).
$$\text{Total processes} = 2^5 = 32 \quad\Rightarrow\quad \text{New (child) processes} = 2^5 - 1 = \mathbf{31}$$
✅ **Answer: 31**

---

### Q7 — Fork + wait() in a while loop
```c
int x = 3;
while (x > 0) {
    fork();
    printf("hello");
    wait(NULL);
    x--;
}
```
**Total number of times `printf` executes?**

| Iteration | Processes entering | Prints this round |
|---|---|---|
| 1st ($x=3$) | 1 | $2^1 = 2$ |
| 2nd ($x=2$) | 2 | $2^2 = 4$ |
| 3rd ($x=1$) | 4 | $2^3 = 8$ |
| **Total** | | **2+4+8 = 14** |

✅ **Answer: 14** — `wait()` synchronizes *when* a process resumes but does **not** stop the child from also independently re-entering the loop and forking again.

---

### Q8 — Single-processor system property
**For a Single-Processor system:**
| Option | Statement |
|---|---|
| A | Processes spend long times waiting to execute |
| **B** ✅ | There will never be more than one **running** process |
| C | Input-output always causes CPU slowdown |
| D | Process scheduling is always optimal |

**Why B:** a single CPU core can execute only one instruction stream at a time — only one process can be in the **Running** state at any instant (others are Ready/Waiting).
**Why others wrong:** A, C, D are not guaranteed/universal facts about single-processor systems.

---

### Q9 — Fork return-value guarantees
> **S1:** A successful call to `fork()` is guaranteed to return **different** values in the parent and child processes.
> **S2:** On a successful call to `fork()`, the returned child PID is guaranteed to be **unique** across all PIDs of running processes in the system.

| Option | Statement |
|---|---|
| A | Only S1 |
| B | Only S2 |
| **C** ✅ | Both |
| D | None |

**Why C:** S1 is true by definition (`0` to child, nonzero PID to parent — always different). S2 is true because the OS's PID-allocation scheme guarantees uniqueness among currently active processes.

---

### Q10 — Counting distinct memory copies
```c
main(int argc, char **argv) {
    int child = fork();
    int c = 5;
    if (child == 0) {
        c += 5;
    } else {
        child = fork();
        c += 10;
        if (child)
            c += 5;
    }
}
```
**Q: How many different copies of variable `c` exist?**
Process tree: **P** (original) → **C1** (from 1st fork, takes the `if` branch — no further fork) ; **P** also reaches the **`else`** branch → calls 2nd `fork()` → **C2**.
Total distinct processes = 3 (**P, C1, C2**) ⇒ 3 independent copies of `c`.
✅ **Answer: 3**

---

### Q11 — Recursive fork counting
```c
main(int argc, char **argv) {
    forkthem(5);
}
void forkthem(int n) {
    if (n > 0) {
        fork();
        forkthem(n - 1);
    }
}
```
**Q: How many processes are created (new processes) when this runs?**

Let $P(n)$ = total processes resulting from a call `forkthem(n)`:
$$P(0) = 1, \qquad P(n) = 2 \cdot P(n-1) \;\Rightarrow\; P(n) = 2^n$$

For $n=5$: $P(5) = 2^5 = 32$ total processes ⇒ **new** processes created $= 2^5 - 1 = \mathbf{31}$.
✅ **Answer: 31**

---

### Q12 — Copy timing of a global variable
```c
int count = 0;
ret = fork();
if (ret == 0) {
    printf("count in child=%d\n", count);
} else {
    count = 1;
}
```
> The parent executes `count = 1` **before** the child executes for the first time. What value does the child print? (Assume a **regular** — non copy-on-write — fork.)

✅ **Answer: 0** — the child's copy of `count` was snapshotted **at `fork()` time**, when `count` was still `0`. Since parent and child have **separate memory** after fork, the parent's *later* assignment `count = 1` never reaches the child's copy — regardless of execution order.

---

### Q13 — Galvin textbook classic
```c
int value = 5;
int main() {
    pid_t pid;
    pid = fork();
    if (pid == 0) {                      /* child */
        value += 15;
        return 0;
    } else if (pid > 0) {                /* parent */
        wait(NULL);
        printf("PARENT: value = %d", value);  /* LINE A */
        return 0;
    }
}
```
**Q: What value is printed at LINE A?**
✅ **Answer: 5** — the child's `value += 15` only modifies **its own copy**; the parent's copy is untouched, so `wait()` finishing doesn't change what the parent prints.

---

### Q14 — When is LINE J reached?
```c
else if (pid == 0) {  /* child process */
    execlp("/bin/ls", "ls", NULL);
    printf("LINE J");   /* LINE J */
}
```
**Q: Under what condition does `LINE J` execute?**
✅ **Answer: Only if `execlp()` fails.** If `execlp()` succeeds, the child's memory image is replaced entirely and control **never returns** to any code after it.

---

### Q15 — `pid` variable vs actual PID via `getpid()`
```c
pid_t pid, pid1;
pid = fork();
if (pid == 0) {                       /* child */
    pid1 = getpid();
    printf("child: pid = %d", pid);   /* A */
    printf("child: pid1 = %d", pid1); /* B */
} else {                              /* parent */
    pid1 = getpid();
    printf("parent: pid = %d", pid);  /* C */
    printf("parent: pid1 = %d", pid1);/* D */
    wait(NULL);
}
```
Suppose parent's real PID = 823, child's real PID = 715.

| Line | Prints | Value |
|---|---|---|
| A (child, `pid`) | fork() return value stored in child | **0** |
| B (child, `pid1`) | child's real PID via `getpid()` | **715** |
| C (parent, `pid`) | fork() return value stored in parent = child's PID | **715** |
| D (parent, `pid1`) | parent's real PID via `getpid()` | **823** |

**Takeaway:** the **stored** `fork()` return value (`pid`) and the process's **actual own PID** (`getpid()`) are different things — don't conflate them.

---

## 8. Memory Tricks / Mnemonics

- 🧠 **`< 0`, `== 0`, `> 0`** → **Fail, Child, Parent** (in that literal numeric order — easy to remember as "negative-zero-positive").
- 🧠 **`&&` = "AND punishes the child"** — the false/child branch short-circuits and never gets to fork again.
- 🧠 **`||` = "OR punishes the parent"** — the true/parent branch short-circuits and never gets to fork again.
- 🧠 **A stray semicolon after `if(...)` is a silent trap** — turns the next statement into an unconditional one.
- 🧠 **`exec()` = "process gets a new brain, keeps the old skull"** — same PID, entirely new program; if it succeeds, it *never returns*.
- 🧠 **`fork()` never shares memory, ever** — even without copy-on-write, the snapshot is taken *at the moment of the call*, so timing of later writes never crosses process boundaries.
- 🧠 **`wait()` synchronizes *timing*, not *memory*** — a child finishing before a parent reads a variable does **not** make the parent see the child's changes.

---

## 9. Quick Revision Checklist

- [ ] `fork()` return value rules: `<0` fail, `==0` child, `>0` parent (with child's PID)
- [ ] Total processes from `n` unconditional forks = $2^n$; new processes = $2^n - 1$
- [ ] `&&` short-circuit prunes the child/false branch's further forking
- [ ] `\|\|` short-circuit prunes the parent/true branch's further forking
- [ ] A short-circuited fork pair gives a **×3** multiplier, not ×4
- [ ] Loop + gated print (`if(fork()==0) print()`) → prints = $2^n - 1$
- [ ] Loop with `fork(); print();` (print after) → prints = $2^{n+1}-2$
- [ ] Loop with `print(); fork();` (print before) → prints = $2^n - 1$
- [ ] `while` + `fork()` + `wait()` loop → prints = $\sum 2^i$ (wait does NOT stop child from also looping)
- [ ] Recursive forking: $P(n) = 2\cdot P(n-1)$, $P(0)=1 \Rightarrow P(n)=2^n$
- [ ] Nested fork **inside only one branch** extends a chain, does not double the whole tree
- [ ] `exec()` replaces process image; code after successful `exec()` never runs
- [ ] `wait()` blocks parent until child terminates, but does not share memory
- [ ] Virtual addresses of variables are identical in parent & child after fork, even though physical memory differs
- [ ] A single-processor system can have only **one Running process** at any instant
- [ ] fork() return values are guaranteed different (parent vs child) AND guaranteed unique PIDs across the system
- [ ] Counting "distinct copies of a variable" = counting **distinct processes** in the tree
- [ ] Stored `fork()` return value (`pid`) ≠ actual process ID via `getpid()`

---

*Notes compiled from GeeksforGeeks GATE CS&IT — Principles of OS, Lecture 05 (Fork System Call).*
