<!-- 
SEO METADATA & KEYWORDS (Invisible to readers, visible to Google Crawlers)
Keywords: IW Cyber Ops, Muhammad Imran, OS Internals, Virtual Memory Architecture, Process Management, C Dynamic Memory, Malloc Internals, Ring 0 vs Ring 3, System Calls, Cache Mapping, Memory Leaks, Valgrind, Cybersecurity Knowledge Base.
-->

# ⚙️ Month 04: OS Internals, Virtual Memory, Process Management & C Memory

> **Knowledge Base Directory:** Phase 01 / Month 04  
> **System Operator & Author:** Muhammad Imran (Founder, **IW Cyber Ops**)  
> **Objective:** Understand the boundary between source code, virtual process memory, CPU execution rings, and operating system kernels.

---

## 🏛️ The Imperative of Operating System Internals

Software does not execute in a vacuum; it operates under the strict governance of the Operating System and CPU architecture. 

To discover zero-days, exploit memory corruption, or build advanced system tools, one must stop viewing computers through the lens of a high-level programmer. An elite vulnerability researcher must understand exactly how a binary is loaded, how virtual addresses are translated into physical silicon, how the kernel isolates processes, and how the C standard library orchestrates heap memory dynamically.

This directory serves as the **IW Cyber Ops Knowledge Base** for Month 04. It strips away the magic of computing and documents the harsh realities of OS internals, memory management, and process execution. Here, we transition from writing code to dissecting the exact environment in which code lives and breathes.

---

## 🧠 Core Domains Documented in this Directory

The notes contained within this module cover the following theoretical and practical pillars:

1. **Privilege Separation & Execution Rings:** Understanding the absolute boundary between Kernel Space (Ring 0) and User Space (Ring 3), context switching mechanics, hardware interrupts, and the critical System Call transition.
2. **Virtual Memory Architecture:** The mathematics of memory: Multi-level Page Tables, Page Directories, Translation Lookaside Buffers (TLB), Virtual-to-Physical translation, the MMU, demand paging, and swapping.
3. **Process Memory Layout:** Mapping a live binary in RAM—identifying the Text (executable), Data (initialized), BSS (uninitialized), Heap (dynamic upward growth), and Stack (downward call frames) segments.
4. **UNIX Process Management & IPC:** Tracing process lifecycles via `fork()`, `vfork()`, `execve()`, and `waitpid()`. Establishing Inter-Process Communication using file descriptors, anonymous pipes, FIFOs, and POSIX shared memory.
5. **Dynamic Memory Management in C:** Analyzing the internal workings of `malloc()`, `calloc()`, `realloc()`, and `free()`. Tracking pointer lifetimes, dangling pointers, undefined behaviors, and triaging memory leaks using `Valgrind`.

---

## 📂 Index of Technical Notes

*Below is the living index of all Markdown notes generated during this month's research. Click on any topic to access the detailed documentation.*

| Status | Technical Topic | File Reference |
| :---: | :--- | :--- |
| 📝 | Execution Rings, Context Switches & System Calls | `[01-execution-rings-syscalls.md](./01-execution-rings-syscalls.md)` |
| 📝 | Virtual Memory, Page Tables & The MMU | `[02-virtual-memory-page-tables.md](./02-virtual-memory-page-tables.md)` |
| 📝 | Process Memory Layout: Text, BSS, Heap & Stack | `[03-process-memory-layout.md](./03-process-memory-layout.md)` |
| 📝 | Process Lifecycle (`fork`/`execve`) & UNIX IPC | `[04-process-lifecycle-ipc.md](./04-process-lifecycle-ipc.md)` |
| 📝 | C Dynamic Memory: `malloc` Internals & Valgrind | `[05-c-dynamic-memory-valgrind.md](./05-c-dynamic-memory-valgrind.md)` |
| 📝 | Assembly Stack Frames & Hardware Cache Mapping | `[06-asm-stack-hardware-cache.md](./06-asm-stack-hardware-cache.md)` |

*(Note: As the month progresses, new `.md` files will be added to this folder and linked above.)*

---

## 🛡️ About the Author

**Muhammad Imran** is an independent systems researcher and the Founder of **IW Cyber Ops**. This knowledge base is an active repository complementing a rigorous 42-month journey engineered for absolute depth, intellectual rigor, and high-impact vulnerability research.

To view the complete overarching roadmap, visit the official [IW-Mission-Control](https://github.com/iwcyberops/IW-Mission-Control) repository.

<br>

---
*Generated & Curated by **IW Cyber Ops** | High-Assurance Cyber Operations & Research*
