<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 01 — Linux Kernel Interface & Core CLI
   DOCUMENT: Day 07 — Linux Permissions, Special Bits & Access Control Internals
   ========================================================================= -->

# 🛡️ Day 07: Linux Permissions, Special Bits & Access Control Internals

> **IW Cyber Ops Research Vault | Module 01: Linux & Systems Foundations**  
> *Author: Muhammad Imran (@iwcyberops)*  
> *Track: Discretionary Access Control (DAC), Special Bits, POSIX ACLs & Exploitation*

---

## 1. Discretionary Access Control (DAC) & The UGO Model

Linux security relies on the **Discretionary Access Control (DAC)** model. Every file and directory on the system is owned by a specific **User** and **Group**, with access permissions divided into three distinct entities:

```
               ┌─────────────────────── Permissions String: -rwxr-xr-- ──────────────────────┐
               │                                                                             │
               ▼                     ▼                     ▼                                 ▼
         [ File Type ]       [ Owner / User ]       [ Group Owner ]                  [ Others / World ]
         - = Regular          r w x (4+2+1=7)        r - x (4+0+1=5)                  r - - (4+0+0=4)
         d = Directory         (Full Access)         (Read & Execute)                   (Read Only)
         l = Symlink
```

### The Binary Bitmask Architecture

Permissions are evaluated by the Linux kernel as a 3-bit binary triplet for each identity class:

| Permission | Symbol | Binary Value | Octal Weight | Kernel Action on Files | Kernel Action on Directories |
| :--- | :---: | :---: | :---: | :--- | :--- |
| **Read** | `r` | `100` | **4** | Open and read file content | List filenames inside the directory (`ls`) |
| **Write** | `w` | `010` | **2** | Modify/overwrite file content | Create, rename, or delete files inside directory |
| **Execute** | `x` | `001` | **1** | Run file as a compiled/script binary | Enter/traverse directory (`cd`) and access Inodes |
| **None** | `-` | `000` | **0** | No permissions granted | No access |

---

## 2. File vs Directory Permissions (Critical Nuance)

> [!NOTE]
> ### ⚠️ The Directory Deletion Paradox
> In Linux, **the permission to delete or rename a file resides in the parent directory, NOT the file itself.**  
> If an unprivileged user has write (`w`) and execute (`x`) permissions on a directory, that user can delete **any** file inside it (even a file owned by `root` with `chmod 000` permissions).

```
┌─────────────┬──────────────────────────────────────┬──────────────────────────────────────────┐
│ Permission  │ Impact on a File                     │ Impact on a Directory                    │
├─────────────┼──────────────────────────────────────┼──────────────────────────────────────────┤
│ **Read (r)**│ View contents (`cat`, `less`, `head`)│ View list of contained files (`ls`)      │
│ **Write(w)**│ Modify/truncate content (`>`, `nano`)│ Create, remove, or rename files within it│
│ **Exec (x)**│ Execute binary/script (`./script.sh`)│ Traverse/enter directory (`cd`, `path/`) │
└─────────────┴──────────────────────────────────────┴──────────────────────────────────────────┘
```

---

## 3. Core Permission Management Commands

### 1. `chmod` (Change File Mode Bits)
Modifies file access modes using **Octal (Numeric)** or **Symbolic** syntax.

#### Octal Syntax (Absolute Mode)
* `chmod 755 script.sh` — User: `rwx` (7), Group: `r-x` (5), Others: `r-x` (5).
* `chmod 600 id_rsa` — User: `rw-` (6), Group: `---` (0), Others: `---` (0) *(SSH Key standard)*.
* `chmod 644 /etc/passwd` — User: `rw-` (6), Group: `r--` (4), Others: `r--` (4).
* `chmod 777 /tmp/payload` — Full read, write, and execute for everyone *(Dangerous)*.

#### Symbolic Syntax (Relative Mode)
* `chmod u+x script.sh` — Add execute permission to the **User**.
* `chmod g-w file.txt` — Remove write permission from the **Group**.
* `chmod o=r file.txt` — Explicitly set **Others** to read-only.
* `chmod a+r file.txt` — Add read permission to **All** (User, Group, Others).
* `chmod -R 750 /opt/app/` — Recursively apply permissions across all nested directories and files.

---

### 2. `chown` & `chgrp` (Ownership Management)
* `chown user file.txt` — Change owner of the file to `user`.
* `chown user:group file.txt` — Change both user owner and group owner simultaneously.
* `chown -R www-data:www-data /var/www/html` — Recursively update web directory ownership.
* `chgrp devteam project/` — Change group ownership only.

---

### 3. `umask` (User File Creation Mode Mask)
The `umask` determines the **default permissions** subtracted from newly created files and directories.

* **Default Base Permissions:**
  * Files: `666` (`rw-rw-rw-`) — Files are never granted execute (`x`) by default for security.
  * Directories: `777` (`rwxrwxrwx`) — Directories require execute (`x`) for traversal.

$$\text{Final Permissions} = \text{Base Mode} - \text{umask}$$

* If `umask` is `022`:
  * New Directory: $777 - 022 = \mathbf{755}$ (`rwxr-xr-x`)
  * New File: $666 - 022 = \mathbf{644}$ (`rw-r--r--`)
* If `umask` is `077` (High Security / Isolated):
  * New Directory: $777 - 077 = \mathbf{700}$ (`rwx------`)
  * New File: $666 - 077 = \mathbf{600}$ (`rw-------`)

---

## 4. Special Permission Bits (The Crown Jewels)

Special permissions allow execution privilege escalation, group collaboration, and world-writable deletion protection.

```
                    ┌────── Special Bits Octal Position: 4 7 5 5 ──────┐
                    │                                                  │
                    ▼                                                  ▼
             [ Special Bit ]                                   [ Standard Bits ]
             4 = SUID (SetUID)                                   7 = User (rwx)
             2 = SGID (SetGID)                                   5 = Group (r-x)
             1 = Sticky Bit                                      5 = Others (r-x)
```

---

### 1. SUID (Set User ID — Octal `4000`)
* **Behavior:** When an executable with the SUID bit is executed, it runs with the privileges of the **File Owner** (usually `root`), rather than the user invoking it.
* **Notation:** Indicated by an `s` in the user execute position (`-rwsr-xr-x`).
* **Operational Purpose:** Allows unprivileged users to execute tasks requiring temporary root privileges (e.g., `passwd`, `sudo`, `ping`).

> 🔴 **Cyber Ops Exploitation Target (GTFOBins):**  
> If a binary capable of shell spawning or arbitrary file reading (e.g., `find`, `vim`, `bash`, `python`) is configured with the SUID bit, an unprivileged attacker can spawn an immediate root shell.

---

### 2. SGID (Set Group ID — Octal `2000`)
* **On Files:** The file executes with the permissions of the **Group Owner** (`-rwxr-sr-x`).
* **On Directories:** Any new file or sub-directory created inside will automatically **inherit the group ownership** of the parent directory rather than the primary group of the creating user.

---

### 3. Sticky Bit (Restricted Deletion — Octal `1000`)
* **Behavior:** Applied to world-writable directories (`rwxrwxrwt`). Prevents users from deleting, renaming, or modifying files owned by other users. **Only the file owner or root can delete the file.**
* **Notation:** Indicated by a `t` in the others execute position (`drwxrwxrwt`).
* **Standard Implementation:** `/tmp` and `/var/tmp`.

---

### 4. The Upper vs Lowercase Identifier (`s`/`S` and `t`/`T`)
When viewing `ls -l`, special permissions are displayed as either lowercase or uppercase letters:

```
-rwsr-xr-x  --> Lowercase 's' = SUID is Active AND underlying execute ('x') is ENABLED.
-rwSr-xr-x  --> Uppercase 'S' = SUID is Active BUT underlying execute ('x') is MISSING (Broken configuration).

drwxrwxrwt  --> Lowercase 't' = Sticky Bit is Active AND others execute ('x') is ENABLED.
drwxrwxrwT  --> Uppercase 'T' = Sticky Bit is Active BUT others execute ('x') is MISSING.
```

---

## 5. POSIX Access Control Lists (ACLs)

Standard UGO permissions only support one user and one group. **ACLs** allow granular permission assignments for arbitrary multiple users and groups on a single file.

```
# Indicated by a '+' at the end of permissions in ls -l:
-rw-rwxr--+ 1 user user 1024 Aug 24 10:00 confidential.pdf
```

### Core ACL Management Commands

* `getfacl filename` — Display detailed access control list metadata.
* `setfacl -m u:imran:rwx file.txt` — Grant user `imran` specific `rwx` permissions.
* `setfacl -m g:auditors:r-- file.txt` — Grant group `auditors` read-only access.
* `setfacl -x u:imran file.txt` — Remove specific ACL rule for user `imran`.
* `setfacl -b file.txt` — Strip **all** extended ACLs (reverts to standard UGO DAC).
* `setfacl -R -m u:imran:rw- /opt/project/` — Recursively apply ACL.

---

## 6. Linux File Attributes (`chattr` & `lsattr`)

File attributes operate below standard permissions and are enforced directly by the filesystem (ext4/xfs). Even `root` cannot bypass these without modifying the attribute first.

* `chattr +i file.txt` — **Immutable bit:** The file cannot be modified, deleted, renamed, overwritten, or symlinked (even by `root`).
* `chattr -i file.txt` — Remove immutable attribute.
* `chattr +a /var/log/audit.log` — **Append-Only bit:** Data can only be appended (`>>`); existing content cannot be truncated or modified (ideal for tamper-proof logging).
* `lsattr file.txt` — Display active filesystem attributes.

---

## 7. Offensive & Defensive Operations Matrix

| Objective | Command / Vector | Context |
| :--- | :--- | :--- |
| **Hunt SUID Binaries** | `find / -perm -4000 -type f 2>/dev/null` | Identify Privilege Escalation paths via GTFOBins |
| **Hunt SGID Binaries** | `find / -perm -2000 -type f 2>/dev/null` | Privilege escalation to specific group contexts |
| **Find World-Writable Directories** | `find / -type d -perm -0002 2>/dev/null` | Find drop-zones for staging reverse shells |
| **Find World-Writable Files** | `find / -type f -perm -0002 2>/dev/null` | Identify hijacked script/configuration targets |
| **Inspect Hidden ACL Backdoors**| `getfacl -R /etc/ 2>/dev/null \| grep "user:"` | Detect hidden user rights on root files |
| **Set Root Anti-Tamper Lock** | `chattr +i /etc/passwd /etc/shadow` | Prevent unauthorized user additions during attacks |

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
