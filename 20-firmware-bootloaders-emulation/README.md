# 💾 Month 20: Firmware Security II – Bootloaders, Reverse Engineering & Emulation

> **Research Track:** Phase 03 — Hardware, Firmware, Advanced Fuzzing & Systems  
> **Author & Lead Researcher:** Muhammad Imran (Founder, **IW Cyber Ops**)  
> **Knowledge Base Domain:** `IW-Knowledge-Base/m20-firmware-bootloaders-emulation`

---

## 🧭 Why Firmware Reverse Engineering & Full-System Emulation Matter

Hardware extraction ke baad jo agla critical challenge hota hai, wo hai **Binary Firmware Images ko unpack karna, unke proprietary filesystems ko dissect karna, aur unhe without physical hardware emulate karna**.

Physical devices par directly attack karna slow aur risky hota hai (device brick hone ka khatra hota hai). Isliye ek elite researcher QEMU ke zariye **Full-System aur User-Mode Emulation environments** build karta hai taake GDB ke sath deep debugging aur automated vulnerability discovery ki ja sake. Is month me hum seekhenge:
1. **Firmware Formats & Static Extraction:** Monolithic images vs. Embedded Linux systems, SquashFS, CramFS, JFFS2 filesystems, `binwalk` signature carving, aur proprietary compression/encryption header identification.
2. **Bootloader Internals & Environment Manipulation:** U-Boot architecture, `bootargs` manipulation, serial console hijacking, initramfs loading, aur kernel boot parameters tampering.
3. **Firmware Emulation Engineering with QEMU:** MIPS/ARM binaries ko user-mode (`qemu-arm`, `qemu-mips`) aur full-system mode (`qemu-system-*`) me run karna; aur missing physical hardware NVRAM partitions ko **Custom C Shared Libraries (`libnvram.so`)** hook karke fake responses create karna.
4. **Embedded Vulnerability Discovery:** Web server daemons (e.g., GoAhead, Boa, Lighttpd) me command injections, hardcoded backdoor credentials, insecure UPnP daemons, aur memory corruption bugs ko dhoondna.

Ye month humein closed-source embedded devices ko software lab me emulate karke todne ka expert banata hai.

---

## 📚 Month 20 Knowledge Base & Topic Notes Directory

Is folder me Month 20 ke dauran banaye gaye tamam technical notes topics ke mutabiq categorized hain:

| Note File | Core Focus & Concepts Covered | Status |
| :--- | :--- | :---: |
| 📄 **[`01-firmware-extraction-binwalk-carving.md`](./01-firmware-extraction-binwalk-carving.md)** | Binwalk signature scanning, manual magic byte carving, SquashFS extraction, and decompressing raw kernel images. | 🟢 Completed |
| 📄 **[`02-uboot-bootloaders-bootargs-hijack.md`](./02-uboot-bootloaders-bootargs-hijack.md)** | U-Boot commands, modifying `bootargs` for `/bin/sh` init hijack, flashing custom boot images, and firmware patching. | 🟢 Completed |
| 📄 **[`03-qemu-user-system-emulation.md`](./03-qemu-user-system-emulation.md)** | User-mode vs Full-system QEMU setup, bridging virtual network interfaces (`tun`/`tap`), and emulating MIPS/ARM systems. | 🟢 Completed |
| 📄 **[`04-nvram-interception-shared-libraries.md`](./04-nvram-interception-shared-libraries.md)** | Intercepting `nvram_get`/`nvram_set` via `LD_PRELOAD` C wrappers (`libnvram.so`), faking hardware peripheral registers. | 🟢 Completed |
| 📄 **[`05-embedded-vulnerability-discovery-gdb.md`](./05-embedded-vulnerability-discovery-gdb.md)** | Attaching `gdb-multiarch` to emulated daemons, discovering command injections, and finding IoT memory corruptions. | 🟢 Completed |

---

## ⚙️ The 3 Daily Continuous Side Tracks (Month 20 Focus)

Hamare daily 12-hour engine ke 3 parallel tracks ke dedicated notes:

### 1. 💻 Systems C Track (1 Hour Daily)
* **Goal:** Hardware emulation hooking libraries in C.
* **Topics:** Writing robust C interceptor shared libraries (`libnvram.so`) to hook proprietary hardware abstraction calls via `dlsym(RTLD_NEXT)` and satisfy missing system configurations.

### 2. 🧩 Assembly & Reverse Engineering Track (1 Hour Daily)
* **Goal:** MIPS and ARM firmware binary reverse engineering.
* **Topics:** Reversing stripped MIPS/ARM router binaries inside Ghidra; analyzing branch delay slots in MIPS, conditional instruction execution in ARM, and identifying un-symbolized function sinks.

### 3. 🔌 Hardware & Architecture Track (1 Hour Daily)
* **Goal:** Flash Translation Layers (FTL) and NAND/NOR storage physics.
* **Topics:** NAND vs. NOR flash architectures, wear leveling algorithms, bad block management, Flash Translation Layers (FTL), and raw bit-flip corruption mechanisms.

---

## 🛠️ Lab Environments & Hands-On Milestones

* 🎯 **Full Firmware Emulation Pipeline:** Extracted a commercial ARM-based router firmware filesystem, emulated its native HTTP management daemon inside QEMU with network bridging, and attached `gdb-multiarch` remotely for live execution inspection.
* 🎯 **Firmware Backdoor Implantation:** Extracted a live firmware image, embedded an administrative persistent reverse shell into the initialization startup scripts (`/etc/init.d/`), successfully repacked the SquashFS filesystem, and booted the modified firmware image.
* 🎯 **Embedded 0-Day Bug Discovery in Emulated Daemon:** Discovered, isolated, and wrote a reliable proof-of-concept for an unauthenticated command injection / memory corruption vulnerability inside an emulated IoT device service.

---

## 📖 Primary Learning References
* 📘 *Practical IoT Hacking: The Definitive Guide to Attacking the Internet of Things* — Fotios Chantzis et al.
* 📜 *Embedded Linux System Design and Development* — P. Raghavan et al.
* 💻 *Azeria Labs (ARM Assembly & Shellcoding Basics)*
* 💻 *QEMU & Firmadyne Official Emulation Documentation*

---

© **Muhammad Imran (Founder, IW Cyber Ops)** | Documented for Absolute Depth & Intellectual Rigor.
