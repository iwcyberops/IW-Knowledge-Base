<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran Wakeel | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 02 — Network Protocols, Packet Dissection & Traffic Engineering
   DOCUMENT: Chapter 04 — Layer 4: Transport Layer Architecture & TCP Dissection (Part 4.1)
   ========================================================================= -->

# 🚚 Chapter 04: Layer 4 — Transport Layer (TCP & UDP Deep Dive) (Part 4.1)

> **IW Cyber Ops Research Vault | Module 02: Network Protocols & Traffic Engineering**  
> *Author: Muhammad Imran Wakeel (@iwcyberops)*  
> *Track: RFC 793 TCP Wire Specification, 3-Way Handshake Dynamics & Flag Dissection*

---

## 1. Transmission Control Protocol (RFC 793) Architecture

The **Transmission Control Protocol (TCP)** operates at Layer 4 of the OSI model. Unlike UDP, TCP provides a **connection-oriented, reliable, full-duplex, ordered byte-stream delivery service** across unreliable Layer 3 IP networks.

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Sequence Number                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Acknowledgment Number                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Data |           |U|A|P|R|S|F|                               |
| Offset| Reserved  |R|C|S|S|Y|I|            Window             |
|       |           |G|K|H|T|N|N|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           Checksum            |        Urgent Pointer         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options (0 to 40 Bytes)                    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Padding (Variable 0-3B)                    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
│ ◄──────────────── Base Header: Exactly 20 Bytes ────────────► │
```

---

## 2. Exhaustive Bit-by-Bit Field Dissection

### 1. Source Port & Destination Port (16 Bits Each)
* **Range:** $0$ to $65,535$ ($2^{16} - 1$).
* **Function:** Demultiplexes incoming traffic to bind packets to specific application processes running in user space.
  * `0 - 1023` $\longrightarrow$ **Well-Known / System Ports** (Requires root / `CAP_NET_BIND_SERVICE`).
  * `1024 - 49151` $\longrightarrow$ **Registered Ports** (User applications).
  * `49152 - 65535` $\longrightarrow$ **Ephemeral / Dynamic Ports** (Allocated by the client kernel for outbound connections).

---

### 2. Sequence Number — SEQ (32 Bits)
* **Function:** Tracks the position of the first data byte in this segment relative to the total byte stream.
* **Range:** $0$ to $4,294,967,295$ ($2^{32} - 1$).
* **Roll-Over (Wrap-Around):** When SEQ reaches $2^{32}-1$, it rolls back to $0$. On a $10\text{ Gbps}$ link, wrap-around occurs in **less than 3.5 seconds**!

---

### 3. Acknowledgment Number — ACK (32 Bits)
* **Function:** Informs the sender of the **NEXT expected byte** in the data stream.
* **Cumulative ACK:** An ACK value of $N$ confirms that all bytes up to $N - 1$ have been successfully received and buffered.

---

### 4. Data Offset / Header Length (4 Bits)
* **Unit of Measurement:** Expressed in units of **32-bit words (4-byte blocks)**.
* **Minimum Value:** `0101` (Decimal $5 \implies 5 \times 4 = \mathbf{20\text{ Bytes}}$ — Base header without options).
* **Maximum Value:** `1111` (Decimal $15 \implies 15 \times 4 = \mathbf{60\text{ Bytes}}$ — Base header + $40\text{ bytes}$ options).

---

### 5. The TCP Flag Register (9 Bits in Modern RFCs)

Flags are 1-bit boolean switches that govern TCP connection state transitions and traffic flow:

```
  Bit 0   Bit 1   Bit 2   Bit 3   Bit 4   Bit 5   Bit 6   Bit 7   Bit 8
┌───────┬───────┬───────┬───────┬───────┬───────┬───────┬───────┬───────┐
│  NS   │  CWR  │  ECE  │  URG  │  ACK  │  PSH  │  RST  │  SYN  │  FIN  │
└───────┴───────┴───────┴───────┴───────┴───────┴───────┴───────┴───────┘
```

| Flag | Name | Bit Value | Operational Function |
| :---: | :--- | :---: | :--- |
| **`FIN`** | Finish | `0x001` | Graceful connection termination request. Consumes 1 sequence number. |
| **`SYN`** | Synchronize | `0x002` | Synchronizes Initial Sequence Numbers during connection establishment. |
| **`RST`** | Reset | `0x004` | Forcibly aborts a connection. Drops socket memory immediately. |
| **`PSH`** | Push | `0x008` | Forces the receiving kernel to push buffered data directly to the userland app without waiting for the buffer to fill. |
| **`ACK`** | Acknowledgment| `0x010` | Confirms the `Acknowledgment Number` field is valid. Active on 100% of packets after the initial SYN. |
| **`URG`** | Urgent | `0x020` | Out-of-band data is present (Indicated by the Urgent Pointer). |
| **`ECE`** | ECN-Echo | `0x040` | Signals Explicit Congestion Notification received at the IP layer. |
| **`CWR`** | Congestion Window Reduced | `0x080` | Confirms sender reduced its congestion window in response to ECE. |
| **`NS`**  | ECN-Nonce Sum | `0x100` | RFC 3540 protection against receiver concealment of dropped packets. |

---

### 6. Window Size (16 Bits)
* **Function:** Implements **Flow Control**. Informs the sender how many bytes of buffer space the receiver has available.
* **Maximum Value:** $2^{16} - 1 = \mathbf{65,535\text{ Bytes}}$ (Expanded up to $1\text{ Gigabyte}$ using the **Window Scale Option**).

---

### 7. TCP Checksum & The Pseudo-Header (16 Bits)

Unlike the IPv4 header checksum (which covers only the IP header), the **TCP Checksum covers the TCP Header, the TCP Payload, AND an in-memory 12-byte IPv4 Pseudo-Header**:

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        IN-MEMORY TCP PSEUDO-HEADER (12 BYTES)                          │
├───────────────────────────────────────────┬────────────────────────────────────────────┤
│ 32-Bit Source IP Address                  │ 32-Bit Destination IP Address              │
├─────────────────────┬─────────────────────┼────────────────────────────────────────────┤
│ Zero (8 Bits: 0x00) │ Protocol (8B: 0x06) │ TCP Segment Length (16 Bits: Hdr + Payload)│
└─────────────────────┴─────────────────────┴────────────────────────────────────────────┘
```

> 💡 **Why the Pseudo-Header Exists:**  
> It prevents **misrouted packets**. If an intermediate router accidentally corrupts the Destination IP address in the Layer 3 header, the receiving host detects that the pseudo-header checksum does not match and discards the packet, preventing data from leaking into the wrong application socket!

---

### 8. TCP Options (0 to 40 Bytes)

Options are negotiated primarily during the 3-Way Handshake to establish advanced capabilities:

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        COMMON TCP OPTIONS SPECIFICATIONS                               │
├──────┬─────────┬──────────────────────┬────────────────────────────────────────────────┤
│ Kind │ Length  │ Option Name          │ Operational Purpose                            │
├──────┼─────────┼──────────────────────┼────────────────────────────────────────────────┤
│ `0`  │ 1 Byte  │ **EOL**              │ End of Option List (Padding terminator)        │
│ `1`  │ 1 Byte  │ **NOP**              │ No-Operation (Used for 32-bit boundary alignment)│
│ `2`  │ 4 Bytes │ **MSS**              │ Maximum Segment Size (e.g., 1460 Bytes)        │
│ `3`  │ 3 Bytes │ **Window Scale**     │ Bit-shift multiplier (WSOPT) for 1GB windows   │
│ `4`  │ 2 Bytes │ **SACK-Permitted**   │ Selective Acknowledgment enabled               │
│ `8`  │ 10 Bytes│ **Timestamps (TSopt)│ RTT calculation and PAWS (Anti-Wrap protection)│
└──────┴─────────┴──────────────────────┴────────────────────────────────────────────────┘
```

---

## 3. The TCP 3-Way Handshake Mechanics

To establish a bidirectional, full-duplex session, the client and server must synchronize their Initial Sequence Numbers (**ISN**):

```
       [ CLIENT: 192.168.1.50 ]                         [ SERVER: 93.184.216.34 ]
       (State: CLOSED)                                  (State: LISTEN on Port 80)
              │                                                     │
              │ 1. [SYN]                                            │
              │    SEQ = X (Client ISN), ACK = 0, Flags = [SYN]     │
              │    Options: MSS=1460, WS=7, SACK-Permitted          │
              ├────────────────────────────────────────────────────►│ (State: SYN_RECEIVED)
              │                                                     │ Allocates Backlog Queue Memory
              │ 2. [SYN-ACK]                                        │
              │    SEQ = Y (Server ISN), ACK = X + 1                │
              │    Flags = [SYN, ACK]                               │
              │◄────────────────────────────────────────────────────┤
              │                                                     │
              │ 3. [ACK]                                            │
              │    SEQ = X + 1, ACK = Y + 1                         │
              │    Flags = [ACK]                                    │
              ├────────────────────────────────────────────────────►│
              ▼                                                     ▼
      (State: ESTABLISHED)                                  (State: ESTABLISHED)
```

---

### The Phantom Byte (Why SYN and FIN Consume a Sequence Number)

Notice that in Step 2, the server acknowledges the SYN with `ACK = X + 1`, even though the SYN packet carried **0 bytes of user payload data**!

* **The Rule:** Any control flag that alters connection state (**`SYN`** or **`FIN`**) logically consumes **exactly 1 Sequence Number (a "Phantom Byte")**.
* **Reason:** This guarantees that the SYN and FIN flags are acknowledged reliably. If the ACK did not increment, the sender would never know if the SYN itself was received or lost!

---

### 🛡️ Initial Sequence Number (ISN) Randomization (RFC 6528)

In early TCP implementations (RFC 793), the ISN was a simple global counter incremented every 4 microseconds.
* **The Vulnerability (Kevin Mitnick / Blind TCP Hijacking):** Attackers could predict the exact next ISN ($Y$) of a remote server, spoof a trusted IP address, and complete a "blind" 3-way handshake without ever seeing the server's SYN-ACK response!
* **The Modern Fix (RFC 6528):** Modern kernels compute the ISN using a cryptographic pseudo-random function:

$$\mathbf{\text{ISN} = M + F(\text{Local IP}, \text{Remote IP}, \text{Local Port}, \text{Remote Port}, \text{Secret Key})}$$

Where $M$ is a 4-microsecond monotonic clock counter and $F$ is a cryptographic hash (e.g., SipHash or MD5) seeded with a secret boot-time kernel key. This renders the ISN completely unpredictable to external attackers.

---

## 4. 🔬 Raw Wire Hex Dissection: Live TCP SYN Segment

Below is an annotated hex dump of an initial TCP SYN packet captured via `tcpdump -xx`:

```text
c0 01 00 50 1a 2b 3c 4d 00 00 00 00 a0 02 72 10 3b a5 00 00 02 04 05 b4 04 02 08 0a 01 23 45 67 00 00 00 00 01 03 03 07
```

```
 c0 01 │ 00 50 │ 1a 2b 3c 4d │ 00 00 00 00 │ a0 02 │ 72 10 │ 3b a5 │ 00 00 │ Options...
 ──────┼───────┼─────────────┼─────────────┼───────┼───────┼───────┼───────┼───────────
  Src   Dest     Sequence      Acknowledgment Offset   Window  Check   Urgent   MSS, SACK,
  Port  Port     Number        Number         & Flags  Size    sum     Pointer  Timestamps
 (49153)(80)     (0x1a2b3c4d)  (0x00000000)   (SYN)    (29200) (0x3ba5)(0)      (20 Bytes)
```

* `c0 01` $\longrightarrow$ Source Port: `49153` (Ephemeral).
* `00 50` $\longrightarrow$ Destination Port: `80` (HTTP).
* `1a 2b 3c 4d` $\longrightarrow$ Sequence Number: `439,041,101` (Random ISN).
* `00 00 00 00` $\longrightarrow$ Acknowledgment Number: `0` (Unset during initial SYN).
* `a0 02` $\longrightarrow$
  * `a` (Binary `1010` = Decimal 10): Data Offset = $10 \times 4 = \mathbf{40\text{ Bytes}}$ total header.
  * `0 02` (Binary `0000 0000 0010`): Flag **SYN = 1** (Bit 1 set).
* `72 10` $\longrightarrow$ Window Size: `29,200` Bytes.
* `3b a5` $\longrightarrow$ TCP Checksum: `0x3BA5`.
* `02 04 05 b4` $\longrightarrow$ Option: **MSS** (`Kind 2`, `Len 4`, `Value: 0x05B4` = **1460 Bytes**).

