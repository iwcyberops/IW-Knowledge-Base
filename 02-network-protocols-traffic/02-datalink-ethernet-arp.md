<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran Wakeel | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 02 — Network Protocols, Packet Dissection & Traffic Engineering
   DOCUMENT: Chapter 02 — Layer 1 & Layer 2: Physical & Data Link Architecture (Part 2.1)
   ========================================================================= -->

# 🔌 Chapter 02: Layer 1 & Layer 2 — Data Link, Ethernet & ARP (Part 2.1)

> **IW Cyber Ops Research Vault | Module 02: Network Protocols & Traffic Engineering**  
> *Author: Muhammad Imran Wakeel (@iwcyberops)*  
> *Track: Physical Signaling, Line Encoding, Ethernet II Frame Dissection & CRC-32 Math*

---

## 1. Physical Layer Physics: Signaling, Modulation & Encoding

Before a packet can exist as an abstract data structure in operating system memory, it must traverse physical copper or optical fiber as continuous electromagnetic or optical signals.

```
       [ Binary Stream: 1 0 1 1 0 ] ──> [ Line Encoder / PHY ] ──> [ Analog Voltages / Pulses ]
```

---

### 1. Baud Rate vs Bit Rate (The Nyquist & Shannon Bounds)
* **Bit Rate ($R$):** The number of discrete binary bits ($0$s and $1$s) transmitted per second ($\text{bps}$).
* **Baud Rate ($S$):** The number of physical signal state transitions (pulses/symbols) occurring per second ($\text{symbols/sec}$).

$$R = S \times \log_2(M) \quad (M = \text{Number of voltage/signal states per symbol})$$

#### Theoretical Physical Bounds:
1. **Nyquist Maximum Data Rate (Noiseless Channel):**
   $$C = 2B \log_2(M) \quad (B = \text{Analog Bandwidth in Hz})$$
2. **Shannon-Hartley Theorem (Noisy Channel Physical Capacity):**
   $$C = B \log_2\left(1 + \frac{S}{N}\right) \quad \left(\frac{S}{N} = \text{Signal-to-Noise Ratio}\right)$$
   * Even with infinite computational power, physical background thermal noise imposes an absolute mathematical ceiling on how much data can traverse a physical link.

---

### 2. Line Encoding Schemes

If an operating system sends a continuous stream of zeros (`00000000...`) down a wire, an unencoded signal remains flat. The receiver's physical clock drifts out of synchronization, losing bit-alignment. **Line Encoding** embeds clock synchronization directly into the transmitted bitstream:

```
    NON-RETURN-TO-ZERO (NRZ)                       MANCHESTER ENCODING (10BASE-T)
    (Flat line; clock drifts on long runs)         (Bit transition in the MIDDLE of every clock cycle)
     ┌───┐       ┌───┐                              ┌─┐   ┌─┐   ┌─┐
   ──┘   └───────┘   └─── (1 0 0 0 1)             ──┘ └───┘ └───┘ └── (Guarantees self-clocking)
```

| Encoding Standard | Physical Implementation | Mechanical Function |
| :--- | :--- | :--- |
| **NRZ (Non-Return-to-Zero)**| Direct voltage mapping: High = `1`, Low = `0` | Simplest scheme; suffers from clock drift and baseline wander during long identical bit sequences. |
| **Manchester Encoding** | Mid-bit transition: Low-to-High = `1`, High-to-Low = `0` | Self-clocking ($10\text{BASE-T}$ Ethernet); consumes twice the bandwidth of raw data rate. |
| **PAM-5 (Pulse Amplitude)** | 5 discrete voltage levels ($\pm 2\text{V}, \pm 1\text{V}, 0\text{V}$) | Transmits 2 bits per symbol across 4 twisted pairs simultaneously, achieving $1000\text{BASE-T}$ ($1\text{ Gbps}$). |
| **8B/10B Encoding** | Maps 8-bit data bytes into 10-bit line symbols | Used in Gigabit fiber and SATA; prevents DC imbalance and guarantees transitions for clock recovery. |

---

## 2. The Ethernet II Frame Architecture & Hex Dissection

In modern networks, the **Ethernet II (DIX Ethernet)** frame format dominates 100% of switched IP traffic, having superseded the legacy IEEE 802.3 LLC frame.

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        ETHERNET II (DIX) FRAME SPECIFICATION                           │
├───────────────┬────────────────────────────────────────────────────────┬───────────────┤
│ PHYSICAL L1   │ LAYER 2 ETHERNET FRAME: 64 to 1518 BYTES (Untagged)    │ PHYSICAL L1   │
├───────────────┼────────────┬────────────┬───────────┬────────┬─────────┼───────────────┤
│ Preamble/SFD  │ Dest MAC   │ Src MAC    │ EtherType │ Data   │ FCS     │ Inter-Packet  │
│ 8 Bytes       │ 6 Bytes    │ 6 Bytes    │ 2 Bytes   │ Payload│ (CRC32) │ Gap (IPG)     │
│ (Not in PCAP) │ (48 Bits)  │ (48 Bits)  │ (16 Bits) │ 46-1500│ 4 Bytes │ 96 Bit-Times  │
└───────────────┴────────────┴────────────┴───────────┴────────┴─────────┴───────────────┘
```

---

### Frame Fields Breakdown:

1. **Preamble (7 Bytes):** `0x55 55 55 55 55 55 55` (Alternating `10101010` bit pattern). Synchronizes the receiving PHY clock.
2. **SFD (Start Frame Delimiter — 1 Byte):** `0xD5` (Bit sequence `10101011`). The terminating dual-ones signal the start of destination addressing.
3. **Destination MAC (6 Bytes / 48 Bits):** Hardware address of the receiving device on the local Layer 2 segment.
4. **Source MAC (6 Bytes / 48 Bits):** Hardware address of the transmitting network interface.
5. **EtherType (2 Bytes / 16 Bits):** Identifies which Layer 3 protocol payload is encapsulated inside the frame:
   * `0x0800` $\longrightarrow$ **IPv4** (Internet Protocol version 4)
   * `0x0806` $\longrightarrow$ **ARP** (Address Resolution Protocol)
   * `0x86DD` $\longrightarrow$ **IPv6** (Internet Protocol version 6)
   * `0x8100` $\longrightarrow$ **VLAN-Tagged Frame** (IEEE 802.1Q)
6. **Payload (46 to 1500 Bytes):** The Layer 3 packet. If data is $<46\text{ bytes}$, padding bytes (`0x00`) are appended to satisfy the 64-byte minimum frame size.
7. **Frame Check Sequence (FCS — 4 Bytes):** 32-bit Cyclic Redundancy Check (CRC-32) computed over all fields from Destination MAC through Payload.

---

### 🔬 Low-Level Hexadecimal Frame Dissection (Raw Wire Capture)

Below is an annotated hex dump of an actual raw Ethernet II frame captured via `tcpdump -xx`:

```text
00:50:56:c0:00:08 00:0c:29:ab:cd:ef 08 00 45 00 00 2c ...
```

```
 00 50 56 c0 00 08 │ 00 0c 29 ab cd ef │ 08 00 │ 45 00 00 2c ...
 ─────────────────┼───────────────────┼───────┼────────────────
  Destination MAC │ Source MAC        │ Ether │ Encapsulated
  (VMware Gateway)│ (Kali Linux Host) │ Type  │ IPv4 Packet
                  │                   │(IPv4) │ (Starts with 0x45)
```

* `00 50 56 c0 00 08` $\longrightarrow$ Destination MAC Address ($6\text{ bytes}$).
* `00 0c 29 ab cd ef` $\longrightarrow$ Source MAC Address ($6\text{ bytes}$).
* `08 00` $\longrightarrow$ EtherType: Indicates IPv4 payload follows ($2\text{ bytes}$).
* `45 00 ...` $\longrightarrow$ IPv4 Header (`0x45` = Version 4, Header Length 5 words = $20\text{ bytes}$).

---

## 3. The Mathematical Integrity Engine: CRC-32 (FCS)

The **Frame Check Sequence (FCS)** verifies that bits have not flipped due to electrical interference, noise, or damaged media during wire transit. It uses **Cyclic Redundancy Check (CRC-32)**.

```
       [ Payload Bits D ] ───( Append 32 Zeros )───> [ Divide by Generator G ] ───> [ Remainder R (CRC) ]
```

---

### Modulo-2 Arithmetic & Polynomial Division

CRC-32 treats the bitstream as a massive mathematical polynomial where addition and subtraction are performed via **bitwise XOR ($\oplus$) with zero carries**.

The IEEE 802.3 standard defines the static **Generator Polynomial ($G$)**:

$$G(x) = x^{32} + x^{26} + x^{23} + x^{22} + x^{16} + x^{12} + x^{11} + x^{10} + x^8 + x^7 + x^5 + x^4 + x^2 + x + 1$$

* Hexadecimal Representation: `0xEDB88320` (Reversed) or `0x04C11DB7`.
* Hardware Execution: Implemented directly in NIC silicon via a **32-stage Linear Feedback Shift Register (LFSR)**.
* Verification: The receiver divides the entire received frame (including the appended 4-byte CRC remainder) by the generator polynomial $G$. If the final remainder is exactly **`0`** (or a known CRC magic constant), the frame is clean. If non-zero, the NIC hardware drops the frame immediately.

> 🔴 **Security & Cryptographic Reality:**  
> **CRC-32 is an error-detection code, NOT a cryptographic integrity hash (like HMAC-SHA256).**  
> CRC provides **zero protection against an active adversary**. Because modulo-2 division is linear, an attacker modifying data bits in transit can mathematically calculate the corresponding bit-flips in the CRC field to produce a perfectly valid FCS trailer in microseconds!

---

## 4. Collision Arbitration: CSMA/CD & Slot Time Physics

In legacy half-duplex Ethernet (shared coax or Hub networks), multiple hosts share the exact same physical copper medium.

```
                                      [ SHARED HALF-DUPLEX WIRE ]
                                                  │
                 ┌────────────────────────────────┴────────────────────────────────┐
                 ▼                                                                 ▼
           [ Host A Transmits ]                                          [ Host B Transmits ]
                 │                                                                 │
                 └──────────────────────► 💥 COLLISION! ◄──────────────────────────┘
```

---

### CSMA/CD Engine (Carrier Sense Multiple Access / Collision Detect)

1. **Carrier Sense:** Listen before transmitting. If the wire is busy, wait.
2. **Multiple Access:** All nodes have equal rights to transmit when the medium is idle.
3. **Collision Detection:** Listen while transmitting. If an electrical voltage spike occurs ($>2\text{V}$ on coax, signaling collision), abort transmission immediately.
4. **Jam Signal:** Broadcast an intense 32-bit **Jam Signal** across the wire so all connected hosts detect the collision.

---

### The Mathematical Proof: Why 64 Bytes is the Minimum Frame Size!

* **The Engineering Problem:** If a frame is too small, a host could finish transmitting the entire frame and disconnect before the collision signal physically travels across the cable and returns! The host would assume the packet was delivered safely when it was actually destroyed.
* **Network Diameter Limit:** On a $10\text{ Mbps}$ network spanning $2.5\text{ km}$ with repeaters, the maximum **Round-Trip Propagation Time (Slot Time)** is approximately **$51.2\text{ microseconds}$**.
* **Minimum Bit Transmission Math:**
  $$\text{Minimum Bits} = 10,000,000\text{ bits/sec} \times 0.0000512\text{ sec} = \mathbf{512\text{ Bits}}$$
  $$\text{Minimum Frame Size} = \frac{512\text{ Bits}}{8\text{ bits/byte}} = \mathbf{64\text{ Bytes}}$$

> 💡 **Core Rule:** An Ethernet frame must be **at least 64 bytes** so that the transmitting host is **still sending bits** when a worst-case collision echo returns, allowing hardware to catch the collision and invoke retransmission!

---

### Truncated Binary Exponential Backoff Algorithm
After a collision, hosts do not retransmit at the same time (which would cause infinite collisions). They execute the **Exponential Backoff Algorithm**:

$$\text{Wait Time} = r \times \text{Slot Time} \quad (51.2\text{ }\mu\text{s})$$

Where $r$ is a pseudorandom integer chosen uniformly from the set:
$$r \in [0, 2^k - 1] \quad (\text{where } k = \min(\text{collision\_attempt}, 10))$$

* After 1 collision: $r \in [0, 1]$ (Waits 0 or 1 slot time).
* After 3 collisions: $r \in [0, 7]$ (Waits between 0 and 7 slot times).
* After 16 consecutive collisions: Transmission **fails permanently** and drops the packet.

---

## 5. Live Layer 2 Frame Diagnostics & Packet Carving

```bash
# 1. Capture Raw Ethernet Frames with Layer 2 Headers (-e) and Hex Output (-xx)
sudo tcpdump -i eth0 -e -xx -c 1

# Example Output Breakdown:
# 00:0c:29:ab:cd:ef > 00:50:56:c0:00:08, ethertype IPv4 (0x0800), length 74:
#   0x0000:  0050 56c0 0008 000c 29ab cdef 0800 4500  .PV.....).....E.
#   0x0010:  003c 4a21 4000 4006 1234 c0a8 0132 c0a8  .<J!@.@..4...2..

# 2. Extract Specific EtherType Traffic (Filter for ARP: 0x0806)
sudo tcpdump -i eth0 -e 'ether proto 0x0806'

# 3. Audit Physical Port Speed, Duplex, and Auto-Negotiation
sudo ethtool eth0 | grep -Ei "Speed|Duplex|Auto-negotiation"
```

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
   
---


# 🔌 Chapter 02: Layer 1 & Layer 2 — Data Link, Ethernet & ARP (Part 2.2)

> **IW Cyber Ops Research Vault | Module 02: Network Protocols & Traffic Engineering**  
> *Author: Muhammad Imran Wakeel (@iwcyberops)*  
> *Track: Hardware Addressing (EUI-48), CAM Table Silicon & 802.1Q VLAN Tagging*

---

## 1. MAC Address Architecture (EUI-48 Specification)

A **Media Access Control (MAC) Address** is a 48-bit (6-byte) physical hardware identifier burned into the Network Interface Card (NIC) during manufacturing (**BIA — Burned-In Address**). It provides globally unique Layer 2 addressing across shared broadcast domains.

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        48-BIT (6-BYTE) MAC ADDRESS STRUCTURE                           │
├───────────────────────────────────────────┬────────────────────────────────────────────┤
│ 24 BITS (3 BYTES): OUI                    │ 24 BITS (3 BYTES): NIC SPECIFIC EXTENSION  │
├───────────────────────────────────────────┼────────────────────────────────────────────┤
│ Assigned by IEEE to Vendor / Manufacturer │ Assigned by Manufacturer to Hardware Card  │
│ Example: 00:0C:29 (VMware Inc.)           │ Example: AB:CD:EF (Specific Interface ID)  │
└───────────────────────────────────────────┴────────────────────────────────────────────┘
```

---

### The Two Critical Control Bits in the First Octet

The first byte (8 bits) of a MAC address contains **two reserved control bits** that fundamentally alter how the hardware processes the frame:

```text
First Octet Binary: b7  b6  b5  b4  b3  b2  b1  b0
                                             │   └── Bit 0: I/G Bit (Individual vs Group)
                                             └────── Bit 1: U/L Bit (Universal vs Local)
```

```
┌───────────────┬───────────────┬────────────────────────────────────────────────────────┐
│ Control Bit   │ Binary Value  │ Functional Hardware / Protocol Action                  │
├───────────────┼───────────────┼────────────────────────────────────────────────────────┤
│ **I/G Bit**   │ **`0`**       │ **Individual (Unicast):** Addressed to a single host.  │
│ (Bit 0 / LSB) │ **`1`**       │ **Group (Multicast / Broadcast):** Addressed to many.  │
├───────────────┼───────────────┼────────────────────────────────────────────────────────┤
│ **U/L Bit**   │ **`0`**       │ **Universally Administered:** IEEE burned-in address.  │
│ (Bit 1)       │ **`1`**       │ **Locally Administered:** MAC has been spoofed or set  │
│               │               │ by software (Docker, VMs, `macchanger`).              │
└───────────────┴───────────────┴────────────────────────────────────────────────────────┘
```

> 🔍 **Forensic Reconnaissance Tip:**  
> When performing traffic analysis, inspect the second hexadecimal character of the MAC address. If it is **`2`**, **`6`**, **`A`**, or **`E`** (where the binary U/L bit is `1`), the MAC address is **Locally Administered (Software Spoofed)**, indicating potential evasion or virtualized containers.

---

### Unicast vs Broadcast vs Multicast MAC Types

```bash
# 1. Broadcast MAC Address: 48 continuous binary ones
FF:FF:FF:FF:FF:FF   # Ingested by every single NIC on the local Layer 2 segment

# 2. IPv4 Multicast MAC Mapping (RFC 1112 Specification)
01:00:5E:xx:xx:xx   # High-order 24 bits are fixed as 01:00:5E; low-order 23 bits map the IP
```

#### The RFC 1112 Multicast Address Ambiguity (Exploit Surface):
When an IPv4 multicast address (e.g., `224.0.0.1`) is mapped to an Ethernet MAC:
* An IPv4 multicast group contains **28 bits** of unique addressing.
* IEEE only allocated **23 bits** of MAC address space (`01:00:5E:00:00:00` to `01:00:5E:7F:FF:FF`).
* **The Mathematical Collision:** $28 - 23 = \mathbf{5\text{ bits of address loss}}$. Exactly **32 distinct IP multicast groups map to the exact same Ethernet MAC address**! An attacker on the local wire can inject packets into a multicast MAC that unintended subscriber applications will ingest.

---

## 2. Switch Architecture: The CAM Table Engine

Layer 2 switches do not route packets; they forward Ethernet frames using an internal hardware database called the **Content-Addressable Memory (CAM) Table** (also known as the **MAC Address Table**).

```
                            [ ETHERNET FRAME INGRESS ON PORT 1 ]
                               (Src: MAC-A | Dest: MAC-B)
                                           │
                                           ▼
                   ┌───────────────────────────────────────────────┐
                   │ STEP 1: CAM TABLE LEARNING                    │
                   │ Associates Source MAC-A with physical Port 1. │
                   │ Starts/Resets aging timer (Default: 300 sec). │
                   └───────────────────────┬───────────────────────┘
                                           │
                                           ▼
                   ┌───────────────────────────────────────────────┐
                   │ STEP 2: FORWARDING / FILTERING DECISION       │
                   │ Searches CAM table for Destination MAC-B      │
                   └───────────────────────┬───────────────────────┘
                                           │
                 ┌─────────────────────────┴─────────────────────────┐
                 ▼ (MAC-B Found in Table)                            ▼ (MAC-B NOT in Table)
     [ FORWARD OUT SPECIFIC PORT ]                       [ UNKNOWN UNICAST FLOODING ]
     Transmits frame out Port 2 ONLY                     Broadcasts frame out EVERY port
     (Isolated Collision Domain)                         except the incoming ingress port!
```

---

### The 5 States of the Transparent Bridging Algorithm

1. **Learning:** The switch examines the **Source MAC** of every incoming frame and records `[MAC Address, Port Number, VLAN ID, Aging Timer]` into the CAM table.
2. **Aging:** Every CAM entry has an aging countdown (typically $300\text{ seconds}$). If no frame is seen from that MAC before the timer expires, the entry is purged to free memory.
3. **Forwarding:** If the **Destination MAC** exists in the CAM table, the switch forwards the frame directly to that mapped physical port.
4. **Filtering:** If the Destination MAC resides on the same port where the frame entered, the frame is silently dropped (filtered) because the target has already seen it.
5. **Flooding (Unknown Unicast Flooding):** If the Destination MAC is not in the CAM table (or is broadcast `FF:FF:FF:FF:FF:FF`), the frame is copied and flooded out **all active ports** within that VLAN.

---

## 3. VLANs & IEEE 802.1Q Tagging

A **Virtual Local Area Network (VLAN)** partitions a single physical switch into multiple isolated logical Layer 2 broadcast domains. Hosts on VLAN 10 cannot communicate with hosts on VLAN 20 without traversing a Layer 3 router.

```
       [ SWITCH ACCESS PORTS ]                                 [ SWITCH TRUNK PORT ]
  Untagged Frames: PC to Switch                           Tagged Frames: Switch to Switch
 ┌──────────────────────────────┐                       ┌─────────────────────────────────┐
 │ Port 1 (VLAN 10): Standard   │ ──( Tag Injected )──> │ 802.1Q Trunk Link               │
 │ Ethernet Frame (1518 Bytes)  │                       │ Expanded Frame (1522 Bytes)     │
 └──────────────────────────────┘                       └─────────────────────────────────┘
```

---

### The 802.1Q Frame Format: Injected 4-Byte Tag

When an Ethernet frame crosses an inter-switch **Trunk Link**, the switch hardware injects a **4-byte (32-bit) 802.1Q Header** directly between the Source MAC and EtherType fields:

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        IEEE 802.1Q INJECTED VLAN HEADER (4 BYTES)                      │
├───────────────────────────────────────────┬────────────────────────────────────────────┤
│ TPID (16 BITS)                            │ TCI: TAG CONTROL INFORMATION (16 BITS)     │
├───────────────────────────────────────────┼──────────┬───────────┬─────────────────────┤
│ Tag Protocol Identifier                   │ PCP      │ DEI       │ VID                 │
│ Fixed Value: 0x8100                       │ 3 Bits   │ 1 Bit     │ 12 Bits             │
└───────────────────────────────────────────┴──────────┴───────────┴─────────────────────┘
```

#### Field Bit Breakdown:
1. **TPID (Tag Protocol Identifier — 16 Bits):** Always set to **`0x8100`**. Signals to receiving hardware that an 802.1Q tag follows.
2. **PCP (Priority Code Point — 3 Bits):** Implements **IEEE 802.1p Quality of Service (QoS)**. Values range from $0$ (Best Effort) to $7$ (Network Control / Critical Traffic).
3. **DEI (Drop Eligible Indicator — 1 Bit):** May be set to `1` to indicate that this frame can be dropped first during network congestion.
4. **VID (VLAN Identifier — 12 Bits):** The actual numerical VLAN ID:
   * 12-bit range: $2^{12} = \mathbf{4096\text{ possible IDs}}$ ($0$ to $4095$).
   * Reserved: VLAN $0$ (Priority tagged) and VLAN $4095$ (System reserved).
   * **Usable Range:** VLAN **`1` through `4094`**.

---

### Frame Expansion & "Baby Giant" Frames
* Standard Ethernet II Maximum Frame Size: **$1518\text{ Bytes}$** (without FCS).
* 802.1Q Tagged Maximum Frame Size: **$1522\text{ Bytes}$** ($1518 + 4\text{ bytes tag}$).
* **Legacy Switch Failure:** Legacy hardware that strictly enforces a 1518-byte limit will drop 802.1Q tagged frames as **Giant Frames** (Oversized), causing silent trunk connection failures.

---

### Native VLAN & Security Implications

* **Native VLAN Definition:** On an 802.1Q trunk link, the **Native VLAN** (Default: VLAN 1) carries traffic **without an 802.1Q tag**.
* **The Vulnerability (VLAN Hopping via Double Tagging):**
  If an attacker is connected to an access port matching the Native VLAN of the trunk link, they can execute **802.1Q Double Tagging**:

```
 [ Attacker on Native VLAN 1 ]
   │
   ▼ Constructs Double-Tagged Frame: [ Outer Tag: VLAN 1 ][ Inner Tag: VLAN 20 ][ Payload ]
 [ Switch 1 ]
   │ Sees Outer Tag (VLAN 1 == Native VLAN). Strips outer tag and forwards frame over trunk untagged!
   ▼ Frame now contains ONLY: [ Inner Tag: VLAN 20 ][ Payload ]
 [ Switch 2 ]
   │ Reads Inner Tag (VLAN 20). Forwards frame directly into target VLAN 20!
   ▼
 [ Victim on VLAN 20 Receives Injected Packet (Unidirectional Attack)! ]
```

> 🛡️ **Defensive Hardening Rule:**  
> Always change the Native VLAN to an unused, isolated dummy ID (e.g., VLAN 999) across all trunk ports, and force trunk links to tag the native VLAN: `vlan dot1q tag native`.

---

## 4. Live Layer 2 Terminal Inspection & Diagnostics

```bash
# 1. Inspect the Linux Kernel CAM / Forwarding Database (FDB)
bridge fdb show
# Displays learned MAC addresses, associated network devices, and aging status

# 2. Extract 802.1Q Tagged Frames using tcpdump
sudo tcpdump -i eth0 -e -nn vlan
# Output explicitly exposes the 802.1Q tag:
# 00:0c:29:11:22:33 > 00:50:56:44:55:66, ethertype 802.1Q (0x8100), vlan 10, p 0, ethertype IPv4 (0x0800)

# 3. Inspect Physical Port Hardware MAC & Administrative Settings
ip link show eth0
# Resolves: link/ether 00:0c:29:ab:cd:ef brd ff:ff:ff:ff:ff:ff

# 4. Create a Virtual 802.1Q VLAN Interface in Linux (Tag 20 on eth0)
sudo ip link add link eth0 name eth0.20 type vlan id 20
sudo ip link set up dev eth0.20
```

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->

---

# 🔌 Chapter 02: Layer 1 & Layer 2 — Data Link, Ethernet & ARP (Part 2.3)

> **IW Cyber Ops Research Vault | Module 02: Network Protocols & Traffic Engineering**  
> *Author: Muhammad Imran Wakeel (@iwcyberops)*  
> *Track: Spanning Tree Protocol (STP), BPDU Frame Dissection & Root Hijacking*

---

## 1. The Layer 2 Loop Catastrophe (Why STP Exists)

To provide physical redundancy, enterprise networks deploy multiple physical links between switches. However, redundant Layer 2 links create **Bridging Loops** that can destroy an entire network within seconds.

```
                   ┌──────────────┐
                   │   Switch A   │
                   └──┬────────┬──┘
                      │        │  (Redundant Physical Links)
           Broadcast  │        │  Loops Infinitely!
             Flood    │        │
                   ┌──┴────────┴──┐
                   │   Switch B   │
                   └──────────────┘
```

---

### Why Layer 2 Loops Are Fatal (The Missing TTL Reality)

Unlike Layer 3 IPv4 packets which feature a **Time-To-Live (TTL)** field that decrements at each hop to kill routing loops, **Ethernet II frames have NO TTL or Hop Limit field**.

Once an Ethernet frame enters a Layer 2 loop, it circulates infinitely until hardware power is cut or a cable is disconnected, triggering three fatal failure modes:

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        LAYER 2 LOOP FAILURE TAXONOMY                                   │
├─────────────────────────┬──────────────────────────────────────────────────────────────┤
│ 1. Broadcast Storms     │ Broadcast frames (e.g., ARP Requests) are replicated by all  │
│                         │ switches out every port, consuming 100% of link bandwidth.   │
├─────────────────────────┼──────────────────────────────────────────────────────────────┤
│ 2. CAM Table Thrashing  │ Switches ingest identical frames from alternating ports every│
│    (MAC Instability)    │ millisecond, overwriting their CAM tables constantly and     │
│                         │ locking switch ASIC CPUs at 100% utilization.                │
├─────────────────────────┼──────────────────────────────────────────────────────────────┤
│ 3. Multiple Frame Copies│ End-host endpoints receive thousands of duplicate copies of  │
│                         │ the same unicast payload, crashing socket queues.            │
└─────────────────────────┴──────────────────────────────────────────────────────────────┘
```

---

## 2. Spanning Tree Protocol (IEEE 802.1D / 802.1w RSTP)

Invented by **Radia Perlman**, the **Spanning Tree Protocol (STP)** uses graph theory to logically break loops by calculating an acyclic minimum spanning tree across physical networks. Redundant links are dynamically placed into a **Blocking (Standby)** state and brought online only if an active link fails.

```
       PHYSICAL TOPOLOGY (Redundant Loop)               LOGICAL STP TOPOLOGY (Loop-Free Tree)
             ┌──────────┐                                     ┌──────────┐
             │ Switch A │ (Root Bridge)                       │ Switch A │ (Root Bridge)
             └──┬────┬──┘                                     └──┬────┬──┘
                │    │                                           │    │
       Forward  │    │ Forward                          Forward  │    │ Forward
                │    │                                           │    │
             ┌──┴────┴──┐                                     ┌──┴────┴──┐
             │ Switch B │                                     │ Switch B │
             └──┬────┬──┘                                     └──┬───────┘
                │    │ (Redundant Link)                          │    X (BLOCKING PORT: Logically
                └────┘                                           └────┘  Severed to Prevent Loops)
```

---

### 1. STP Port Roles
* **Root Bridge:** The master logical reference switch for the entire Layer 2 domain. All active ports on the Root Bridge are **Designated Ports** (Forwarding).
* **Root Port (RP):** The single port on a non-root switch with the lowest administrative path cost to reach the Root Bridge.
* **Designated Port (DP):** The port on a network segment that has the best path cost toward the Root Bridge. Forwards traffic.
* **Non-Designated / Alternate Port (AP):** Blocked port. Drops all data frames; listens only to STP control messages.

---

### 2. Classical 802.1D Port State Machine & Convergence Timers
To transition from a redundant link to an active link without creating temporary micro-loops, legacy 802.1D forces ports through a slow **30 to 50-second state transition cycle**:

```
 [ BLOCKING ] ──( 20s Max Age )──> [ LISTENING ] ──( 15s Forward Delay )──> [ LEARNING ] ──( 15s Forward Delay )──> [ FORWARDING ]
 (Drops Data,                      (Evaluates BPDUs,                       (Learns MAC addresses,                  (Forwards frames,
  Listens BPDUs)                    Cannot learn MACs)                      Builds CAM table)                       Full Active State)
```

* **802.1w Rapid STP (RSTP):** Replaced legacy timers with an active **Proposal/Agreement handshake**, reducing convergence times from 50 seconds down to **sub-second ($<1\text{ second}$)** transitions.

---

## 3. BPDU Frame Anatomy & Dissection

Switches discover network topology and elect the Root Bridge by exchanging specialized Layer 2 control messages called **Bridge Protocol Data Units (BPDUs)** every $2\text{ seconds}$ by default.

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        STP CONFIGURATION BPDU ON-THE-WIRE                              │
├─────────────────┬─────────────────┬───────────┬────────────────────────────────────────┤
│ Dest MAC        │ Src MAC         │ LLC Encaps│ BPDU PAYLOAD (35 BYTES)                │
│ 01:80:C2:00:00:00│ Switch Port MAC │ 3 Bytes   │ Root ID, Path Cost, Bridge ID, Timers  │
├─────────────────┴─────────────────┴───────────┼────────────────────────────────────────┤
│ (IEEE Reserved Spanning Tree Multicast)       │ (Logical Link Control: 0x42 0x42 0x03) │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

---

### BPDU Payload Fields (Byte-by-Byte Breakdown)

```text
Field Name                 Size       Description
─────────────────────────────────────────────────────────────────────────────────────────
Protocol Identifier        2 Bytes    Always 0x0000 (IEEE 802.1D Spanning Tree)
Protocol Version           1 Byte     0x00 = Classic STP (802.1D), 0x02 = RSTP (802.1w)
BPDU Type                  1 Byte     0x00 = Configuration BPDU, 0x80 = Topology Change (TCN)
Flags                      1 Byte     Bit 0: Topology Change (TC), Bit 7: TC Acknowledgment
Root Identifier (BID)      8 Bytes    Bridge ID of the assumed Root Bridge
Root Path Cost             4 Bytes    Cumulative metric cost to reach the Root Bridge
Bridge Identifier (BID)    8 Bytes    Bridge ID of the switch transmitting this specific BPDU
Port Identifier            2 Bytes    Port Priority (1 Byte) + Port Number (1 Byte)
Message Age                2 Bytes    Time elapsed since the Root Bridge generated this BPDU
Max Age                    2 Bytes    Timeout limit before BPDU expires (Default: 20 seconds)
Hello Time                 2 Bytes    Interval between configuration BPDUs (Default: 2 seconds)
Forward Delay              2 Bytes    Time spent in Listening and Learning states (Default: 15s)
```

---

### The Bridge Identifier (BID) Architecture
The 8-byte **Bridge ID (BID)** determines the Root Bridge election:

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                           8-BYTE BRIDGE IDENTIFIER (BID)                               │
├───────────────────────────────────────────┬────────────────────────────────────────────┤
│ 2 BYTES: BRIDGE PRIORITY                  │ 6 BYTES: SYSTEM MAC ADDRESS                │
├────────────────────┬──────────────────────┼────────────────────────────────────────────┤
│ Priority Multiplier│ Extended System ID   │ Base MAC Address of Switch Hardware        │
│ 4 Bits (Steps 4096)│ 12 Bits (VLAN ID)    │ Example: 00:0C:29:AB:CD:EF                 │
└────────────────────┴──────────────────────┴────────────────────────────────────────────┘
```

* **Default Priority:** `32768` (Configurable in steps of $4096$: $0, 4096, 8192 \dots 61440$).
* **Root Bridge Election Rule:** The switch with the **LOWEST Bridge ID (BID)** wins the election.
  1. Compares **Priority**: Lower priority wins.
  2. If Priorities are tied: Compares **MAC Address**: Lower numerical MAC address wins.

---

## 4. Offensive Operations: STP Root Hijacking (MITM Attack)

Because legacy STP lacks cryptographic authentication, switches trust all received BPDUs implicitly. An attacker on an unhardened access port can inject weaponized BPDUs to seize the Root Bridge role.

```
       LEGITIMATE TOPOLOGY                              MALICIOUS ROOT HIJACK ATTACK
       ┌──────────────────┐                              ┌──────────────────┐
       │ Core Switch A    │ (Root Bridge)                │ Core Switch A    │ (Loses Root Status)
       └────────┬─────────┘                              └────────┬─────────┘
                │                                                 │
                ▼                                                 ▼
       ┌──────────────────┐                              ┌──────────────────┐
       │ Access Switch B  │                              │ Access Switch B  │
       └────────┬─────────┘                              └────────┬─────────┘
                │                                                 │
                ▼                                                 ▼
       [ Victim Client ]                                 [ Attacker Laptop ] ──( Injects BPDU with )
                                                         ( Priority: 0     )   ( Priority 0 & Low MAC! )
                                                         ( Becomes ROOT!   )
                                                         All L2 Inter-Switch Traffic Routes to Attacker!
```

---

### Execution Mechanics (Using Scapy or Yersinia)
1. The attacker attaches to an access port on an unhardened switch.
2. The attacker uses packet crafting engines to transmit continuous Configuration BPDUs advertising:
   * **Root Priority:** `0` (Absolute lowest possible value).
   * **Root MAC:** `00:00:00:00:00:01` (Lowest possible hardware address).
   * **Root Path Cost:** `0`.
3. The legitimate switches compare their current Root Bridge (`Priority: 32768`) against the attacker's BPDU (`Priority: 0`).
4. **The Network Re-converges:** The legitimate switches step down, elect the **attacker as the new Root Bridge**, and transition their links facing the attacker into **Forwarding (Root Ports)**.
5. **Impact:** The entire enterprise Layer 2 traffic flow pivots toward the attacker's machine, enabling transparent **Man-in-the-Middle (MITM) sniffing and traffic modification**.

---

### The Topology Change Notification (TCN) DoS Flood
* **Mechanics:** An attacker floods continuous **TCN (Topology Change Notification)** BPDUs into the network.
* **Impact:** Every time a switch receives a TCN, it reduces its **CAM table aging timer from 300 seconds down to 15 seconds (Forward Delay)**.
* **Result:** The switch permanently flushes its learned MAC entries, falling back to **Unknown Unicast Flooding** on all ports, degrading switch performance and exposing traffic to sniffing.

---

## 5. Defensive Hardening: STP Control Mitigation

Enterprise networks mitigate Layer 2 STP hijacking by enforcing three essential switchport hardening controls:

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        STP DEFENSIVE HARDENING CONTROLS                                │
├───────────────────┬────────────────────────────────────────────────────────────────────┤
│ **BPDU Guard**    │ Configured on all Access Ports facing endpoints. If ANY BPDU frame │
│                   │ is received on the port, the switch immediately disables the port  │
│                   │ (**err-disable** state), neutralizing rogue Root Bridge injectors. │
├───────────────────┼────────────────────────────────────────────────────────────────────┤
│ **Root Guard**    │ Configured on trunk ports facing downstream switches. Prevents     │
│                   │ that specific port from ever becoming a Root Port. If a superior   │
│                   │ BPDU is received, the port transitions to **Root-Inconsistent**     │
│                   │ (Blocking) state until the rogue BPDUs stop.                       │
├───────────────────┼────────────────────────────────────────────────────────────────────┤
│ **BPDU Filter**   │ Completely suppresses sending or processing BPDUs on edge ports.   │
└───────────────────┴────────────────────────────────────────────────────────────────────┘
```

---

## 6. Live BPDU Dissection & Terminal Diagnostics

```bash
# 1. Capture Raw STP Configuration BPDUs via tcpdump
sudo tcpdump -i eth0 -nn -vvv -c 1 'ether dst 01:80:c2:00:00:00'

# Example Wire Capture Output:
# 00:0c:29:ab:cd:ef > 01:80:c2:00:00:00, 802.3, length 43: LLC, dsap STP (0x42) Individual, ssap STP (0x42) Command, ctrl 0x03:
#   STP 802.1d, Config, Flags [none], bridge-id 8000.00:0c:29:ab:cd:ef.8001, length 43
#     message-age 0.00s, max-age 20.00s, hello-time 2.00s, forward-delay 15.00s
#     root-id 8000.00:0c:29:ab:cd:ef, root-pathcost 0, port-role Designated

# 2. Inspect Linux Kernel Bridge STP State (If host runs bridge interfaces)
bridge link show
brctl showstp br0 2>/dev/null || true
```

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->

---

# 🔌 Chapter 02: Layer 1 & Layer 2 — Data Link, Ethernet & ARP (Part 2.4)

> **IW Cyber Ops Research Vault | Module 02: Network Protocols & Traffic Engineering**  
> *Author: Muhammad Imran Wakeel (@iwcyberops)*  
> *Track: RFC 826 ARP Internals, Cache Poisoning MITM, CAM Flooding & Port Security*

---

## 1. Address Resolution Protocol (RFC 826 Specification)

Because Layer 3 IPv4 addresses are logical abstractions, network interfaces cannot deliver data over physical cables without knowing the destination hardware **Layer 2 MAC Address**. 

**ARP (Address Resolution Protocol)** dynamically maps 32-bit logical IPv4 addresses to 48-bit physical MAC addresses across local broadcast domains.

```
       [ Host A (192.168.1.10) ]                         [ Host B (192.168.1.50) ]
                   │                                                 │
                   │ 1. ARP REQUEST (Broadcast: FF:FF:FF:FF:FF:FF)   │
                   │    "Who has 192.168.1.50? Tell 192.168.1.10"   │
                   ├────────────────────────────────────────────────►│
                   │                                                 │
                   │ 2. ARP REPLY (Unicast: Directed to Host A MAC)  │
                   │    "192.168.1.50 is at 00:0C:29:AA:BB:CC"       │
                   │◄────────────────────────────────────────────────┤
                   ▼                                                 ▼
        [ Updates ARP Cache ]                             [ Updates ARP Cache ]
```

---

### RFC 826 Packet Structure (28 Bytes)

The ARP payload is encapsulated directly inside an Ethernet II frame with EtherType **`0x0806`**:

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        RFC 826 ARP PACKET WIRE FORMAT (28 BYTES)                       │
├──────────────────────────┬──────────────────────────┬─────────────┬────────────────────┤
│ Hardware Type (HTYPE)    │ Protocol Type (PTYPE)    │ HLEN (1B)   │ PLEN (1B)          │
│ 2 Bytes: 0x0001 (Ethernet│ 2 Bytes: 0x0800 (IPv4)   │ 0x06 (MAC)  │ 0x04 (IPv4)        │
├──────────────────────────┴──────────────────────────┼─────────────┴────────────────────┤
│ Opcode (OPER) — 2 Bytes: 0x0001 (Req) / 0x0002 (Rep)│ Sender MAC (SHA) — 6 Bytes       │
├─────────────────────────────────────────────────────┴──────────────────────────────────┤
│ Sender IP (SPA) — 4 Bytes                           │ Target MAC (THA) — 6 Bytes       │
├─────────────────────────────────────────────────────┴──────────────────────────────────┤
│ Target IP (TPA) — 4 Bytes                           │ (In Request: THA is 00:00:00...) │
└─────────────────────────────────────────────────────┴──────────────────────────────────┘
```

#### Field-by-Field Breakdown:
1. **Hardware Type (HTYPE — 16 Bits):** Network protocol family (`0x0001` = Ethernet).
2. **Protocol Type (PTYPE — 16 Bits):** Layer 3 protocol being resolved (`0x0800` = IPv4).
3. **Hardware Address Length (HLEN — 8 Bits):** Byte length of MAC address (`6`).
4. **Protocol Address Length (PLEN — 8 Bits):** Byte length of IP address (`4`).
5. **Opcode (OPER — 16 Bits):** Defines packet function:
   * `0x0001` $\longrightarrow$ **ARP Request**
   * `0x0002` $\longrightarrow$ **ARP Reply**
   * `0x0003` $\longrightarrow$ RARP Request (Reverse ARP - Legacy)
   * `0x0004` $\longrightarrow$ RARP Reply
6. **Sender Hardware Address (SHA — 48 Bits):** MAC address of the transmitting node.
7. **Sender Protocol Address (SPA — 32 Bits):** IPv4 address of the transmitting node.
8. **Target Hardware Address (THA — 48 Bits):** In an ARP Request, this is set to **`00:00:00:00:00:00`** (Unknown); in an ARP Reply, contains target MAC.
9. **Target Protocol Address (TPA — 32 Bits):** The IPv4 address being resolved.

---

### Gratuitous ARP (GARP)
A **Gratuitous ARP** is an unsolicited ARP broadcast where the **Sender IP equals the Target IP**:
* **Operational Purpose:** Broadcast upon interface startup to announce IP presence and detect **IP address conflicts**.
* **Clustering Failover:** High-availability systems (VRRP, CARP, HSRP) broadcast GARPs to immediately update all switch CAM tables and host ARP caches to point to a backup router MAC during hardware failover.

---

## 2. 🔬 Hexadecimal Dissection: Live ARP Request & Reply

Below is an annotated hex dump of an ARP exchange captured via `tcpdump -xx`:

### 1. ARP Request on the Wire:
```text
ff ff ff ff ff ff 00 0c 29 11 22 33 08 06 00 01 08 00 06 04 00 01 00 0c 29 11 22 33 c0 a8 01 0a 00 00 00 00 00 00 c0 a8 01 32
```

```
 Ethernet Header:
  ff ff ff ff ff ff ──> Destination MAC: Broadcast (All hosts listen)
  00 0c 29 11 22 33 ──> Source MAC: 00:0C:29:11:22:33 (Host A)
  08 06             ──> EtherType: ARP (0x0806)
 ARP Payload:
  00 01             ──> Hardware Type: Ethernet (1)
  08 00             ──> Protocol Type: IPv4 (0x0800)
  06                ──> Hardware Size: 6 Bytes
  04                ──> Protocol Size: 4 Bytes
  00 01             ──> Opcode: ARP Request (1)
  00 0c 29 11 22 33 ──> Sender MAC: 00:0C:29:11:22:33
  c0 a8 01 0a       ──> Sender IP: 192.168.1.10 (Hex c0.a8.01.0a)
  00 00 00 00 00 00 ──> Target MAC: Unknown (00:00:00:00:00:00)
  c0 a8 01 32       ──> Target IP: 192.168.1.50 (Hex c0.a8.01.32)
```

---

## 3. ARP Cache Poisoning (The Stateless Flaw & MITM)

### The Underlying Architectural Vulnerability
RFC 826 was authored in 1982 with **zero authentication mechanisms**:
1. Operating systems are **completely stateless**: hosts update their local ARP cache upon receiving an ARP Reply **even if they never sent an ARP Request** (**Unsolicited ARP Reply**).
2. Endpoints implicitly trust any sender claiming an IP address.

---

### The Bidirectional Man-in-the-Middle (MITM) Execution Flow

```
                                [ ATTACKER (192.168.1.100) ]
                                [ MAC: AA:AA:AA:AA:AA:AA   ]
                                        ▲          │
                    Intercepted Packets │          │ Forwarded Packets
                                        │          ▼
      [ VICTIM (192.168.1.50) ] ◄───────┴──────────┴───────► [ GATEWAY (192.168.1.1) ]
      [ MAC: VV:VV:VV:VV:VV:VV]                               [ MAC: GG:GG:GG:GG:GG:GG]
```

1. **Poisoning the Victim:** The attacker transmits continuous unsolicited ARP Replies to Victim (`192.168.1.50`):  
   $$\text{"192.168.1.1 is at AA:AA:AA:AA:AA:AA"}$$
   * *Result:* The victim overwrites its ARP cache, binding the Gateway’s IP to the **Attacker’s MAC**.
2. **Poisoning the Gateway:** The attacker transmits continuous unsolicited ARP Replies to Gateway (`192.168.1.1`):  
   $$\text{"192.168.1.50 is at AA:AA:AA:AA:AA:AA"}$$
   * *Result:* The gateway overwrites its ARP cache, binding the Victim’s IP to the **Attacker’s MAC**.
3. **Kernel Forwarding:** The attacker enables IP forwarding in the Linux kernel:
   ```bash
   sudo sysctl -w net.ipv4.ip_forward=1
   ```
4. **Impact:** All outbound and inbound traffic between the Victim and the Internet now passes transparently through the Attacker's network interface, enabling unencrypted payload sniffing, DNS spoofing, and credential harvesting.

---

## 4. MAC Flooding & CAM Table Overflow Attacks

While ARP poisoning targets endpoint operating systems, **MAC Flooding** targets the physical Layer 2 switch hardware.

```
  [ Attacker running macof ] ──( Floods 100,000+ random MACs/sec )──> [ SWITCH CAM TABLE ]
                                                                                │
                                                                                ▼
                                                                     [ CAM MEMORY EXHAUSTION! ]
                                                                                │
                                                                                ▼
                                                                     [ SWITCH FAILS OPEN! ]
                                                                     Behaves like a legacy HUB;
                                                                     floods all unicast traffic!
```

---

### Mechanics of the Switch "Fail-Open" State:
1. Physical switches have finite memory allocated to their ASIC CAM tables (typically $8,000$ to $128,000$ entries).
2. The attacker uses tools like `macof` to generate tens of thousands of frames per second with randomized, fake Source MAC addresses.
3. The switch's CAM table fills completely in milliseconds, purging legitimate learned MACs.
4. **Fail-Open Transition:** When legitimate frames arrive, the switch cannot locate the Destination MAC in its depleted CAM table. It is forced into **Unknown Unicast Flooding**, broadcasting every host's private traffic out **every physical port**, allowing the attacker to passively sniff the entire network.

---

## 5. Enterprise Layer 2 Defensive Architecture

Modern enterprise networks implement hardware protections directly in switch firmware to neutralize ARP and MAC attacks:

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        LAYER 2 SECURITY HARDENING MATRIX                               │
├───────────────────┬────────────────────────────────────────────────────────────────────┤
│ **Port Security** │ Enforces a maximum MAC limit on switchports (e.g., maximum 1 MAC). │
│                   │ **Violation Modes:**                                               │
│                   │ - `Protect`: Drops unauthorized frames silently.                   │
│                   │ - `Restrict`: Drops frames and logs an SNMP security alert.        │
│                   │ - `Shutdown`: Instantly disables the port (**err-disable** state). │
├───────────────────┼────────────────────────────────────────────────────────────────────┤
│ **DHCP Snooping** │ Builds a trusted database of `[MAC, IP, Switchport, VLAN]` bindings│
│                   │ by inspecting valid DHCP transactions on untrusted access ports.   │
├───────────────────┼────────────────────────────────────────────────────────────────────┤
│ **DAI**           │ **Dynamic ARP Inspection:** Evaluates all incoming ARP packets     │
│                   │ against the trusted DHCP Snooping database. Drops any ARP Reply    │
│                   │ that attempts to advertise an invalid or mismatched IP-to-MAC map. │
└───────────────────┴────────────────────────────────────────────────────────────────────┘
```

