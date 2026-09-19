<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 01 — Linux Kernel Interface & Core CLI
   DOCUMENT: Day 26 — Signal Handling, Traps & Anti-Interruption Hardening
   ========================================================================= -->

# 🛡️ Day 26: Signal Handling, Traps & Anti-Interruption Hardening

> **IW Cyber Ops Research Vault | Module 01: Linux & Systems Foundations**  
> *Author: Muhammad Imran (@iwcyberops)*  
> *Track: Asynchronous POSIX Signals, Kernel Trap Handlers & Anti-Forensic Cleanup*

---

## 1. Asynchronous Signal Interception Architecture

In Linux, a **Signal** is an asynchronous hardware or software interrupt sent by the kernel to force a process to execute an event handler. 

By default, pressing `Ctrl + C` sends `SIGINT`, terminating the running script immediately and leaving temporary files, open network sockets, or corrupted staging folders behind on the disk.

The **`trap`** built-in allows scripts to register custom callback routines to intercept signals before the process terminates.

```
                           [ Script Running: PID 1337 ]
                                        │
                                        ▼
                  [ User Presses Ctrl+C ──> Kernel sends SIGINT ]
                                        │
                                        ▼
                         { Is a TRAP Handler Registered? }
                                        │
               ┌────────────────────────┴────────────────────────┐
               ▼ (NO)                                            ▼ (YES)
    [ Default Action: Kill ]                        [ Divert to Custom Routine ]
    Leaves orphan temp files                         - Wipes /tmp artifacts
    Exits with code 130                              - Closes network sockets
                                                     - Exits cleanly (Exit 0 or 1)
```

---

## 2. Core Interceptable Signals vs Pseudo-Signals

### Standard POSIX Signals

| Signal Name | Number | Keyboard Trigger | Default Action | Trappable? |
| :--- | :---: | :---: | :--- | :---: |
| **`SIGHUP`** | `1` | SSH Disconnect | Terminate process | ✅ **Yes** |
| **`SIGINT`** | `2` | `Ctrl + C` | Interrupt process | ✅ **Yes** |
| **`SIGQUIT`**| `3` | `Ctrl + \` | Quit & dump core | ✅ **Yes** |
| **`SIGTERM`**| `15`| `kill <PID>` | Graceful request to terminate | ✅ **Yes** |
| **`SIGTSTP`**| `20`| `Ctrl + Z` | Suspend process | ✅ **Yes** |
| **`SIGKILL`**| `9` | `kill -9` | Immediate kernel termination | ❌ **NO (Uncatchable)** |
| **`SIGSTOP`**| `19`| `kill -19`| Immediate kernel pause | ❌ **NO (Uncatchable)** |

---

### Bash Special Pseudo-Signals

Bash introduces synthetic signals that do not exist at the kernel level but trigger on specific internal execution states:

| Pseudo-Signal | Trigger Condition | Primary Operational Use Case |
| :--- | :--- | :--- |
| **`EXIT`** (or `0`) | Triggers on **ANY script termination** (Normal finish, `exit` call, or caught signal). | **The Ultimate Cleanup Pattern:** Guarantees temp file deletion regardless of how the script ends. |
| **`ERR`** | Triggers whenever a command returns a **Non-Zero Exit Code**. | Custom error logging and automatic stack-tracing. |
| **`DEBUG`** | Triggers **BEFORE every single command** is executed. | Building custom runtime debuggers or live telemetry tracers. |
| **`RETURN`**| Triggers when a function finishes or a sourced script completes. | Post-function execution audit hooks. |

---

## 3. The `trap` Command Syntax & Modifiers

$$\text{Syntax Pattern: } \mathbf{\text{trap 'COMMAND\_OR\_FUNCTION' SIGNAL\_LIST}}$$

```bash
# 1. Register a trap for multiple signals
trap 'cleanup_routine' SIGINT SIGTERM SIGHUP EXIT

# 2. View all currently active traps in the running session
trap -p

# 3. Reset a specific signal trap back to default kernel behavior
trap - SIGINT

# 4. Ignore a signal completely (Immunity Mode - Empty string '')
trap '' SIGINT SIGTSTP               # Script is now completely IMMUNE to Ctrl+C and Ctrl+Z!
```

---

## 4. The Gold Standard: Automated Cleanup Handlers

The professional security standard is to bind an automated cleanup handler to the **`EXIT`** pseudo-signal. 

Because `EXIT` triggers on normal execution finish, syntax errors, and caught fatal signals, registering it once ensures zero leftover forensic traces on disk:

```bash
#!/usr/bin/env bash

# =========================================================================
# Step 1: Create a secure temporary workspace
# =========================================================================
TEMP_DIR=$(mktemp -d /tmp/iw_recon_XXXXXX)

# =========================================================================
# Step 2: Define the Cleanup Handler
# =========================================================================
cleanup() {
    local exit_code=$?
    echo -e "\n\033[1;33m[*] Cleaning up temporary artifacts in ${TEMP_DIR}...\033[0m"
    
    # Securely wipe temporary directory
    if [[ -d "$TEMP_DIR" ]]; then
        rm -rf "$TEMP_DIR"
    fi
    
    echo -e "\033[1;32m[+] Cleanup complete. Terminating with code: ${exit_code}\033[0m"
    exit "$exit_code"
}

# =========================================================================
# Step 3: Bind Handler to EXIT and common signals
# =========================================================================
trap 'cleanup' EXIT SIGINT SIGTERM SIGHUP

# =========================================================================
# Script Payload / Main Logic
# =========================================================================
echo "[+] Active workspace: $TEMP_DIR"
echo "Staged sensitive telemetry data" > "${TEMP_DIR}/payload.txt"

echo "[*] Simulating intensive operation (Press Ctrl+C to test trap)..."
sleep 10

echo "[+] Operation completed successfully."
```

---

## 5. Advanced Error Trapping: Line Numbers & Call Stacks (`ERR`)

Using `trap ... ERR`, operators can intercept unexpected command failures and print the exact file, function name, and line number where the failure occurred:

```bash
#!/usr/bin/env bash

# Error Trapper Function
error_handler() {
    local failed_line="$1"
    local failed_command="$2"
    local exit_code="$3"
    
    echo -e "\n\033[1;31m[-] CRITICAL ERROR DETECTED:\033[0m" >&2
    echo "  - File:       ${BASH_SOURCE[0]}" >&2
    echo "  - Line:       ${failed_line}" >&2
    echo "  - Command:    '${failed_command}'" >&2
    echo "  - Exit Code:  ${exit_code}" >&2
    exit "$exit_code"
}

# Trap ERR and pass line number, failing command string, and exit status
trap 'error_handler "$LINENO" "$BASH_COMMAND" "$?"' ERR

# Operational Code
echo "[*] Initializing tasks..."
valid_operation=$(echo "Valid command")

# Deliberate intentional failure:
cat /root/non_existent_restricted_file.txt

echo "[+] This line will never execute because ERR trap aborts."
```

---

## 6. Cyber Operations & Threat Hardening

```
┌────────────────────────────────────────────────────────────────────────┐
│                    TACTICAL TRAP THREAT VECTORS                        │
├──────────────────────────┬─────────────────────────────────────────────┤
│ 1. Anti-Interruption     │ Ignores Ctrl+C during sensitive memory      │
│                          │ writes or critical payload deployments      │
│ 2. Anti-Forensic Shred   │ Overwrites dumped credentials using shred   │
│                          │ the millisecond the process is terminated   │
│ 3. Telemetry Callback    │ Fires a webhook alert if script is killed   │
└──────────────────────────┴─────────────────────────────────────────────┘
```

---

### 🛡️ Tactical Lab: Secure In-Memory Shredder on Interruption

```bash
#!/usr/bin/env bash

# Generate memory-backed sensitive staging key
LOOT_FILE="/dev/shm/dumped_creds.txt"
echo "ADMIN_PASSWORD_SUPER_SECRET_1337" > "$LOOT_FILE"

# Anti-Forensic Secure Wiper
emergency_shred() {
    echo -e "\n[!] Emergency Signal Received: Shredding evidence..."
    if [[ -f "$LOOT_FILE" ]]; then
        shred -u -z -n 3 "$LOOT_FILE" 2>/dev/null # 3-pass overwrite + zero fill + delete
    fi
    exit 1
}

# Intercept all possible termination signals
trap 'emergency_shred' SIGINT SIGTERM SIGHUP SIGQUIT

echo "[*] Staged credentials in RAM: $LOOT_FILE"
echo "[*] Running long data exfiltration loop..."
sleep 20
```

---

## 7. Signal Operations Reference Matrix

| Goal | Syntax Pattern | Mechanism |
| :--- | :--- | :--- |
| **Catch All Exits** | `trap 'cleanup' EXIT` | Fires on normal finish, errors, and caught kills |
| **Catch Keyboard Break**| `trap 'abort' SIGINT` | Intercepts `Ctrl + C` |
| **Ignore Signal** | `trap '' SIGINT SIGTERM` | Makes script immune to interruption |
| **Reset to Default** | `trap - SIGINT` | Restores default OS signal handling |
| **Trace Failing Lines** | `trap 'log_err $LINENO' ERR` | Captures runtime failure coordinates |
| **List Active Traps** | `trap -p` | Prints registered handler mappings |

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
