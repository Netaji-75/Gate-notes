# Computer Networks — Lecture 3: Data Link Layer, MAC Address & Error Detection (VRC, LRC, Checksum, CRC)

> GeeksforGeeks GATE CS & IT | Computer Networks | Lecture 3
> Note: this lecture is **Computer Networks**, not OS. File it under your CN notes.

---

## Topics Covered

1. [Where DLL and MAC fit (TL vs DLL addressing)](#1-addressing-in-the-network-big-picture)
2. [Data Link Layer: functions](#2-data-link-layer-dll--layer-2)
3. [MAC address: format, hex notation, types](#3-mac-address)
4. [Hex to binary reference table](#4-hex--binary-reference)
5. [Error handling: what is an error, single-bit vs burst](#5-error-handling-in-the-data-link-layer)
6. [Dataword, codeword, redundancy](#6-dataword--codeword)
7. [VRC (parity)](#7-vrc--vertical-redundancy-check-parity)
8. [LRC (block parity)](#8-lrc--longitudinal-redundancy-check)
9. [Checksum](#9-checksum)
10. [Binary addition/subtraction tables (XOR basics)](#10-binary-addition--subtraction-basics)
11. [CRC (Cyclic Redundancy Check)](#11-crc--cyclic-redundancy-check)
12. [Choosing G(x) to detect odd-bit errors](#12-generator-polynomial-gx-properties)
13. [GATE PYQs](#13-gate-pyqs)
14. [Quick revision checklist](#14-quick-revision-checklist)

---

## 1. Addressing in the Network (Big Picture)

The lecture's diagram: hosts sit inside different networks, connected through **Router / Brouter / Gateway** devices.

| Layer | Address used | Scope |
|---|---|---|
| **Transport Layer (TL)** | **Port number** | Identifies the process, end to end (source host to destination host) |
| **Data Link Layer (DLL)** | **MAC address** | Identifies the node, hop to hop within one network |

- At every hop the **MAC address changes**; the **port number stays the same** end to end.
- **Router, Brouter, Gateway** are the devices that interconnect networks.

**Formula on the title slide (handwritten):**

$$T_d \ge 2 \times P_d$$

- $T_d$ = transmission delay (frame size / bandwidth)
- $P_d$ = propagation delay
- This is the CSMA/CD condition: transmission time must be at least twice the propagation delay so that a collision can be detected before the frame finishes sending. (Covered in later lectures.)

---

## 2. Data Link Layer (DLL) — Layer 2

**DLL = Logical Link Control (LLC) + Medium Access Control (MAC)** sublayers.

**Functions of DLL:**
- **MAC addressing**
- **Error control**
- **Access control** (this is the *Medium Access Control* part)
- **Flow control**
- **Framing**
- **Node-to-node delivery within a network**

---

## 3. MAC Address

### Format
- Length: **48 bits = 6 bytes (octets)**
- Written in **hexadecimal**, each byte = 2 hex digits.
- Total addresses possible: $2^{48}$

| Notation | Layout | Compatible with |
|---|---|---|
| Byte-wise | `Byte1:Byte2:Byte3:Byte4:Byte5:Byte6` (8 bits each) | **IPv4 style** (8-bit groups) |
| 16-bit groups | `D2F1:36FE:4BDF` (16 : 16 : 16 bits) | **IPv6 style** (16-bit groups) |

### Worked example (hexadecimal notation)

`D2:F1:36:FE:4B:DF`

| Byte | Hex | Binary |
|---|---|---|
| 1 | D2 | 1101 0010 |
| 2 | F1 | 1111 0001 |
| 3 | 36 | 0011 0110 |
| 4 | FE | 1111 1110 |
| 5 | 4B | 0100 1011 |
| 6 | DF | 1101 1111 |

- **Leftmost byte (D2) = most significant**; **rightmost byte (DF) = least significant**.
- Each **hex digit = 4 bits** (nibble).

### Types of MAC addresses

The type is decided by the **LSB of the first (most significant) byte**, i.e. whether the **second hex digit of the first byte is even or odd**.

| Type | First byte's LSB | Hex digit (of first byte) | Binary bit | Example |
|---|---|---|---|---|
| **Unicast** | 0 | **Even** | `0` | `3A:4C:FF:2E:10:31` (A = 1010, even) |
| **Multicast** | 1 | **Odd** | `1` | `57:21:3F:4C:11:83` (7 = 0111, odd) |
| **Broadcast** | 1 | all F | all 1s | `FF:FF:FF:FF:FF:FF` |

- Unicast = one specific receiver
- Multicast = a group of receivers
- Broadcast = every node in the network

---

## 4. Hex → Binary Reference

Hex digits run from **0 to 15** (`0–9, A, B, C, D, E, F`). Weights per bit: **8 4 2 1**.

| Dec | Hex | Binary | | Dec | Hex | Binary |
|---|---|---|---|---|---|---|
| 0 | 0 | 0000 | | 8 | 8 | 1000 |
| 1 | 1 | 0001 | | 9 | 9 | 1001 |
| 2 | 2 | 0010 | | 10 | A | 1010 |
| 3 | 3 | 0011 | | 11 | B | 1011 |
| 4 | 4 | 0100 | | 12 | C | 1100 |
| 5 | 5 | 0101 | | 13 | D | 1101 |
| 6 | 6 | 0110 | | 14 | E | 1110 |
| 7 | 7 | 0111 | | 15 | F | 1111 |

---

## 5. Error Handling in the Data Link Layer

**Definition:** **Sent ≠ Received ⇒ Error occurred.**

### Ways an error can show up

| Sent | Received | What happened |
|---|---|---|
| `1100` | `110` | bit lost |
| `1100` | `11100` | extra bit added |
| `1100` | `1110` | bit flipped |

### Types of errors (bit-flip errors)

| Type | Meaning | Examples (sent `1100`) |
|---|---|---|
| **Single-bit error** | Exactly **1** bit flipped | `1101`, `1110`, `1000`, `0100` |
| **Burst error** | **2 or more** bits flipped | `1111` (2 bits), `1011` (3 bits), `0011` (4 bits) |

### Error-handling techniques

| Category | Capability | Techniques |
|---|---|---|
| **Error Detection Code** | Identifies **presence** of error **only** | **VRC, LRC, CRC, Checksum** |
| **Error Detection & Correction Code** | Identifies **presence and position** of error | **Hamming Code** |

---

## 6. Dataword & Codeword

$$\text{Dataword } (D) + \text{Check/Parity/Redundant bit(s)} = \text{Codeword } (C)$$

- The sender computes redundant bits (**mandatory**), appends them to $D$, and transmits the **codeword**.
- Receiver gets $C_R$ and checks it.
- Notation: $C_S$ = codeword sent, $C_R$ = codeword received.
- **Check bits, parity bits and redundant bits all mean the same thing.**

---

## 7. VRC — Vertical Redundancy Check (Parity)

- Also called the **Parity Mechanism**.
- Idea: count the **number of 1s in the codeword** and make it **even** (even parity) or **odd** (odd parity).
- **Sender:** add **1 parity bit** to the dataword.

**Example:** $D = 1011$ (three 1s)

| Scheme | Parity bit added | Codeword $C$ |
|---|---|---|
| **Even parity** | 1 (to make count of 1s = 4) | `10111` |
| **Odd parity** | 0 (count stays 3) | `10110` |

### Detection behaviour

**Even parity, $C_S = 10111$:**

| $C_R$ | Bits modified | Result |
|---|---|---|
| `11111` | 1 | Error detected |
| `10110` | 1 | Error detected |
| `10000` | 3 | Error detected |
| `11011` | 2 | **No error detected** (missed) |
| `11000` | 4 | **No error detected** (missed) |

**Odd parity, $C_S = 10110$:**

| $C_R$ | Bits modified | Result |
|---|---|---|
| `10111` | 1 | Error detected |
| `11000` | 3 | Error detected |
| `01001` | 5 | Error detected |
| `10101` | 2 | **No error detected** (missed) |
| `11001` | 4 | **No error detected** (missed) |

**Key rules**
- VRC **detects all odd-numbered bit errors** (1, 3, 5, ...).
- VRC **fails for even-numbered bit errors** (2, 4, ...).
- Receiver check: even parity expects an even count of 1s; odd parity expects an odd count.
- VRC cannot locate the error and cannot correct it.

---

## 8. LRC — Longitudinal Redundancy Check

- Also called **Block Parity Mechanism** (a 2-D version of parity: parity per **column**).

| Sender | Receiver |
|---|---|
| 1. Divide $D$ into **fixed-size blocks** $B_1, B_2, B_3, \dots$ | 1. Divide $C_R$ into blocks |
| 2. Arrange blocks **line by line** (row-wise) | 2. Arrange them the same way |
| 3. Calculate **parity column-wise** → $R$ | 3. Calculate **parity column-wise** |
| 4. $D + R = C_S$, transmit | Result **all 0s → No error**; **non-zero → Error detected** |

### Worked example (block size = 8 bits, even parity)

$D$ = `1101 0011 1001 1000 1101` (20 bits). The leftmost block is short, so it is padded with leading zeros to 8 bits.

| Block | Bits |
|---|---|
| $B_1$ | `1000 1101` |
| $B_2$ | `0011 1001` |
| $B_3$ | `0000 1101` |
| **$R$ (column even parity)** | **`1011 1001`** |

Column check: col1 (1,0,0) → 1; col2 (0,0,0) → 0; col3 (0,1,0) → 1; col4 (0,1,0) → 1; col5 (1,1,1) → 1; col6 (1,0,1) → 0; col7 (0,0,0) → 0; col8 (1,1,1) → 1.

$$C_S = D + R = \texttt{1101 0011 1001 1000 1101 1011 1001}$$

**Receiver side (corrupted codeword example):**

| Block | Bits |
|---|---|
| $B_1$ | `1010 1001` |
| $B_2$ | `1010 1101` |
| $B_3$ | `0011 0001` |
| $B_4$ | `0000 1001` |
| **Column parity** | **`0011 1100`** → **non-zero → error detected** |

*(Blocks read from the handwriting; the method is what matters.)*

### Limitation shown in the lecture

- In the second example, red bits are flipped in pairs within the **same columns** (8 flipped bits, an even number per column). The column parity comes out as all `00000000`, so **the error is NOT detected**.
- The lecture's note says some even-numbered errors can be caught by LRC (unlike VRC), but errors that form a "rectangle" pattern (an even number of flips in the same column) slip through.

---

## 9. Checksum

| Sender | Receiver |
|---|---|
| 1. Divide $D$ into **fixed-size blocks** $B_1, B_2, B_3, \dots$ | 1. Divide $C_R$ into blocks |
| 2. Add all blocks: $S = (((B_1+B_2)+B_3)+B_4)+\dots$ (**end-around carry**: carry out is added back to the LSB) | 2. Add all blocks (including the checksum block) → $S$ |
| 3. $R = \overline{S}$ (**1's complement** of $S$) | 3. Take $\overline{S}$ |
| 4. $D + R = C_S$, transmit | $\overline{S} = 0$ → **No error**; non-zero → **Error** |

### Worked example (block size = 4 bits)

$D$ = `11 0111 0011 1001 1001`. The leftmost partial block `11` is padded to `0011`.

**Sender**

| Step | Operation | Result |
|---|---|---|
| $B_1 + B_2$ | `1001 + 1001` = `1 0010` → wrap carry: `0010 + 1` | `0011` |
| $+ B_3$ (`0011`) | `0011 + 0011` | `0110` |
| $+ B_4$ (`0111`) | `0110 + 0111` | `1101` |
| $+ B_5$ (`0011`) | `1101 + 0011` = `1 0000` → wrap: `0000 + 1` | `0001` |

- $S = 0001$
- $R = \overline{S} = \mathbf{1110}$

$$C_S = \texttt{11 0111 0011 1001 1001 1110}$$

**Receiver** (blocks from right: $B_1$ = 1110 (checksum), $B_2$ = 1001, $B_3$ = 1001, $B_4$ = 0011, $B_5$ = 0111, $B_6$ = 0011)

| Step | Operation | Result |
|---|---|---|
| $B_1 + B_2$ | `1110 + 1001` = `1 0111` → wrap | `1000` |
| $+ B_3$ | `1000 + 1001` = `1 0001` → wrap | `0010` |
| $+ B_4$ | `0010 + 0011` | `0101` |
| $+ B_5$ | `0101 + 0111` | `1100` |
| $+ B_6$ | `1100 + 0011` | `1111` |

- $S = 1111$ → $\overline{S} = 0000$ → **Zero → No error.**

---

## 10. Binary Addition & Subtraction Basics

| A | B | Sum | Carry | A ⊕ B | Difference | Borrow |
|---|---|---|---|---|---|---|
| 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 0 | 1 | 1 | 0 | 1 | 1 | 1 |
| 1 | 0 | 1 | 0 | 1 | 1 | 0 |
| 1 | 1 | 0 | 1 | 0 | 0 | 0 |

**Key takeaway (starred in lecture):** **Sum = Difference = A ⊕ B (XOR)**. Ignoring carry/borrow, binary addition and subtraction are the same operation. This is why **CRC division uses XOR**.

---

## 11. CRC — Cyclic Redundancy Check

(Starred as very important in the lecture.)

Division terminology: **Divisor | Dividend → Quotient, Remainder**.

### Sender algorithm

1. Append **$(n-1)$ zeros** to $D$: $D + (n-1)\text{ zeros} = $ **Dividend**, where **$n$ = number of bits in the divisor**.
2. **Divide**: repeatedly perform **XOR**.
3. Check the **MSB**:
   - **MSB = 1** → XOR with the divisor.
   - **MSB = 0** → XOR with zeros, i.e. just bring down the next bit from the dividend.
4. Repeat until **all dividend bits are used**.
5. The **$(n-1)$-bit remainder = $R$**.
6. **$D + R = C_S$**, transmit.

**Receiver:** divide $C_R$ by the same divisor. **Remainder = 0 → No error**; non-zero → error.

### Divisor as a Generator Polynomial $G(x)$

The CRC divisor is given as a polynomial: write a bit for each power of $x$ (present = 1, absent = 0).

| $G(x)$ | Expansion | Divisor bits | Check bits ($n-1$) |
|---|---|---|---|
| $x^3 + x + 1$ | $1x^3 + 0x^2 + 1x^1 + 1x^0$ | `1011` | 3 |
| $x^5 + x^3 + 1$ | $x^5 \dots x^0$ = 1 0 1 0 0 1 | `101001` | 5 |

- **Degree of $G(x)$ = number of check bits** = (divisor bits − 1).
- Quick illustration from the lecture: `1000000000 0000...1` with 1s at positions 31, 14 and 0 represents $x^{31} + x^{14} + 1$.

### Worked example: $D = 110101001$, divisor $= 1001$ ($x^3+1$)

**Sender:** dividend = $D$ + 3 zeros = `110101001000`

| Step | Working bits | XOR with | Result (remainder so far) |
|---|---|---|---|
| 1 | `1101` | `1001` | `0100` → keep `100` |
| 2 | bring down 0 → `1000` | `1001` | `0001` → `001` |
| 3 | bring down 1 → `0011` | `0000` (MSB 0) | `011` |
| 4 | bring down 0 → `0110` | `0000` | `110` |
| 5 | bring down 0 → `1100` | `1001` | `0101` → `101` |
| 6 | bring down 1 → `1011` | `1001` | `0010` → `010` |
| 7 | bring down 0 → `0100` | `0000` | `100` |
| 8 | bring down 0 → `1000` | `1001` | `0001` → `001` |
| 9 | bring down 0 → `0010` | `0000` | **`010`** |

- **CRC remainder $R = 010$**
- $C_S = D + R = \texttt{110101001010}$

**Receiver:** divide `110101001010` by `1001` → **remainder = `000`** → **No error**.

*(In the lecture's shorthand, "shl" means shift left: after an XOR whose leading bits became zero, shift to bring down bits.)*

---

## 12. Generator Polynomial $G(x)$ Properties

**To detect an odd number of bit errors:**

| Condition on $G(x)$ | Why |
|---|---|
| **$x^0$ coefficient must be 1** | $G(x)$ must not be divisible by $x$; otherwise the last bit of the frame is not protected |
| **Must be divisible by $(x+1)$** | $(x+1)$ = `11` (binary) = 3 (decimal); divisibility by $(x+1)$ guarantees **all odd-bit errors are detected** |
| **Must NOT be divisible by $x$** | $x$ = `10` (binary) = 2 (decimal); same as the $x^0$ rule |

- Divisible by $(x+1)$ is equivalent to **$G(x)$ having an even number of terms**.
- The lecture circles **"odd number of terms"** and links it to **even-numbered bit errors**: an odd term count is NOT the criterion for catching odd errors.

**Sample polynomials listed on the board:**

| Decimal | Binary | Polynomial |
|---|---|---|
| 5 | `101` | $x^2 + 1$ |
| 9 | `1001` | $x^3 + 1$ |
| 21 | `10101` | $x^4 + x^2 + 1$ |

(Heads-up: of these, $x^2+1$ and $x^3+1$ have an even number of terms and are divisible by $x+1$; $x^4+x^2+1$ has 3 terms, so it is **not** divisible by $x+1$. Treat the list as examples of writing polynomials as bit patterns, and check each against the conditions.)

---

## 13. GATE PYQs

### Q1. [GATE 2021] CRC with $G(x) = x^3 + x + 1$

Message $m_4m_3m_2m_1m_0 = 11000$ is transmitted. Check bits $c_2c_1c_0$ are appended using CRC. The value of $c_2c_1c_0$ is:

- (A) 101
- (B) 110
- **(C) 100** ✅
- (D) 111

**Solution:** $G = 1011$, dividend = `11000` + `000` = `11000000`.
`1100`⊕`1011`=`0111` → `1110`⊕`1011`=`0101` → `1010`⊕`1011`=`0001` → `0010` → `0100`. **Remainder = `100`**.

**Why others are wrong:** the long division gives `100`; `101`, `110`, `111` are not the remainder (codewords `11000101/110/111` leave a non-zero remainder when divided by `1011`).

---

### Q2. [GATE 2005] CRC of message M = 1010001101

Using divisor polynomial $x^5 + x^4 + x^2 + 1$ (= `110101`), the CRC is:

- **(a) 01110** ✅
- (b) 01011
- (c) 10101
- (d) 10110

**Solution:** dividend = `1010001101` + `00000`. Divide by `110101` with XOR; final 5-bit remainder = **`01110`**.

**Why others are wrong:** `01011` and `10110` appear only as intermediate remainders during the division; `10101` never arises. Only `01110` is the final remainder.

---

### Q3. [GATE 2007] Message 11001001 with CRC polynomial $x^3 + 1$

The message that should be transmitted is:

- (a) 11001001000
- **(b) 11001001011** ✅
- (c) 11001010
- (d) 110010010011

**Solution:** $G = 1001$, dividend = `11001001000`. Remainder $R = \mathbf{011}$. Transmitted = `11001001` + `011` = **`11001001011`**.

**Why others are wrong:**
- (a) appends `000`, which is just the padded dividend with no remainder added.
- (c) is only 8 bits, so no CRC bits were appended.
- (d) appends 4 bits (`0011`); the CRC for a 4-bit divisor must be 3 bits.

---

### Q4. [GATE 2026, as labelled on the slide] Data bits 110001011, generator 1001

Which option shows the remainder bit pattern appended to the data bits before transmission?

- (A) 011
- (B) 101
- (C) 000
- **(D) 100** ✅

**Solution:** dividend = `110001011` + `000` = `110001011000`, divisor `1001`. Final remainder = **`100`**.

**Why others are wrong:** `000` would mean the data is already divisible (it is not); `011` and `101` appear only as intermediate remainders, not the final one.

---

### Q5. [GATE 2009] Condition on $G(x)$ to detect an odd number of bit errors

- (a) $G(x)$ contains more than two terms
- (b) $G(x)$ does not divide $1 + x^k$ for any $k$ not exceeding the frame length
- **(c) $1 + x$ is a factor of $G(x)$** ✅
- (d) $G(x)$ has an odd number of terms

**Why others are wrong:**
- (a) The number of terms being more than two is neither necessary nor sufficient for odd-error detection.
- (b) This condition is for detecting **double-bit (two-error) patterns**, not odd-bit errors. Lecture example: $x^3+1$ = `1001`, frame length > 3 bits.
- (d) An odd number of terms means $G(1) = 1$, so $(x+1)$ does **not** divide it. The lecture ties this to **even** numbers of bit errors, not odd.

---

### Q6. [GATE 2007] Slotted LAN: probability exactly one station transmits

$n$ stations, each transmits with probability $p$ in a slot. Probability that **only one** station transmits in a given slot:

- **(a) $np(1-p)^{n-1}$** ✅
- (b) $(1-p)^{n-1}$
- (c) $p(1-p)^{n-1}$
- (d) $1-(1-p)^{n-1}$

**Derivation (from the lecture):**
- 1st station transmits, others don't: $p(1-p)^{n-1}$
- 2nd station only: $p(1-p)^{n-1}$ ... and so on up to the $n$-th station.
- These are mutually exclusive, so add the $n$ equal terms:

$$P(\text{success}) = n\,p\,(1-p)^{n-1}$$

**Why others are wrong:**
- (b) is the probability that the *other* $n-1$ stations stay silent (nobody is selected).
- (c) is the probability that one *specific* station is the only transmitter, missing the factor $n$.
- (d) is the probability that at least one of the other $n-1$ stations transmits.

---

### Q7. [GATE 2015] Slotted LAN with four nodes, collision-free first slot

Probabilities of generating a frame in a slot: S1 = 0.1, S2 = 0.2, S3 = 0.3, S4 = 0.4. A collision occurs if more than one node transmits in the same slot. Find the probability of sending a frame in the first slot **without collision**.

**Solution:** success means **exactly one** station transmits. Sum over each station $i$ being the only transmitter:

$$P = \sum_{i} p_i \prod_{j \ne i}(1-p_j)$$

| Only transmitter | Computation | Value |
|---|---|---|
| S1 | $0.1 \times 0.8 \times 0.7 \times 0.6$ | 0.0336 |
| S2 | $0.9 \times 0.2 \times 0.7 \times 0.6$ | 0.0756 |
| S3 | $0.9 \times 0.8 \times 0.3 \times 0.6$ | 0.1296 |
| S4 | $0.9 \times 0.8 \times 0.7 \times 0.4$ | 0.2016 |
| **Total** | | **0.4404** ✅ |

---

## 14. Memory Aids

The lecture itself does not give explicit mnemonics. These are **added by me** (not from the lecture):

- **Port = Process, MAC = Machine (hop).**
- **Unicast ends even, multicast ends odd** (LSB of the first byte); **broadcast = all F**.
- **VRC = one parity bit, odd errors only. LRC = parity per column. Checksum = add and complement. CRC = XOR division.**
- **CRC: append (n−1) zeros, divide with XOR, remainder replaces zeros.**
- **Sum = Difference = XOR**, which is why CRC division is carry-free.
- **Odd detection needs $(x+1)$ as a factor.**
- **Slotted success formula: $n p (1-p)^{n-1}$**, the "$n$" is because any one of the $n$ stations can be the winner.

---

## 15. Quick Revision Checklist

- [ ] TL uses port number (process); DLL uses MAC address (node, hop to hop)
- [ ] Router / Brouter / Gateway interconnect networks
- [ ] $T_d \ge 2 P_d$ (CSMA/CD condition)
- [ ] DLL = LLC + MAC sublayers
- [ ] DLL functions: MAC addressing, error control, access control, flow control, framing, node-to-node delivery
- [ ] MAC: 48 bits / 6 bytes, hex notation, IPv4-style (bytes) vs IPv6-style (16-bit groups)
- [ ] MSB = leftmost byte, LSB = rightmost byte
- [ ] Unicast (even), multicast (odd), broadcast (FF:FF:FF:FF:FF:FF)
- [ ] Hex 0–15 to 4-bit binary table
- [ ] Error: sent ≠ received; single-bit vs burst errors
- [ ] Detection-only codes: VRC, LRC, CRC, Checksum; detection + correction: Hamming
- [ ] Dataword + check bits = Codeword ($C_S$, $C_R$)
- [ ] VRC: even/odd parity, detects odd-bit errors, misses even-bit errors
- [ ] LRC: blocks, column-wise parity, receiver all-zero check, rectangle-pattern weakness
- [ ] Checksum: add blocks with end-around carry, 1's complement, receiver result 0
- [ ] Binary addition/subtraction tables; Sum = Difference = XOR
- [ ] CRC steps: append $(n-1)$ zeros, XOR division, MSB 1 vs 0 rule, remainder $R$, $C_S = D + R$
- [ ] Polynomial to bit pattern: $x^3+x+1 \to 1011$, $x^5+x^3+1 \to 101001$, $x^3+1 \to 1001$
- [ ] $G(x)$ conditions: $x^0$ term = 1, divisible by $(x+1)$, not divisible by $x$
- [ ] Odd number of terms in $G(x)$ links to even-bit errors, not odd
- [ ] PYQs: GATE 2021 (100), GATE 2005 (01110), GATE 2007 (11001001011), GATE 2026 slide (100), GATE 2009 (option c), GATE 2007 slotted LAN ($np(1-p)^{n-1}$), GATE 2015 (0.4404)
