# 🐧 Month 01: Linux Kernel Interface, Shell Automation & Lab Engineering

> **Research Track:** Phase 01 — Foundations & Systems Architecture  
> **Author & Lead Researcher:** Muhammad Imran (Founder, **IW Cyber Ops**)  
> **Knowledge Base Domain:** `IW-Knowledge-Base/m01-linux-kernel-interface`

---

## 🧭 Why Linux is the Core Engine of Cybersecurity & 0-Day Research

Linux koi aam operating system nahi hai; ye modern internet, cloud infrastructure, servers, containers (Docker/K8s), Android devices, aur embedded/IoT systems ki **reeh ki haddi (backbone)** hai.

Agar aapko low-level binary exploitation, kernel rootkits, privilege escalation, ya malware analysis seekhna hai, to GUI (Graphical User Interface) par depend rehna namumkin hai. Ek Elite Vulnerability Researcher ko:
1. **System Call Interface (`syscalls`)** samajh aani chahiye ke user-space aur kernel-space aapas me baat kaise karte hain.
2. **Virtual Filesystems (`/proc`, `/sys`)** ke zariye live running memory aur kernel state inspect karni aani chahiye.
3. **Discretionary Access Control (DAC)** aur permission boundaries (`SUID`, `capabilities`) ko mathematically understand karna chahiye.
4. **Shell & Text Manipulation (`awk`, `sed`, `grep`)** se massive log files aur memory dumps ko automate karna aana chahiye.

Ye month hamari **Complete CLI Independence** aur computational base ki pehli mazboot eent hai.

---

## 📚 Month 01 Knowledge Base & Topic Notes Directory

Is folder me Month 01 ke dauran banaye gaye tamam deep technical notes topics ke mutabiq categorized hain:

| Note File | Core Focus & Concepts Covered | Status |
| :--- | :--- | :---: |
| 📄 **[`01-fhs-and-virtual-filesystems.md`](./01-fhs-and-virtual-filesystems.md)** | In-depth security of `/etc`, `/proc`, `/sys`, `/dev`, `/tmp`, and memory-mapped pseudo-filesystems. | 🟢 Completed |
| 📄 **[`02-linux-permissions-dac-suid.md`](./02-linux-permissions-dac-suid.md)** | DAC, UID/GID security boundaries, SUID/SGID execution mechanics, Sticky Bit, `umask`, and `sudoers`. | 🟢 Completed |
| 📄 **[`03-process-lifecycle-signals.md`](./03-process-lifecycle-signals.md)** | PID/PPID hierarchy, `fork()` and `execve()` execution transitions, UNIX signal dispatch (`SIGSEGV`, `SIGKILL`). | 🟢 Completed |
| 📄 **[`04-text-processing-automation.md`](./04-text-processing-automation.md)** | High-speed regex parsing with `grep -E`, stream editing with `sed`, column extraction with `awk`, and `xargs`. | 🟢 Completed |
| 📄 **[`05-git-internals-plumbing.md`](./05-git-internals-plumbing.md)** | Git DAG internals, Objects (`blob`, `tree`, `commit`), `refs`, `HEAD`, and security patch tracking via `git log -p`. | 🟢 Completed |

---

## ⚙️ The 3 Daily Continuous Side Tracks (Month 01 Focus)

Hamare daily 12-hour engine ke 3 parallel tracks ke dedicated notes:

### 1. 💻 Systems C Track (1 Hour Daily)
* **Goal:** Understand source-to-binary compilation pipeline.
* **Topics:** C syntax, compilation stages (`gcc -E` Preprocessor, `gcc -S` Assembly, `gcc -c` Object code, `gcc -o` Binary), and memory allocation foundations.

### 2. 🧩 Assembly & Reverse Engineering Track (1 Hour Daily)
* **Goal:** Dissect machine execution architecture.
* **Topics:** x86-64 General Purpose Registers (`RAX`, `RBX`, `RCX`, `RDX`, `RSI`, `RDI`, `RSP`, `RBP`) and register sub-division bit-widths (64-bit, 32-bit, 16-bit, 8-bit).

### 3. 🔌 Hardware & Architecture Track (1 Hour Daily)
* **Goal:** Understand the silicon beneath the software.
* **Topics:** Ohm's Law, Kirchhoff's Laws, voltage/current fundamentals, discrete logic gates (`AND`, `OR`, `XOR`, `NOT`), and digital logic simulation in Logisim.

---

## 🛠️ Lab Environments & Hands-On Milestones

* 🎯 **Bandit Wargames (OverTheWire):** Levels 1 to 34 (Command line manipulation mastery).
* 🎯 **Virtual Lab Setup:** Isolated Dual-NIC Debian/Kali Linux environment built inside VMware Workstation Pro.
* 🎯 **Capstone Tool:** Automated Lab Deployment Engine (Bash automation suite).

---

## 📖 Primary Learning References
* 📘 *How Linux Works: What Every Superuser Should Know* — Brian Ward
* 📘 *The Linux Command Line* — William Shotts
* 🌐 *pwn.college* (Linux Luminarium Module)

---

© **Muhammad Imran (Founder, IW Cyber Ops)** | Documented for Absolute Depth & Intellectual Rigor.
