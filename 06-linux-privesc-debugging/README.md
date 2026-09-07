<!-- 
SEO METADATA & KEYWORDS (Invisible to readers, visible to Google Crawlers)
Keywords: IW Cyber Ops, Muhammad Imran, Linux Privilege Escalation, GDB Debugging, System Enumeration, SUID Exploitation, Post-Exploitation, POSIX Capabilities, LD_PRELOAD, Cybersecurity Knowledge Base, Vulnerability Research.
-->

# 💀 Month 06: System Enumeration, Linux Privilege Escalation & Debugging Fundamentals

> **Knowledge Base Directory:** Phase 01 / Month 06  
> **System Operator & Author:** Muhammad Imran (Founder, **IW Cyber Ops**)  
> **Objective:** Move from system administration to systematic host enumeration, exploit Linux permission misconfigurations, and master low-level GDB debugging.

---

## 🏛️ The Imperative of Escalation & Debugging

A foundational understanding of Linux is insufficient if one cannot weaponize that knowledge. 

The transition from a low-privileged user to absolute system control (`root`) requires surgical enumeration. It is the ability to look at a system, identify flawed developer assumptions, misconfigured cron jobs, or exposed capabilities, and chain them into a local privilege escalation (LPE) vector. 

Equally critical is the ability to inspect exactly *why* a program behaves the way it does. The GNU Debugger (GDB) is the magnifying glass for the apex hacker. Guesswork is unacceptable; when an exploit or script fails, the researcher must immediately attach a debugger, inspect memory layouts, trace control flow, and dereference registers at the byte level.

This directory serves as the **IW Cyber Ops Knowledge Base** for Month 06. It documents the pivot from defensive architecture to offensive enumeration, privilege escalation, and runtime execution control.

---

## 🧠 Core Domains Documented in this Directory

The notes contained within this module cover the following theoretical and practical pillars:

1. **System Enumeration Methodology:** Systematic identification of attack surfaces, querying cron jobs, analyzing systemd timers, hunting writable configuration files, inspecting environment variables, and mapping active processes/network sockets.
2. **Linux Privilege Escalation Vectors:** Exploiting misconfigured SUID/SGID binaries, abusing Linux POSIX capabilities (`cap_setuid`, `cap_dac_override`), hijacking `/etc/sudoers` wildcard rules, and executing `LD_PRELOAD` / `LD_LIBRARY_PATH` environment hijacks.
3. **Post-Exploitation Fundamentals:** Upgrading dumb shells to fully interactive TTYs, establishing reverse vs. bind shells, ensuring persistence via SSH keys and cron, and executing secure artifact cleanup.
4. **Low-Level Debugging Mechanics (GDB):** Mastering runtime inspection by setting software breakpoints (`int 3`), configuring hardware watchpoints, analyzing memory block layouts (`x/32gx`), dereferencing raw CPU registers, and backtracing complex call stacks.
5. **Operating System Scheduling:** Understanding how memory bus protocols and hardware timer interrupts drive preemptive OS process scheduling.

---

## 📂 Index of Technical Notes

*Below is the living index of all Markdown notes generated during this month's research. Click on any topic to access the detailed documentation.*

| Status | Technical Topic | File Reference |
| :---: | :--- | :--- |
| 📝 | System Enumeration & Surface Mapping | `[01-system-enumeration-mapping.md](./01-system-enumeration-mapping.md)` |
| 📝 | SUID, Sudoers & Capabilities Exploitation | `[02-suid-sudoers-capabilities.md](./02-suid-sudoers-capabilities.md)` |
| 📝 | Environment Hijacking: `PATH` & `LD_PRELOAD` | `[03-env-hijacking-ldpreload.md](./03-env-hijacking-ldpreload.md)` |
| 📝 | Interactive TTYs, Shells & Post-Exploitation | `[04-interactive-shells-persistence.md](./04-interactive-shells-persistence.md)` |
| 📝 | Advanced GDB: Breakpoints, Registers & Memory | `[05-gdb-breakpoints-memory.md](./05-gdb-breakpoints-memory.md)` |
| 📝 | Hardware Timers & Preemptive OS Scheduling | `[06-hardware-timers-scheduling.md](./06-hardware-timers-scheduling.md)` |

*(Note: As the month progresses, new `.md` files will be added to this folder and linked above.)*

---

## 🛡️ About the Author

**Muhammad Imran** is an independent systems researcher and the Founder of **IW Cyber Ops**. This knowledge base is an active repository complementing a rigorous 42-month journey engineered for absolute depth, intellectual rigor, and high-impact vulnerability research.

To view the complete overarching roadmap, visit the official [IW-Mission-Control](https://github.com/iwcyberops/IW-Mission-Control) repository.

<br>

---
*Generated & Curated by **IW Cyber Ops** | High-Assurance Cyber Operations & Research*
