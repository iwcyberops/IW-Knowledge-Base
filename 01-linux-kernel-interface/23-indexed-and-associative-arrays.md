<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 01 — Linux Kernel Interface & Core CLI
   DOCUMENT: Day 23 — Indexed Arrays, Associative Arrays & Data Structures
   ========================================================================= -->

# 🛡️ Day 23: Indexed Arrays, Associative Arrays & Data Structures

> **IW Cyber Ops Research Vault | Module 01: Linux & Systems Foundations**  
> *Author: Muhammad Imran (@iwcyberops)*  
> *Track: In-Memory Structures, Hashmaps, Stream Ingestion & Sparse Arrays*

---

## 1. Array Architecture in Bash

Bash provides two native memory-backed collection models: **Indexed Arrays** (ordered sequential lists referenced by integer indices) and **Associative Arrays** (Hashmaps/Dictionaries referenced by arbitrary string keys).

```
   INDEXED ARRAY (declare -a)                  ASSOCIATIVE ARRAY (declare -A)
  ┌───────┬─────────────────┐                 ┌─────────────┬─────────────────┐
  │ Index │ Stored Value    │                 │ String Key  │ Stored Value    │
  ├───────┼─────────────────┤                 ├─────────────┼─────────────────┤
  │   0   │ "10.10.14.1"    │                 │ "ssh"       │ 22              │
  │   1   │ "10.10.14.2"    │                 │ "http"      │ 80              │
  │   2   │ "10.10.14.5"    │                 │ "https"     │ 443             │
  └───────┴─────────────────┘                 └─────────────┴─────────────────┘
  (Zero-Indexed Integer Pointers)             (Key-Value In-Memory Hash Table)
```

---

## 2. Indexed Arrays (`declare -a`)

### 1. Initialization & Element Appending
```bash
# 1. Compound Assignment Initialization
declare -a TARGETS=("10.10.14.1" "10.10.14.2" "10.10.14.5")

# 2. Individual Index Assignment
TARGETS[3]="10.10.14.9"

# 3. Dynamic Append Operator (+=)
TARGETS+=("10.10.14.15" "10.10.14.20")
```

---

### 2. Reading, Slicing & Array Metrics
<!--$$\text{Syntax Pattern: } \mathbf{\$\{\text{ARRAY}[\text{INDEX}]\} \quad | \quad \$\{\#\text{ARRAY}[@]\} \quad | \quad \$\{\text{ARRAY}[@]:\text{OFFSET}:\text{LEN}\}}$$
-->

```bash
# 1. Accessing Elements
echo "${TARGETS[0]}"                  # First element: "10.10.14.1"
echo "${TARGETS[-1]}"                 # Negative index: Last element ("10.10.14.20")

# 2. Total Count & Length Calculations
echo "${#TARGETS[@]}"                 # Total number of elements stored in the array
echo "${#TARGETS[0]}"                 # Character length of the string at index 0

# 3. Array Slicing (Offset : Length)
echo "${TARGETS[@]:1:2}"              # Slices 2 elements starting from index 1

# 4. Extracting All Active Indices (!)
echo "${!TARGETS[@]}"                 # Output: 0 1 2 3 4 5 (Lists all occupied index IDs)
```

---

### 3. Critical Difference: `"${ARRAY[@]}"` vs `"${ARRAY[*]}"`

```bash
# "${ARRAY[@]}" preserves individual word boundaries (Mandatory for loops):
for IP in "${TARGETS[@]}"; do
    echo "Scanning: $IP"              # Runs 1 iteration per element
done

# "${ARRAY[*]}" flattens the entire array into a single concatenated string:
echo "${TARGETS[*]}"                  # Output: "10.10.14.1 10.10.14.2 10.10.14.5 ..."
```

---

## 3. High-Speed Stream Ingestion: `mapfile` / `readarray`

Instead of looping with `while read`, the built-in `mapfile` (or `readarray`) loads entire files or command outputs directly into an indexed array in **kernel memory at high speed**.

$$\text{Syntax: } \mathbf{\text{mapfile -t ARRAY < filename.txt}}$$

```
┌────────────────────────────────────────────────────────────────────────┐
│                      THE MANDATORY '-t' FLAG                           │
├────────────────────────────────────────────────────────────────────────┤
│ By default, mapfile leaves the trailing newline (`\n`) attached to     │
│ each array element. The `-t` flag STRIPS the trailing newline cleanly! │
└────────────────────────────────────────────────────────────────────────┘
```

```bash
# 1. Load entire wordlist into array in milliseconds
mapfile -t WORDLIST < /usr/share/wordlists/rockyou.txt

# 2. Ingest command output directly via Process Substitution
mapfile -t ACTIVE_USERS < <(awk -F: '$3 >= 1000 {print $1}' /etc/passwd)
echo "Total human accounts loaded: ${#ACTIVE_USERS[@]}"
```

---

## 4. Associative Arrays / Hashmaps (`declare -A`)

Associative arrays store key-value pairs. **They MUST be explicitly declared using `declare -A` before use** (otherwise Bash treats string keys as integer `0`).

$$\text{Syntax: } \mathbf{\text{declare -A HASHMAP}}$$

```bash
# Step 1: Explicit Declaration
declare -A SERVICE_MAP

# Step 2: Assign Key-Value Pairs
SERVICE_MAP["http"]=80
SERVICE_MAP["https"]=443
SERVICE_MAP["ssh"]=22
SERVICE_MAP["smb"]=445

# Or initialize directly:
declare -A PORT_MAP=(
    ["21"]="FTP"
    ["22"]="SSH"
    ["80"]="HTTP"
    ["443"]="HTTPS"
)
```

---

### Querying, Iterating & Key-Existence Checks

```bash
# 1. Read Value by Key
echo "SSH Port is: ${SERVICE_MAP["ssh"]}"  # Output: 22

# 2. Extract All Keys (!)
echo "Configured Services: ${!SERVICE_MAP[@]}"

# 3. Iterate Over Key-Value Pairs
for SERVICE in "${!SERVICE_MAP[@]}"; do
    PORT="${SERVICE_MAP[$SERVICE]}"
    echo "[*] Service: $SERVICE ──> Port: $PORT"
done

# 4. Check If Key Exists in Hashmap (-v Flag)
TARGET_PORT="80"
if [[ -v PORT_MAP["$TARGET_PORT"] ]]; then
    echo "[+] Port $TARGET_PORT is mapped to: ${PORT_MAP[$TARGET_PORT]}"
else
    echo "[-] Port $TARGET_PORT is unmapped/unknown."
fi
```

---

## 5. Deletion & The Sparse Array Trap

When an element is deleted using `unset`, Bash does **not** shift subsequent elements down. The array becomes **Sparse** (contains gaps in index numbering).

```bash
ARR=("A" "B" "C" "D")
# Index: 0   1   2   3

unset ARR[1]                          # Deletes element "B" at Index 1
# Array is now Sparse: Index 0="A", Index 2="C", Index 3="D"
```

```
┌────────────────────────────────────────────────────────────────────────┐
│                     ⚠️ THE SPARSE ARRAY ITERATION BUG                  │
├────────────────────────────────────────────────────────────────────────┤
│ ❌ BROKEN: for (( i=0; i<${#ARR[@]}; i++ )); do echo ${ARR[i]}; done   │
│ (Fails because ${#ARR[@]} is 3, but the highest index is 3! Index 1 is │
│ null, and Index 3 is skipped entirely!)                                │
│                                                                        │
│ ✅ SAFE: for i in "${!ARR[@]}"; do echo "${ARR[i]}"; done              │
│ (Always iterates over actual active indices, ignoring empty gaps!)     │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Cyber Operations Practical Lab

### 🛠️ Lab: In-Memory IP Frequency Counter (Tracking SSH Attackers)

Using an Associative Array as an in-memory hash table to tally attacker attempts directly from `/var/log/auth.log`:

```bash
#!/usr/bin/env bash
# High-speed in-memory telemetry aggregator

declare -A ATTACK_COUNTS

echo "[*] Parsing authentication logs into memory hashmap..."

# Ingest failed login IP stream
while read -r IP; do
    [[ -z "$IP" ]] && continue
    # Increment key counter in hashmap
    (( ATTACK_COUNTS["$IP"]++ ))
done < <(grep "Failed password" /var/log/auth.log 2>/dev/null | awk '{print $(NF-3)}')

echo -e "\n\033[1;31m[!] TOP IDENTIFIED ATTACKERS:\033[0m"
printf "%-18s %s\n" "IP Address" "Failed Attempts"
printf "%-18s %s\n" "----------------" "---------------"

for IP in "${!ATTACK_COUNTS[@]}"; do
    printf "%-18s %d\n" "$IP" "${ATTACK_COUNTS[$IP]}"
done
```

---

## 7. Array Operations Reference Matrix

| Goal / Operation | Syntax | Description |
| :--- | :--- | :--- |
| **Declare Indexed** | `declare -a ARR` | Initialize indexed collection |
| **Declare Associative** | `declare -A MAP` | **Mandatory** for Key-Value Hashmaps |
| **Ingest File** | `mapfile -t ARR < file` | High-speed array population from disk |
| **Total Count** | `${#ARR[@]}` | Returns total stored element count |
| **All Elements** | `"${ARR[@]}"` | Expands elements preserving word boundaries |
| **All Keys / Indices** | `"${!MAP[@]}"` | Returns list of all active keys/indices |
| **Check Key Exists** | `[[ -v MAP["key"] ]]` | Returns TRUE (0) if key is defined |
| **Append Element** | `ARR+=("val")` | Inserts element at the end of array |
| **Delete Element** | `unset ARR[1]` | Removes element (Creates sparse index) |

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
