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
