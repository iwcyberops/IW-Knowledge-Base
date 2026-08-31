# ⚙️ Month 04: OS Internals, Virtual Memory, Process Management & C Memory

> **Research Track:** Phase 01 — Foundations & Systems Architecture  
> **Author & Lead Researcher:** Muhammad Imran (Founder, **IW Cyber Ops**)  
> **Knowledge Base Domain:** `IW-Knowledge-Base/m04-os-internals-virtual-memory`

---

## 🧭 Why OS Internals & Virtual Memory are the Core of Exploitation

Source code aur physical hardware ke darmiyan jo sab se critical boundary hai, wo **Operating System Kernel aur Virtual Memory** hai.

Jab tak aapko process memory ki internal geography ka ilm nahi hoga, tab tak Binary Exploitation (Buffer Overflows, ROP, Heap corruption) sirf ek andhera teer hoga. Ek Elite Vulnerability Researcher ke liye ye samajhna zaroori hai ke:
1. **CPU Rings & Privilege Separation:** Ring 3 (User Space) se Ring 0 (Kernel Space) me transition system calls (`syscall`) ke zariye kaise hoti hai.
2. **Virtual Memory Translation:** CPU ka MMU (Memory Management Unit) aur 4-Level Page Tables kis tarah virtual addresses ko physical RAM me map karte hain.
3. **Process Address Layout:** `.text` (code), `.data` (initialized globals), `.bss`, Heap (jo upar grow hota hai), aur Stack (jo neeche grow hota hai) ke darmiyan memory boundaries kaise kaam karti hain.
4. **Dynamic Memory Lifecycle:** `malloc()` aur `free()` ke internal metadata headers, heap chunks, dangling pointers, aur Use-After-Free (UAF) ke roots kahan bante hain.

Ye month humein raw program execution ki anatomy ka master banata hai.

---

## 📚 Month 04 Knowledge Base & Topic Notes Directory

Is folder me Month 04 ke dauran banaye gaye tamam technical notes topics ke mutabiq categorized hain:

| Note File | Core Focus & Concepts Covered | Status |
| :--- | :--- | :---: |
| 📄 **[`01-cpu-rings-privilege-separation.md`](./01-cpu-rings-privilege-separation.md)** | Ring 0 vs Ring 3 execution, hardware interrupts, context switching, and system call dispatch mechanics. | 🟢 Completed |
| 📄 **[`02-virtual-memory-page-tables.md`](./02-virtual-memory-page-tables.md)** | MMU, multi-level Page Tables, Page Directories, TLB caching, virtual-to-physical address translation, and demand paging. | 🟢 Completed |
| 📄 **[`03-process-memory-layout.md`](./03-process-memory-layout.md)** | In-depth dissection of `/proc/[pid]/maps`, `.text`, `.rodata`, `.data`, `.bss`, Stack frame boundaries, and Heap growth. | 🟢 Completed |
| 📄 **[`04-syscalls-ipc-process-control.md`](./04-syscalls-ipc-process-control.md)** | File descriptors, `fork()`, `vfork()`, `execve()`, `waitpid()`, anonymous pipes, FIFOs, and POSIX shared memory. | 🟢 Completed |
| 📄 **[`05-dynamic-memory-allocator-internals.md`](./05-dynamic-memory-allocator-internals.md)** | `malloc()`, `calloc()`, `realloc()`, `free()`, chunk metadata, memory fragmentation, dangling pointers, and undefined behavior. | 🟢 Completed |

---

## ⚙️ The 3 Daily Continuous Side Tracks (Month 04 Focus)

Hamare daily 12-hour engine ke 3 parallel tracks ke dedicated notes:

### 1. 💻 Systems C Track (1 Hour Daily)
* **Goal:** Custom memory allocator wrapper and metadata tracking.
* **Topics:** Writing a custom C memory management wrapper that embeds header metadata (allocation size, canary guards) to detect out-of-bounds writes.

### 2. 🧩 Assembly & Reverse Engineering Track (1 Hour Daily)
* **Goal:** Stack frame initialization and destruction.
* **Topics:** Dissecting function prologues (`push rbp; mov rbp, rsp; sub rsp, N`) and epilogues (`leave; ret`), base pointer alignment, and local variable stack offsets.

### 3. 🔌 Hardware & Architecture Track (1 Hour Daily)
* **Goal:** Memory bus architecture and cache hierarchies.
* **Topics:** SRAM vs. DRAM physics, CPU memory buses, memory bus contention, L1/L2/L3 cache architectures, and direct-mapped vs. set-associative cache mapping.

---

## 🛠️ Lab Environments & Hands-On Milestones

* 🎯 **Custom UNIX Shell in C:** Fully functional command-line interpreter supporting command execution, multi-stage pipes (`|`), and I/O redirection (`<`, `>`).
* 🎯 **Process Memory Map Extractor:** A native C utility parsing `/proc/[pid]/maps` in real-time, dumping memory segment permissions (`rwxp`) for live targets.
* 🎯 **Valgrind Memory Leak Profiler:** Rigorous leak detection and memory profiling reports identifying heap leaks and uninitialized pointer reads.

---

## 📖 Primary Learning References
* 📘 *Computer Systems: A Programmer's Perspective (CS:APP)* — Randal E. Bryant & David R. O'Hallaron
* 📘 *The Linux Programming Interface (TLPI)* — Michael Kerrisk

---

© **Muhammad Imran (Founder, IW Cyber Ops)** | Documented for Absolute Depth & Intellectual Rigor.
