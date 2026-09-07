<!-- 
SEO METADATA & KEYWORDS (Invisible to readers, visible to Google Crawlers)
Keywords: IW Cyber Ops, Muhammad Imran, Cloud Security, AWS IAM Exploitation, Kubernetes Security, Container Breakout, Linux Namespaces, Cgroups, Docker Architecture, IMDSv2 SSRF, RBAC Auditing, AMD SEV Intel SGX, Cybersecurity Knowledge Base.
-->

# ☁️ Month 17: Cloud Infrastructure, Identity Management & Container Security

> **Knowledge Base Directory:** Phase 02 / Month 17  
> **System Operator & Author:** Muhammad Imran (Founder, **IW Cyber Ops**)  
> **Objective:** Understand cloud identity topologies, evaluate IAM boundaries across AWS/Azure/GCP, and master container runtime security.

---

## 🏛️ The Imperative of Cloud & Container Security

Enterprise infrastructure has fundamentally shifted from monolithic on-premise servers to elastic cloud topologies and containerized microservices.

In modern target environments, identity is the true perimeter. An adversary who gains a foothold does not simply scan IP subnets; they query Instance Metadata Services (IMDS), steal Security Token Service (STS) credentials, and traverse complex cross-account Identity and Access Management (IAM) trust policies. Understanding the mathematical boundaries of cloud permissions is mandatory for offensive dominance.

Simultaneously, containers are not lightweight virtual machines—they are isolated Linux processes governed entirely by kernel primitives: **Namespaces** and **Control Groups (cgroups)**. When these boundaries are misconfigured, or when dangerous Linux capabilities (`CAP_SYS_ADMIN`) and Docker socket mounts are exposed, an elite researcher can break out of container sandboxes directly to host root.

This directory serves as the **IW Cyber Ops Knowledge Base** for Month 17. It documents the systematic exploitation of cloud identity mechanisms, container breakouts, Kubernetes RBAC misconfigurations, and hardware-enforced cloud isolation.

---

## 🧠 Core Domains Documented in this Directory

The notes contained within this module cover the following theoretical and practical pillars:

1. **Cloud Identity & IAM Topology:** Dissecting AWS/Azure/GCP IAM policies, role assumption lifecycles, STS session tokens, cross-account trusts, and navigating IMDSv1 vs. IMDSv2 token session requirements.
2. **Cloud Attack Paths:** Auditing cloud storage misconfigurations (S3 bucket ACLs, Azure Blobs), executing IAM privilege escalation vectors, finding exposed secrets in serverless architectures, and exploiting CloudGoat scenarios.
3. **Container Internals & Isolation Primitives:** Deep-dive into Linux kernel namespaces (PID, Mount, Net, IPC, UTS, User), Control Groups (cgroups v1/v2), Docker daemon architecture, and rootless container engines.
4. **Container Breakout Mechanics:** Weaponizing container escape primitives: abusing exposed `docker.sock`, exploiting overly permissive Linux capabilities (`CAP_SYS_ADMIN`, `CAP_NET_ADMIN`), and mounting host filesystems.
5. **Kubernetes Cluster Security:** Mapping control plane components (API Server, `etcd`, Kubelet), analyzing Role-Based Access Control (RBAC) misconfigurations, abusing privileged Service Accounts, and auditing pod security standards.
6. **Low-Level Isolation & Hardware Enclaves:** Interacting directly with Linux namespaces via C syscalls (`clone()`, `unshare()`), reverse engineering Go-compiled cloud agents, and studying hardware-enforced memory encryption (AMD SEV, Intel SGX).

---

## 📂 Index of Technical Notes

*Below is the living index of all Markdown notes generated during this month's research. Click on any topic to access the detailed documentation.*

| Status | Technical Topic | File Reference |
| :---: | :--- | :--- |
| 📝 | Cloud Identity: AWS IAM Policies, STS & IMDSv2 | `[01-cloud-iam-sts-imdsv2.md](./01-cloud-iam-sts-imdsv2.md)` |
| 📝 | Cloud Attack Paths & IAM Privilege Escalation | `[02-cloud-attack-paths-iam-privesc.md](./02-cloud-attack-paths-iam-privesc.md)` |
| 📝 | Container Internals: Linux Namespaces & Cgroups | `[03-container-namespaces-cgroups.md](./03-container-namespaces-cgroups.md)` |
| 📝 | Container Breakout Primitives & Capability Abuse | `[04-container-breakout-capabilities.md](./04-container-breakout-capabilities.md)` |
| 📝 | Kubernetes Security: Control Plane, RBAC & `etcd` | `[05-kubernetes-rbac-cluster-security.md](./05-kubernetes-rbac-cluster-security.md)` |
| 📝 | Hardware Memory Encryption: AMD SEV & Intel SGX | `[06-cloud-hardware-isolation-sev-sgx.md](./06-cloud-hardware-isolation-sev-sgx.md)` |

*(Note: As the month progresses, new `.md` files will be added to this folder and linked above.)*

---

## 🛡️ About the Author

**Muhammad Imran** is an independent systems researcher and the Founder of **IW Cyber Ops**. This knowledge base is an active repository complementing a rigorous 42-month journey engineered for absolute depth, intellectual rigor, and high-impact vulnerability research.

To view the complete overarching roadmap, visit the official [IW-Mission-Control](https://github.com/iwcyberops/IW-Mission-Control) repository.

<br>

---
*Generated & Curated by **IW Cyber Ops** | High-Assurance Cyber Operations & Research*
