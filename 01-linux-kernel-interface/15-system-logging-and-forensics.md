<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 01 — Linux Kernel Interface & Core CLI
   DOCUMENT: Day 15 — System Logging, Kernel Rings & Forensic Audit Trails
   ========================================================================= -->

# 🛡️ Day 15: System Logging, Kernel Rings & Forensic Audit Trails

> **IW Cyber Ops Research Vault | Module 01: Linux & Systems Foundations**  
> *Author: Muhammad Imran (@iwcyberops)*  
> *Track: Kernel Logging Subsystems, Journald, Artifact Forensics & Anti-Tamper*

---

## 1. Linux Logging Architecture: Syslog vs Systemd Journal

Modern Linux uses a dual-engine logging pipeline: **`systemd-journald`** captures low-level events directly from the kernel, early boot, and user-space daemons in structured binary format, which are then forwarded to traditional text-based **Syslog daemons (`rsyslog`)** for persistent disk storage.

```
  [ Linux Kernel ] ──────> /dev/kmsg ──┐
  [ User Daemons ] ──────> stdout/err ──┼──> [ systemd-journald ] ──> /run/log/journal/ (RAM)
  [ System Calls ] ──────> auditd ─────┘             │
                                                     ▼ (Forward)
                                              [ rsyslogd ]
                                                     │
                                                     ▼
                                              /var/log/* (Text Files)
```

### Syslog Event Severity Levels (RFC 5424)
Every log entry is categorized by a numerical **Facility** (source) and a **Severity Level** (0 to 7):

| Level | Code | Severity | Description |
| :---: | :---: | :--- | :--- |
| **`0`** | `emerg` | **Emergency** | System is completely unusable (Kernel panic) |
| **`1`** | `alert` | **Alert** | Immediate operational action required |
| **`2`** | `crit` | **Critical** | Critical hardware/software subsystem failure |
| **`3`** | `err` | **Error** | Non-fatal error conditions (Failed service start) |
| **`4`** | `warning`| **Warning** | Warning conditions (Resource approaching limits) |
| **`5`** | `notice` | **Notice** | Normal but significant condition |
| **`6`** | `info` | **Informational**| Normal operational messages (User login, IP assigned) |
| **`7`** | `debug` | **Debug** | Detailed diagnostic output for developers |

---

## 2. Kernel Ring Buffer & Diagnostics (`dmesg`)

The **Kernel Ring Buffer** is a fixed-size, cyclic memory buffer allocated in RAM during boot. It captures hardware initialization, device drivers, and kernel panic messages before filesystems are even mounted.

```bash
# 1. Inspect Kernel Messages with Human Timestamps
sudo dmesg -T                         # Converts uptime ticks to human-readable date/time

# 2. Filter by Message Severity Level
sudo dmesg -l err,crit,alert,emerg    # Display only error and critical kernel alerts

# 3. Dedicated Facility Filtering
sudo dmesg -k                         # Display only Kernel messages
sudo dmesg -u                         # Display only Userspace service messages

# 4. Real-Time Hardware & Driver Monitoring
sudo dmesg -w                         # Follow mode: Stream new hardware/USB/network events live

# 5. Clear Kernel Buffer (Root Operation)
sudo dmesg -C                         # Wipes the active ring buffer in RAM
```

> 🔴 **Security Relevance:**  
> `dmesg` reveals **Segmentation Faults (`segfault`)**, memory corruption errors, and buffer overflow crashes. When developing exploits or testing payloads, `dmesg` unmasks whether the target process crashed due to memory protection violations (**ASLR, Stack Canaries, DEP/NX**).

---

## 3. The Modern Systemd Journal: `journalctl`

`systemd-journald` writes indexed, tamper-resistant binary logs. The `journalctl` utility queries, filters, and formats this binary data.

### Practical `journalctl` Query Operations

```bash
# 1. Real-Time Daemon Monitoring
journalctl -u ssh.service -f          # Follow live SSH authentication events
journalctl -u nginx.service -n 50      # View the last 50 log lines of Nginx

# 2. Boot & Time-Window Filtering
journalctl -b                          # Logs from the CURRENT system boot only
journalctl -b -1                       # Logs from the PREVIOUS system boot
journalctl --since "2026-08-20 00:00:00" --until "2026-08-24 12:00:00"
journalctl --since "1 hour ago"        # Dynamic time-window query

# 3. Severity & Kernel Filtering
journalctl -p err..alert               # Display messages from priority 'err' to 'alert'
journalctl -k                          # Query kernel ring buffer directly through journald

# 4. Storage Management & Maintenance
journalctl --disk-usage                # Check physical disk space consumed by journal logs
sudo journalctl --vacuum-size=500M     # Purge old logs, retaining maximum 500 MB
sudo journalctl --vacuum-time=7d       # Delete journal logs older than 7 days
```

---

## 4. Traditional File Artifacts (`/var/log/`) & User Tracking

Unlike the binary journal, files under `/var/log/` are plain-text (except authentication databases) and can be parsed with standard CLI tools (`grep`, `awk`, `tail`).

```
/var/log/
├── auth.log (or secure)    --> Authentication, sudo invocations, SSH connections
├── syslog (or messages)    --> General system-wide operational activity
├── faillog                 --> Failed login attempts per user account
├── wtmp                    --> Binary history of all successful logins and reboots
├── btmp                    --> Binary history of all failed/bad login attempts
└── lastlog                 --> Binary record of each user's most recent login event
```

---

### Interactive User Session Tracking

```bash
# 1. Active Logged-in Sessions
who                                    # List all active logged-in users, TTYs, and login time
w                                      # Detailed report: Active users + what process they are running

# 2. Historical Login Auditing (Binary Databases)
last                                   # Reads /var/log/wtmp: Shows login history & reboots
last -a                                # Displays remote hostname/IP in the last column
last -n 10                             # Display only the 10 most recent logins
sudo lastb                             # Reads /var/log/btmp: Lists ALL bad/failed login attempts
lastlog                                # Reads /var/log/lastlog: Shows last login date of every account
```

---

## 5. Log Rotation Architecture (`logrotate`)

To prevent multi-gigabyte log files from exhausting disk space, the `logrotate` daemon runs daily via cron/systemd timers to compress, rename, and discard old logs.

* **Main Configuration:** `/etc/logrotate.conf`
* **Service Configurations:** `/etc/logrotate.d/` (e.g., `/etc/logrotate.d/nginx`, `/etc/logrotate.d/rsyslog`)

```text
# Example /etc/logrotate.d/authlog
/var/log/auth.log {
    weekly                             # Rotate file once per week
    rotate 4                           # Keep 4 historical backlogs before deletion
    compress                           # Compress old files with gzip (.gz)
    delaycompress                      # Delay compression until the next rotation cycle
    missingok                          # Do not throw error if file is missing
    notifempty                         # Do not rotate if the file is 0 bytes
    create 0640 root adm               # Re-create fresh empty file with permissions
    postrotate
        /usr/lib/rsyslog/rsyslog-rotate # Reload syslog daemon to point to new file
    endscript
}
```

---

## 6. Cyber Operations: Forensics vs Anti-Forensics

```
┌────────────────────────────────────────────────────────────────────────┐
│                     FORENSIC LOG THREAT MATRIX                         │
├──────────────────────────┬─────────────────────────────────────────────┤
│ 1. Brute-Force Detection │ Tracking mass 'Failed password' in auth.log │
│ 2. Sudo Abuse Tracking   │ Correlating COMMAND executions to real UID  │
│ 3. Selective Tampering   │ Scrubbing specific IP lines vs Total Wipe   │
│ 4. Anti-Tamper Locks     │ Enforcing +a append-only attributes on logs │
└──────────────────────────┴─────────────────────────────────────────────┘
```

---

### Blue Team Incident Response: Hunting in `auth.log`

```bash
# 1. Identify Attacker IPs Brute-Forcing SSH:
grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -nr | head -n 10

# 2. Track Unauthorized Sudo Command Invocations:
grep "sudo:" /var/log/auth.log | grep "COMMAND=" | awk -F: '{print $1, $4, $5}'

# 3. Detect Accepted External SSH Logins:
grep "Accepted publickey\|Accepted password" /var/log/auth.log
```

---

### Red Team Anti-Forensics & Evasion Analysis

1. **The Mistake of Total Log Deletion (`rm /var/log/auth.log`):**
   * Completely wiping a log file triggers an immediate High-Severity alert in SIEM/SOC systems. The missing Inode and broken file descriptor are obvious indicators of compromise (IoC).
2. **Selective Line Scrubbing (Evasion):**
   * Adversaries often use `sed` to delete only lines containing their specific source IP address:
     ```bash
     sed -i '/10\.10\.14\.5/d' /var/log/auth.log
     ```
3. **Defensive Countermeasure: Append-Only Immutable Bits (`chattr`):**
   * System administrators defend log integrity by setting the append-only attribute:
     ```bash
     sudo chattr +a /var/log/auth.log /var/log/syslog
     ```
   * *Result:* New logs can be appended via `>>`, but `sed -i`, `rm`, `vi`, and `>` truncation are blocked at the filesystem level.

---

## 7. Incident Response & Threat Hunting Reference Matrix

| Investigation Goal | Command Syntax | Evidence Source |
| :--- | :--- | :--- |
| **Inspect Failed SSH Logins** | `sudo lastb -n 20` | `/var/log/btmp` |
| **Inspect System Reboots** | `last reboot` | `/var/log/wtmp` |
| **Detect Active SSH Sessions** | `w -f` | In-Memory Kernel TTYs |
| **Follow Live Sudo Commands** | `journalctl _COMM=sudo -f` | `systemd-journald` |
| **Inspect Kernel Crashes** | `dmesg -T \| grep -Ei "segfault\|error\|panic"`| Kernel Ring Buffer |
| **Identify Active Log Locks** | `lsattr /var/log/auth.log` | Filesystem Inode Flags |

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
