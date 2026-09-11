<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 01 — Linux Kernel Interface & Core CLI
   DOCUMENT: Day 11 — Linux Storage Internals, Inodes & Filesystem Mounting
   ========================================================================= -->

# 🛡️ Day 11: Linux Storage Internals, Inodes & Filesystem Mounting

> **IW Cyber Ops Research Vault | Module 01: Linux & Systems Foundations**  
> *Author: Muhammad Imran (@iwcyberops)*  
> *Track: Virtual Filesystem (VFS), Inode Structures, Mount Hardening & Raw Disk Carving*

---

## 1. Linux Storage Subsystem Architecture

The Linux kernel interacts with physical storage through the **Virtual Filesystem (VFS)** abstraction layer. This layer provides a unified system call interface (`open`, `read`, `write`) regardless of the underlying hardware or filesystem format (ext4, XFS, Btrfs, FAT32).

```
                     ┌────────────────────────────────────────────────────────┐
                     │          USERSPACE APPLICATIONS / COMMANDS             │
                     └──────────────────────────┬─────────────────────────────┘
                                                │  (System Calls: read / write / open)
                                                ▼
                     ┌────────────────────────────────────────────────────────┐
                     │            VIRTUAL FILESYSTEM SWITCH (VFS)             │
                     └──────────────────────────┬─────────────────────────────┘
                                                │
                 ┌──────────────────────────────┼──────────────────────────────┐
                 ▼                              ▼                              ▼
          [ ext4 Driver ]                [ XFS Driver ]                [ tmpfs Driver ]
                 │                              │                              │
                 ▼                              ▼                              ▼
        ┌──────────────────┐           ┌──────────────────┐           ┌──────────────────┐
        │ Block Device     │           │ Block Device     │           │ Dynamic RAM      │
        │ (/dev/nvme0n1p1) │           │ (/dev/sda1)      │           │ (/dev/shm)       │
        └──────────────────┘           └──────────────────┘           └──────────────────┘
```

---

## 2. Block Device Discovery & Partitioning (`lsblk`, `blkid`, `fdisk`)

### 1. `lsblk` (List Block Devices)
Queries the `sysfs` filesystem to visualize block storage hierarchies, partition schemes, and mount associations.

* `lsblk` — Tree view of all detected storage hardware and partitions.
* `lsblk -f` — Output filesystems, UUIDs, and active mount points.
* `lsblk -m` — Display device permissions, UID/GID ownership, and block sizes.
* `lsblk -p` — Print full absolute device paths (e.g., `/dev/sda1` instead of `sda1`).

---

### 2. `blkid` (Block Device Attribute Locator)
Locates block devices and reads their low-level metadata attributes directly from disk headers.

* `sudo blkid` — Print UUIDs, PARTUUIDs, and Filesystem Types (`TYPE="ext4"`, `TYPE="vfat"`) of all partitions.
* `sudo blkid /dev/sdb1` — Inspect a specific target partition.

---

### 3. `fdisk` & `gdisk` (Partition Table Manipulation)
Low-level sector-by-sector partition editors. `fdisk` supports legacy **MBR** (Master Boot Record) and modern **GPT** (GUID Partition Table); `gdisk` is dedicated exclusively to GPT.

* `sudo fdisk -l` — List all physical disks, partition tables, sector sizes, and disk identifiers.
* `sudo fdisk -l /dev/sda` — Inspect sector layout and partition boundaries of `/dev/sda`.
* `sudo fdisk /dev/sdb` — Enter interactive partition editor mode:
  * `p` — Print partition table.
  * `n` — Create new partition.
  * `d` — Delete partition.
  * `t` — Toggle partition type (e.g., Linux LVM, Linux Swap).
  * `w` — Write changes to disk and exit (Commits partition table to disk).
  * `q` — Quit without saving changes.

---

## 3. Storage Diagnostics & Usage Profiling (`df`, `du`)

### 1. `df` (Disk Free — Filesystem Capacity)
Queries the superblock of mounted filesystems to report total, used, and available space.

* `df -h` — Human-readable capacity report (KB, MB, GB).
* `df -T` — Display underlying filesystem type column (`ext4`, `xfs`, `overlay`).
* `df -h /home` — Show capacity of the specific partition hosting `/home`.
* `df -i` — **Inode capacity report** (Shows percentage of used Inodes rather than disk bytes).

---

### 2. `du` (Disk Usage — Directory Space Consumption)
Recursively crawls directory trees to compute actual disk block consumption of individual files.

* `du -sh /var/log` — Summary total size of `/var/log` in human-readable format.
* `du -ah /opt` — List sizes of all individual files and folders inside `/opt`.
* `du -h --max-depth=1 /var` — Display top-level directory footprints inside `/var` without infinite recursion.
* `du -ah --threshold=500M /` — Filter and display only files/directories consuming greater than 500 MB.

---

### ⚠️ The Ghost File Anomaly (`df` vs `du` Discrepancy)

```
[ Problem Scenario ]
`df -h` shows partition is 100% FULL.
`du -sh /` shows only 20 GB used out of 100 GB.
```

* **Root Cause:** A process holds an **open file descriptor** to a file that was deleted from disk via `rm`. The filesystem cannot free the disk blocks until the running process closes the file descriptor or the process is terminated.
* **Triage & Remediation:**
```bash
# Locate unlinked deleted files held open in RAM
lsof +L1 2>/dev/null | grep deleted

# Identify process PID and truncate/kill it
kill -9 <PID>
```

---

## 4. Inode Architecture & Hard vs Soft Links

An **Inode (Index Node)** is a fundamental data structure on Unix filesystems that stores all metadata about a file **except its filename and actual data contents**.

```
                           ┌───────────────────────────────┐
                           │        INODE STRUCTURE        │
                           ├───────────────────────────────┤
                           │ - Inode Number (Unique ID)    │
                           │ - File Type & Permissions     │
                           │ - UID & GID Ownership         │
                           │ - File Size (Bytes)           │
                           │ - MACB Timestamps             │
                           │ - Hard Link Counter           │
                           │ - Direct & Indirect Pointers  │
                           └──────────────┬────────────────┘
                                          │
                                          ▼  (Pointers)
                           ┌───────────────────────────────┐
                           │    RAW DATA STORAGE BLOCKS    │
                           │   [Block 1] [Block 2] [Block 3│
                           └───────────────────────────────┘
```

> [!NOTE]
> Filenames are stored exclusively inside **Directory Entries (dirent)**, which map human-readable strings to their respective Inode numbers.

---

### Hard Links vs Symbolic (Soft) Links

```
     [ HARD LINK ]                                      [ SYMBOLIC / SOFT LINK ]

   Filename A      Filename B                         Symlink Pointer      Original File
   ┌────────┐      ┌────────┐                           ┌─────────┐         ┌───────────┐
   │ file.c │      │ hard.c │                           │ link.sh │ ──────> │ target.sh │
   └───┬────┘      └───┬────┘                           └─────────┘         └─────┬─────┘
       │               │                                                          │
       └───────┬───────┘                                                          │
               ▼                                                                  ▼
        ┌─────────────┐                                                    ┌─────────────┐
        │ Inode: 1042 │                                                    │ Inode: 2088 │
        └──────┬──────┘                                                    └──────┬──────┘
               ▼                                                                  ▼
        ┌─────────────┐                                                    ┌─────────────┐
        │ Data Blocks │                                                    │ Data Blocks │
        └─────────────┘                                                    └─────────────┘
```

| Metric | Hard Link (`ln file link`) | Soft / Symbolic Link (`ln -s file link`) |
| :--- | :--- | :--- |
| **Inode Number** | Shares the **identical Inode** as target | Allocated a **new, unique Inode** |
| **Cross-Filesystem** | ❌ Cannot cross partition boundaries | ✅ Works across different partitions / drives |
| **Directory Linking**| ❌ Blocked by kernel (prevents loops) | ✅ Can link directories |
| **Target Deleted** | ✅ Data persists (Link counter decrements) | ❌ Broken / Dangling pointer (`dangling symlink`) |

```bash
# Practical Link Management:
ln source.txt hardlink.txt             # Create Hard Link (Shares same Inode)
ln -s /var/log/auth.log symlink.log    # Create Symbolic Link (Points to path)
ls -i source.txt hardlink.txt          # Inspect and verify identical Inode numbers
```

---

### Inode Exhaustion Attack (Denial-of-Service)
A filesystem can run out of storage space even if gigabytes of physical disk space remain, if all available Inodes are consumed by millions of 0-byte files.

```bash
# Verify Inode saturation:
df -i

# Hunt directory with highest Inode density:
find / -xdev -printf '%h\n' 2>/dev/null | sort | uniq -c | sort -k1 -n | tail -n 10
```

---

## 5. Filesystem Mounting & `/etc/fstab` Hardening

Mounting attaches a storage device's filesystem to a specific directory in the global Linux tree hierarchy (`/`).

### 1. Manual Mount & Unmount Operations
```bash
# 1. Mount block device to directory
sudo mount /dev/sdb1 /mnt/usb/

# 2. Mount with restrictive security options
sudo mount -o ro,noexec,nosuid /dev/sdb1 /mnt/secure_drive/

# 3. Mount an ISO disk image directly (Loop Device)
sudo mount -o loop kali-linux.iso /mnt/iso/

# 4. Unmount operations
sudo umount /mnt/usb/                  # Standard unmount
sudo umount -l /mnt/usb/               # Lazy unmount (Detaches immediately, cleans references later)
sudo umount -f /mnt/usb/               # Force unmount (Used on unresponsive network NFS shares)
```

---

### 2. Automated Filesystem Mounting: `/etc/fstab`

`/etc/fstab` controls persistent mounts parsed during boot. It is configured across 6 standardized columns:

```text
UUID=a1b2c3d4-01   /var/log   ext4   defaults,noexec,nosuid   0   2
       │              │        │               │              │   └── 6. Fsck Order (0=Skip, 1=Root /, 2=Other)
       │              │        │               │              └────── 5. Dump Backup (0=Disable, 1=Enable)
       │              │        │               └───────────────────── 4. Mount Options
       │              │        └───────────────────────────────────── 3. Filesystem Type
       │              └────────────────────────────────────────────── 2. Mount Point Target
       └───────────────────────────────────────────────────────────── 1. Device Identifier (UUID / Device node)
```

---

### 🛡️ System Hardening: Secure Mount Flags

Applying restrictive mount options to world-writable and temporary directories neutralizes many privilege escalation and malware staging techniques:

| Mount Flag | Security / Operational Function | Target Directory |
| :--- | :--- | :--- |
| `noexec` | Blocks binary and script execution directly from the partition | `/tmp`, `/var/tmp`, `/dev/shm` |
| `nosuid` | Disables SUID and SGID execution bits on the partition | `/home`, `/tmp`, `/mnt` |
| `nodev`  | Prevents the creation/interpretation of character/block device files | `/tmp`, `/var` |
| `ro`     | Mounts the filesystem in strictly **Read-Only** mode | `/boot`, forensic drives |

```bash
# Example Secure /etc/fstab entry for /tmp:
tmpfs   /tmp   tmpfs   defaults,noexec,nosuid,nodev   0   0
```

---

## 6. Forensics, Data Recovery & Threat Vectors (Operator's Lens)

### 1. Bit-by-Bit Raw Disk Imaging (`dd`)
Used in incident response to preserve bit-exact forensic disk copies before evidence contamination:

```bash
# Clone entire drive /dev/sda into raw forensic image with SHA-256 validation
sudo dd if=/dev/sda of=/mnt/evidence/disk_image.raw bs=64K status=progress conv=noerror,sync
```

---

### 2. In-Memory Deleted File Recovery via `/proc`
If an attacker or malware deletes an active payload from disk to evade static AV scans, the entire binary remains recoverable from kernel memory:

```bash
# Step 1: Identify PID holding the deleted file
lsof 2>/dev/null | grep -i "deleted"

# Step 2: Carve raw binary directly back to disk from file descriptor
cp /proc/<PID>/fd/<FD_NUM> /tmp/recovered_payload.elf
chmod +x /tmp/recovered_payload.elf
```

---

### 3. Bypassing `noexec` Mount Restrictions
If a target system mounts `/tmp` or `/dev/shm` with `noexec`, executing `./payload` yields `Permission Denied`. Operators bypass this by passing the payload directly as an argument to the dynamic linker or interpreter:

```bash
# Method A: Dynamic ELF Linker / Loader Bypass
/lib64/ld-linux-x86-64.so.2 /tmp/payload.elf

# Method B: Direct Interpreter Invocation
python3 /tmp/script.py
bash /tmp/shell.sh
```

---

## 7. Storage & Filesystem Operations Matrix

| Objective | Command Syntax | Tactical Focus |
| :--- | :--- | :--- |
| **Inspect Partition UUIDs** | `lsblk -f` | Identify drives and partition types cleanly |
| **Audit Inode Saturation** | `df -i` | Detect Inode exhaustion DoS conditions |
| **Unmask Open Deleted Files**| `lsof +L1` | Find hidden running processes with unlinked files |
| **Audit Active Mount Options**| `mount \| column -t` | Identify missing `noexec` / `nosuid` flags |
| **Trace File Inode Number** | `ls -i <file>` | Validate Hard Link mappings |
| **Reload `/etc/fstab` Mounts**| `sudo mount -a` | Test fstab syntax errors without rebooting |

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
