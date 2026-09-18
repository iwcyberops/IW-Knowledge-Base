<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 01 — Linux Kernel Interface & Core CLI
   DOCUMENT: Day 24 — Modular Functions, Scoping & Return Channels
   ========================================================================= -->

# 🛡️ Day 24: Modular Functions, Scoping & Return Channels

> **IW Cyber Ops Research Vault | Module 01: Linux & Systems Foundations**  
> *Author: Muhammad Imran (@iwcyberops)*  
> *Track: Modular Shell Engineering, Dynamic Scoping, Namerefs & Stack Frames*

---

## 1. Function Architecture in Linux Shells

In Bash, a **Function** is a named compound command stored directly in the active shell's memory table. When invoked, it executes inside the **current shell process** without spawning a new child PID (unless explicitly wrapped in a subshell).

```
                      [ SCRIPT EXECUTION ENTRY ]
                                  │
                                  ▼
                     [ Function Call: probe_host "10.10.10.1" ]
                                  │
                                  ▼
                     ┌──────────────────────────┐
                     │    FUNCTION STACK FRAME  │
                     │  - $1 = "10.10.10.1"     │ ──( Isolated Positional Args )
                     │  - local socket_fd       │ ──( Memory Isolated )
                     └────────────┬─────────────┘
                                  │
                                  ▼
                     [ Returns Exit Code: 0-255 ] ──> [ Evaluated by Caller ]
```

---

### Declaration Syntax Patterns

```bash
# Pattern 1: POSIX Standard (Recommended for Portability)
probe_port() {
    local target="$1"
    local port="$2"
    nc -zvw1 "$target" "$port" &>/dev/null
}

# Pattern 2: Bash Keyword Syntax
function audit_system {
    echo "[*] Auditing active users..."
}
```

> 💡 **Parser Context:** Functions **must be defined before they are called** in the script. Bash interprets files sequentially from top to bottom.

---

## 2. Argument Handling & Positional Parameter Shadowing

Inside a function, positional parameters (`$1`, `$2`, `$@`, `$#`) are **shadowed**: they refer exclusively to the arguments passed to that specific function, **not** the arguments passed to the root script.

```bash
#!/usr/bin/env bash
# Script invoked as: ./recon.sh "GLOBAL_ARG"

my_function() {
    echo "Function Argument 1: $1"    # Refers to argument passed to function: "LOCAL_ARG"
    echo "Function Argument Count: $#" # Output: 1
}

echo "Root Script Arg 1: $1"          # Output: "GLOBAL_ARG"
my_function "LOCAL_ARG"
```

---

## 3. Variable Scoping: Global Leakage vs `local` Isolation

By default, **all variables declared inside a function in Bash are GLOBAL**. If a function assigns a variable without the `local` keyword, it overwrites the variable in the main script memory!

```
 ┌────────────────────────────────────────────────────────────────────────┐
 │ ❌ THE GLOBAL LEAKAGE BUG (Without 'local')                            │
 ├────────────────────────────────────────────────────────────────────────┤
 │ TARGET="10.0.0.1"                                                      │
 │ exploit() { TARGET="192.168.1.50"; } # Overwrites GLOBAL $TARGET!     │
 │ exploit                                                                │
 │ echo "$TARGET"  # OUTPUT: "192.168.1.50" (Global state corrupted!)    │
 └────────────────────────────────────────────────────────────────────────┘

 ┌────────────────────────────────────────────────────────────────────────┐
 │ ✅ SECURE ISOLATION (With 'local')                                     │
 ├────────────────────────────────────────────────────────────────────────┤
 │ TARGET="10.0.0.1"                                                      │
 │ exploit() { local TARGET="192.168.1.50"; }                             │
 │ exploit                                                                │
 │ echo "$TARGET"  # OUTPUT: "10.0.0.1" (Protected in main memory!)      │
 └────────────────────────────────────────────────────────────────────────┘
```

---

### Dynamic Scoping in Bash (Critical Architectural Quirk)
Unlike C, Python, or Rust which use *Lexical Scoping*, Bash uses **Dynamic Scoping**. A variable declared as `local` inside Function A is **visible to all child functions called by Function A**.

```bash
func_child() {
    echo "Child sees parent var: $PARENT_VAR" # Accessible due to Dynamic Scoping!
}

func_parent() {
    local PARENT_VAR="SECRET_TOKEN_1337"
    func_child
}

func_parent
```

---

## 4. The Three Return Channels in Bash

Bash functions cannot return complex data structures (objects, arrays) natively. Operators use **three distinct communication channels**:

```
                              ┌── Channel 1: Exit Status (`return 0-255`)
                              │
  [ Function Communication ] ─┼── Channel 2: Standard Output Capture (`stdout`)
                              │
                              └── Channel 3: Pass-by-Reference (`declare -n` Namerefs)
```

---

### Channel 1: Numeric Exit Status (`return`)
Used exclusively to return boolean success (`0`) or failure (`1-255`) states:

```bash
is_root() {
    if [[ "$EUID" -eq 0 ]]; then
        return 0                      # SUCCESS / TRUE
    else
        return 1                      # FAILURE / FALSE
    fi
}

# Evaluated directly inside conditional statements:
if is_root; then
    echo "[+] Running as Superuser."
else
    echo "[-] Access Denied: Requires Root privileges." >&2
    exit 1
fi
```

---

### Channel 2: Standard Output Stream Capture (`$(...)`)
The function prints data to `stdout`, and the caller captures it via command substitution:

```bash
generate_sha256() {
    local raw_data="$1"
    echo -n "$raw_data" | sha256sum | awk '{print $1}'
}

# Capture returned string:
HASH_VALUE=$(generate_sha256 "operator_password")
echo "Computed Hash: $HASH_VALUE"
```
> ⚠️ **Performance Warning:** `RESULT=$(func)` spawns an internal subshell. Calling this thousands of times inside loops incurs CPU overhead.

---

### Channel 3: High-Performance Pass-by-Reference (`declare -n`)
Introduced in Bash 4.3+, **Namerefs (`declare -n`)** allow functions to modify caller variables and arrays directly in memory **without spawning subshells**.

```bash
# Function accepts a variable reference by name (Pointer mechanics)
normalize_ip() {
    declare -n ip_ref="$1"            # 'ip_ref' becomes an alias/pointer to the caller's variable
    ip_ref="${ip_ref// /}"            # Strip all spaces directly from caller's memory
    ip_ref="${ip_ref,,}"              # Convert to lowercase
}

TARGET_NODE="  192.168.1.100  "
normalize_ip TARGET_NODE              # Pass the NAME of the variable (without '$')

echo "Cleaned IP: '$TARGET_NODE'"     # Output: Cleaned IP: '192.168.1.100'
```

---

## 5. Library Modularization & Function Exporting

### 1. Creating Reusable Modular Libraries
In enterprise scripts, shared functions are stored in external `.sh` modules and imported using `source`:

```bash
# File: /opt/iw_arsenal/lib/net_utils.sh
check_alive() {
    ping -c 1 -W 1 "$1" &>/dev/null
}

# Main Script: /opt/iw_arsenal/bin/scanner.sh
#!/usr/bin/env bash
source /opt/iw_arsenal/lib/net_utils.sh || { echo "[-] Failed to load net_utils library"; exit 1; }

if check_alive "10.10.14.1"; then
    echo "[+] Node is online."
fi
```

---

### 2. Exporting Functions to Child Subshells (`export -f`)
By default, functions do **not** survive when a subshell is spawned. `export -f` makes the function available across all child processes and `xargs` worker threads:

```bash
probe_node() {
    echo "[*] Worker $$ probing: $1"
}

export -f probe_node                 # Export function to environment table

# Now available to multi-threaded xargs:
echo -e "10.10.1.1\n10.10.1.2" | xargs -n 1 -P 2 bash -c 'probe_node "$@"' _
```

---

## 6. Cyber Operations Practical Labs

### 🛠️ Lab 1: Fast TCP Banner Grabber Function

```bash
#!/usr/bin/env bash

# Pure Bash socket banner grabber (Zero external dependencies)
grab_banner() {
    local target="${1:?Target IP required}"
    local port="${2:?Target Port required}"
    local timeout="${3:-3}"
    local banner=""

    # Open bidirectional TCP socket on File Descriptor 3
    if exec 3<>/dev/tcp/"$target"/"$port" 2>/dev/null; then
        # Send HTTP probe if port 80/8080, else wait for raw daemon banner
        if [[ "$port" == "80" || "$port" == "8080" ]]; then
            echo -e "HEAD / HTTP/1.0\r\n\r\n" >&3
        fi
        
        # Read response with timeout
        read -r -t "$timeout" banner <&3
        exec 3>&-                     # Close File Descriptor 3
        
        echo "$banner"
        return 0
    else
        return 1
    fi
}

# Execution
echo "[*] Querying SSH Banner..."
BANNER=$(grab_banner "127.0.0.1" "22")
echo -e "[+] Captured Banner: \033[1;32m$BANNER\033[0m"
```

---

## 7. Function Operations Reference Matrix

| Feature / Objective | Syntax Pattern | Functional Mechanism |
| :--- | :--- | :--- |
| **Local Variable** | `local VAR="val"` | Restricts variable memory to current function frame |
| **Readonly Local** | `local -r VAR="val"` | Immutable stack-local constant |
| **Pass-by-Reference**| `declare -n REF="$1"` | Modifies caller's variable without subshell capture |
| **Exit Status** | `return 0` / `return 1` | Sets `$?` exit code for conditional evaluations |
| **Stream Capture** | `OUT=$(my_func "$1")` | Captures `stdout` buffer via command substitution |
| **Export Function** | `export -f my_func` | Propagates function definition to child subshells |
| **Inspect Function** | `type -a my_func` | Prints full source code definition from memory |
| **Delete Function** | `unset -f my_func` | Strips function from shell memory table |

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
