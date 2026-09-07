<!-- 
SEO METADATA & KEYWORDS (Invisible to readers, visible to Google Crawlers)
Keywords: IW Cyber Ops, Muhammad Imran, Reverse Engineering, Ghidra Decompilation, Disassembly Analysis, Binary Analysis, Control Flow Graphs, Ghidra Python Scripting, x86-64 Disassembly, CrackMe Solving, Deobfuscation, Cybersecurity Knowledge Base.
-->

# 🧩 Month 11: Reverse Engineering I – Disassembly, Control Flow & Ghidra Mastery

> **Knowledge Base Directory:** Phase 02 / Month 11  
> **System Operator & Author:** Muhammad Imran (Founder, **IW Cyber Ops**)  
> **Objective:** Reconstruct binary application logic, reverse compiler transformations, and automate binary analysis using Ghidra.

---

## 🏛️ The Imperative of Reverse Engineering

Source code is a luxury; binaries are the reality. 

In the high-stakes arena of vulnerability research and malware analysis, security researchers are constantly forced to operate against closed-source targets. When source code is unavailable, **Reverse Engineering (RE) is the ultimate superpower.** It allows an engineer to look past stripped symbols and raw hex bytes to reconstruct high-level algorithms, recover complex data structures, and identify critical security flaws buried within compiled binaries.

True reverse engineering is not merely staring at decompiled C output; it is understanding how compilers transform logic into machine code, how control flow is routed through jump tables, and how CPU micro-operations execute instructions. 

This directory serves as the **IW Cyber Ops Knowledge Base** for Month 11. It documents the transition from basic disassembly reading to advanced static binary analysis, algorithmic reconstruction, and automated Ghidra scripting.

---

## 🧠 Core Domains Documented in this Directory

The notes contained within this module cover the following theoretical and practical pillars:

1. **Disassembly Analysis:** Converting raw opcodes into assembly, resolving cross-references (`Xrefs`), identifying memory offsets, and reconstructing function signatures and missing `struct` definitions.
2. **Control Flow Reconstruction:** Manually mapping and identifying complex high-level control structures: `if`/`else` branches, nested loops, `switch` jump tables, function prologues/epilogues, and compiler tail-call optimizations.
3. **Static & Dynamic Analysis Integration:** Seamlessly correlating static reverse engineering workflows in Ghidra with live dynamic breakpoint execution in GDB and `x64dbg`.
4. **Automated Reverse Engineering:** Writing automated headless scripts in Python and Java via the Ghidra `FlatProgramAPI` to extract encrypted strings, automate data type definitions, and deobfuscate logic.
5. **Advanced x86-64 Instructions:** Analyzing complex string manipulation instructions (`rep movsb`, `rep stosb`, `cmpsb`) and SIMD vector instructions (`AVX`/`SSE`).
6. **Hardware CPU Decoding:** Understanding how the physical CPU decodes assembly instructions into micro-operations (`µops`) and executes them across pipelined execution units.

---

## 📂 Index of Technical Notes

*Below is the living index of all Markdown notes generated during this month's research. Click on any topic to access the detailed documentation.*

| Status | Technical Topic | File Reference |
| :---: | :--- | :--- |
| 📝 | Disassembly Fundamentals & Opcode Translation | `[01-disassembly-opcode-translation.md](./01-disassembly-opcode-translation.md)` |
| 📝 | Control Flow Structures & Jump Tables | `[02-control-flow-jump-tables.md](./02-control-flow-jump-tables.md)` |
| 📝 | Ghidra Workflow: Struct Recovery & Types | `[03-ghidra-struct-recovery.md](./03-ghidra-struct-recovery.md)` |
| 📝 | Automated Ghidra Scripting with Python | `[04-ghidra-scripting-automation.md](./04-ghidra-scripting-automation.md)` |
| 📝 | Algorithmic Reconstruction & Keygenning | `[05-algorithm-reconstruction-keygens.md](./05-algorithm-reconstruction-keygens.md)` |
| 📝 | x86-64 String Ops & CPU Micro-Operations | `[06-x86-string-ops-micro-ops.md](./06-x86-string-ops-micro-ops.md)` |

*(Note: As the month progresses, new `.md` files will be added to this folder and linked above.)*

---

## 🛡️ About the Author

**Muhammad Imran** is an independent systems researcher and the Founder of **IW Cyber Ops**. This knowledge base is an active repository complementing a rigorous 42-month journey engineered for absolute depth, intellectual rigor, and high-impact vulnerability research.

To view the complete overarching roadmap, visit the official [IW-Mission-Control](https://github.com/iwcyberops/IW-Mission-Control) repository.

<br>

---
*Generated & Curated by **IW Cyber Ops** | High-Assurance Cyber Operations & Research*
