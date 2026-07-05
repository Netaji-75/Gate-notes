# Lecture 04 — System Calls, Privileged Instructions & Mode Switching (GATE CS/IT — OS)

> Source: GeeksforGeeks GATE CS&IT — Principles of Operating Systems, Lecture 04

## 📌 Topics Covered (Table of Contents)

| # | Topic |
|---|-------|
| 1 | Modes of Execution & Mode Switching |
| 2 | Privileged Instructions |
| 3 | Trap Mechanism |
| 4 | System Calls — definition, types, implementation |
| 5 | System Call Parameter Passing |
| 6 | Types of System Calls (Process, File, Device, Info, Comm, Protection) |
| 7 | UNIX vs Windows System Call Examples |
| 8 | System Programs |
| 9 | `fork()` System Call — Case Study & Process Tree Problems |
| 10 | UNIX `fork()+exec()+wait()` Flow |
| 11 | GATE PYQs & Practice MCQs |

---

## 1. Modes of Execution & Dual-Mode Architecture

**Why two modes?** Without boundaries, a single buggy/malicious user program could overwrite critical memory, hijack the CPU, or crash the entire system.

- **User Mode** — restricted, "safe" zone. Application programs (browsers, text editors, compilers) run here. **No direct hardware access.**
- **Kernel Mode** — privileged, "high-risk" zone. OS runs here with **full access** to hardware, memory, and devices.

### The Mode Bit
A dedicated hardware bit in the CPU tells the current privilege level.

| Mode Bit Value | Mode | Access |
|---|---|---|
| **1** | User Mode | Restricted, indirect hardware access (via OS only) |
| **0** | Kernel Mode | Full, direct, unrestricted hardware access |

> 🧠 **Mnemonic:** *Mode bit = 1 → "1 user standing outside"; Mode bit = 0 → "0 restrictions inside kernel."*

**The Rule:** Hardware continuously checks this bit. If a program attempts a restricted action while bit = 1 (user mode), the CPU **forcibly rejects it** (generates a trap/exception).

### User Mode vs Kernel Mode — Comparison

| Feature | User Mode | Kernel Mode |
|---|---|---|
| Mode Bit | 1 | 0 |
| Status | Restricted, Safe ("Public Lobby") | Privileged, High-Risk ("Control Room") |
| Hardware Access | Indirect (requires OS) | Direct & Unrestricted |
| Example processes | Text editors, browsers, compilers | Process scheduling, memory management, device drivers, file systems |
| Failure consequence | Only that app crashes | **Entire OS crashes** |

---

## 2. Privileged Instructions

**Definition:** An instruction that can be executed **only in Kernel Mode**. If attempted in user mode → processor generates an **exception/trap**.

**Core rule:** Privileged instructions are those that can alter the **state or configuration of the entire system** — not just a local variable/app.

### Examples of Privileged Instructions
- Enable / Disable Interrupts
- Set Timer Registers
- Load Page-Table Base Register
- Modify Memory Protection Registers
- Perform Direct I/O Operations
- Halt the Processor
- Change Processor State

### Why are they necessary?
Without privileged instructions, any application could:
- Access another program's memory
- Disable interrupts
- Corrupt OS data
- Gain unauthorized hardware access
- Crash the entire system

### Role in Memory Management
Privileged instructions configure MMU registers, update page tables, enable paging, modify address-translation parameters, and control memory protection.

### Role in Process Management
The OS needs privileged instructions for: context switching, scheduling, timer management, process creation, and process termination.

### GATE-Style Rule of Thumb
> **If the instruction can affect the stability of the ENTIRE system → Privileged.**
> **If it only affects the local app/variables → NOT Privileged.**

| Instruction | Privileged? | Why |
|---|---|---|
| Modifying a Page Table | ✅ Yes | Alters memory mapping for the entire system |
| Integer Addition | ❌ No | Only affects local application variables |
| Disable Interrupts | ✅ Yes | System-wide effect |
| String comparison | ❌ No | Local operation |

---

## 3. Trap Mechanism (Crossing the User↔Kernel Boundary)

Applications **cannot** directly execute privileged instructions. They must invoke a **system call**, which internally uses a **trap**.

### Three Triggers that force Mode Bit: 1 → 0 (User → Kernel)

| Trigger | Origin | Intent | Example |
|---|---|---|---|
| **System Call** | Internal (application) | Voluntary / requested | `read()`, `fork()` |
| **Exception / Trap** | Internal (CPU execution) | Involuntary — error or violation | Divide by zero, illegal privileged-instruction attempt |
| **Interrupt** | External (hardware/device) | Involuntary — asynchronous | Keystroke, disk I/O completion |

All three **universally force the Mode Bit to 0** (User → Kernel).

> 🧠 **Myth vs Reality:**
> - ❌ *"System calls and privileged instructions are the same thing."* → ✅ System calls are **requests** made by the user program; privileged instructions are the actual hardware commands the OS executes as a result. (System calls *use* privileged instructions internally.)
> - ❌ *"Interrupts and traps are interchangeable."* → ✅ Interrupts = **external** (hardware). Traps = **internal** (software exceptions or intentional syscalls).
> - ❌ *"Mode switching is instant and free."* → ✅ It requires **state saving/restoring**, causing measurable performance overhead.

### The "Trap → Kernel → Return" Flow

```
USER MODE (mode bit = 1)
  User Process Executing
        ↓
  Get System Call ──────► [trap, mode bit = 0]
        ↑                        ↓
  Return From System Call   KERNEL MODE
        ↑                  Execute System Call (mode bit = 0)
        └───── [trap, mode bit = 1] ◄──┘
```

**Step-by-step (Trap Mechanism):**
1. CPU detects the privileged-instruction violation attempt.
2. Hardware generates a **Trap**.
3. Control transfers to the OS.
4. OS handles the exception/service appropriately.
5. Control (and mode bit) is restored, returning to User Mode.

### Cost of Crossing: Context Switch Overhead
Before the CPU can service an interrupt/trap, it must **freeze and save the exact state** (registers, program counter) of the currently running user program.

**The Context-Switch Cycle:**
User Mode Execution → **Save State** → Switch to Kernel (bit=0) & Execute Service → **Restore State** → Return to User Mode (bit=1)

> ⚠️ **Trade-off:** Frequent mode switching (e.g., poorly optimized I/O loops) **severely degrades system speed** due to constant state saving/restoring. This is a classic GATE trap-answer topic.

---

## 4. System Calls

### Definition
A **system call** is the **programming interface** to the services provided by the OS — the *only* legal way for a user program to request privileged/kernel services.

- Typically written in a high-level language (C or C++).
- Mostly accessed via a high-level **API** (Application Program Interface) rather than direct system-call use.
  - e.g. `fork()`, `read()`, `brk()` are wrapped by APIs.

### Three Common APIs

| API | Platform |
|---|---|
| **Win32 API** | Windows |
| **POSIX API** | UNIX, Linux, Mac OS X (all POSIX-based systems) |
| **Java API** | Java Virtual Machine (JVM) |

### Function Categories (Handwritten Diagram Recap)

```
                 Functions (fns)
        ┌──────────────┬──────────────┐
   User-Defined   Predefined Library   System Calls
   e.g. foo(){      e.g. sqrt(), malloc()   e.g. fork(), read(),
        fork();  }                          write(), getpid()
                    │
        ┌───────────┴───────────┐
   Runs in User Mode (UM)    UM → KM transition
   e.g. strcmp, strcat       e.g. printf()/scanf() (stdio.h)
                              internally call write()/read() (syscalls)
```
> 🧠 **Key distinction:** A library call (e.g. `printf()`) may **internally** invoke a system call (`write()`), but not every library function does — e.g. `strcmp()`/`strcat()` stay entirely in User Mode.

### System Call Implementation — "Dispatch Table"
- Each system call has an associated **number**.
- The **system-call interface** maintains a table indexed by these numbers — called the **Dispatch Table**.
- The interface invokes the intended kernel routine and returns status + return values.
- **The caller needs to know NOTHING about the implementation** — just obey the API. Most OS-interface detail is hidden by the API.
- Managed by the **run-time support library** (functions built into libraries shipped with the compiler).

**Flow diagram:**
```
User Application → open() [User Mode]
        │
   System Call Interface  (boundary)
        │
   Dispatch Table (indexed by syscall number i)
        │
        ▼
   Implementation of open() system call → Return
                                        [Kernel Mode]
```

### Standard C Library Example
`printf()` (library call) internally invokes the `write()` **system call**.

```c
#include <stdio.h>
int main()
{
    printf("Greetings");   // library call
    return 0;
}
```
Flow: `printf()` [User Mode, Standard C Library] → `write()` [Kernel Mode, actual system call]

---

## 5. System Call Parameter Passing

Often more information than just the syscall's identity is required. **Three general methods** to pass parameters to the OS:

1. **Simplest: pass parameters in Registers**
   - Limitation: there may be more parameters than available registers.
2. **Parameters stored in a block/table in Memory**, and the *address* of that block is passed as a parameter in a register.
   - This approach is used by **Linux and Solaris**.
3. **Parameters placed/pushed onto the Stack** by the program, and popped off by the OS.
   - Block and stack methods do **not** limit the number/length of parameters passed (unlike the register method).

### Parameter Passing via Table (Diagram)

```
user program                  register           Operating System
┌─────────────────┐                              ┌───────────────────┐
│ X: parameters    │──────────► [ X ] ───────────►│ use parameters    │
│    for call       │                              │ from table X      │  code for
│ load address X    │──────────────────────────────►│                   │  system call 13
│ system call 13     │                              └───────────────────┘
└─────────────────┘
```

---

## 6. Types of System Calls

### a) Process Control
`end`, `abort`, `load`, `execute`, `create process`, `terminate process`, `get/set process attributes`, `wait for time`, `wait event`, `signal event`

### b) File Management
`create file`, `delete file`, `open`, `close`, `read`, `write`, `reposition`, `get/set file attributes`

### c) Device Management
`request device`, `release device`, `read`, `write`, `reposition`, `get/set device attributes`, `logically attach/detach devices`

### d) Information Maintenance
`get/set time or date`, `get/set system data`, `get/set process/file/device attributes`

### e) Communications
- Create/delete communication connection
- Send/receive messages (message-passing model, client ↔ server)
- Shared-memory model: create and access memory regions
- Transfer status info, attach/detach remote devices

### f) Protection
Control access to resources; get/set permissions; allow/deny user access

### Common Process-Management System Calls

| Call | Description |
|---|---|
| `pid = fork()` | Create a child process identical to the parent |
| `pid = waitpid(pid, &statloc, options)` | Wait for a child to terminate |
| `s = execve(name, argv, environp)` | Replace a process' core image (run new program) |
| `exit(status)` | Terminate process execution and return status |

### Common File-Management System Calls

| Call | Description |
|---|---|
| `fd = open(file, how, ...)` | Open a file for reading, writing, or both |
| `s = close(fd)` | Close an open file |
| `n = read(fd, buffer, nbytes)` | Read data from a file into a buffer |
| `n = write(fd, buffer, nbytes)` | Write data from a buffer into a file |
| `position = lseek(fd, offset, whence)` | Move the file pointer |
| `s = stat(name, &buf)` | Get a file's status information |

### Directory / File-System Management System Calls

| Call | Description |
|---|---|
| `s = mkdir(name, mode)` | Create a new directory |
| `s = rmdir(name)` | Remove an empty directory |
| `s = link(name1, name2)` | Create a new entry, name2, pointing to name1 |
| `s = unlink(name)` | Remove a directory entry |
| `s = mount(special, name, flag)` | Mount a file system |
| `s = umount(special)` | Unmount a file system |

### Miscellaneous System Calls

| Call | Description |
|---|---|
| `s = chdir(dirname)` | Change the working directory |
| `s = chmod(name, mode)` | Change a file's protection bits |
| `s = kill(pid, signal)` | Send a signal to a process |
| `seconds = time(&seconds)` | Get elapsed time since Jan 1, 1970 |

---

## 7. UNIX vs Windows — System Call Comparison

| Category | Windows | Unix |
|---|---|---|
| Process control | `CreateProcess()`, `ExitProcess()`, `WaitForSingleObject()` | `fork()`, `exit()`, `wait()` |
| File manipulation | `CreateFile()`, `ReadFile()`, `WriteFile()`, `CloseHandle()` | `open()`, `read()`, `write()`, `close()` |
| Device manipulation | `SetConsoleMode()`, `ReadConsole()`, `WriteConsole()` | `ioctl()`, `read()`, `write()` |
| Information maintenance | `GetCurrentProcessID()`, `SetTimer()`, `Sleep()` | `getpid()`, `alarm()`, `sleep()` |
| Communication | `CreatePipe()`, `CreateFileMapping()`, `MapViewOfFile()` | `pipe()`, `shmget()`, `mmap()` |
| Protection | `SetFileSecurity()`, `InitializeSecurityDescriptor()`, `SetSecurityDescriptorGroup()` | `chmod()`, `umask()`, `chown()` |

---

## 8. System Programs

Provide a convenient environment for program development and execution — some are simple UI wrappers around system calls; others are far more complex.

- **File management** — create, delete, copy, rename, print, dump, list, manipulate files/directories
- **Status information** — date, time, available memory, disk space, number of users; some provide detailed performance/logging/debugging info; typically formatted and printed to terminal; some systems use a **registry** to store/retrieve configuration info
- **File modification** — text editors; commands to search/transform file contents
- **Programming-language support** — compilers, assemblers, debuggers, interpreters
- **Program loading & execution** — absolute loaders, relocatable loaders, linkage editors, overlay-loaders, debugging systems
- **Communications** — virtual connections among processes/users/systems — messaging, web browsing, email, remote login, file transfer

---

## 9. `fork()` System Call — Case Study

**`fork()` creates a new/child process.** The child is (initially) an identical copy of the parent, and *both* continue execution from the statement right after the `fork()` call.

### Case 1 — No fork
```c
main()
{
    printf("Hello");
}
```
Only **1 process**, prints `Hello` **once**.

### Case 2 — Single fork
```c
main()
{
    fork();
    printf("Hello");
}
```
- `fork()` executes only **once**, in the **parent** (the child never re-executes statements *before* the fork point — it starts execution *from* the `fork()` return).
- **Parent:** executes `fork()` then `printf()` → prints `Hello`
- **Child:** starts right after `fork()` → prints `Hello`
- **Output:** `Hello` printed **2 times** total.

### Case 3 — Two forks + one printf
```c
main()
{
    fork();
    fork();
    printf("Hi");
}
```
Process tree:
```
                Parent
               /      \
             C1        C3
            /
          C2
```
- Parent → 2 forks → 4 total processes (Parent, C1, C2, C3) → each independently reaches `printf("Hi")`.
- **Output:** `Hi` printed **4 times**.

### Case 4 — printf interleaved with forks
```c
main()
{
    printf("Hi");     // → 1 (only parent runs this line, since it's BEFORE any fork)
    fork();
    printf("Hello");  // → 2 (2 processes now: parent + 1 child)
    fork();
    printf("Done");   // → 4 (4 processes now)
}
```
| Statement | Processes alive at that point | Times printed |
|---|---|---|
| `printf("Hi")` | 1 | 1 |
| `printf("Hello")` | 2 | 2 |
| `printf("Done")` | 4 | 4 |

> 🧠 **Mnemonic:** Every `printf` after `fork()` gets executed by **every process that exists at that point** in the code, since each process resumes execution from right after the last fork it "witnessed."

### Case 5 — Three forks, one printf at the end
```c
main()
{
    fork();
    fork();
    fork();
    printf("*");
}
```
3 forks in series → total processes = $2^3 = 8$ → `printf("*")` executes 8 times.

### General Formula — `fork()` in Series

$$\text{Total Processes} = 2^n$$
$$\text{Number of Child Processes} = 2^n - 1$$

Where:
- $n$ = number of `fork()` calls in a straight-line series (each fork executed once by every process alive at that point)

| No. of `fork()` in series | Total Processes | No. of Child Processes |
|---|---|---|
| 1 | 2 | 1 |
| 2 | 4 | 3 |
| 3 | 8 | 7 |
| 4 | 16 | 15 |
| $n$ | $2^n$ | $2^n - 1$ |

### `fork()` Inside a Loop — Special Case

```c
main()
{
    int i, n;
    for (i = 1; i <= n; ++i)
        fork();
}
```
> ⚠️ **This is NOT the same as `n` forks in series** — because each iteration's `fork()` call is executed independently by **every process that currently exists**, and moreover each child also *continues the loop* from its own `i` value onward — leading to complex branching trees (see the worked handwritten diagram in the lecture for `n=2`: Parent runs 2 loop iterations, C1 (child of iter-1) runs only iter-2, C2 runs nothing further, etc.)

> 🧠 **GATE Tip:** For `fork()` **inside a for-loop**, don't blindly apply $2^n$ — trace the process tree carefully, since children created in later iterations do NOT re-execute earlier iterations.

---

## 10. UNIX `fork() + exec() + wait()` — Standard Idiom

**Pattern:**
- `fork()` system call creates a new process.
- `exec()` system call is used *after* a `fork()` to **replace the process's memory space with a new program**.
- Parent process calls `wait()`, waiting for the child to terminate.

```
parent ──► pid = fork() ──┬── parent (pid > 0) ──► wait() ──► parent resumes
                            └── child (pid = 0) ──► exec() ──► exit()
```

### C Code Example

```c
#include <sys/types.h>
#include <stdio.h>
#include <unistd.h>

int main()
{
    pid_t pid;

    /* fork a child process */
    pid = fork();

    if (pid < 0) {                    /* error occurred */
        fprintf(stderr, "Fork Failed");
        return 1;
    } else if (pid == 0) {            /* child process */
        execlp("/bin/ls", "ls", NULL);
    } else {                          /* parent process */
        /* parent will wait for the child to complete */
        wait(NULL);
        printf("Child Complete");
    }
    return 0;
}
```

> 🧠 **Mnemonic:** `pid < 0` → fork failed; `pid == 0` → you're in the **child**; `pid > 0` → you're in the **parent** (and `pid` holds the child's PID).

---

## 11. GATE-Style / Practice MCQs

### Q1. Which of the following is generally a Privileged instruction?
(a) Integer addition  (b) **Disable interrupts** ✅  (c) String comparison  (d) Multiplication

> **Reasoning:** (a), (c), (d) only affect local computation/variables. (b) affects system-wide interrupt handling → requires kernel mode.

### Q2. A user process attempting to execute a Privileged Instruction causes:
(a) Normal execution  (b) **Trap/Exception** ✅  (c) Context switch only  (d) Compilation error

> **Reasoning:** Privileged instructions attempted in user mode are rejected by hardware, generating a trap/exception — not silently executed, not a mere context switch, and definitely not a compile-time issue (it's a runtime hardware check).

### Q3. Which mode allows execution of Privileged Instructions?
(a) User mode  (b) **Kernel mode** ✅  (c) Batch mode  (d) Virtual mode

> **Reasoning:** By definition, privileged instructions can only run when mode bit = 0 (Kernel Mode). "Batch mode" and "virtual mode" aren't privilege levels at all.

### Q4. The primary purpose of Privileged Instructions is:
(a) Increase speed  (b) Reduce memory  (c) **Provide protection** ✅  (d) Reduce compilation time

> **Reasoning:** Their entire purpose is system protection (isolating processes, safeguarding hardware) — not performance or memory optimization.

### Q5. Page table modification is usually performed in:
(a) User mode  (b) **Kernel mode** ✅  (c) Debug mode  (d) Batch mode

> **Reasoning:** Modifying page tables affects the entire system's memory mapping → a privileged operation, hence kernel mode only.

### Q6. The interface between User Programs and the OS is provided by:
(a) Compiler  (b) **System call Interface** ✅  (c) Scheduler  (d) File system

> **Reasoning:** Compiler translates code, scheduler picks processes to run, file system manages storage — none of these is the *interface* for requesting OS services; that's specifically the system call interface.

### Q7. Which of the following best defines a System Call?
(a) A command entered by the user in the terminal  (b) A function that manages memory allocation  (c) **A request by a program to the OS to perform a service** ✅  (d) A direct access to hardware

> **Reasoning:** (a) describes a shell command, (b) describes a library function like `malloc`, (d) is impossible from user mode — system calls exist precisely because direct hardware access isn't allowed.

### Q8. Which of the following is/are NOT a type of System Call?
(a) Process control  (b) File manipulation  (c) **Image Editor** ✅ (NOT a syscall type)  (d) **Web browsing** ✅ (NOT a syscall type)

> **Reasoning:** Process control and file manipulation are official syscall categories. "Image editor" and "web browsing" are *applications* that internally use many system calls — they are not categories of system calls themselves.

### Q9. Which System Call is used to create a new Process in Unix/Linux systems?
(a) `exec()` — executes a specified program (doesn't create a new process)  (b) **`fork()`** ✅  (c) `exit()`  (d) `kill()`

> **Reasoning:** `exec()` replaces the current process image (no new process); `exit()` terminates; `kill()` sends a signal. Only `fork()` actually creates a new process.

### Q10. The `exec()` system call is used to:
(a) Create a new process  (b) Terminate a process  (c) **Replace the current process image with a new process image** ✅  (d) Duplicate a process

> **Reasoning:** `fork()` creates/duplicates; `exit()` terminates. `exec()`'s defining job is to load a new program into the *existing* process's memory space.

### Q11. What does the `wait()` system call do?
(a) Terminates the calling process  (b) Pauses the process for a fixed amount of time  (c) **Waits for child process to complete** ✅  (d) Waits for a file to be opened

> **Reasoning:** `wait()` specifically blocks the parent until a child process terminates — it's not a timer (that's `sleep()`), not `exit()`, and not I/O-related.

### Q12. System Calls are usually implemented using:
(a) High-level languages only  (b) **APIs and interrupts** ✅  (c) Database queries  (d) BIOS routines

> **Reasoning:** System calls are invoked via APIs (Win32/POSIX/Java) and internally trigger software interrupts (traps) — not database queries or BIOS routines.

### Q13. Which system call is/are NOT a System Call?
(a) `halt()` — is a syscall  (b) `fork()` — is a syscall  (c) **`strcmp()`** ✅ (NOT a syscall)  (d) **`sqrt()`** ✅ (NOT a syscall)

> **Reasoning:** `halt()` and `fork()` require kernel/privileged action. `strcmp()` and `sqrt()` are pure user-mode library functions — no kernel involvement needed.

### Q14. Which of the following is/are a File management system call?
(a) **`open()`** ✅  (b) `exec()` — process control, not file mgmt  (c) **`lseek()`** ✅  (d) **`read()`** ✅

> **Reasoning:** `open()`, `lseek()`, and `read()` all directly manipulate files. `exec()` belongs to the process-control category instead.

### Q15. A process can be terminated by which System Call?
(a) **`kill()`** ✅  (b) `fork()` — creates a process  (c) `mount()` — filesystem mounting  (d) `nice()` — changes priority of process (does not terminate)

> **Reasoning:** Only `kill()` (sending a termination signal) directly ends a process among the given options.

### Q16 (GATE PYQ style). A CPU has two Modes — Privileged and Non-Privileged. In order to change the mode from Non-Privileged (UM) to Privileged (KM):
(a) A Hardware Interrupt is needed.  (b) **A Software Interrupt is needed.** ✅  (c) A Privileged Instruction (which does not generate an interrupt) is needed.  (d) A Non-Privileged Instruction (which does not generate an interrupt) is needed.

> **Reasoning:** UM → KM transition is achieved via a controlled software interrupt (system call/trap) — not a hardware interrupt (external event), and definitely not by simply executing a privileged instruction directly from user mode (that's illegal and would trap/fault, not smoothly transition).

### Q17. System Calls are usually invoked by using:
(a) **A Software Interrupt** ✅  (b) Polling  (c) An Indirect jump  (d) A Privileged Instruction

> **Reasoning:** The standard mechanism is a software interrupt (trap instruction, e.g., `int 0x80`/`syscall`) — not polling (that's for I/O status checks), not a plain indirect jump, and not a privileged instruction (user mode can't execute those directly).

### Q18. A Processor needs Software Interrupt to:
(a) Test the Interrupt System of the Processor  (b) Implement Co-Routines  (c) **Obtain system services which need execution of privileged instructions** ✅  (d) Return from subroutine

> **Reasoning:** The core purpose of a software interrupt (trap) is precisely to safely request kernel-only (privileged) services from user mode.

### Q19 (GATE PYQ). A computer handles several interrupt sources — from CPU temperature sensor, mouse, keyboard, and hard disk. Which is handled at HIGHEST priority?
(a) Interrupt from Hard Disk  (b) Interrupt from Mouse  (c) Interrupt from Keyboard  (d) **Interrupt from CPU temperature sensor** ✅

> **Reasoning:** A CPU-overheat condition threatens the physical safety/integrity of the entire system (risk of hardware damage) — it must always be serviced before I/O-device interrupts (mouse/keyboard/disk), which only affect data/UX, not hardware survival.

### Q20 (GATE PYQ style — Multi-Select). Which standard C library functions will ALWAYS invoke a System Call when executed from a program in UNIX/Linux?
(a) `strlen` — pure user-mode string ops  (b) **`sleep`** ✅  (c) `malloc` — usually doesn't always trigger a syscall (uses heap already allocated via prior `brk`/`sbrk`)  (d) **`exit`** ✅

> **Reasoning:** `sleep()` must relinquish the CPU/schedule a wakeup — inherently kernel work. `exit()` must terminate the process and reclaim resources — inherently kernel work. `strlen` is pure computation; `malloc` often just manages already-obtained heap memory in user space without a fresh syscall every time.

### Q21 (GATE PYQ style — Multi-Select). Which option(s) guarantee a computer system will transition from User Mode to Kernel Mode?
(a) Function call — stays in user mode  (b) `malloc` call — usually stays in user mode  (c) **Page Fault** ✅  (d) **System call** ✅

> **Reasoning:** Both a page fault (hardware exception needing OS's page-management routines) and a system call (explicit request for OS service) forcibly trap into kernel mode. A plain function call or (usually) a `malloc` call stay entirely within user-mode execution.

### Q22 (GATE PYQ — classic `fork()` in a loop). Count total `printf`/`print` outputs:
```c
main()
{
    int k, n;
    for (k = 1; k <= n; ++k)
    {
        fork();
        print("GfG");
    }
}
```
> **Approach:** `fork()` is BEFORE `print()` inside the loop, so at iteration `k`, the number of processes existing (before this iteration's forks) determines how many times `print` executes at that iteration. Trace the process tree per iteration (each process forks once per loop pass, all currently-alive processes execute `print` after their own fork). For a loop running `n` times, the total print count is $2^{(n+1)} - 2$ (build the process tree per iteration to verify for small `n`, e.g., `n=2` → 6 prints).

**Variant — `print()` BEFORE `fork()` inside the loop:**
```c
main()
{
    int k, n;
    for (k = 1; k <= n; ++k)
    {
        print("GfG");
        fork();
    }
}
```
> **Key difference:** Since `print()` executes *before* the `fork()` in each iteration, the print count follows the cleaner doubling pattern per iteration — total prints = $2^n - 1$ (same as `n` forks in series scenario), since fork() at the end of the loop doesn't add extra prints in that same iteration.

> 🧠 **GATE Exam Tip:** Always ask *"does fork() come before or after the print/statement in question, and is it inside a loop or straight-line code?"* This single detail changes the entire counting formula. Draw the process tree — don't blindly memorize $2^n$.

---

## 🧠 Memory Tricks / Mnemonics Recap

- **Mode Bit:** `1` = User (restricted, like "1 person, public lobby"); `0` = Kernel ("0 restrictions, control room").
- **Trap Triggers:** System Call (voluntary, internal) / Exception (involuntary, internal) / Interrupt (involuntary, external) — **all force bit 1→0**.
- **Privileged Instruction Test:** *"Does it affect the WHOLE system, or just my local variables?"* Whole system → privileged.
- **System Call ≠ Privileged Instruction:** System call = the *request*; privileged instruction = the *actual hardware command* executed by OS as a result.
- **`fork()` counting:** count = $2^n$ processes for `n` forks in **straight-line series**; but **inside a loop**, always trace the process tree — children don't re-run earlier iterations.
- **`pid` values:** `pid < 0` → fork failed | `pid == 0` → child | `pid > 0` → parent.

---

## ✅ Quick Revision Checklist

- [ ] Why dual-mode (User/Kernel) architecture exists — protection rationale
- [ ] Mode Bit values (1 = User, 0 = Kernel) and hardware enforcement
- [ ] Definition of Privileged Instructions + 7 examples (interrupts, timer, page-table, I/O, halt, etc.)
- [ ] Why privileged instructions are necessary (prevents memory corruption, unauthorized HW access, system crash)
- [ ] Role of privileged instructions in Memory Management & Process Management
- [ ] Trap Mechanism — 4-step flow (detect → trap → transfer control → OS handles)
- [ ] Three triggers for Kernel entry: System Call vs Exception/Trap vs Interrupt (origin + intent table)
- [ ] Context-switch overhead of mode switching (save/restore state cost)
- [ ] Definition of System Call + how it differs from a library call
- [ ] Three APIs: Win32, POSIX, Java
- [ ] Dispatch Table concept — how syscall numbers map to kernel routines
- [ ] `printf()` → `write()` example (library call vs system call distinction)
- [ ] 3 methods of parameter passing to OS: registers / memory block+register / stack
- [ ] 6 types of system calls: Process control, File mgmt, Device mgmt, Info maintenance, Communication, Protection
- [ ] Key syscalls: `fork`, `waitpid`, `execve`, `exit`, `open`, `close`, `read`, `write`, `lseek`, `stat`, `mkdir`, `rmdir`, `link`, `unlink`, `mount`, `umount`, `chdir`, `chmod`, `kill`, `time`
- [ ] UNIX vs Windows syscall name mapping (6 categories)
- [ ] System Programs categories (file mgmt, status info, file modification, language support, program loading, communications)
- [ ] `fork()` behavior — child starts execution right after the fork call, not from the top
- [ ] Formula: $n$ forks in series → $2^n$ total processes, $2^n-1$ children
- [ ] `fork()` inside a loop ≠ straightforward $2^n$ — must trace process tree
- [ ] Standard UNIX idiom: `fork()` + `exec()` + `wait()` (parent waits, child execs new program)
- [ ] Full `fork()`-error-handling C code pattern (`pid<0`, `pid==0`, `else`)
- [ ] All GATE MCQs above — re-attempt without looking at answers

---
*End of Lecture 04 Notes — System Calls*
