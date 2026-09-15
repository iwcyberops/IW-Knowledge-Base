<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 01 — Linux Kernel Interface & Core CLI
   DOCUMENT: Day 16 — Kernel Modules, Hardware & Sysctl Runtime Tuning
   ========================================================================= -->

# 🛡️ Day 16: Kernel Modules, Hardware & Sysctl Runtime Tuning

> **IW Cyber Ops Research Vault | Module 01: Linux & Systems Foundations**  
> *Author: Muhammad Imran (@iwcyberops)*  
> *Track: Kernel Ring Architecture, Loadable Modules (LKM), Sysfs & Kernel Hardening*

---

## 1. Kernel Architecture: Monolithic vs LKMs

The Linux kernel is **Monolithic**: the entire operating system core executes in a single privileged address space. To avoid recompiling the massive kernel every time new hardware is added, Linux uses **Loadable Kernel Modules (LKMs)**—dynamic object files (`.ko`) loaded and unloaded from kernel memory at runtime.

```
                  ┌────────────────────────────────────────────────────────┐
                  │                 USER SPACE (Ring 3)                    │
                  │   User Applications, Shells, Daemons, Glibc Libraries │
                  └──────────────────────────┬─────────────────────────────┘
                                             │  (System Calls / syscalls)
                  ═══════════════════════════╪══════════════════════════════
                                             │  (Hardware Privilege Boundary)
                  ┌──────────────────────────▼─────────────────────────────┐
                  │                KERNEL SPACE (Ring 0)                   │
                  │                                                        │
                  │   ┌────────────────────────────────────────────────┐   │
                  │   │             Core Monolithic Kernel             │   │
                  │   │     (Process Scheduler, VFS, Memory Manager)   │   │
                  │   └───────────────────────┬────────────────────────┘   │
                  │                           │                            │
                  │               [ Dynamic LKM Interface ]                │
                  │                           │                            │
                  │    ┌──────────────┬───────┴──────┬──────────────┐      │
                  │    ▼              ▼              ▼              ▼      │
                  │ [ Net Driver ] [ GPU LKM ] [ Filesystem ] [ LKM Rootkit]│
                  └──────────────────────────┬─────────────────────────────┘
                                             │
                                             ▼
                                    [ PHYSICAL HARDWARE ]
```

---

## 2. Hardware Bus Enumeration & Diagnostics

The kernel discovers hardware through peripheral buses and registers device nodes in `/sys/` (sysfs) and `/proc/`.

### 1. `lspci` (PCI / PCIe Bus Inspection)
Displays all devices connected via the PCI/PCIe bus (Network cards, GPUs, NVMe controllers, Host bridges).

* `lspci` — One-line summary of all detected PCI hardware.
* `lspci -v` — Verbose mode showing IRQs, I/O memory ranges, and capabilities.
* `lspci -k` — **Crucial Flag:** Displays the active **Kernel Driver** and **Kernel Modules** controlling each PCI device.
* `lspci -nn` — Output raw vendor and device ID codes (useful for finding custom exploit/firmware targets).

---

### 2. `lsusb` (Universal Serial Bus Enumeration)
Lists USB host controllers and connected peripherals.

* `lsusb` — Print connected USB bus numbers, device IDs, and vendor names.
* `lsusb -t` — Hierarchical tree view showing physical USB ports, speeds (480M, 5G, 10G), and active drivers.
* `lsusb -v` — Deep inspection of device descriptors, power consumption, and endpoints.

---

### 3. CPU & Memory Diagnostics (`lscpu`, `dmidecode`)
* `lscpu` — Comprehensive CPU architecture report (Cores, Sockets, Cache L1/L2/L3, Vulnerability mitigations like Meltdown/Spectre).
  * Look for the **`Virtualization`** flag (`VT-x` / `AMD-V`) and the **`nx` (No-Execute / DEP)** security bit in `/proc/cpuinfo`.
* `sudo dmidecode` — DMI/SMBIOS table decoder (Extracts serial numbers, BIOS version, motherboard model, and RAM slot hardware specs).
  * `sudo dmidecode -t bios` — Inspect BIOS/UEFI firmware version and release date.
  * `sudo dmidecode -t memory` — Inspect physical RAM stick capacities, speeds, and slot occupancy.

---

## 3. Loadable Kernel Module (LKM) Management

Kernel modules are stored on disk under: `/lib/modules/$(uname -r)/kernel/`.

```
/lib/modules/$(uname -r)/
├── kernel/
│   ├── drivers/          --> Hardware and peripheral drivers (net, gpu, usb)
│   ├── fs/               --> Filesystem drivers (ext4, btrfs, nfs)
│   └── net/              --> Network protocol stacks (ipv4, ipv6, wireguard)
└── modules.dep           --> Binary dependency resolution map
```

---

### Core LKM CLI Commands

```bash
# 1. List Loaded Kernel Modules (lsmod)
lsmod                                 # Reads /proc/modules (Module name, RAM size, dependent modules)

# 2. Inspect Module Metadata (modinfo)
modinfo wireguard                     # View author, license, description, parameters, and on-disk path
modinfo -p e1000e                     # List configurable runtime parameters accepted by the module

# 3. Intelligent Module Loading (modprobe - Recommended)
sudo modprobe wireguard               # Auto-resolves dependencies via modules.dep and loads into kernel
sudo modprobe -r wireguard            # Safely unloads module and unused dependent modules
sudo modprobe -v dummy                # Verbose mode: prints the exact insmod calls executed

# 4. Low-Level Manual Module Insertion (insmod / rmmod)
sudo insmod /path/to/custom_mod.ko    # Inserts raw .ko binary directly (Does NOT resolve dependencies)
sudo rmmod custom_mod                 # Force unloads module from kernel memory
```

---

### Module Blacklisting (Disabling Vulnerable Drivers)
To permanently prevent a kernel module from auto-loading (e.g., blocking firewire, thunderbolt, or vulnerable legacy filesystems):

```bash
# Append blacklist directive to modprobe configurations:
echo "blacklist cramfs" | sudo tee /etc/modprobe.d/blacklist-cramfs.conf
echo "install cramfs /bin/true" | sudo tee -a /etc/modprobe.d/blacklist-cramfs.conf

# Rebuild initramfs to apply changes to early boot:
sudo update-initramfs -u
```

---

## 4. Live Kernel Runtime Tuning: `sysctl` & `/proc/sys/`

The `/proc/sys/` virtual filesystem allows reading and writing kernel parameters on-the-fly without rebooting. The `sysctl` utility provides the administrative interface.

$$\text{Parameter Path: } /proc/sys/\mathbf{net/ipv4/ip\_forward} \iff \text{sysctl key: } \mathbf{net.ipv4.ip\_forward}$$

```bash
# 1. Querying Runtime Kernel State
sysctl -a                             # Dump ALL active kernel runtime parameters
sysctl net.ipv4.ip_forward            # Query specific parameter value

# 2. Temporary Runtime Modification (Lost on Reboot)
sudo sysctl -w net.ipv4.ip_forward=1  # Enable kernel IP routing (Used in Man-in-the-Middle/Routing)
# (Alternative direct write):
echo "1" | sudo tee /proc/sys/net/ipv4/ip_forward

# 3. Permanent Modification across Reboots
# Add entry to /etc/sysctl.conf or /etc/sysctl.d/99-security.conf
sudo sysctl -p                        # Reload and apply /etc/sysctl.conf immediately
```

---

### 🛡️ Critical Kernel Security Parameters

| Kernel Parameter | Recommended Value | Security & Cyber Ops Function |
| :--- | :---: | :--- |
| `kernel.randomize_va_space` | **`2`** | **Full ASLR:** Randomizes Stack, VDSO, Heap, and Shared Library base addresses. |
| `kernel.dmesg_restrict` | **`1`** | Blocks non-root users from reading `dmesg` (Prevents kernel address leak recon). |
| `kernel.kptr_restrict` | **`2`** | Hides raw kernel memory pointers in `/proc/kallsyms` (Mitigates KASLR bypass exploits). |
| `net.ipv4.tcp_syncookies` | **`1`** | Defends kernel networking stack against **TCP SYN Flood Denial-of-Service** attacks. |
| `net.ipv4.ip_forward` | **`0` / `1`** | `0` = Host only. Set to `1` by attackers during ARP spoofing / MITM operations. |
| `fs.protected_symlinks` | **`1`** | Mitigates standard symlink/hardlink race-condition attacks in `/tmp`. |

---

## 5. Ring 0 Cyber Operations: LKM Rootkits

Because code executing in **Ring 0** shares the exact same privilege level as the core kernel, a malicious LKM (LKM Rootkit) possesses absolute power over userland.

```
┌────────────────────────────────────────────────────────────────────────┐
│                   RING 0 ROOTKIT CAPABILITIES                          │
├──────────────────────────┬─────────────────────────────────────────────┤
│ 1. Syscall Table Hooking │ Intercepts sys_read, sys_getdents64         │
│ 2. Process Cloaking      │ Unlinks attacker tasks from the active      │
│                          │ kernel process list (Invisible to ps/top)   │
│ 3. Network Port Hiding   │ Hooks /proc/net/tcp to hide active C2 ports │
│ 4. File Invisibility     │ Filters directory listings in kernel memory │
└──────────────────────────┴─────────────────────────────────────────────┘
```

---

### Rootkit Detection & Kernel Integrity Verification

1. **Kernel Taint Analysis:**
   The kernel flags itself as "tainted" if an unverified, non-GPL, or forced module is loaded:
   ```bash
   cat /proc/sys/kernel/tainted
   # Value 0 = Clean / Untainted. Non-zero = Proprietary or unsigned module loaded.
   ```
2. **Hidden LKM Detection (Module List Discrepancy):**
   Rootkits often remove their entry from `/proc/modules` (making `lsmod` blind). Incident responders detect this by scanning `/sys/module/` which operates on a separate kernel kobject structure:
   ```bash
   diff <(lsmod | awk '{print $1}' | sort) <(ls -1 /sys/module/ | sort)
   ```
3. **Module Signing Enforcement (`Kernel Lockdown`):**
   Modern hardened kernels enforce cryptographic signature verification on all LKMs via UEFI Secure Boot:
   ```bash
   cat /sys/kernel/security/lockdown
   # Output: [none] integrity confidentiality (If 'integrity' is active, unsigned LKMs are rejected).
   ```

---

## 6. Incident Response & Kernel Audit Matrix

| Tactical Objective | Command Syntax | Source / Interface |
| :--- | :--- | :--- |
| **Inspect Driver for PCI Device** | `lspci -nnk` | Kernel Device Drivers |
| **Check ASLR Status** | `cat /proc/sys/kernel/randomize_va_space` | Memory Hardening Engine |
| **List Exported Kernel Symbols** | `sudo cat /proc/kallsyms \| head -n 20` | Kernel Symbol Table |
| **Inspect Module Dependencies** | `cat /lib/modules/$(uname -r)/modules.dep` | Kernel Dependency Map |
| **Reload Sysctl Configurations** | `sudo sysctl --system` | `/etc/sysctl.d/` Pipeline |
| **Inspect Tainted Kernel Bits** | `cat /proc/sys/kernel/tainted` | Kernel Integrity Flag |

---

<!-- =========================================================================
   [IW CYBER Ops] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
