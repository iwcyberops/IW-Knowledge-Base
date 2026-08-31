# 🔍 Month 11: Reverse Engineering I – Disassembly, Control Flow & Ghidra Mastery

> **Research Track:** Phase 02 — Offensive Security, Windows, RE & Exploitation  
> **Author & Lead Researcher:** Muhammad Imran (Founder, **IW Cyber Ops**)  
> **Knowledge Base Domain:** `IW-Knowledge-Base/m11-reverse-engineering-ghidra`

---

## 🧭 Why Reverse Engineering & Ghidra Automation are Essential

Vulnerability Research aur Binary Exploitation ki dunya me aapko aksar aisi binaries aur closed-source applications ka samna karna padega jinka **source code dunya me kisi ke paas nahi hota** (commercial software, proprietary firmware, malware samples, C2 agents).

Yahan **Reverse Engineering (RE)** ek super-power ban jati hai—compiled machine code (raw bytes aur opcodes) ko wapas human-readable logic me convert karna. Ek elite reverse engineer banne ke liye humein seekhna hai:
1. **Disassembly & Cross-Reference Analysis:** Raw hex bytes ko assembly instructions me decode karna, cross-references (`Xrefs`) ko trace karna, aur binary offsets se missing structs aur function signatures ko reconstruct karna.
2. **Control Flow Graph (CFG) Reconstruction:** Compiler optimizations ke bawajood branching logic (`if`/`else`), loops (`while`, `for`), switch jump tables, aur tail-call optimizations ko pehchanna.
3. **Static vs. Dynamic Analysis Synergy:** Ghidra me static decompilation ko live debuggers (GDB/x64dbg) ke runtime breakpoint traces ke sath link karna taake memory state verify ho sake.
4. **Automated Ghidra Scripting:** Java aur Python (`FlatProgramAPI`) ke zariye Ghidra ko automate karna taake custom string deobfuscation, XOR loops, aur cryptographic tables secondon me decrypt ho sakein.

Ye month humein closed-source black-box binaries ke andar jhaank kar unka logic cheernay ka fan sikhata hai.

---

## 📚 Month 11 Knowledge Base & Topic Notes Directory

Is folder me Month 11 ke dauran banaye gaye tamam technical notes topics ke mutabiq categorized hain:

| Note File | Core Focus & Concepts Covered | Status |
| :--- | :--- | :---: |
| 📄 **[`01-disassembly-xrefs-struct-recovery.md`](./01-disassembly-xrefs-struct-recovery.md)** | Opcode decoding, analyzing `Xrefs` (data/code references), variable typing, and reconstructing struct layouts from memory offsets. | 🟢 Completed |
| 📄 **[`02-control-flow-loops-jump-tables.md`](./02-control-flow-loops-jump-tables.md)** | Reconstructing high-level branches, nested loops, switch-case jump tables, function prologues/epilogues, and tail-calls. | 🟢 Completed |
| 📄 **[`03-ghidra-decompiler-type-fixing.md`](./03-ghidra-decompiler-type-fixing.md)** | Resolving `undefined1 *` pointers, creating custom C structure data types in Ghidra, and decompiler cleanup. | 🟢 Completed |
| 📄 **[`04-ghidra-scripting-automation.md`](./04-ghidra-scripting-automation.md)** | Automating binary analysis using Ghidra Python/Java API (`FlatProgramAPI`), automated pattern scanning, and custom exporters. | 🟢 Completed |
| 📄 **[`05-static-dynamic-analysis-integration.md`](./05-static-dynamic-analysis-integration.md)** | Correlating Ghidra static analysis with x64dbg/GDB dynamic register inspection and runtime memory state dumping. | 🟢 Completed |

---

## ⚙️ The 3 Daily Continuous Side Tracks (Month 11 Focus)

Hamare daily 12-hour engine ke 3 parallel tracks ke dedicated notes:

### 1. 💻 Systems C/C++ Track (1 Hour Daily)
* **Goal:** Intentional code obfuscation and manual reversal.
* **Topics:** Writing custom C programs with intentional obfuscation (function pointer arrays, computed gotos, state-machine dispatchers); compiling with `-O2` and reversing them in Ghidra.

### 2. 🧩 Assembly & Reverse Engineering Track (1 Hour Daily)
* **Goal:** Complex x86-64 string and vector instructions.
* **Topics:** Dissecting fast string operations (`rep movsb`, `rep stosb`, `cmpsb`), SIMD vector operations (`AVX2`, `SSE4.2`), and vectorized memory comparisons in assembly.

### 3. 🔌 Hardware & Architecture Track (1 Hour Daily)
* **Goal:** CPU instruction decoding micro-operations (µops).
* **Topics:** Complex Instruction Set (CISC) hardware decoding, micro-operation (µop) cache, instruction decode queue, and superscalar execution pipelines.

---

## 🛠️ Lab Environments & Hands-On Milestones

* 🎯 **CrackMe Resolution Portfolio:** 10 complex binaries reversed across varying difficulty levels (from Crackme.one / Reversing.kr); full algorithm reconstructions documented and custom keygens written in Python.
* 🎯 **Automated Ghidra String Deobfuscator:** Custom Python Ghidra script that automatically traverses binary code sections, locates encrypted string arrays, and decrypts custom XOR/RC4 tables.
* 🎯 **Proprietary Protocol File Parser:** Unknown closed-source file format reverse engineered from a binary parser, generating an exact, compilation-ready C structure definition.

---

## 📖 Primary Learning References
* 📘 *Practical Reverse Engineering: x86, x64, ARM, Windows Kernel, Reversing Tools, and Obfuscation* — Bruce Dang, Alexandre Gazet, Elias Bachaalany, Sébastien Josse
* 📘 *Practical Binary Analysis* — Dennis Andriesse
* 💻 *pwn.college* (Reverse Engineering Track)
* 💻 *Crackme.one & Reversing.kr Platforms*

---

© **Muhammad Imran (Founder, IW Cyber Ops)** | Documented for Absolute Depth & Intellectual Rigor.
