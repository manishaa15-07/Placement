# 📘 PART 2: DATA LINK LAYER & NETWORK LAYER

---

# PHASE 5: DATA LINK LAYER (Layer 2)

## 🎯 Learning Objectives
- Understand framing and why it's needed
- Master error detection techniques (CRC, Hamming)
- Learn flow control protocols deeply (Stop-and-Wait, GBN, SR)
- Understand MAC addressing, Ethernet, switching, VLANs

---

## 5.1 Framing

**What:** Breaking a continuous bit stream into manageable units called **frames**.

**Why:** The Physical Layer just sends raw bits — we need boundaries to know where one message ends and another begins.

### Framing Methods

| Method | How It Works | Problem |
|--------|-------------|---------|
| **Character Count** | Frame header contains length | If count corrupted, all subsequent frames misaligned |
| **Byte Stuffing** | Special flag byte marks start/end. If flag appears in data, escape it | Overhead from stuffing |
| **Bit Stuffing** | Flag = `01111110`. After 5 consecutive 1s in data, insert a 0 | Used in HDLC; slight overhead |

**Bit Stuffing Example:**
```
Flag: 01111110
Data:       011111110100
After stuff: 0111110110100  (0 inserted after five 1s)
```

---

## 5.2 Error Detection & Correction

### Types of Errors
- **Single-bit error:** One bit flipped
- **Burst error:** Multiple consecutive bits corrupted

### Parity Check
- **Single parity bit:** Add 1 bit to make total 1s even (even parity) or odd (odd parity)
- **Detects:** Single-bit errors ✅ | **Misses:** Even number of bit errors ❌
- **2D Parity:** Arrange data in matrix, parity for each row AND column — can detect AND correct single-bit errors

### Checksum
1. Divide data into segments of equal size
2. Add all segments using 1's complement arithmetic
3. Take complement of the sum = checksum
4. Receiver adds all segments + checksum; result should be all 1s

### CRC (Cyclic Redundancy Check) ⭐

**Most important error detection method — asked frequently in interviews and exams!**

**How CRC Works:**
1. Sender and receiver agree on a **generator polynomial** (divisor)
2. Append zeros (degree of generator) to data
3. Perform **binary division** (XOR-based, NOT arithmetic division)
4. Remainder = CRC bits, appended to original data
5. Receiver divides received data by same generator — remainder 0 means no error

**Worked Example:**
```
Data:       1101011011
Generator:  10011 (degree 4, so append 4 zeros)

Data + zeros: 11010110110000

Perform XOR division:
11010110110000 ÷ 10011

... (long division with XOR) ...

Remainder:    1110

Transmitted:  1101011011 | 1110
```

**🎤 Interview Q:** *Can CRC correct errors?*
**A:** No — CRC only **detects** errors (and is very good at it, detecting all single-bit, double-bit, odd-number-of-bit errors, and most burst errors). Correction requires retransmission.

### Hamming Code ⭐

**Purpose:** Error **detection AND correction** of single-bit errors.

**Key Rules:**
- Parity bits are placed at positions that are powers of 2: 1, 2, 4, 8, 16...
- Each parity bit covers specific data positions
- Number of parity bits r: `2^r ≥ m + r + 1` (where m = data bits)

**Parity Bit Coverage (Position-based):**
```
Position:  1  2  3  4  5  6  7  8  9  10  11  12
Type:      P1 P2 D  P4 D  D  D  P8 D  D   D   D

P1 checks: 1, 3, 5, 7, 9, 11  (positions with bit 0 set in binary)
P2 checks: 2, 3, 6, 7, 10, 11 (positions with bit 1 set)
P4 checks: 4, 5, 6, 7, 12     (positions with bit 2 set)
P8 checks: 8, 9, 10, 11, 12   (positions with bit 3 set)
```

**Finding Error Position:** XOR all positions where parity fails → gives error position.

**🎤 Interview Q:** *How many parity bits needed for 8 data bits?*
**A:** Using `2^r ≥ 8 + r + 1`: r = 4 (since 2^4 = 16 ≥ 13). Total bits = 12.

---

## 5.3 Flow Control Protocols

**Why:** Receiver may be slower than sender — without flow control, receiver buffers overflow and data is lost.

### Stop-and-Wait

```
Sender                 Receiver
  |── Frame 0 ──────────►|
  |                       |
  |◄──────── ACK 0 ───── |
  |                       |
  |── Frame 1 ──────────►|
  |                       |
  |◄──────── ACK 1 ───── |
```

- Send **one frame**, wait for **ACK**, then send next
- **Simple but inefficient** — link is idle during ACK wait time
- **Efficiency** = `1 / (1 + 2a)` where `a = Propagation delay / Transmission delay`
- Uses **sequence numbers 0 and 1** (1-bit)

### Stop-and-Wait ARQ (with error handling)
- If ACK not received within **timeout**, retransmit
- If duplicate frame received, receiver re-sends ACK but discards frame

### Go-Back-N (GBN) ⭐

```
Window size = 4:

Sender:  [0][1][2][3] → sends all 4
                         Frame 1 lost!
Receiver: ACK 0 ✓, [1 lost], discards 2, discards 3
Sender:   Timeout on 1 → retransmits [1][2][3] (go back to 1)
```

- **Sender window size:** Up to `2^n - 1` (where n = sequence number bits)
- **Receiver window size:** Always **1**
- **On error:** Sender goes back to the lost frame and retransmits it AND all subsequent frames
- **Receiver discards** all out-of-order frames
- Uses **cumulative ACKs** (ACK n means all frames up to n received)
- **Efficiency** = `W / (1 + 2a)` where W = window size (if W ≤ 1+2a)

### Selective Repeat (SR) ⭐

```
Window size = 4:

Sender:  [0][1][2][3] → sends all 4
                         Frame 1 lost!
Receiver: ACK 0 ✓, buffers [2] ✓, buffers [3] ✓, NAK 1
Sender:   Retransmits only [1]
Receiver: Receives [1], delivers [1][2][3] in order
```

- **Sender window size:** Up to `2^(n-1)`
- **Receiver window size:** Same as sender window `2^(n-1)`
- **On error:** Only the **specific lost frame** is retransmitted
- **Receiver buffers** out-of-order frames
- Uses **individual ACKs** (each frame acknowledged separately)
- More efficient but requires more receiver buffer

### GBN vs SR — Critical Comparison

| Feature | Go-Back-N | Selective Repeat |
|---------|-----------|-----------------|
| **Sender window** | 2^n - 1 | 2^(n-1) |
| **Receiver window** | 1 | 2^(n-1) |
| **Retransmission** | All from lost frame onward | Only the lost frame |
| **Receiver buffering** | No (discards out-of-order) | Yes (buffers out-of-order) |
| **ACK type** | Cumulative | Individual |
| **Efficiency** | Lower (wastes bandwidth) | Higher |
| **Complexity** | Simpler | More complex |
| **Best when** | Low error rate | High error rate |

**🎤 Interview Q:** *Why can't GBN sender window be 2^n?*
**A:** With window = 2^n, if all ACKs are lost, the receiver can't distinguish between new frames and retransmitted frames (ambiguity between seq 0 of next round vs retransmitted seq 0).

---

## 5.4 MAC (Media Access Control) Addressing

- **MAC address:** 48-bit (6 bytes) hardware address burned into NIC
- **Format:** `AA:BB:CC:DD:EE:FF` (hex)
- **First 3 bytes:** OUI (Organizationally Unique Identifier) — identifies manufacturer
- **Last 3 bytes:** Device-specific
- **Broadcast MAC:** `FF:FF:FF:FF:FF:FF`
- **Scope:** Works within a single LAN/subnet (Layer 2)

**🎤 Interview Q:** *Can two devices have the same MAC address?*
**A:** They shouldn't (globally unique), but MAC spoofing is possible. Also, VMs may use software-assigned MACs. Same MAC on same network segment causes conflicts.

---

## 5.5 MAC Protocols (Multiple Access)

### Channel Partitioning
- **TDMA:** Time slots assigned to each node
- **FDMA:** Frequency bands assigned to each node
- **CDMA:** Unique codes for simultaneous transmission

### Random Access
- **ALOHA:** Transmit whenever ready; if collision, wait random time and retry
  - Pure ALOHA efficiency: **18.4%** (1/2e)
  - Slotted ALOHA efficiency: **36.8%** (1/e)
- **CSMA (Carrier Sense Multiple Access):** Listen before transmitting
  - **CSMA/CD (Collision Detection):** Used in wired Ethernet — detect collision and stop
  - **CSMA/CA (Collision Avoidance):** Used in Wi-Fi — avoid collision using RTS/CTS

### Controlled Access
- **Polling:** Central controller polls each node
- **Token Passing:** Token circulates; only token holder can transmit

---

## 5.6 Ethernet (IEEE 802.3)

- Most widely used LAN technology
- Uses **CSMA/CD** for access control
- Frame format:

```
┌──────────┬───────┬──────┬──────┬──────┬──────┬─────┬─────┐
│ Preamble │  SFD  │ Dst  │ Src  │ Type │ Data │ Pad │ FCS │
│  7 bytes │1 byte │ MAC  │ MAC  │  2B  │46-1500│     │ 4B  │
│          │       │  6B  │  6B  │      │bytes │     │(CRC)│
└──────────┴───────┴──────┴──────┴──────┴──────┴─────┴─────┘
```

- **Minimum frame size:** 64 bytes (for collision detection to work)
- **Maximum frame size:** 1518 bytes
- **MTU (Max Transmission Unit):** 1500 bytes (data portion)

---

## 5.7 Switching (Layer 2)

### Switch vs Hub

| Feature | Hub | Switch |
|---------|-----|--------|
| **Layer** | Physical (L1) | Data Link (L2) |
| **Intelligence** | None (broadcasts everything) | Learns MAC addresses |
| **Collision domain** | Entire network = 1 domain | Each port = separate domain |
| **Broadcast domain** | 1 | 1 (unless VLANs used) |
| **Speed** | Shared bandwidth | Dedicated bandwidth per port |

### How a Switch Works
1. Frame arrives on a port
2. Switch reads **source MAC** → adds to **MAC address table** with port
3. Switch reads **destination MAC** → looks up in table
4. If found: **forward** to specific port
5. If not found: **flood** to all ports (except source)
6. If broadcast: **flood** to all ports

---

## 5.8 VLANs (Virtual LANs)

**What:** Logical segmentation of a switch into multiple broadcast domains.

**Why:** Without VLANs, all devices on a switch are in one broadcast domain → broadcast storms, security concerns, no logical grouping.

**How:**
- **Port-based VLAN:** Assign ports to VLANs (most common)
- **MAC-based VLAN:** Assign based on MAC address
- **Protocol-based VLAN:** Assign based on protocol type

**Trunk Port:** Carries traffic for multiple VLANs between switches using **802.1Q tagging** (adds 4-byte VLAN tag to frame).

**🎤 Interview Q:** *Can devices on different VLANs communicate?*
**A:** Not directly — they need a **Layer 3 device (router or L3 switch)** to route between VLANs. This is called **inter-VLAN routing**.

---

# PHASE 6: NETWORK LAYER (Layer 3)

## 🎯 Learning Objectives
- Master IPv4 and IPv6 addressing
- Perform subnetting calculations confidently
- Understand routing protocols
- Know ARP, ICMP, NAT, and fragmentation

---

## 6.1 IPv4 Addressing

### Structure
- **32-bit** address divided into **4 octets**: `192.168.1.100`
- Two parts: **Network ID** + **Host ID**
- Total addresses: 2^32 = ~4.3 billion

### Classful Addressing

| Class | First Octet Range | Default Mask | Network/Host Bits | Purpose |
|-------|-------------------|-------------|-------------------|---------|
| **A** | 1-126 | 255.0.0.0 (/8) | 8 net / 24 host | Large organizations |
| **B** | 128-191 | 255.255.0.0 (/16) | 16 net / 16 host | Medium organizations |
| **C** | 192-223 | 255.255.255.0 (/24) | 24 net / 8 host | Small organizations |
| **D** | 224-239 | — | — | Multicast |
| **E** | 240-255 | — | — | Reserved/experimental |

**Special Addresses:**
- `127.0.0.1` — Loopback (localhost)
- `0.0.0.0` — Default/unknown
- `255.255.255.255` — Broadcast
- `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16` — Private (RFC 1918)

---

## 6.2 Subnetting ⭐⭐⭐

**The most frequently tested topic in CN interviews!**

### Core Concept
Subnetting = Borrowing bits from the **host portion** to create **sub-networks**.

### Key Formulas
- **Number of subnets** = 2^(borrowed bits)
- **Hosts per subnet** = 2^(remaining host bits) - 2 (subtract network & broadcast)
- **Block size** = 256 - subnet mask octet value

### Worked Example

**Problem:** Subnet `192.168.1.0/24` into 4 subnets.

```
Step 1: Need 4 subnets → 2^n = 4 → borrow 2 bits
Step 2: New mask = /24 + 2 = /26 = 255.255.255.192
Step 3: Block size = 256 - 192 = 64
Step 4: Subnets:

| Subnet | Network Address | First Host | Last Host | Broadcast |
|--------|----------------|------------|-----------|-----------|
| 1 | 192.168.1.0    | 192.168.1.1   | 192.168.1.62  | 192.168.1.63  |
| 2 | 192.168.1.64   | 192.168.1.65  | 192.168.1.126 | 192.168.1.127 |
| 3 | 192.168.1.128  | 192.168.1.129 | 192.168.1.190 | 192.168.1.191 |
| 4 | 192.168.1.192  | 192.168.1.193 | 192.168.1.254 | 192.168.1.255 |

Hosts per subnet: 2^6 - 2 = 62
```

### CIDR (Classless Inter-Domain Routing)
- Notation: `IP/prefix_length` (e.g., `10.0.0.0/12`)
- Replaces classful addressing
- Allows arbitrary prefix lengths → more efficient allocation
- Enables **route aggregation** (supernetting)

**🎤 Interview Q:** *Given 192.168.10.0/24, create subnets for departments with 50, 30, 20, and 10 hosts.*

**A:** Use **VLSM (Variable Length Subnet Masking):**
```
50 hosts → need 2^6=64 → /26 → 192.168.10.0/26 (62 usable hosts)
30 hosts → need 2^5=32 → /27 → 192.168.10.64/27 (30 usable hosts)
20 hosts → need 2^5=32 → /27 → 192.168.10.96/27 (30 usable hosts)
10 hosts → need 2^4=16 → /28 → 192.168.10.128/28 (14 usable hosts)
```

---

## 6.3 IPv6

| Feature | IPv4 | IPv6 |
|---------|------|------|
| **Address size** | 32-bit | 128-bit |
| **Notation** | Dotted decimal | Hexadecimal colon-separated |
| **Example** | 192.168.1.1 | 2001:0db8:85a3:0000:0000:8a2e:0370:7334 |
| **Address space** | ~4.3 billion | ~3.4 × 10^38 |
| **Header** | Variable (20-60 bytes) | Fixed 40 bytes |
| **Checksum** | Yes | No (removed for speed) |
| **NAT** | Required | Not needed |
| **IPSec** | Optional | Built-in |
| **Fragmentation** | Routers can fragment | Only source can fragment |
| **Broadcast** | Yes | No (uses multicast) |

**IPv6 address shortening rules:**
1. Remove leading zeros: `0db8` → `db8`
2. Replace consecutive groups of zeros with `::` (only once): `2001:db8::8a2e:370:7334`

---

## 6.4 Routing

### Types of Routing
- **Static:** Manually configured routes (small networks)
- **Dynamic:** Routes learned automatically via protocols
- **Default:** Catch-all route for unknown destinations (`0.0.0.0/0`)

### Distance Vector Routing (RIP)

| Feature | Detail |
|---------|--------|
| **Algorithm** | Bellman-Ford |
| **Metric** | Hop count (max 15, 16 = unreachable) |
| **Updates** | Periodic (every 30 seconds), shares entire routing table |
| **Convergence** | Slow |
| **Problem** | Count-to-infinity |
| **Solutions** | Split horizon, poison reverse, hold-down timers |
| **Use** | Small networks |

### Link State Routing (OSPF)

| Feature | Detail |
|---------|--------|
| **Algorithm** | Dijkstra's shortest path |
| **Metric** | Cost (based on bandwidth) |
| **Updates** | Event-triggered, shares only changes (LSAs) |
| **Convergence** | Fast |
| **Hierarchy** | Uses areas (Area 0 = backbone) |
| **Use** | Large enterprise networks |

### Path Vector Routing (BGP)

| Feature | Detail |
|---------|--------|
| **Type** | Exterior Gateway Protocol (EGP) |
| **Use** | Between Autonomous Systems (ISPs) |
| **Algorithm** | Path vector (tracks AS path) |
| **Protocol** | Runs on TCP port 179 |
| **Decision** | Based on policies, AS path length, etc. |
| **Scale** | Routes the entire internet (~900K+ routes) |

**🎤 Interview Q:** *What is the count-to-infinity problem?*
**A:** In distance vector routing, when a link goes down, routers keep incrementing hop counts through each other in a loop, slowly counting to infinity. Solutions: split horizon, poison reverse, defining infinity as 16.

---

## 6.5 ARP (Address Resolution Protocol)

**Purpose:** Maps **IP address → MAC address** within a LAN.

```
Flow:
1. Host A wants to send to 192.168.1.5 but doesn't know its MAC
2. Host A broadcasts ARP Request: "Who has 192.168.1.5?"
3. All devices receive it; only 192.168.1.5 responds
4. Host B (192.168.1.5) sends ARP Reply: "I'm at AA:BB:CC:DD:EE:FF"
5. Host A caches this mapping in its ARP table
6. Host A sends the frame with the correct MAC
```

**RARP:** Reverse ARP — maps MAC → IP (obsolete, replaced by DHCP)

**🎤 Interview Q:** *What's an ARP spoofing attack?*
**A:** Attacker sends fake ARP replies, mapping the victim's IP to the attacker's MAC. This enables man-in-the-middle attacks by redirecting traffic through the attacker.

---

## 6.6 ICMP (Internet Control Message Protocol)

**Purpose:** Error reporting and diagnostic tool for IP.

| Type | Message | Use |
|------|---------|-----|
| 0 | Echo Reply | Ping response |
| 3 | Destination Unreachable | Can't reach host/port |
| 5 | Redirect | Better route available |
| 8 | Echo Request | Ping request |
| 11 | Time Exceeded | TTL expired (used by traceroute) |

**Ping:** Uses ICMP Echo Request (type 8) and Echo Reply (type 0)
**Traceroute:** Sends packets with incrementing TTL (1, 2, 3...) to discover each hop via ICMP Time Exceeded messages.

---

## 6.7 NAT (Network Address Translation)

**Why:** IPv4 addresses are exhausted. NAT allows multiple private IP devices to share one public IP.

```
Private Network              NAT Router              Internet
┌──────────────┐         ┌───────────┐         ┌──────────┐
│ 192.168.1.10 │─────────│ NAT Table │─────────│  Server  │
│ 192.168.1.11 │─────────│ Public IP:│─────────│203.0.0.1 │
│ 192.168.1.12 │─────────│ 52.1.2.3  │─────────│          │
└──────────────┘         └───────────┘         └──────────┘
```

### NAT Types

| Type | Description |
|------|-------------|
| **Static NAT** | 1:1 mapping (private:public) — for servers |
| **Dynamic NAT** | Pool of public IPs assigned on demand |
| **PAT/NAT Overload** | Many private IPs → 1 public IP using port numbers (most common!) |

**PAT Example:**
```
192.168.1.10:5001 → 52.1.2.3:40001
192.168.1.11:5001 → 52.1.2.3:40002
192.168.1.12:8080 → 52.1.2.3:40003
```

**🎤 Interview Q:** *How does the router know which internal device to send the response to?*
**A:** PAT maintains a **NAT translation table** mapping internal (IP:port) to external (IP:port). When a response comes back to the external port, it looks up the table to find the correct internal device.

---

## 6.8 IP Fragmentation

**Why:** Different networks have different **MTU (Maximum Transmission Unit)**. If a packet is larger than the next link's MTU, it must be fragmented.

**Key Fields in IP Header:**
- **Identification:** All fragments of the same packet share this
- **Flags:**
  - DF (Don't Fragment): If set and fragmentation needed → packet dropped + ICMP error
  - MF (More Fragments): 1 for all fragments except the last
- **Fragment Offset:** Position of this fragment (in units of 8 bytes)

**Example:** 4000-byte packet, MTU = 1500
```
Fragment 1: Offset=0,    Data=1480 bytes, MF=1
Fragment 2: Offset=185,  Data=1480 bytes, MF=1  (185 × 8 = 1480)
Fragment 3: Offset=370,  Data=1020 bytes, MF=0  (last fragment)
```

**🎤 Interview Q:** *In IPv4, who reassembles fragments?*
**A:** Only the **destination host** reassembles — intermediate routers never reassemble (they may further fragment). This is different from IPv6 where only the source fragments.

---

## 📝 Phase 5-6 Revision Box

> **Quick Recall:**
> - CRC detects errors, Hamming corrects single-bit errors
> - Hamming: parity bits at positions 2^n (1, 2, 4, 8...)
> - GBN: sender window = 2^n - 1, receiver = 1, retransmit all
> - SR: sender = receiver window = 2^(n-1), retransmit only lost
> - Ethernet min frame = 64 bytes, MTU = 1500 bytes
> - MAC = 48-bit, IP = 32-bit (v4) / 128-bit (v6)
> - Subnetting: hosts = 2^(host bits) - 2, block = 256 - mask
> - ARP: IP → MAC | RARP: MAC → IP
> - NAT/PAT: Many private IPs share one public IP using ports
> - OSPF = Dijkstra = fast convergence | RIP = Bellman-Ford = slow
