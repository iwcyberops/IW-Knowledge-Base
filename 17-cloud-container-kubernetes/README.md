# ☁️ Month 17: Cloud Infrastructure, Identity Management & Container Security

> **Research Track:** Phase 02 — Offensive Security, Windows, RE & Exploitation  
> **Author & Lead Researcher:** Muhammad Imran (Founder, **IW Cyber Ops**)  
> **Knowledge Base Domain:** `IW-Knowledge-Base/m17-cloud-container-kubernetes`

---

## 🧭 Why Cloud IAM Topologies & Container Runtime Breakouts Matter

Modern production systems ab physical bare-metal par nahi, balki **Cloud Providers (AWS / Azure / GCP)** aur **Container Orchestration Platforms (Docker / Kubernetes)** ke upar microservices ban kar chalte hain.

Traditional perimeter security (firewalls) cloud me fail ho chuki hai; yahan **Identity & Access Management (IAM)** aur **Kernel Namespace Isolation** hi asal boundary hain. Ek elite offensive researcher ke liye ye samajhna zaroori hai ke:
1. **Cloud Identity & Role Mechanics:** IAM Policies, STS (Security Token Service) temporary credentials, role assumption, cross-account trusts, aur metadata services (**AWS IMDSv1 vs. IMDSv2** token headers).
2. **Cloud Privilege Escalation Paths:** S3/Blob storage policies abuse, IAM permission combination vectors (`iam:PassRole`, `lambda:CreateFunction`), aur serverless environment flaws.
3. **Container Internals (Linux Under the Hood):** Linux Namespaces (`PID`, `Mount`, `Net`, `IPC`, `UTS`, `User`), Control Groups (`cgroups v1/v2`), Docker daemon architecture, aur rootless container limits.
4. **Container Breakouts & Kubernetes Auditing:** Misconfigured Docker capabilities (`CAP_SYS_ADMIN`), exposed `docker.sock` mounts se host root access lena, aur Kubernetes Control Plane (`etcd`, API Server, Kubelet) me misconfigured **RBAC policies** ko weaponize karna.

Ye month humein modern cloud-native architectures ko audit aur dominate karne ka master banata hai.

---

## 📚 Month 17 Knowledge Base & Topic Notes Directory

Is folder me Month 17 ke dauran banaye gaye tamam technical notes topics ke mutabiq categorized hain:

| Note File | Core Focus & Concepts Covered | Status |
| :--- | :--- | :---: |
| 📄 **[`01-cloud-iam-policies-role-assumption.md`](./01-cloud-iam-policies-role-assumption.md)** | AWS/Azure IAM policies, STS temporary tokens, role assumption chains, cross-account trusts, and IMDSv1 vs IMDSv2. | 🟢 Completed |
| 📄 **[`02-cloud-attack-paths-serverless-s3.md`](./02-cloud-attack-paths-serverless-s3.md)** | S3/Blob storage misconfigurations, IAM privilege escalation vectors, Lambda execution contexts, and secret exfiltration. | 🟢 Completed |
| 📄 **[`03-container-internals-namespaces-cgroups.md`](./03-container-internals-namespaces-cgroups.md)** | Linux kernel namespaces (`clone`/`unshare`), `cgroups v1/v2` resource limits, Docker daemon model, and rootless setups. | 🟢 Completed |
| 📄 **[`04-container-breakout-privilege-escapes.md`](./04-container-breakout-privilege-escapes.md)** | Container breakout vectors: `CAP_SYS_ADMIN` abuse, mounted `/var/run/docker.sock` escape to host root, and `release_agent` tricks. | 🟢 Completed |
| 📄 **[`05-kubernetes-control-plane-rbac-audit.md`](./05-kubernetes-control-plane-rbac-audit.md)** | Kubernetes architecture (`API Server`, `etcd`, `Kubelet`), Service Account tokens, RBAC privilege escalation, and network policies. | 🟢 Completed |

---

## ⚙️ The 3 Daily Continuous Side Tracks (Month 17 Focus)

Hamare daily 12-hour engine ke 3 parallel tracks ke dedicated notes:

### 1. 💻 Systems C/C++ Track (1 Hour Daily)
* **Goal:** Direct manipulation of Linux kernel namespaces.
* **Topics:** Writing C programs using low-level kernel system calls `clone()` with flags (`CLONE_NEWPID`, `CLONE_NEWNS`, `CLONE_NEWNET`) and `unshare()` to build a minimal container runtime from scratch.

### 2. 🧩 Assembly & Reverse Engineering Track (1 Hour Daily)
* **Goal:** Reverse engineering Go-compiled binaries.
* **Topics:** Analyzing compiled Go binaries (standard in Docker/K8s/Cloud tools) in Ghidra; understanding Go runtime goroutines, interface tables (`itab`), and stripped Go symbol recovery.

### 3. 🔌 Hardware & Architecture Track (1 Hour Daily)
* **Goal:** Hardware-enforced cloud memory isolation.
* **Topics:** Multi-tenant cloud CPU isolation, AMD SEV (Secure Encrypted Virtualization), and Intel SGX (Software Guard Extensions) memory encryption enclaves.

---

## 🛠️ Lab Environments & Hands-On Milestones

* 🎯 **CloudGoat IAM Escalation Suite:** 5 complex multi-tier scenarios deployed and solved on CloudGoat, documenting chained IAM privilege escalation to full AWS account takeover.
* 🎯 **Container Breakout Demonstration Lab:** Controlled test environment proving container breakout to host machine root via mounted Docker sockets and abused Linux capabilities (`CAP_SYS_ADMIN`).
* 🎯 **Kubernetes Security Audit Report:** Complete multi-tenant Kubernetes cluster audit using `Peirates` and `Trivy`, identifying misconfigured RBAC roles, cluster role bindings, and privilege paths.

---

## 📖 Primary Learning References
* 🌐 *Hacking the Cloud* — Nick Frichette
* 📜 *Cloud Security Alliance (CSA) Security Guidance*
* 💻 *Kubernetes Official Security Documentation & Hardening Guides*
* 💻 *Rhino Security Labs CloudGoat Repository*

---

© **Muhammad Imran (Founder, IW Cyber Ops)** | Documented for Absolute Depth & Intellectual Rigor.
