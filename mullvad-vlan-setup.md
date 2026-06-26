# MikroTik: Network Segmentation with Mullvad VPN

**Hardware:** MikroTik Chateau LTE6 ax (RouterOS v7.19.6)
**VPN Provider:** Mullvad
**Protocol:** WireGuard

---

## Architecture

```
[home 192.168.10.0/24] → WiFi SSID: home → Mullvad VPN (wg-al1) → fully isolated
[ai   192.168.20.0/24] → WiFi SSID: ai   → Mullvad VPN (wg-ai)  → accessible from soc
[soc  192.168.88.0/24] → main network    → VPS WireGuard         → accessible to ai
```

**Design decisions:**
- `home` VLAN is completely isolated — no access to/from any other VLAN
- `soc` ↔ `ai` communication allowed for SOC monitoring purposes
- Each VLAN routes through a separate VPN tunnel for traffic isolation

---

## Step 1: Get Mullvad Config

1. Go to [mullvad.net/account/wireguard-config](https://mullvad.net/account/wireguard-config)
2. Click **Generate key**
3. Select platform **Linux**, choose country and server
4. Download `.conf` file

Config structure:
```ini
[Interface]
PrivateKey = <device private key>
Address = 10.X.X.X/32
DNS = 100.64.X.X

[Peer]
PublicKey = <Mullvad server public key>
AllowedIPs = 0.0.0.0/0
Endpoint = X.X.X.X:51820
```

> ⚠️ Each VLAN (home, ai) requires a **separate config** — generate as a new device on the Mullvad website.

---

## Step 2: WireGuard Interface for Home Network

### 2.1 Create interface

```routeros
/interface wireguard add name=wg-al1 listen-port=13233 private-key="<PRIVATE_KEY_FROM_CONFIG>" comment="Mullvad home"
```

### 2.2 IP address on interface

```routeros
/ip address add address=<ADDRESS_FROM_CONFIG> interface=wg-al1
```

### 2.3 Add peer (Mullvad server)

```routeros
/interface wireguard peers add interface=wg-al1 public-key="<PUBLICKEY_FROM_CONFIG>" endpoint-address=<ENDPOINT_IP> endpoint-port=51820 allowed-address=0.0.0.0/0 persistent-keepalive=25 comment="Mullvad server 1"
```

### 2.4 Direct route to endpoint

> ⚠️ This is critical. Without this route, packets to the Mullvad server will try to go through the tunnel itself — causing a loop and preventing the handshake.

```routeros
/ip route add dst-address=<ENDPOINT_IP>/32 gateway=lte1 comment="Direct route to Mullvad"
```

If `lte1` does not accept the route, find the provider gateway first:
```routeros
/ip route print detail
# Find the gateway for the lte1 route, e.g. 10.X.X.X
/ip route add dst-address=<ENDPOINT_IP>/32 gateway=<LTE_GATEWAY> comment="Direct route to Mullvad"
```

### 2.5 Verify handshake

```routeros
/interface wireguard peers print detail
# Should show last-handshake and growing rx
```

---

## Step 3: Policy Routing for Home Network

```routeros
# Create routing table
/routing table add name=mullvad fib

# Default route through Mullvad
/ip route add dst-address=0.0.0.0/0 gateway=wg-al1 routing-table=mullvad

# Rule — home traffic goes through mullvad table
/routing rule add src-address=192.168.10.0/24 action=lookup table=mullvad comment="Home network via Mullvad"

# NAT
/ip firewall nat add chain=srcnat out-interface=wg-al1 action=masquerade comment="NAT via Mullvad"
```

---

## Step 4: Multiple Mullvad Servers (Optional)

One private key on the interface — multiple peers with different servers:

```routeros
/interface wireguard peers add interface=wg-al1 public-key="<SERVER_2_KEY>" endpoint-address=<SERVER_2_IP> endpoint-port=51820 allowed-address=0.0.0.0/0 persistent-keepalive=25 comment="Mullvad server 2"

/ip route add dst-address=<SERVER_2_IP>/32 gateway=lte1 comment="Direct route to Mullvad 2"
```

> To switch servers: Winbox → WireGuard → Peers → enable/disable the desired peer. Only keep one active at a time.

---

## Step 5: VLAN Configuration

### 5.1 Create VLAN interfaces

```routeros
/interface vlan add name=vlan10-home vlan-id=10 interface=bridge comment="Home network"
/interface vlan add name=vlan20-ai vlan-id=20 interface=bridge comment="AI network"
```

### 5.2 IP addresses on VLANs

```routeros
/ip address add address=192.168.10.1/24 interface=vlan10-home
/ip address add address=192.168.20.1/24 interface=vlan20-ai
```

### 5.3 Enable VLAN filtering

```routeros
/interface bridge set bridge vlan-filtering=yes
```

### 5.4 DHCP servers

```routeros
# Home network
/ip pool add name=pool-home ranges=192.168.10.2-192.168.10.254
/ip dhcp-server add name=dhcp-home interface=vlan10-home address-pool=pool-home disabled=no
/ip dhcp-server network add address=192.168.10.0/24 gateway=192.168.10.1 dns-server=<MULLVAD_DNS>

# AI network
/ip pool add name=pool-ai ranges=192.168.20.2-192.168.20.254
/ip dhcp-server add name=dhcp-ai interface=vlan20-ai address-pool=pool-ai disabled=no
/ip dhcp-server network add address=192.168.20.0/24 gateway=192.168.20.1 dns-server=<MULLVAD_DNS>
```

---

## Step 6: WiFi SSIDs

### Via Winbox GUI

**For home (WiFi → New):**
- **Name:** `wifi-home`
- **Master Interface:** `wifi2` (2.4GHz)
- **Configuration → SSID:** `home`
- **Security → WPA2-PSK** → set password
- **Datapath → Bridge:** `bridge`

**For ai (WiFi → New):**
- **Name:** `wifi-ai`
- **Master Interface:** `wifi2` (2.4GHz)
- **Configuration → SSID:** `ai`
- **Security → WPA2-PSK** → set password
- **Datapath → Bridge:** `bridge`

> ⚠️ **Important for Chateau LTE6 ax:** Both virtual APs must be on `wifi2` (2.4GHz). Virtual interfaces do not start on `wifi1` (5GHz).

### Bridge ports with PVID

```routeros
/interface bridge port add interface=wifi-home bridge=bridge pvid=10
/interface bridge port add interface=wifi-ai bridge=bridge pvid=20
```

---

## Step 7: AI Network via Mullvad

Generate a **new** config on the Mullvad website as a new device.

```routeros
# Interface
/interface wireguard add name=wg-ai listen-port=13234 private-key="<AI_PRIVATE_KEY>" comment="Mullvad AI"

# IP address
/ip address add address=<AI_ADDRESS_FROM_CONFIG> interface=wg-ai

# Peer
/interface wireguard peers add interface=wg-ai public-key="<AI_PUBLICKEY>" endpoint-address=<AI_ENDPOINT> endpoint-port=51820 allowed-address=0.0.0.0/0 persistent-keepalive=25 comment="Mullvad AI server"

# Direct route to endpoint
/ip route add dst-address=<AI_ENDPOINT>/32 gateway=lte1 comment="Direct route to Mullvad AI"

# Routing table
/routing table add name=ai fib
/ip route add dst-address=0.0.0.0/0 gateway=wg-ai routing-table=ai
/routing rule add src-address=192.168.20.0/24 action=lookup table=ai comment="AI network via Mullvad"

# NAT
/ip firewall nat add chain=srcnat out-interface=wg-ai action=masquerade comment="NAT via Mullvad AI"
```

---

## Step 8: Network Isolation Firewall Rules

```routeros
# home is fully isolated — no access to or from other VLANs
/ip firewall filter add chain=forward src-address=192.168.10.0/24 dst-address=192.168.20.0/24 action=drop comment="Block home to ai"
/ip firewall filter add chain=forward src-address=192.168.10.0/24 dst-address=192.168.88.0/24 action=drop comment="Block home to soc"
/ip firewall filter add chain=forward src-address=192.168.20.0/24 dst-address=192.168.10.0/24 action=drop comment="Block ai to home"
/ip firewall filter add chain=forward src-address=192.168.88.0/24 dst-address=192.168.10.0/24 action=drop comment="Block soc to home"

# soc ↔ ai allowed (for SOC monitoring)
/ip firewall filter add chain=forward src-address=192.168.88.0/24 dst-address=192.168.20.0/24 action=accept comment="Allow soc to ai"
/ip firewall filter add chain=forward src-address=192.168.20.0/24 dst-address=192.168.88.0/24 action=accept comment="Allow ai to soc"
```

---

## Step 9: Verification

### Check tunnels

```routeros
/interface wireguard peers print detail
# Check: last-handshake, rx, tx
```

### Check routing

```routeros
/routing rule print
/ip route print where routing-table="mullvad"
/ip route print where routing-table="ai"
```

### Check from device

Connect to `home` or `ai` network and open:
```
https://mullvad.net/check
```
Should show Mullvad connection with the correct server IP.

---

## Backup

```routeros
/export file=backup_config
```

> ⚠️ WireGuard private keys are NOT included in the export. Save them separately in a secure location.

---

## Common Issues

| Problem | Cause | Fix |
|---------|-------|-----|
| No handshake | No direct route to endpoint | Add `/ip route add dst-address=<ENDPOINT>/32 gateway=lte1` |
| SSID not visible | VLAN filtering disabled | `/interface bridge set bridge vlan-filtering=yes` |
| wifi-home shows status B | Wrong master interface | Switch to `wifi2` |
| Device gets IP but no internet | Missing NAT or routing rule | Check `/ip firewall nat print` and `/routing rule print` |
| Two Mullvad peers conflict | Both have `allowed-address=0.0.0.0/0` | Keep only one peer active at a time |
| Public key on Mullvad website doesn't match | Wrong key imported | Import public key from the MikroTik interface itself |

---

## Final Architecture Diagram

```
                    ┌─────────────────────────────────────┐
                    │    MikroTik Chateau LTE6 ax          │
                    │                                      │
  [home devices] ───┤ wifi-home → vlan10 → wg-al1 ────────┼──→ Mullvad VPN → Internet
  192.168.10.x      │                    (fully isolated)  │
                    │                                      │
  [ai devices] ─────┤ wifi-ai  → vlan20 → wg-ai  ─────────┼──→ Mullvad VPN → Internet
  192.168.20.x      │                 (accessible from soc)│
                    │                                      │
  [soc devices] ────┤ bridge   → 192.168.88.0/24 ──────────┼──→ VPS WireGuard → Internet
  192.168.88.x      │                                      │
                    └─────────────────────────────────────┘
```
