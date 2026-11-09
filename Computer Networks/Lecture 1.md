# Computer Networks — Lecture 1: Basics, Topologies, Standards & OSI Model

> **GATE weightage:** 8–10 marks
> **Reference books:** (1) *Data Communications & Networking* — B. A. Forouzan, (2) *Computer Networking* — Kurose & Ross, (3) *Computer Networks* — Tanenbaum

---

## Table of Contents

1. [What is a Network?](#1-what-is-a-network)
2. [Modes of Communication](#2-modes-of-communication)
3. [Connection / Transmission Media](#3-connection--transmission-media)
4. [Line Configuration](#4-line-configuration)
5. [Network Topologies](#5-network-topologies)
6. [Tree Topology (Hybrid / Hierarchical)](#6-tree-topology)
7. [Standard vs Protocol](#7-standard-vs-protocol)
8. [OSI Model — Overview](#8-osi-model--overview)
9. [Layer-wise Functions](#9-layer-wise-functions)
10. [Types of Addresses](#10-types-of-addresses-used-in-network-communication)
11. [GATE PYQs](#11-gate-pyqs)
12. [Memory Tricks](#12-memory-tricks--mnemonics)
13. [Quick Revision Checklist](#13-quick-revision-checklist)

---

## 1. What is a Network?

A **network** is:

1. A **group (≥ 2) of nodes**
2. that can **communicate**
3. through a **connection** (medium)

**What counts as a node?** (marked ⭐⭐ in lecture — important)

- Computer
- Network device (router, hub, switch…)
- Analog device
- Digital device
- Person

---

## 2. Modes of Communication

Let **N** = total nodes, **K** = number of receivers.

### 2.1 Based on ability to Send / Receive

| Mode | Direction | Example (from lecture) |
|---|---|---|
| **Simplex (SX)** | One-way only: A → B | Keyboard (I/O → CPU), TV broadcasting, Radio broadcasting |
| **Half Duplex (HX)** | Both ways, **but one at a time**: A ⇄ B | **Bluetooth** |
| **Full Duplex / Duplex (DX)** | Both ways **simultaneously**: A ⇄ B | **Internet** communication |
| **Full-Full Duplex (FX)** | A node sends & receives with **multiple nodes simultaneously** (A, B, C in a triangle) | Multi-party simultaneous links |

> The lecture also has small margin notes ("MM-VM", "SM-FM") next to these modes that are not legible in the PDF — check your video notes for them.

### 2.2 Based on Number of Receivers

| Type | Condition | Ratio | Meaning |
|---|---|---|---|
| **Unicast** | K = 1 | 1 : 1 | One sender → one receiver |
| **Multicast** | 1 < K < N | 1 : Many | One sender → a selected group |
| **Broadcast** | K = N | 1 : N | One sender → everyone |

---

## 3. Connection / Transmission Media

### 3.1 Classification

| Type | Also called | Media |
|---|---|---|
| **Wired** | Guided | Twisted Pair, Coaxial, Optical Fiber |
| **Wireless** | Unguided | Radio, Microwave, Infrared |

### 3.2 Wired / Guided Media

| Medium | Variants | Signal carried | Note |
|---|---|---|---|
| **Twisted Pair** | Shielded (STP), Unshielded (UTP) | Electrical / EM signals | Affected by EMI |
| **Coaxial** | Thick, Thin | Electrical / EM signals | Affected by EMI |
| **Optical Fiber** | Step-index, Graded-index | **Light waves** | **No EMI** |

### 3.3 Wireless / Unguided Media

| Wave | Frequency range (as per lecture) | Line-of-Sight (LoS) needed? | Uses |
|---|---|---|---|
| **Radio** | KHz – MHz | ❌ **No** | General broadcast / wide-area |
| **Microwave** | MHz – GHz | ✅ **Yes** | **Satellite** communication, **Radar** communication |
| **Infrared (IR)** | GHz – THz | ✅ **Yes** | **Short-range** communication, **Remote sensing** applications |

---

## 4. Line Configuration

How a link is shared between devices:

| Configuration | Description |
|---|---|
| **P2P (Point-to-Point)** | A **dedicated link** between exactly **two** devices (drawn as a solid line between two dots; dashed version = wireless equivalent) |
| **Multi-Point** | A **single shared link** connected to **many** devices |

---

## 5. Network Topologies

**Topology** = the **physical (or logical) arrangement of nodes** in a network.

Types: **Bus, Star, Ring, Mesh, Tree (Hierarchical)**

### 5.1 Comparison Table

| Feature | **Bus** | **Star** | **Ring** | **Mesh** |
|---|---|---|---|---|
| **Structure** | Common cable (backbone) with drop lines | All nodes connected to a central **Hub** (controller) | Nodes in a closed loop | Every node connected to every other |
| **Line config** | Multi-point | Point-to-point (to hub) | Point-to-point | Point-to-point |
| **I/O ports per node** | **1** per node | 1 per node (hub has **N** ports) | **2** per node | **(n − 1)** per node |
| **Cables needed** | 1 common cable | **N** cables | n (one per link in loop) | **n(n − 1)/2** |
| **Traffic** | Shared medium | Hub **broadcasts** (universal) | **Uni-directional** | Dedicated links |
| **Privacy / Security** | Low | **No** privacy/security | **Less** privacy/security | **High** privacy |
| **Other points** | **Easy to maintain**; a **single cable break affects total network performance** | Common controller (hub) | – | Most cables & ports → costly |

### 5.2 Formulas (Mesh & Star)

For **n** nodes:

$$\text{Cables (Mesh)} = \frac{n(n-1)}{2}$$

$$\text{I/O ports per node (Mesh)} = n-1$$

$$\text{Cables (Star)} = N \qquad \text{I/O ports at hub} = N$$

- **n / N** = number of nodes in the network
- Mesh: each node links to the other (n − 1) nodes; each link is shared by 2 nodes, hence the division by 2.

> Total I/O ports in a full mesh = n(n − 1) (derived: n nodes × (n − 1) ports each).

---

## 6. Tree Topology

**Tree = hierarchical combination of Star topologies** (multiple controllers).

### 6.1 Rules

- If number of controllers (hubs) **> 1**, total controllers = **i**
  - **1 Primary (Root) controller** — **must be an Active Hub**
  - **(i − 1) Secondary controllers** — can be **Active Hub** *or* **Passive Hub**

$$\text{Secondary controllers} = i - 1$$

- **i** = total number of controllers (hubs).

### 6.2 Hub types

| Hub | What it is |
|---|---|
| **Active Hub** | **Hub + Repeater** (regenerates the signal) |
| **Passive Hub** | Just a connection point (no signal regeneration) |

### 6.3 Worked Example (from lecture diagram)

- Root hub: **8 ports**, two ports used to link to two secondary hubs (each 8-port)
- Ports used for uplinks (marked ✗ / crossed in the diagram) cannot host systems
- **Final answer circled in lecture: 15 systems / nodes** can be connected in total

> My reading of the sketch is approx. root ≈ 6 nodes + hub 2 ≈ 5 nodes + hub 3 ≈ 4 nodes = 15. Re-check the exact split with the video.

General idea: **usable node ports = total ports of all hubs − ports consumed by hub-to-hub links (counted at both ends)**.

### 6.4 Hybrid sketch (top-right of the slide)

A mix of **Bus → Star → Ring** joined together (a hybrid network).

---

## 7. Standard vs Protocol

| | **Standard** | **Protocol** |
|---|---|---|
| **Scope** | A set of rules for a **cluster of networks (Domain)** — a "similar cluster" | Rules followed **inside a particular network** |
| **Question it answers** | *What* is to be done for effective communication? | *How* it is implemented within a network |
| **Examples** | **OSI, TCP/IP, IEEE, ANSI, ATM, X.25** | **HTTP, FTP, SMTP, TCP/IP, TELNET, UDP** |

**Diagram idea:** Networks A, B, C, D (a similar cluster/domain) are all inside one **Standard**; each network has its own **Protocol** inside. An entity outside the cluster (the "CAT coaching" circle marked ✗ in the lecture) does not follow that standard.

### Goals of a Standard / Protocol → **Effective Communication** must be:

- **Timely** ⭐⭐⭐ (most stressed)
- **Error-free**
- **Authorized**

**ISO → OSI** = *International Organization for Standardization* created the **O**pen **S**ystems **I**nterconnection model.

---

## 8. OSI Model — Overview

**Layers (7 → 1):**

| # | Layer | Group |
|---|---|---|
| 7 | **Application** | **User Support** layers |
| 6 | **Presentation** | **User Support** layers |
| 5 | **Session** | **User Support** layers |
| 4 | **Transport** | **Intermediate** layer |
| 3 | **Network** | **System (Network) Support** layers |
| 2 | **Data Link** | **System (Network) Support** layers |
| 1 | **Physical** | **System (Network) Support** layers |

### How data moves (A → B)

- A's data goes **down** layers 7 → 1, crosses the medium, then goes **up** layers 1 → 7 at B.
- Each layer communicates **logically with its peer layer** on the other side → **Peer-to-Peer communication**.

### Control Fields (Header & Trailer)

- When data moves from **Layer X → Layer Y** (downward at the sender):
  - **Sender:** **appends** the control field (Header / Trailer)
  - **Receiver:** control field is **verified & removed**

| Field | Contains |
|---|---|
| **Header** | Source address, Destination address, Length, Timestamp info, … |
| **Trailer** | **Error handling code** (e.g., CRC/checksum) |

---

## 9. Layer-wise Functions

### 9.1 User Support Layers (software)

| Layer | Functions |
|---|---|
| **7. Application** [S/W] | Provide **user interface** |
| **6. Presentation** [S/W] | Verify **syntax** (order/format) & **semantics** (meaning) • **Encryption / Decryption** • **Compression / Decompression** |
| **5. Session** | **Active dialogue controller** • **Token management** • **Synchronization** |

### 9.2 Intermediate Layer

| Layer | Functions |
|---|---|
| **4. Transport** | **Segmentation & Reassembly** • **Port addressing** • **End-to-end delivery** (highlighted — key function) • **Congestion control** |

### 9.3 System Support Layers

| Layer | Functions |
|---|---|
| **3. Network** | **Routing** • **Logical addressing** • **Packetization & Reassembly** • **Source-to-Destination delivery** (highlighted — key function) |
| **2. Data Link (DLL)** | **Framing** • **Physical addressing** • **Error control** • **Access control** • **Flow control** • **Node-to-node delivery within a network** (highlighted — key function) |
| **1. Physical** | **Bit synchronization** • **Representation of data into bits** • **Encoding / Decoding** • **Topology** • **Mode of communication** • **Providing physical requirements to the medium** |

### 9.4 Delivery Scope — Don't Confuse

| Layer | Delivery type |
|---|---|
| **Data Link** | **Node-to-node** (hop-to-hop) within a network |
| **Network** | **Source-to-destination** (host-to-host across networks) |
| **Transport** | **End-to-end** (process-to-process) |

### 9.5 Real-world Illustration (Hyderabad → Odisha)

- Cities (Hyderabad, Pune, Bangalore, Mumbai, Goa, Odisha) = **separate networks** connected by **routers**.
- **Inside a network (e.g., Hyderabad)** → delivery uses **MAC address** [DLL].
- **Between networks** → **Router** uses **IP address** and does **routing** [Network layer], choosing a path through intermediate cities.
- At the destination network, the data reaches the correct **process** via the **port address** [Transport layer].

---

## 10. Types of Addresses Used in Network Communication

| Address | Also called | Identifies | Size | Layer |
|---|---|---|---|---|
| **MAC address** | **Physical address** | **Host / Local node** identification | **48 bits (6 bytes)** | **DLL** |
| **IP address** | **Logical address** | **Network** identification | **IPv4: 32 bits**, **IPv6: 128 bits** | **Network (NL)** |
| **Port address** | **Service point address** | **Process (type)** identification | **16 bits (2 bytes)** | **Transport (TL)** |

---

## 11. GATE PYQs

### PYQ 1 — GATE 2025 (1 Mark): Match Layer ↔ Functionality

| Layer | | Functionality |
|---|---|---|
| (a) Network | | 1. Routing |
| (b) Transport Layer (TL) | | 2. Framing & Error handling |
| (c) Data Link Layer (DLL) | | 3. Host-to-host communication |

**Correct matching (as drawn in lecture):**

| Layer | Functionality |
|---|---|
| (a) Network | **1 — Routing** |
| (b) TL | **3 — Host-to-host communication** |
| (c) DLL | **2 — Framing & Error handling** |

**Reasoning:** Routing = Network layer; Framing + error handling = DLL; the TL provides communication between the end hosts/processes. The "crossed lines" in the lecture show (b)→3 and (c)→2.

---

### PYQ 2 — GATE 2014 (1 Mark): Find the **INCORRECT** pair

| Option | Pair | Verdict |
|---|---|---|
| (a) | Network layer & **Routing** | ✅ Correct pair |
| **(b)** ✔ | **Data Link layer & Bit synchronization** | ❌ **INCORRECT — this is the answer** |
| (c) | Transport layer & **End-to-end process communication** | ✅ Correct pair |
| (d) | **MAC sublayer** & **Channel sharing** | ✅ Correct pair |

**Reasoning:**
- (a) Routing is the Network layer's job.
- **(b) Bit synchronization belongs to the *Physical layer*, not DLL → incorrect.**
- (c) Transport layer gives end-to-end (process-to-process) delivery.
- (d) MAC sublayer handles **access control / channel sharing**. Remember: **DLL = LLC + MAC**.

---

### PYQ 3 — GATE 2013 (1 Mark): Packet visits through Network & DLL

**Setup:** `S — R1 — R2 — D` (Source, two routers, Destination).
**Question:** Each packet visits the **Network layer** and **Data Link layer** how many times in the whole journey?

| Option | (Network, DLL) | Verdict |
|---|---|---|
| (a) | 4, 4 | ❌ DLL count is too low |
| (b) | 4, 3 | ❌ DLL count is too low |
| **(c)** ✔ | **4, 6** | ✅ **Correct** |
| (d) | 2, 6 | ❌ Network layer count too low (routers also use NL) |

**Step-by-step count:**

| Device | Network layer visits | DLL visits |
|---|---|---|
| S (source) | 1 | 1 (sending) |
| R1 | 1 (routing) | 2 (receive + forward) |
| R2 | 1 (routing) | 2 (receive + forward) |
| D (destination) | 1 | 1 (receiving) |
| **Total** | **4** | **6** |

**Reasoning:** Routers only go up to the **Network layer** (L1–L3) — they don't reach Transport/Application. Hosts go through all layers, but only NL & DLL are counted here. Each router has **two DLL passes** (one on the incoming link, one on the outgoing link).

> **Shortcut:** For *k* routers between S and D → **Network = k + 2**, **DLL = 2(k + 1)**. Here k = 2 → NL = 4, DLL = 6.

---

## 12. Memory Tricks / Mnemonics

*(The lecture has few explicit mnemonics; items marked ✏️ are my additions for revision.)*

- **Control field = Header + Trailer** — Header: *Source, Dest address, Length, Timestamp*; Trailer: *Error-handling code*.
- **Delivery scope:** **DLL = Node-to-node**, **NL = Source-to-Dest**, **TL = End-to-end (process)**.
- **Address sizes:** **MAC 48** bits • **IPv4 32 / IPv6 128** bits • **Port 16** bits.
- **Layer groups:** 7-6-5 = **User support**; 4 = **Intermediate**; 3-2-1 = **System support**.
- **Address ↔ identification:** MAC → *which host*; IP → *which network*; Port → *which process*.
- ✏️ **Top-down OSI mnemonic:** *"**A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing"* → Application, Presentation, Session, Transport, Network, Data link, Physical.
- ✏️ **Bit sync / encoding / topology / mode / medium** → all **Physical layer**.
- ✏️ **Mesh cables:** "**n choose 2**" = n(n − 1)/2.

---

## 13. Quick Revision Checklist

**Basics**
- [ ] Definition of network (≥ 2 nodes, communicate, connection)
- [ ] What counts as a node (computer, N/w device, analog, digital, person)

**Modes of Communication**
- [ ] Simplex, Half duplex, Full duplex, Full-Full duplex + examples
- [ ] Unicast (K=1), Multicast (1<K<N), Broadcast (K=N)

**Media**
- [ ] Guided: Twisted pair (STP/UTP), Coaxial (thick/thin), Optical fiber (step/graded index)
- [ ] Why fiber has no EMI
- [ ] Unguided: Radio (KHz–MHz, no LoS), Microwave (MHz–GHz, LoS), IR (GHz–THz, LoS)
- [ ] Uses: satellite/radar (microwave), short-range/remote sensing (IR)

**Line Configuration**
- [ ] Point-to-Point vs Multi-point

**Topologies**
- [ ] Bus: common cable, multi-point, 1 port/node, single break kills network
- [ ] Star: N cables, N ports, hub broadcast, no privacy
- [ ] Ring: 2 ports/node, unidirectional, less privacy
- [ ] Mesh: n(n−1)/2 cables, (n−1) ports, high privacy
- [ ] Tree: i controllers, 1 root (active hub), i−1 secondary (active/passive)
- [ ] Active hub = hub + repeater; example answer 15 nodes

**Standard vs Protocol**
- [ ] Standard (domain/cluster) vs Protocol (within a network) + examples
- [ ] Effective communication: Timely, Error-free, Authorized
- [ ] ISO → OSI = Open Systems Interconnection

**OSI Model**
- [ ] 7 layers in order + User support / Intermediate / System support grouping
- [ ] Peer-to-peer communication
- [ ] Header vs Trailer contents; append at sender, verify & remove at receiver
- [ ] Application (user interface)
- [ ] Presentation (syntax, semantics, encryption, compression)
- [ ] Session (dialogue control, token mgmt, synchronization)
- [ ] Transport (segmentation, port addressing, end-to-end delivery, congestion control)
- [ ] Network (routing, logical addressing, packetization, source-to-dest delivery)
- [ ] DLL (framing, physical addressing, error/access/flow control, node-to-node delivery)
- [ ] Physical (bit sync, bit representation, encoding, topology, mode, physical requirements)

**Addresses**
- [ ] MAC (48 bits, DLL), IP (32/128 bits, NL), Port (16 bits, TL)

**PYQs**
- [ ] GATE 2025: N/w–Routing, TL–Host-to-host, DLL–Framing & error handling
- [ ] GATE 2014: Incorrect pair = DLL & bit synchronization
- [ ] GATE 2013: Network = 4, DLL = 6 (option c)

---
*End of Lecture 1 notes — Computer Networks (GATE CS/IT)*
