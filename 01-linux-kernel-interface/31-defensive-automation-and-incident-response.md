<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 01 — Linux Kernel Interface & Core CLI
   DOCUMENT: Day 31 — Defensive Automation, FIM & Incident Response
   ========================================================================= -->

# 🛡️ Day 31: Defensive Automation, FIM & Incident Response

> **IW Cyber Ops Research Vault | Module 01: Linux & Systems Foundations**  
> *Author: Muhammad Imran (@iwcyberops)*  
> *Track: Blue Team Engineering, Active Defense, Cryptographic FIM & Triage Collection*

---

## 1. The Defensive Systems Automation Lifecycle

In Security Operations Centers (SOC) and Incident Response (IR) environments, manual triage is too slow to contain active adversaries. **Defensive Bash Engineering** bridges the gap by transforming administrative scripts into **Real-Time Detection**, **Automated Containment**, and **Volatile Evidence Preservation** engines.

```
       ┌────────────────────────────────────────────────────────┐
       │             DEFENSIVE AUTOMATION PIPELINE              │
       └──────────────────────────┬─────────────────────────────┘
                                  │
         ┌────────────────────────┼────────────────────────┐
         ▼                        ▼                        ▼
┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│  ACTIVE DEFENSE  │    │  INTEGRITY (FIM) │    │  FORENSIC TRIAGE │
├──────────────────┤    ├──────────────────┤    ├──────────────────┤
│ Real-time log    │    │ Cryptographic    │    │ Order of         │
│ streaming & auto │    │ baseline hashing │    │ volatility live  │
│ IP blacklisting  │    │ & delta alerts   │    │ evidence capture │
└──────────────────┘    └──────────────────┘    └──────────────────┘
```

---

## 2. Active Defense: Real-Time Log Watcher & Auto-Blacklist

This automated defense daemon continuously monitors authentication logs, tallies brute-force thresholds per IP address in an in-memory hashmap, and automatically injects firewall drop rules to neutralize attacks in real time.

```
  [ /var/log/auth.log ] ──( tail -Fn0 )──> [ Stream Parser: Regex Match IP ]
                                                      │
                                                      ▼
                                       [ Memory Hashmap: IP -> Attempts++ ]
                                                      │
                                    { Failed Attempts >= Max Threshold? }
                                                      │
                             ┌────────────────────────┴────────────────────────┐
                             ▼ (YES)                                           ▼ (NO)
                  [ Execute Containment ]                              [ Continue Monitoring ]
                  - iptables -A INPUT -s <IP> -j DROP
                  - Broadcast Alert Telemetry
```

---

### 🛡️ Production Script: Active SSH Brute-Force Defender

```bash
#!/usr/bin/env bash
# =========================================================================
# IW Cyber Ops — Real-Time Authentication Defense Daemon
# =========================================================================
set -Eeuo pipefail

# Mandatory root privileges check
[[ "$EUID" -eq 0 ]] || { echo "[-] FATAL: Must run as root to manage iptables." >&2; exit 1; }

AUTH_LOG="/var/log/auth.log"
THRESHOLD=5
declare -A FAILED_LOGINS

echo -e "\033[1;34m[*] Active Defense Daemon initialized on ${AUTH_LOG}...\033[0m"

# Stream newly appended log entries in real-time
# -F : Follow by name (survives log rotations)
# -n0: Do not read historical backlog; process only incoming live lines
tail -Fn0 "$AUTH_LOG" | while read -r LINE; do
    if [[ "$LINE" =~ Failed\ password\ for.*from\ ([0-9]+\.[0-9]+\.[0-9]+\.[0-9]+) ]]; then
        ATTACKER_IP="${BASH_REMATCH[1]}"
        
        # Increment memory counter for attacker IP
        (( FAILED_LOGINS["$ATTACKER_IP"] = ${FAILED_LOGINS["$ATTACKER_IP"]:-0} + 1 ))
        
        CURRENT_COUNT="${FAILED_LOGINS["$ATTACKER_IP"]}"
        echo -e "\033[1;33m[!] Auth Failure detected:\033[0m $ATTACKER_IP (Attempts: $CURRENT_COUNT/$THRESHOLD)"
        
        if (( CURRENT_COUNT >= THRESHOLD )); then
            echo -e "\033[1;31m[🚨] THRESHOLD BREACHED: Blacklisting $ATTACKER_IP via iptables!\033[0m"
            
            # Inject drop rule at top of firewall table
            iptables -I INPUT -s "$ATTACKER_IP" -j DROP
            
            # Reset counter to prevent duplicate rules
            unset "FAILED_LOGINS[$ATTACKER_IP]"
        fi
    fi
done
```

---

## 3. Cryptographic File Integrity Monitor (FIM)

Adversaries establish persistence by modifying configuration files, replacing system binaries, or dropping webshells into `/var/www/`. 

A **File Integrity Monitor (FIM)** generates a trusted cryptographic baseline of SHA-256 checksums and performs recurring differential audits to identify unauthorized modifications, creations, or deletions.

```
       [ INITIALIZATION PHASE ]                        [ VERIFICATION PHASE ]
  ┌─────────────────────────────────┐             ┌─────────────────────────────────┐
  │ Scan Target Paths (/bin, /etc)  │             │ Re-hash Current Live Filesystem │
  └────────────────┬────────────────┘             └────────────────┬────────────────┘
                   │                                               │
                   ▼                                               ▼
  ┌─────────────────────────────────┐             ┌─────────────────────────────────┐
  │ baseline.sha256 (Trusted State) │ ◄──( Diff )───│ current.sha256 (Live State)   │
  └─────────────────────────────────┘             └─────────────────────────────────┘
                                                                   │
                                                                   ▼
                                                  [ Report Tampered / Injected Files ]
```

---

### 🛡️ Production Script: Native Bash FIM Engine

```bash
#!/usr/bin/env bash
# =========================================================================
# IW Cyber Ops — Cryptographic File Integrity Engine (FIM)
# =========================================================================
set -Eeuo pipefail

MONITOR_DIR="/etc"
BASELINE_FILE="/var/log/fim_baseline.sha256"
CURRENT_SCAN="/tmp/fim_current.sha256"

# 1. Baseline Generation Mode
generate_baseline() {
    echo -e "\033[1;34m[*] Generating cryptographic baseline for ${MONITOR_DIR}...\033[0m"
    find "$MONITOR_DIR" -type f -exec sha256sum {} + 2>/dev/null | sort > "$BASELINE_FILE"
    chmod 600 "$BASELINE_FILE"
    echo -e "\033[1;32m[+] Baseline saved securely to ${BASELINE_FILE} ($(wc -l < "$BASELINE_FILE") files indexed).\033[0m"
}

# 2. Integrity Verification Mode
check_integrity() {
    [[ -f "$BASELINE_FILE" ]] || { echo "[-] Baseline missing. Run with --init first." >&2; exit 1; }
    
    echo -e "\033[1;34m[*] Auditing live integrity against baseline...\033[0m"
    find "$MONITOR_DIR" -type f -exec sha256sum {} + 2>/dev/null | sort > "$CURRENT_SCAN"
    
    # Differential Analysis
    CHANGES=$(diff -u "$BASELINE_FILE" "$CURRENT_SCAN" || true)
    
    if [[ -z "$CHANGES" ]]; then
        echo -e "\033[1;32m[+] INTEGRITY VERIFIED: Zero unauthorized modifications detected.\033[0m"
    else
        echo -e "\033[1;31m[🚨] CRITICAL: Filesystem tampering detected in ${MONITOR_DIR}!\033[0m"
        echo "$CHANGES" | grep -E "^(\+|-)[a-f0-9]" || true
    fi
    rm -f "$CURRENT_SCAN"
}

# Command Switcher
case "${1:-}" in
    --init)  generate_baseline ;;
    --check) check_integrity ;;
    *) echo "Usage: $0 {--init|--check}"; exit 1 ;;
esac
```

---

## 4. Live Incident Response (IR) Volatile Evidence Collector

When a system is actively compromised, responders must adhere to **RFC 3227 (Guidelines for Evidence Collection)**: capture the most volatile artifacts first before powering down or rebooting.

```
                    ┌──────────────────────────────────────────────┐
                    │       ORDER OF VOLATILITY (RFC 3227)         │
                    ├──────────────────────────────────────────────┤
                    │ 1. In-Memory Network Sockets (ss, lsof)      │ (Most Volatile)
                    │ 2. Active Processes & Open FDs (/proc, ps)   │
                    │ 3. Kernel Ring Buffer & Dmesg (RAM Cache)    │
                    │ 4. Logged-in Users & Session States (who, w) │
                    │ 5. System Persistence (Cron, Systemd, SUID)  │ (Least Volatile)
                    └──────────────────────────────────────────────┘
```

---

### 🛡️ Production Script: Automated Tactical Triage Collector

```bash
#!/usr/bin/env bash
# =========================================================================
# IW Cyber Ops — Fast Incident Response Triage Collector
# =========================================================================
set -Eeuo pipefail

# Create secure timestamped forensic bundle directory
CASE_ID="IR_$(date +%Y%m%d_%H%M%S)"
STAGING_DIR="/tmp/${CASE_ID}"
mkdir -m 700 "$STAGING_DIR"

log_triage() { echo -e "\033[1;32m[+] Capturing:\033[0m $1"; }

echo -e "\033[1;34m[*] INITIATING LIVE INCIDENT RESPONSE TRIAGE: ${CASE_ID}\033[0m"

# 1. Volatile Network State
log_triage "Active network sockets and listening ports..."
ss -plantu > "${STAGING_DIR}/network_sockets.txt"
ip route > "${STAGING_DIR}/routing_table.txt"
ip neigh > "${STAGING_DIR}/arp_cache.txt"

# 2. Process Tree & Memory Artifacts
log_triage "Process hierarchy and execution arguments..."
ps auxwwf > "${STAGING_DIR}/process_tree.txt"
lsof -nP -i > "${STAGING_DIR}/open_network_files.txt" 2>/dev/null || true

# 3. User Sessions & History
log_triage "Logged-in users and authentication states..."
w > "${STAGING_DIR}/logged_in_users.txt"
last -a -n 50 > "${STAGING_DIR}/login_history.txt"

# 4. Persistence Hooks
log_triage "Cron schedules and active systemd services..."
ls -la /etc/cron* /etc/crontab /var/spool/cron/crontabs > "${STAGING_DIR}/cron_persistence.txt" 2>/dev/null || true
systemctl list-unit-files --state=enabled > "${STAGING_DIR}/enabled_services.txt"

# 5. SUID / SGID Binaries
log_triage "Scanning for SUID/SGID privileged binaries..."
find / -xdev \( -perm -4000 -o -perm -2000 \) -type f 2>/dev/null > "${STAGING_DIR}/suid_binaries.txt" || true

# Package Evidence Bundle
BUNDLE="/root/${CASE_ID}_evidence.tar.gz"
tar -czf "$BUNDLE" -C "/tmp" "$CASE_ID"
rm -rf "$STAGING_DIR"

# Generate Forensic Cryptographic Hash of the Evidence Bundle
sha256sum "$BUNDLE" > "${BUNDLE}.sha256"

echo -e "\033[1;32m[+] TRIAGE COMPLETE: Evidence packaged securely at ${BUNDLE}\033[0m"
echo -e "[+] Evidence SHA-256: $(cat "${BUNDLE}.sha256")"
```

---

## 5. Threat Operations & Defensive Hardening Matrix

| Tactical Objective | Implementation Mechanism | Defensive / IR Focus |
| :--- | :--- | :--- |
| **Real-Time Stream Parsing** | `tail -Fn0 /var/log/auth.log` | Zero-lag detection without polling overhead |
| **Automated IP Blocking** | `iptables -I INPUT -s <IP> -j DROP` | Instant containment of network brute-force |
| **Cryptographic Baselines** | `find <path> -type f -exec sha256sum {} +` | Tamper detection across system binaries |
| **Volatile State Dumps** | `ss -plantu` & `ps auxwwf` | Capturing C2 connections before termination |
| **Forensic Evidence Chain** | `tar -czf` + `sha256sum` | Preserving integrity for legal/IR standards |

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
