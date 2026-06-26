# 🛡️ SOC Home Lab

> A hands-on Security Operations Center (SOC) home lab built from scratch to develop real-world detection and response skills — documented for learning, transparency, and professional portfolio purposes.

---

## 🎯 Goal

Build a fully functional home SOC lab to:
- Develop hands-on SIEM, log analysis, and threat detection skills
- Simulate real attacker techniques and detect them using Splunk
- Document every step publicly as a professional portfolio for SOC Analyst roles
- Target: **SOC Analyst job offer by December 2025**

---

## 🏗️ Lab Architecture

```
MikroTik Chateau LTE6 ax (RouterOS 7.19.6)
│
├── VLAN SOC  [192.168.88.0/24] ──→ WireGuard VPS ──→ Internet
│   ├── Ubuntu Host (main)
│   │   └── Splunk Free (SIEM)
│   ├── VM: Windows 11 (VirtualBox)
│   │   └── Sysmon + Splunk Universal Forwarder
│   └── VM: Kali Linux (VirtualBox)
│       └── Attack simulation
│
├── VLAN AI   [192.168.20.0/24] ──→ Mullvad VPN ──→ Internet
│   └── macOS Host
│       └── AI tools, CompTIA prep, LinkedIn
│
└── VLAN HOME [192.168.10.0/24] ──→ Mullvad VPN ──→ Internet
    └── Personal devices (fully isolated)
```

---

## 🧰 Stack

| Component | Role | Status |
|-----------|------|--------|
| MikroTik Chateau LTE6 ax | Router, VLAN segmentation, VPN routing | ✅ Done |
| Ubuntu 22.04 (Host) | Splunk Free SIEM | 🔧 In Progress |
| Windows 11 (VM) | Log source — Sysmon | 🔧 In Progress |
| Kali Linux (VM) | Attack simulation | 🔧 In Progress |
| Splunk Free | SIEM — log ingestion, detection, dashboards | 🔧 In Progress |
| Sysmon | Endpoint telemetry on Windows VM | 🔧 In Progress |
| WireGuard (VPS) | Encrypted tunnel for SOC VLAN | ✅ Done |
| Mullvad VPN | Encrypted tunnel for AI + HOME VLANs | ✅ Done |

---

## 📁 Repository Structure

```
soc-home-lab/
│
├── README.md                        ← You are here
│
├── infrastructure/
│   ├── mikrotik/
│   │   ├── wireguard-vps-setup.md   ← WireGuard tunnel: MikroTik ↔ VPS
│   │   └── mullvad-vlan-setup.md    ← VLAN segmentation + Mullvad VPN
│   └── network-diagram/
│       └── architecture.md          ← Network diagram and design decisions
│
└── labs/
    └── (coming soon — detection scenarios)
```

---

## 🗺️ Roadmap

### ✅ Phase 1 — Network Infrastructure
- [x] MikroTik VLAN segmentation (SOC / AI / HOME)
- [x] WireGuard VPN tunnel (SOC VLAN → VPS)
- [x] Mullvad VPN (AI + HOME VLANs)
- [x] Network isolation via firewall rules
- [ ] VM network isolation (Internal Network + Host-Only)

### 🔧 Phase 2 — SIEM Pipeline
- [ ] Splunk Free installed on Ubuntu
- [ ] Sysmon installed on Windows 11 VM (SwiftOnSecurity config)
- [ ] Splunk Universal Forwarder configured
- [ ] First logs flowing end-to-end

### 📋 Phase 3 — Detection Engineering
- [ ] First detection: Kali nmap scan → Splunk alert
- [ ] MITRE ATT&CK mapping for each scenario
- [ ] Custom Splunk dashboards

### 📢 Phase 4 — Portfolio & Visibility
- [ ] 10+ documented lab scenarios on GitHub
- [ ] LinkedIn profile updated with lab
- [ ] SOC Analyst job applications

---

## 🔐 Security Note

All IP addresses in this documentation have been generalized for public sharing.
Sensitive credentials, private keys, and backup files are **never** stored in this repository.

---

## 👤 About

Self-taught cybersecurity practitioner building toward a SOC Analyst role.
Currently studying for CompTIA Security+.
This lab is my primary hands-on training environment.

> *"The best way to learn security is to break things in a controlled environment — and document everything."*
