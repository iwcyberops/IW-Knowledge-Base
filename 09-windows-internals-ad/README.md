# 🪟 Month 09: Windows Internals, Architecture & Active Directory Foundations

> **Research Track:** Phase 02 — Offensive Security, Windows, RE & Exploitation  
> **Author & Lead Researcher:** Muhammad Imran (Founder, **IW Cyber Ops**)  
> **Knowledge Base Domain:** `IW-Knowledge-Base/m09-windows-internals-ad`

---

## 🧭 Why Windows Internals & Active Directory are Crucial for Red Teaming

Dunya ke lagbhag 90%+ corporate enterprise networks **Microsoft Windows aur Active Directory (AD)** par chalte hain. Agar aapko Windows operating system ke kernel internals aur AD domain security ki gehrai nahi pata, to aap modern enterprise offensive operations me kabhi dominate nahi kar sakte.

Linux ki tarah Windows open-source nahi hai, isliye yahan **undocumented structures aur reverse engineering** ka role double ho jata hai. Ek elite security researcher ke liye ye samajhna lazmi hai ke:
1. **The NT Architecture & Kernel Subsystems:** `ntoskrnl.exe`, HAL (Hardware Abstraction Layer), Win32 subsystem (`csrss.exe`, `lsass.exe`), Native API (`Nt`/`Zw` syscall transitions), aur user-space structures (`PEB`, `TEB`).
2. **The Windows Security & Token Model:** SIDs (Security Identifiers), DACLs, SACLs, Access Tokens (Primary vs Impersonation), aur high-privilege tokens (`SeDebugPrivilege`, `SeImpersonatePrivilege`).
3. **PE (Portable Executable) Format:** Windows binary anatomy—DOS Header, PE Signature, Optional Header, Data Directories, Import Address Table (IAT), Export Address Table (EAT), aur Relocations.
4. **Active Directory Domain Architecture:** Enterprise Forest hierarchies, Trusts, LDAP trees, Group Policy Objects (GPO), aur cryptographic authentication protocols (**Kerberos** AS-REQ/REP, TGS-REQ/REP, PAC validation vs **NTLMv2** challenge-response).

Ye month humein enterprise Windows exploitation aur process injection ka master banata hai.

---

## 📚 Month 09 Knowledge Base & Topic Notes Directory

Is folder me Month 09 ke dauran banaye gaye tamam technical notes topics ke mutabiq categorized hain:

| Note File | Core Focus & Concepts Covered | Status |
| :--- | :--- | :---: |
| 📄 **[`01-windows-architecture-executive-kernel.md`](./01-windows-architecture-executive-kernel.md)** | Architecture of `ntoskrnl.exe`, HAL, `csrss.exe`, `lsass.exe`, Native API (`Nt/Zw`), PEB, and TEB memory layouts. | 🟢 Completed |
| 📄 **[`02-windows-security-tokens-privileges.md`](./02-windows-security-tokens-privileges.md)** | SIDs, DACLs, SACLs, Token Impersonation levels, `SeDebugPrivilege`, `SeImpersonatePrivilege`, and token stealing dynamics. | 🟢 Completed |
| 📄 **[`03-pe-file-structure-iat-parsing.md`](./03-pe-file-structure-iat-parsing.md)** | PE32/PE32+ file format, Optional Header, IAT/EAT resolution, Data Directories, and section header properties (`IMAGE_SECTION_HEADER`). | 🟢 Completed |
| 📄 **[`04-active-directory-forests-trusts.md`](./04-active-directory-forests-trusts.md)** | Domain Controllers, Forest topologies, Trust directions/types, LDAP directory trees, schema partitions, and GPO mechanics. | 🟢 Completed |
| 📄 **[`05-kerberos-ntlm-authentication-flow.md`](./05-kerberos-ntlm-authentication-flow.md)** | Kerberos ticket transactions (AS-REQ/AS-REP, TGS-REQ/TGS-REP), PAC signing and validation, and NTLMv2 challenge-response flow. | 🟢 Completed |

---

## ⚙️ The 3 Daily Continuous Side Tracks (Month 09 Focus)

Hamare daily 12-hour engine ke 3 parallel tracks ke dedicated notes:

### 1. 💻 Systems C/C++ Track (1 Hour Daily)
* **Goal:** Win32 API programming and process interaction.
* **Topics:** Writing native C++ programs utilizing Win32 APIs (`OpenProcess`, `VirtualAllocEx`, `WriteProcessMemory`, `CreateRemoteThread`) to understand process memory manipulation.

### 2. 🧩 Assembly & Reverse Engineering Track (1 Hour Daily)
* **Goal:** Microsoft x64 calling conventions and shadow space.
* **Topics:** Analyzing 4-register fastcall convention (`RCX`, `RDX`, `R8`, `R9`), 32-byte shadow space allocation on the stack, and disassembly of Win32 API wrappers.

### 3. 🔌 Hardware & Architecture Track (1 Hour Daily)
* **Goal:** x86 Control Registers and Model-Specific Registers.
* **Topics:** Study of `CR0` (protection enable, paging enable), `CR3` (Page Directory Base Register / page table root pointer), `CR4`, and Model-Specific Registers (MSRs) like `IA32_LSTAR` (syscall entry point).

---

## 🛠️ Lab Environments & Hands-On Milestones

* 🎯 **Automated Multi-VM Active Directory Lab:** Multi-VM enterprise forest lab deployed using Windows Server 2022 and Windows 11 Enterprise, configured with custom OUs, GPOs, and vulnerable service accounts.
* 🎯 **Native Windows PE Header Parser in C++:** Standalone command-line utility in C++ that parses raw PE headers from disk, dumps IAT imported DLLs/functions, and checks Dynamic Base (ASLR) and DEP flags.
* 🎯 **Kerberos Authentication State Diagram:** Step-by-step cryptographic transaction flow diagram mapping ticket requests, PAC verification, and session key generation.

---

## 📖 Primary Learning References
* 📘 *Windows Internals, Part 1: System Architecture, Processes, Threads, Memory Management* — Pavel Yosifovich, Mark E. Russinovich, David A. Solomon, Alex Ionescu
* 📘 *Windows Internals, Part 2* — Mark E. Russinovich, Andrea Allievi, Alex Ionescu, David A. Solomon
* 💻 *Microsoft Learn (Windows Security & Architecture Documentation)*
* 💻 *SpecterOps Technical Research Blogs*

---

© **Muhammad Imran (Founder, IW Cyber Ops)** | Documented for Absolute Depth & Intellectual Rigor.
