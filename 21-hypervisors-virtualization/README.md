<!-- 
SEO METADATA & KEYWORDS (Invisible to readers, visible to Google Crawlers)
Keywords: IW Cyber Ops, Muhammad Imran, Hypervisor Security, Virtualization Architecture, VM Escape Vulnerabilities, Intel VMX VMCS, AMD SVM, QEMU KVM Internals, MMIO Emulation, Extended Page Tables EPT, SLAT, Cloud Security, Cybersecurity Knowledge Base.
-->

# 🛡️ Month 21: Hypervisor Architecture & Virtualization Security Foundations

> **Knowledge Base Directory:** Phase 03 / Month 21  
> **System Operator & Author:** Muhammad Imran (Founder, **IW Cyber Ops**)  
> **Objective:** Understand hardware-assisted virtualization, hypervisor internal boundaries, and guest-to-host attack surfaces.

---

## 🏛️ The Imperative of Virtualization Security

Virtualization is the invisible bedrock of modern cloud computing, multi-tenant data centers, and isolated security sandboxes.

When multiple untrusted guest operating systems share physical hardware, the **Hypervisor is the ultimate security boundary.** Finding a bug in a guest kernel grants ring 0 privileges within the virtual machine, but executing a **Virtual Machine Escape (VM Escape)** to compromise the bare-metal host kernel represents the pinnacle of high-impact exploitation.

A hypervisor escape is not a generic software flaw; it occurs at the intersection of complex hardware virtualization extensions (**Intel VMX / AMD SVM**) and software-emulated virtual devices. When an untrusted guest communicates via Memory-Mapped I/O (MMIO), Port I/O (PIO), or Direct Memory Access (DMA), the hypervisor must emulate legacy hardware peripherals (such as virtual NICs, floppy controllers, and USB hubs). Flaws in these C emulation loops allow an attacker to break out of the guest VM completely.

This directory serves as the **IW Cyber Ops Knowledge Base** for Month 21. It documents the low-level dissection of hardware-assisted virtualization, QEMU/KVM source code architecture, MMIO dispatch routines, and guest-to-host exploit vectors.

---

## 🧠 Core Domains Documented in this Directory

The notes contained within this module cover the following theoretical and practical pillars:

1. **Hardware Virtualization Extensions:** Dissecting Intel VMX (Virtual Machine Control Structure - `VMCS`, Root vs. Non-Root operation, VM Entries, and VM Exits) alongside AMD SVM (`VMCB`).
2. **Hypervisor Taxonomy & Architecture:** Differentiating Type 1 (bare-metal: Xen, ESXi) from Type 2 (hosted: KVM/QEMU, VMware Workstation) hypervisors, and auditing the Linux KVM kernel module.
3. **Guest-to-Host Attack Surfaces:** Mapping all communication vectors between guest and host—focusing on virtual device emulation (virtual NICs, storage controllers, PCI buses), shared memory channels, and MMIO/PIO dispatch handlers.
4. **VM Escape Vulnerability Classes:** Analyzing historical and modern escape primitives, including out-of-bounds array indexing in device loops, unvalidated DMA buffers, race conditions in VM exit handlers, and dissecting the infamous **VENOM** (CVE-2015-3456) floppy controller vulnerability.
5. **Low-Level VMX Assembly:** Analyzing assembly instructions that control CPU virtualization states: `VMLAUNCH`, `VMRESUME`, `VMREAD`, and `VMWRITE`.
6. **Memory Virtualization & SLAT:** Deconstructing Second Level Address Translation (SLAT), Extended Page Tables (EPT), Nested Page Tables (NPT), and physical-to-guest memory mapping mechanisms.

---

## 📂 Index of Technical Notes

*Below is the living index of all Markdown notes generated during this month's research. Click on any topic to access the detailed documentation.*

| Status | Technical Topic | File Reference |
| :---: | :--- | :--- |
| 📝 | Hardware Virtualization: Intel VMX & VMCS Internals | `[01-intel-vmx-vmcs-architecture.md](./01-intel-vmx-vmcs-architecture.md)` |
| 📝 | Hypervisor Architectures: Type 1 vs Type 2 & KVM | `[02-hypervisor-types-kvm-architecture.md](./02-hypervisor-types-kvm-architecture.md)` |
| 📝 | QEMU Virtual Device Emulation & MMIO/PIO Handling | `[03-qemu-device-emulation-mmio.md](./03-qemu-device-emulation-mmio.md)` |
| 📝 | VM Escape Vulnerability Classes & VENOM Case Study | `[04-vm-escape-vulnerabilities-venom.md](./04-vm-escape-vulnerabilities-venom.md)` |
| 📝 | Writing Custom Virtual Hardware Devices in C (QEMU) | `[05-writing-qemu-virtual-hardware.md](./05-writing-qemu-virtual-hardware.md)` |
| 📝 | Memory Virtualization: Extended Page Tables (EPT) | `[06-memory-virtualization-ept-slat.md](./06-memory-virtualization-ept-slat.md)` |

*(Note: As the month progresses, new `.md` files will be added to this folder and linked above.)*

---

## 🛡️ About the Author

**Muhammad Imran** is an independent systems researcher and the Founder of **IW Cyber Ops**. This knowledge base is an active repository complementing a rigorous 42-month journey engineered for absolute depth, intellectual rigor, and high-impact vulnerability research.

To view the complete overarching roadmap, visit the official [IW-Mission-Control](https://github.com/iwcyberops/IW-Mission-Control) repository.

<br>

---
*Generated & Curated by **IW Cyber Ops** | High-Assurance Cyber Operations & Research*
