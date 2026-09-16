<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 01 — Linux Kernel Interface & Core CLI
   DOCUMENT: Day 18 — Bash Script Architecture, Shebang Mechanics & Execution
   ========================================================================= -->

# 🛡️ Day 18: Bash Script Architecture, Shebang Mechanics & Execution

> **IW Cyber Ops Research Vault | Module 01: Linux & Systems Foundations**  
> *Author: Muhammad Imran (@iwcyberops)*  
> *Track: Shell Script Engineering, Kernel Loader Mechanics & Execution Isolation*

---

## 1. The Low-Level Anatomy of the Shebang (`#!`)

Every executable script in Linux begins with a **Shebang** (also known as a *hash-bang* or *pound-bang*). This is not a comment; it is a kernel-level directive.

```text
#!/bin/bash
││└──┬────┘
││   └─────── Absolute filesystem path to the target interpreter binary
│└─────────── ASCII 0x21 (Exclamation mark / Bang)
└──────────── ASCII 0x23 (Hash / Pound symbol)
```

---

### How the Linux Kernel Executes a Script (`execve` System Call)

When you run `./recon.sh`, the shell invokes the Linux kernel system call:  
$$\mathbf{execve("./recon.sh", \text{argv}, \text{envp})}$$

```
                           [ User Runs: ./recon.sh ]
                                      │
                                      ▼
                      [ Kernel System Call: execve() ]
                                      │
                                      ▼
             [ Kernel reads first 2 bytes of the file header ]
                                      │
               ┌──────────────────────┴──────────────────────┐
               ▼                                             ▼
     { Bytes == 0x7F 0x45 }                        { Bytes == 0x23 0x21 }
       (0x7F 'E' 'L' 'F')                              ('#' '!')
               │                                             │
               ▼                                             ▼
  [ Native ELF Binary Executable ]              [ Script Detected! ]
  Maps code directly to RAM                      Kernel extracts interpreter path
                                                 from line 1: /bin/bash
                                                             │
                                                             ▼
                                     [ Kernel executes: /bin/bash ./recon.sh ]
```

1. The kernel reads the first 2 bytes of the file.
2. If the bytes are `0x23` (`#`) and `0x21` (`!`), the kernel recognizes the file as a **script**, not a compiled binary.
3. The kernel reads the rest of the first line up to the newline character (`\n`) to identify the interpreter path.
4. The kernel transparently transforms the execution into: `/bin/bash ./recon.sh`.

---

### `#!/bin/bash` vs `#!/usr/bin/env bash`

| Shebang Directive | Resolution Method | Security & Portability Profile |
| :--- | :--- | :--- |
| **`#!/bin/bash`** | **Hardcoded Path:** Direct pointer to `/bin/bash`. | **High Security:** Prevents `$PATH` hijacking. However, fails on systems where Bash lives elsewhere (e.g., `/usr/local/bin/bash` on FreeBSD/macOS). |
| **`#!/usr/bin/env bash`** | **Dynamic Lookup:** `env` searches `$PATH` for `bash`. | **High Portability:** Works across all UNIX-like OSs. **Threat Vector:** If an attacker manipulates `$PATH`, a malicious `bash` binary can be executed instead. |

---

## 2. Script Execution Modes & Process Isolation

How a script is invoked drastically alters its execution context, process hierarchy, and environment variable persistence.

```
       DIRECT / SUBSHELL EXECUTION                        SOURCED EXECUTION
   (./script.sh OR bash script.sh)                     (source script.sh OR . script.sh)

┌─────────────────────────────────────────┐       ┌─────────────────────────────────────────┐
│     PARENT SHELL (Terminal PID: 1001)   │       │     CURRENT SHELL (Terminal PID: 1001)  │
│     - Holds active variables & state    │       │                                         │
└────────────────────┬────────────────────┘       │   All commands, variables, and exits   │
                     │ fork() + execve()          │   run DIRECTLY inside this PID!         │
                     ▼                            └─────────────────────────────────────────┘
┌─────────────────────────────────────────┐
│      CHILD SUBSHELL (PID: 1042)         │
│  - Runs script in isolated memory       │
│  - Variables created here DIE on exit   │
└─────────────────────────────────────────┘
```

---

### Execution Methods Breakdown

### 1. Direct Execution (`./script.sh`)
* **Requirements:** Requires the **Execute Bit** enabled (`chmod +x script.sh`) and an explicit filesystem path (`./` indicates the current directory).
* **Process Behavior:** Spawns a new **child subshell**; uses the interpreter defined in the shebang line.

### 2. Explicit Interpreter Invocation (`bash script.sh`)
* **Requirements:** Does **not** require execute permissions (`chmod +x` is ignored); only requires read access (`r`).
* **Process Behavior:** Spawns a child subshell. **Ignores the Shebang line entirely** and forces execution through the specified binary (`/bin/bash`).

### 3. Sourcing (`source script.sh` or `. script.sh`)
* **Requirements:** Only requires read access (`r`).
* **Process Behavior:** **No subshell is created.** The commands execute inside the **current active shell process**.
* ⚠️ **Security Warning:** If a sourced script contains an `exit` command, it will **close your active terminal window/SSH session** immediately because it terminates PID 1001 directly.

---

## 3. Exit Status Codes (`$?`) & Modulo 256 Arithmetic

Every command, pipeline, and script in Linux returns an **Exit Status Code** (integer) to the parent process upon completion. This value is captured in the special shell variable **`$?`**.

```text
Exit Code: 0       ──> SUCCESS (Command executed without errors)
Exit Code: 1 - 255 ──> FAILURE / ERROR (Non-zero indicates specific error state)
```

---

### ⚠️ The Modulo 256 Wrap-Around Vulnerability

The Linux kernel allocates an **8-bit unsigned integer** for process exit statuses. This limits valid exit codes to the range **`0` to `255`**.

$$\text{Reported Exit Code} = \text{Passed Value} \pmod{256}$$

```bash
# Example Trap:
exit 256    # 256 % 256 = 0  --> KERNEL REPORTS SUCCESS (0)!
exit 257    # 257 % 256 = 1  --> Kernel reports 1
exit 512    # 512 % 256 = 0  --> Reports SUCCESS!
```

> 🔴 **Security Impact:**  
> If an operator writes `exit 256` to indicate an error inside an authentication or validation script, external automation tools (CI/CD, monitoring daemons) reading `$?` receive `0` (Success), completely bypassing security checks!

---

### Standard POSIX Reserved Exit Codes

| Code | Meaning | Typical Trigger |
| :---: | :--- | :--- |
| **`0`** | **Success** | Clean execution with no errors. |
| **`1`** | **General Error** | Catchall for general errors (e.g., `let "var = 0"` division by zero). |
| **`2`** | **Misuse of Built-ins** | Missing keyword, invalid syntax in `if`/`for`. |
| **`126`** | **Cannot Execute** | Target file found, but permissions missing (`chmod -x`). |
| **`127`** | **Command Not Found** | Binary does not exist in `$PATH` or typo in command name. |
| **`128`** | **Invalid Exit Argument** | `exit 3.14` (Exit only accepts integer values). |
| **`130`** | **Terminated by `SIGINT`** | Process terminated via `Ctrl + C` ($128 + \text{Signal } 2 = 130$). |
| **`137`** | **Terminated by `SIGKILL`**| Process forcefully killed via `kill -9` ($128 + \text{Signal } 9 = 137$). |

---

## 4. ANSI Color Escapes & Output Formatting Engine

Professional security scripts and CLI tools use **ANSI Escape Sequences** to deliver structured, colorized terminal telemetry.

### Syntax Anatomy of an ANSI Escape Code

```text
\033 [ 1 ; 32 m
 │   │ │ │  │ │
 │   │ │ │  │ └── Command Terminator: 'm' (Select Graphic Rendition)
 │   │ │ │  └──── Color Code: 32 = Green
 │   │ │ └─────── Parameter Separator: Semicolon (;)
 │   │ └───────── Text Style Attribute: 1 = Bold / Bright (0 = Normal, 4 = Underline)
 │   └─────────── CSI (Control Sequence Introducer): Bracket '['
 └─────────────── ESC Character: Octal \033 (Hex \x1b or Unicode \e)
```

---

### Enterprise Logging Template for Scripts

Copy and paste this standard header at the start of operational scripts:

```bash
#!/usr/bin/env bash

# =========================================================================
# Terminal Formatting & ANSI Color Codes
# =========================================================================
BOLD="\033[1m"
RED="\033[1;31m"
GREEN="\033[1;32m"
YELLOW="\033[1;33m"
BLUE="\033[1;34m"
RESET="\033[0m" # Essential: Clears color buffer back to normal terminal default

# =========================================================================
# Structured Logging Functions
# =========================================================================
log_info()    { echo -e "${BLUE}[*] INFO:${RESET} $1"; }
log_success() { echo -e "${GREEN}[+] SUCCESS:${RESET} $1"; }
log_warn()    { echo -e "${YELLOW}[!] WARNING:${RESET} $1"; }
log_error()   { echo -e "${RED}[-] ERROR:${RESET} $1" >&2; } # Errors redirected to stderr (FD 2)

# =========================================================================
# Example Operational Usage
# =========================================================================
log_info "Initiating target port scan..."
log_success "Target 10.10.14.5 is ALIVE."
log_warn "Host firewall is filtering TCP probes."
log_error "Failed to establish raw socket connection!"
```

---

## 5. Critical Execution Pitfalls & Troubleshooting

### Pitfall 1: The Windows CRLF Carriage Return Error (`^M`)
When a script is edited on Windows and transferred to Linux, Windows uses **`\r\n` (CRLF)** line endings instead of Linux **`\n` (LF)**.

* **Error Symptom:**
  ```text
  bash: ./recon.sh: /bin/bash^M: bad interpreter: No such file or directory
  ```
* **Why it happens:** The kernel reads `#!/bin/bash\r`. Since no binary named `bash\r` exists, execution crashes.
* **Remediation:**
  ```bash
  # Convert Windows CRLF to UNIX LF:
  dos2unix recon.sh
  # Alternative using stream translation:
  tr -d '\r' < recon.sh > recon_clean.sh && mv recon_clean.sh recon.sh
  ```

---

### Pitfall 2: Unquoted Strings in Output Functions
```bash
# BAD:
echo -e \033[1;32m Text \033[0m   # Shell treats '[' as wildcard globbing pattern!

# GOOD:
echo -e "\033[1;32m Text \033[0m" # Always encapsulate escape strings in double quotes
```

---

## 6. Execution & Architecture Reference Matrix

| Execution Command | Subshell Spawned? | Requires `+x`? | Shebang Evaluated? | Exit Terminates Terminal? |
| :--- | :---: | :---: | :---: | :---: |
| `./script.sh` | ✅ **Yes** | ✅ **Yes** | ✅ **Yes** | ❌ No |
| `bash script.sh` | ✅ **Yes** | ❌ No | ❌ No (Ignored) | ❌ No |
| `source script.sh` | ❌ **No** | ❌ No | ❌ No (Ignored) | ⚠️ **YES (If `exit` is called)** |
| `. script.sh` | ❌ **No** | ❌ No | ❌ No (Ignored) | ⚠️ **YES (If `exit` is called)** |

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
