# 🐧 Month 06: System Enumeration, Linux Privilege Escalation & Debugging Fundamentals

> **Research Track:** Phase 01 — Foundations & Systems Architecture  
> **Author & Lead Researcher:** Muhammad Imran (Founder, **IW Cyber Ops**)  
> **Knowledge Base Domain:** `IW-Knowledge-Base/m06-linux-privesc-debugging`

---

## 🧭 Why Systematic Enumeration & GDB Debugging are Essential

System administration se nikal kar **Offensive Security Operator aur Exploit Developer** banne ka pehla step ye hai ke aapko target system ki internal misconfigurations aur live memory state ko control karna aata ho.

Automated tools (jaise LinPEAS) run karna asaan hai, lekin jab automated scripts fail ho jati hain, tab **manual enumeration aur byte-level debugger tracing** hi target par root access dilwate hain. Is month me hum seekhenge:
1. **Systematic Host Enumeration:** Cron jobs, systemd timers, misconfigured write permissions, environment variables, aur hidden background processes (`pspy`) ko pinpoint karna.
2. **Linux Privilege Escalation Vectors:** SUID/SGID binaries abuse, POSIX Capabilities (`cap_setuid`, `cap_dac_override`), `/etc/sudoers` wildcards, dynamic linker hijacking (`LD_PRELOAD`, `LD_LIBRARY_PATH`), aur PATH variable poisoning.
3. **Post-Exploitation & Interactive Control:** Raw socket shells se full interactive TTY shells spawn karna, reverse vs bind shells, SSH key persistence, aur artifact cleanup.
4. **Low-Level GDB Debugging Mastery:** Software breakpoints (`int 3`), hardware watchpoints, memory dumps (`x/32gx`), register dereferencing, aur call stack analysis (`backtrace`).

Ye month hamare Linux permission boundaries ko break karne aur dynamic memory debugging ka basepoint hai.

---

## 📚 Month 06 Knowledge Base & Topic Notes Directory

Is folder me Month 06 ke dauran banaye gaye tamam technical notes topics ke mutabiq categorized hain:

| Note File | Core Focus & Concepts Covered | Status |
| :--- | :--- | :---: |
| 📄 **[`01-system-enumeration-attack-surface.md`](./01-system-enumeration-attack-surface.md)** | Manual enumeration vectors: cron jobs, systemd services, listening sockets, writable configs, and unquoted PATHs. | 🟢 Completed |
| 📄 **[`02-suid-sgid-sudoers-abuse.md`](./02-suid-sgid-sudoers-abuse.md)** | SUID execution mechanics, `/etc/sudoers` wildcard privilege escalation, GTFOBins dynamics, and `bash -p` privilege retention. | 🟢 Completed |
| 📄 **[`03-posix-capabilities-ld-preload.md`](./03-posix-capabilities-ld-preload.md)** | Abusing Linux POSIX Capabilities (`cap_setuid`, `cap_dac_override`), shared library hijacking via `LD_PRELOAD`, and `RPATH` / `RUNPATH`. | 🟢 Completed |
| 📄 **[`04-post-exploitation-tty-persistence.md`](./04-post-exploitation-tty-persistence.md)** | Python/script TTY spawning, stty raw echo, persistent SSH backdoor keys, cron persistence, and clean artifact cleanup. | 🟢 Completed |
| 📄 **[`05-gdb-pwndbg-debugging-mechanics.md`](./05-gdb-pwndbg-debugging-mechanics.md)** | GDB with Pwndbg/GEF, breakpoint injection (`int 3`), hardware watchpoints, examining registers, and stack backtracing. | 🟢 Completed |

---

## ⚙️ The 3 Daily Continuous Side Tracks (Month 06 Focus)

Hamare daily 12-hour engine ke 3 parallel tracks ke dedicated notes:

### 1. 💻 Systems C Track (1 Hour Daily)
* **Goal:** POSIX process and permission management tools.
* **Topics:** Writing custom C tools utilizing POSIX APIs (`chmod`, `chown`, `setuid`, `setgid`, `execve`) to create controlled escalation wrappers.

### 2. 🧩 Assembly & Reverse Engineering Track (1 Hour Daily)
* **Goal:** Dynamic control flow tracing in GDB.
* **Topics:** Stepping through execution instruction-by-instruction (`stepi`, `nexti`), examining conditional register states, and inspecting EFLAGS (`ZF`, `SF`, `OF`).

### 3. 🔌 Hardware & Architecture Track (1 Hour Daily)
* **Goal:** Hardware timer interrupts and OS preemptive scheduling.
* **Topics:** Programmable Interval Timers (PIT), APIC architecture, timer interrupts driving the Linux kernel scheduler, and context switch overhead.

---

## 🛠️ Lab Environments & Hands-On Milestones

* 🎯 **Vulnerable Privilege Escalation VM:** Custom Debian virtual machine created with 8 distinct, deliberate privilege escalation misconfigurations.
* 🎯 **Automated Linux Security Audit Tool:** Custom Python tool automatically scanning Linux hosts for misconfigured capabilities, sudo rules, and dangerous file permissions.
* 🎯 **Privilege Escalation Casebook:** Detailed root-cause analysis and exploitation writeups documenting root acquisition across 10 vulnerable machines on Hack The Box & TryHackMe.

---

## 📖 Primary Learning References
* 💻 *TryHackMe* (Linux Privilege Escalation Path)
* 💻 *Hack The Box* (Easy Linux Security Track)
* 💻 *pwn.college* (Privilege Escalation Track)
* 💻 *GDB Official Documentation & Pwndbg Cheatsheet*

---

© **Muhammad Imran (Founder, IW Cyber Ops)** | Documented for Absolute Depth & Intellectual Rigor.
