# 🌐 Month 07: Advanced Web Security I – Server-Side Vulnerability Analysis

> **Research Track:** Phase 01 — Foundations & Systems Architecture  
> **Author & Lead Researcher:** Muhammad Imran (Founder, **IW Cyber Ops**)  
> **Knowledge Base Domain:** `IW-Knowledge-Base/m07-server-side-web-security`

---

## 🧭 Why Server-Side Vulnerability Analysis & Filter Bypassing Matter

Modern enterprise applications sirf static web pages nahi hotin; wo complex server-side backends, databases, cloud APIs, aur microservices ke mesh par chalti hain.

Automated web scanners (jaise ZAP ya Nikto) sirf surface-level flaws pakad pate hain. Asal high-severity impact (Remote Code Execution, Cloud Infrastructure Takeover, Database Breaches) **manual vulnerability analysis aur filter bypassing** se hasil hota hai. Is month me hum seekhenge:
1. **Advanced SQL Injection:** Boolean/Time-based blind extraction channels, out-of-band (OAST) exfiltration, second-order SQLi, aur WAF keyword filter bypasses.
2. **Broken Object Level Authorization (BOLA/IDOR):** Complex multi-tenant authorization matrix bypasses, predictable UUID patterns, aur state-changing HTTP verb manipulation.
3. **Server-Side Request Forgery (SSRF):** DNS rebinding mechanics, cloud metadata API exploitation (AWS IMDSv1 vs. IMDSv2, Azure, GCP), aur protocol smuggling via `gopher://`.
4. **XML External Entity (XXE):** XML parser vulnerabilities, local system entity disclosures, Billion Laughs DoS, aur blind OAST exfiltration via external DTDs.
5. **Server-Side Template Injection (SSTI):** Template engine fingerprinting (Jinja2, Twig, Freemarker, Velocity), sandbox escape dynamics, aur achieving native RCE.

Ye month server-side logic flaws aur input handling boundaries ko master karne ka pivotal stage hai.

---

## 📚 Month 07 Knowledge Base & Topic Notes Directory

Is folder me Month 07 ke dauran banaye gaye tamam technical notes topics ke mutabiq categorized hain:

| Note File | Core Focus & Concepts Covered | Status |
| :--- | :--- | :---: |
| 📄 **[`01-advanced-sqli-oast-bypass.md`](./01-advanced-sqli-oast-bypass.md)** | Blind boolean/timing channels, out-of-band DNS/HTTP exfiltration, second-order injection, and WAF evasion. | 🟢 Completed |
| 📄 **[`02-bola-idor-access-control.md`](./02-bola-idor-access-control.md)** | Multi-tenant authorization bypasses, horizontal/vertical privilege escalation, IDOR arrays, and verb tampering. | 🟢 Completed |
| 📄 **[`03-ssrf-cloud-metadata-exploitation.md`](./03-ssrf-cloud-metadata-exploitation.md)** | DNS rebinding, AWS IMDSv1 vs IMDSv2 token headers, cloud instance takeover, and `gopher://` protocol smuggling. | 🟢 Completed |
| 📄 **[`04-xxe-xml-injection-attacks.md`](./04-xxe-xml-injection-attacks.md)** | XML entity declaration, local file disclosure, parameter entity exploitation, external DTD hosting, and XML parsing sinks. | 🟢 Completed |
| 📄 **[`05-ssti-template-sandbox-escapes.md`](./05-ssti-template-sandbox-escapes.md)** | Template engine fingerprinting, MRO class hierarchy navigation, Python/Java sandbox escapes, and RCE payload mechanics. | 🟢 Completed |

---

## ⚙️ The 3 Daily Continuous Side Tracks (Month 07 Focus)

Hamare daily 12-hour engine ke 3 parallel tracks ke dedicated notes:

### 1. 💻 Systems C Track (1 Hour Daily)
* **Goal:** Input sanitization and memory-safe binary parsers.
* **Topics:** Writing custom C input parsing algorithms, validating string lengths before buffer copy, and preventing off-by-one boundary vulnerabilities.

### 2. 🧩 Assembly & Reverse Engineering Track (1 Hour Daily)
* **Goal:** Reversing compiled web backends and server modules.
* **Topics:** Dissecting compiled C-based web server extensions (e.g., custom NGINX/Apache modules) inside Ghidra to trace input parsing logic.

### 3. 🔌 Hardware & Architecture Track (1 Hour Daily)
* **Goal:** Physical network processing and hardware interfaces.
* **Topics:** Physical Network Interface Cards (NICs), hardware MAC address filtering, DMA ring buffers, and line-rate hardware packet processing.

---

## 🛠️ Lab Environments & Hands-On Milestones

* 🎯 **Custom Automated Blind SQLi Engine:** Asynchronous Python tool extracting database tables via binary search timing attacks with statistical jitter correction.
* 🎯 **Deliberately Vulnerable Polyglot Web App:** Python Flask application demonstrating intentional SSTI, SSRF, and complex multi-tenant IDOR bugs.
* 🎯 **PortSwigger Certified Practitioner Lab Clear:** Comprehensive completion and writeups of all Server-Side vulnerability modules on PortSwigger Web Security Academy.

---

## 📖 Primary Learning References
* 💻 *PortSwigger Web Security Academy*
* 📖 *OWASP Web Security Testing Guide (WSTG)*
* 📖 *The Web Application Hacker’s Handbook* — Dafydd Stuttard & Marcus Pinto

---

© **Muhammad Imran (Founder, IW Cyber Ops)** | Documented for Absolute Depth & Intellectual R
