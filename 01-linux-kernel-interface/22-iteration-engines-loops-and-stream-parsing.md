<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 01 — Linux Kernel Interface & Core CLI
   DOCUMENT: Day 22 — Iteration Engines, Loops & Stream Ingestion
   ========================================================================= -->

# 🛡️ Day 22: Iteration Engines, Loops & Stream Ingestion

> **IW Cyber Ops Research Vault | Module 01: Linux & Systems Foundations**  
> *Author: Muhammad Imran (@iwcyberops)*  
> *Track: High-Performance Iteration, Stream Processing, Subshell Traps & Flow Control*

---

## 1. The Shell Iteration Architecture

Loops allow automated repeated execution over lists, numerical ranges, file streams, and conditional runtime polling.

```
                           [ Loop Initialization ]
                                      │
                                      ▼
                        ┌───► [ Evaluate Condition ] ───( False )───┐
                        │                 │                         │
                        │             ( True )                      │
                        │                 ▼                         │
                        │        [ Execute Loop Body ]              │
                        │                 │                         │
                        │                 ▼                         │
                        └─── [ Step / Next Element ]                ▼
                                                           [ Loop Terminated ]
```

### Loop Constructs Comparison

| Construct | Execution Condition | Primary Operational Use Case |
| :--- | :--- | :--- |
| **`for ... in`** | Iterates over discrete elements in a list | Processing wordlists, IP lists, file arrays |
| **`for (( ))`** | C-Style 3-part arithmetic loop | Numerical indexing, fixed-iteration algorithms |
| **`while`** | Continues while condition returns **`0` (TRUE)** | Stream parsing, socket listeners, daemon polling |
| **`until`** | Continues while condition returns **Non-Zero (FALSE)** | Waiting for host/port reachability |

---

## 2. `for` Loop Implementations

### 1. List & Brace Expansion Iteration
$$\text{Syntax: } \mathbf{\text{for VAR in LIST; do ... done}}$$

```bash
# 1. Discrete Wordlist / IP Iteration
for TARGET in "10.10.14.1" "10.10.14.2" "10.10.14.3"; do
    echo "[*] Probing target node: $TARGET"
done

# 2. Brace Expansion (Range with Step Increment)
# Syntax: {START..END..STEP}
for PORT in {20..25}; do
    echo "[*] Scanning Port: $PORT"
done

for OCTET in {10..100..10}; do
    echo "Subnet boundary: 192.168.$OCTET.0/24"
done
```

---

### 2. Filesystem Globbing Iteration
```bash
# Iterate through all configuration files
for CONF in /etc/*.conf; do
    # Guard check: Ensure file exists (Prevents literal '*.conf' error on empty dirs)
    [[ -f "$CONF" ]] || continue
    echo "Auditing config file: $CONF"
done
```
> 💡 **Best Practice:** Enable `shopt -s nullglob` in scripts. This ensures that if no files match the wildcard pattern, the loop does not execute on the raw unexpanded string `*.conf`.

---

### 3. C-Style Arithmetic `for` Loop
$$\text{Syntax: } \mathbf{\text{for (( INIT; CONDITION; STEP )); do ... done}}$$

```bash
# C-style loop for precise numeric control
for (( i = 1; i <= 5; i++ )); do
    echo "Attempt: $i / 5"
done
```

---

## 3. Dynamic Evaluation: `while` & `until`

### 1. The `while` Loop (Condition == TRUE)
```bash
# Continuous Service Polling Daemon
ATTEMPTS=0
MAX_ATTEMPTS=5

while (( ATTEMPTS < MAX_ATTEMPTS )); do
    (( ATTEMPTS++ ))
    echo "[*] Connection attempt $ATTEMPTS..."
    
    # Try connecting to port 80; if successful, break loop
    if nc -zvw1 10.10.10.1 80 &>/dev/null; then
        echo "[+] Service is UP!"
        break
    fi
    sleep 2
done
```

---

### 2. The `until` Loop (Condition == FALSE)
`until` continues executing until the condition evaluates to `0` (Success):

```bash
# Wait until a specific lockfile is deleted by another process
until [[ ! -f "/tmp/app.lock" ]]; do
    echo "[*] Process locked. Waiting for lockfile release..."
    sleep 3
done
echo "[+] Lock released. Proceeding with execution."
```

---

## 4. The Gold Standard: Safe Stream Ingestion (`read -r`)

Reading files line-by-line using `for line in $(cat file)` is a **major anti-pattern**—it collapses whitespace and breaks on paths containing spaces.

The only production-grade way to ingest text streams line-by-line is using **`while IFS= read -r`**:

$$\mathbf{\text{while IFS= read -r LINE || [[ -n "\$LINE" ]]; do ... done < target\_file.txt}}$$

```
                      ┌────────────────────────────────────────────────────────┐
                      │    STREAM INGESTION ENGINE: while IFS= read -r LINE    │
                      ├───────────┬────────────────────────────────────────────┤
                      │ Parameter │ Critical Parsing Function                  │
                      ├───────────┼────────────────────────────────────────────┤
                      │ `IFS=`    │ Clears Internal Field Separator. Prevents  │
                      │           │ stripping leading/trailing spaces & tabs.  │
                      ├───────────┼────────────────────────────────────────────┤
                      │ `-r`      │ Raw Mode. Disables backslash (`\`) escape  │
                      │           │ interpretation (Preserves Windows paths).  │
                      ├───────────┼────────────────────────────────────────────┤
                      │ `\|\| [[`  │ Ensures the final line is processed even if│
                      │ `-n ...`  │ the file lacks a trailing newline (`\n`).  │
                      └───────────┴────────────────────────────────────────────┘
```

```bash
# Production Example: Line-by-Line Wordlist / IP Processing
TARGET_LIST="targets.txt"

while IFS= read -r TARGET || [[ -n "$TARGET" ]]; do
    # Skip blank lines and comments
    [[ -z "$TARGET" || "$TARGET" =~ ^# ]] && continue

    echo "[+] Processing target node: $TARGET"
done < "$TARGET_LIST"
```

---

## 5. ⚠️ The Deadly Piped Subshell Loop Trap

A common bug occurs when piping data directly into a `while` loop:

```bash
# ❌ BROKEN CODE (Variable Mutation Fails):
COUNT=0

cat /etc/passwd | while IFS=: read -r USER _; do
    (( COUNT++ ))
done

echo "Total Users Processed: $COUNT"
# OUTPUT: Total Users Processed: 0   <-- WHY?!
```

### Why Did `$COUNT` Reset to 0?
In Bash, the pipeline operator (`|`) **spawns a child subshell** in an isolated memory space for the right-hand process (`while`). The variable `$COUNT` was incremented inside the subshell, which was destroyed the moment the loop finished!

```
  [ Parent Process ] ──> ( COUNT=0 ) ──[ Stays 0! ] ──> echo $COUNT -> 0
                                │
                                ▼ ( Pipe Spawns Child Subshell )
                      ┌─────────────────────────────────┐
                      │ Subshell Memory: COUNT++ (1,2..)│ ──> [ DIES ON EXIT ]
                      └─────────────────────────────────┘
```

---

### 🛡️ Defensive Fixes:

#### Fix A: Standard Input Redirection (`<`)
```bash
COUNT=0
while IFS=: read -r USER _; do
    (( COUNT++ ))
done < /etc/passwd

echo "Total Users Processed: $COUNT"  # Output: Accurate count!
```

#### Fix B: Process Substitution (`< <(command)`)
```bash
COUNT=0
while read -r OPEN_PORT; do
    (( COUNT++ ))
done < <(ss -tulpn | awk '{print $5}')

echo "Active Sockets: $COUNT"         # Preserves parent variable state!
```

---

## 6. Flow Control: `break` & `continue` (Multi-Tier Escape)

* `continue` — Skips the remainder of the *current iteration* and jumps to the next loop cycle.
* `break` — Terminates the loop immediately.
* `break N` — Breaks out of **`N` levels of nested loops** simultaneously.

```bash
# Multi-Tier Nested Loop Break (break 2)
for SUBNET in "192.168.1" "10.0.0"; do
    for HOST in {1..254}; do
        IP="$SUBNET.$HOST"
        if [[ "$IP" == "192.168.1.50" ]]; then
            echo "[+] Target found: $IP! Terminating ALL search loops."
            break 2   # Breaks BOTH inner ($HOST) and outer ($SUBNET) loops!
        fi
    done
done
```

---

## 7. Cyber Operations Automation Examples

### 🛠️ Example: Pure Bash Network Alive Sweeper (Ping Sweeper)

```bash
#!/usr/bin/env bash
# High-speed subnet alive-check automation

SUBNET="192.168.1"
TIMEOUT=1

echo "[*] Initiating ICMP sweep on ${SUBNET}.0/24..."

for HOST in {1..20}; do
    TARGET="${SUBNET}.${HOST}"
    
    # Send 1 ICMP packet with 1 second timeout; suppress output
    if ping -c 1 -W $TIMEOUT "$TARGET" &>/dev/null; then
        echo -e "\033[1;32m[+] Host ALIVE:\033[0m $TARGET"
    fi
done
```

---

## 8. Loop Architecture Reference Matrix

| Goal / Pattern | Syntax Pattern | Key Advantage |
| :--- | :--- | :--- |
| **Number Range** | `for i in {1..100}; do ... done` | Fast inline range generation |
| **C-Style Counter** | `for (( i=0; i<N; i++ )); do ... done` | Exact numeric index control |
| **Safe File Stream** | `while IFS= read -r L; do ... done < file` | Preserves spaces, backslashes & lines |
| **Command Stream** | `while read -r L; do ... done < <(cmd)` | Prevents piped subshell variable loss |
| **Infinite Daemon** | `while true; do ... sleep N; done` | Continuous monitoring loops |
| **Multi-Loop Exit** | `break 2` | Instantly escapes nested loops |

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
