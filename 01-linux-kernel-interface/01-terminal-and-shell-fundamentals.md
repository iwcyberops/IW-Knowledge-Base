
<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 01 — Linux Kernel Interface & Core CLI
   DOCUMENT: Day 01 — Terminal & Shell Fundamentals
   ========================================================================= -->

# 🛡️ Day 01: Terminal & Shell Fundamentals

> **IW Cyber Ops Research Vault | Module 01: Linux & Systems Foundations**  
> *Author: Muhammad Imran (@iwcyberops)*  
> *Track: Core Linux CLI, Kernel Interface & System Architecture*

---

## 1. System Architecture: Terminal vs Shell vs Kernel

| Component | Function |
| :--- | :--- |
| **Terminal** | Graphical / text interface (GUI/TUI) used to accept input and display output. |
| **Shell** | Command interpreter that processes user commands and communicates with the OS. |
| **Kernel** | The core of the OS managing CPU, Memory, I/O devices, Processes, and Filesystem. |

```
[ User ] ──> [ Terminal (Interface) ] ──> [ Shell (Interpreter) ] ──> [ Linux Kernel (Core) ] ──> [ Hardware ]
```

---

## 2. Shell Prompt & Privileges

```text
┌──(user㉿kali)-[~]
└─$ 
```

* `$` — Standard non-privileged user prompt.
* `#` — Root/Superuser prompt (Administrative privileges).

### Shell Diagnostic Variables
* `echo $SHELL` — Displays the path of the current active shell (e.g., `/bin/zsh` or `/bin/bash`).
* `echo $$` — Prints the Process ID (PID) of the current shell session.

---

## 3. Path Resolution: Absolute vs Relative

* **Absolute Path:** Path starting from the root directory (`/`). Always reaches the target regardless of current location.
  * *Example:* `/home/user/Downloads/file.txt`
* **Relative Path:** Path calculated relative to the current working directory.
  * *Example:* `./Downloads/file.txt` or `../project/file.txt`

### Special Path Shortcuts
* `~` — Current user's home directory (`/home/username` or `/root`).
* `.` — Current working directory.
* `..` — Parent directory (one level up).
* `-` — Previous working directory (toggle between last two paths).

---

## 4. Essential CLI Navigation & Management

> **Quick Memory Triad:**
> * `pwd` → *Where am I?*
> * `ls`  → *What's here?*
> * `cd`  → *Go somewhere.*

### Navigation & Identification
* `pwd` — Print Current Working Directory.
* `cd [path]` — Change directory.
* `cd` (without arguments) — Returns directly to `$HOME`.
* `whoami` — Prints current logged-in username.
* `hostname` — Displays the system network name.
* `clear` / `Ctrl + L` — Clears the terminal screen buffer.

### Directory Operations (`mkdir`, `rmdir`)
* `mkdir dir1` — Create a directory.
* `mkdir dir1 dir2 dir3` — Create multiple directories in one command.
* `mkdir -p /path/to/nested/dir` — Create parent directories recursively if they do not exist.
* `rmdir dir1` — Remove an **empty** directory.

### File Creation (`touch`)
* `touch file.txt` — Create an empty file or update timestamp of an existing file.
* `touch f1.txt f2.sh f3.c` — Create multiple files simultaneously.
* `touch -t YYYYMMDDhhmm file.txt` — Explicitly set file modification timestamp.

### Directory Listing (`ls`)
* `ls` — List directory contents.
* `ls -l` — Detailed long-listing format (permissions, ownership, size, date).
* `ls -a` — Show all files (including hidden files starting with `.`).
* `ls -A` — Show all files except current (`.`) and parent (`..`) directories.
* `ls -h` — Human-readable file sizes (KB, MB, GB) when combined with `-l` (`ls -lh`).
* `ls -t` — Sort listing by modification time (newest first).
* `ls -S` — Sort listing by file size.
* `ls -r` — Reverse any applied sort order.
* `ls -la /path` — Combine flags for comprehensive listing.

#### Breakdown of `ls -l` Output:
```text
- rw-r--r--  1  user  user  123  Aug 13  test.txt
│     │       │    │     │    │       │       └── Name
│     │       │    │     │    │       └──────── Last Modified Date
│     │       │    │     │    └──────────────── File Size (Bytes)
│     │       │    │     └───────────────────── Group Owner
│     │       │    └─────────────────────────── User Owner
│     │       └──────────────────────────────── Hard Link Count
│     └──────────────────────────────────────── File Permissions (Owner/Group/Others)
└────────────────────────────────────────────── File Type (- = file, d = directory, l = symlink)
```

### Copying Files & Directories (`cp`)
* `cp src.txt dest.txt` — Copy file to target location.
* `cp -r dir1/ dir2/` — Recursively copy directory and its contents.
* `cp -i src.txt dest/` — Interactive mode (prompts before overwriting existing files).
* `cp -v src.txt dest/` — Verbose mode (shows files being copied).
* `cp -a dir1/ dir2/` — Archive mode (preserves file attributes, permissions, timestamps).

### Moving & Renaming (`mv`)
* `mv old_name.txt new_name.txt` — Rename a file or directory.
* `mv file.txt /target/dir/` — Move file to another directory.
* `mv -i src dest` — Prompt before overwriting.
* `mv -v src dest` — Verbose output.

### Deleting Files & Directories (`rm`)
* `rm file.txt` — Remove a file.
* `rm -r dir/` — Recursively remove directory and everything inside it.
* `rm -f file.txt` — Force remove without confirmation prompts or non-existent file warnings.
* `rm -rf dir/` — Force recursive deletion (⚠️ Use with extreme caution).
* `rm -i file.txt` — Interactive prompt for deletion confirmation.

---

## 5. Core Environment Variables

| Variable | Description |
| :--- | :--- |
| `$USER` | Current active user name |
| `$HOME` | Absolute path of user's home directory |
| `$SHELL` | Absolute path of default login shell |
| `$PWD` | Current working directory |
| `$OLDPWD` | Previous working directory before last `cd` |
| `$PATH` | Colon-delimited list of directories searched for executable binaries |
| `$?` | Exit status code of the last executed command (`0` = Success, `non-zero` = Error) |
| `$$` | Process ID (PID) of the current shell instance |

---

## 6. Essential Terminal Shortcuts (Readline / CLI)

* `Ctrl + A` — Jump cursor to the **beginning** of the line.
* `Ctrl + E` — Jump cursor to the **end** of the line.
* `Ctrl + U` — Cut/delete line from current cursor position to the **start**.
* `Ctrl + K` — Cut/delete line from current cursor position to the **end**.
* `Ctrl + W` — Delete the single word **preceding** the cursor.
* `Ctrl + L` — Clear screen (equivalent to `clear`).
* `Ctrl + C` — Send `SIGINT` signal to terminate/interrupt current running process.
* `Tab` — Command and path auto-completion.

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
