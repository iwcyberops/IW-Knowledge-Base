<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 01 — Linux Kernel Interface & Core CLI
   DOCUMENT: Day 02 — Linux Filesystem Hierarchy Standard (FHS) & Security Internals
   ========================================================================= -->

# 🛡️ Day 02: Linux Filesystem Hierarchy Standard (FHS) & Security Internals

> **IW Cyber Ops Research Vault | Module 01: Linux & Systems Foundations**  
> *Author: Muhammad Imran (@iwcyberops)*  
> *Track: Core Linux Architecture, Filesystem Internals & Threat Surface Analysis*

---

## 1. Linux Filesystem Hierarchy (Tree Overview)

In Linux, everything originates from the root (`/`) directory. Under the **Filesystem Hierarchy Standard (FHS)**, files and directories are organized strictly based on their volatility (static vs. dynamic) and necessity for system booting.

```text
/ (Root Directory)
├── bin          --> Essential user command binaries (ls, cp, ping)
├── boot         --> Static bootloader, initramfs, & Linux kernel image (vmlinuz)
├── dev          --> Device nodes representing hardware & pseudo-devices
├── etc          --> System-wide configuration files & startup scripts
├── home         --> User personal storage and individual workspaces
├── lib / lib64  --> Shared system libraries & dynamic kernel modules
├── media        --> Mount point for auto-mounted removable media (USBs, CD-ROMs)
├── mnt          --> Temporary manual mount points for administrators
├── opt          --> Optional / third-party vendor standalone software
├── proc         --> Virtual filesystem exposing kernel & runtime process state (RAM)
├── root         --> Home directory for the Superuser / Root administrator
├── run          --> Volatile runtime data, active process locks, and domain sockets
├── sbin         --> Essential system administration binaries (fdisk, reboot, iptables)
├── srv          --> Site-specific data served by this system (FTP, Web)
├── sys          --> Virtual filesystem exposing kernel hardware tree & driver models
├── tmp          --> Volatile temporary file storage (World-writable + Sticky Bit)
├── usr          --> Secondary hierarchy for user utilities and shared read-only data
│   ├── bin      --> Non-essential general user binaries
│   ├── sbin     --> Non-essential admin binaries
│   ├── lib      --> 32/64-bit application libraries
│   └── share    --> Architecture-independent shared data (man pages, wordlists)
└── var          --> Variable data directory (logs, mail spools, web root, cron)
    ├── log      --> System, auth, and service log files
    ├── www      --> Standard web-server root
    ├── tmp      --> Temporary files preserved across system reboots
    └── spool    --> Queued tasks (cron, mail, printing)
```

---

## 2. Comprehensive Directory Breakdown & Security Vectors

### 1. `/` (Root Directory)
* **Description:** Top-level node of the directory tree. Contains base directories required to boot the system before secondary partitions are mounted.
* **Security Context:** If root filesystem permissions are modified inappropriately or corrupted, the entire operating system fails to boot.

---

### 2. `/etc` — Configuration Epicenter
* **Description:** Host-specific, system-wide configuration files and initialization scripts. All files here are static text-based configuration entries.

| Key File / Path | Operational Purpose | Security / Exploit Significance |
| :--- | :--- | :--- |
| `/etc/passwd` | Account metadata (UID, GID, Home, Shell). | World-readable reconnaissance target. If misconfigured with write permissions (`w`), attackers can add custom UID `0` root users. |
| `/etc/shadow` | Cryptographic hashes of user account passwords. | Restricted to root (`chmod 640/600`). Ultimate target for offline hash cracking (e.g., John the Ripper / Hashcat). |
| `/etc/sudoers` | Sudo privilege configuration file. | Insecure rules (e.g., `NOPASSWD`) enable instant privilege escalation via GTFOBins. |
| `/etc/crontab` & `/etc/cron.d/` | System-wide automated task schedulers. | Prime persistence location. Attackers inject reverse shells executing at timed intervals. |
| `/etc/ssh/` | OpenSSH daemon configuration and host keys. | Misconfigurations allow root login, weak ciphers, or compromised authorized keys. |
| `/etc/profile` / `/etc/bash.bashrc` | Global environment initialization scripts. | Script poisoning triggers payloads whenever any user establishes an interactive shell. |

---

### 3. `/proc` — Virtual Process & Kernel Runtime Filesystem
* **Description:** Pseudo-filesystem mounted dynamically by the kernel at runtime. Exists entirely in **RAM** (occupies 0 bytes on physical storage). Exposes low-level kernel abstractions and live process states.

#### Important Subdirectories & Security Artifacts:
* `/proc/[PID]/cmdline` — Command line invocation string of the process.  
  > ⚠️ *Security Risk:* Poorly designed scripts leaking cleartext credentials/tokens via CLI arguments.
* `/proc/[PID]/environ` — Environment variables loaded into the process memory space.  
  > ⚠️ *Security Risk:* Cleartext API keys, DB passwords, and secrets leakage.
* `/proc/[PID]/fd/` — Directory of open file descriptors. Attackers inspect open pipes, sockets, and unlinked locked files.
* `/proc/kcore` — Full image of system physical memory represented in ELF format. Extractable via debuggers (GDB) for live RAM forensics and credential harvesting.
* `/proc/self/` — Symbolic link pointing directly to the `/proc` directory of the currently executing process (used heavily in exploit development and local sandbox escapes).
* `/proc/net/tcp` & `/proc/net/udp` — Active socket tables for local port/network reconnaissance without standalone tooling (`netstat`/`ss`).
* `/proc/kallsyms` — Kernel symbol table. Crucial for kernel exploit development (bypassing KASLR).
* `/proc/modules` — List of all currently loaded kernel modules (LKM rootkit detection).

---

### 4. `/dev` — Device Nodes & Hardware Bridge
* **Description:** Special device files representing system hardware, block devices, character streams, and pseudo-devices managed via `udev`.

| Path | Description & Threat Vector |
| :--- | :--- |
| `/dev/sda`, `/dev/nvme0n1` | Raw storage block devices. Root access allows raw disk cloning, direct data carving, or partition wiping via `dd`. |
| `/dev/shm` | Shared memory directory implemented as `tmpfs`. World-writable, RAM-backed space used by attackers to stage in-memory payloads without hitting physical disk (bypasses basic disk forensics). |
| `/dev/null` | Bit bucket / Digital dustbin. Discards all data written to it. |
| `/dev/urandom` / `/dev/random` | Kernel entropy sources for pseudorandom/cryptographic data generation. |
| `/dev/tty` & `/dev/pts/*` | Controlling terminal interfaces and pseudo-terminals (targets for TTY hijacking and credential sniffing). |

---

### 5. `/sys` — Sysfs Virtual Hardware Model
* **Description:** Unified kernel object database exposing device drivers, hardware buses, and real-time kernel control settings.
* `/sys/devices/` — Global hardware device tree used for target system hardware fingerprinting.
* `/sys/fs/cgroup/` — Control group allocations (CPU/Memory limits). Heavily targeted during **Docker / Container Breakout** attacks.
* `/sys/class/net/` — Direct status and MAC configuration of network interfaces.

---

### 6. `/bin`, `/sbin`, `/usr/bin`, `/usr/sbin` — Binary Hierarchies
* `/bin` — Essential single-user mode binaries accessible to all users (`cat`, `ls`, `cp`, `bash`).
* `/sbin` — Essential administrative binaries reserved for system maintenance (`reboot`, `iptables`, `fsck`).
* `/usr/bin` & `/usr/sbin` — Standard user and admin software binaries.

> 🔴 **Threat Vector (Binary Hijacking & Trojans):**  
> If an attacker replaces binaries in these paths (or manipulates the `$PATH` variable), legitimate user commands trigger malicious reverse shells. Forensics teams monitor binary file hashes (`md5sum`/`sha256sum`) and anomalous modification timestamps (`mtime`).

---

### 7. `/lib` & `/lib64` — Shared System Libraries
* **Description:** Dynamic shared libraries (`.so` files) required by binaries during startup, alongside loadable kernel modules.
* `/lib/libc.so.*` — Core C standard library. Tampering completely compromises all native executable execution.
* `/lib/ld-linux.so.*` — Dynamic Linker/Loader. Resolves and maps shared libraries into process memory.
* `/lib/security/pam_*.so` — Pluggable Authentication Modules. Tampering allows complete authentication bypass (backdoored PAM modules allow login with master passwords).

> ⚠️ **Library Hijacking (Shared Object Injection):**  
> Attackers place rogue shared objects in library paths or configure `/etc/ld.so.preload` to intercept system API calls (e.g., hiding processes from `ps`).

---

### 8. `/tmp` vs `/var/tmp` — Temporary Storage
* `/tmp` — Volatile storage wiped on reboot. World-writable (`rwxrwxrwt`) protected by a **Sticky Bit (`t`)** (preventing users from deleting other users' files).
  * *Exploitation:* Universal drop zone for reverse shells, compiler outputs, and exploit payloads.
* `/var/tmp` — Persistent temporary files. Preserved across reboots. Attackers place reboot-surviving persistence scripts here.
* **Database & IPC Sockets:** MySQL (`/tmp/mysql.sock`) or runtime sockets dropped here with weak permissions can lead to unauthorized service hijacking.

---

### 9. `/var` — Variable Data & Dynamic State
* `/var/log/` — Audit and execution logs (`/var/log/auth.log`, `/var/log/syslog`).
  * *Blue Team:* Incident detection, forensics, and tracking.
  * *Red Team:* Primary post-exploitation target for log tampering/clearing.
* `/var/www/` — Default document root for HTTP servers (e.g., Apache/Nginx). Source code review target for web vulnerabilities.
* `/var/spool/` — Spool areas for queued mail and cron jobs.

---

### 10. Secondary & Administrative Mounts
* `/opt` — Third-party proprietary packages. Often contains custom applications with lax permissions leading to local privilege escalation.
* `/run` — Runtime operational information (PID files, daemon communication sockets). Allows interacting directly with background daemons (Docker, DBus).
* `/mnt` & `/media` — Static/dynamic mount locations. Critical during post-exploitation reconnaissance to discover unmounted secondary hard drives, external backup arrays, or NFS shares.
* `/root` vs `/home` — High-security superuser perimeter vs isolated user domains. The primary objective during privilege escalation is pivoting from `/home/[user]` to `/root`.

---

## 3. Quick Reference Matrix (Security Operations)

| Target Path | Primary Security Relevance | Critical Audit Command |
| :--- | :--- | :--- |
| `/etc/shadow` | Credential dumping / Offline cracking | `ls -la /etc/shadow` |
| `/etc/sudoers` | Sudo escalation vectors | `sudo -l` |
| `/dev/shm` | Memory-only stealth payload execution | `ls -la /dev/shm` |
| `/proc/[PID]/environ`| In-memory credential sniffing | `cat /proc/$$/environ \| tr '\0' '\n'` |
| `/var/log/auth.log` | Authentication brute force / audit tracking | `tail -f /var/log/auth.log` |
| `/tmp` | Staging world-writable scripts & backdoors | `find /tmp -type f -perm -002` |

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
