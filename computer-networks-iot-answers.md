# Computer Networks & IoT — Solved Question Bank (7 marks each)

---

## 1. Framing and its Types

**Definition.** The data link layer receives a raw bit stream from the physical layer. **Framing** is the process of dividing this continuous stream into manageable, identifiable units called *frames*, each carrying a source address, destination address, control information and payload. Framing lets the receiver know where one message ends and the next begins.

**Why framing is needed**
- Error control is practical only on small units — if one frame is corrupted, only that frame is retransmitted.
- Allows addressing so that a message can be delivered to the correct node on a shared medium.
- Provides synchronisation between sender and receiver.

### Types of Framing

**A. Fixed-size framing**
- Every frame is of the same, pre-agreed length.
- No delimiter is required — the length itself defines the boundary.
- Example: ATM cells (53 bytes each).
- *Drawback:* internal fragmentation when data is smaller than the frame; inflexible.

**B. Variable-size framing**
Frame length varies, so an explicit boundary marker is required. Two approaches:

**(i) Character-oriented (byte-oriented) framing**
- Data is a sequence of 8-bit characters from a coding system (ASCII, EBCDIC).
- A special 8-bit **flag** byte marks the start and end of a frame.
- Problem: the flag pattern may appear inside the data → solved by **byte stuffing**.
- Example: PPP, BISYNC.

```
+------+--------+---------------+-------+------+
| Flag | Header |      Data     | Trail | Flag |
+------+--------+---------------+-------+------+
```

**(ii) Bit-oriented framing**
- Data is treated as a plain sequence of bits (not characters).
- Delimiter flag is the bit pattern `01111110`.
- Problem: the flag pattern may occur in the data → solved by **bit stuffing** (insert a 0 after five consecutive 1s).
- Example: HDLC, SDLC.

**(iii) Length-count framing (character count)**
- A field in the header states the number of bytes in the frame.
- Simple, but if the count field is corrupted the receiver loses all frame boundaries.

**Comparison table**

| Basis | Fixed-size | Character-oriented | Bit-oriented |
|---|---|---|---|
| Delimiter | Not needed | Flag byte | Flag bit pattern `01111110` |
| Transparency method | — | Byte stuffing | Bit stuffing |
| Data unit | Bits | 8-bit characters | Bits |
| Example | ATM | PPP, BISYNC | HDLC |

---

## 2. Byte Stuffing (with example)

**Need.** In character-oriented framing, the flag byte `01111110` marks the frame boundary. If this same pattern appears as part of the *data* (which happens with images, executables, compressed files), the receiver would wrongly conclude the frame has ended. This destroys **data transparency**.

**Definition.** Byte stuffing (character stuffing) is the process of adding one extra 8-bit **escape character (ESC = `01111101`)** to the data section of a frame whenever a byte with the same pattern as the **flag** or the **ESC** character itself occurs.

### Rules
1. Sender scans the data field byte by byte.
2. If a byte equals **FLAG** → insert an **ESC** before it.
3. If a byte equals **ESC** → insert another **ESC** before it.
4. Receiver removes every ESC and treats the byte that follows it as ordinary data.

### Example

Let **FLAG = F** and **ESC = E**.

Original data to send:

```
A   B   F   C   E   D
```

After byte stuffing at the sender, the transmitted frame is:

```
FLAG | A  B  E  F  C  E  E  D | FLAG
              ^stuffed    ^stuffed
```

At the receiver:
- The first `E` is removed, the following `F` is accepted as **data**, not as a flag.
- The second `E` is removed, the following `E` is accepted as **data**.
- Recovered data = `A B F C E D` — identical to the original. ✔

### Numeric illustration

| | Bit pattern |
|---|---|
| FLAG | 01111110 |
| ESC | 01111101 |
| Data | 11001100 **01111110** 10101010 |
| Sent | 01111110 · 11001100 · **01111101 01111110** · 10101010 · 01111110 |

**Advantage:** guarantees transparency for any binary data.
**Disadvantage:** frame size grows; depends on 8-bit character sets, so unsuitable for systems using 16-bit characters (Unicode) — bit stuffing is preferred there.

---

## 3. Network Topologies

**Topology** is the geometric/logical arrangement of links and nodes in a network — i.e. how devices are physically or logically interconnected.

### (a) Bus Topology
All devices connect to a single backbone cable through drop lines and taps; terminators at both ends absorb signals.

```
   T---+-----+-----+-----+---T
       |     |     |     |
      PC1   PC2   PC3   PC4
```
- **Advantages:** least cabling, cheap, easy to install for small networks.
- **Disadvantages:** backbone failure kills the whole network; difficult fault isolation; heavy collisions; signal reflection at taps.

### (b) Star Topology
Every device has a dedicated point-to-point link to a **central hub/switch**. No direct device-to-device link.

```
        PC1
         |
  PC4--[HUB]--PC2
         |
        PC3
```
- **Advantages:** easy to install and reconfigure; robust — one link failure affects only one node; simple fault identification; less cabling than mesh.
- **Disadvantages:** single point of failure (hub); more cable than bus; hub cost.

### (c) Ring Topology
Each device has a dedicated point-to-point link with only the two devices on either side. Signal travels in one direction, regenerated by each repeater.

```
      PC1 --> PC2
       ^        |
       |        v
      PC4 <-- PC3
```
- **Advantages:** easy to install; fault isolation is simple (alarm signal); no collisions with token passing.
- **Disadvantages:** a single break disrupts the entire ring (solved by dual ring); unidirectional traffic adds delay; adding a node breaks the ring.

### (d) Mesh Topology
Every device has a dedicated point-to-point link to every other device.
Number of links = **n(n−1)/2**; each node needs **n−1** I/O ports.
- Discussed in detail in Q7.

### (e) Tree (Hierarchical) Topology
A variation of star — secondary hubs connect to a central root hub, forming a hierarchy.
- **Advantages:** allows more devices, supports signal regeneration, easy expansion, priority isolation of groups.
- **Disadvantages:** root failure collapses the network; more cabling; harder to configure.

### (f) Hybrid Topology
A combination of two or more topologies (e.g. star-bus, star-ring) — used in most large real networks.
- **Advantages:** flexible, scalable, reliable, faults confined to a segment.
- **Disadvantages:** complex design, expensive hubs/switches.

### Comparison

| Topology | Links needed | Cost | Reliability | Fault isolation |
|---|---|---|---|---|
| Bus | 1 backbone | Very low | Low | Difficult |
| Star | n | Moderate | Good (except hub) | Easy |
| Ring | n | Low | Moderate | Moderate |
| Mesh | n(n−1)/2 | Very high | Very high | Very easy |
| Tree | n | Moderate–high | Moderate | Easy |

---

## 4. Bandwidth–Delay Product & Utilisation (Numerical)

### Bandwidth–Delay Product (BDP)

**Definition.** The bandwidth-delay product is the product of the link bandwidth (bits/sec) and the round-trip delay (sec):

> **BDP = Bandwidth × Round-trip Delay**

**Meaning.** It gives the **maximum number of bits that can be in transit (in flight) on the link at any instant** — i.e. the "volume" or capacity of the pipe. It defines the number of bits a sender may transmit before the first acknowledgement can possibly return, and therefore determines the **optimum window size** of a sliding-window protocol.

- If **window size < BDP** → the link is under-utilised (sender idles waiting for ACK).
- If **window size = BDP** → link is 100 % utilised.

### Given Data
- Bandwidth, B = 2 Mbps = 2 × 10⁶ bits/s
- Round-trip time (RTT) for 1 bit = 20 ms = 20 × 10⁻³ s
- Frame size = 1000 bits
- Protocol: Stop-and-Wait ARQ

### Step 1 — Bandwidth-Delay Product

```
BDP = Bandwidth × RTT
    = (2 × 10⁶ bits/s) × (20 × 10⁻³ s)
    = 40,000 bits
```

**BDP = 40,000 bits = 40 kbits**

The link can hold 40,000 bits in flight during one round trip.

### Step 2 — Utilisation of the Link

In Stop-and-Wait ARQ the sender transmits **only one frame (1000 bits)** and then waits idle for the whole round-trip time.

```
Utilisation = (Frame size) / (Bandwidth-Delay Product)
            = 1000 / 40,000
            = 0.025
            = 2.5 %
```

### Result
> **Bandwidth-delay product = 40,000 bits**
> **Link utilisation = 2.5 %** (97.5 % of capacity is wasted)

### Interpretation
Only 1000 of the 40,000 possible in-flight bits are used, so Stop-and-Wait is extremely inefficient on high-bandwidth or long-delay links. To reach 100 % utilisation the sender would need a window of 40,000/1000 = **40 frames**, i.e. a sliding-window protocol (Go-Back-N or Selective Repeat) with window size ≥ 40.

---

## 5. CIDR Notation

**CIDR = Classless Inter-Domain Routing** (RFC 1519), introduced in 1993 to replace the rigid classful scheme.

**Definition.** CIDR notation is a compact way of writing an IP address together with the length of its network prefix, in the form:

> **x.y.z.w / n**

where `n` (the *prefix length* or *slash notation*) is the number of leading bits that identify the **network**, and the remaining `32 − n` bits identify the **host**.

### Key points
1. **Mask derivation:** the subnet mask is `n` ones followed by `(32−n)` zeros.
   `/24` → `11111111.11111111.11111111.00000000` → `255.255.255.0`
2. **Number of addresses in the block = 2^(32−n)**
   Usable hosts = 2^(32−n) − 2 (network address + broadcast address excluded).
3. **First address (network address)** = address AND mask.
   **Last address (broadcast)** = address OR (complement of mask).
4. Block size must be a **power of 2**, and the first address must be **divisible by the block size**.

### Example
**192.168.10.0/26**
- Prefix = 26 bits → mask = 255.255.255.192
- Block size = 2^(32−26) = 2⁶ = 64 addresses
- Range = 192.168.10.0 → 192.168.10.63
- Network = 192.168.10.0, Broadcast = 192.168.10.63, Usable hosts = 62

### Advantages of CIDR
- **Efficient address utilisation** — blocks of any power-of-two size (variable-length subnetting, VLSM), reducing IPv4 exhaustion.
- **Route aggregation / supernetting** — many contiguous networks are advertised as one route, shrinking routing tables (e.g. 200.1.0.0/22 replaces four /24 routes).
- **Hierarchical allocation** — ISPs receive large blocks and sub-allocate flexibly.
- Removes the artificial Class A/B/C boundaries.

---

## 6. Classful Addressing Scheme

In the original IPv4 design (pre-1993), the 32-bit address space was divided into five fixed classes, determined by the **leading bits** of the first octet. Each address consists of a **NetID** and a **HostID** whose lengths are fixed by the class.

### The Five Classes

| Class | Leading bits | First octet range | NetID / HostID | No. of networks | Hosts per network | Default mask |
|---|---|---|---|---|---|---|
| **A** | 0 | 0 – 127 | 8 / 24 | 2⁷ = 128 | 2²⁴ − 2 = 16,777,214 | 255.0.0.0 (/8) |
| **B** | 10 | 128 – 191 | 16 / 16 | 2¹⁴ = 16,384 | 2¹⁶ − 2 = 65,534 | 255.255.0.0 (/16) |
| **C** | 110 | 192 – 223 | 24 / 8 | 2²¹ = 2,097,152 | 2⁸ − 2 = 254 | 255.255.255.0 (/24) |
| **D** | 1110 | 224 – 239 | Multicast (no host part) | — | — | Not defined |
| **E** | 1111 | 240 – 255 | Reserved / experimental | — | — | Not defined |

### Structure

```
Class A:  0|  NetID (7)  |        HostID (24)         |
Class B:  10|   NetID (14)     |     HostID (16)      |
Class C:  110|      NetID (21)         | HostID (8)   |
Class D:  1110|         Multicast address (28)        |
Class E:  1111|            Reserved (28)              |
```

### Address occupancy
Class A = 50 %, Class B = 25 %, Class C = 12.5 %, Class D = 6.25 %, Class E = 6.25 % of the total space.

### Special / reserved addresses
- **127.0.0.0/8** → loopback (127.0.0.1)
- **0.0.0.0** → this host / default route
- **255.255.255.255** → limited broadcast
- Private ranges: 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16

### Advantages
- Simple: the class (and hence the mask) is known from the address itself — no mask needs to be carried in routing updates.
- Easy for routers to extract NetID quickly.

### Disadvantages (why it was abandoned)
- **Severe address wastage.** An organisation with 300 hosts needed a Class B (65,534 addresses) — over 65,000 wasted. One with 2 hosts still consumed a Class C block.
- **Rapid depletion** of Class B addresses.
- **Routing table explosion** — no aggregation possible.
- Fixed, coarse granularity — nothing between 254 and 65,534 hosts.

These problems led to subnetting, then to **CIDR / classless addressing**.

---

## 7. Advantages and Disadvantages of Mesh Topology

**Definition.** In a mesh topology every device has a **dedicated point-to-point link** with every other device in the network. The links carry traffic only between the two devices they connect.

For **n** nodes:
- Number of duplex links = **n(n − 1)/2**
- I/O ports required per device = **n − 1**

Example: for n = 6 → 6(5)/2 = **15 links**, 5 ports per node.

```
       A-------B
       |\ \   /|\
       | \  X  | \
       |  \/ \ |  \
       C--/---\D---E
        \      |  /
         \-----F-/
```

### Advantages

1. **Dedicated links / no traffic contention** — each link carries data of only two devices, eliminating sharing problems and collisions; guarantees bandwidth.
2. **High robustness and fault tolerance** — failure of one link does not disable the entire system; traffic is rerouted through alternative paths.
3. **Privacy and security** — a message travels along a dedicated line seen only by the intended receiver.
4. **Easy fault identification and isolation** — traffic can be routed to avoid a suspected link, and faults are localised to a single link.
5. **No single point of failure** — unlike star (hub) or bus (backbone).
6. **Low latency / high throughput** — no queuing behind other nodes' traffic.
7. **Expansion does not disturb existing users** (in partial mesh).

### Disadvantages

1. **Huge cabling requirement** — n(n−1)/2 links; for 100 nodes that is 4,950 links.
2. **Large number of I/O ports** — every node needs n−1 interfaces, raising hardware cost.
3. **Very high installation cost** — cable, ducts, connectors and NICs dominate the budget.
4. **Difficult installation and reconfiguration** — physical space for wiring can exceed available capacity in walls/ceilings/floors.
5. **Poor scalability** — adding one node requires links to every existing node (cost grows as O(n²)).
6. **High maintenance overhead** and redundant, often unused capacity.
7. **Bulk of wiring** may cause management and cooling problems.

### Practical use
Because of cost, **full mesh** is used only where reliability is critical and node count is small — e.g. the backbone of a telephone network, connections between core routers of an ISP, or a hybrid where a mesh backbone links star-connected LANs. Wireless networks (WSN, IoT, Zigbee) use **partial mesh** since the "cabling" cost disappears.

---

## 8. CRC Problem

**Given:**
- Message (dataword) **M = 11100011** (8 bits)
- Generator polynomial **G(x) = x³ + x² + 1** → divisor **1101** (4 bits)
- Degree of generator, r = **3** → 3 redundant (CRC) bits

### (i) Determine the Transmitted Frame

**Step 1: Append r = 3 zeros to the message**

```
Dividend = 11100011 000  →  11100011000
```

**Step 2: Perform modulo-2 (XOR) division by 1101**

```
                    1 0 1 0 0 1 1 0
             ┌──────────────────────────
      1101   │ 1 1 1 0 0 0 1 1 0 0 0
               1 1 0 1
               ───────
               0 0 1 1 0
                   0 0 0 0
                   ───────
                 0 1 1 0 0
                   1 1 0 1
                   ───────
                   0 0 0 1 1
                       0 0 0 0
                       ───────
                       0 0 1 1 1
                           0 0 0 0
                           ───────
                           0 1 1 1 0
                             1 1 0 1
                             ───────
                             0 0 1 1 0
                                 0 0 0 0
                                 ───────
                                 0 1 1 0 0
                                   1 1 0 1
                                   ───────
                                   0 0 0 1  ← Remainder = 001
```

**Remainder (CRC / checksum) = 001**

**Step 3: Form the codeword**
Replace the appended zeros with the remainder:

```
Transmitted frame = Message + CRC
                  = 11100011 + 001
```

> ### **Transmitted frame = 1 1 1 0 0 0 1 1 0 0 1  (11 bits)**

**Verification at receiver (no error):** dividing `11100011001` by `1101` gives remainder **000** → frame accepted. ✔

---

### (ii) Last Bit Flipped — Is the Error Detected?

Transmitted frame : `1110001100**1**`
Last bit flipped → Received frame : `1110001100**0**`

**Divide received frame 11100011000 by 1101:**

This is exactly the division performed in Step 2 above, whose remainder was **001**.

| | |
|---|---|
| Received frame | 11100011000 |
| Divisor | 1101 |
| **Syndrome (remainder)** | **001 ≠ 000** |

### Conclusion
Since the remainder (syndrome) is **non-zero (001)**, the receiver concludes that the frame has been **corrupted during transmission** and **discards it**, requesting retransmission.

> **Yes — the single-bit error in the last bit IS detected.**

**Reason:** a generator polynomial with more than one term and a non-zero constant term (x⁰ = 1) detects **all single-bit errors**. Here G(x) = x³ + x² + 1 satisfies this condition, so every single-bit error, regardless of position, produces a non-zero syndrome.

---

## 9. TCP/IP Protocol Architecture (in detail)

The **TCP/IP protocol suite** (Internet Protocol Suite) is the de-facto standard model on which the Internet is built. It was developed before the OSI model and is organised as a **hierarchical, layered set of interactive modules**, each providing a specific functionality. Layers are relatively independent — each can be modified without disturbing the others.

The original model had **four** layers; it is commonly shown today as **five** layers, mapping to the seven OSI layers as below.

```
   OSI Model                     TCP/IP Model                Protocols / Units
 ┌──────────────┐            ┌──────────────────┐
 │ Application  │            │                  │      HTTP, FTP, SMTP, DNS,
 ├──────────────┤            │   Application    │      SNMP, TELNET, DHCP
 │ Presentation │  ───────►  │                  │      → MESSAGE
 ├──────────────┤            │                  │
 │   Session    │            │                  │
 ├──────────────┤            ├──────────────────┤
 │  Transport   │  ───────►  │    Transport     │      TCP, UDP, SCTP
 ├──────────────┤            │                  │      → SEGMENT / DATAGRAM
 │   Network    │  ───────►  ├──────────────────┤
 │              │            │ Network/Internet │      IP, ICMP, IGMP, ARP, RARP
 ├──────────────┤            ├──────────────────┤      → PACKET / DATAGRAM
 │  Data Link   │  ───────►  │    Data Link     │      Ethernet, PPP, HDLC, Wi-Fi
 ├──────────────┤            ├──────────────────┤      → FRAME
 │   Physical   │  ───────►  │     Physical     │      Coax, Fibre, Radio, DSL
 └──────────────┘            └──────────────────┘      → BITS
```

### Layer-by-layer description

**1. Physical Layer**
- Transmits individual **bits** over the physical medium.
- Defines mechanical/electrical specifications, voltage levels, data rate, bit synchronisation, transmission mode (simplex/half/full duplex), and physical topology.
- Devices: hubs, repeaters, cables, connectors.

**2. Data Link Layer**
- Groups bits into **frames**; provides node-to-node (hop-to-hop) delivery.
- Functions: **framing, physical (MAC) addressing, flow control, error control (CRC), access control** to the shared medium.
- Protocols: Ethernet (IEEE 802.3), Wi-Fi (802.11), PPP, HDLC.
- Devices: switches, bridges, NICs.

**3. Network / Internet Layer**
- Provides **host-to-host (source-to-destination) delivery** across multiple networks.
- Core protocol: **IP (IPv4/IPv6)** — connectionless, unreliable, best-effort datagram service; handles logical addressing, routing, fragmentation and reassembly.
- Supporting protocols:
  - **ICMP** – error reporting and query messages (ping, traceroute)
  - **IGMP** – multicast group management
  - **ARP** – maps logical IP address → physical MAC address
  - **RARP** – maps MAC → IP (obsolete, replaced by DHCP)
- Routing protocols: RIP, OSPF, BGP. Device: **router**.

**4. Transport Layer**
- Provides **process-to-process (end-to-end) delivery** using **port numbers**.
- **TCP (Transmission Control Protocol)** — connection-oriented, reliable, byte-stream. Provides three-way handshake, sequencing, acknowledgements, retransmission, sliding-window flow control and congestion control. Used by HTTP, FTP, SMTP.
- **UDP (User Datagram Protocol)** — connectionless, unreliable, minimal overhead (8-byte header). Used by DNS, DHCP, TFTP, VoIP, streaming.
- **SCTP** — combines features of both; multi-streaming and multi-homing, used in signalling.

**5. Application Layer**
- Combines the OSI application, presentation and session layers.
- Provides services directly to the user and defines message formats.
- Protocols: **HTTP/HTTPS** (web), **FTP** (file transfer), **SMTP/POP3/IMAP** (e-mail), **DNS** (name resolution), **TELNET/SSH** (remote login), **SNMP** (management), **DHCP** (auto-configuration), **MQTT/CoAP** (IoT).

### Encapsulation
As data travels down the stack each layer adds its own header (and the data-link layer adds a trailer):

```
Application :                     [ DATA ]
Transport   :            [TCP Hdr][ DATA ]                 → Segment
Network     :     [IP Hdr][TCP Hdr][ DATA ]                → Packet
Data Link   : [Frame Hdr][IP Hdr][TCP Hdr][DATA][Trailer]  → Frame
Physical    : 0101110100101110101110100101...              → Bits
```

### Advantages
Open standard and vendor-independent; scalable; supports routing across heterogeneous networks; proven and universally deployed; flexible (many protocols per layer).

### Limitations
Layers are not cleanly separated (application layer merges three OSI layers); no clear distinction between service, interface and protocol; the model was derived *after* the protocols, so it does not generalise well to other stacks.

---

## 10. IEEE WF-IoT Reference Architecture

The **IEEE World Forum on Internet of Things (WF-IoT) Reference Model**, presented in 2014, is a **seven-level** architecture that describes how data flows from physical "things" to business processes and people. Levels are numbered bottom-up; data moving up is *processed and abstracted*, while control commands flow down.

```
 ┌──────────────────────────────────────────────────────┐
 │ Level 7 : Collaboration & Processes (People, Business)│  ← involving people
 ├──────────────────────────────────────────────────────┤
 │ Level 6 : Application  (Reporting, Analytics, Control)│  ← reporting/analytics
 ├──────────────────────────────────────────────────────┤
 │ Level 5 : Data Abstraction (Aggregation & Access)     │  ← data ready for apps
 ├──────────────────────────────────────────────────────┤
 │ Level 4 : Data Accumulation (Storage)                 │  ← data at rest
 ├──────────────────────────────────────────────────────┤
 │ Level 3 : Edge / Fog Computing (Data Element Analysis)│  ← data in motion
 ├──────────────────────────────────────────────────────┤
 │ Level 2 : Connectivity (Communication & Processing)   │
 ├──────────────────────────────────────────────────────┤
 │ Level 1 : Physical Devices & Controllers ("The Things")│
 └──────────────────────────────────────────────────────┘
```

### Level 1 — Physical Devices and Controllers ("The Things")
Sensors, actuators, RFID tags, machines, RTUs, PLCs, smart meters, wearables. They generate raw data and are the endpoints that can be queried and controlled over the network. Characteristics: constrained power, memory and processing; wide diversity of form factors.

### Level 2 — Connectivity
Reliable, timely transmission of data between devices, between the network and Level 3, and across networks. Includes gateways, routers, switches, and protocol translation (Zigbee/BLE/LoRa ↔ IP). Functions: communication between devices/network, network routing, translation, network-level security.

### Level 3 — Edge (Fog) Computing
Converts network data flows into information suitable for storage and higher-level processing — "**data in motion**". Performed as close to the edge as possible. Activities: evaluating and filtering data, formatting, expanding/decoding, distillation/reduction, event generation and alarming. Benefits: reduces bandwidth, provides low-latency local response.

### Level 4 — Data Accumulation
Converts **data in motion into data at rest**. Event-based, real-time streams are captured and stored so applications can access them non-real-time on demand. Functions: converting packets to database tables, transition from event-driven to query-driven, determining whether data is of interest, and reducing data volume.

### Level 5 — Data Abstraction
Renders stored data and its storage usable by applications: aggregating data from multiple formats and sources into a single schema, reconciling differences, normalising/denormalising, indexing, providing consistent semantics and access control. It gives applications a **simple, unified view** of heterogeneous data.

### Level 6 — Application
Interprets information for business purposes. Includes control applications, monitoring/reporting dashboards, business intelligence, analytics and machine learning. The nature of the application varies from simple device monitoring to complex predictive analytics.

### Level 7 — Collaboration and Processes
The IoT becomes useful only when **people and business processes** act on the information. This level covers communication and collaboration — sharing insights across departments, triggering workflows, and enabling business decisions. It may span multiple organisations.

### Significance of the model
- Provides a **common vocabulary** and clear separation of concerns for IoT system design.
- Shows where processing, storage and security must be applied at each level.
- Makes it explicit that **security must be implemented at every level**, not bolted on.
- Helps decide edge-vs-cloud placement of functions.

---

## 11. OT and IT Responsibilities

**IT (Information Technology)** deals with systems that store, process and transmit **information** — data-centric business systems.
**OT (Operational Technology)** deals with hardware and software that **detect or cause changes in physical processes** — machines, valves, motors, turbines, assembly lines.

IoT succeeds only where **IT and OT converge**, since IoT data originates in the OT domain but is analysed and consumed in the IT domain.

### Responsibilities of IT

| Area | IT responsibility |
|---|---|
| Focus | Data, information, transactions |
| Systems | Servers, databases, ERP/CRM, e-mail, cloud, enterprise networks |
| Primary goal | **Confidentiality** of data (C-I-A priority) |
| Uptime | Scheduled downtime acceptable; patches applied frequently |
| Lifecycle | 3–5 years; frequent upgrades |
| Protocols | TCP/IP, HTTP, SNMP, standard Ethernet |
| Security | Firewalls, antivirus, IDS/IPS, encryption, identity & access management, patch management |
| Data handling | Storage, backup, analytics, data governance, compliance (GDPR etc.) |
| Interfaces | Human-facing applications, dashboards, reporting |
| Key metric | Data integrity and throughput |

### Responsibilities of OT

| Area | OT responsibility |
|---|---|
| Focus | Physical processes, machinery, production |
| Systems | SCADA, DCS, PLC, RTU, HMI, industrial sensors and actuators |
| Primary goal | **Availability and safety** (A-I-C priority) |
| Uptime | 24×7×365; downtime is extremely costly/unsafe — patching is rare |
| Lifecycle | 15–30 years; legacy equipment common |
| Protocols | Modbus, PROFIBUS/PROFINET, DNP3, BACnet, OPC-UA, CAN, fieldbus |
| Security | Physical access control, air-gapping/segmentation, safety instrumented systems |
| Data handling | Real-time control loops, deterministic latency, alarm handling |
| Interfaces | Machine-facing control, HMI panels |
| Key metric | Process safety, reliability, real-time response |

### Comparison Summary

| Parameter | IT | OT |
|---|---|---|
| Deals with | Information/data | Physical equipment & processes |
| Security priority | Confidentiality → Integrity → Availability | Availability → Integrity → Confidentiality |
| Timing | Non-real-time, delay tolerable | Hard real-time, deterministic |
| Failure impact | Data loss, financial loss | Equipment damage, injury, loss of life |
| Environment | Air-conditioned data centres | Harsh: heat, vibration, dust, moisture |
| Update cycle | Frequent | Very infrequent |
| Managed by | CIO / IT department | Plant/Engineering/Operations department |

### IT–OT Convergence in IoT
IoT bridges the two: OT devices are connected to IP networks so that IT systems can analyse their data (predictive maintenance, energy optimisation, quality analytics).

**Benefits:** unified data, better decisions, lower cost, remote monitoring.
**Challenges:** OT legacy systems lack authentication/encryption; connecting them expands the attack surface (e.g. Stuxnet); differing cultures, skill sets and priorities; protocol incompatibility. Solutions include IoT gateways for protocol translation, network segmentation (Purdue model / DMZ), and joint IT-OT governance teams.

---

## 12. ISP Address Allocation Problem

**Given:** block **190.100.0.0/16** → 2^(32−16) = **65,536 addresses**
Range: 190.100.0.0 – 190.100.255.255

### Group 1 — 64 customers × 256 addresses each

- Addresses per customer = 256 = 2⁸ → host bits = 8 → prefix = 32 − 8 = **/24**
- Total for group = 64 × 256 = **16,384 addresses**

| Customer | Block (CIDR) | Range |
|---|---|---|
| 1 | 190.100.**0**.0/24 | 190.100.0.0 – 190.100.0.255 |
| 2 | 190.100.**1**.0/24 | 190.100.1.0 – 190.100.1.255 |
| … | … | … |
| 64 | 190.100.**63**.0/24 | 190.100.63.0 – 190.100.63.255 |

**Group 1 occupies 190.100.0.0 → 190.100.63.255** (= 190.100.0.0/18)

### Group 2 — 128 customers × 128 addresses each

- Addresses per customer = 128 = 2⁷ → host bits = 7 → prefix = **/25**
- Total for group = 128 × 128 = **16,384 addresses**
- Starts at the next free address: **190.100.64.0**

| Customer | Block (CIDR) | Range |
|---|---|---|
| 1 | 190.100.64.**0**/25 | 190.100.64.0 – 190.100.64.127 |
| 2 | 190.100.64.**128**/25 | 190.100.64.128 – 190.100.64.255 |
| 3 | 190.100.65.**0**/25 | 190.100.65.0 – 190.100.65.127 |
| … | … | … |
| 128 | 190.100.127.**128**/25 | 190.100.127.128 – 190.100.127.255 |

**Group 2 occupies 190.100.64.0 → 190.100.127.255** (= 190.100.64.0/18)

### Group 3 — 128 customers × 64 addresses each

- Addresses per customer = 64 = 2⁶ → host bits = 6 → prefix = **/26**
- Total for group = 128 × 64 = **8,192 addresses**
- Starts at **190.100.128.0**

| Customer | Block (CIDR) | Range |
|---|---|---|
| 1 | 190.100.128.**0**/26 | 190.100.128.0 – 190.100.128.63 |
| 2 | 190.100.128.**64**/26 | 190.100.128.64 – 190.100.128.127 |
| 3 | 190.100.128.**128**/26 | 190.100.128.128 – 190.100.128.191 |
| 4 | 190.100.128.**192**/26 | 190.100.128.192 – 190.100.128.255 |
| … | … | … |
| 128 | 190.100.159.**192**/26 | 190.100.159.192 – 190.100.159.255 |

**Group 3 occupies 190.100.128.0 → 190.100.159.255** (= 190.100.128.0/19)

### Total Allocated

```
Group 1 :  64 × 256  = 16,384
Group 2 : 128 × 128  = 16,384
Group 3 : 128 ×  64  =  8,192
                       ───────
Total allocated      = 40,960 addresses
```

### Addresses Still Available

```
Available = 65,536 − 40,960 = 24,576 addresses
```

> ### **Remaining addresses = 24,576**
> **Range: 190.100.160.0 → 190.100.255.255**

Expressed in CIDR the reserve is:
- **190.100.160.0/19** → 8,192 addresses (190.100.160.0 – 190.100.191.255)
- **190.100.192.0/18** → 16,384 addresses (190.100.192.0 – 190.100.255.255)
Total = 8,192 + 16,384 = **24,576** ✔

### Allocation Map

```
190.100.0.0   ┌───────────────────────┐
              │ Group 1  (64 × /24)   │ 16,384
190.100.64.0  ├───────────────────────┤
              │ Group 2  (128 × /25)  │ 16,384
190.100.128.0 ├───────────────────────┤
              │ Group 3  (128 × /26)  │  8,192
190.100.160.0 ├───────────────────────┤
              │      AVAILABLE        │ 24,576
190.100.255.255└──────────────────────┘
```

---

## 13. Sliding Window Protocol — Selective Repeat ARQ

### Sliding Window Concept
To overcome the inefficiency of Stop-and-Wait, sliding-window protocols allow the sender to transmit **several frames before receiving an acknowledgement**. Frames are numbered with **sequence numbers** in the range 0 to 2^m − 1 (m bits), and the "window" is an imaginary box covering the frames that may currently be outstanding. As acknowledgements arrive the window **slides** forward.

### Selective Repeat ARQ (SR-ARQ)
In Go-Back-N, one lost frame forces retransmission of that frame *and all frames after it*, which wastes bandwidth on noisy links. **Selective Repeat retransmits only the specific damaged or lost frame.**

### Key Features

1. **Window sizes:** Sender window size = Receiver window size = **2^(m−1)** (half the sequence-number space). With m = 3 bits → sequence numbers 0–7 and window size = 4.
2. **Receiver window > 1:** The receiver can **accept and buffer out-of-order frames**.
3. **Individual acknowledgements:** ACKs are for specific frames (not cumulative as in GBN); a **NAK** may be sent immediately for a detected damaged frame.
4. **Separate timer per frame** at the sender.
5. **Sorting/buffering** is required at the receiver so frames are delivered in order to the network layer.

### Operation

**Sender side**
- Transmits frames while the window is not full; keeps a copy of every unacknowledged frame.
- Starts a timer for each frame sent.
- On receiving an ACK for frame *i*, marks *i* as acknowledged; slides the window only when the frame at the **left edge (base)** is acknowledged.
- On timeout or NAK for frame *i*, **retransmits only frame i**.

**Receiver side**
- Accepts any frame whose sequence number falls inside the receive window, even if out of order, and buffers it.
- Sends an ACK for each correctly received frame.
- Sends a NAK for a frame detected as corrupted/missing.
- Delivers frames to the upper layer **only in order**; slides the window past every contiguous, correctly received frame.

### Flow Diagram (Frame 2 lost)

```
      Sender                                    Receiver
   window = 4, m = 3                        window = 4
        │                                        │
 Frame 0│──────────────────────────────────────► │ deliver 0
        │ ◄─────────────────── ACK 0 ─────────── │
 Frame 1│──────────────────────────────────────► │ deliver 1
        │ ◄─────────────────── ACK 1 ─────────── │
 Frame 2│───────────X (LOST)                     │
 Frame 3│──────────────────────────────────────► │ out of order → BUFFER 3
        │ ◄─────────────────── NAK 2 ─────────── │
 Frame 4│──────────────────────────────────────► │ BUFFER 4
        │                                        │
 Frame 2│ (retransmit ONLY frame 2)              │
   ─────│──────────────────────────────────────► │ deliver 2, 3, 4 in order
        │ ◄─────────────────── ACK 4 ─────────── │
        │  window slides                         │
```

### Why window size ≤ 2^(m−1)
If the window were larger, a duplicate retransmitted frame could fall inside the *new* receive window and be mistaken for a new frame. Limiting both windows to half the sequence space guarantees the old and new windows never overlap.

### Advantages
- **Highest efficiency** of the three ARQ protocols — only the erroneous frame is resent.
- Excellent on **noisy links** and **high bandwidth-delay** paths.
- Fewer retransmissions ⇒ better bandwidth utilisation.

### Disadvantages
- Receiver needs **buffer memory** and sorting logic — more complex hardware.
- **Separate timer per outstanding frame** at the sender.
- More complex to implement than Go-Back-N.

### Comparison

| Feature | Stop-and-Wait | Go-Back-N | Selective Repeat |
|---|---|---|---|
| Sender window | 1 | 2^m − 1 | 2^(m−1) |
| Receiver window | 1 | 1 | 2^(m−1) |
| Out-of-order frames | Discarded | Discarded | Buffered |
| Retransmission | 1 frame | N frames | Only the bad frame |
| ACK type | Individual | Cumulative | Individual + NAK |
| Complexity | Lowest | Medium | Highest |
| Efficiency | Lowest | Medium | Highest |

---

## 14. Hamming Code Generation

**Hamming code** is a linear **error-correcting code** (FEC) developed by R. W. Hamming. It can **detect up to two-bit errors and correct any single-bit error** by adding parity bits at specific positions in the data word.

### Step 1 — Determine the number of redundant (parity) bits

If **m** = number of data bits and **r** = number of parity bits, then r must satisfy:

> **2^r ≥ m + r + 1**

Example: for m = 4 → try r = 3: 2³ = 8 ≥ 4 + 3 + 1 = 8 ✔ → **r = 3**
Total codeword length n = m + r = **7** → this is the Hamming(7,4) code.

### Step 2 — Position the parity bits

Bits are numbered **1 to n from the left**. Parity bits occupy positions that are **powers of 2** (1, 2, 4, 8, 16 …); all other positions carry data bits.

```
Position :  1    2    3    4    5    6    7
Bit      :  p1   p2   d1   p3   d2   d3   d4
```

### Step 3 — Assign coverage of each parity bit

Each parity bit p at position 2^k checks all positions whose binary representation has a 1 in bit-position k:

| Parity bit | Position | Checks positions |
|---|---|---|
| p1 | 1 | 1, 3, 5, 7, 9, 11 … (skip 1, check 1) |
| p2 | 2 | 2, 3, 6, 7, 10, 11 … (skip 2, check 2) |
| p3 | 4 | 4, 5, 6, 7, 12, 13, 14, 15 … (skip 4, check 4) |
| p4 | 8 | 8–15, 24–31 … |

### Step 4 — Compute each parity bit (even parity)

Each parity bit is set so that the total number of 1s in its group is **even** (XOR of the covered data bits).

### Step 5 — Transmit the codeword.

### Step 6 — Error detection & correction at the receiver

Receiver recomputes each parity check to obtain check bits c1, c2, c3 …
The **syndrome** = c₃c₂c₁ (read as a binary number).
- Syndrome = 0 → no error.
- Syndrome = k → the bit at **position k** is in error; **flip it** to correct.

---

### Worked Example — Data = 1011 (m = 4, r = 3)

**Placement** (d1 d2 d3 d4 = 1 0 1 1):

```
Position :  1    2    3    4    5    6    7
Bit      :  p1   p2   1    p3   0    1    1
```

**Calculate p1** (positions 1, 3, 5, 7 → p1, 1, 0, 1):
p1 = 1 ⊕ 0 ⊕ 1 = **0**

**Calculate p2** (positions 2, 3, 6, 7 → p2, 1, 1, 1):
p2 = 1 ⊕ 1 ⊕ 1 = **1**

**Calculate p3** (positions 4, 5, 6, 7 → p3, 0, 1, 1):
p3 = 0 ⊕ 1 ⊕ 1 = **0**

**Transmitted Hamming codeword:**

```
Position :  1  2  3  4  5  6  7
Codeword :  0  1  1  0  0  1  1
```

> ### **Codeword = 0110011**

---

### Error Detection & Correction Example

Suppose **bit 5 is flipped** during transmission.

Received = `0 1 1 0 **1** 1 1` = **0110111**

**Recompute check bits (even parity):**

| Check | Positions | Bits | XOR | Result |
|---|---|---|---|---|
| c1 | 1,3,5,7 | 0,1,1,1 | 0⊕1⊕1⊕1 | **1** |
| c2 | 2,3,6,7 | 1,1,1,1 | 1⊕1⊕1⊕1 | **0** |
| c3 | 4,5,6,7 | 0,1,1,1 | 0⊕1⊕1⊕1 | **1** |

**Syndrome = c3 c2 c1 = 1 0 1 = (5)₁₀**

→ The error is in **position 5**. Flip bit 5 from 1 back to 0.

**Corrected codeword = 0110011** → extract data bits (positions 3,5,6,7) = **1011** ✔ Original data recovered.

### Advantages
- Corrects single-bit errors without retransmission (useful in simplex links, memory ECC, satellite links).
- Simple XOR-based hardware implementation.

### Disadvantages
- Cannot correct burst/multiple-bit errors (an extended Hamming code with an overall parity bit — SECDED — detects two-bit errors but still corrects only one).
- Redundancy overhead: 3 extra bits for 4 data bits (43 %), though the ratio improves for larger m.

---

## 15. Stop-and-Wait ARQ (with flow diagram)

**Stop-and-Wait ARQ (Automatic Repeat reQuest)** is the simplest connection-oriented data-link protocol that provides **both flow control and error control** over a noisy channel.

### Principle
The sender transmits **one frame at a time** and then **stops and waits** for an acknowledgement. It sends the next frame only after the ACK for the previous one arrives. If no ACK arrives before a timer expires, the frame is retransmitted.

### Design Features

1. **Sequence numbers:** frames are numbered **0 and 1 alternately** (1 bit, modulo 2) so the receiver can distinguish a new frame from a duplicate.
2. **Acknowledgement numbers:** ACK 0 / ACK 1 — the ACK number is the sequence number of the **next expected** frame.
3. **Copy retained:** the sender keeps a copy of the last transmitted frame until its ACK arrives.
4. **Timer:** started with every transmission; expiry ⇒ retransmission.
5. **Error detection:** CRC/checksum; a corrupted frame is silently discarded by the receiver, causing a timeout at the sender.

### Cases Handled

| Case | Event | Action |
|---|---|---|
| Normal | Frame + ACK arrive safely | Send next frame |
| Lost/damaged frame | Receiver discards; no ACK | Timer expires → retransmit same frame |
| Lost ACK | Frame received but ACK lost | Timer expires → retransmit; receiver detects duplicate sequence number, **discards frame** but resends the ACK |
| Delayed ACK | ACK arrives after timeout | Duplicate frame discarded by receiver; late ACK discarded by sender |

### Flow Diagram

```
        SENDER                                    RECEIVER
   S = 0 (next seq to send)                    R = 0 (next expected)
          │                                          │
   Start  │───── Frame 0 ──────────────────────────► │  R=0 ✓ accept
   timer  │                                          │  R → 1
          │ ◄──────────────────────── ACK 1 ──────── │
   S → 1  │                                          │
          │───── Frame 1 ──────────────────────────► │  R=1 ✓ accept
          │                                          │  R → 0
          │ ◄──────────────────────── ACK 0 ──────── │
   S → 0  │                                          │
          │───── Frame 0 ───────X  (LOST)            │
          │                                          │
  Time-out│  ⏰                                       │
          │───── Frame 0 (retransmit) ─────────────► │  R=0 ✓ accept
          │                                          │  R → 1
          │ ◄──────────────────────── ACK 1 ──────── │
   S → 1  │                                          │
          │───── Frame 1 ──────────────────────────► │  R=1 ✓ accept, R → 0
          │             (ACK 0 LOST)  X ◄─────────── │
  Time-out│  ⏰                                       │
          │───── Frame 1 (retransmit) ─────────────► │  seq 1 ≠ expected 0
          │                                          │  DUPLICATE → discard
          │ ◄──────────────────────── ACK 0 ──────── │  (resend ACK)
   S → 0  │                                          │
          ▼                                          ▼
```

### Efficiency

```
Total time per frame, T = Tt + 2Tp        (Tt = transmission time, Tp = propagation time)

Efficiency (η) = Tt / (Tt + 2Tp) = 1 / (1 + 2a)     where a = Tp / Tt
```

**Throughput = η × Bandwidth**

*Example:* For the earlier problem — frame = 1000 bits, B = 2 Mbps, RTT = 20 ms:
Tt = 1000/2×10⁶ = 0.5 ms; 2Tp = 20 ms → η = 0.5 / 20.5 ≈ 2.44 % (≈ 2.5 % using the BDP method).

### Advantages
- Extremely simple to implement; minimal buffer (one frame) at both ends.
- Only 1-bit sequence number needed.
- No out-of-order delivery; inherently provides flow control.

### Disadvantages
- **Very low link utilisation** when propagation delay ≫ transmission time (long or fast links).
- Only one frame in flight — bandwidth is idle most of the time.
- Timeout value must be chosen carefully (too small → unnecessary retransmissions; too large → slow recovery).

**Remedy:** use sliding-window protocols — **Go-Back-N ARQ** or **Selective Repeat ARQ**.

---

*End of solution set.*
