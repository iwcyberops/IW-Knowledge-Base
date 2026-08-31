# 🎯 Month 13: Binary Exploitation II – Format Strings, Advanced ROP & SROP

> **Research Track:** Phase 02 — Offensive Security, Windows, RE & Exploitation  
> **Author & Lead Researcher:** Muhammad Imran (Founder, **IW Cyber Ops**)  
> **Knowledge Base Domain:** `IW-Knowledge-Base/m13-format-strings-srop`

---

## 🧭 Why Format Strings, SROP & Universal Gadgets are Game Changers

Jab kisi binary me standard buffer overflow ke liye ROP gadgets kam hon ya binary completely stripped ho, tab ek researcher ko **Arbitrary Memory Read/Write Primitives** aur **Advanced Signal-Driven Hijacking** ka sahara lena padta hai.

Is month me hum un techniques ko deconstruct karenge jo memory corruption ko surgically precise banati hain:
1. **Format String Exploitation:** Variadic functions (`printf`, `sprintf`) ke positional parameter specifiers (`%k$x`) ke zariye stack aur heap data leak karna, aur `%n` / `%hhn` ke zariye arbitrary memory addresses par single-byte precise writes perform karna.
2. **RELRO & Target Selection:** Partial RELRO (jisme Global Offset Table / GOT overwrite ho sakta hai) vs Full RELRO (jisme hum stack return addresses, `__malloc_hook`, ya `__free_hook` ko target karte hain).
3. **Sigreturn-Oriented Programming (SROP):** Linux kernel ke signal handling mechanism ko hijack karna—stack par ek fake `sigcontext` frame forge karke sirf ek `rt_sigreturn` syscall (gadget) se CPU ke tamam 64-bit registers (`RAX`, `RDI`, `RSI`, `RDX`, `RIP`, `RSP`) ko ek hi jhatke me control karna.
4. **Universal Gadgets (`ret2csu`):** Stripped x86-64 binaries me standard `pop rdi; pop rsi` gadgets na hone par `__libc_csu_init` ke 2-stage execution flow se parameters pass karna.

Ye month hamare binary exploitation arsenal ko mathematical precision aur universality faraham karta hai.

---

## 📚 Month 13 Knowledge Base & Topic Notes Directory

Is folder me Month 13 ke dauran banaye gaye tamam technical notes topics ke mutabiq categorized hain:

| Note File | Core Focus & Concepts Covered | Status |
| :--- | :--- | :---: |
| 📄 **[`01-format-string-leaks-writes.md`](./01-format-string-leaks-writes.md)** | Variadic functions, positional specifiers (`%k$p`), memory leak primitives, and single-byte arbitrary writes with `%hhn`. | 🟢 Completed |
| 📄 **[`02-relro-got-overwrite-attacks.md`](./02-relro-got-overwrite-attacks.md)** | Partial RELRO vs Full RELRO mechanics, `.got.plt` overwrite techniques, and targeting allocator hooks (`__malloc_hook`). | 🟢 Completed |
| 📄 **[`03-sigreturn-oriented-programming-srop.md`](./03-sigreturn-oriented-programming-srop.md)** | Linux kernel signal delivery, `ucontext_t` / `sigcontext` frame forging, and register takeover via `SYS_rt_sigreturn` (`0x0f`). | 🟢 Completed |
| 📄 **[`04-ret2csu-universal-gadgets.md`](./04-ret2csu-universal-gadgets.md)** | Dissecting `__libc_csu_init`, Stage 1 and Stage 2 register population (`R12`..`R15` to `RDI`, `RSI`, `RDX`), and stripped binary ROP. | 🟢 Completed |
| 📄 **[`05-pwntools-fmtstr-srop-automation.md`](./05-pwntools-fmtstr-srop-automation.md)** | Automating format string payload generation via `FmtStr` and forging complete SROP frames using `SigreturnFrame()` in pwntools. | 🟢 Completed |

---

## ⚙️ The 3 Daily Continuous Side Tracks (Month 13 Focus)

Hamare daily 12-hour engine ke 3 parallel tracks ke dedicated notes:

### 1. 💻 Systems C/C++ Track (1 Hour Daily)
* **Goal:** UNIX signal handling and kernel context switching.
* **Topics:** Implementing custom signal handlers in C using `sigaction()`, examining signal stack setup (`sigaltstack`), and analyzing how the kernel restores register context upon return.

### 2. 🧩 Assembly & Reverse Engineering Track (1 Hour Daily)
* **Goal:** In-depth dissection of `__libc_csu_init`.
* **Topics:** Dissecting the exact assembly opcodes of the two `__libc_csu_init` gadget blocks to understand indirect call register dependencies (`call qword ptr [r12+rbx*8]`).

### 3. 🔌 Hardware & Architecture Track (1 Hour Daily)
* **Goal:** CPU Interrupt Descriptor Table (IDT) and interrupt gates.
* **Topics:** Study of hardware Interrupt Descriptor Tables (IDT), interrupt vectors, trap gates vs interrupt gates, and hardware interrupt dispatch mechanics.

---

## 🛠️ Lab Environments & Hands-On Milestones

* 🎯 **Automated Format String Exploit Engine:** Reusable Python utility that dynamically detects stack offsets and automatically formats sorted `%hhn` single-byte memory write payloads.
* 🎯 **Pure SROP Exploitation Suite:** Fully working exploit weaponizing only two gadgets (`pop rax; syscall` and `syscall`) to execute `execve("/bin/sh")` via a forged sigreturn frame.
* 🎯 **`ret2csu` Reusable Template:** Modular `pwntools` harness automating register initialization (`RDI`, `RSI`, `RDX`) across stripped 64-bit target binaries.

---

## 📖 Primary Learning References
* 📘 *Practical Binary Analysis* — Dennis Andriesse
* 💻 *RPISEC Modern Binary Exploitation (MBE Course)*
* 📄 *Framing Signals — A Return to Portable Shellcode* — Erik Bosman & Herbert Bos (VU Amsterdam)
* 💻 *Pwnable.kr & pwn.college (Advanced Exploitation Challenges)*

---

© **Muhammad Imran (Founder, IW Cyber Ops)** | Documented for Absolute Depth & Intellectual Rigor.
