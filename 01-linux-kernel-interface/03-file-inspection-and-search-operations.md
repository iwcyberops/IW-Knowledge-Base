<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 01 — Linux Kernel Interface & Core CLI
   DOCUMENT: Day 03 — File Viewing, Inspection & Search Operations
   ========================================================================= -->

# 🛡️ Day 03: File Viewing, Inspection & Search Operations

> **IW Cyber Ops Research Vault | Module 01: Linux & Systems Foundations**  
> *Author: Muhammad Imran (@iwcyberops)*  
> *Track: Core Linux CLI, File System Probing & Tactical Search*

---

## 1. File Content Viewing Utilities

### 1. `cat` (Concatenate & Display)
Streams entire file contents directly to standard output (`stdout`).

* `cat file.txt` — Print file content to the terminal.
* `cat -n file.txt` — Number all output lines (useful for code review).
* `cat -b file.txt` — Number only non-blank lines.
* `cat -E file.txt` — Display `$` at the end of each line (identifies hidden trailing spaces/carriage returns).
* `cat -s file.txt` — Squeeze multiple adjacent blank lines into a single blank line.
* `cat f1.txt f2.txt > merged.txt` — Concatenate multiple files into one.

---

### 2. `tac` (Reverse Cat)
Prints file content line-by-line in reverse order (bottom-to-top).

* `tac /var/log/syslog` — Read log events starting from the most recent entries up to the oldest.

---

### 3. `head` (View File Header)
Outputs the beginning (first few lines/bytes) of a file. Defaults to 10 lines.

* `head file.txt` — Display the first 10 lines.
* `head -n 20 file.txt` — Display the first 20 lines.
* `head -c 100 file.txt` — Display the first 100 bytes of the file.

---

### 4. `tail` (View File Footer & Real-Time Monitoring)
Outputs the end (last few lines/bytes) of a file. Defaults to 10 lines.

* `tail file.txt` — Display the last 10 lines.
* `tail -n 25 file.txt` — Display the last 25 lines.
* `tail -f /var/log/auth.log` — Follow mode: dynamically output appended data in real time (crucial for live attack / brute-force monitoring).
* `tail -F /var/log/syslog` — Follow by file name (retries if the log file is rotated/truncated).

---

### 5. `less` & `more` (Terminal Pagers)
Interactive pagers for navigating large files without loading the entire content into terminal memory at once.

* `less large_file.log` — Open interactive viewer (superior to `more`).

#### Key `less` Navigation Shortcuts:
* `Space` / `f` — Scroll forward one full page.
* `b` — Scroll backward one full page.
* `G` — Jump directly to the end of the file.
* `g` — Jump directly to the start of the file.
* `/pattern` — Search forward for a keyword (press `n` for next match, `N` for previous).
* `?pattern` — Search backward for a keyword.
* `q` — Exit the pager cleanly.

---

## 2. File Identification & Metadata Inspection

### 6. `file` (File Type & Magic Bytes Identification)
Determines the true file format based on internal header signatures (Magic Numbers) rather than untrusted file extensions.

* `file payload.bin` — Detect file architecture (ELF 64-bit, ASCII, PDF, etc.).
* `file -b file.txt` — Brief mode; omits the file name from the output.
* `file -i file.png` — Output MIME type string (`image/png; charset=binary`).
* `file -k binary` — Keep going; identify all matching magic signatures instead of stopping at the first match.

> 🔴 **Cyber Ops Context:**  
> Attackers frequently disguise executable payloads with fake extensions (e.g., `backdoor.jpg`). The `file` utility reveals the actual ELF/PE binary header.

---

### 7. `stat` (Detailed Inode & Metadata Inspection)
Displays granular filesystem attributes including Inode number, block allocation, access rights, and standard **MACB** timestamps.

* `stat sample.sh` — Print full metadata record.
* `stat -c "%a %n"` — Display only octal permissions and file name (e.g., `755 sample.sh`).

#### MACB Timestamps Breakdown:
* **Access (atime):** Last time the file was read.
* **Modify (mtime):** Last time the file content was altered.
* **Change (ctime):** Last time file metadata/permissions were changed.
* **Birth:** File creation timestamp (supported on modern filesystems like ext4/Btrfs).

---

### 8. `wc` (Word, Line, & Byte Counter)
Computes newline, word, and byte counts for files or piped inputs.

* `wc file.txt` — Display lines, words, and byte counts.
* `wc -l /etc/passwd` — Count total user accounts registered on the system.
* `wc -w report.txt` — Count total words.
* `wc -c binary.bin` — Count total file size in bytes.

---

## 3. Search & Location Operations

### 9. `grep` (Global Regular Expression Print)
Searches for text patterns inside files.

* `grep "root" /etc/passwd` — Match lines containing the string `root`.
* `grep -i "error" /var/log/syslog` — Case-insensitive search.
* `grep -r "PASSWORD" /var/www/` — Recursively search all files within a directory.
* `grep -n "admin" users.txt` — Display matched lines with their line numbers.
* `grep -v "nologin" /etc/passwd` — Invert match (show lines that do **not** contain `nologin`).
* `grep -c "Failed password" /var/log/auth.log` — Return only the count of matched lines.
* `grep -l "secret" *.conf` — List only the names of files containing matches.

---

### 10. `find` (Real-Time Directory Search Engine)
Recursively searches the filesystem hierarchy using metadata attributes, permissions, names, and sizes.

* `find /etc -name "*.conf"` — Search for files matching exact pattern.
* `find /var/log -iname "*.log"` — Case-insensitive name search.
* `find /home -type f` — Search only for standard files (`f` = file, `d` = directory, `l` = symlink).
* `find / -size +100M` — Locate files larger than 100 MB.
* `find / -perm -4000 -type f 2>/dev/null` — Find all SUID binaries (standard privilege escalation recon).
* `find /tmp -mtime -2` — Find files modified in the last 48 hours.
* `find /tmp -name "*.tmp" -exec rm -f {} \;` — Find and immediately delete matching files.

---

### 11. `which` & `whereis` (Binary & Source Resolution)
* `which nmap` — Returns the exact absolute path of the executable located in `$PATH`.
* `which -a python3` — Displays all matching executables found across the entire `$PATH`.
* `whereis bash` — Locates binary, source code, and man page documentation paths simultaneously.
* `whereis -b ssh` — Search only for binary files.
* `whereis -m ssh` — Search only for manual pages.

---

### 12. `locate` & `updatedb` (Database Index Search)
Fast indexing tool that searches a pre-built local database (`mlocate.db`) instead of crawling the live disk.

* `sudo updatedb` — Force refresh/update the filesystem index database.
* `locate id_rsa` — Instantly search for files containing `id_rsa` in their name.
* `locate -i "exploit"` — Case-insensitive database query.
* `locate -e target.txt` — Match only files that still physically exist on the system.

---

### 13. `tree` (Hierarchical Directory Visualizer)
Renders a graphical, tree-like structure of directories and sub-files.

* `tree` — Display full recursive directory structure.
* `tree -L 2` — Restrict depth to 2 directory levels.
* `tree -d` — List directories only (exclude individual files).
* `tree -a` — Include hidden files in the visualization.

---

## 4. Quick Command & Security Reference

| Command | Primary Use Case | Security / Recon Focus |
| :--- | :--- | :--- |
| `tail -f /var/log/auth.log` | Real-time log monitoring | Track SSH brute-force & login anomalies |
| `file <file>` | Inspect file headers | Unmask hidden binaries disguised as images |
| `stat <file>` | Inspect Inode & MACB times | Forensic timeline reconstruction |
| `grep -rn "API_KEY" /` | Recursive keyword search | Locate hardcoded credentials and secrets |
| `find / -perm -u=s` | Locate SUID executables | Local Privilege Escalation identification |
| `which <tool>` | Verify execution path | Detect binary spoofing & `$PATH` hijacking |

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
