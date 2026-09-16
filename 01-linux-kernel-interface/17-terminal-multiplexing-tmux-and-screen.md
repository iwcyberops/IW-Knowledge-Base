<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 01 — Linux Kernel Interface & Core CLI
   DOCUMENT: Day 17 — Terminal Multiplexing, Session Persistence & Tactical Ops
   ========================================================================= -->

# 🛡️ Day 17: Terminal Multiplexing, Session Persistence & Tactical Ops

> **IW Cyber Ops Research Vault | Module 01: Linux & Systems Foundations**  
> *Author: Muhammad Imran (@iwcyberops)*  
> *Track: Session Decoupling, Headless Process Persistence, Tmux & Screen Tradecraft*

---

## 1. Terminal Multiplexing Architecture & The SIGHUP Shield

When a standard SSH connection or local terminal window closes, the kernel's TTY driver broadcasts a **`SIGHUP` (Signal Hangup / 1)** to all child processes, terminating active scans, brute-force attacks, and scripts.

A **Terminal Multiplexer** decouples the running shell processes from the controlling terminal. It spawns a persistent background daemon (**Multiplexer Server**) that hosts the processes independently of the client's network connection.

```
 [ Local Machine ]                                [ Remote Target Server ]
┌─────────────────┐                              ┌──────────────────────────────────────────────┐
│ Terminal Window │ ──( SSH / Network Drops )──X │  [ Tmux / Screen Server Background Daemon ]  │
└─────────────────┘                              ├──────────────────────────────────────────────┤
                                                 │  ├── Session 1: nmap -p- (Still running!)    │
                                                 │  ├── Session 2: Hashcat brute-force          │
                                                 │  └── Session 3: Active Netcat C2 Listener    │
                                                 └──────────────────────────────────────────────┘
                                                                       ▲
                                                 [ Reconnect SSH ] ────┘ (Attach cleanly)
```

---

## 2. Tmux Hierarchy & Session Management

`tmux` (Terminal Multiplexer) organizes terminal workspaces into a strict 4-tier hierarchy:

$$\text{Server (Daemon)} \longrightarrow \mathbf{Sessions} \longrightarrow \mathbf{Windows} \text{ (Tabs)} \longrightarrow \mathbf{Panes} \text{ (Split Terminals)}$$

```
┌──────────────────────────────── Tmux Session: "redteam_ops" ────────────────────────────────┐
│ Window 1: [ recon ]                         │ Window 2: [ exploit ]                         │
│ ┌───────────────────────┬─────────────────┐ │ ┌───────────────────────────────────────────┐ │
│ │ Pane 1: nmap scan     │ Pane 2: ss -tulp│ │ │ Pane 1: Active Interactive Root Shell     │ │
│ ├───────────────────────┴─────────────────┤ │ └───────────────────────────────────────────┘ │
│ │ Pane 3: tail -f /var/log/auth.log       │ │                                               │
│ └─────────────────────────────────────────┘ │                                               │
└─────────────────────────────────────────────┴───────────────────────────────────────────────┘
```

---

### Core CLI Session Commands

```bash
# 1. Create Named Sessions
tmux new -s ops                       # Launch new session named 'ops'
tmux new -s recon -d                  # Create session in background (Detached mode)

# 2. Session Discovery & Attachment
tmux ls                               # List all active running tmux sessions
tmux attach -t ops                    # Attach to target session 'ops' (Short: tmux a -t ops)
tmux attach                           # Attach to the most recently active session

# 3. Session Termination
tmux kill-session -t ops              # Terminate target session and kill all child processes
tmux kill-server                      # Terminate all tmux sessions and stop the tmux daemon
```

---

## 3. Tmux Interactive Shortcuts Matrix

Inside a `tmux` session, all commands are triggered by pressing the **Prefix Key** (`Ctrl + B` by default), releasing it, and then pressing the operational hotkey.

### Pane Management (Splitting & Resizing)

| Action | Shortcut Sequence | Tactical Operational Purpose |
| :--- | :---: | :--- |
| **Split Vertically** | `Ctrl + B` then `%` | Split current pane into Left / Right view |
| **Split Horizontally** | `Ctrl + B` then `"` | Split current pane into Top / Bottom view |
| **Navigate Panes** | `Ctrl + B` then `Arrow Keys` | Switch active cursor focus between panes |
| **Toggle Pane Zoom** | `Ctrl + B` then `z` | Maximize current pane to full screen (toggle back with `z`) |
| **Kill Active Pane** | `Ctrl + B` then `x` | Close and kill running process in current pane |
| **Cycle Pane Layouts** | `Ctrl + B` then `Space` | Auto-rearrange layout (Even vertical, tiled, main-horizontal) |

---

### Window Management (Tabs) & Buffer Navigation

| Action | Shortcut Sequence | Tactical Operational Purpose |
| :--- | :---: | :--- |
| **Create New Window** | `Ctrl + B` then `c` | Opens a fresh, clean full-screen terminal tab |
| **Switch Window by Number**| `Ctrl + B` then `0-9` | Instant jump to specific window ID |
| **Next / Previous Window** | `Ctrl + B` then `n` / `p` | Cycle sequentially through open tabs |
| **Rename Window** | `Ctrl + B` then `,` | Assign functional name (e.g., `logs`, `c2`, `recon`) |
| **Interactive Window List**| `Ctrl + B` then `w` | Visual interactive tree menu of all sessions & windows |
| **Detach Session** | `Ctrl + B` then `d` | **Detach cleanly** (Leaves all tasks running in background) |
| **Scroll / Copy Mode** | `Ctrl + B` then `[` | Enter scrollback buffer (Navigate with `PgUp`/`PgDn` or Vi keys; hit `q` to exit) |

---

## 4. Advanced Tactical Operations & Keystroke Synchronization

### Keystroke Synchronization (`synchronize-panes`)
Allows typing a single command and mirroring it across **all open panes simultaneously**. Ideal for mass updates, multi-server SSH configurations, or clustered log monitoring.

```bash
# Inside Tmux Command Mode (Trigger via: Ctrl + B then :)
:setw synchronize-panes on            # Keystrokes now broadcast to EVERY pane in window
:setw synchronize-panes off           # Restore independent pane control
```

---

### Customization Profile: `~/.tmux.conf`
Configures persistent usability enhancements (Mouse support, Vi scroll bindings, fast reloading):

```tmux
# Configuration: ~/.tmux.conf

# Enable mouse mode (Click to switch panes, drag to resize, mouse-wheel to scroll)
set -g mouse on

# Enable Vi keybindings in copy/scroll mode
setw -g mode-keys vi

# Increase scrollback buffer history limit (Default is only 2000 lines)
set -g history-limit 50000

# Optimize status bar colors for tactical readability
set -g status-bg black
set -g status-fg green
```

* Apply changes immediately: `tmux source-file ~/.tmux.conf`

---

## 5. Lightweight Alternative: GNU `screen`

GNU `screen` is the legacy terminal multiplexer pre-installed by default across almost every minimal Linux server, rescue image, and embedded firmware where `tmux` may be missing.

* **Screen Prefix Key:** `Ctrl + A`

```bash
# 1. Screen Session Lifecycle
screen -S brute_force                 # Start new screen session named 'brute_force'
screen -ls                            # List active screen sessions
screen -r brute_force                 # Re-attach to detached screen session
screen -d -r brute_force              # Force detach remote instance and attach locally

# 2. Interactive Screen Hotkeys
# Ctrl + A then d                      # Detach cleanly from screen
# Ctrl + A then c                      # Create new screen window
# Ctrl + A then n / p                  # Next / Previous window
# Ctrl + A then k                      # Kill active screen window
# Ctrl + A then [                      # Enter scrollback mode (Hit 'Esc' to exit)
```

---

## 6. Cyber Operations & Red Team Tradecraft

```
┌────────────────────────────────────────────────────────────────────────┐
│                   TACTICAL MULTIPLEXER USE CASES                       │
├──────────────────────────┬─────────────────────────────────────────────┤
│ 1. Scan / Attack Survival│ Prevents dropped SSH pipes from killing jobs│
│ 2. Operator HUD Matrix   │ Unified 4-way split monitoring cockpit      │
│ 3. Multi-Operator Socket │ Two operators sharing a live terminal session│
└──────────────────────────┴─────────────────────────────────────────────┘
```

---

### Vector 1: Headless Long-Running Scan Execution
When initiating an intensive 12-hour network scan or password audit over an unstable remote connection:

```bash
# Spawn detached session executing heavy task directly:
tmux new -s scan_job -d 'nmap -sS -p- -T4 -oA full_network_scan 10.10.10.0/24'

# Check progress at any time:
tmux attach -t scan_job
```

---

### Vector 2: Shared Multi-Operator Console (Socket Sharing)
Multiple red team operators can connect to the exact same interactive terminal session in real time via shared UNIX domain sockets:

```bash
# Operator 1: Create session with shared socket
tmux -S /tmp/shared_socket new -s redteam_session
chmod 777 /tmp/shared_socket

# Operator 2: Connect directly to Operator 1's active view
tmux -S /tmp/shared_socket attach -t redteam_session
# Result: Both operators see identical real-time keystrokes and output simultaneously.
```

---

## 7. Tactical Operations Reference Matrix

| Operational Goal | Tmux Command / Sequence | Context |
| :--- | :--- | :--- |
| **Launch Persistent Task** | `tmux new -s <name> -d '<cmd>'` | Background headless job execution |
| **List All Active Sessions** | `tmux ls` *(or `screen -ls`)* | Audit running multiplexer daemons |
| **Emergency Detach** | `Ctrl + B` then `d` | Safe session disconnect |
| **Search Scrollback Buffer** | `Ctrl + B` then `[` then `?keyword` | Search historical terminal output |
| **Fullscreen Active Pane** | `Ctrl + B` then `z` | Maximize/minimize pane on demand |
| **Kill Broken Session** | `tmux kill-session -t <name>` | Wipe hung processes cleanly |

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
