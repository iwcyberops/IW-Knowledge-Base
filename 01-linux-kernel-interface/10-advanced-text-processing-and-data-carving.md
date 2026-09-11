<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 01 — Linux Kernel Interface & Core CLI
   DOCUMENT: Day 10 — Advanced Text Processing, Stream Wrangling & Data Carving
   ========================================================================= -->

# 🛡️ Day 10: Advanced Text Processing, Stream Wrangling & Data Carving

> **IW Cyber Ops Research Vault | Module 01: Linux & Systems Foundations**  
> *Author: Muhammad Imran (@iwcyberops)*  
> *Track: Text Streams, Log Parsing, In-Memory Data Wrangling & Regex Filters*

---

## 1. Stream Processing Pipeline Architecture

In Unix philosophy, text streams are the universal interface between programs. Raw logs, network captures, memory scrapes, and configuration databases are sliced, transformed, and aggregated via chained pipelines without intermediate disk writes.

```
                     ┌────────────────────────────────────────────────────────┐
                     │              RAW UNSTRUCTURED DATA STREAM              │
                     └──────────────────────────┬─────────────────────────────┘
                                                │
                                                ▼  [ Filter / Delimit ]
                                    ┌───────────────────────┐
                                    │    cut / tr / paste   │
                                    └───────────┬───────────┘
                                                │
                                                ▼  [ Order / Deduplicate ]
                                    ┌───────────────────────┐
                                    │      sort | uniq      │
                                    └───────────┬───────────┘
                                                │
                                                ▼  [ Transform / Compute ]
                                    ┌───────────────────────┐
                                    │       sed / awk       │
                                    └───────────┬───────────┘
                                                │
                                                ▼
                     ┌────────────────────────────────────────────────────────┐
                     │            CLEAN STRUCTURED ACTIONABLE INTEL           │
                     └────────────────────────────────────────────────────────┘
```

---

## 2. Character Translation & Slicing (`cut`, `tr`, `paste`, `column`)

### 1. `cut` (Field & Byte Extractor)
Extracts sections from each line of input based on delimiters, byte offsets, or character counts.

* `cut -d: -f1 /etc/passwd` — Extract 1st column (usernames) using `:` as delimiter.
* `cut -d: -f1,6,7 /etc/passwd` — Extract username, home directory, and shell columns simultaneously.
* `cut -d: -f3- /etc/passwd` — Extract from field 3 to the end of the line.
* `cut -c 1-16 hashes.txt` — Extract exact character range (characters 1 through 16).
* `cut -d: --complement -f2 /etc/passwd` — Return all fields **except** field 2.

---

### 2. `tr` (Translate & Squeeze Filter)
Performs single-character transliteration, deletion, and compression directly on standard input (`stdin`).

* `tr 'a-z' 'A-Z' < input.txt` — Convert lowercase to uppercase (POSIX: `tr '[:lower:]' '[:upper:]'`).
* `tr -d '\r' < win_file.txt > unix_file.txt` — Delete Windows carriage returns (`CRLF` to `LF`).
* `tr -d '0-9' < mixed.txt` — Strip all numeric digits from the stream.
* `tr -s ' ' < spaces.txt` — **Squeeze** multiple consecutive spaces into a single space.
* `tr -cd '[:print:]\n' < binary.bin` — Delete everything **except** printable ASCII and newlines.

---

### 3. `paste` & `column` (Horizontal Merging & Tabulation)
* `paste -d: users.txt passwords.txt` — Merge two files horizontally line-by-line using `:` delimiter.
* `paste -s -d, ips.txt` — Collapse an entire column of IP addresses into a single comma-delimited line.
* `column -t -s: /etc/passwd` — Auto-align colon-delimited data into visually formatted tabular columns.

---

## 3. Sorting, Counting & Deduplication (`sort`, `uniq`, `nl`)

### 1. `sort` (Stream Ordering Engine)
Sorts lines of text based on lexicographical, numerical, or column keys.

* `sort targets.txt` — Standard alphabetical sort (ascending).
* `sort -r targets.txt` — Reverse sort order (descending).
* `sort -u targets.txt` — Sort and eliminate duplicate entries inline.
* `sort -n numbers.txt` — Evaluate strings as actual numerical values (`10` comes after `2`).
* `sort -h sizes.txt` — Sort human-readable storage sizes (`2K`, `5M`, `3G`).
* `sort -t: -k3 -n /etc/passwd` — Sort `/etc/passwd` numerically by the 3rd field (**UID**).
* `sort -k2,2nr data.txt` — Sort exclusively by 2nd field in reverse numeric order.

---

### 2. `uniq` (Adjacent Duplicate Processor)
Filters duplicate adjacent lines from a sorted stream.

* `sort logs.txt | uniq` — Remove all duplicate lines.
* `sort logs.txt | uniq -c` — Count occurrence frequencies of each unique line.
* `sort logs.txt | uniq -c | sort -nr` — Top-frequency analysis (highest occurrences first).
* `sort logs.txt | uniq -u` — Print **only lines that appeared exactly once** (Zero duplicates).
* `sort logs.txt | uniq -d` — Print **only repeated lines** (Discards single occurrences).
* `uniq -i file.txt` — Case-insensitive comparison.

---

### 3. `nl` & `fold` (Line Numbering & Boundary Wrapping)
* `nl -ba script.sh` — Number all lines (including empty/blank lines).
* `nl -w3 -s": " file.txt` — Format line numbers with width 3 and custom separator `: `.
* `fold -w 64 base64.txt` — Wrap continuous text lines to a strict maximum width of 64 characters.
* `fold -w 80 -s text.txt` — Wrap at 80 characters without breaking whole words (`-s`).

---

## 4. Differential Comparison & Set Intersections (`diff`, `comm`, `cmp`)

```
   File A (Sorted)        File B (Sorted)
  ┌──────────────┐       ┌──────────────┐
  │ 192.168.1.1  │       │ 192.168.1.1  │ ──> Common to Both  (Column 3 in 'comm')
  │ 192.168.1.5  │       │ 192.168.1.50 │ ──> Unique to File B (Column 2 in 'comm')
  │ 192.168.1.10 │       └──────────────┘
  └──────────────┘
         │
         └──> Unique to File A (Column 1 in 'comm')
```

### 1. `diff` (Line-by-Line File Comparison)
* `diff old_config.conf new_config.conf` — Compare two files line-by-line.
* `diff -u fileA.txt fileB.txt` — Unified format output (Standard patch/Git format).
* `diff -y -W 120 fileA.txt fileB.txt` — Side-by-side split screen visual comparison.
* `diff -w fileA.txt fileB.txt` — Ignore all whitespace differences.
* `diff -r dirA/ dirB/` — Recursively compare all files inside two directories.

---

### 2. `comm` (Set Operations on Sorted Files)
Compares two sorted files and outputs 3 tab-delimited columns: `[Only in A]` `[Only in B]` `[In Both]`.

* `comm fileA.txt fileB.txt` — Standard 3-column set comparison.
* `comm -12 fileA.txt fileB.txt` — **Intersection:** Show only lines common to BOTH files.
* `comm -23 fileA.txt fileB.txt` — **Difference:** Show lines unique to File A (missing in File B).
* `comm -13 fileA.txt fileB.txt` — **Difference:** Show lines unique to File B (missing in File A).

---

### 3. `cmp` (Byte-by-Byte Binary Inspection)
* `cmp file1.bin file2.bin` — Detect if two files are identical and report first byte offset of difference.
* `cmp -l file1.bin file2.bin` — Print all differing bytes with their decimal values and offsets.

---

## 5. The Stream Editor: `sed`

`sed` parses input line-by-line, applies script transformations via pattern matching, and writes results to `stdout`.

### Syntax Anatomy:
$$\text{sed } \text{'[address] s/pattern/replacement/flags'} \text{ input\_file}$$

```bash
# 1. Basic Substitution
sed 's/admin/root/' users.txt          # Replace FIRST occurrence of 'admin' on each line
sed 's/admin/root/g' users.txt         # Global replacement: Replace ALL occurrences on each line
sed 's/http:/https:/gi' urls.txt       # Case-insensitive global replacement

# 2. In-Place File Editing (-i)
sed -i 's/127.0.0.1/10.10.14.5/g' config.env      # Directly overwrite the file on disk
sed -i.bak 's/DEBUG=1/DEBUG=0/g' settings.py      # Edit in-place AND create 'settings.py.bak'

# 3. Line Deletion Operations
sed '/^#/d' config.conf                # Delete all comment lines starting with '#'
sed '/^$/d' input.txt                  # Delete all empty / blank lines
sed '1,5d' input.txt                   # Delete lines 1 through 5
sed '$d' input.txt                     # Delete the last line of the file

# 4. Selective Line Printing (-n with 'p')
sed -n '10,25p' access.log             # Print only lines 10 through 25
sed -n '/CRITICAL/p' syslog            # Print only lines matching 'CRITICAL'

# 5. Insert, Append & Transform
sed '1i # IW Cyber Ops Build' run.sh  # Insert text at line 1 (before)
sed '$a # EOF' run.sh                  # Append text after the final line
sed 's/^/10.10.10./' hosts.txt         # Prepend prefix to the beginning of every line
sed 's/$/:8080/' hosts.txt             # Append suffix to the end of every line
```

---

## 6. Pattern Scanning & Processing Language: `awk`

`awk` treats each line as a **Record** and splits it into **Fields** (columns) referenced as `$1`, `$2` ... `$NF`. `$0` represents the entire line.

```
                  ┌──────────────────── AWK Line Model ───────────────────┐
                  │                                                       │
                  ▼            ▼            ▼                             ▼
                $1           $2           $3                            $NF
              10.10.1.50   [01:00]      "GET"     ...                  404
              └─────────────────────────────────────────────────────────────┘
                                           $0 (Full Record)
```

### Core Built-in Variables:
* `$0` : Full current line string.
* `$1 .. $NF` : Field 1, Field 2, up to the Last Field (`$NF`).
* `NF` : Number of Fields in the current line.
* `NR` : Current Record Number (Line Number).
* `FS` : Input Field Separator (Default: whitespace).
* `OFS` : Output Field Separator (Default: space).

---

### Tactical `awk` Commands & One-Liners

```bash
# 1. Field Extraction & Custom Delimiters (-F)
awk '{print $1}' access.log                         # Print 1st column (Client IP) from space-delimited log
awk -F: '{print $1, $6}' /etc/passwd                # Custom delimiter ':' -> Print User & Home directory
awk -F: '{print $1 " has home at " $6}' /etc/passwd # Custom formatted string output

# 2. Dynamic Field Targeting via $NF (Last Column)
awk '{print $NF}' scan_results.txt                 # Print the last column regardless of line width
awk '{print $(NF-1)}' data.txt                      # Print the second to last column

# 3. Pattern Matching & Conditional Filtering
awk '$9 == 404 {print $1, $7}' access.log          # Print IP ($1) and Path ($7) ONLY if status code ($9) is 404
awk -F: '$3 >= 1000 {print $1, $3}' /etc/passwd    # List regular human users (UID >= 1000)
awk '/FAILED/ {print $0}' auth.log                  # Print entire line if it contains the keyword 'FAILED'

# 4. Line Number Filtering
awk 'NR >= 5 && NR <= 10' target.txt               # Extract lines 5 through 10
awk 'NR % 2 == 1 {print $0}' list.txt               # Extract only odd-numbered lines (1, 3, 5...)

# 5. Computations & Aggregations (BEGIN / END Blocks)
awk '{sum += $5} END {print "Total Size:", sum}' files.txt # Calculate sum of column 5
awk -F: 'END {print "Total Accounts:", NR}' /etc/passwd    # Count total records processed
```

---

## 7. Cyber Ops Data Wrangling Cheatsheet

| Tactical Objective | One-Liner Pipeline |
| :--- | :--- |
| **Extract Unique IPs from Access Log** | `awk '{print $1}' access.log \| sort -u` |
| **Rank Top Attacking IPs by Request Count** | `awk '{print $1}' access.log \| sort \| uniq -c \| sort -nr \| head -n 10` |
| **Clean Custom Wordlist (Lowercase & Deduplicate)** | `tr '[:upper:]' '[:lower:]' < raw.txt \| sort -u > wordlist.txt` |
| **Strip Trailing Windows Characters from Script** | `sed -i 's/\r$//' payload.sh` |
| **Extract All SUID Binaries Paths Cleanly** | `find / -perm -4000 2>/dev/null \| awk -F/ '{print $NF}' \| sort -u` |
| **Format Raw Network Port Scan to CSV** | `cat ports.txt \| tr -s ' ' ',' \| column -t -s,` |
| **Isolate Unique Active Users from System Auth** | `grep "Accepted password" /var/log/auth.log \| awk '{print $9}' \| sort -u` |

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
