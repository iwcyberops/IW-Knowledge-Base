<!-- 
SEO METADATA & KEYWORDS (Invisible to readers, visible to Google Crawlers)
Keywords: IW Cyber Ops, Muhammad Imran, Linux Kernel Interface, Bash Shell Automation, Cybersecurity Knowledge Base, Hacking Roadmap, System Administration, Process Signals, SUID SGID Permissions, FHS, Git Internals, Vulnerability Research, InfoSec Notes.
-->

# 🐧 Month 01: Linux Kernel Interface, Shell Automation & Lab Engineering

> **Knowledge Base Directory:** Phase 01 / Month 01  
> **System Operator & Author:** Muhammad Imran (Founder, **IW Cyber Ops**)  
> **Objective:** Reach absolute Command-Line Interface (CLI) independence and structurally master the Linux operating system architecture.

---

## 🏛️ The Imperative of Linux Mastery

In the realm of elite vulnerability research and offensive security, **Linux is not just an operating system; it is the fundamental battleground.** 

Every modern infrastructure—from cloud hypervisors and embedded IoT devices to mobile Android kernels and corporate web servers—runs on UNIX-based foundations. Attempting to exploit, reverse engineer, or secure systems without an atomic understanding of the Linux Kernel Interface, Discretionary Access Control (DAC), and Process Memory Models is a mathematical impossibility.

This directory serves as the **IW Cyber Ops Knowledge Base** for Month 01. It documents the critical transition from basic system administration to systematic architectural comprehension. 

Here, we do not just learn commands; we dissect *how* the Linux kernel handles processes, maps memory, enforces permissions, and interacts with hardware.

---

## 🧠 Core Domains Documented in this Directory

The notes contained within this module cover the following theoretical and practical pillars:

1. **Filesystem Hierarchy Standard (FHS):** The precise architectural purpose of root directories (`/etc`, `/proc`, `/sys`, `/dev`) and their security implications.
2. **Process Models & IPC:** Deconstructing process lifecycles (PID/PPID), systemd units, and the exact mechanics of UNIX signals (`SIGKILL`, `SIGSEGV`, `SIGTERM`).
3. **Permission Boundaries:** Deep-dive into DAC, `umask` calculations, Sticky bits, and the critical execution mechanics of `SUID`/`SGID` binaries.
4. **Text Manipulation Pipelines:** Advanced data extraction and stream editing utilizing `grep -E`, `awk`, `sed`, and `xargs`.
5. **Version Control Plumbing:** The low-level mechanics of Git (`objects`, `HEAD`, `refs`) for tracking source code archaeology and security commits.
6. **Low-Level Synergies:** Initial integration of C compilation stages (`gcc`), x86-64 General Purpose Registers (GPRs), and digital logic gates.

---

## 📂 Index of Technical Notes

*Below is the living index of all Markdown notes generated during this month's research. Click on any topic to access the detailed documentation.*

| Status | Technical Topic | File Reference |
| :---: | :--- | :--- |
| 📝 | Linux FHS & Security Implications | `[01-linux-fhs-security.md](./01-linux-fhs-security.md)` |
| 📝 | DAC, Permissions, SUID & SGID | `[02-dac-permissions-suid.md](./02-dac-permissions-suid.md)` |
| 📝 | Process Management & UNIX Signals | `[03-process-model-signals.md](./03-process-model-signals.md)` |
| 📝 | Shell Automation & Regex Pipelines | `[04-bash-regex-pipelines.md](./04-bash-regex-pipelines.md)` |
| 📝 | Git Plumbing & Security Archaeology | `[05-git-internals-plumbing.md](./05-git-internals-plumbing.md)` |
| 📝 | C Compilation & x86-64 Register Basics | `[06-c-compilation-x86-registers.md](./06-c-compilation-x86-registers.md)` |

*(Note: As the month progresses, new `.md` files will be added to this folder and linked above.)*

---

## 🛡️ About the Author

**Muhammad Imran** is a systems researcher and the Founder of **IW Cyber Ops**. This knowledge base is part of a rigorous 42-month journey engineered for absolute depth, intellectual rigor, and high-impact vulnerability research. 

To view the complete overarching roadmap, visit the [IW-Mission-Control](https://github.com/iwcyberops/IW-Mission-Control) repository.

<br>

---
*Generated & Curated by **IW Cyber Ops** | High-Assurance Cyber Operations & Research*
