<!-- 
SEO METADATA & KEYWORDS (Invisible to readers, visible to Google Crawlers)
Keywords: IW Cyber Ops, Muhammad Imran, Web Application Security, Server-Side Vulnerabilities, SQL Injection, SSRF, XXE, SSTI, BOLA IDOR, Bug Bounty, Web Exploitation, Cybersecurity Knowledge Base.
-->

# 🕸️ Month 07: Advanced Web Security I – Server-Side Vulnerability Analysis

> **Knowledge Base Directory:** Phase 01 / Month 07  
> **System Operator & Author:** Muhammad Imran (Founder, **IW Cyber Ops**)  
> **Objective:** Develop manual web vulnerability discovery skills, bypass input filters, and construct robust server-side exploits.

---

## 🏛️ The Imperative of Server-Side Exploitation

The modern web is not just HTML and JavaScript; it is a massive orchestration of backend databases, template engines, XML parsers, and internal cloud metadata APIs. 

Automated vulnerability scanners can only catch the lowest-hanging fruit. To truly compromise a target, an elite researcher must manually trace the flow of untrusted user input from the HTTP request all the way down to the backend execution sink. Understanding how backend infrastructure parses data allows a researcher to break logic boundaries, extract sensitive databases blindly, forge server-side requests, and ultimately achieve Remote Code Execution (RCE).

This directory serves as the **IW Cyber Ops Knowledge Base** for Month 07. It documents the systematic deconstruction of the most critical server-side vulnerability classes. We move beyond simple payloads, focusing on timing channels, out-of-band exfiltration, and complex filter bypass techniques.

---

## 🧠 Core Domains Documented in this Directory

The notes contained within this module cover the following theoretical and practical pillars:

1. **Advanced SQL Injection (SQLi):** Exploiting Boolean-based blind extraction, time-based blind timing channels, out-of-band (OAST) exfiltration, second-order SQLi, and bypassing Web Application Firewall (WAF) keyword filters.
2. **Broken Object Level Authorization (BOLA/IDOR):** Mapping authorization matrices, exploiting multi-tenant authorization bypasses, breaking UUID predictability, and manipulating state-changing HTTP verbs.
3. **Server-Side Request Forgery (SSRF):** Bypassing URL blacklists via DNS rebinding, exploiting internal cloud metadata APIs (AWS IMDSv1 vs IMDSv2, GCP, Azure), and executing protocol smuggling via the `gopher://` schema.
4. **XML External Entity (XXE):** Weaponizing XML parsers to execute Billion Laughs Denial of Service (DOS), retrieving local files via system entities, and achieving blind out-of-band exfiltration using parameter entities.
5. **Server-Side Template Injection (SSTI):** Identifying template engines (Jinja2, Twig, Freemarker, Velocity), crafting sandbox escape payloads, and escalating template flaws to Remote Code Execution (RCE).
6. **Physical Network Foundations:** Understanding how physical Network Interface Cards (NICs), MAC filtering, and hardware-level packet processing interact with incoming web requests.

---

## 📂 Index of Technical Notes

*Below is the living index of all Markdown notes generated during this month's research. Click on any topic to access the detailed documentation.*

| Status | Technical Topic | File Reference |
| :---: | :--- | :--- |
| 📝 | Advanced SQLi: Blind, Timing & OAST | `[01-sqli-blind-timing-oast.md](./01-sqli-blind-timing-oast.md)` |
| 📝 | Authorization Flaws: BOLA, IDOR & Matrices | `[02-bola-idor-authorization.md](./02-bola-idor-authorization.md)` |
| 📝 | SSRF: DNS Rebinding & Cloud Metadata Abuse | `[03-ssrf-cloud-metadata-gopher.md](./03-ssrf-cloud-metadata-gopher.md)` |
| 📝 | XXE: Entity Extraction & OOB Exfiltration | `[04-xxe-oob-exfiltration.md](./04-xxe-oob-exfiltration.md)` |
| 📝 | SSTI: Engine Identification & Sandbox Escapes | `[05-ssti-sandbox-escapes-rce.md](./05-ssti-sandbox-escapes-rce.md)` |
| 📝 | NICs, MAC Filtering & Hardware Packet Flows | `[06-nics-mac-filtering-hardware.md](./06-nics-mac-filtering-hardware.md)` |

*(Note: As the month progresses, new `.md` files will be added to this folder and linked above.)*

---

## 🛡️ About the Author

**Muhammad Imran** is an independent systems researcher and the Founder of **IW Cyber Ops**. This knowledge base is an active repository complementing a rigorous 42-month journey engineered for absolute depth, intellectual rigor, and high-impact vulnerability research.

To view the complete overarching roadmap, visit the official [IW-Mission-Control](https://github.com/iwcyberops/IW-Mission-Control) repository.

<br>

---
*Generated & Curated by **IW Cyber Ops** | High-Assurance Cyber Operations & Research*
