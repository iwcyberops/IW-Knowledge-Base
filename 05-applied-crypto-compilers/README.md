# 🔐 Month 05: Applied Cryptography, Web Foundations & Compilation Pipelines

> **Research Track:** Phase 01 — Foundations & Systems Architecture  
> **Author & Lead Researcher:** Muhammad Imran (Founder, **IW Cyber Ops**)  
> **Knowledge Base Domain:** `IW-Knowledge-Base/m05-applied-crypto-compilers`

---

## 🧭 Why Cryptography, Web Internals & Compilers Matter

Ek elite researcher ke paas sirf ek domain ka ilm kafi nahi hota. Asal taqat tab aati hai jab aap **Cryptographic math, Web protocols, aur Compiler binary emission** ke aapas ke talluq ko samajhte hain.

Is month me hum un teenon buniyaadi pillars ko cover karenge jo modern security systems ko secure banate hain:
1. **Applied Cryptography:** Symmetric ciphers (AES-CBC, AES-GCM), IV reuse attacks, Asymmetric cryptography (RSA modular arithmetic, ECC), Hashing (SHA-256, HMAC), aur TLS 1.3 handshake mechanics.
2. **Web Architecture & Client Boundaries:** HTTP/1.1 request/response lifecycles, security headers, cookie flags (`Secure`, `HttpOnly`, `SameSite`), SOP (Same-Origin Policy), CORS, aur DOM rendering.
3. **Compiler & Linker Pipeline:** Source code ka AST (Abstract Syntax Tree) banna, assembly generation, object file relocations (`.o`), static vs dynamic linking (`ld.so`), aur sab se critical: **PLT (Procedure Linkage Table) & GOT (Global Offset Table)** resolving mechanics.

Ye month source code se binary banne ke safar aur secure web communications ko crystal clear karta hai.

---

## 📚 Month 05 Knowledge Base & Topic Notes Directory

Is folder me Month 05 ke dauran banaye gaye tamam technical notes topics ke mutabiq categorized hain:

| Note File | Core Focus & Concepts Covered | Status |
| :--- | :--- | :---: |
| 📄 **[`01-applied-cryptography-ciphers.md`](./01-applied-cryptography-ciphers.md)** | AES-CBC/GCM, IV reuse vulnerabilities, RSA modular math, ECC discrete logarithms, HMAC, and collision resistance. | 🟢 Completed |
| 📄 **[`02-pki-x509-tls-handshake.md`](./02-pki-x509-tls-handshake.md)** | Public Key Infrastructure (PKI), Root & Intermediate CAs, X.509 certificate validation chains, and TLS 1.3 cryptographic transactions. | 🟢 Completed |
| 📄 **[`03-web-protocols-sop-cors.md`](./03-web-protocols-sop-cors.md)** | HTTP/1.1 protocol internals, cookie security flags, DOM tree structures, client-side JS models, and SOP/CORS boundaries. | 🟢 Completed |
| 📄 **[`04-compiler-pipeline-ast-linking.md`](./04-compiler-pipeline-ast-linking.md)** | Preprocessing (`cpp`), Lexing/Parsing, AST emission, Object files (`.o`), Relocations, and Static vs Dynamic Linking (`ld.so`). | 🟢 Completed |
| 📄 **[`05-elf-plt-got-resolution.md`](./05-elf-plt-got-resolution.md)** | Procedure Linkage Table (PLT), Global Offset Table (GOT), runtime symbol binding, and position-independent execution (`-fPIC`). | 🟢 Completed |

---

## ⚙️ The 3 Daily Continuous Side Tracks (Month 05 Focus)

Hamare daily 12-hour engine ke 3 parallel tracks ke dedicated notes:

### 1. 💻 Systems C Track (1 Hour Daily)
* **Goal:** Multi-file compilation and build automation.
* **Topics:** Writing modular C software using separate header files (`.h`), managing compilation units, and writing production Makefiles.

### 2. 🧩 Assembly & Reverse Engineering Track (1 Hour Daily)
* **Goal:** Compiler optimization transformations.
* **Topics:** Reconstructing high-level logic from compiler transformations: loop unrolling, register allocation algorithms, and function inlining in assembly.

### 3. 🔌 Hardware & Architecture Track (1 Hour Daily)
* **Goal:** CPU instruction execution pipeline.
* **Topics:** Classic 5-stage CPU pipeline: Fetch, Decode, Execute, Memory Access, and Writeback (FDEMW), along with CPU microcode decoding.

---

## 🛠️ Lab Environments & Hands-On Milestones

* 🎯 **Dynamic C HTTP/1.0 Socket Web Server:** Multi-threaded web server in pure C parsing raw HTTP requests, headers, and serving static filesystem objects securely.
* 🎯 **Custom Enterprise PKI Lab:** Private Root CA and Intermediate CA configured using OpenSSL, issuing signed certificates and enforcing TLS validation.
* 🎯 **Optimization Disassembly Analysis Report:** Comprehensive disassembly comparison of identical C routines compiled under `-O0`, `-O2`, and `-Os`.

---

## 📖 Primary Learning References
* 📘 *Serious Cryptography: A Practical Introduction to Modern Encryption* — Jean-Philippe Aumasson
* 💻 *Cryptopals Crypto Challenges* (Set 1)
* 💻 *PortSwigger Web Security Academy*
* 💻 *GCC & Clang Compiler Internals Documentation*

---

© **Muhammad Imran (Founder, IW Cyber Ops)** | Documented for Absolute Depth & Intellectual Rigor.
