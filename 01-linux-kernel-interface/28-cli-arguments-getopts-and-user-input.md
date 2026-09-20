<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 01 — Linux Kernel Interface & Core CLI
   DOCUMENT: Day 28 — CLI Arguments, Getopts & Interactive User Input
   ========================================================================= -->

# 🛡️ Day 28: CLI Arguments, Getopts & Interactive User Input

> **IW Cyber Ops Research Vault | Module 01: Linux & Systems Foundations**  
> *Author: Muhammad Imran (@iwcyberops)*  
> *Track: Option Parsing State Machines, POSIX CLI Standards & Secure Inputs*

---

## 1. The CLI Argument Processing Architecture

Professional CLI security utilities adhere to standard POSIX conventions:
* **Short Flags:** Single hyphen followed by a single character (e.g., `-v`, `-h`).
* **Flags with Values:** Flags accepting arguments (e.g., `-t 10.10.14.5`, `-p 8080`).
* **Flag Bundling:** Merging multiple boolean flags together (e.g., `-vh` instead of `-v -h`).
* **Long Flags:** Double hyphens for full words (e.g., `--target`, `--verbose`).

```
                    ┌────────────────────────────────────────────────────────┐
                    │      INPUT: ./tool.sh -v -t 10.10.14.5 -p 4444         │
                    └──────────────────────────┬─────────────────────────────┘
                                               │
                                               ▼
                    ┌────────────────────────────────────────────────────────┐
                    │               THE GETOPTS STATE ENGINE                 │
                    │  1. Parses '-v'  ──> Sets VERBOSE=true                 │
                    │  2. Parses '-t'  ──> Reads $OPTARG ("10.10.14.5")      │
                    │  3. Parses '-p'  ──> Reads $OPTARG ("4444")            │
                    │  4. Advances $OPTIND pointer index to end of flags     │
                    └──────────────────────────┬─────────────────────────────┘
                                               │
                                               ▼
                    ┌────────────────────────────────────────────────────────┐
                    │       shift $(( OPTIND - 1 )) ──> Remaining Raw Args   │
                    └────────────────────────────────────────────────────────┘
```

---

## 2. The `getopts` State Machine & Syntax Rules

`getopts` is a high-performance shell built-in designed to parse short options, handle flag bundling, and extract arguments safely.

$$\text{Syntax: } \mathbf{\text{while getopts ":ht:p:v" OPTION; do ... done}}$$

---

### 🔍 Anatomy of the `OPTSTRING` (`":ht:p:v"`)

The option string defines how the parser evaluates flags:

```text
: h t : p : v
│ │ │ │ │ │ └── Flag 'v' (Boolean: No argument expected)
│ │ │ │ └───┴── Flag 'p' followed by ':' (Requires mandatory argument!)
│ │ └───┴────── Flag 't' followed by ':' (Requires mandatory argument!)
│ └──────────── Flag 'h' (Boolean: No argument expected)
└────────────── Leading ':' enables SILENT ERROR MODE (Suppresses shell noise)
```

1. **A character without a colon (`h`, `v`):** A standalone boolean switch (True/False).
2. **A character followed by a colon (`t:`, `p:`):** Expects a **mandatory argument** immediately following the flag. The argument value is automatically loaded into the **`$OPTARG`** variable.
3. **Leading colon (`:ht:p:`):** Enables **Silent Error Mode**. Instead of printing ugly default shell errors to the screen, missing arguments route to `:` and unknown flags route to `?`, allowing custom error handling.

---

### The Internal `getopts` State Variables

| Variable | Maintained By | Purpose & Behavior |
| :---: | :---: | :--- |
| **`$OPTARG`** | Kernel/Shell | Stores the value passed to an option (e.g., for `-t 10.0.0.1`, `$OPTARG` is `"10.0.0.1"`). |
| **`$OPTIND`** | Kernel/Shell | The **Index Pointer** tracking the position of the next argument to be evaluated. |
| **`$OPTERR`** | User/Script | Controls default shell error printing (`1` = Print errors, `0` = Disable). |

---

## 3. Shifting Positional Parameters (`shift`)

After `getopts` finishes parsing all flags, the processed options still occupy the script's positional parameters (`$1`, `$2`, etc.).

To access any remaining non-flag arguments (such as trailing target filenames or URLs), operators use **`shift`**:

```bash
# Shift the positional parameter array by the number of parsed options:
shift $(( OPTIND - 1 ))

# $1 now points to the first RAW argument AFTER all the flags!
REMAINING_TARGET="$1"
```

---

## 4. Parsing Long Options (`--target`, `--port`) in Pure Bash

Because POSIX `getopts` natively supports only single-character short options, long options are handled using a custom `while-case` parser with explicit `shift` control:

```bash
#!/usr/bin/env bash

TARGET=""
PORT=80
VERBOSE=false

while [[ $# -gt 0 ]]; do
    case "$1" in
        -t|--target)
            TARGET="${2:?[-] Error: --target requires a value!}"
            shift 2 # Consume flag and its argument value
            ;;
        -p|--port)
            PORT="${2:?[-] Error: --port requires a value!}"
            shift 2
            ;;
        -v|--verbose)
            VERBOSE=true
            shift 1 # Consume boolean flag only
            ;;
        -h|--help)
            echo "Usage: $0 --target <IP> [--port <PORT>] [--verbose]"
            exit 0
            ;;
        --) # Explicit end of options delimiter
            shift
            break
            ;;
        *)
            echo "[-] Unknown option: $1" >&2
            exit 1
            ;;
    esac
done
```

---

## 5. Interactive Terminal Input Engineering (`read`)

The `read` built-in captures interactive user input from standard input (`stdin`).

### Practical `read` Modifiers

```bash
# 1. Inline Prompt (-p) and Raw Mode (-r)
read -r -p "[?] Enter Target IP Address: " TARGET_IP

# 2. Silent / Secret Password Entry (-s)
# Disables terminal echo (Essential for passwords/tokens):
read -r -s -p "[?] Enter Root Password: " SECRET_KEY
echo "" # Mandatory newline after silent read

# 3. Timeout Constraint (-t)
# Aborts/falls back if user does not respond within 5 seconds:
if read -r -t 5 -p "[?] Continue execution? (Default: Y in 5s): " CONFIRM; then
    echo "[*] User responded: $CONFIRM"
else
    echo -e "\n[*] Timeout reached. Auto-accepting default: YES"
    CONFIRM="Y"
fi

# 4. Single-Character Confirmation (-n 1)
# Returns immediately without waiting for user to hit 'Enter':
read -r -n 1 -p "[?] Deploy exploit payload? [y/N]: " CHOICE
echo ""
if [[ "$CHOICE" =~ ^[Yy]$ ]]; then
    echo "[+] Deploying..."
fi

# 5. Read Line into Array (-a)
read -r -a PORTS -p "[?] Enter space-separated ports: "
# User enters: 80 443 8080 -> PORTS[0]=80, PORTS[1]=443, PORTS[2]=8080
```

---

## 6. Complete Production CLI Framework Template

This master template demonstrates the gold-standard architecture for building professional security tools:

```bash
#!/usr/bin/env bash
# =========================================================================
# IW Cyber Ops — Production CLI Utility Framework
# =========================================================================
set -Eeuo pipefail

# Default Configuration State
TARGET=""
PORT=443
VERBOSE=false
OUTPUT_FILE=""

# Display Help / Usage Manual
usage() {
    cat << EOF
Usage: $(basename "$0") [OPTIONS] <additional_args>

Options:
  -t <IP>       Target IPv4/IPv6 Address (Mandatory)
  -p <PORT>     Target TCP Port (Default: 443)
  -o <FILE>     Output file path to save report
  -v            Enable verbose diagnostic output
  -h            Display this help manual and exit

Example:
  $(basename "$0") -t 10.10.14.5 -p 8080 -v -o report.txt
EOF
    exit 0
}

# =========================================================================
# Option Parsing Engine (getopts)
# =========================================================================
while getopts ":ht:p:o:v" OPT; do
    case "$OPT" in
        t)
            TARGET="$OPTARG"
            ;;
        p)
            PORT="$OPTARG"
            ;;
        o)
            OUTPUT_FILE="$OPTARG"
            ;;
        v)
            VERBOSE=true
            ;;
        h)
            usage
            ;;
        :) # Missing argument for flag expecting a value
            echo -e "\033[1;31m[-] Error: Flag '-$OPTARG' requires a mandatory argument!\033[0m" >&2
            usage
            ;;
        \?) # Unknown invalid flag
            echo -e "\033[1;31m[-] Error: Invalid option '-$OPTARG'!\033[0m" >&2
            usage
            ;;
    esac
done

# Strip parsed flags from positional arguments
shift $(( OPTIND - 1 ))

# =========================================================================
# Input Assertions & Validation
# =========================================================================
if [[ -z "$TARGET" ]]; then
    echo -e "\033[1;31m[-] FATAL: Target IP (-t) is mandatory!\033[0m" >&2
    usage
fi

if ! [[ "$PORT" =~ ^[0-9]+$ ]] || (( PORT < 1 || PORT > 65535 )); then
    echo -e "\033[1;31m[-] FATAL: Port must be an integer between 1 and 65535!\033[0m" >&2
    exit 1
fi

# =========================================================================
# Execution Logic
# =========================================================================
echo -e "\033[1;32m[+] Configuration Parsed Successfully:\033[0m"
echo "  - Target Node: $TARGET"
echo "  - Target Port: $PORT"
echo "  - Verbose:     $VERBOSE"
[[ -n "$OUTPUT_FILE" ]] && echo "  - Report File: $OUTPUT_FILE"
[[ $# -gt 0 ]] && echo "  - Extra Args:  $*"
```

---

## 7. CLI Argument Operations Reference Matrix

| Feature | Syntax Pattern | Functional Mechanism |
| :--- | :--- | :--- |
| **Mandatory Option** | `getopts "t:" OPT` | Flag requires value, loaded into `$OPTARG` |
| **Boolean Option** | `getopts "v" OPT` | True/False flag without argument |
| **Silent Mode** | `getopts ":t:" OPT` | Suppresses default error messages |
| **Missing Value Trap**| `:) echo "Missing arg"` | Catches flags missing required values |
| **Invalid Flag Trap** | `\?) echo "Bad flag"` | Catches unregistered options |
| **Shift Options** | `shift $((OPTIND - 1))` | Advances positional array past flags |
| **Silent Read** | `read -s -p "Pass: "` | Captures passwords without terminal echo |
| **Timed Read** | `read -t 5 -p "Prompt: "`| Aborts read after 5 seconds |

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
