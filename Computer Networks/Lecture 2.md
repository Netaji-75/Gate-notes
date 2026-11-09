# Computer Networks — Lecture 2: TCP/IP Model, PDUs, Network Delays & CSMA/CD (GATE Notes)

> Continuation of Lecture 1 (Intro, Topologies, OSI). This lecture is **numerical-heavy**, so master the formulas in Section 8 and the PYQs in Section 9.

---

## Table of Contents

1. [TCP/IP Model (Internet Model) vs OSI](#1-tcpip-model-internet-model-vs-osi)
2. [TCP/IP Protocol Suite](#2-tcpip-protocol-suite)
3. [PDU — Protocol Data Unit & Encapsulation](#3-pdu--protocol-data-unit--encapsulation)
4. [Network Delay (4 types)](#4-network-delay-4-types)
5. [Transmission Delay](#5-transmission-delay)
6. [Propagation Delay](#6-propagation-delay)
7. [CSMA/CD (IEEE 802.3 Ethernet) Condition](#7-csmacd-ieee-8023-ethernet-condition)
8. [Formulas & Unit Conversions](#8-formulas--unit-conversions)
9. [GATE PYQs & Worked Examples](#9-gate-pyqs--worked-examples)
10. [Segmentation → Packetization → Fragmentation](#10-segmentation--packetization--fragmentation)
11. [Memory Tricks / Analogies](#11-memory-tricks--analogies)
12. [Quick Revision Checklist](#12-quick-revision-checklist)

---

## 1. TCP/IP Model (Internet Model) vs OSI

**Origin (as told in lecture):** U.S. Army → a group of **9 people** → designed the **protocols** → **ARPANET** (**A**dvanced **R**esearch **P**rojects **A**gency **Net**work) → **IETF** → **TCP/IP**.

### OSI → TCP/IP layer mapping (5-layer version drawn in lecture)

| OSI Layer | TCP/IP Layer |
|---|---|
| 7 Application (AL) | **Application Layer** (AL) |
| 6 Presentation (PL) | ↑ merged into Application |
| 5 Session (SL) | ↑ merged into Application |
| 4 Transport (TL) | **Transport Layer** (TL) |
| 3 Network (NL) | **Internet Layer** |
| 2 Data Link (DLL) | **Data Link Layer** (DLL) |
| 1 Physical (PL) | **Physical Layer** (PL) |

- **OSI = 7 layers → TCP/IP (as drawn) = 5 layers.**
- Layers 5-6-7 of OSI collapse into one **Application layer**.
- **Network layer = "Internet layer"** in TCP/IP.

---

## 2. TCP/IP Protocol Suite

| Layer | Protocols |
|---|---|
| **Application (AL)** | HTTP, SMTP, FTP, TELNET, POP, IMAP … |
| **Transport (TL)** | SCTP, TCP, UDP |
| **Network / Internet (NL)** | IP, RIP, OSPF, BGP … |
| **Data Link (DLL)** | HDLC, SDLC, PPP, Sliding Window Protocol … |
| **Physical (PL)** | (shares the bottom block with DLL in the lecture's diagram) |

> Data flows **down** the stack at the sender and **up** at the receiver (arrows in slide).

---

## 3. PDU — Protocol Data Unit & Encapsulation

**PDU** = the unit of data at each layer (data + that layer's control field).

| Layer | PDU name | Common name |
|---|---|---|
| 5 (Application) | **APDU** | **Message** |
| 4 (Transport) | **TPDU** | **Segment** (TCP) / **Datagram** (UDP) |
| 3 (Network) | **NPDU** | **Packet** |
| 2 (Data Link) | **DPDU** | **Frame** |
| 1 (Physical) | **PPDU** | **Bits** |

### How it flows (lecture example: user sends "Hai")

**Sender (going down) — each layer adds its control field:**
- L5: `Hai` + **H5** → APDU
- L4: adds **H4** → TPDU
- L3: adds **H3** → NPDU (packet)
- L2: adds **H2** (header) **and** a **trailer** → DPDU (frame)
- L1: converts to **bits** → PPDU

**Receiver (going up) — each layer verifies and removes its own header (and trailer at DLL)**, until only "Hai" reaches the user.

> **Key point:** DLL is the only layer that adds a **trailer** (error-detection code) as well as a header.

**Network picture from slide:** S → intermediate node(s) → D, all inside one network cloud.

---

## 4. Network Delay (4 types)

**Total network delay:**

$$T = T_Q + T_{Pr} + T_T + T_P$$

| Delay | Symbol | Meaning | Typical magnitude | Considered in GATE? |
|---|---|---|---|---|
| **Queuing delay** | $T_Q$ | Time a packet **waits in queue** to get processed | ns | ❌ Ignored |
| **Processing delay** | $T_{Pr}$ | Time to **process** the packet (header check etc.) | μs | ❌ Ignored |
| **Transmission delay** | $T_T$ | Time to **load** the packet onto the medium | ms | ✅ |
| **Propagation delay** | $T_P$ | Time for the packet to **travel** to the other node | ms – sec | ✅ |

**Ignoring $T_Q$ and $T_{Pr}$ (standard GATE assumption):**

$$T = T_T + T_P$$

---

## 5. Transmission Delay

**Definition:** The time needed for a packet to be **loaded onto the medium**.

- $T_T \propto \dfrac{1}{\text{Bandwidth}}$ — more bandwidth, less delay
- $T_T \propto \text{Packet size}$ — bigger packet, more delay

$$T_T = \frac{\text{Packet size}}{\text{Bandwidth}}$$

- **Packet size** → in bits (or bytes — keep units consistent with bandwidth)
- **Bandwidth** → bits/sec (or bytes/sec)

*(Slide analogy: a wider "pipe" loads more bits per second.)*

---

## 6. Propagation Delay

**Definition:** The time taken by a packet to **reach the destination/other node**.

$$T_P = \frac{\text{Length of channel}}{\text{Propagation speed}}$$

### LAN vs WAN comparison

| Network | Relation | Why |
|---|---|---|
| **Small (LAN)** | $T_T \ge T_P$ | Short distance → tiny propagation delay |
| **Large (WAN)** | $T_T < T_P$ | Long distance → propagation dominates |

---

## 7. CSMA/CD (IEEE 802.3 Ethernet) Condition

**IEEE 802.3 = Ethernet = CSMA/CD.**

To **detect a collision**, the sender must still be transmitting when the collision signal comes back (a **round trip**):

$$T_T \ge 2 \times T_P$$

Expanded:

$$\frac{\text{Packet size}}{\text{Bandwidth}} \ge 2 \times \frac{\text{Length}}{\text{Speed}}$$

**Minimum frame size:**

$$\text{Min frame size} = 2 \times \frac{\text{Length}}{\text{Speed}} \times \text{Bandwidth}$$

**Maximum cable length** (for given frame size): rearrange the same inequality:

$$\text{Length}_{max} = \frac{\text{Frame size} \times \text{Speed}}{2 \times \text{Bandwidth}}$$

---

## 8. Formulas & Unit Conversions

### 8.1 Formula sheet

| Quantity | Formula | Variables |
|---|---|---|
| Total delay | $T = T_Q + T_{Pr} + T_T + T_P$ | Queuing, processing, transmission, propagation |
| Simplified delay | $T = T_T + T_P$ | When $T_Q, T_{Pr}$ ignored |
| Transmission delay | $T_T = \dfrac{L}{B}$ | $L$ = packet size, $B$ = bandwidth |
| Propagation delay | $T_P = \dfrac{d}{v}$ | $d$ = channel length, $v$ = signal speed |
| CSMA/CD condition | $T_T \ge 2T_P$ | Round trip must fit inside transmission |
| Min frame size | $L_{min} = 2\dfrac{d}{v}B$ | |

**Pipelined store-and-forward shortcut** (derived from the lecture's method; equal links, $k$ links, $n$ packets, per-link $T_T = t$, $T_P = p$):

$$T_{total} = (n + k - 1)\,t + k\,p$$

- Check with GATE 2012 numbers: $(1000+3-1)(1) + 3(1) = 1005$ ms ✅

### 8.2 Data units

| Unit | Value |
|---|---|
| 4 bits | 1 **nibble** |
| 8 bits | 1 **byte** |
| 1 KB | $2^{10}$ = 1024 bytes |
| 1 MB | $2^{20}$ bytes (1024 KB) |
| 1 GB | $2^{30}$ bytes (1024 MB) |
| 1 TB | $2^{40}$ bytes (1024 GB) |
| 1 PB (Peta) | $2^{50}$ bytes (1024 TB) |
| Exa → Zetta → Yotta | each ×1024 of the previous |

### 8.3 Approximation for CN problems

| Binary | Decimal approx (use in CN) |
|---|---|
| $2^{10}$ (K) | $10^3$ |
| $2^{20}$ (M) | $10^6$ |
| $2^{30}$ (G) | $10^9$ |
| $2^{40}$ (T) | $10^{12}$ |
| $2^{50}$ (P) | $10^{15}$ |

> **For CN problems, values can be approximated** (bandwidth: 1 Kbps = $10^3$ bps, 1 Mbps = $10^6$, 1 Gbps = $10^9$).

---

## 9. GATE PYQs & Worked Examples

### PYQ 1 — GATE 2014 (1M): Match Group 1 ↔ Group 2

| Group 1 | Group 2 |
|---|---|
| P. Data link | 1. Ensures reliable transport of data over a physical point-to-point link |
| Q. Network layer | 2. Encodes/decodes data for physical transmission |
| R. Transport layer | 3. Allows end-to-end communication between two processes |
| | 4. Routes data from one network node to the next |

| Option | Mapping | Verdict |
|---|---|---|
| **A ✔** | **P-1, Q-4, R-3** | ✅ Correct |
| B | P-2, Q-4, R-1 | ❌ Encoding/decoding is Physical layer, not DLL |
| C | P-2, Q-3, R-1 | ❌ Same DLL error; also end-to-end process communication is Transport, not Network |
| D | P-1, Q-3, R-2 | ❌ Network ≠ end-to-end process communication; Transport doesn't do encoding |

**Answer: A.** Lecture hint: DLL = "error-free transport" over a point-to-point link.

---

### PYQ 2 — GATE 2019: Minimum number of switches

*15 machines in a LAN using **8-port Ethernet switches** with **no separate uplink ports**. Minimum switches needed = ?*

- 2 switches: 16 ports − 2 (one port at each end to link them) = **14 < 15** ❌
- 3 switches (chain): 24 ports − 4 (2 links × 2 ends) = **20 ≥ 15** ✅

**Answer: 3**

---

### PYQ 3 — GATE 2003: Minimum packet size

*2 km broadcast LAN, $10^7$ bps, CSMA/CD, signal speed $2\times10^8$ m/s.*

Options: a) 50 bytes b) 100 bytes c) 200 bytes **d) None of these ✔**

$$L \ge 2 \times \frac{2000}{2\times10^8} \times 10^7 = 2\times10^{-5}\times10^{7} = 200 \text{ bits} = 25 \text{ bytes}$$

| Option | Why wrong |
|---|---|
| a) 50 B | Doesn't equal 25 B |
| b) 100 B | Doesn't equal 25 B |
| c) 200 B | Confuses **200 bits** with 200 bytes (bits→bytes conversion missed) |

**Answer: (d) None of these** (correct value = 25 bytes)

---

### PYQ 4 — GATE 2005: Minimum frame size

*CSMA/CD, 1 Gbps, 1 km cable, no repeaters, speed $2\times10^8$ m/s.*

$$L \ge 2\times\frac{1000}{2\times10^8}\times10^9 = 10^{-5}\times10^{9} = 10^4 \text{ bits}$$

| Option | Verdict |
|---|---|
| **a) 10000 bits ✔** | Correct |
| b) 10000 bytes | Bits/bytes mix-up (8× too large) |
| c) 5000 bits | Forgot the factor of **2** (round trip) |
| d) 5000 bytes | Both mistakes |

**Answer: (a)** *(slide note: use 1K = $10^3$)*

---

### PYQ 5 — GATE 2007: Slotted LAN probability

*n stations, each transmits with probability p per slot. P(exactly one station transmits) = ?*

| Option | Expression | Verdict |
|---|---|---|
| **a) ✔** | $np(1-p)^{n-1}$ | ✅ Pick which station (n ways) transmits (p), other n−1 stay silent |
| b) | $(1-p)^{n-1}$ | ❌ Only the probability that n−1 specific stations are silent |
| c) | $p(1-p)^{n-1}$ | ❌ Probability that one *particular* station is the sole transmitter (missing factor n) |
| d) | $1-(1-p)^{n-1}$ | ❌ Probability that at least one of n−1 others transmits |

**Answer: (a)** *(not ticked on the slide; solved here with the binomial reasoning)*

---

### PYQ 6 — GATE 2008: Minimum frame size

*CSMA/CD, 1 Gbps, 200 m cable, speed $2\times10^8$ m/s.*

$$L \ge 2\times\frac{200}{2\times10^8}\times10^9 = 2000 \text{ bits} = 250 \text{ bytes}$$

| Option | Verdict |
|---|---|
| a) 125 B | Forgot the factor 2 |
| **b) 250 B ✔** | Correct |
| c) 500 B | Doubled twice |
| d) None | Not applicable |

**Answer: (b) 250 bytes**

---

### PYQ 7 — GATE 2012: PDU of Application layer (Internet stack)

Options: a) segment b) datagram **c) message ✔** d) frame

| Option | Why wrong |
|---|---|
| a) segment | Transport (TCP) PDU |
| b) datagram | Transport (UDP) / Network PDU |
| d) frame | Data Link PDU |

**Answer: (c) message**

---

### PYQ 8 — GATE 2012: File transfer S → R1 → R2 → D (store-and-forward, pipelined)

*File = $10^6$ bits, split into **1000 packets of 1000 bits**. 3 links (L1: S–R1, L2: R1–R2, L3: R2–D), each **100 km**, speed $10^8$ m/s, bandwidth **1 Mbps**. Find total transmission + propagation delay.*

**Per link per packet:**

$$T_T = \frac{10^3}{10^6} = 10^{-3}\text{ s} = 1\text{ ms}, \qquad T_P = \frac{10^5}{10^8} = 1\text{ ms}$$

**One packet through 3 links:** $T_{T1}+T_{P1}+T_{T2}+T_{P2}+T_{T3}+T_{P3} = 6$ ms

**Pipelining table:**

| Packet # | Arrives at D |
|---|---|
| 1st | 6 ms |
| 2nd | 7 ms |
| 3rd | 8 ms |
| … | … |
| 100th | 105 ms |
| … | … |
| **1000th** | **1005 ms** |

*(Each new packet arrives 1 ms after the previous, because the transmission time of one packet is 1 ms.)*

| Option | Verdict |
|---|---|
| **a) 1005 ms ✔** | $6 + 999\times1$ |
| b) 1010 ms | Over-counts the pipeline |
| c) 3000 ms | Ignores pipelining; counts only the transmission for 3 links one after another |
| d) 3003 ms | No pipelining at all |

**Answer: (a) 1005 ms**

---

### PYQ 9 — GATE 2015: Signal speed

*CSMA/CD, 100 Mbps, 1 km cable, no repeaters, min frame size = 1250 bytes. Signal speed (km/s)?*

$$\frac{1250\times8}{10^8} \ge 2\times\frac{1}{x} \;\Rightarrow\; 10^{-4} \ge \frac{2}{x} \;\Rightarrow\; x \ge 2\times10^{4}\text{ km/s}$$

| Option | Verdict |
|---|---|
| a) 8000, b) 10000, c) 16000 | ❌ All below 20000 → round trip would be longer than transmission time |
| **d) 20000 ✔** | Correct (the required minimum) |

**Answer: (d) 20000 km/s**

---

### PYQ 10 — GATE 2013: Maximum cable length

*500 Mbps Ethernet LAN, frame size 10000 bits, signal speed 200000 km/s. Maximum length (km)?*

$$2\times\frac{l}{2\times10^5} \le \frac{10^4}{5\times10^8} \;\Rightarrow\; l \le \frac{10^4\times10^5}{5\times10^8} = 2\text{ km}$$

| Option | Verdict |
|---|---|
| a) 1 | Satisfies inequality but isn't the **maximum** |
| **b) 2 ✔** | Correct |
| c) 2.5, d) 5 | Exceed the limit → collision cannot be detected |

**Answer: (b) 2 km**

---

### PYQ 11 — GATE 2016: Minimum frame size

*Bandwidth $20\times10^6$ bps, CSMA/CD, max propagation time 40 μs. Min frame size (bytes)?*

$$L \ge 2\times40\times10^{-6}\times20\times10^{6} = 1600\text{ bits} = \mathbf{200\text{ bytes}}$$

**Answer: 200 bytes**

---

### PYQ 12 — GATE 2026: Bottleneck link file transfer

*File = 4 million bytes over 3 consecutive links of **2 Mbps, 500 kbps, 1 Mbps**. Processing and propagation delays negligible, no background traffic, no overhead. Total time (s)? ($1\text{M}=10^6, 1\text{K}=10^3$)*

Options: A) 731 B) 64 C) 8 D) 16

**Time to push the entire file through each link (lecture working):**

| Link | Bandwidth | Time = $\dfrac{32\times10^6\text{ bits}}{BW}$ |
|---|---|---|
| 1 | 2 Mbps | $T_1 = 16$ s |
| 2 | 500 kbps | $T_2 = 64$ s |
| 3 | 1 Mbps | $T_3 = 32$ s |

(File: $4\times10^6$ B $= 32\times10^6$ bits.)

- Because data streams through the links continuously (the pipe-with-chunks drawing at the end of this problem), the **slowest (bottleneck) link decides the total time** → **64 s**.
- A (731) is far larger than any sensible total; C (8) and D (16) are *smaller than the bottleneck link's own 64 s*, which is impossible.

**Answer: (B) 64 s** *(the slide stops before the final tick; this is derived from the bottleneck idea in the lecture's own working)*

---

### Homework (unsolved in lecture)

*Ethernet segment, $10^8$ bits/s, max segment length 500 m, signal speed $2\times10^8$ m/s. Minimum frame size (bits) for collision detection?*

$$L \ge 2\times\frac{500}{2\times10^8}\times10^8 = \mathbf{500\text{ bits}}$$

*(My solution, using the lecture's formula.)*

---

### PYQ 13 — GATE 2014 (MCQ, 2M): File sent in 1, 10 or 20 packets

**Setup:** A → R1 → R2 → B. **Header = 100 bytes per packet.** File = $10^3$ bytes. Bandwidth of each link = $10^6$ bytes/s. (Propagation ignored.)

| Case | Packets | Packet size | $T_T$ per link | One packet end-to-end |
|---|---|---|---|---|
| 1 | 1 | 1000 + 100 = **1100 B** | 1.1 ms | 3.3 ms |
| 2 | 10 | 100 + 100 = **200 B** | 0.2 ms | 0.6 ms |
| 3 | 20 | 50 + 100 = **150 B** | 0.15 ms | 0.45 ms |

**Case 1:** $T_1 = 3\times1.1 = \mathbf{3.3\text{ ms}}$

**Case 2:** 1st packet = 0.6 ms; each next arrives +0.2 ms
- 2nd = 0.8 ms … 10th = $0.6 + 9\times0.2 = \mathbf{2.4\text{ ms}} = T_2$

**Case 3:** $T_3 = 0.45 + 19\times0.15 = 0.45 + 2.85 = \mathbf{3.3\text{ ms}}$

| Option | Statement | Verdict |
|---|---|---|
| a) | $T_1 < T_2 < T_3$ | ❌ $T_2$ is the smallest |
| b) | $T_1 > T_2 > T_3$ | ❌ $T_2 < T_3$ |
| c) | $T_2 = T_3,\ T_3 < T_1$ | ❌ $T_2 \ne T_3$, and $T_3 = T_1$ |
| **d) ✔** | **$T_1 = T_3,\ T_3 > T_2$** | ✅ 3.3 = 3.3 > 2.4 |

**Answer: (d)**

> **Insight:** Splitting into more packets improves pipelining but adds more header overhead. There's a sweet spot (10 packets here); too many packets bring the total back up.

---

## 10. Segmentation → Packetization → Fragmentation

### 10.1 Message breakdown (lecture diagram)

```
Message (at source S)
 ├── Segment 0   (Transport layer segmentation)
 │     ├── Packet 0 ──► frames   (Network layer packetization,
 │     ├── Packet 1 ──► frames    then DLL framing)
 │     └── Packet 2 ──► frames
 └── Segment 1
       └── Packets … ──► frames
```

- **Transport layer** → splits message into **segments**
- **Network layer** → splits segments into **packets**
- **Data link layer** → splits packets into **frames** → physical layer sends **bits**
- A **gateway** appears on the right side of the slide (where packets pass between networks).

### 10.2 Router fragmentation example (my reading of the slide)

- A packet from the Network layer: **content = 100 bytes + header H3 = 50 bytes** (H3 carries info such as **length**).
- Incoming link can carry **200 bytes**; the next link only **70 bytes**.
- With a 50-byte header on each piece, only **20 bytes** of content fit per 70-byte unit.

$$\frac{100\text{ bytes}}{20\text{ bytes}} = 5 \text{ fragments}$$

- Each fragment = 20 B data + 50 B header = 70 B → total sent = 350 B (more overhead!).

> ⚠️ The slide shows only numbers and sketches (100 + 50 → five 20s; pipes of 200 B and 70 B). The above is my interpretation (this is the IP fragmentation idea; the limit is called **MTU**). Verify with your video.

---

## 11. Memory Tricks / Analogies

**Bus-ticket-counter analogy (from lecture) — the 4 delays:**

| Scene | Delay |
|---|---|
| People **waiting in the queue** at the bus ticket counter | **Queuing delay** $T_Q$ |
| The person at the counter **issuing your ticket** | **Processing delay** $T_{Pr}$ |
| Passengers **boarding / loading into the bus** | **Transmission delay** $T_T$ |
| The **bus journey** from A to B (e.g., 200 km at a given speed) | **Propagation delay** $T_P$ |

**Other tips**
- **$T_T$ = loading into the pipe; $T_P$ = travelling along the pipe.**
- **LAN:** loading is slow relative to travel ($T_T \ge T_P$). **WAN:** travel is slow ($T_T < T_P$).
- **CSMA/CD rule: "Transmit for at least twice the travel time"** → $T_T \ge 2T_P$.
- **PDU names bottom-up:** Bits → Frame → Packet → Segment/Datagram → Message.
- **Always convert units first** (bits vs bytes, ms vs s, km vs m) — most wrong GATE options are unit slips (÷8, ×2).
- **Common option traps:** forgetting the factor **2**; mixing **bits ↔ bytes**; ignoring pipelining in multi-packet problems.

---

## 12. Quick Revision Checklist

**TCP/IP model**
- [ ] ARPANET → IETF → TCP/IP origin
- [ ] OSI (7) ↔ TCP/IP (5-layer drawing) mapping; AL = OSI 5+6+7; Internet layer = Network layer
- [ ] Protocols at each layer (HTTP/SMTP/FTP/TELNET/POP/IMAP; SCTP/TCP/UDP; IP/RIP/OSPF/BGP; HDLC/SDLC/PPP/Sliding Window)

**PDU & encapsulation**
- [ ] APDU/TPDU/NPDU/DPDU/PPDU and their names (message, segment/datagram, packet, frame, bits)
- [ ] Headers added going down, removed going up; trailer only at DLL

**Delays**
- [ ] $T = T_Q + T_{Pr} + T_T + T_P$ and which ones are ignored
- [ ] Orders of magnitude (ns, μs, ms, ms–s)
- [ ] $T_T$ = packet size / bandwidth; $T_P$ = length / speed
- [ ] LAN: $T_T \ge T_P$; WAN: $T_T < T_P$
- [ ] Bus-ticket-counter analogy

**CSMA/CD**
- [ ] IEEE 802.3 = Ethernet = CSMA/CD
- [ ] $T_T \ge 2T_P$ → min frame size, max cable length, min signal speed (rearranging)

**Units**
- [ ] nibble / byte / KB…YB ladder; CN approximations ($10^3, 10^6, 10^9, 10^{12}$)

**PYQs / problem types**
- [ ] Layer–function matching (GATE 2014)
- [ ] Min switches (GATE 2019) → 3
- [ ] Min packet size (GATE 2003) → 25 B (None of these)
- [ ] Min frame size (GATE 2005) → 10000 bits
- [ ] Slotted LAN probability (GATE 2007) → $np(1-p)^{n-1}$
- [ ] Min frame size (GATE 2008) → 250 B
- [ ] PDU of application layer (GATE 2012) → message
- [ ] Pipelined file transfer (GATE 2012) → 1005 ms
- [ ] Signal speed (GATE 2015) → 20000 km/s
- [ ] Max cable length (GATE 2013) → 2 km
- [ ] Min frame size (GATE 2016) → 200 B
- [ ] Bottleneck link transfer (GATE 2026) → 64 s
- [ ] Packet-splitting 1/10/20 packets (GATE 2014, 2M) → $T_1 = T_3 > T_2$
- [ ] Homework: Ethernet 500 m, $10^8$ bps → 500 bits

**Segmentation & fragmentation**
- [ ] Message → segments → packets → frames → bits
- [ ] Router fragmentation: 100 B data, 50 B header, 70 B limit → 5 fragments

---
*Source: GeeksforGeeks GATE lecture slides — Computer Networks Lecture 2.*
