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

---

# 📡 Chapter 03: Layer 3 — Network Layer (IPv4, IPv6 & Routing) (Part 3.2)

> **IW Cyber Ops Research Vault | Module 02: Network Protocols & Traffic Engineering**  
> *Author: Muhammad Imran Wakeel (@iwcyberops)*  
> *Track: Binary Subnet Math, CIDR Bitmasks, VLSM Engineering & Route Sizing*

---

## 1. IPv4 Addressing Architecture & Allocations

An IPv4 address is an unsigned **32-bit binary integer** represented in human-readable **Dotted Decimal Notation** (4 octets separated by periods, where each octet spans 8 bits: $0\text{ to }255$).

```
       Dotted Decimal:        192       .       168       .        1        .       50
       Binary Octets:       11000000        10101000          00000001          00110010
       Bit Positions:      b31    b24      b23    b16        b15     b8        b7     b0
```

---

### 1. Legacy Classful Addressing (RFC 791)
Prior to 1993, IPv4 was rigidly partitioned into five discrete classes determined by the **leading high-order bits** of the first octet:

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        LEGACY CLASSFUL ADDRESSING TAXONOMY                             │
├───────┬──────────────────┬──────────────┬───────────────┬──────────────────────────────┤
│ Class │ Leading Bit Rule │ First Octet  │ Default Mask  │ Total Usable Hosts Per Net   │
├───────┼──────────────────┼──────────────┼───────────────┼──────────────────────────────┤
│ **A** │ `0...`           │ `1 - 126`    │ `/8` (255.0.0.0)    │ $2^{24} - 2 = \mathbf{16,777,214}$ │
│ **B** │ `10...`          │ `128 - 191`  │ `/16` (255.255.0.0) │ $2^{16} - 2 = \mathbf{65,534}$     │
│ **C** │ `110...`         │ `192 - 223`  │ `/24` (255.255.255.0)│ $2^8 - 2 = \mathbf{254}$         │
│ **D** │ `1110...`        │ `224 - 239`  │ None          │ Reserved for **Multicast**   │
│ **E** │ `1111...`        │ `240 - 254`  │ None          │ Reserved for Experimental    │
└───────┴──────────────────┴──────────────┴───────────────┴──────────────────────────────┘
```

> ⚠️ **The Classful Depletion Problem:**  
> If an organization required 500 IP addresses, allocating a Class C network ($254\text{ hosts}$) was insufficient, forcing the allocation of an entire Class B network ($65,534\text{ hosts}$). Over $65,000$ IP addresses were permanently wasted, leading to rapid global IPv4 address exhaustion.

---

### 2. Special & Reserved IPv4 Blocks (RFC 6890)

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        CRITICAL RESERVED IPv4 SUBNETS                                  │
├─────────────────────┬──────────────────┬───────────────────────────────────────────────┤
│ Subnet Block        │ RFC Reference    │ Operational / Security Purpose                │
├─────────────────────┼──────────────────┼───────────────────────────────────────────────┤
│ `10.0.0.0/8`        │ RFC 1918         │ Private Enterprise Internal Networks          │
│ `172.16.0.0/12`     │ RFC 1918         │ Private Enterprise Internal Networks          │
│ `192.168.0.0/16`    │ RFC 1918         │ Private Enterprise Internal Networks          │
│ `127.0.0.0/8`       │ RFC 1122         │ Host Loopback Interface (`127.0.0.1` = localhost)│
│ `169.254.0.0/16`    │ RFC 3927         │ Link-Local / APIPA (Auto-assigned when DHCP fails)│
│ `100.64.0.0/10`     │ RFC 6598         │ Carrier-Grade NAT (CGNAT between ISP & Customer)│
│ `0.0.0.0/8`         │ RFC 1122         │ Source Address ("This host on this network")  │
│ `255.255.255.255/32`│ RFC 919          │ Limited Local Broadcast (Never forwarded by L3)│
└─────────────────────┴──────────────────┴───────────────────────────────────────────────┘
```

---

## 2. The Binary Mathematics of Subnetting & CIDR

In 1993, **CIDR (Classless Inter-Domain Routing — RFC 1519)** abolished Class A, B, and C boundaries. Under CIDR, a network boundary is dictated dynamically by a **Subnet Mask**.

$$\text{Total Bits (32)} = \text{Network Bits } (n) + \text{Host Bits } (h)$$

```
  CIDR /26 Notation:
  [ 26 Network Bits: 11111111.11111111.11111111.11 ] [ 6 Host Bits: 000000 ]
  Subnet Mask:      255   .   255   .   255   .  192
```

---

### 1. The Kernel Bitwise AND Operation ($\&$)

When an operating system or router receives a packet destined for an IP, the CPU evaluates which network segment it belongs to by executing a **Bitwise AND ($\&$)** between the IP address and the Subnet Mask:

$$\mathbf{\text{Network ID} = \text{IP Address} \ \& \ \text{Subnet Mask}}$$

#### Step-by-Step Mathematical Proof:
* Given IP: `192.168.1.150`
* Given Mask: `/26` (`255.255.255.192`)

```text
  IP:   192.168.1.150 ──> 11000000 . 10101000 . 00000001 . 10010110
  Mask: /26           ──> 11111111 . 11111111 . 11111111 . 11000000  (Bitwise AND: 1&1=1, else 0)
  ──────────────────────────────────────────────────────────────────
  Net:  192.168.1.128 ──> 11000000 . 10101000 . 00000001 . 10000000
```
* **Network ID:** `192.168.1.128`

---

### 2. Computing Host Capacities & Boundaries

For any given subnet with $h$ host bits:

1. **Total IP Addresses:**
   $$\text{Total IPs} = 2^h$$
2. **Usable Host Addresses:**
   $$\text{Usable Hosts} = 2^h - 2$$
   * Subtracted **`2`**:
     * **Network ID (First IP):** All host bits set to `0` (Identifies the wire).
     * **Broadcast ID (Last IP):** All host bits set to `1` (Directs traffic to all hosts).
3. **The "Magic Number" (Subnet Block Increment):**
   The block size of subnets in the interesting octet is calculated as:
   $$\text{Magic Number} = 256 - \text{Subnet Mask Octet}$$
   * For `/26` (Mask `255.255.255.192`): $256 - 192 = \mathbf{64}$.
   * Subnets increment by 64: `192.168.1.0`, `192.168.1.64`, `192.168.1.128`, `192.168.1.192`.

---

### 3. Special Boundary Subnets: `/30`, `/31`, and `/32`

* **`/30` Subnets ($2^2 - 2 = 2\text{ Hosts}$):** Legacy standard for point-to-point router links (Consumes 4 IPs: Network, Host 1, Host 2, Broadcast).
* **`/31` Subnets (RFC 3021 — $2\text{ Hosts}$):** Modern standard for point-to-point links. **Eliminates Network and Broadcast IDs entirely!** Both IPs are usable, saving millions of addresses in cloud/ISP backbones.
* **`/32` Subnets ($2^0 = 1\text{ Host}$):** Host route. Defines a single, isolated IP endpoint (Used for Loopback interfaces and VPN tunnels).

---

## 3. CIDR Subnetting Reference Master Matrix

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        COMPLETE CIDR / IPv4 PREFIX REFERENCE TABLE                     │
├─────────┬──────────────────────┬─────────────┬──────────────┬──────────────────────────┤
│ Prefix  │ Subnet Mask          │ Total IPs   │ Usable Hosts │ Primary Implementation   │
├─────────┼──────────────────────┼─────────────┼──────────────┼──────────────────────────┤
│ **`/24`**│ 255.255.255.0        │ 256         │ 254          │ Standard LAN Access Pod  │
│ **`/25`**│ 255.255.255.128      │ 128         │ 126          │ Medium Department Segment│
│ **`/26`**│ 255.255.255.192      │ 64          │ 62           │ DMZ Server Enclave       │
│ **`/27`**│ 255.255.255.224      │ 32          │ 30           │ Infrastructure Services  │
│ **`/28`**│ 255.255.255.240      │ 16          │ 14           │ Management / Bastion Pod │
│ **`/29`**│ 255.255.255.248      │ 8           │ 6            │ Multi-Router WAN Cluster │
│ **`/30`**│ 255.255.255.252      │ 4           │ 2            │ Legacy Point-to-Point WAN│
│ **`/31`**│ 255.255.255.254      │ 2           │ 2 (RFC 3021) │ Modern Cloud P2P Link    │
│ **`/32`**│ 255.255.255.255      │ 1           │ 1            │ Host Endpoint / Loopback │
└─────────┴──────────────────────┴─────────────┴──────────────┴──────────────────────────┘
```

---

## 4. Variable Length Subnet Masking (VLSM)

**VLSM (Variable Length Subnet Masking)** allows network engineers to recursively carve an IP allocation into subnets of varying sizes to perfectly match host requirements with near-zero address waste.

```
                         [ BASE ALLOCATION: 192.168.10.0/24 (256 IPs) ]
                                                │
         ┌──────────────────────────────────────┴──────────────────────────────────────┐
         ▼                                                                             ▼
  [ 192.168.10.0/25 ]                                                           [ 192.168.10.128/25 ]
   Subnet 1: Engineering                                                               │
   (Allocates 128 IPs for 100 Hosts)                                                   ▼
                                                                        ┌──────────────┴──────────────┐
                                                                        ▼                             ▼
                                                               [ 192.168.10.128/26 ]        [ 192.168.10.192/26 ]
                                                                Subnet 2: Marketing                  │
                                                                (Allocates 64 IPs for 50 Hosts)      ▼
                                                                                      ┌──────────────┴──────────────┐
                                                                                      ▼                             ▼
                                                                             [ 192.168.10.192/27 ]        [ 192.168.10.224/27 ]
                                                                              Subnet 3: Servers            Unallocated Future
                                                                              (Allocates 32 IPs for 20)    Growth Pool
```

---

### Real-World VLSM Engineering Design Scenario

**Objective:** Segment `10.0.0.0/24` ($256\text{ IPs}$) across four distinct corporate security zones:
* **Zone A (Workstations):** Requires $100\text{ hosts}$ $\implies$ Needs block size $\ge 102$ ($2^7 = 128$) $\implies$ **`/25`**
* **Zone B (Development):** Requires $50\text{ hosts}$ $\implies$ Needs block size $\ge 52$ ($2^6 = 64$) $\implies$ **`/26`**
* **Zone C (DMZ / Web):** Requires $25\text{ hosts}$ $\implies$ Needs block size $\ge 27$ ($2^5 = 32$) $\implies$ **`/27`**
* **Zone D (Router P2P):** Requires $2\text{ hosts}$ $\implies$ Needs block size $\ge 4$ ($2^2 = 4$) $\implies$ **`/30`**

#### The Sequential VLSM Allocation Map:
1. **Sort requirements in descending order:** Always allocate the largest subnet first!
2. **Zone A:** `10.0.0.0/25`
   * Network ID: `10.0.0.0` | Usable Range: `10.0.0.1` to `10.0.0.126` | Broadcast: `10.0.0.127`
3. **Zone B:** `10.0.0.128/26`
   * Network ID: `10.0.0.128` | Usable Range: `10.0.0.129` to `10.0.0.190` | Broadcast: `10.0.0.191`
4. **Zone C:** `10.0.0.192/27`
   * Network ID: `10.0.0.192` | Usable Range: `10.0.0.193` to `10.0.0.222` | Broadcast: `10.0.0.223`
5. **Zone D:** `10.0.0.224/30`
   * Network ID: `10.0.0.224` | Usable Range: `10.0.0.225` to `10.0.0.226` | Broadcast: `10.0.0.227`
6. **Remaining Spare Capacity:** `10.0.0.228` through `10.0.0.255` ($28\text{ IPs}$) unallocated and clean.

---

## 5. Offensive Reconnaissance & Target Scope Mathematics

Understanding CIDR and subnet boundaries is essential for scoping penetration tests, managing scan performance, and identifying pivot boundaries.

```
┌────────────────────────────────────────────────────────────────────────┐
│                   TARGET SCOPE RECONNAISSANCE MATHEMATICS              │
├──────────────────────────┬─────────────────────────────────────────────┤
│ 1. Scope Definition      │ Translating client IP ranges into optimal   │
│                          │ CIDR targets for port scanning engines      │
│ 2. Lateral Movement      │ Identifying whether a target IP is on the   │
│                          │ local wire vs traversing a monitored gateway│
│ 3. Scan Duration Math    │ Estimating packet volume based on CIDR size │
└──────────────────────────┴─────────────────────────────────────────────┘
```

---

### Scanning Duration Estimation Formula
When scanning an entire CIDR block (e.g., `/16` with $65,536\text{ hosts}$) across all 65,535 TCP ports:

$$\text{Total Probes} = 2^{32 - \text{CIDR}} \times \text{Target Ports}$$

* Scanning a `/24` ($256\text{ hosts}$) on Top 1000 ports $= 256 \times 1,000 = \mathbf{256,000\text{ Packets}}$.
* Scanning a `/16` ($65,536\text{ hosts}$) on Top 1000 ports $= 65,536 \times 1,000 = \mathbf{65,536,000\text{ Packets}}$!
* At a transmission rate of $1,000\text{ packets/sec}$, scanning the `/16` takes $\mathbf{\approx 18.2\text{ Hours}}$, risking network saturation and firewall blacklisting unless scoped to specific subnets.

---

## 6. Live Subnet Diagnostics & Terminal Tools

```bash
# 1. Inspect Assigned Interface CIDR Prefixes
ip -brief addr
# Output: eth0 UP 192.168.1.50/24

# 2. Automated Subnet Calculation via ipcalc
ipcalc 192.168.1.150/26
# Output displays: Address, Netmask, Wildcard, Network, HostMin, HostMax, Broadcast, Hosts/Net

# 3. Target Scoping with Nmap (Generate Target List from CIDR without scanning)
nmap -sL 192.168.1.0/28
# Performs reverse DNS resolution and prints all 16 target IP boundaries cleanly
```

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->

---

# 📡 Chapter 03: Layer 3 — Network Layer (IPv4, IPv6 & Routing) (Part 3.3)

> **IW Cyber Ops Research Vault | Module 02: Network Protocols & Traffic Engineering**  
> *Author: Muhammad Imran Wakeel (@iwcyberops)*  
> *Track: IP Fragmentation Engines, PMTUD, Traceroute Mechanics & ICMP Weaponization*

---

## 1. IP Fragmentation & Reassembly (RFC 791 Mechanics)

Physical networks impose a hard ceiling on frame capacity known as the **Maximum Transmission Unit (MTU)**. Standard Ethernet enforces an MTU of **1500 Bytes**. 

When an IPv4 datagram exceeds the MTU of an egress physical link (e.g., encountering a $1492\text{-byte}$ PPPoE link or a $1400\text{-byte}$ VPN tunnel), the router must either **Fragment** the datagram into smaller units or drop it.

```
       [ Original Datagram: 4000 Bytes (20B IP Header + 3980B Payload) ] ──> MTU: 1500 Bytes
                                        │
         ┌──────────────────────────────┼──────────────────────────────┐
         ▼                              ▼                              ▼
  [ Fragment 1 ]                 [ Fragment 2 ]                 [ Fragment 3 ]
  Total Len: 1500 Bytes          Total Len: 1500 Bytes          Total Len: 1040 Bytes
  Header: 20B                    Header: 20B                    Header: 20B
  Data: 1480B                    Data: 1480B                    Data: 1020B
  MF = 1 (More Follows)          MF = 1 (More Follows)          MF = 0 (Last Fragment)
  Offset = 0                     Offset = 185 (1480 / 8)        Offset = 370 (2960 / 8)
```

---

### The Mathematical Rules of Fragmentation

Every IP fragment is an independent, valid IPv4 datagram containing its own 20-byte IP header. Three fields in the header govern the fragmentation and reassembly engine:

1. **Identification (IP ID — 16 Bits):** Identical across all fragments. Informs the receiving operating system that incoming fragments belong to the same parent datagram.
2. **Flags (3 Bits):**
   * **`DF` (Don't Fragment — Bit 1):** If set to `1`, routers are strictly forbidden from fragmenting. If a packet exceeds the MTU, the router **must drop it** and return an ICMP error.
   * **`MF` (More Fragments — Bit 2):** Set to `1` for all fragments *except the final fragment*, which has `MF = 0`.
3. **Fragment Offset (13 Bits):** Dictates where the fragment's payload fits into the original unfragmented stream.
   * **The 8-Byte Block Rule:** Because the offset field is only 13 bits (cannot address 65,535 individual bytes), it measures position in **units of 8-byte (64-bit) blocks**:

$$\text{Byte Offset Position} = \text{Fragment Offset Value} \times 8$$

$$\text{Data Payload per Intermediate Fragment} \equiv 0 \pmod{8} \quad (\text{Must be a multiple of 8!})$$

---

### ⚠️ The Layer 4 Blind Spot (Critical Security Reality)
* **Fragment 0 (Offset 0):** Contains the IPv4 header **AND the Layer 4 header** (TCP Source/Destination Ports, Sequence Numbers, or UDP headers).
* **Fragments 1 through N (Offset > 0):** Contain the IPv4 header and **raw payload data ONLY**. They have **NO TCP or UDP port headers**!

> 🔴 **Firewall Evasion Implications:**  
> Legacy stateless firewalls that filter traffic by inspecting Layer 4 ports (e.g., blocking Port 4444) can only inspect Fragment 0. If non-zero fragments are forwarded, the firewall cannot see the destination port, allowing raw exploit fragments to pass through uninspected!

---

## 2. Path MTU Discovery (PMTUD) & The Black Hole Problem

Modern operating systems avoid IP fragmentation by discovering the smallest MTU across the entire routed path using **PMTUD (RFC 1191)**.

```
       [ Sender: Sets DF = 1 ] ──( 1500B Datagram )──> [ Intermediate Router: MTU = 1400B ]
                                                                   │
                                                                   ▼ (Cannot Fragment: DF=1)
                                                       [ Packet Dropped Silently! ]
                                                                   │
       [ Sender Receives ICMP Alert ] ◄──( ICMP Type 3, Code 4 )───┘
       "Fragmentation Needed, DF Set. Next-Hop MTU = 1400"
       Sender reduces TCP MSS (Max Segment Size) to fit 1400B!
```

---

### The PMTUD "Black Hole" Failure
* **The Problem:** Many paranoid system administrators configure firewalls to blindly **drop ALL ICMP traffic** (`iptables -A INPUT -p icmp -j DROP`).
* **The Failure Mode:** When an intermediate router drops a packet that exceeds its MTU, it sends back an **ICMP Type 3 Code 4** packet. The sender's firewall drops this ICMP packet.
* **The Symptom (Black Hole Connection):** Small packets (ping, TCP 3-way handshake) succeed because they are $<1400\text{ bytes}$. But the moment a large payload is sent (TLS Certificate exchange, HTTP POST, file transfer), the connection completely freezes and times out without an error.

---

## 3. Time-to-Live (TTL) & The Traceroute Engine

The 8-bit **TTL** field prevents packets from looping infinitely. The `traceroute` utility leverages TTL decrements and ICMP error generation to map Layer 3 router paths:

```
                  ┌────────────────────────────────────────────────────────┐
                  │              THE TRACEROUTE EXECUTION ENGINE           │
                  └──────────────────────────┬─────────────────────────────┘
                                             │
      ┌──────────────────────────────────────┼──────────────────────────────────────┐
      ▼ (Probe 1: TTL = 1)                   ▼ (Probe 2: TTL = 2)                   ▼ (Probe 3: TTL = 3)
 [ Router 1: Decrements TTL to 0 ]      [ Router 2: Decrements TTL to 0 ]      [ Target Destination Reached ]
  Drops packet; generates:               Drops packet; generates:               Processes payload; generates:
  ICMP Type 11, Code 0                   ICMP Type 11, Code 0                   ICMP Type 3, Code 3 (Port Unreach)
  (Time-to-Live Exceeded!)               (Time-to-Live Exceeded!)               (Or TCP SYN-ACK / Echo Reply)
```

---

### Implementation Discrepancies: Linux vs Windows

| Operating System | Tool Name | Transport Mechanism | Target Probe Signature |
| :--- | :--- | :--- | :--- |
| **Linux / Unix** | `traceroute` | **UDP Datagrams** | Probes high, randomized UDP ports (`33434` to `33534`). |
| **Microsoft Windows** | `tracert` | **ICMP Echo Requests** | Transmits standard ICMP Type 8 Echo Requests. |
| **Offensive Scanners** | `tcptraceroute` | **TCP SYN Segments** | Probes open web ports (e.g., Port 80/443) to traverse firewalls. |

---

## 4. ICMP Protocol Architecture (RFC 792)

The **Internet Control Message Protocol (ICMP)** is the diagnostic and error-reporting mechanism for Layer 3. It is encapsulated directly inside IPv4 with **Protocol Number `1`**.

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        RFC 792 ICMP PACKET WIRE FORMAT                                 │
├─────────────────────┬─────────────────────┬────────────────────────────────────────────┤
│ Type (8 Bits)       │ Code (8 Bits)       │ Checksum (16 Bits: 1's Complement Sum)     │
├─────────────────────┴─────────────────────┴────────────────────────────────────────────┤
│ Rest of Header (4 Bytes: Depends on Type - e.g., Identifier & Sequence Number)         │
├────────────────────────────────────────────────────────────────────────────────────────┤
│ Data Payload: For Error Messages, includes the ORIGINAL IP HEADER + 8 BYTES of Payload │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

> 💡 **Why ICMP Errors Include 8 Bytes of the Original Payload:**  
> When a router returns an error (e.g., Destination Unreachable), the sending operating system needs to know **which process and socket triggered the error**. The first 8 bytes of the Layer 4 payload contain the **Source and Destination Ports**, allowing the Linux kernel to map the error back to the originating process!

---

### Critical ICMP Types and Codes Reference

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        CORE ICMP MESSAGE TYPES & CODES                                 │
├──────┬──────┬───────────────────────────────┬──────────────────────────────────────────┤
│ Type │ Code │ Message Name                  │ Operational Trigger                      │
├──────┼──────┼───────────────────────────────┼──────────────────────────────────────────┤
│ **0**│ `0`  │ **Echo Reply**                │ Response to Ping (Ping Success)          │
├──────┼──────┼───────────────────────────────┼──────────────────────────────────────────┤
│ **3**│ `0`  │ Net Unreachable               │ Routing table has no route to network    │
│      │ `1`  │ Host Unreachable              │ Final router cannot resolve target ARP   │
│      │ `2`  │ Protocol Unreachable          │ Receiving OS does not support L4 protocol│
│      │ `3`  │ **Port Unreachable**          │ Target UDP port is closed (No listener)  │
│      │ `4`  │ **Fragmentation Needed (DF)** │ Packet > MTU, but DF flag was set        │
├──────┼──────┼───────────────────────────────┼──────────────────────────────────────────┤
│ **5**│ `0`  │ Redirect for Network          │ Informs host of a better gateway route   │
│      │ `1`  │ **Redirect for Host**         │ Informs host of a better gateway for host│
├──────┼──────┼───────────────────────────────┼──────────────────────────────────────────┤
│ **8**│ `0`  │ **Echo Request**              │ Diagnostic Ping Probe                    │
├──────┼──────┼───────────────────────────────┼──────────────────────────────────────────┤
│**11**│ `0`  │ **Time-to-Live Exceeded**     │ TTL reached 0 in transit (Traceroute)    │
│      │ `1`  │ Frag Reassembly Time Exceeded │ Reassembly timer expired before all frags│
└──────┴──────┴───────────────────────────────┴──────────────────────────────────────────┘
```

---

## 5. Offensive Exploitation & Layer 3 Attack Classes

```
┌────────────────────────────────────────────────────────────────────────┐
│                   LAYER 3 EXPLOIT MECHANICS                            │
├──────────────────────────┬─────────────────────────────────────────────┤
│ 1. Teardrop Attack       │ Overlapping fragment offsets crash OS kernel│
│ 2. Tiny Fragment Evasion │ Hides Layer 4 ports past packet filters     │
│ 3. ICMP Redirect MITM    │ Injects rogue gateway routes via ICMP Type 5│
└──────────────────────────┴─────────────────────────────────────────────┘
```

---

### 1. The Teardrop Attack (Fragment Overlap Crash)
* **Vulnerability:** Older TCP/IP kernel stacks allocated memory buffers based on the payload size of Fragment 1, and calculated subsequent fragment sizes by subtracting offsets:
  $$\text{Payload Length} = \text{Total Length} - \text{Offset}$$
* **Exploitation:** An attacker crafts Fragment 2 with an **Offset that overlaps Fragment 1** (e.g., Fragment 1 has bytes $0\text{--}99$, and Fragment 2 claims offset $80$, but specifies a total length smaller than the offset).
* **Impact:** The calculation yielded a **negative number** or an integer underflow wrapped to a massive 32-bit integer, causing a kernel memory corruption crash (**Blue Screen / Kernel Panic**).

---

### 2. ICMP Redirect Route Hijacking (Layer 3 MITM)
If a host has multiple routers on its subnet, an attacker on the same local segment can transmit a spoofed **ICMP Type 5 Code 1 (Redirect for Host)** message:

```text
Spoofed ICMP Redirect sent to Victim:
"To reach 8.8.8.8, use Gateway 192.168.1.100 (Attacker's IP) instead of 192.168.1.1!"
```
* **Impact:** The victim's kernel updates its local dynamic route cache, forwarding all traffic destined for `8.8.8.8` directly to the attacker, establishing a transparent Layer 3 Man-in-the-Middle.

---

## 6. Live Fragment & ICMP Diagnostics Matrix

```bash
# 1. Capture Raw ICMP Traffic and Decode Types/Codes via tcpdump
sudo tcpdump -i eth0 -nn -v 'icmp'

# 2. Transmit Custom Ping with Explicit MTU Sweep and DF Flag Enabled
# Tests exact path MTU without fragmentation (1472B Data + 20B IP + 8B ICMP = 1500B MTU):
ping -c 1 -M do -s 1472 1.1.1.1
# If output returns "Frag needed and DF set", the link MTU is smaller than 1500!

# 3. Defensive Hardening: Disable ICMP Redirect Acceptance in Linux Kernel
# Prevents ICMP Redirect MITM attacks:
sudo sysctl -w net.ipv4.conf.all.accept_redirects=0
sudo sysctl -w net.ipv4.conf.default.accept_redirects=0
sudo sysctl -w net.ipv4.conf.all.send_redirects=0

# 4. Audit Fragment Reassembly Timers
cat /proc/sys/net/ipv4/ipfrag_time
# Default: 30 seconds before kernel discards incomplete fragment queues
```

---

<!-- =========================================================================
   [IW CYBER Ops] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
