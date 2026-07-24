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
| Ubuntu 22.04 (Host) | Splunk Enterprise 10.4.0 SIEM host | ✅ Done |
| Windows 11 (VM) | Log source — Sysmon v15.21 | ✅ Done |
| Kali Linux (VM) | Attack simulation platform | ✅ Done |
| WireGuard (VPS) | Encrypted tunnel for SOC VLAN | ✅ Done |
| Mullvad VPN | Encrypted tunnel for AI + HOME VLANs | ✅ Done |
| Splunk Enterprise 10.4.0 | SIEM — 23,067 events indexed | ✅ Done |
| Sysmon v15.21 | Endpoint telemetry on Windows VM | ✅ Done |
| Splunk Universal Forwarder | Log shipping Windows → Splunk | ✅ Done |

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
- [x] VM network isolation (Internal Network + Host-Only)

### ✅ Phase 2 — SIEM Pipeline
- [x] Splunk Enterprise 10.4.0 installed on Ubuntu
- [x] Sysmon v15.21 installed on Windows 11 (SwiftOnSecurity config)
- [x] Splunk Universal Forwarder configured
- [x] 23,067 events flowing end-to-end

### ✅ Phase 3 — Detection Engineering
- [x] First detection: SMB enumeration from Kali → Splunk alert
- [x] MITRE ATT&CK T1021.002 mapped
- [ ] Custom Splunk dashboards
- [ ] Automated alerts

### 📋 Phase 4 — Portfolio & Visibility
- [ ] 10+ documented lab scenarios on GitHub
- [ ] LinkedIn profile updated with lab
- [ ] SOC Analyst job applications

### ✅ Phase 5 — Splunk Universal Forwarder
- [x] Universal Forwarder installed on Windows 11
- [x] outputs.conf configured → 192.168.56.1:9997
- [x] inputs.conf configured (Sysmon, Security, System logs)
- [x] SplunkForwarder running as LocalSystem
- [x] 23,067 events indexed in Splunk

### ✅ Phase 6 — First Detection
- [x] SMB enumeration from Kali → EventCode 4625
- [x] KALI-ATTACKER identified in Splunk
- [x] MITRE ATT&CK T1021.002 mapped

### ✅ Phase 7 — Automated Detection
- [x] Splunk brute force alert built
- [x] Alert auto-triggering every minute (cron: * * * * *)
- [x] condition=1 confirmed in scheduler.log
- [x] MITRE ATT&CK T1110 mapped

### ✅ Phase 8 — Attack Simulations
- [x] Nmap aggressive scan → Sysmon EventCode 3 (T1046)
- [x] RDP brute force → 23 × EventCode 4625 (T1110)
- [x] Metasploit MS17-010 → EventCode 5379 (T1555)
- [x] Full kill chain detected in Splunk
- [x] 4 MITRE ATT&CK techniques mapped

### ⏳ Phase 9 — Active Directory Lab
- [ ] Windows Server 2022 installation
- [ ] Active Directory Domain Services
- [ ] Domain: soc.local
- [ ] Windows 11 joined to domain
- [ ] AD attack simulations

### ⏳ Phase 10 — Portfolio & Career
- [ ] CV finalized and published
- [ ] LinkedIn profile updated
- [ ] SOC Analyst job applications submitted

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
