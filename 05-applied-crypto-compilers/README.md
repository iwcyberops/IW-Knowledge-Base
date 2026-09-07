<!-- 
SEO METADATA & KEYWORDS (Invisible to readers, visible to Google Crawlers)
Keywords: IW Cyber Ops, Muhammad Imran, Applied Cryptography, Web Security Architecture, Compilation Pipelines, ELF Internals, Linker Mechanics, GOT PLT, AES GCM, TLS Handshake, SOP CORS, Cybersecurity Knowledge Base.
-->

# 🔐 Month 05: Applied Cryptography, Web Foundations & Compilation Pipelines

> **Knowledge Base Directory:** Phase 01 / Month 05  
> **System Operator & Author:** Muhammad Imran (Founder, **IW Cyber Ops**)  
> **Objective:** Understand secure communication channels, web client-server protocols, and the complete source-to-binary compilation process.

---

## 🏛️ The Imperative of Cryptography & Compilation Pipelines

Security is an illusion without cryptographic enforcement, and reverse engineering is guesswork without understanding how code is compiled. 

An elite vulnerability researcher must possess a holistic view of the technology stack. You must understand how raw source code is parsed, optimized, and linked into a binary executable (`ELF`) before it ever touches memory. Simultaneously, you must comprehend how that executable communicates securely over untrusted networks, managing encryption keys, TLS handshakes, and web security boundaries (SOP/CORS).

This directory serves as the **IW Cyber Ops Knowledge Base** for Month 05. It bridges the gap between discrete components, mapping the lifecycle of data from plain text to cryptographic ciphertexts, and tracing source code from high-level C all the way down to dynamic linker resolutions (`PLT/GOT`).

---

## 🧠 Core Domains Documented in this Directory

The notes contained within this module cover the following theoretical and practical pillars:

1. **Applied Cryptography:** Mechanics of symmetric ciphers (AES-CBC, AES-GCM, IV reuse vulnerabilities), asymmetric cryptography (RSA modular arithmetic, ECC discrete logarithms), Hashing integrity (SHA-256, HMAC), and Public Key Infrastructure (PKI / X.509).
2. **Secure Communication:** Dissecting the TLS 1.3 handshake, certificate authorities (CAs), and establishing secure enterprise encryption architectures.
3. **Web Application Architecture:** The lifecycle of HTTP requests/responses, critical security headers, cookie flags (`Secure`, `HttpOnly`, `SameSite`), Cross-Origin Resource Sharing (CORS), the Same-Origin Policy (SOP), and client-side DOM execution models.
4. **Compiler Internals:** Tracing the exact stages of GCC/Clang: Preprocessing (`cpp`), lexical analysis, Abstract Syntax Tree (AST) generation, and the impact of compiler optimization flags (`-O0`, `-O2`, `-Os`) on emitted assembly.
5. **Linker Mechanics & ELF Structures:** Static vs. Dynamic linking (`ld.so`), dissecting Executable and Linkable Format (ELF) sections, relocations, and the dynamic resolution of shared libraries via the Global Offset Table (GOT) and Procedure Linkage Table (PLT).

---

## 📂 Index of Technical Notes

*Below is the living index of all Markdown notes generated during this month's research. Click on any topic to access the detailed documentation.*

| Status | Technical Topic | File Reference |
| :---: | :--- | :--- |
| 📝 | Applied Cryptography: AES, RSA, ECC & Hashing | `[01-applied-crypto-aes-rsa.md](./01-applied-crypto-aes-rsa.md)` |
| 📝 | Enterprise PKI, X.509 & TLS 1.3 Mechanics | `[02-pki-x509-tls13.md](./02-pki-x509-tls13.md)` |
| 📝 | Web Architecture: HTTP, SOP, CORS & Cookies | `[03-web-arch-sop-cors.md](./03-web-arch-sop-cors.md)` |
| 📝 | Compiler Internals: Preprocessing to AST | `[04-compiler-internals-ast.md](./04-compiler-internals-ast.md)` |
| 📝 | Linker Mechanics: ELF, Relocations & GOT/PLT | `[05-linker-elf-got-plt.md](./05-linker-elf-got-plt.md)` |
| 📝 | CPU Instruction Cycles & Optimization Disassembly | `[06-cpu-cycles-optimizations.md](./06-cpu-cycles-optimizations.md)` |

*(Note: As the month progresses, new `.md` files will be added to this folder and linked above.)*

---

## 🛡️ About the Author

**Muhammad Imran** is an independent systems researcher and the Founder of **IW Cyber Ops**. This knowledge base is an active repository complementing a rigorous 42-month journey engineered for absolute depth, intellectual rigor, and high-impact vulnerability research.

To view the complete overarching roadmap, visit the official [IW-Mission-Control](https://github.com/iwcyberops/IW-Mission-Control) repository.

<br>

---
*Generated & Curated by **IW Cyber Ops** | High-Assurance Cyber Operations & Research*
