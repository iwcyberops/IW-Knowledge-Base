<!-- 
SEO METADATA & KEYWORDS (Invisible to readers, visible to Google Crawlers)
Keywords: IW Cyber Ops, Muhammad Imran, Windows Internals, Active Directory Architecture, PE File Format, Win32 API, Ntoskrnl, Kerberos Authentication, LSASS, Access Tokens, Cybersecurity Knowledge Base, Vulnerability Research.
-->

# 🪟 Month 09: Windows Internals, Architecture & Active Directory Foundations

> **Knowledge Base Directory:** Phase 02 / Month 09  
> **System Operator & Author:** Muhammad Imran (Founder, **IW Cyber Ops**)  
> **Objective:** Dissect Windows operating system architecture, the NT executive subsystem, and enterprise Active Directory domain topologies.

---

## 🏛️ The Imperative of Windows Architecture

Phase 02 marks the transition from foundational mechanics into aggressive offensive security. 

To dominate an enterprise network, one must intimately understand its core operating system. Windows powers the vast majority of corporate endpoints and domain infrastructures worldwide. Merely running exploitation tools against a Windows machine is inadequate; an elite researcher must understand exactly how the `ntoskrnl` handles memory, how the Win32 subsystem translates API calls, and how access tokens govern security boundaries.

Simultaneously, enterprise security relies entirely on Active Directory (AD). Before attempting to exploit AD protocols, one must comprehend its architectural logic—Trusts, Group Policy, and the cryptographic state machines driving Kerberos. 

This directory serves as the **IW Cyber Ops Knowledge Base** for Month 09. It documents the systematic deconstruction of the Windows OS, the Portable Executable (PE) binary format, and the foundational topologies of Active Directory.

---

## 🧠 Core Domains Documented in this Directory

The notes contained within this module cover the following theoretical and practical pillars:

1. **Windows OS Subsystems:** Exploring the architecture of `ntoskrnl.exe`, the Hardware Abstraction Layer (HAL), Win32 subsystem (`csrss.exe`, `lsass.exe`), the Native API (`Nt/Zw` syscalls), and user-mode structures like PEB and TEB.
2. **Windows Security Model:** Dissecting Security Identifiers (SIDs), DACLs, SACLs, Access Tokens (Primary vs. Impersonation), and critical Token Privileges (`SeDebugPrivilege`, `SeImpersonatePrivilege`).
3. **Portable Executable (PE) Architecture:** Manually parsing the DOS Header, PE Signature, File/Optional Headers, Import Address Table (IAT), Export Tables, and Section Headers (`.text`, `.data`, `.rsrc`).
4. **Active Directory Domain Architecture:** Mapping Domain Controllers, Forests, Trusts, LDAP directory trees, and Group Policy Objects (GPO).
5. **Kerberos Authentication:** Tracking the exact cryptographic workflow of Kerberos: AS-REQ, AS-REP, TGS-REQ, TGS-REP, and PAC validation, alongside NTLMv2 challenge-response mechanics.
6. **Win32 API & Hardware Interaction:** Interacting with OS processes via `OpenProcess` and `CreateRemoteThread`, mapping the Microsoft x64 calling convention, and analyzing x86 Control Registers (`CR0`, `CR3`, `CR4`).

---

## 📂 Index of Technical Notes

*Below is the living index of all Markdown notes generated during this month's research. Click on any topic to access the detailed documentation.*

| Status | Technical Topic | File Reference |
| :---: | :--- | :--- |
| 📝 | Windows Subsystems, PEB/TEB & Native API | `[01-windows-subsystems-peb-teb.md](./01-windows-subsystems-peb-teb.md)` |
| 📝 | Windows Security Model: Tokens, SIDs & DACLs | `[02-win-security-tokens-dacls.md](./02-win-security-tokens-dacls.md)` |
| 📝 | Portable Executable (PE) Format & IAT Parsing | `[03-pe-format-iat-parsing.md](./03-pe-format-iat-parsing.md)` |
| 📝 | AD Architecture: Forests, Domains & Trusts | `[04-ad-architecture-forests-trusts.md](./04-ad-architecture-forests-trusts.md)` |
| 📝 | Kerberos Authentication Workflow & NTLMv2 | `[05-kerberos-workflow-ntlmv2.md](./05-kerberos-workflow-ntlmv2.md)` |
| 📝 | Win32 API, x64 Fastcall & Control Registers | `[06-win32-api-x64-fastcall.md](./06-win32-api-x64-fastcall.md)` |

*(Note: As the month progresses, new `.md` files will be added to this folder and linked above.)*

---

## 🛡️ About the Author

**Muhammad Imran** is an independent systems researcher and the Founder of **IW Cyber Ops**. This knowledge base is an active repository complementing a rigorous 42-month journey engineered for absolute depth, intellectual rigor, and high-impact vulnerability research.

To view the complete overarching roadmap, visit the official [IW-Mission-Control](https://github.com/iwcyberops/IW-Mission-Control) repository.

<br>

---
*Generated & Curated by **IW Cyber Ops** | High-Assurance Cyber Operations & Research*
