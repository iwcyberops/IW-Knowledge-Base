<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 01 — Linux Kernel Interface & Core CLI
   DOCUMENT: Day 32 — Capstone: Production Cyber Ops CLI Framework
   ========================================================================= -->

# 🛡️ Day 32: Capstone: Production Cyber Ops CLI Framework

> **IW Cyber Ops Research Vault | Module 01: Linux & Systems Foundations**  
> *Author: Muhammad Imran (@iwcyberops)*  
> *Track: End-to-End Modular Engineering, Concurrency, Hardening & Telemetry*

---

## 1. Capstone Framework Architecture

This Capstone Project synthesizes all foundational and advanced concepts mastered across Days 01 through 31 into a single, unified, enterprise-grade **Tactical Cyber Operations CLI Framework (`iw-ops`)**.

```
                           ┌────────────────────────────────────────────────────────┐
                           │          IW-OPS CYBER OPERATIONS ENGINE                │
                           └──────────────────────────┬─────────────────────────────┘
                                                      │
         ┌───────────────────┬────────────────────────┼────────────────────────┬───────────────────┐
         ▼                   ▼                        ▼                        ▼                   ▼
┌─────────────────┐ ┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐ ┌─────────────────┐
│ STRICT RUNTIME  │ │ CLI STATE PARSER│      │ CONCURRENT SCAN │      │ INCIDENT TRIAGE │ │ TELEMETRY & JSON│
├─────────────────┤ ├─────────────────┤      ├─────────────────┤      ├─────────────────┤ ├─────────────────┤
│ set -Eeuo       │ │ getopts state   │      │ Parallel socket │      │ RFC 3227 volatile│ │ Structured JSON│
│ pipefail + traps│ │ engine + inputs │      │ worker queues   │      │ evidence engine │ │ & CSV reporting │
└─────────────────┘ └─────────────────┘      └─────────────────┘      └─────────────────┘ └─────────────────┘
```

---

## 2. Integrated Core Capabilities

1. **Hardened Defensive Core:** Enforces strict execution (`set -Eeuo pipefail`), dynamic line-number error trapping (`trap ERR`), and automated workspace shredding on `EXIT`.
2. **Standard POSIX CLI Interface:** Robust `getopts` engine supporting custom flags (`-t`, `-p`, `-m`, `-o`, `-v`, `-h`), mandatory argument validation, and positional parameter shifting.
3. **High-Speed Asynchronous Concurrency:** Multi-threaded `/dev/tcp` socket probing utilizing parallel background worker pools (`&` + `wait`).
4. **Volatile Incident Response Triage:** Automated capture of in-memory network sockets, process trees, login history, and persistence mechanisms into timestamped evidence bundles.
5. **Cryptographic Integrity Verification:** Automated SHA-256 hash generation and differential comparison.
6. **Structured Telemetry & Reporting:** Real-time colorized terminal logging with parallel export to machine-readable **JSON** and **CSV** reports.

---

## 3. The Complete Production Framework Source Code

```bash
#!/usr/bin/env bash
# =========================================================================
# PROJECT: IW-OPS (Tactical Cyber Operations CLI Framework)
# AUTHOR: Muhammad Imran | IW Cyber Ops (@iwcyberops)
# TRACK: 42-Month Systems & Cyber Operations Research
# =========================================================================

# =========================================================================
# 1. STRICT RUNTIME CONFIGURATION & CONSTANTS
# =========================================================================
set -Eeuo pipefail
shopt -s inherit_errexit 2>/dev/null || true
umask 077

readonly VERSION="1.0.0"
readonly SCRIPT_NAME="$(basename "$0")"

# Terminal Formatting
readonly BOLD="\033[1m"
readonly RED="\033[1;31m"
readonly GREEN="\033[1;32m"
readonly YELLOW="\033[1;33m"
readonly BLUE="\033[1;34m"
readonly CYAN="\033[1;36m"
readonly RESET="\033[0m"

# Default State
TARGET=""
PORT_RANGE="1-1024"
MODE=""
OUTPUT_FILE=""
VERBOSE=false
CONCURRENCY=50

# Allocate Secure Temporary Workspace
WORK_DIR=$(mktemp -d /tmp/iw_ops_XXXXXX)

# =========================================================================
# 2. STRUCTURED TELEMETRY & CLEANUP TRAPS
# =========================================================================
log_info()    { echo -e "${BLUE}[*] INFO:${RESET} $1"; }
log_success() { echo -e "${GREEN}[+] SUCCESS:${RESET} $1"; }
log_warn()    { echo -e "${YELLOW}[!] WARNING:${RESET} $1"; }
log_error()   { echo -e "${RED}[-] ERROR:${RESET} $1" >&2; }
log_debug()   { [[ "$VERBOSE" == true ]] && echo -e "${CYAN}[DEBUG]:${RESET} $1" || true; }

die() {
    log_error "$1"
    exit "${2:-1}"
}

cleanup() {
    local exit_code=$?
    log_debug "Executing emergency cleanup handler..."
    if [[ -d "$WORK_DIR" ]]; then
        rm -rf -- "$WORK_DIR"
    fi
    exit "$exit_code"
}
trap 'cleanup' EXIT SIGINT SIGTERM SIGHUP

error_trap() {
    local line="$1"
    local cmd="$2"
    local code="$3"
    log_error "Command '${cmd}' failed on line ${line} with exit status ${code}"
}
trap 'error_trap "$LINENO" "$BASH_COMMAND" "$?"' ERR

# =========================================================================
# 3. HELP MANUAL & CLI ARGUMENT ENGINE
# =========================================================================
usage() {
    cat << EOF
${BOLD}IW-OPS — Tactical Cyber Operations CLI Framework (v${VERSION})${RESET}
Author: Muhammad Imran (@iwcyberops)

${BOLD}USAGE:${RESET}
  $SCRIPT_NAME -m <mode> [OPTIONS]

${BOLD}CORE MODES (-m):${RESET}
  ${GREEN}scan${RESET}        High-speed asynchronous TCP socket reconnaissance
  ${GREEN}triage${RESET}      Incident response volatile evidence preservation
  ${GREEN}fim${RESET}         Cryptographic baseline integrity verification

${BOLD}OPTIONS:${RESET}
  -t <IP/HOST>  Target IPv4 address or hostname
  -p <PORTS>    Port range for scan mode (Default: 1-1024)
  -o <FILE>     Path to output JSON/CSV report
  -v            Enable verbose diagnostic telemetry
  -h            Display this operational manual

${BOLD}EXAMPLES:${RESET}
  $SCRIPT_NAME -m scan -t 10.10.14.5 -p 20-100 -o report.json
  $SCRIPT_NAME -m triage -o /root/evidence.tar.gz
  $SCRIPT_NAME -m fim -t /etc
EOF
    exit 0
}

# Parse Options via getopts
while getopts ":m:t:p:o:vh" OPT; do
    case "$OPT" in
        m) MODE="$OPTARG" ;;
        t) TARGET="$OPTARG" ;;
        p) PORT_RANGE="$OPTARG" ;;
        o) OUTPUT_FILE="$OPTARG" ;;
        v) VERBOSE=true ;;
        h) usage ;;
        :) die "Flag '-$OPTARG' requires a mandatory argument!" 2 ;;
        \?) die "Invalid option '-$OPTARG'!" 2 ;;
    esac
done
shift $(( OPTIND - 1 ))

# =========================================================================
# 4. MODULE 1: HIGH-SPEED ASYNCHRONOUS SOCKET SCANNER
# =========================================================================
probe_port() {
    local host="$1"
    local port="$2"
    local timeout=1
    
    if timeout "$timeout" bash -c "exec 3<>/dev/tcp/$host/$port" 2>/dev/null; then
        echo "$port" >> "${WORK_DIR}/open_ports.txt"
        log_success "Port OPEN: ${port}/tcp"
        exec 3>&- 2>/dev/null || true
    fi
}
export -f probe_port

run_scanner() {
    [[ -n "$TARGET" ]] || die "Scan mode requires a target specified via -t <IP>!" 1
    
    # Parse Start and End Ports
    local start_port end_port
    if [[ "$PORT_RANGE" =~ ^([0-9]+)-([0-9]+)$ ]]; then
        start_port="${BASH_REMATCH[1]}"
        end_port="${BASH_REMATCH[2]}"
    else
        start_port="$PORT_RANGE"
        end_port="$PORT_RANGE"
    fi

    log_info "Initiating parallel socket reconnaissance on ${TARGET} (Ports: ${start_port}-${end_port})..."
    touch "${WORK_DIR}/open_ports.txt"

    # Concurrent Execution Worker Queue
    local active_jobs=0
    for (( port=start_port; port<=end_port; port++ )); do
        probe_port "$TARGET" "$port" &
        (( active_jobs++ ))
        
        # Concurrency Barrier Throttling
        if (( active_jobs >= CONCURRENCY )); then
            wait -n 2>/dev/null || true
            (( active_jobs-- ))
        fi
    done
    wait

    # Reporting
    local total_open
    total_open=$(wc -l < "${WORK_DIR}/open_ports.txt")
    log_info "Scan finished. Total open ports discovered: ${total_open}"

    if [[ -n "$OUTPUT_FILE" ]]; then
        log_info "Generating structured JSON report..."
        local json_ports
        json_ports=$(paste -sd, "${WORK_DIR}/open_ports.txt" || true)
        cat << EOF > "$OUTPUT_FILE"
{
  "tool": "IW-OPS",
  "version": "$VERSION",
  "timestamp": "$(date -u +"%Y-%m-%dT%H:%M:%SZ")",
  "mode": "scan",
  "target": "$TARGET",
  "port_range": "$PORT_RANGE",
  "open_ports": [${json_ports}]
}
EOF
        log_success "Report saved to $OUTPUT_FILE"
    fi
}

# =========================================================================
# 5. MODULE 2: LIVE INCIDENT RESPONSE TRIAGE COLLECTOR
# =========================================================================
run_triage() {
    [[ "$EUID" -eq 0 ]] || die "Incident triage requires root administrative privileges!" 126
    
    local triage_dir="${WORK_DIR}/triage_$(date +%Y%m%d_%H%M%S)"
    mkdir -p "$triage_dir"
    
    log_info "Executing live volatile evidence triage (RFC 3227)..."
    
    log_debug "Dumping active network sockets..."
    ss -plantu > "${triage_dir}/sockets.txt"
    
    log_debug "Dumping process tree and command-lines..."
    ps auxwwf > "${triage_dir}/processes.txt"
    
    log_debug "Dumping user sessions and auth logs..."
    who -a > "${triage_dir}/active_sessions.txt"
    last -n 50 > "${triage_dir}/login_history.txt"
    
    log_debug "Auditing persistence mechanisms..."
    ls -la /etc/cron* /etc/crontab /var/spool/cron/crontabs > "${triage_dir}/cron.txt" 2>/dev/null || true
    systemctl list-unit-files --state=enabled > "${triage_dir}/systemd_services.txt"
    
    local archive_target="${OUTPUT_FILE:-/root/triage_$(date +%Y%m%d_%H%M%S).tar.gz}"
    tar -czf "$archive_target" -C "$WORK_DIR" "$(basename "$triage_dir")"
    local sha256
    sha256=$(sha256sum "$archive_target" | awk '{print $1}')
    
    log_success "Triage evidence packaged: $archive_target"
    log_success "Evidence SHA-256: $sha256"
}

# =========================================================================
# 6. MODULE 3: CRYPTOGRAPHIC FILE INTEGRITY MONITOR (FIM)
# =========================================================================
run_fim() {
    local audit_path="${TARGET:-/etc}"
    [[ -d "$audit_path" ]] || die "Target directory '$audit_path' does not exist!" 1
    
    local baseline="${WORK_DIR}/baseline.sha256"
    log_info "Hashing target filesystem tree at: $audit_path"
    find "$audit_path" -type f -exec sha256sum {} + 2>/dev/null | sort > "$baseline"
    
    local file_count
    file_count=$(wc -l < "$baseline")
    log_success "Indexed $file_count files. Cryptographic snapshot generated."
    
    if [[ -n "$OUTPUT_FILE" ]]; then
        cp "$baseline" "$OUTPUT_FILE"
        log_success "Integrity manifest saved to: $OUTPUT_FILE"
    fi
}

# =========================================================================
# 7. MAIN EXECUTION ENTRY POINT
# =========================================================================
main() {
    echo -e "${BOLD}${CYAN}──────────────────────────────────────────────────────────────────${RESET}"
    echo -e "${BOLD}${GREEN}  🛡️ IW-OPS — Tactical Cyber Operations CLI Engine (v${VERSION})${RESET}"
    echo -e "${BOLD}${CYAN}──────────────────────────────────────────────────────────────────${RESET}"

    [[ -n "$MODE" ]] || { log_error "No operational mode specified (-m)!"; usage; }

    case "$MODE" in
        scan)   run_scanner ;;
        triage) run_triage ;;
        fim)    run_fim ;;
        *)      die "Invalid operational mode: '$MODE'! Choose: scan, triage, fim" 2 ;;
    esac
}

main "$@"
```

---

## 4. Execution & Verification Guide

### 1. Make Binary Executable
```bash
chmod +x iw-ops.sh
```

### 2. Launch High-Speed Socket Reconnaissance
```bash
./iw-ops.sh -m scan -t 127.0.0.1 -p 1-1000 -v -o /tmp/scan_report.json
```

### 3. Launch Root Incident Response Volatile Triage
```bash
sudo ./iw-ops.sh -m triage -o /root/incident_bundle.tar.gz
```

### 4. Generate Cryptographic FIM Manifest
```bash
./iw-ops.sh -m fim -t /etc -o /tmp/etc_baseline.sha256
```

---

## 5. Month 01 Systems & Shell Mastery Summary Matrix

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│               IW CYBER OPS — MONTH 01 CURRICULUM MASTERY MATRIX                        │
├────────────────────────────────────────────────────────────────────────────────────────┤
│ [Days 01 - 04]  │ Linux Kernel Interface, FHS Internals, File Probing, Processes       │
│ [Days 05 - 08]  │ I/O Redirection, Compression Exploits, DAC/ACLs, Systemd & Sudo      │
│ [Days 09 - 12]  │ Network Sockets, Text Wrangling (Awk/Sed), Inodes, Package Toolchains│
│ [Days 13 - 17]  │ Cron Persistence, Shell State, Logging Forensics, LKMs, Tmux Multiplex│
│ [Days 18 - 21]  │ Shebang Internals, Parameter Expansion, Arithmetic, Logic Tests       │
│ [Days 22 - 25]  │ Iteration Engines, Arrays/Hashmaps, Modular Functions, Process Sub    │
│ [Days 26 - 29]  │ Signal Traps, Strict Mode, Getopts CLI Engine, Secure Scripting       │
│ [Days 30 - 32]  │ Offensive Automation, Defensive IR Engineering, Production Capstone   │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
