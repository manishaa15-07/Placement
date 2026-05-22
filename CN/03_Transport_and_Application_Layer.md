# 📘 PART 3: TRANSPORT LAYER & APPLICATION LAYER

---

# PHASE 7: TRANSPORT LAYER (Layer 4)

## 🎯 Learning Objectives
- Master TCP internals: handshake, reliability, flow & congestion control
- Understand UDP and when to use it
- Be able to answer any TCP vs UDP question with depth
- Know TCP congestion algorithms (interview favorite!)

---

## 7.1 Transport Layer Overview

**Purpose:** Provides **end-to-end** (process-to-process) communication between applications.

**Key Responsibilities:**
- Segmentation & reassembly
- Multiplexing/demultiplexing (using port numbers)
- Error control
- Flow control
- Connection management

### Port Numbers

| Range | Name | Examples |
|-------|------|---------|
| 0-1023 | **Well-known** | HTTP(80), HTTPS(443), DNS(53), FTP(21), SSH(22), SMTP(25) |
| 1024-49151 | **Registered** | MySQL(3306), PostgreSQL(5432), MongoDB(27017) |
| 49152-65535 | **Dynamic/Private** | Assigned by OS for client connections |

**Socket = IP Address + Port Number** (e.g., `192.168.1.5:8080`)

---

## 7.2 TCP (Transmission Control Protocol) ⭐⭐⭐

### Characteristics
- **Connection-oriented** — must establish connection first
- **Reliable** — guaranteed delivery
- **Ordered** — data arrives in sequence
- **Full-duplex** — bidirectional simultaneous communication
- **Flow-controlled** — sender doesn't overwhelm receiver
- **Congestion-controlled** — sender adjusts for network capacity
- **Byte-stream** — no message boundaries

### TCP Header (20 bytes minimum)

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
┌───────────────────────────┬───────────────────────────────────┐
│      Source Port (16)     │     Destination Port (16)         │
├───────────────────────────┴───────────────────────────────────┤
│                    Sequence Number (32)                        │
├───────────────────────────────────────────────────────────────┤
│                 Acknowledgment Number (32)                     │
├──────┬────────┬───────────┬───────────────────────────────────┤
│HL(4) │Resv(3) │ Flags(9)  │         Window Size (16)          │
├──────┴────────┴───────────┼───────────────────────────────────┤
│     Checksum (16)         │      Urgent Pointer (16)          │
├───────────────────────────┴───────────────────────────────────┤
│                   Options (variable)                          │
└───────────────────────────────────────────────────────────────┘
```

**Key Flags:** SYN, ACK, FIN, RST, PSH, URG

---

### TCP 3-Way Handshake ⭐⭐⭐

**Purpose:** Establish a reliable connection and synchronize sequence numbers.

```
    Client                          Server
      │                               │
      │──── SYN (seq=x) ────────────►│  Step 1: Client says "Let's talk"
      │                               │
      │◄─── SYN+ACK (seq=y, ack=x+1) │  Step 2: Server says "OK, let's talk"
      │                               │
      │──── ACK (seq=x+1, ack=y+1) ──►│  Step 3: Client says "Great!"
      │                               │
      │       CONNECTION ESTABLISHED   │
```

**Why 3-way? Why not 2-way?**
- 2-way handshake can cause **half-open connections** from delayed duplicate SYNs
- 3-way ensures BOTH sides confirm and synchronize sequence numbers
- Prevents old SYN packets from creating ghost connections

**Real-World Analogy:**
- "Hey, can you hear me?" (SYN)
- "Yes, I can hear you. Can you hear me?" (SYN+ACK)
- "Yes, I can hear you too. Let's talk." (ACK)

---

### TCP 4-Way Termination ⭐⭐

```
    Client                          Server
      │                               │
      │──── FIN (seq=u) ────────────►│  Step 1: Client says "I'm done sending"
      │                               │
      │◄─── ACK (ack=u+1) ───────── │  Step 2: Server says "OK, noted"
      │                               │  (Server may still send data)
      │◄─── FIN (seq=v) ────────────│  Step 3: Server says "I'm done too"
      │                               │
      │──── ACK (ack=v+1) ──────────►│  Step 4: Client says "OK, goodbye"
      │                               │
      │     TIME_WAIT (2×MSL)         │
      │       CONNECTION CLOSED        │
```

**Why 4 steps and not 3?**
- Connection is full-duplex — each direction must be closed independently
- Server may still have data to send after receiving client's FIN

**TIME_WAIT State:**
- Client waits **2 × MSL** (Maximum Segment Lifetime, typically 60 seconds) after sending final ACK
- **Why?** To handle lost final ACK (server would retransmit FIN) and to let old packets expire

**🎤 Interview Q:** *What happens if the final ACK is lost?*
**A:** The server retransmits its FIN. The client, still in TIME_WAIT, receives it and re-sends the ACK, restarting the TIME_WAIT timer.

---

### TCP Reliability Mechanisms

| Mechanism | How |
|-----------|-----|
| **Sequence numbers** | Every byte is numbered; receiver knows exact order |
| **Acknowledgments** | Receiver confirms received bytes |
| **Retransmission** | Timeout-based or fast retransmit (3 duplicate ACKs) |
| **Checksum** | Detects corrupted segments |
| **Duplicate detection** | Sequence numbers identify duplicates |
| **Ordering** | Receiver reorders using sequence numbers |

---

### TCP Flow Control (Sliding Window) ⭐

**Purpose:** Prevent sender from overwhelming a slow receiver.

**How:** Receiver advertises a **window size** (rwnd) — the amount of buffer space available.

```
Sender:     [Sent & ACKed] [Sent, not ACKed] [Can send] [Cannot send]
            ◄──────────────►◄────────────────►◄────────►◄───────────►
                              Sender Window (= rwnd)
```

- Sender can send up to `rwnd` bytes without waiting for ACK
- As receiver processes data, it sends ACKs with updated window size
- **Window = 0:** Receiver says "STOP! Buffer full!" (sender enters persist mode, sends probe segments)

**Analogy:** Like a conveyor belt with a speed dial controlled by the receiver.

---

### TCP Congestion Control ⭐⭐⭐

**Why:** Flow control protects the receiver; **congestion control protects the network**.

**Key Variable:** `cwnd` (congestion window) — maintained by the sender.

**Effective window = min(cwnd, rwnd)**

#### Algorithm 1: Slow Start

```
cwnd starts at 1 MSS (Maximum Segment Size)
For each ACK received: cwnd = cwnd + 1 MSS
Effect: cwnd doubles every RTT (exponential growth!)

cwnd: 1 → 2 → 4 → 8 → 16 → ... until threshold (ssthresh)
```

#### Algorithm 2: Congestion Avoidance (AIMD)

```
When cwnd ≥ ssthresh:
  Additive Increase: cwnd += 1 MSS per RTT (linear growth)
  
On packet loss (timeout):
  Multiplicative Decrease: ssthresh = cwnd / 2, cwnd = 1 MSS
  Go back to Slow Start
```

#### Algorithm 3: Fast Retransmit

```
If sender receives 3 duplicate ACKs:
  → Retransmit the lost segment immediately
  → Don't wait for timeout (faster recovery)
```

#### Algorithm 4: Fast Recovery

```
On 3 duplicate ACKs:
  ssthresh = cwnd / 2
  cwnd = ssthresh + 3 MSS  (don't reset to 1!)
  Enter Congestion Avoidance directly (skip Slow Start)
```

### TCP Congestion Control — Visual Timeline

```
cwnd
 ↑
 │            ╱╲ ssthresh hit
 │           ╱  ╲  ← Multiplicative Decrease (timeout)
 │          ╱    ╲
 │         ╱      ╲
 │    ____╱        ╲_____ Congestion Avoidance (linear)
 │   ╱ Slow Start         ╲
 │  ╱ (exponential)         ╲
 │ ╱                         ╲
 │╱                           ╲
 └─────────────────────────────── Time →
```

### TCP Variants Summary

| Variant | Mechanism | Key Feature |
|---------|-----------|-------------|
| **TCP Tahoe** | Slow Start + AIMD | Reset cwnd to 1 on any loss |
| **TCP Reno** | + Fast Retransmit/Recovery | cwnd to half on 3 dup ACKs |
| **TCP NewReno** | Improved Fast Recovery | Better handling of multiple losses |
| **TCP CUBIC** | Cubic function for cwnd | Default in Linux, optimized for high BDP |
| **TCP BBR** | Bottleneck Bandwidth & RTT | Google's algorithm, model-based |

---

## 7.3 UDP (User Datagram Protocol)

### Characteristics
- **Connectionless** — no handshake
- **Unreliable** — no guaranteed delivery
- **Unordered** — packets may arrive out of order
- **No flow/congestion control**
- **Lightweight** — 8-byte header only
- **Message-oriented** — preserves message boundaries

### UDP Header (8 bytes)

```
┌─────────────────┬─────────────────┐
│  Source Port (16)│  Dest Port (16) │
├─────────────────┼─────────────────┤
│   Length (16)    │  Checksum (16)  │
└─────────────────┴─────────────────┘
```

### When to Use UDP
- Real-time applications (video streaming, VoIP, gaming)
- DNS queries (small, single request-response)
- DHCP (client doesn't have IP yet)
- IoT (lightweight devices)
- When speed > reliability

---

## 7.4 TCP vs UDP — Master Comparison ⭐⭐⭐

| Feature | TCP | UDP |
|---------|-----|-----|
| **Connection** | Connection-oriented (3-way handshake) | Connectionless |
| **Reliability** | Guaranteed delivery | Best-effort, no guarantee |
| **Ordering** | Ordered delivery | No ordering |
| **Speed** | Slower (overhead) | Faster (minimal overhead) |
| **Header size** | 20-60 bytes | 8 bytes |
| **Flow control** | Yes (sliding window) | No |
| **Congestion control** | Yes (AIMD, slow start) | No |
| **Broadcast/Multicast** | No | Yes |
| **Use cases** | HTTP, FTP, Email, SSH | DNS, DHCP, Streaming, Gaming, VoIP |
| **Analogy** | Registered mail (tracked) | Postcards (hope it arrives) |

**🎤 Interview Q:** *Can you build reliable communication over UDP?*
**A:** Yes! You implement reliability at the application layer — add sequence numbers, ACKs, retransmission logic yourself. QUIC protocol (used in HTTP/3) does exactly this — reliable transport over UDP.

---

# PHASE 8: APPLICATION LAYER

## 🎯 Learning Objectives
- Understand major application protocols deeply
- Know how DNS, HTTP, DHCP work internally
- Be ready for "what happens when you type a URL" questions

---

## 8.1 HTTP (HyperText Transfer Protocol)

### HTTP Basics
- **Port:** 80 (HTTP), 443 (HTTPS)
- **Model:** Client-server, request-response
- **Stateless:** Each request is independent (cookies/sessions add state)
- **Transport:** TCP

### HTTP Methods

| Method | Purpose | Idempotent? | Safe? |
|--------|---------|-------------|-------|
| **GET** | Retrieve data | Yes | Yes |
| **POST** | Create/submit data | No | No |
| **PUT** | Update/replace data | Yes | No |
| **PATCH** | Partial update | No | No |
| **DELETE** | Remove data | Yes | No |
| **HEAD** | Like GET but no body | Yes | Yes |
| **OPTIONS** | Check available methods | Yes | Yes |

### HTTP Status Codes

| Range | Category | Key Codes |
|-------|----------|-----------|
| **1xx** | Informational | 100 Continue |
| **2xx** | Success | 200 OK, 201 Created, 204 No Content |
| **3xx** | Redirection | 301 Moved Permanently, 302 Found, 304 Not Modified |
| **4xx** | Client Error | 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 429 Too Many Requests |
| **5xx** | Server Error | 500 Internal Error, 502 Bad Gateway, 503 Service Unavailable |

### HTTP Version Evolution

| Version | Key Features |
|---------|-------------|
| **HTTP/1.0** | New TCP connection per request (slow!) |
| **HTTP/1.1** | Persistent connections (keep-alive), pipelining, chunked transfer |
| **HTTP/2** | Binary framing, multiplexing (multiple streams on 1 connection), header compression (HPACK), server push |
| **HTTP/3** | Uses **QUIC** (UDP-based), eliminates head-of-line blocking, faster handshake (0-RTT) |

**🎤 Interview Q:** *What is HTTP/2 multiplexing?*
**A:** HTTP/2 allows multiple requests/responses to be interleaved on the same TCP connection simultaneously. Each is a separate "stream." This eliminates HTTP/1.1's head-of-line blocking where one slow response delays all others.

---

## 8.2 HTTPS

- HTTP + **TLS/SSL** encryption
- Uses **port 443**
- Provides: **Confidentiality** (encryption), **Integrity** (data not tampered), **Authentication** (server identity verified)

### TLS Handshake (simplified)
```
Client                              Server
  │── ClientHello (supported ciphers) ──►│
  │◄── ServerHello + Certificate ────────│
  │    (Client verifies cert via CA)      │
  │── Key Exchange (pre-master secret) ──►│
  │    Both derive session keys           │
  │── Finished ──────────────────────────►│
  │◄── Finished ─────────────────────────│
  │      Encrypted communication begins   │
```

---

## 8.3 DNS (Domain Name System) ⭐⭐⭐

**Purpose:** Translates domain names to IP addresses (the internet's phone book).

### DNS Resolution Flow

```
User types: www.example.com

Step 1: Browser cache → OS cache → Router cache → ISP DNS cache
        (If found, done! This is FASTER)

Step 2: If not cached, start RECURSIVE resolution:

User's PC ──► Local DNS Server (Recursive Resolver)
                    │
                    ├──► Root DNS Server (.)
                    │    "I don't know, try .com server"
                    │
                    ├──► TLD DNS Server (.com)
                    │    "I don't know, try example.com's nameserver"
                    │
                    └──► Authoritative DNS Server (example.com)
                         "www.example.com = 93.184.216.34"

Step 3: Local DNS caches the result and returns to user
```

### DNS Record Types

| Record | Purpose | Example |
|--------|---------|---------|
| **A** | Domain → IPv4 | example.com → 93.184.216.34 |
| **AAAA** | Domain → IPv6 | example.com → 2606:2800:220:1:... |
| **CNAME** | Alias to another domain | www.example.com → example.com |
| **MX** | Mail server | example.com → mail.example.com |
| **NS** | Nameserver | example.com → ns1.example.com |
| **TXT** | Text info (SPF, verification) | Various |
| **SOA** | Start of Authority | Zone metadata |
| **PTR** | IP → Domain (reverse DNS) | 93.184.216.34 → example.com |

### DNS Query Types
- **Recursive:** Client asks resolver to get the final answer (most common)
- **Iterative:** Server gives best referral; client queries next server itself

**🎤 Interview Q:** *What's the difference between recursive and iterative DNS?*
**A:** In recursive, the DNS resolver does all the work and returns the final answer. In iterative, the resolver returns a referral to the next server, and the client must query that server itself. End-user queries are typically recursive; inter-server queries are iterative.

---

## 8.4 DHCP (Dynamic Host Configuration Protocol)

**Purpose:** Automatically assigns IP addresses and network configuration to devices.

### DORA Process ⭐

```
Client                              DHCP Server
  │── DISCOVER (broadcast) ───────────►│  "Anyone have an IP for me?"
  │◄── OFFER ──────────────────────────│  "How about 192.168.1.50?"
  │── REQUEST (broadcast) ────────────►│  "Yes, I'll take that one"
  │◄── ACKNOWLEDGE ────────────────────│  "It's yours for 24 hours"
```

**Key Points:**
- Uses UDP (ports 67 server, 68 client)
- Client starts with no IP, so uses broadcast
- Assigns: IP address, subnet mask, default gateway, DNS servers
- **Lease time:** IP is temporary; must be renewed
- **Why UDP?** Client doesn't have an IP yet, can't establish TCP connection

---

## 8.5 Other Application Layer Protocols

### Email Protocols

| Protocol | Purpose | Port | Transport |
|----------|---------|------|-----------|
| **SMTP** | Send email | 25 (587 with TLS) | TCP |
| **POP3** | Download email (deletes from server) | 110 (995 TLS) | TCP |
| **IMAP** | Access email (keeps on server) | 143 (993 TLS) | TCP |

**SMTP Flow:** Client → SMTP Server → DNS (MX lookup) → Recipient's SMTP Server → POP3/IMAP → Recipient

### File Transfer

| Protocol | Port | Key Feature |
|----------|------|-------------|
| **FTP** | 20 (data), 21 (control) | Two connections; active vs passive mode |
| **SFTP** | 22 | FTP over SSH — encrypted |
| **TFTP** | 69 | Simple, UDP-based, no authentication |

**FTP Modes:**
- **Active:** Server connects TO client's data port (firewall problems)
- **Passive:** Client connects to server's data port (firewall-friendly)

### Remote Access

| Protocol | Port | Security |
|----------|------|----------|
| **Telnet** | 23 | **None!** Plaintext — NEVER use in production |
| **SSH** | 22 | Encrypted, authenticated — use this instead |

**SSH Features:** Encrypted shell access, file transfer (SCP/SFTP), port forwarding/tunneling, key-based authentication

### WebSockets

- **Purpose:** Full-duplex, persistent communication between client and server
- **Upgrade:** Starts as HTTP, then "upgrades" to WebSocket
- **Use Cases:** Chat apps, live sports scores, stock tickers, collaborative editing, gaming
- **vs HTTP:** HTTP is request-response; WebSocket is bidirectional and persistent

```
HTTP: Client ──request──► Server ──response──► Client (done)
WS:   Client ◄════════════════════════════════► Server (persistent)
```

### REST API Basics

**REST = Representational State Transfer**

**Core Principles:**
1. **Client-Server** — separation of concerns
2. **Stateless** — each request contains all info needed
3. **Cacheable** — responses must state if cacheable
4. **Uniform Interface** — standard HTTP methods + URIs
5. **Layered System** — client can't tell if connected directly to server

**Example:**
```
GET    /api/users          → List all users
GET    /api/users/42       → Get user 42
POST   /api/users          → Create new user
PUT    /api/users/42       → Update user 42
DELETE /api/users/42       → Delete user 42
```

---

## 📝 Phase 7-8 Revision Box

> **Quick Recall:**
> - TCP 3-way: SYN → SYN+ACK → ACK
> - TCP 4-way close: FIN → ACK → FIN → ACK + TIME_WAIT
> - Congestion: Slow Start (exponential) → AIMD (linear) → Fast Retransmit (3 dup ACK)
> - TCP Reno: cwnd halved on 3 dup ACK | TCP Tahoe: cwnd reset to 1
> - HTTP: 80 | HTTPS: 443 | DNS: 53 | FTP: 21/20 | SSH: 22 | SMTP: 25
> - DNS hierarchy: Root → TLD → Authoritative
> - DHCP DORA: Discover → Offer → Request → Acknowledge
> - WebSocket = persistent full-duplex | HTTP = request-response
> - REST = stateless, uniform interface, HTTP methods + URIs
