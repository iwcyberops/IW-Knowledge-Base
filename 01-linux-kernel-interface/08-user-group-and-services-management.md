<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 01 — Linux Kernel Interface & Core CLI
   DOCUMENT: Day 08 — User & Group Administration, Sudoers & Systemd Services
   ========================================================================= -->

# 🛡️ Day 08: User & Group Administration, Sudoers & Systemd Services

> **IW Cyber Ops Research Vault | Module 01: Linux & Systems Foundations**  
> *Author: Muhammad Imran (@iwcyberops)*  
> *Track: Identity Access Management (IAM), Privilege Delegation & Service Daemons*

---

## 1. User Identity Architecture & Core Files

In the Linux kernel, users and groups do not exist as human names; they are handled entirely as numerical identifiers: **UID (User ID)** and **GID (Group ID)**.

```
┌────────────────────────────────────────────────────────────────────────┐
│                        LINUX UID CLASSIFICATION                        │
├─────────────────────┬──────────────────┬───────────────────────────────┤
│ UID 0               │ Superuser (Root) │ Absolute administrative rights│
│ UID 1 – 999         │ System Accounts  │ Daemons / Services (no login) │
│ UID 1000 – 60000+   │ Regular Users    │ Human interactive accounts    │
└─────────────────────┴──────────────────┴───────────────────────────────┘
```

---

### Key Authentication & Account Databases

#### 1. `/etc/passwd` (Account Metadata — World Readable)
Formatted as 7 colon-delimited fields:
```text
imran : x : 1001 : 1001 : Muhammad Imran,CyberOps : /home/imran : /bin/bash
  │     │    │      │              │                    │            └── 7. Login Shell
  │     │    │      │              │                    └─────────────── 6. Home Directory
  │     │    │      │              └──────────────────────────────────── 5. GECOS (Comments/Full Name)
  │     │    │      └─────────────────────────────────────────────────── 4. Primary GID
  │     │    └────────────────────────────────────────────────────────── 3. UID
  │     └─────────────────────────────────────────────────────────────── 2. Password Placeholder ('x' = stored in shadow)
  └───────────────────────────────────────────────────────────────────── 1. Username
```

#### 2. `/etc/shadow` (Cryptographic Hashes — Restricted `640` / `root:shadow`)
Contains encrypted password hashes and password expiration policies:
```text
imran : $6$rounds=5000$salt$hash... : 19700 : 0 : 90 : 7 :   :   :
  │                   │                │     │   │   │   │   │   └── 9. Reserved
  │                   │                │     │   │   │   │   └────── 8. Account Expiration Date
  │                   │                │     │   │   │   └────────── 7. Inactivity Days before lock
  │                   │                │     │   │   └────────────── 6. Warning Period (Days)
  │                   │                │     │   └────────────────── 5. Maximum Password Age (Days)
  │                   │                │     └────────────────────── 4. Minimum Password Age (Days)
  │                   │                └──────────────────────────── 3. Last Password Change (Days since Epoch)
  │                   └───────────────────────────────────────────── 2. Encrypted Hash Signature
  └────────────────────────────────────────────────────────────────── 1. Username
```

> 🔴 **Password Hash Signatures:**
> * `$1$` = MD5 *(Legacy / Deprecated)*
> * `$5$` = SHA-256
> * `$6$` = SHA-512 *(Standard Debian/Ubuntu)*
> * `$y$` = Yescrypt *(Modern Kali / Debian 12 standard)*

---

## 2. User Account Management Operations

### Low-Level `useradd` vs High-Level `adduser`
* `useradd` — Native low-level binary backend. Requires explicit flags (does not create home directory by default without `-m`).
* `adduser` — Interactive Perl wrapper around `useradd` (automates prompt, home creation, and shell assignment).

---

### Practical User Commands with Flags

```bash
# 1. Create a user with explicit Home, Bash shell, UID, and Comment
sudo useradd -m -d /home/operator -s /bin/bash -u 1337 -c "IW Cyber Operator" operator

# 2. Create a restricted System Daemon Account (No home, No interactive login)
sudo useradd -r -s /usr/sbin/nologin -M svc_proxy

# 3. Modify existing user account (usermod)
sudo usermod -s /bin/zsh operator              # Change default shell
sudo usermod -l newname oldname                # Rename user account
sudo usermod -L operator                       # Lock user account (prepends '!' in /etc/shadow)
sudo usermod -U operator                       # Unlock user account
sudo usermod -d /opt/operator -m operator      # Move home directory to new path

# 4. Password and Aging Policies (passwd & chage)
sudo passwd operator                           # Assign/Update password
sudo passwd -d operator                        # Delete password (Account becomes passwordless)
sudo chage -l operator                         # List password expiration details
sudo chage -M 60 -W 7 operator                 # Max 60 days validity with 7 days warning

# 5. Delete User Account
sudo userdel operator                          # Deletes user (Leaves home directory on disk)
sudo userdel -r operator                       # Recursive wipe: Deletes user AND home directory
```

---

## 3. Group Administration & Collaborative Permissions

Groups allow multiple users to share access to common resources without exposing files to all system users.

* **Primary Group:** Assigned at account creation (Listed in `/etc/passwd`). All newly created files inherit this GID.
* **Secondary / Supplementary Groups:** Additional group memberships (Listed in `/etc/group`).

---

### Group Commands & Collaborative Access Lab

```bash
# 1. Create and manage groups
sudo groupadd redteam
sudo groupadd blueteam
sudo groupdel oldteam

# 2. Append user to a secondary group (CRITICAL: Always use -a with -G)
sudo usermod -aG redteam operator              # Append 'operator' to 'redteam'
# WARNING: 'usermod -G redteam operator' WITHOUT -a will strip all other secondary groups!

# 3. Direct group member management via gpasswd
sudo gpasswd -a operator redteam               # Add user to group
sudo gpasswd -d operator redteam               # Remove user from group
```

---

### 🛠️ Practical Ops Lab: Creating a Secure Collaborative Directory

To configure a shared folder where all members of `redteam` can create and edit files, while preventing unauthorized access and enforcing group ownership inheritance:

```bash
# Step 1: Create shared directory
sudo mkdir /opt/redteam_vault

# Step 2: Change group ownership to redteam
sudo chown root:redteam /opt/redteam_vault

# Step 3: Apply Permissions (2770)
# 2 = SGID (New files inherit 'redteam' group)
# 7 = User/Root (rwx)
# 7 = Group/RedTeam (rwx)
# 0 = Others/World (--- No access)
sudo chmod 2770 /opt/redteam_vault

# Verify permissions
ls -ld /opt/redteam_vault
# Output: drwxrws--- 2 root redteam 4096 /opt/redteam_vault
```

---

## 4. Sudoers Architecture & Privilege Delegation

The `sudo` (Superuser Do) engine allows users to run commands with administrative privileges without sharing the root password.

* **Main Configuration File:** `/etc/sudoers` *(Always edit using `sudo visudo` to prevent syntax locking)*.
* **Drop-in Configuration Directory:** `/etc/sudoers.d/`

---

### Sudoers Syntax Anatomy

```text
operator   ALL = (ALL:ALL)   ALL
   │        │      │   │      └── Commands allowed (ALL or specific path: /bin/systemctl)
   │        │      │   └───────── Run as Group (Defaults to root)
   │        │      └───────────── Run as User (Defaults to root)
   │        └──────────────────── Target Hosts this rule applies to
   └───────────────────────────── Target User or %Group (%sudo, %admin)
```

```bash
# Sudoers Configuration Directives:

# Allow group 'redteam' full root rights with password
%redteam ALL=(ALL:ALL) ALL

# Grant specific binary execution without password prompt (High Threat Surface)
operator ALL=(ALL) NOPASSWD: /usr/bin/systemctl, /usr/bin/journalctl

# Audit user's sudo privileges
sudo -l
```

---

## 5. Systemd Daemon & Service Management

`systemd` is the init system and service manager (**PID 1**). It initializes the kernel userspace, manages background daemons, and handles target runlevels.

```
[ Linux Kernel ] ──> [ systemd (PID 1) ] ──┬──> [ sshd.service ]
                                           ├──> [ apache2.service ]
                                           └──> [ networking.target ]
```

---

### `systemctl` Core Lifecycle Commands

```bash
# Service Lifecycle Control
sudo systemctl start nginx.service             # Start daemon immediately
sudo systemctl stop nginx.service              # Terminate running daemon
sudo systemctl restart nginx.service           # Full stop and start
sudo systemctl reload nginx.service            # Reload configuration without dropping connections
sudo systemctl status nginx.service            # Inspect runtime status, PID, and recent logs

# Boot Persistence Control
sudo systemctl enable nginx.service            # Auto-start on system boot (creates symlink)
sudo systemctl disable nginx.service           # Disable auto-start on system boot
sudo systemctl is-active nginx.service         # Return active/inactive state
sudo systemctl is-enabled nginx.service        # Check if enabled on boot

# Emergency Masking (Prevents service from starting manually or by other services)
sudo systemctl mask apache2.service            # Symlinks unit to /dev/null
sudo systemctl unmask apache2.service          # Restores normal service operation
```

---

### ⚙️ Anatomy of a Custom Systemd Service (`.service` Unit)

Custom services are stored in `/etc/systemd/system/`.

```ini
# Path: /etc/systemd/system/iw_tunnel.service

[Unit]
Description=IW Cyber Ops Automated Reverse Tunnel
After=network.target

[Service]
Type=simple
User=operator
WorkingDirectory=/home/operator
ExecStart=/usr/bin/python3 /home/operator/agent.py
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

#### Activating New Service Units:
```bash
# 1. Reload systemd daemon to parse new unit files
sudo systemctl daemon-reload

# 2. Enable and start the new service
sudo systemctl enable --now iw_tunnel.service
```

---

## 6. Threat Operations & Auditing Matrix

| Tactical Objective | Command Syntax | Cyber Ops & Security Context |
| :--- | :--- | :--- |
| **Detect Rogue UID 0 Users** | `awk -F: '($3 == "0") {print $1}' /etc/passwd` | Unmasks backdoor root accounts. |
| **Audit Accounts without Password** | `sudo awk -F: '($2 == "") {print $1}' /etc/shadow` | Identifies unauthenticated login risks. |
| **Enumerate Sudo Vectors** | `sudo -l` | Identifies privilege escalation paths via GTFOBins. |
| **Detect Malicious Services** | `systemctl list-unit-files --type=service \| grep enabled` | Audits startup persistence hooks. |
| **Inspect Failed Service Logs** | `journalctl -u service_name.service -xe` | Low-level diagnostic troubleshooting. |

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
