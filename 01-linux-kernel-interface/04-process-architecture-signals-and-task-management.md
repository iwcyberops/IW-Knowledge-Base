<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 01 — Linux Kernel Interface & Core CLI
   DOCUMENT: Day 04 — Process Architecture, Signals & Task Management
   ========================================================================= -->

# 🛡️ Day 04: Process Architecture, Signals & Task Management

> **IW Cyber Ops Research Vault | Module 01: Linux & Systems Foundations**  
> *Author: Muhammad Imran (@iwcyberops)*  
> *Track: Linux Kernel Process Model, Signal Handling & Execution Control*

---

## 1. Process Lifecycle & State Machine

Every executable in execution is assigned a unique **Process ID (PID)** and originates from a **Parent Process ID (PPID)** via the `fork()` and `execve()` system calls. The root ancestor of all user-space processes is `systemd` / `init` (**PID 1**).

---
```
[ Fork ] [ Kernel Schedule ]
┌─────┐ ┌─────────┐
───>│ NEW │ ──(Ready Queue)──> │ RUNNING │ ──(Terminated)──> [ ZOMBIE / EXIT ]
└─────┘ └────┬────┘ ▲
│ │
Wait for I/O / │ Wakeup / Event │
Signal Event ▼ Triggered │
┌───────────┐ │
│ SLEEPING │ ───────────────────────┘
│ (S / D) │
└───────────┘
```
---

### Standard Process States (`STAT` Column)
* `R` (**Running / Runnable**): Actively executing on CPU or waiting in the run queue.
* `S` (**Interruptible Sleep**): Waiting for an event or I/O signal (e.g., keyboard input).
* `D` (**Uninterruptible Sleep**): Directly waiting on hardware/disk I/O (cannot be terminated by `kill -9`).
* `T` (**Stopped / Traced**): Suspended via signal (e.g., `Ctrl + Z` / `SIGSTOP`) or active debugger (GDB).
* `Z` (**Zombie / Defunct**): Process completed execution but parent has not read its exit status via `wait()`.
* `+` / `<` / `N`: Foreground process (`+`), High priority (`<`), Low priority (`N`).

---

## 2. Process Inspection & Monitoring Commands

### 1. `ps` (Process Snapshot)
Captures a static snapshot of current active processes.

* `ps aux` — BSD syntax listing all running processes across all users.
  * `a` = All users, `u` = Display user/owner format, `x` = Include processes without a controlling TTY (daemons).
* `ps -ef` — Standard POSIX syntax showing full command arguments and PPID hierarchy.
* `ps -u username` — List processes owned by a specific user.
* `ps -eo pid,ppid,user,%cpu,%mem,cmd --sort=-%cpu` — Custom output columns sorted by highest CPU usage.

---

### 2. `top` & `htop` (Real-Time Interactive Monitors)
* `top` — Native real-time dynamic view of system resource allocation and task states.
  * `k` — Kill a process by entering PID directly inside the interface.
  * `M` / `P` — Sort processes by Memory / CPU consumption.
  * `1` — Expand and view individual CPU core utilization.
* `htop` — Enhanced interactive visual manager with color coding, mouse support, and horizontal tree navigation.

---

### 3. `pstree` & `pgrep` (Hierarchy & Lookup)
* `pstree -p` — Display recursive tree of parent-child processes with their respective PIDs.
* `pstree -u` — Show username transitions along the process tree.
* `pgrep sshd` — Return only the PIDs matching the process name `sshd`.
* `pgrep -u root -l` — List PIDs and names of all processes executed by `root`.

---

## 3. POSIX Signals & Termination

Signals are asynchronous notifications sent by the kernel or user space to force a process to handle an event.

### Critical POSIX Signals

| Signal Number | Name | Action | Catchable? | Security / Operational Focus |
| :---: | :--- | :--- | :---: | :--- |
| `1` | `SIGHUP` | Hangup / Reload | Yes | Force daemons (Nginx, SSH) to reload config without restarting. |
| `2` | `SIGINT` | Interrupt (`Ctrl+C`) | Yes | Graceful terminal interruption sent from keyboard. |
| `9` | `SIGKILL` | Force Kill | **No** | Uncatchable kernel-level immediate termination. Used on unresponsive malware. |
| `15` | `SIGTERM` | Terminate (Default) | Yes | Requests clean shutdown (allows process to flush buffers & close sockets). |
| `18` | `SIGCONT` | Continue | Yes | Resume a previously paused/suspended process. |
| `19` | `SIGSTOP` | Force Stop (`Ctrl+Z`) | **No** | Immediate uncatchable process pause. |

---

### Process Termination Commands

* `kill [PID]` — Send `SIGTERM` (15) for graceful shutdown.
* `kill -9 [PID]` — Send `SIGKILL` (9) to force immediate termination.
* `kill -1 [PID]` — Send `SIGHUP` to reload configuration.
* `killall apache2` — Terminate all processes matching the binary name `apache2`.
* `pkill -f "python3 script.py"` — Terminate matching full command line string.
* `pkill -u [user]` — Kill all active processes owned by a specific target user.

---

## 4. Job Control & Process Scheduling

### Foreground vs Background Execution
* `command &` — Append `&` to run the command directly in the background detached from stdin.
* `Ctrl + Z` — Suspend active foreground process (sends `SIGTSTP`).
* `jobs -l` — List active background jobs and their state with PIDs.
* `bg %1` — Resume suspended job `#1` in the background.
* `fg %1` — Bring background job `#1` back to the active foreground.
* `nohup ./implant.sh &` — Prevent process from receiving `SIGHUP` when terminal session disconnects.

---

### Process Priority Management (`nice` & `renice`)
CPU scheduling priority is measured on a **Niceness scale** ranging from **-20 (Highest Priority)** to **19 (Lowest Priority)**. Default is `0`.

* `nice -n -10 ./critical_task` — Launch process with elevated priority (requires `sudo` for negative values).
* `nice -n 19 ./heavy_scan.sh` — Launch resource-heavy scan with lowest priority to avoid system lag.
* `renice -n -5 -p 1337` — Modify priority of an already running process with PID `1337`.

---

## 5. Security & Tactical Reference

| Objective | Command | Threat / Tactical Context |
| :--- | :--- | :--- |
| **Inspect Rogue Binary Path** | `ls -la /proc/[PID]/exe` | Unmasks binaries deleted from disk but running in RAM. |
| **Inspect Open Sockets** | `lsof -p [PID] -i` | Identify active command & control (C2) network connections. |
| **Kill Malicious Process Group** | `kill -9 -[PGID]` | Kills entire malicious process tree simultaneously. |
| **Persist Background Task** | `disown -h %1` | Detaches job from current shell session table. |

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
