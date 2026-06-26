# Network Architecture

## Design Philosophy

The network is designed with **security-first segmentation** from the ground up. Three isolated VLANs ensure that a compromise in one zone does not affect others — a principle directly borrowed from enterprise SOC architecture.

---

## VLAN Design

| VLAN | ID | Subnet | Purpose | VPN | Access |
|------|----|--------|---------|-----|--------|
| SOC | 88 | 192.168.88.0/24 | Security lab — Ubuntu, VMs | WireGuard VPS | ↔ AI |
| AI | 20 | 192.168.20.0/24 | macOS — AI tools, study, LinkedIn | Mullvad | ↔ SOC |
| HOME | 10 | 192.168.10.0/24 | Personal devices | Mullvad | Fully isolated |

---

## Traffic Flow

```
┌─────────────────────────────────────────────────────────────┐
│                  MikroTik Chateau LTE6 ax                    │
│                     RouterOS 7.19.6                          │
│                                                              │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐    │
│  │  VLAN SOC    │   │   VLAN AI    │   │  VLAN HOME   │    │
│  │ .88.0/24     │   │  .20.0/24    │   │  .10.0/24    │    │
│  │              │   │              │   │              │    │
│  │ Ubuntu Host  │   │ macOS Host   │   │  Personal    │    │
│  │  ├ Splunk    │   │  ├ AI Tools  │   │  Devices     │    │
│  │  ├ Win11 VM  │   │  ├ CompTIA   │   │              │    │
│  │  └ Kali VM   │   │  └ LinkedIn  │   │              │    │
│  └──────┬───────┘   └──────┬───────┘   └──────┬───────┘    │
│         │    ↔ allowed ↔   │           blocked │            │
│         ▼                  ▼                   ▼            │
│    WireGuard VPS      Mullvad VPN         Mullvad VPN       │
└─────────────────────────────────────────────────────────────┘
         │                  │                   │
         ▼                  ▼                   ▼
      Internet           Internet            Internet
   (VPS IP exit)    (Mullvad IP exit)   (Mullvad IP exit)
```

---

## Security Design Decisions

### Why three VLANs?
- **Blast radius reduction** — if Kali VM is misconfigured and attacks escape, they hit only the SOC VLAN
- **Traffic isolation** — AI research traffic (ChatGPT, LinkedIn, GitHub) never mixes with attack simulation traffic
- **VPN separation** — SOC uses a personal VPS (full control, logging capability), AI/HOME use Mullvad (privacy-focused)

### Why WireGuard for SOC, Mullvad for AI/HOME?
- SOC VLAN uses a self-hosted VPS — full visibility into tunnel traffic, no third-party logging
- AI and HOME VLANs use Mullvad — zero-log provider, appropriate for personal/research traffic
- Different threat models require different solutions

### Why SOC ↔ AI communication is allowed?
- Allows monitoring of AI VLAN from SOC (future use case)
- macOS on AI VLAN can send files/screenshots to SOC via Telegram (operational workflow)
- HOME remains fully isolated — no exceptions

### Kill Switch implementation
- If the WireGuard tunnel drops, a firewall rule blocks all LAN traffic from exiting via `lte1`
- Prevents accidental de-anonymization or traffic leakage during tunnel reconnection

---

## Future Additions

- [ ] Add network diagram image (screenshot from Winbox)
- [ ] Document inter-VLAN firewall rule logic in detail
- [ ] Consider adding dedicated management VLAN
- [ ] Evaluate adding IDS/IPS (Zeek or Suricata) on SOC VLAN boundary
