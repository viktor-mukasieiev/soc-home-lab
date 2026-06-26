# WireGuard VPN: MikroTik ↔ VPS Setup

**Date:** June 2026
**Environment:** MikroTik RouterOS 7.19.6 (LTE router) + Ubuntu 22.04 VPS
**Protocol:** WireGuard
**Result:** ✅ Success — all LAN traffic exits through VPS IP, no DNS leaks, Kill Switch active.

---

## Architecture

```
LAN devices (192.168.88.0/24)
        ↓
MikroTik (lte1 — SIM card)
        ↓  [WireGuard tunnel, encrypted]
VPS Ubuntu 22.04 (172.16.X.1)
        ↓  [NAT, ip_forward]
Internet (exits with VPS IP)
```

**Tunnel addressing:**
- VPS (server): `172.16.X.1/24`
- MikroTik (client): `172.16.X.2/24`
- WireGuard port: `51820/UDP`

> ⚠️ Real IP addresses are intentionally generalized in this documentation.

---

## Step 1: VPS — Server Setup

```bash
# Install WireGuard
apt install -y wireguard wireguard-tools iptables

# Generate server keys
wg genkey | tee /etc/wireguard/server_private.key | wg pubkey | tee /etc/wireguard/server_public.key

# Generate client keys (MikroTik)
wg genkey | tee /etc/wireguard/client_private.key | wg pubkey | tee /etc/wireguard/client_public.key
```

**Final `/etc/wireguard/wg0.conf`:**
```ini
[Interface]
Address = 172.16.X.1/24
ListenPort = 51820
PrivateKey = <SERVER_PRIVATE_KEY>
PostUp = sysctl -w net.ipv4.ip_forward=1; iptables -t nat -A POSTROUTING -s 172.16.X.0/24 -o <INTERFACE> -j MASQUERADE; iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -j ACCEPT
PostDown = iptables -t nat -D POSTROUTING -s 172.16.X.0/24 -o <INTERFACE> -j MASQUERADE; iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -o wg0 -j ACCEPT

[Peer]
# MikroTik
PublicKey = <MIKROTIK_PUBLIC_KEY>
AllowedIPs = 172.16.X.2/32
```

> ⚠️ Always check your interface name first: `ip route | grep default`

```bash
# Persistent ip_forward
echo "net.ipv4.ip_forward=1" | tee /etc/sysctl.d/99-forward.conf
sysctl -p

# Enable autostart
systemctl enable --now wg-quick@wg0

# Save iptables rules
apt install -y iptables-persistent
netfilter-persistent save
```

---

## Step 2: MikroTik — Client Setup

**WireGuard interface:**
```routeros
/interface wireguard add name=wg-vpn listen-port=13231 private-key="<CLIENT_PRIVATE_KEY>"
```

**Peer (VPS):**
```routeros
/interface wireguard peers add \
  interface=wg-vpn \
  public-key="<SERVER_PUBLIC_KEY>" \
  endpoint-address="<VPS_IP>" \
  endpoint-port=51820 \
  allowed-address=0.0.0.0/0 \
  persistent-keepalive=25
```

**Tunnel IP address:**
```routeros
/ip address add address=172.16.X.2/24 interface=wg-vpn
```

**Routing table:**
```routeros
/routing table add fib name=vpn-route
```

**Mangle — mark LAN traffic:**
```routeros
/ip firewall mangle add \
  chain=prerouting \
  action=mark-routing \
  new-routing-mark=vpn-route \
  src-address=192.168.88.0/24 \
  dst-address=!192.168.88.0/24 \
  passthrough=yes \
  comment="Mark LAN for VPN routing"
```

> ℹ️ The `dst-address=!192.168.88.0/24` exclusion is critical — prevents DNS loops where local DNS requests get pushed into the tunnel.

**Route in vpn-route table:**
```routeros
/ip route add dst-address=0.0.0.0/0 gateway=wg-vpn routing-table=vpn-route
```

**NAT:**
```routeros
/ip firewall nat add chain=srcnat action=masquerade out-interface=wg-vpn comment="NAT via WireGuard"
```

**Kill Switch:**
```routeros
/ip firewall filter add chain=forward action=drop out-interface=lte1 src-address=192.168.88.0/24 comment="Kill Switch: block LAN if VPN down"
```

**DNS via tunnel:**
```routeros
/ip dns set servers=1.1.1.1,8.8.8.8 allow-remote-requests=yes
```

---

## Lessons Learned — Mistakes & Fixes

> This section documents every mistake made during setup. In a real SOC environment, post-incident documentation is as important as the fix itself.

### ❌ Mistake 1: Tunnel subnet conflict with VPS provider
**What happened:** Chose `10.0.0.0/24` for the tunnel, but the VPS provider gateway also used `10.0.0.1`. This caused a routing conflict and the VPS lost internet.

**Lesson:** Always check `ip route show` and `cat /etc/network/interfaces` on the VPS before choosing a tunnel subnet.

**Fix:** Changed to `172.16.X.0/24`.

---

### ❌ Mistake 2: iptables not installed on fresh VPS
**What happened:** Fresh Ubuntu 22.04 VPS did not have `iptables` installed. `PostUp` commands in `wg0.conf` failed with `command not found` and the tunnel did not start.

**Lesson:** Always run `apt install -y iptables` before configuring WireGuard on a fresh Ubuntu VPS.

---

### ❌ Mistake 3: Static route instead of policy routing
**What happened:** Added a default route `0.0.0.0/0 gateway=wg-vpn` directly to the MikroTik main routing table. This created a loop — WireGuard keepalive packets tried to route through the tunnel itself.

**Lesson:** On MikroTik with an LTE interface, use **policy routing via mangle**. Router's own traffic (WireGuard keepalive, SSH) goes through `lte1`. LAN traffic is tagged by mangle and sent to a separate `vpn-route` table via `wg-vpn`.

---

### ❌ Mistake 4: Wrong VPS network interface name in iptables rules
**What happened:** Used `ens3` in iptables rules, but during debugging confused local Ubuntu machine (which used `enp0s5`). NAT rules did not apply.

**Lesson:** Always check interface name first with `ip route | grep default` and use that exact name everywhere.

---

### ❌ Mistake 5: ip_forward not persistent after reboot
**What happened:** Applied `net.ipv4.ip_forward=1` only via `sysctl -w` (temporary). After service restart, forwarding was disabled and traffic stopped routing.

**Lesson:** Always set `ip_forward` in two places:
1. `/etc/sysctl.d/99-forward.conf` (permanent)
2. `PostUp` in WireGuard config (failsafe on tunnel start)

---

### ❌ Mistake 6: Two peers with different keys after router reset
**What happened:** After MikroTik reset, a new key pair was generated. Old public key remained in `wg0.conf` on VPS, new one was added separately via `wg set`. Result: two conflicting peers.

**Lesson:** After any MikroTik reset or WireGuard interface recreation — immediately update `PublicKey` in `wg0.conf` on VPS and restart `wg-quick`.

---

### ❌ Mistake 7: DNS loop
**What happened:** LAN devices sent DNS requests to `192.168.88.1` (router). Mangle tagged them and pushed them into the tunnel. VPS did not know what to do with requests to a private address.

**Lesson:** Add `dst-address=!192.168.88.0/24` exclusion to the mangle rule so local traffic (DNS to router, DHCP) does not enter the tunnel.

---

## Quick Deployment Checklist

### VPS (do this first):
- [ ] `apt install -y wireguard wireguard-tools iptables iptables-persistent`
- [ ] Generate 2 key pairs (server + client)
- [ ] Check interface name: `ip route | grep default`
- [ ] Create `/etc/wireguard/wg0.conf` with correct interface in PostUp/PostDown
- [ ] Confirm tunnel subnet does not conflict with provider
- [ ] `echo "net.ipv4.ip_forward=1" > /etc/sysctl.d/99-forward.conf`
- [ ] `systemctl enable --now wg-quick@wg0`
- [ ] `netfilter-persistent save`
- [ ] Verify: `sudo wg show` — interface present, waiting for peer

### MikroTik (do this second):
- [ ] Create WireGuard interface with client private key
- [ ] Verify key saved: `/interface wireguard print`
- [ ] Add peer with server public key
- [ ] Add tunnel IP address
- [ ] Create routing table `vpn-route`
- [ ] Add mangle rule with local subnet exclusion
- [ ] Add route in `vpn-route` table
- [ ] Add NAT masquerade for `wg-vpn`
- [ ] Add Kill Switch rule
- [ ] Set DNS: `1.1.1.1, 8.8.8.8`

### After setup:
- [ ] Update MikroTik PublicKey in `/etc/wireguard/wg0.conf` on VPS
- [ ] Verify handshake: `/interface wireguard peers print detail`
- [ ] Verify IP: open `ifconfig.me` from a LAN device
- [ ] Verify DNS: `dnsleaktest.com`
- [ ] Verify Kill Switch: disable `wg-vpn`, confirm LAN loses internet

---

## Troubleshooting Reference

### Tunnel not coming up:
```bash
# VPS
sudo wg-quick up wg0   # watch for errors
sudo wg show           # check handshake
```

### LAN devices have no internet:
```bash
# VPS — check in order:
cat /proc/sys/net/ipv4/ip_forward          # must be 1
sudo iptables -L FORWARD -n -v             # rules must exist with non-zero counters
sudo iptables -t nat -L POSTROUTING -n -v
sudo tcpdump -i wg0 -n                     # is traffic arriving from MikroTik?
sudo tcpdump -i <INTERFACE> -n src 172.16.X.0/24  # is traffic being forwarded out?
```

```routeros
# MikroTik — check in order:
/interface wireguard peers print detail        # is there a handshake?
/ip firewall mangle print stats               # are counters growing?
/ip firewall nat print stats                  # are NAT counters growing?
/ip route print where routing-table=vpn-route # is route active?
```

### After VPS reboot everything broke:
```bash
sudo systemctl status wg-quick@wg0   # check service status
cat /proc/sys/net/ipv4/ip_forward    # check forwarding
sudo iptables -L FORWARD -n -v       # check rules
```

---

## SOC Analyst Notes

**Monitoring points for this tunnel:**
- Tunnel health: `wg show` — `latest handshake` should never be older than 3 minutes
- Traffic flow: `transfer` counter in `wg show` should grow continuously
- DNS leaks: periodic check via `dnsleaktest.com`
- Kill Switch: if `wg-vpn` goes down, all LAN traffic must be blocked by firewall rule

**MikroTik logs:**
```routeros
/log print where topics~"wireguard"
```

**VPS logs:**
```bash
sudo journalctl -u wg-quick@wg0 -f
```
