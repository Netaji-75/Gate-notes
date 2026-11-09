# Computer Networks — Lecture 4: ARP/RARP Recap, Hamming Code, Delay Problems, Access Control, CSMA & Binary Exponential Backoff

> GeeksforGeeks GATE CS & IT | Computer Networks | Lecture 4
> Continues from Lecture 3 (DLL, MAC address, VRC/LRC/Checksum/CRC). Filed under CN, not OS.

---

## Topics Covered

1. [IP address, ARP and RARP (intro sketch)](#1-ip-address-arp--rarp-quick-sketch)
2. [Hamming code: why it is different, how many check bits](#2-hamming-code--concept)
3. [Hamming code: sender worked example](#3-hamming-code--sender-example-d--11011001)
4. [Hamming code: receiver, locating and fixing the error](#4-hamming-code--receiver-example)
5. [Hamming code: bigger example (21-bit codeword)](#5-hamming-code--bigger-example-16-data-bits)
6. [Delay formulas and GATE problems (file transfer, packet switch, satellite)](#6-delay-formulas--gate-problems)
7. [Access control (MAC sublayer protocols)](#7-access-control--mac-sublayer-protocols)
8. [CSMA and persistence variants](#8-csma-carrier-sense-multiple-access)
9. [Binary Exponential Backoff (BEB)](#9-binary-exponential-backoff-beb)
10. [GATE PYQs (all in one place)](#10-gate-pyqs)
11. [Memory aids](#11-memory-aids)
12. [Quick revision checklist](#12-quick-revision-checklist)

---

## 1. IP Address, ARP & RARP (Quick Sketch)

The lecture opens with a diagram connecting networks through routers.

$$\text{IP address} = \text{Network ID (NID)} + \text{Host ID (HID)}$$

- **Network ID** = network identification (which network); **Host ID** = which host inside it.
- Example on the board: `192.168.3.?` with host number 7.

| Protocol | Full form | Table held | Maps |
|---|---|---|---|
| **ARP** | Address Resolution Protocol | NID, HID, MAC | **IP address → MAC address** |
| **RARP** | Reverse (Inverse) Address Resolution Protocol | MAC, IP | **MAC address → IP address** |

*(The board only draws the tables and names the protocols; the direction of mapping is the standard definition.)*

---

## 2. Hamming Code — Concept

### What is different?

| Code | Redundant bits $R$ | Where $R$ goes |
|---|---|---|
| VRC | $D$ + single bit | **Contiguous** (appended at the end) |
| LRC | $D$ + block bits | Contiguous |
| CRC | $D$ + $(n-1)$ bits | Contiguous |
| Checksum | $D$ + block bits | Contiguous |
| **Hamming** | $D$ + $R$ bits | **Non-contiguous**, at **power-of-2 positions** |

- The first four are detection-only. **Hamming detects and corrects** (it finds the exact position of the error).

### Positions of redundant bits

$R_i$ sits at position $2^i$ ($i = 0, 1, 2, \dots$), counting positions from **1 at the right (LSB)**.

| Redundant bit | Position |
|---|---|
| $R_0$ | $2^0$ = 1st |
| $R_1$ | $2^1$ = 2nd |
| $R_2$ | $2^2$ = 4th |
| $R_3$ | $2^3$ = 8th |
| $R_4$ | $2^4$ = 16th |
| ... | 32, 64, ... |

All other positions hold **data bits**.

### Formula: minimum number of redundant bits

$$2^R \ge D + R + 1$$

- $D$ = number of data bits
- $R$ = number of redundant (check) bits
- The $+1$ is for the "no error" case; the rest covers an error in any of the $D+R$ positions.

**Example ($D = 13$):**

| $R$ | Check | Holds? |
|---|---|---|
| 4 | $2^4 = 16 \ge 13+4+1 = 18$ | **False** |
| 5 | $2^5 = 32 \ge 13+5+1 = 19$ | **True** → $R = 5$ |

Codeword length = $13 + 5 = 18$ bits. Layout for $D = 1010111011001$ (positions 18 → 1):

| Pos | 18 | 17 | 16 | 15 | 14 | 13 | 12 | 11 | 10 | 9 | 8 | 7 | 6 | 5 | 4 | 3 | 2 | 1 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Bit | 1 | 0 | $R_4$ | 1 | 0 | 1 | 1 | 1 | 0 | 1 | $R_3$ | 1 | 0 | 0 | $R_2$ | 1 | $R_1$ | $R_0$ |

### Which positions does each $R_i$ check?

$R_i$ checks every position whose **binary representation has bit $i$ equal to 1** (even parity, XOR of those positions).

| Check bit | Positions covered (up to 12) |
|---|---|
| $R_0$ | 1, 3, 5, 7, 9, 11 |
| $R_1$ | 2, 3, 6, 7, 10, 11 |
| $R_2$ | 4, 5, 6, 7, 12 |
| $R_3$ | 8, 9, 10, 11, 12 |

(Board table lists positions 1–12 in 4-bit binary, with the covering bit marked.)

---

## 3. Hamming Code — Sender Example ($D = 11011001$)

- $D = 11011001$ (8 bits). $2^4 = 16 \ge 8+4+1 = 13$ → $R = 4$.
- Codeword $C = D + R$ = **8 + 4 = 12 bits**. Even parity.

**Layout (positions 12 → 1), data placed in non-power-of-2 positions:**

| Pos | 12 | 11 | 10 | 9 | 8 | 7 | 6 | 5 | 4 | 3 | 2 | 1 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Before | 1 | 1 | 0 | 1 | $R_3$ | 1 | 0 | 0 | $R_2$ | 1 | $R_1$ | $R_0$ |

**Computing the check bits (XOR, even parity):**

| Bit | Positions | Values | Result |
|---|---|---|---|
| $R_0$ | 3 ⊕ 5 ⊕ 7 ⊕ 9 ⊕ 11 | 1 ⊕ 0 ⊕ 1 ⊕ 1 ⊕ 1 | **0** |
| $R_1$ | 3 ⊕ 6 ⊕ 7 ⊕ 10 ⊕ 11 | 1 ⊕ 0 ⊕ 1 ⊕ 0 ⊕ 1 | **1** |
| $R_2$ | 5 ⊕ 6 ⊕ 7 ⊕ 12 | 0 ⊕ 0 ⊕ 1 ⊕ 1 | **0** |
| $R_3$ | 9 ⊕ 10 ⊕ 11 ⊕ 12 | 1 ⊕ 0 ⊕ 1 ⊕ 1 | **1** |

$$C_S = \texttt{1101 1100 0110}$$
(positions: 12:1, 11:1, 10:0, 9:1, 8:$R_3$=1, 7:1, 6:0, 5:0, 4:$R_2$=0, 3:1, 2:$R_1$=1, 1:$R_0$=0)

---

## 4. Hamming Code — Receiver Example

Sent: $C_S = 110111000110$. Received (bit 9 flipped, 1 → 0): $C_R = 110011000110$.

| Pos | 12 | 11 | 10 | 9 | 8 | 7 | 6 | 5 | 4 | 3 | 2 | 1 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| $C_R$ | 1 | 1 | 0 | **0** | 1 | 1 | 0 | 0 | 0 | 1 | 1 | 0 |

**Recompute each check, now including the received check bit itself:**

| Bit | Positions | Values | Result |
|---|---|---|---|
| $R_0$ | 1 ⊕ 3 ⊕ 5 ⊕ 7 ⊕ 9 ⊕ 11 | 0 ⊕ 1 ⊕ 0 ⊕ 1 ⊕ 0 ⊕ 1 | **1** |
| $R_1$ | 2 ⊕ 3 ⊕ 6 ⊕ 7 ⊕ 10 ⊕ 11 | 1 ⊕ 1 ⊕ 0 ⊕ 1 ⊕ 0 ⊕ 1 | **0** |
| $R_2$ | 4 ⊕ 5 ⊕ 6 ⊕ 7 ⊕ 12 | 0 ⊕ 0 ⊕ 0 ⊕ 1 ⊕ 1 | **0** |
| $R_3$ | 8 ⊕ 9 ⊕ 10 ⊕ 11 ⊕ 12 | 1 ⊕ 0 ⊕ 0 ⊕ 1 ⊕ 1 | **1** |

$$\text{Error position} = (\dots R_3 R_2 R_1 R_0)_2 = (1001)_2 = 9^{\text{th}} \text{ position}$$

- **Error-position word = 0000 → no error.** Otherwise it is the binary position of the flipped bit.
- **Correction:** flip the bit at that position (here, bit 9 back to 1).
- Hamming (basic form) corrects **single-bit** errors.

---

## 5. Hamming Code — Bigger Example (16 data bits)

Problem on the board: a network uses Hamming code; the sender transmits a message $M$ (the lecture computes the check bits for a 16-bit message in a 21-bit codeword; the board's problem statement is unfinished).

- $2^5 = 32 \ge 16+5+1 = 22$ → $R = 5$ ($2^4 = 16 < 22$ fails). Codeword = **21 bits**.
- Data bits (positions 21 → 1, skipping 16, 8, 4, 2, 1): positions 21–17 = `11111`; 15–9 = `0100010`; 7–5 = `110`; 3 = `1`.

| Check bit | Positions | Values (even parity) | Result |
|---|---|---|---|
| $R_0$ | 3, 5, 7, 9, 11, 13, 15, 17, 19, 21 | 1,0,1,0,0,0,0,1,1,1 | **1** |
| $R_1$ | 3, 6, 7, 10, 11, 14, 15, 18, 19 | 1,1,1,1,0,1,0,1,1 | **1** |
| $R_2$ | 5, 6, 7, 12, 13, 14, 15, 20, 21 | 0,1,1,0,0,1,0,1,1 | **1** |
| $R_3$ | 9–15 | 0,1,0,0,0,1,0 | **0** |
| $R_4$ | 17–21 | 1,1,1,1,1 | **1** |

$$C = \texttt{11111}\;\underline{1}\;\texttt{0100010}\;\underline{0}\;\texttt{110}\;\underline{1}\;\texttt{1}\;\underline{1}\;\underline{1}$$
$$= \texttt{111111010001001101111}$$
(underlined = $R_4, R_3, R_2, R_1, R_0$.)

---

## 6. Delay Formulas & GATE Problems

Formulas used in the lecture's problems (the lecture applies them in worked solutions):

$$T_d = \frac{L}{B} \qquad P_d = \frac{d}{v} \qquad T_{total} = T_d + P_d$$

- $T_d$ = transmission delay, $L$ = packet/file size in **bits**, $B$ = bandwidth (bits/s)
- $P_d$ = propagation delay, $d$ = link length, $v$ = propagation speed
- $\text{RTT} = 2 P_d$ (used again in CSMA)
- In a path of several links, **throughput is limited by the slowest link (bottleneck)** when packets are pipelined.

### 6.1 File transfer over three links [GATE 2026 as labelled on slide]

File = 4 million bytes over three consecutive links of **2 Mbps, 500 kbps, 1 Mbps**. Processing and propagation delays are negligible; no other traffic or overhead. Total time to transfer (s)?

- A) 731  B) **64** ✅  C) 8  D) 16

**Solution:** bottleneck = 500 kbps.

$$t = \frac{4 \times 10^6 \times 8 \text{ bits}}{500 \times 10^3 \text{ bits/s}} = 64 \text{ s}$$

**Why others are wrong:** 16 s is the time on the 2 Mbps link alone (the fastest link, not the bottleneck); 8 s and 731 s match no link's transfer time. (The lecture's reasoning assumes the transfer is pipelined so the slowest link sets the pace.)

### 6.2 Packet switch [GATE 2015]

Two hosts connected via a packet switch, links of $10^7$ bits/s, propagation delay 20 µs each, switch starts forwarding **35 µs after receiving** the packet. 10000 bits of data sent with a packet size of **5000 bits**. Time from the first bit transmitted to the last bit received (µs)?

**Answer: 1575 µs** ✅

- Data splits into **P1 and P2**, 5000 bits each.
- $T_d = 5000 / 10^7 = 500$ µs per packet per link.

**Timeline (µs):**

| Event | Time |
|---|---|
| P1 transmitted by A | 0 → 500 |
| P2 transmitted by A | 500 → 1000 |
| P1 reaches switch, processing 35 µs | 520 → 555 |
| Switch transmits P1 | 555 → 1055 |
| P2 reaches switch (1000 + 20), processing 35 µs | 1020 → 1055 |
| Switch transmits P2 (starts exactly when link frees up) | 1055 → 1555 |
| P2 reaches B (+20 µs propagation) | **1575** |

Lecture's sum: $500 + 500 + 55 + 500 + 20 = 1575$ µs
(500 = $T_d$ of P1 at A, 500 = $T_d$ of P2 at A, 55 = 20 PD + 35 holding at switch, 500 = $T_d$ of P2 at switch, 20 = PD switch to B.)
For reference, P1 alone: $500 + 20 + 35 + 500 + 20 = 1075$ µs.

### 6.3 Satellite link [GATE 2022]

100 Mbps link, earth station to satellite at **2100 km**, signal speed $3 \times 10^8$ m/s, packet = **1000 bytes**. Time (ms, 2 decimals) for the receiver to receive the whole packet?

**Answer: 7.08 ms** ✅

$$T_d = \frac{8000}{100 \times 10^6} = 0.08 \text{ ms} \qquad P_d = \frac{2100 \times 10^3}{3 \times 10^8} = 7 \text{ ms}$$

$$T_{total} = T_d + P_d = 0.08 + 7 = 7.08 \text{ ms}$$

---

## 7. Access Control — MAC Sublayer Protocols

- **Purpose:** to **reduce** the occurrence of collisions. (The lecturer strikes out "avoid" and writes "reduce": collisions can't be eliminated entirely on a shared medium.)
- Setting: stations A, B, C, D share one medium; if two transmit simultaneously, a collision occurs.

| Class | Idea | Protocols |
|---|---|---|
| **Random Access** | Any station may try to send; collisions possible | **ALOHA, CSMA** |
| **Controlled Access** | Stations take turns / need permission | **Polling, Reservation, Token Passing** |
| **Channelization** | Medium is divided among stations | **FDMA, TDMA, CDMA** |

---

## 8. CSMA — Carrier Sense Multiple Access

### Timing background (A → B)

- A sends a data packet to B: takes $P_d$. B replies: takes another $P_d$.
- $\text{RTT} = 2 P_d$ (round-trip time).
- Feedback types: **+ve acknowledgement = ACK**, **−ve acknowledgement = NAK**, or **no response** (timeout).

### CSMA rule

If a station wants to transmit:
1. **Sense** the medium (is it idle or busy, i.e. is someone transmitting a packet?).
2. **If idle, transmit; else wait.**

**Gap:** CSMA does **not define the waiting time**. Three variants fix this:

| Variant | Waiting time when the medium is busy |
|---|---|
| **1-Persistent CSMA** | **Zero**: keep sensing and transmit as soon as the medium becomes idle |
| **Non-Persistent CSMA** | Wait for a **random time**, then sense again → random time chosen via **BEB algorithm** |
| **P-Persistent CSMA** | Wait for a **random amount of time** (as written on the board; BEB arrow points to this too) |

*(Standard textbook note, not on the board: in p-persistent, once the medium is idle the station transmits with probability $p$, else waits for the next slot. Check how your reference books phrase it.)*

---

## 9. Binary Exponential Backoff (BEB)

Used to choose a random waiting time after a collision.

### Rule

- $i$ = **number of collisions that have occurred for the respective packet** (resets for each new packet).
- The station picks a random integer from

$$[\,0,\; 2^i - 1\,]$$

and waits that many slot times before retrying. The smallest number wins (unique minimum transmits; a tie collides again).

| Collisions $i$ | Range |
|---|---|
| 1 | [0, 1] |
| 2 | [0, 1, 2, 3] |
| 3 | [0, 1, 2, ..., 7] |

### Case 1: A and B collide for the first time ($i_A = i_B = 1$)

| A | B | Outcome |
|---|---|---|
| 0 | 0 | Collision |
| 0 | 1 | **A transmits** |
| 1 | 0 | **B transmits** |
| 1 | 1 | Collision |

$P(A) = \tfrac14 = 25\%$, $P(B) = \tfrac14 = 25\%$, $P(\text{collision}) = \tfrac24 = 50\%$.

### Case 2: A won the first race; both A (next packet) and B (same packet) collide again

- A chose 0, B chose 1 → A sends P1. Now A's next packet P2 collides with B's packet: **A has $i=1$ → [0,1]**, **B has $i=2$ → [0,1,2,3]**. Total 2 × 4 = 8 equally likely outcomes.

| A \ B | 0 | 1 | 2 | 3 |
|---|---|---|---|---|
| **0** | Coll | A | A | A |
| **1** | B | Coll | A | A |

$P(A) = \tfrac58$, $P(B) = \tfrac18$, $P(\text{collision}) = \tfrac28 = 25\%$.

### Case 3: A keeps winning

- A chose 0, B chose 2 → A sends P2, P3 while B waits; then A's P4 collides with B's packet again: **A: $i=1$ → [0,1]**, **B: $i=3$ → [0..7]**. Total 2 × 8 = 16 outcomes.
- A = 0: B = 0 collision; B = 1…7 → A wins (7 cases).
- A = 1: B = 0 → B wins; B = 1 collision; B = 2…7 → A wins (6 cases).

$P(A) = \tfrac{13}{16}$, $P(B) = \tfrac1{16}$, $P(\text{collision}) = \tfrac{2}{16} = \tfrac18 = 12.5\%$.

**Pattern:** the winner keeps a small window, the loser's window keeps doubling, so the loser's chance keeps dropping (the "capture effect").

### Three stations (A, B, C), all with $i = 1$ → [0, 1]

8 combinations of $(A, B, C)$: 000, 001, 010, 011, 100, 101, 110, 111. The board marks the rows as "coll". By the unique-minimum rule, 000, 001, 010, 100 and 111 collide, and 011, 101, 110 would each have a winner. Check this against the video, since the board's marking is hard to read.

---

## 10. GATE PYQs

### Q1. [GATE 2021] 12-bit Hamming codeword

A 12-bit Hamming codeword (8 data bits, 4 check bits) is $d_8 d_7 d_6 d_5 c_8 d_4 d_3 d_2 c_4 d_1 c_2 c_1$.

| Data | $d_8$ | $d_7$ | $d_6$ | $d_5$ | $d_4$ | $d_3$ | $d_2$ | $d_1$ |
|---|---|---|---|---|---|---|---|---|
| Value | 1 | 1 | 0 | $x$ | 0 | 1 | 0 | 1 |

| Check | $c_8$ | $c_4$ | $c_2$ | $c_1$ |
|---|---|---|---|---|
| Value | $y$ | 0 | 1 | 0 |

Correct values of $x$ and $y$?

- **(A) $x=0$, $y=0$** ✅
- (B) $x=0$, $y=1$
- (C) $x=1$, $y=0$
- (D) $x=1$, $y=1$

**Solution:** positions 12→1 = `1 1 0 x y 0 1 0 0 1 1 0`.
- $R_0$ (positions 3, 5, 7, 9, 11) must give $c_1 = 0$: $1 \oplus 0 \oplus 0 \oplus x \oplus 1 = 0 \Rightarrow x = 0$.
- $R_3$ (positions 9–12): $y = x \oplus 0 \oplus 1 \oplus 1 = 0$ ⇒ $y = 0$ (board: $0 \oplus 0 \oplus 1 \oplus 1 = 0$).
- Cross-check: $c_4$ (5, 6, 7, 12) = $0 \oplus 1 \oplus 0 \oplus 1 = 0$ ✓; $c_2$ (3, 6, 7, 10, 11) = $1 \oplus 1 \oplus 0 \oplus 0 \oplus 1 = 1$ ✓.

**Why others are wrong:** $x = 1$ (C, D) makes the $c_1$ parity odd, which contradicts $c_1 = 0$; $y = 1$ (B) makes the position-8 group (9–12) parity odd.

### Q2. [GATE 2026 as labelled] File transfer over three links

See [6.1](#61-file-transfer-over-three-links-gate-2026-as-labelled-on-slide). Answer **(B) 64**.

### Q3. [GATE 2015] Packet switch

See [6.2](#62-packet-switch-gate-2015). Answer **1575 µs**.

### Q4. [GATE 2022] Satellite link

See [6.3](#63-satellite-link-gate-2022). Answer **7.08 ms**.

### Q5. [GATE 2004] BEB second race

A and B are the only two stations on an Ethernet, each with a steady queue of frames. Both attempt, collide, and **A wins the first back-off race**. At the end of A's successful transmission, both A and B attempt and collide. Probability that **A wins the second back-off race**?

- (a) 0.5
- **(b) 0.625** ✅
- (c) 0.75
- (d) 1.0

**Solution:** A's new frame: $i=1$ → [0,1]; B's frame (second collision): $i=2$ → [0..3]. 8 outcomes: $P(\text{coll}) = 2/8 = 25\%$, $P(A) = 5/8 = 0.625$, $P(B) = 1/8 = 0.125$.

**Why others are wrong:**
- (a) 0.5 ignores that B's window is bigger than A's (it treats the stations as equal).
- (c) 0.75 does not come from any count of the 8 equally likely outcomes (A wins in 5).
- (d) 1.0 is impossible: collision (2/8) and B winning (1/8) both have non-zero probability.

---

## 11. Memory Aids

The lecture states no formal mnemonics. These are **added by me**:

- **Hamming = bits at power-of-2 spots (1, 2, 4, 8, 16...).** Other codes just tack $R$ on the end.
- **Error position = $R_3R_2R_1R_0$ read as binary** (all zeros = clean).
- **$2^R \ge D + R + 1$:** keep trying $R = 1, 2, 3, \dots$ until true.
- **$i$ collisions → pick from $[0, 2^i - 1]$.** Window doubles each time.
- **Persistence ladder:** 1-persistent = impatient (zero wait), non-persistent = goes away and tries later, p-persistent = flips a biased coin.
- **RTT = 2 × PD.**
- **Pipelined transfer time = size ÷ slowest link.**

---

## 12. Quick Revision Checklist

- [ ] IP address = Network ID + Host ID
- [ ] ARP (IP → MAC) vs RARP (MAC → IP)
- [ ] VRC/LRC/CRC/Checksum add $R$ contiguously; Hamming adds $R$ at non-contiguous power-of-2 positions
- [ ] $R_i$ at position $2^i$ ($R_0$=1, $R_1$=2, $R_2$=4, $R_3$=8, $R_4$=16)
- [ ] Formula $2^R \ge D+R+1$ and testing $R$ values (e.g., $D=13 \Rightarrow R=5$)
- [ ] Which positions each $R_i$ covers (binary bit $i$ = 1)
- [ ] Sender: compute even-parity check bits (example $D=11011001 \to 110111000110$)
- [ ] Receiver: recompute, read $R_3R_2R_1R_0$ as error position (example → position 9), flip to correct
- [ ] 21-bit codeword example with 5 check bits
- [ ] Delay formulas: $T_d = L/B$, $P_d = d/v$, total = $T_d + P_d$, RTT = $2P_d$
- [ ] GATE 2022 satellite: 7.08 ms
- [ ] GATE 2015 packet switch: 1575 µs (timeline with 35 µs holding delay)
- [ ] GATE 2026-labelled file transfer: bottleneck 500 kbps → 64 s
- [ ] GATE 2021 Hamming: $x=0$, $y=0$
- [ ] Access control purpose: reduce collisions
- [ ] Three classes: Random Access (ALOHA, CSMA), Controlled Access (Polling, Reservation, Token Passing), Channelization (FDMA, TDMA, CDMA)
- [ ] CSMA rule: sense, transmit if idle else wait; waiting time undefined
- [ ] ACK / NAK / no response
- [ ] 1-persistent (wait zero), non-persistent (random wait), p-persistent (random wait)
- [ ] BEB: $[0, 2^i-1]$, $i$ = collisions for that packet
- [ ] BEB probabilities: Case 1 (25 / 25 / 50%), Case 2 ($5/8$, $1/8$, $2/8$), Case 3 ($13/16$, $1/16$, $2/16$)
- [ ] GATE 2004: $P(A) = 0.625$
