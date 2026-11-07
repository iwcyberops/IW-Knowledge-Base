<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran Wakeel | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 02 — Network Protocols, Packet Dissection & Traffic Engineering
   DOCUMENT: Chapter 06 — Packet Crafting & Traffic Analysis Tools (Part 6.1)
   ========================================================================= -->

# 🛠️ Chapter 06: Packet Crafting & Traffic Analysis Tools (Part 6.1)

> **IW Cyber Ops Research Vault | Module 02: Network Protocols & Traffic Engineering**  
> *Author: Muhammad Imran Wakeel (@iwcyberops)*  
> *Track: Wireshark Internal Architecture, Dissector Engines, BPF vs Display Filters & File Carving*

---

## 1. Wireshark Internal Architecture & Dissection Pipeline

**Wireshark** is the industry-standard network protocol analyzer. It does not capture packets directly; it interfaces with kernel capture drivers to read raw packet streams and reconstructs them into hierarchical protocol trees using its internal dissection engine (**`epan` — Ethereal Packet Analyzer**).

```
  [ Physical Wire / NIC ]
            │
            ▼ (Promiscuous Mode Ingress)
  [ Kernel Packet Filter (BPF) ] ──> Evaluates Capture Filter (Drops non-matching packets in kernel)
            │
            ▼ (Direct Memory Access / Memory Map)
  [ libpcap / Npcap Driver ]
            │
            ▼ (Wiretap Library: Parses PCAP / PCAPNG formats)
  ┌────────────────────────────────────────────────────────┐
  │         WIRESHARK CORE ENGINE (libwireshark / epan)    │
  ├────────────────────────────────────────────────────────┤
  │ 1. Frame Ingestion & Timestamp Alignment               │
  │ 2. Dissector Tree Evaluation (L2 ──> L3 ──> L4 ──> L7) │
  │ 3. Protocol State Tracking & TCP Stream Reassembly     │
  │ 4. Post-Dissection Display Filter Application          │
  └──────────────────────────┬─────────────────────────────┘
                             │
                             ▼
  [ Graphical UI / Qt Interface / Packet Details Pane ]
```

---

### The Hierarchical Dissector Tree Engine
When a raw packet arrives, Wireshark traverses a dynamic chain of **Protocol Dissectors**:

1. **Root Dissector:** Inspects the physical link-layer header (e.g., Ethernet II).
2. **Handoff Mechanism:** Reads the `EtherType` field (`0x0800`). The table maps `0x0800` directly to the **IPv4 Dissector**.
3. **Network Layer Dissection:** IPv4 dissector parses the IP header and reads the `Protocol` field (`6`). The table hands execution off to the **TCP Dissector**.
4. **Transport Layer Dissection:** TCP dissector inspects port numbers (`Destination Port: 80`) and routes the payload to the **HTTP Dissector**.
5. **Heuristic Dissectors:** If traffic runs on a non-standard port (e.g., HTTP on Port 8080), heuristic dissectors inspect the initial payload bytes (e.g., checking for strings like `GET /` or `HTTP/1.1`) to dynamically assign the correct protocol dissector.

---

## 2. Capture Filters (BPF) vs Display Filters

One of the most critical operational distinctions in traffic analysis is knowing where filtering occurs:

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        CAPTURE FILTERS vs DISPLAY FILTERS                              │
├───────────────────┬──────────────────────────────────┬─────────────────────────────────┤
│ Parameter         │ Capture Filter (BPF)             │ Display Filter (Wireshark)      │
├───────────────────┼──────────────────────────────────┼─────────────────────────────────┤
│ **Execution Space**│ **Kernel Space (Ring 0)**        │ **User Space (Ring 3)**         │
│ **Execution Point**│ **BEFORE** packet hits RAM       │ **AFTER** full packet dissection│
│ **Syntax Basis**  │ Berkeley Packet Filter (BPF)     │ Wireshark Protocol Field Syntax │
│ **Performance**   │ ⚡ Ultra-fast; drops at line-rate│ 🐢 Memory-heavy; parses all data│
│ **Granularity**   │ Low (Ports, IPs, Prototypes)     │ Extreme (Any protocol bit/field)│
│ **Packet Storage**│ Non-matching packets are **LOST**│ Packets preserved; view hidden  │
└───────────────────┴──────────────────────────────────┴─────────────────────────────────┘
```

---

### Comparison Syntax Matrix

| Operational Goal | Capture Filter (BPF Engine) | Display Filter (Wireshark Post-Parse) |
| :--- | :--- | :--- |
| **Filter by Target IP** | `host 10.10.14.5` | `ip.addr == 10.10.14.5` |
| **Filter by TCP Port** | `port 443` | `tcp.port == 443` |
| **Exclude Subnet** | `not net 192.168.1.0/24` | `!(ip.net == 192.168.1.0/24)` |
| **Isolate SYN Flag** | `tcp[tcpflags] & tcp-syn != 0` | `tcp.flags.syn == 1` |
| **String / Payload Search**| *Complex raw byte offsets* | `http.request.uri contains "admin"` |

---

## 3. TCP Stream Reassembly & Flow Tracking

TCP segments often arrive out-of-order, duplicated, or fragmented across thousands of MTU frames. Analyzing application payloads requires reconstructing the raw stream.

```
 [ Packet 1: SEQ 1-1460 ] ──┐
 [ Packet 3: SEQ 2921-4380] ──┼──> [ WIRESHARK TCP REASSEMBLY ] ──> [ RECONSTRUCTED DATA STREAM ]
 [ Packet 2: SEQ 1461-2920] ──┘      Re-orders by Sequence Number     Complete HTTP / File Payload
```

---

### The "Follow TCP Stream" Mechanism
1. Wireshark indexes connections by their unique 5-tuple:  
   $$\mathbf{\{\text{Protocol, Source IP, Source Port, Destination IP, Destination Port}\}}$$
2. Assigns an internal integer index: `tcp.stream eq 0`, `tcp.stream eq 1`, etc.
3. Strips all Ethernet, IP, and TCP headers, assembling the raw payload buffers in correct sequential byte order based on **Sequence Numbers (`tcp.seq`)**.
4. **Terminal / UI Presentation:**
   * **Red Text:** Data transmitted from **Client to Server**.
   * **Blue Text:** Data transmitted from **Server to Client**.

---

## 4. Advanced Security Display Filters (Threat Hunting & Forensics)

Wireshark display filters provide precise protocol introspection for threat detection:

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        TACTICAL THREAT HUNTING DISPLAY FILTERS                         │
├──────────────────────────┬─────────────────────────────────────────────────────────────┤
│ Target Threat Class      │ High-Efficiency Wireshark Display Filter Syntax             │
├──────────────────────────┼─────────────────────────────────────────────────────────────┤
│ **Cleartext Credentials**│ `http.request.method == "POST" || http.authorization`       │
│                          │ `ftp.request.command == "USER" || ftp.request.command=="PASS"`│
├──────────────────────────┼─────────────────────────────────────────────────────────────┤
│ **TCP Stealth Scans**    │ `tcp.flags == 0x000` (NULL Scan)                            │
│                          │ `tcp.flags == 0x029` (Xmas Scan: FIN+PSH+URG)               │
│                          │ `tcp.flags.syn == 1 && tcp.flags.ack == 0` (Half-Open Probes)│
├──────────────────────────┼─────────────────────────────────────────────────────────────┤
│ **RST Tear-Down Attacks**│ `tcp.flags.reset == 1 && tcp.seq > 0`                       │
├──────────────────────────┼─────────────────────────────────────────────────────────────┤
│ **DNS Exfiltration**     │ `dns.flags.response == 0 && string(dns.qry.name) matches`  │
│                          │ `".{30,}"` (Finds unusually long encoded subdomain queries) │
├──────────────────────────┼─────────────────────────────────────────────────────────────┤
│ **TLS Handshake Inspection** `tls.handshake.type == 1` (ClientHello)                   │
│                          │ `tls.handshake.extension.type == 0` (Inspects SNI domains)  │
├──────────────────────────┼─────────────────────────────────────────────────────────────┤
│ **Zero-Window Denial**   │ `tcp.analysis.zero_window` (Identifies saturated endpoints) │
├──────────────────────────┼─────────────────────────────────────────────────────────────┤
│ **File Download Carving**│ `http.response.code == 200 && http.content_type contains`   │
│                          │ `"application"`                                             │
└──────────────────────────┴─────────────────────────────────────────────────────────────┘
```

---

### Advanced Byte-Slicing Filters

Operators can perform arbitrary byte slices at any protocol offset using the syntax:  
$$\mathbf{\text{protocol}[offset:length]}$$

```bash
# Find all IPv4 packets where the Time-to-Live (TTL) is <= 5 (Traceroute or loop probes):
ip[8:1] <= 5

# Match Ethernet frames where Destination MAC starts with VMware OUI (00:0c:29):
eth.dst[0:3] == 00:0c:29

# Detect HTTP traffic carrying raw Windows PE Executable magic bytes ("MZ" / 0x4D5A):
http.file_data[0:2] == 4d:5a
```

---

## 5. Binary Payload Carving & File Extraction from PCAPs

When analyzing traffic captures containing malware staging, credential dumps, or exfiltrated files, analysts extract the raw files directly from the packet stream without running the target software.

```
  [ Captured PCAP File ] ──> [ Reassemble Application Protocol ] ──> [ Carve Raw File Stream ]
                                 (HTTP, SMB, FTP, TFTP, IMF)             (.exe, .elf, .pdf, .zip)
```

---

### 1. Automated Extraction via Wireshark GUI
* Navigate to: **`File` $\longrightarrow$ `Export Objects` $\longrightarrow$ `[Protocol]`**
  * **HTTP:** Extracts web downloads, staging scripts, images, and HTML bodies.
  * **SMB / SMB2:** Extracts files transferred across Windows network shares.
  * **IMF (Internet Message Format):** Extracts raw email payloads and MIME attachments.
  * **TFTP:** Extracts unencrypted firmware files and router backup configurations.

---

### 2. Manual Hex Carving (When Protocols Are Obfuscated)

If a payload was transferred over a raw non-standard TCP socket (e.g., via Netcat on Port 4444):

1. Right-click any packet in the session $\longrightarrow$ **`Follow` $\longrightarrow$ `TCP Stream`**.
2. Set the bottom display dropdown from **ASCII** to **`Raw`** (Displays pure hexadecimal byte stream).
3. Switch direction to capture only the server-to-client or client-to-server data.
4. Click **`Save as...`** and save the output as `payload.bin`.
5. **Verify File Signature via Linux CLI:**
   ```bash
   file payload.bin
   # Output: ELF 64-bit LSB executable, x86-64, version 1 (SYSV)...
   ```

---

## 6. Live Analysis & Triage Matrix

| Operational Goal | Wireshark Feature / Filter | CLI Diagnostic Equivalent |
| :--- | :--- | :--- |
| **Inspect Protocol Hierarchy**| `Statistics` $\longrightarrow$ `Protocol Hierarchy` | `tshark -r file.pcap -qz io,phs` |
| **Identify Top Talking IPs**  | `Statistics` $\longrightarrow$ `IPv4 Statistics` $\longrightarrow$ `Destinations`| `tshark -r file.pcap -T fields -e ip.src \| sort \| uniq -c \| sort -nr` |
| **Isolate Replayed Traffic**  | `tcp.analysis.retransmission` | Identifies network packet loss and link congestion |
| **Locate Cleartext Post Auth** | `http.request.method == "POST"` | `ngrep -q -W byline 'POST' 'tcp port 80'` |
| **Audit SSL/TLS Server Name** | `tls.handshake.extensions_server_name` | Carves unencrypted SNI domains in transit |

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
