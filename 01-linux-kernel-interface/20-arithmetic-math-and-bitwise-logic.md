<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 01 — Linux Kernel Interface & Core CLI
   DOCUMENT: Day 20 — Mathematical Computations, Arithmetic & Bitwise Logic
   ========================================================================= -->

# 🛡️ Day 20: Mathematical Computations, Arithmetic & Bitwise Logic

> **IW Cyber Ops Research Vault | Module 01: Linux & Systems Foundations**  
> *Author: Muhammad Imran (@iwcyberops)*  
> *Track: Shell Arithmetic Engines, Floating-Point Precision, Bitmasks & XOR Math*

---

## 1. The Shell Arithmetic Architecture

Because Bash treats all variables as plain-text strings by default, evaluating mathematical expressions requires routing through the shell's **Arithmetic Expansion Engine: `$(( ... ))`**.

```
    [ Variable X="10" ] ──+
                          ├── ( Plain Concatenation ) ──> X + Y = "10+20"  (String)
    [ Variable Y="20" ] ──+
                          │
                          └── ( Engine: $(( X + Y )) ) ──> 30  (Integer Evaluation)
```

---

### Arithmetic Evaluation Mechanisms Comparison

| Mechanism | Syntax | Type | Portability & Performance |
| :--- | :--- | :--- | :--- |
| **Arithmetic Expansion** | `$(( expression ))` | Shell Built-in | ⚡ **Fastest & Standard:** POSIX compliant; no subshells. |
| **`let` Command** | `let "VAR = expression"` | Shell Built-in | Fast, but does not return text directly to `stdout`. |
| **`expr` Utility** | `expr $X + $Y` | External Binary | 🐢 **Slow:** Spawns external `/usr/bin/expr` process. Deprecated. |
| **`bc` Calculator** | `echo "..." \| bc` | External Engine | Required for **Floating-Point (Decimal)** calculations. |

> 💡 **Parser Rule:** Inside `$(( ... ))`, variable names do **not** require the `$` prefix. `$(( a + b ))` is identical to `$(( $a + $b ))` but parses faster.

---

## 2. Integer Operators & Number Bases

Bash natively supports standard 64-bit signed integer arithmetic.

```bash
# Core Mathematical Operations
A=20
B=6

ADD=$(( A + B ))                      # Addition: 26
SUB=$(( A - B ))                      # Subtraction: 14
MUL=$(( A * B ))                      # Multiplication: 120 (No wildcard escaping required!)
DIV=$(( A / B ))                      # Integer Division (Truncates decimals): 3
MOD=$(( A % B ))                      # Modulo (Remainder of 20 / 6): 2
EXP=$(( A ** 2 ))                     # Exponentiation (20^2): 400
```

---

### Increment / Decrement & Compound Assignment

```bash
COUNT=10

# 1. Post-Increment vs Pre-Increment
echo $(( COUNT++ ))                   # Prints 10, then increments COUNT to 11
echo $(( ++COUNT ))                   # Increments COUNT to 12, then prints 12

# 2. Compound Assignment Operators
(( COUNT += 5 ))                      # Equivalent to: COUNT = COUNT + 5
(( COUNT *= 2 ))                      # Equivalent to: COUNT = COUNT * 2
```

---

### Number Base Conversions (Hexadecimal, Octal, Binary)

Bash allows performing calculations directly across different mathematical radices using the **`base#number`** syntax:

```bash
# Base Syntax: base#value
HEX_VAL=$(( 16#FF ))                  # Hexadecimal FF to Decimal -> 255
BIN_VAL=$(( 2#11001000 ))             # Binary 11001000 to Decimal -> 200
OCT_VAL=$(( 8#755 ))                  # Octal 755 to Decimal -> 493

# Native Prefixes:
echo $(( 0xFF ))                      # Hex prefix '0x' -> 255
echo $(( 0755 ))                      # Octal prefix '0' -> 493
```

---

## 3. Floating-Point Precision Engine: `bc`

Because the native shell arithmetic engine **only evaluates integers** ($10 / 3 = 3$), decimal calculations must be piped into **`bc` (Basic Calculator)**.

$$\text{Syntax Pattern: } \mathbf{\text{echo "scale=N; EXPRESSION" | bc}}$$

```
┌────────────────────────────────────────────────────────────────────────┐
│                        THE 'scale' DIRECTIVE                           │
├────────────────────────────────────────────────────────────────────────┤
│ `scale` defines the number of decimal digits after the decimal point.  │
│ Default scale is 0 (Integer mode). You must set scale explicitly.      │
└────────────────────────────────────────────────────────────────────────┘
```

```bash
# 1. Basic Floating-Point Division with Precision
RESULT=$(echo "scale=4; 10 / 3" | bc)
echo "$RESULT"                        # Output: 3.3333

# 2. Advanced Float Calculations using Here-Strings (Zero Pipe Overhead)
CPU_USAGE=84.65
THRESHOLD=90.00
DIFF=$(bc <<< "scale=2; $THRESHOLD - $CPU_USAGE")
echo "Buffer remaining: $DIFF%"       # Output: Buffer remaining: 5.35%

# 3. Floating-Point Boolean Conditionals in Bash
# 'bc' outputs 1 for TRUE and 0 for FALSE:
IS_GREATER=$(bc <<< "$CPU_USAGE > 80.0")

if [[ "$IS_GREATER" -eq 1 ]]; then
    echo "[!] Alert: High CPU Load detected!"
fi
```

---

## 4. Low-Level Bitwise Logic & Cyber Operations

Bitwise operators manipulate individual binary bits of integers in memory. These are essential for subnet calculations, binary flag checking, and simple data obfuscation.

```
       Bitwise AND (&)                Bitwise OR (|)                 Bitwise XOR (^)
    1 1 0 0  (12)                  1 1 0 0  (12)                  1 1 0 0  (12)
  & 1 0 1 0  (10)                | 1 0 1 0  (10)                ^ 1 0 1 0  (10)
    ───────                        ───────                        ───────
    1 0 0 0  (8)                   1 1 1 0  (14)                  0 1 1 0  (6)
```

---

### Bitwise Operator Matrix

```bash
A=12  # Binary: 1100
B=10  # Binary: 1010

echo $(( A & B ))                     # Bitwise AND: 8  (Binary: 1000)
echo $(( A | B ))                     # Bitwise OR:  14 (Binary: 1110)
echo $(( A ^ B ))                     # Bitwise XOR: 6  (Binary: 0110)
echo $(( ~A ))                        # Bitwise NOT (Invert bits): -13 (Two's complement)
echo $(( A << 2 ))                    # Bit Shift Left (Multiply by 2^2): 48
echo $(( A >> 2 ))                    # Bit Shift Right (Divide by 2^2): 3
```

---

### 🛠️ Cyber Ops Use Case 1: IP Subnet CIDR Mask Calculation

A `/24` subnet mask has 24 leading ones (`255.255.255.0`). We can compute the total usable hosts dynamically using bit shifts:

$$\text{Usable Hosts} = (2^{(32 - \text{CIDR})}) - 2$$

```bash
CIDR=28                               # Target Subnet: /28

TOTAL_IPS=$(( 1 << (32 - CIDR) ))    # 1 << 4 = 16 Total IP addresses
USABLE_HOSTS=$(( TOTAL_IPS - 2 ))     # Subtract Network & Broadcast IDs

echo "Total IPs: $TOTAL_IPS | Usable Hosts: $USABLE_HOSTS"
# Output: Total IPs: 16 | Usable Hosts: 14
```

---

### 🛠️ Cyber Ops Use Case 2: In-Memory XOR String Obfuscation

XOR encryption is symmetric: $\mathbf{(\text{Data} \oplus \text{Key}) \oplus \text{Key} = \text{Data}}$.

```bash
KEY=0x5A                              # Single-byte XOR Key (Decimal 90)
BYTE=0x41                             # Character 'A' (Decimal 65)

# Encrypt:
CIPHER=$(( BYTE ^ KEY ))              # 65 ^ 90 = 27 (Obfuscated Byte)

# Decrypt:
PLAIN=$(( CIPHER ^ KEY ))             # 27 ^ 90 = 65 ('A' Recovered)
printf "Recovered ASCII Character: \\x$(printf '%x' $PLAIN)\n"
# Output: Recovered ASCII Character: A
```

---

## 5. Critical Arithmetic Traps & Error Handling

### ⚠️ Trap 1: The Fatal Octal Trap (Leading Zero Bug)
In Bash arithmetic, any number starting with a leading zero `0` is automatically parsed as **Octal (Base-8)**!

* **Bug Scenario:** Parsing date strings or months (`08` for August, `09` for September).
```bash
MONTH="08"
echo $(( MONTH + 1 ))
# ERROR: bash: 08: value too great for base (error token is "08")
# (Because Octal digits only range from 0 to 7; '8' is illegal!)
```

* **🛡️ Defensive Fix (Force Base 10):**
```bash
MONTH="08"
echo $(( 10#$MONTH + 1 ))             # '10#' forces decimal parsing. Output: 9
```

---

### ⚠️ Trap 2: Division by Zero Crashes
Dividing by zero in Bash generates an uncatchable runtime error that will terminate a script if strict error modes are enabled:

```bash
# Defensive Guard:
DIVISOR=0
if [[ "$DIVISOR" -ne 0 ]]; then
    RESULT=$(( 100 / DIVISOR ))
else
    echo "[-] Error: Division by zero avoided!"
fi
```

---

## 6. Math & Bitwise Reference Matrix

| Operation | Syntax | Description | Example Output |
| :--- | :--- | :--- | :--- |
| **Add / Sub** | `$(( A + B ))` / `$(( A - B ))` | Standard Integer Addition/Subtraction | `$(( 10 + 5 )) = 15` |
| **Mul / Div** | `$(( A * B ))` / `$(( A / B ))` | Multiplication / Floor Division | `$(( 10 / 4 )) = 2` |
| **Modulo** | `$(( A % B ))` | Returns division remainder | `$(( 10 % 3 )) = 1` |
| **Power** | `$(( A ** B ))` | Raises A to power B | `$(( 2 ** 8 )) = 256` |
| **Float Math** | `bc <<< "scale=2; 5/2"` | Precision decimal calculation | `2.50` |
| **Shift Left** | `$(( A << N ))` | Fast multiply: $A \times 2^N$ | `$(( 1 << 8 )) = 256` |
| **Bitwise XOR**| `$(( A ^ B ))` | Binary exclusive OR (Obfuscation) | `$(( 0xFF ^ 0xAA )) = 85` |
| **Base Force** | `$(( 10#$VAR ))` | Strips octal interpretation on leading `0` | `$(( 10#09 )) = 9` |

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
