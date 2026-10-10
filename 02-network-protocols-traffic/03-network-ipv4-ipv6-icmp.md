<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran Wakeel | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 02 — Network Protocols, Packet Dissection & Traffic Engineering
   DOCUMENT: Chapter 03 — Layer 3: Network Layer Architecture & IPv4 Dissection (Part 3.1)
   ========================================================================= -->

# 📡 Chapter 03: Layer 3 — Network Layer (IPv4, IPv6 & Routing) (Part 3.1)

> **IW Cyber Ops Research Vault | Module 02: Network Protocols & Traffic Engineering**  
> *Author: Muhammad Imran Wakeel (@iwcyberops)*  
> *Track: RFC 791 IPv4 Specification, Bit-Level Header Dissection & Checksum Math*

---

## 1. The IPv4 Datagram Architecture (RFC 791)

The **Internet Protocol version 4 (IPv4)** operates at Layer 3 of the OSI model. It is an **unreliable, connectionless, best-effort packet delivery protocol**:
* **Unreliable:** It does not guarantee delivery; packets may be dropped, corrupted, delayed, or duplicated.
* **Connectionless:** Each datagram is handled independently by intermediate routers without pre-allocating an end-to-end path.

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Version|  IHL  |    DSCP   |ECN|         Total Length          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|         Identification        |Flags|     Fragment Offset     |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Time to Live |    Protocol   |        Header Checksum        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                       Source IP Address                       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Destination IP Address                     |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options (0 to 40 Bytes)                    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Padding (Variable 0-3B)                    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
│ ◄──────────────── Base Header: Exactly 20 Bytes ────────────► │
```

---

## 2. Exhaustive Bit-by-Bit Field Dissection

### 1. Version (4 Bits)
* **Value:** `0100` (Decimal **4**).
* **Function:** Identifies the protocol as IPv4. If a router's hardware parser detects any value other than `4` (e.g., `0110` for IPv6), the datagram is rejected.

---

### 2. Internet Header Length — IHL (4 Bits)
* **Measurement:** Expressed in units of **32-bit words (4-byte blocks)**.
* **Minimum Value:** `0101` (Decimal $5 \implies 5 \times 4 = \mathbf{20\text{ Bytes}}$ — Base header without options).
* **Maximum Value:** `1111` (Decimal $15 \implies 15 \times 4 = \mathbf{60\text{ Bytes}}$ — Base header + $40\text{ bytes}$ options).
* ⚠️ **Malformed Packet Bug:** Any packet where $\text{IHL} < 5$ is an illegal malformed datagram dropped immediately by the kernel.

---

### 3. Quality of Service: DSCP (6 Bits) & ECN (2 Bits)
Replaced the legacy RFC 791 "Type of Service (ToS)" byte:
* **DSCP (Differentiated Services Code Point — 6 Bits):** Classifies traffic priority (e.g., Voice, Video, Best Effort) across network routers.
* **ECN (Explicit Congestion Notification — 2 Bits / RFC 3168):**
  * `00` : Non ECN-Capable Transport (`Non-ECT`).
  * `01` / `10` : ECN-Capable Transport (`ECT(1)` / `ECT(0)`).
  * `11` : **Congestion Encountered (`CE`):** A congested router marks these bits to notify endpoints to reduce transmission speed **without dropping the packet**.

---

### 4. Total Length (16 Bits)
* **Measurement:** Total length of the entire IP datagram (**Header + Payload**) in bytes.
* **Maximum Theoretical Size:** $2^{16} - 1 = \mathbf{65,535\text{ Bytes}}$.
* **Minimum Theoretical Size:** $20\text{ Bytes}$ (Header only, zero payload).

---

### 5. Identification (16 Bits)
* **Function:** A unique sequential or randomized integer assigned by the sending host to identify all fragments belonging to a single original unfragmented datagram.

---

### 6. Flags (3 Bits)
Controls fragmentation behavior:
```text
Bit 0: Reserved (Must be 0)
Bit 1: DF (Don't Fragment)  ──> 1 = Routers MUST NOT fragment; drop if > MTU
Bit 2: MF (More Fragments)   ──> 1 = More fragments follow; 0 = Last fragment
```

---

### 7. Fragment Offset (13 Bits)
* **Function:** Indicates the position of the fragmented data relative to the beginning of the original unfragmented payload.
* **Unit of Measurement:** Expressed in units of **8-byte (64-bit) blocks**.
* **Formula:** $\text{Byte Offset} = \text{Fragment Offset Value} \times 8$.

---

### 8. Time-to-Live — TTL (8 Bits)
* **Range:** $0$ to $255$.
* **Function:** Prevents packets from circulating indefinitely in Layer 3 routing loops.
* **Mechanics:** Every intermediate Layer 3 router decrements the TTL by at least **`1`**. If TTL reaches **`0`**, the router drops the packet and transmits an **ICMP Type 11 (Time to Live Exceeded)** message back to the sender (The core mechanism of `traceroute`).

---

### 9. Protocol (8 Bits)
Identifies the encapsulated Layer 4 protocol payload:

| Protocol Value (Hex) | Protocol Name | Operational Function |
| :---: | :--- | :--- |
| **`0x01`** (`1`) | **ICMP** | Internet Control Message Protocol (Ping / Diagnostics) |
| **`0x02`** (`2`) | **IGMP** | Internet Group Management Protocol (Multicast) |
| **`0x06`** (`6`) | **TCP**  | Transmission Control Protocol (Reliable transport) |
| **`0x11`** (`17`)| **UDP**  | User Datagram Protocol (Stateless transport) |
| **`0x29`** (`41`)| **IPv6** | IPv6 encapsulation over IPv4 (6to4 Tunneling) |
| **`0x2F`** (`47`)| **GRE**  | Generic Routing Encapsulation (VPN Tunneling) |
| **`0x32`** (`50`)| **ESP**  | Encapsulating Security Payload (IPsec encryption) |

---

### 10. Header Checksum (16 Bits)
* **Scope:** Covers **ONLY the 20-60 byte IPv4 Header**, NOT the payload!
* **Router Penalty:** Because the TTL field decrements at every single router hop, **every router along a path MUST recompute the IP Header Checksum** before forwarding the packet!

---

### 11. Source & Destination IP Addresses (32 Bits Each)
* **Source IP:** 4 bytes representing the transmitting node's logical address.
* **Destination IP:** 4 bytes representing the intended recipient.

---

### 12. IPv4 Options (0 to 40 Bytes)
Optional diagnostic parameters appended to the base 20-byte header:
* **Record Route (RR):** Routers record their outbound IP addresses directly into the packet header.
* **Strict Source Routing (SSRR):** The sender dictates the exact path of IP hops the packet must traverse.
* **Loose Source Routing (LSRR):** The sender specifies mandatory intermediate hops, but allows dynamic routing between them.

---

## 3. The Mathematical Checksum Engine (Ones' Complement)

The IPv4 Header Checksum algorithm relies on **16-bit Ones' Complement Addition**:

```
 [ Step 1: Divide Header into 16-bit Words ] ──> [ Step 2: Sum all Words ] ──> [ Step 3: Invert Bits (NOT) ]
```

---

### Mathematical Calculation Step-by-Step

1. Set the **Checksum field to `0x0000`**.
2. Divide the entire header into sequential 16-bit (2-byte) integers.
3. Compute the mathematical sum of all 16-bit words.
4. If a carry bit occurs beyond 16 bits (Bit 17), add the carry back into the lowest-order bit (**End-Around Carry**).
5. Perform a bitwise **NOT (One's Complement)** on the final sum. The result is placed into the Checksum field.
6. **Receiver Verification:** The receiving host performs the exact same 16-bit sum over the entire header (including the checksum). If the final result is **`0xFFFF` (or `0x0000` inverted)**, the header is mathematically verified. If non-zero, it is silently discarded!

---

## 4. 🔬 Raw Wire Hex Dissection: Live IPv4 Header

Below is an annotated hex dump of a raw 20-byte IPv4 packet header captured via `tcpdump -xx`:

```text
45 00 00 3c 1a 2b 40 00 40 06 b2 c8 c0 a8 01 0a c0 a8 01 32
```

```
 45 │ 00 │ 00 3c │ 1a 2b │ 40 00 │ 40 │ 06 │ b2 c8 │ c0 a8 01 0a │ c0 a8 01 32
 ───┼────┼───────┼───────┼───────┼────┼────┼───────┼─────────────┼────────────
  │   │     │       │       │      │    │      │           │             └── Dest IP
  │   │     │       │       │      │    │      │           └──────────────── Src IP
  │   │     │       │       │      │    │      └──────────────────────────── Checksum
  │   │     │       │       │      │    └─────────────────────────────────── Protocol
  │   │     │       │       │      └──────────────────────────────────────── TTL (64)
  │   │     │       │       └─────────────────────────────────────────────── Flags & Offset
  │   │     │       └─────────────────────────────────────────────────────── IP ID (0x1a2b)
  │   │     └─────────────────────────────────────────────────────────────── Total Length (60B)
  │   └───────────────────────────────────────────────────────────────────── DSCP / ECN (0)
  └───────────────────────────────────────────────────────────────────────── Version 4, IHL 5
```

* `45` $\longrightarrow$ Version: `4`, IHL: `5` ($5 \times 4 = 20\text{ Bytes}$).
* `00` $\longrightarrow$ DSCP: `0`, ECN: `0` (Standard Best Effort).
* `00 3c` $\longrightarrow$ Total Length: `0x003C` = **60 Bytes**.
* `1a 2b` $\longrightarrow$ Identification: `0x1A2B`.
* `40 00` $\longrightarrow$ Binary: `0100 0000 0000 0000` $\implies$ Flag **DF=1** (Don't Fragment), **MF=0**, Offset = **0**.
* `40` $\longrightarrow$ TTL: `0x40` = **64 Hops**.
* `06` $\longrightarrow$ Protocol: **TCP** (`6`).
* `b2 c8` $\longrightarrow$ Header Checksum: `0xB2C8`.
* `c0 a8 01 0a` $\longrightarrow$ Source IP: `192.168.1.10`.
* `c0 a8 01 32` $\longrightarrow$ Destination IP: `192.168.1.50`.

---

## 5. Offensive Tradecraft & Layer 3 Threat Surface

```
┌────────────────────────────────────────────────────────────────────────┐
│                      LAYER 3 RECONNAISSANCE & ATTACKS                  │
├──────────────────────────┬─────────────────────────────────────────────┤
│ 1. OS Fingerprinting     │ Passive detection via Default TTL & IP ID   │
│ 2. Source Routing Bypass │ Routing through trust zones via IP Options  │
│ 3. The Evil Bit (RFC3514)│ Reserved Flag 0 covert communication        │
└──────────────────────────┴─────────────────────────────────────────────┘
```

---

### 1. Passive OS Fingerprinting via Default TTL Values
Different operating systems load distinct initial default TTL values into the IPv4 header:

| Operating System | Default Initial TTL | Ping Output Observed (1 Hop Away) |
| :--- | :---: | :---: |
| **Linux Kernel (Modern)** | **`64`** | `ttl=63` |
| **Microsoft Windows** | **`128`** | `ttl=127` |
| **Cisco IOS / Network Gear**| **`255`** | `ttl=254` |
| **FreeBSD / OpenBSD** | **`64`** | `ttl=63` |

> 🔍 **Reconnaissance Vector:** If an Nmap or ping response returns `ttl=56`, calculating $64 - 56 = 8\text{ hops}$ confirms the target is a **Linux host traversing 8 intermediate routers**.

---

### 2. IP Options Exploitation: Source Routing Perimeter Bypasses
* **Attack Scenario:** A target internal server (`10.0.0.50`) is shielded behind a firewall that drops external connections from the Internet.
* **The Exploit:** An attacker crafts an IP datagram with **Loose Source Routing (LSRR)** enabled:
  ```bash
  # Dictates intermediate hops: Forces packet to route via dual-homed host:
  nmap --ip-options "L 192.168.1.1 10.0.0.1" 10.0.0.50
  ```
* **Impact:** The firewall forwards the packet to the trusted intermediate server, which then routes it directly to the protected target, bypassing perimeter firewall boundary rules.
* 🛡️ **Defense:** Modern edge routers drop all IPv4 packets containing IP Options by default (`ip options drop`).

---

## 6. Live Layer 3 Diagnostics & Kernel Inspection

```bash
# 1. Capture Raw IPv4 Headers with Decoded TTL and IP ID via tcpdump
sudo tcpdump -i eth0 -nn -v -c 1 'ip'

# 2. Inspect Linux Path MTU Discovery (DF-bit setting)
cat /proc/sys/net/ipv4/ip_no_pmtu_disc
# 0 = Enabled (Kernel sets DF=1), 1 = Disabled

# 3. Audit Local Ephemeral IP Port Allocations
cat /proc/sys/net/ipv4/ip_local_port_range
# Output: 32768 60999 (Dynamic source ports allocated to outgoing TCP/UDP)
```

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
