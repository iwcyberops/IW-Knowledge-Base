<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 01 — Linux Kernel Interface & Core CLI
   DOCUMENT: Day 19 — Variables, Scope & Parameter Expansion Mastery
   ========================================================================= -->

# 🛡️ Day 19: Variables, Scope & Parameter Expansion Mastery

> **IW Cyber Ops Research Vault | Module 01: Linux & Systems Foundations**  
> *Author: Muhammad Imran (@iwcyberops)*  
> *Track: Shell Memory Model, Variable Typing, String Carving & Defensive Expansion*

---

## 1. Variable Typing & Declaration Attributes (`declare`)

By default, Bash treats all variables as untyped plain-text strings. The `declare` (or `typeset`) built-in allows operators to enforce strict data types, read-only constraints, and attribute modifications at the memory level.

```
                  ┌────────────────────────────────────────────────────────┐
                  │                 declare [FLAGS] VAR="VAL"              │
                  ├───────────┬────────────────────────────────────────────┤
                  │ Flag      │ Enforced Kernel/Shell Memory Behavior      │
                  ├───────────┼────────────────────────────────────────────┤
                  │ `-r`      │ Read-Only (Immutable Constant)             │
                  │ `-i`      │ Integer Arithmetic Evaluation Only         │
                  │ `-a`      │ Indexed Array                              │
                  │ `-A`      │ Associative Array (Hashmap / Key-Value)    │
                  │ `-x`      │ Exported Environment Variable (Inherited)  │
                  │ `-l`      │ Force Lowercase conversion on assignment   │
                  │ `-u`      │ Force Uppercase conversion on assignment   │
                  └───────────┴────────────────────────────────────────────┘
```

---

### Practical Type Enforcement Examples

```bash
# 1. Read-Only Constant (Cannot be overwritten or deleted via unset)
declare -r C2_SERVER="10.10.14.5"
C2_SERVER="127.0.0.1"                 # FAILS: bash: C2_SERVER: readonly variable
unset C2_SERVER                       # FAILS: bash: unset: C2_SERVER: cannot unset: readonly variable

# 2. Integer Strict Typing (-i)
declare -i PORT=8080
PORT="8080 + 20"                      # Automatically evaluates arithmetic without $(( ))
echo "$PORT"                          # Output: 8100
PORT="malicious_string"               # Evaluates invalid strings to integer: 0

# 3. Case Enforcing Attributes (-u and -l)
declare -u PROTOCOL="tcp"             # Output automatically converted to: "TCP"
declare -l HASH="A1B2C3D4"            # Output automatically converted to: "a1b2c3d4"
```

---

## 2. Positional Parameters & Special Shell Variables

When a script is executed with arguments (e.g., `./scanner.sh 10.10.10.1 80 443`), Bash assigns these arguments to indexed **Positional Parameters**.

```
    Command:  ./scanner.sh   10.10.10.1      80        443      ...      8443
                 │               │           │          │                  │
    Parameter:  $0              $1          $2         $3                ${10}
```

---

### Why Braces `{}` Are Mandatory for Double-Digit Parameters

```bash
echo "$10"      # PARSING ERROR: Evaluates as ($1) followed by the literal character '0'!
echo "${10}"    # CORRECT: Braces force the shell parser to treat '10' as a single multi-digit index.
```

---

### Special Variable Reference Matrix

| Variable | Definition | Security / Operational Focus |
| :---: | :--- | :--- |
| **`$0`** | Script execution path/name | Unmasks how script was called (Used in self-referencing / payload cloning). |
| **`$#`** | Total argument count | Input validation (`if [[ $# -lt 2 ]]; then ... fi`). |
| **`$@`** | Array of all arguments | Expands to separate words when quoted: `"$1" "$2" "$3"`. |
| **`$*`** | Single string of all arguments| Merges all arguments into one string separated by `$IFS`: `"$1 $2 $3"`. |
| **`$$`** | Process ID (PID) of current shell | Generating unique temporary files (e.g., `/tmp/scan_$$`). |
| **`$!`** | PID of last background process | Tracking asynchronous background jobs, reverse shells, and sub-threads. |
| **`$?`** | Exit code of last command | Error checking and assertion testing (`0` = Success). |
| **`$_`** | Final argument of previous command | Re-using dynamic targets in multi-stage CLI operations. |

---

### The Critical Difference: `"$@"` vs `"$*"`

```bash
# Given arguments: "arg 1" "arg 2"

for item in "$*"; do
    echo "$item"                      # Executes 1 iteration: "arg 1 arg 2" (Single string)
done

for item in "$@"; do
    echo "$item"                      # Executes 2 iterations: "arg 1" then "arg 2" (Preserves array)
done
```
> 🛡️ **Defensive Rule:** Always use **`"$@"`** when iterating over user inputs to prevent argument collapsing and word-splitting bugs.

---

## 3. Advanced Parameter Expansion Mechanics

Parameter expansion allows complex string manipulation, fallback assignment, and path slicing directly in Bash **without calling slow external processes** like `sed`, `awk`, `cut`, or `basename`.

$$\text{Syntax Pattern: } \mathbf{\$\{\text{VARIABLE}\text{ OPERATOR }\text{EXPRESSION}\}}$$

---

### 1. Default Value Fallbacks & Assertions

```bash
# 1. Fallback if unset/null (Does NOT modify original variable)
TARGET="${1:-127.0.0.1}"              # If $1 is empty, TARGET="127.0.0.1", but $1 remains empty.

# 2. Assign Default to Variable (Modifies original variable)
: "${PORT:=4444}"                     # If PORT is empty, it is assigned 4444 permanently.

# 3. Alternate Value (Use replacement if variable IS set)
echo "${API_KEY:+Key is configured}"  # Prints string only if API_KEY exists and is not null.

# 4. Mandatory Assertion / Fail-Fast Trap (:?)
TARGET_IP="${1:?[-] ERROR: Target IP address must be supplied as Argument 1!}"
# If $1 is missing, script aborts immediately with non-zero exit code and prints error to stderr.
```

---

### 2. String Length Calculation (`#`)

```bash
TOKEN="a1b2c3d4e5f6"
echo "${#TOKEN}"                      # Output: 12 (Computes length in memory with zero subshell overhead)
```

---

### 3. Substring Extraction & Slicing (`:offset:length`)

```bash
DATA="abcdefghij"
#     0123456789 (Zero-indexed offsets)

echo "${DATA:0:3}"                    # Output: "abc" (Starts at index 0, extracts 3 characters)
echo "${DATA:4}"                      # Output: "efghij" (Starts at index 4 to the end)
echo "${DATA: -3}"                    # Output: "hij" (Negative offset: extracts last 3 characters)
# NOTE: The space before '-3' is MANDATORY to prevent collision with the ':-' fallback operator!
```

---

### 4. Search & Pattern Replacement (`/` and `//`)

```bash
URL="http://target.local:8080/admin/http:login"

echo "${URL/http:/https:}"            # Output: "https://target.local:8080/admin/http:login" (Replaces 1st match)
echo "${URL//http:/https:}"           # Output: "https://target.local:8080/admin/https:login" (Global replace)
echo "${URL/#http:/https:}"           # Anchor match: Replaces ONLY if pattern is at the START
echo "${URL/%login/dashboard}"        # Anchor match: Replaces ONLY if pattern is at the END
```

---

### 5. Prefix & Suffix Truncation (Path & Extension Carving)

This is the fastest native way to parse filesystem paths and filenames.

```
       #  ──> Strip shortest match from the FRONT (Prefix)
       ## ──> Strip longest match from the FRONT (Prefix)
       %  ──> Strip shortest match from the BACK (Suffix)
       %% ──> Strip longest match from the BACK (Suffix)
```

```bash
FILEPATH="/var/log/audit/auth.log.tar.gz"

# 1. Emulate 'dirname' (Strip everything from the back after last '/')
echo "${FILEPATH%/*}"                 # Output: "/var/log/audit"

# 2. Emulate 'basename' (Strip everything from the front up to last '/')
echo "${FILEPATH##*/}"                # Output: "auth.log.tar.gz"

# 3. Strip Single File Extension (Shortest match from back)
echo "${FILEPATH%.*}"                 # Output: "/var/log/audit/auth.log.tar"

# 4. Strip All Nested Extensions (Longest match from back)
echo "${FILEPATH%%.*}"                # Output: "/var/log/audit/auth"
```

---

## 4. Indirect Variable Referencing (`!`)

Indirect expansion treats the value of a variable as the name of another target variable (Pointer mechanics).

```bash
DEV_IP="192.168.1.10"
PROD_IP="10.0.0.1"

ENVIRONMENT="PROD_IP"

# Read the value of whichever variable is referenced by $ENVIRONMENT:
echo "${!ENVIRONMENT}"                # Output: "10.0.0.1"
```

---

## 5. Security & Defensive Coding Traps

### ⚠️ The Devastating Unset Root Path Bug

Consider an unvalidated cleanup routine:
```bash
# VULNERABLE CODE:
TARGET_DIR=""                         # Variable was unset due to failed input
rm -rf /tmp/staging/${TARGET_DIR}/*   # Expands to: rm -rf /tmp/staging//*
# What if the script had: rm -rf /${TARGET_DIR} -> EXPANDS TO: rm -rf / !
```

### 🛡️ Defensive Hardening Fix:
```bash
# SECURE HARDENING:
# Force abort if variable is unset or empty using :?
rm -rf "/tmp/staging/${TARGET_DIR:?Error: Target directory not set!}"/*
```

---

## 6. Parameter Expansion Quick Reference Matrix

| Operator | Syntax | Functional Result | Example (`VAR="recon_10.10.14.5.tar.gz"`) |
| :--- | :--- | :--- | :--- |
| **Length** | `${#VAR}` | Returns character length | `23` |
| **Default** | `${VAR:-default}` | Fallback if unset/null | Returns value of `$VAR` |
| **Assign** | `${VAR:=default}` | Set default permanently if unset | Modifies and returns `$VAR` |
| **Slice** | `${VAR:6:10}` | Extract substring from offset | `10.10.14.5` |
| **Replace** | `${VAR//10./172.}` | Global find and replace | `recon_172.172.14.5.tar.gz` |
| **Strip Prefix** | `${VAR##*_}` | Strip longest prefix up to `_` | `10.10.14.5.tar.gz` |
| **Strip Suffix** | `${VAR%%.*}` | Strip longest suffix from first `.` | `recon_10` |
| **Uppercase** | `${VAR^^}` | Convert all characters to uppercase | `RECON_10.10.14.5.TAR.GZ` |

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
