<!-- 
SEO METADATA & KEYWORDS (Invisible to readers, visible to Google Crawlers)
Keywords: IW Cyber Ops, Muhammad Imran, Network Protocols, Packet Dissection, Traffic Engineering, TCP/IP Architecture, Wireshark Packet Analysis, Network Security, Nmap Scanning Heuristics, Scapy Packet Crafting, Cybersecurity Knowledge Base, Vulnerability Research.
-->

# 🌐 Month 02: Network Protocols, Packet Dissection & Traffic Engineering

> **Knowledge Base Directory:** Phase 01 / Month 02  
> **System Operator & Author:** Muhammad Imran (Founder, **IW Cyber Ops**)  
> **Objective:** Understand network communication at the raw packet byte level, analyze state machines, and master network traffic diagnostics.

---

## 🏛️ The Imperative of Protocol Mastery

In the domain of offensive security, a network is not merely a medium for data transfer; it is a continuously shifting attack surface. 

You cannot exploit what you cannot see, and you cannot secure what you do not understand. Relying solely on automated tools without comprehending the underlying raw bytes is a dangerous vulnerability in a researcher's methodology. Mastery requires peering beneath the abstractions—stripping away the GUI to look directly at Ethernet frames, IP headers, and TCP control bits.

This directory houses the **IW Cyber Ops Knowledge Base** for Month 02. It documents the transition from high-level networking concepts to granular, byte-level packet dissection. 

Here, we reconstruct the OSI and TCP/IP models from the ground up, craft custom malformed packets, map application protocol state machines, and engineer traffic to bypass modern network filters.

---

## 🧠 Core Domains Documented in this Directory

The notes contained within this module cover the following theoretical and practical pillars:

1. **Encapsulation & Framing:** Dissecting the OSI vs. TCP/IP models, Ethernet II framing, Maximum Transmission Units (MTU), packet fragmentation, and the mechanics of ARP spoofing/cache poisoning.
2. **Layer 3/4 Protocol Mechanics:** Granular analysis of IPv4/IPv6 headers, CIDR subnetting mathematics, ICMP types, the TCP 3-way handshake, sliding windows, sequence tracking, and connectionless UDP dynamics.
3. **Core Application Protocols:** Binary formatting of DNS query/responses, the DHCP DORA state machine, HTTP/1.1 methods and header structures, and TLS 1.3 handshake cryptographic records.
4. **Port Scanning Heuristics:** The low-level mechanics of network reconnaissance, including TCP SYN stealth scanning, UDP ICMP port unreachable dynamics, and OS fingerprinting heuristics.
5. **Packet Engineering & Capture:** Utilizing `tcpdump`, `Wireshark`, and `tshark` for traffic telemetry, alongside `hping3` and `scapy` for crafting custom protocol anomalies.

---

## 📂 Index of Technical Notes

*Below is the living index of all Markdown notes generated during this month's research. Click on any topic to access the detailed documentation.*

| Status | Technical Topic | File Reference |
| :---: | :--- | :--- |
| 📝 | OSI Model, Ethernet Framing & ARP Mechanics | `[01-ethernet-framing-arp.md](./01-ethernet-framing-arp.md)` |
| 📝 | IPv4/IPv6 Headers, Subnetting & Fragmentation | `[02-ip-headers-fragmentation.md](./02-ip-headers-fragmentation.md)` |
| 📝 | TCP/UDP Mechanics & State Machines | `[03-tcp-udp-mechanics.md](./03-tcp-udp-mechanics.md)` |
| 📝 | Application Protocols: DNS, DHCP, HTTP & TLS | `[04-application-protocols.md](./04-application-protocols.md)` |
| 📝 | Network Scanning Heuristics & OS Fingerprinting | `[05-scanning-heuristics-nmap.md](./05-scanning-heuristics-nmap.md)` |
| 📝 | Packet Crafting & Manipulation with Scapy | `[06-packet-crafting-scapy.md](./06-packet-crafting-scapy.md)` |

*(Note: As the month progresses, new `.md` files will be added to this folder and linked above.)*

---

## 🛡️ About the Author

**Muhammad Imran** is an independent systems researcher and the Founder of **IW Cyber Ops**. This knowledge base is an active repository complementing a rigorous 42-month journey engineered for absolute depth, intellectual rigor, and high-impact vulnerability research.

To view the complete overarching roadmap, visit the official [IW-Mission-Control](https://github.com/iwcyberops/IW-Mission-Control) repository.

<br>

---
*Generated & Curated by **IW Cyber Ops** | High-Assurance Cyber Operations & Research*
