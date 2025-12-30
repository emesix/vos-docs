---
title: RADIUS WiFi and VPN Authentication
description: WPA3-Enterprise WiFi authentication with dynamic VLAN assignment using Authentik RADIUS
published: true
date: 2025-12-28T18:47:26.527Z
tags: vlan, wifi, authentik, radius, authentication, vpn, wpa3, 802.1x
editor: markdown
dateCreated: 2025-12-28T18:47:23.915Z
---

---
title: "RADIUS WiFi and VPN Authentication"
uid: "identity-radius-wifi"
kind: [howto, identity, network]
category: "03"
sub-category: "05"
visibility: 3.0
status: 3.0
tags: [radius, wifi, vpn, authentication, wpa3, 802.1x, authentik, vlan]
parent_uid: "svc-authentik"
---

# RADIUS WiFi and VPN Authentication

**Category:** 03.05 - Identity Services  
**Purpose:** WiFi and VPN authentication with dynamic VLAN  
**Updated:** 2025-12-28

## Overview

Authentik RADIUS outpost handles WiFi (WPA3-Enterprise), VPN (OpenVPN/WireGuard), and switch 802.1X authentication. Returns VLAN assignment based on user/device group membership.

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    RADIUS AUTHENTICATION                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   User/Device                                               │
│        │                                                    │
│        │ Connect to WiFi/VPN/Port                          │
│        ▼                                                    │
│   ┌─────────────┐                                           │
│   │   AP/VPN/   │                                           │
│   │   Switch    │                                           │
│   └──────┬──────┘                                           │
│          │ RADIUS request                                   │
│          ▼                                                   │
│   ┌────────────────────┐                                     │
│   │  Authentik RADIUS  │                                     │
│   │  <LAN_IP>    │                                     │
│   └──────────┬─────────┘                                     │
│              │                                               │
│              │ Validate + Get Groups                         │
│              ▼                                               │
│   ┌────────────────────┐                                     │
│   │  Return VLAN ID    │                                     │
│   │  based on group    │                                     │
│   └────────────────────┘                                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Dynamic VLAN Assignment

| User/Group | VLAN | Subnet | Access |
|------------|------|--------|--------|
| admins | 101 | <LAN_IP>/24 | Full privileged |
| privileged-users | 101 | <LAN_IP>/24 | LAN + services |
| guests | 110 | <LAN_IP>/24 | Internet only |
| home-devices | 111 | <LAN_IP>/24 | Home network |
| iot-devices | 112 | <LAN_IP>/24 | Local only |
| work-devices | 120 | <LAN_IP>/24 | Isolated |
| unknown | 100 | <LAN_IP>/24 | Quarantine |

## WiFi Configuration

### SSIDs

| SSID | Auth | VLAN | Purpose |
|------|------|------|--------|
| VOS-Home | WPA3-Enterprise | Dynamic | Trusted users |
| VOS-Guest | Captive Portal | 110 | Guests |
| VOS-IoT | WPA2-PSK | 112 | IoT devices |

### OpenWRT AP Configuration (RT-2980)

```bash
# /etc/config/wireless

config wifi-iface 'wlan0_enterprise'
    option device 'radio0'
    option mode 'ap'
    option ssid 'VOS-Home'
    option encryption 'wpa3'
    option auth_server '<LAN_IP>'
    option auth_port '1812'
    option auth_secret 'RADIUS_SECRET_HERE'
    option acct_server '<LAN_IP>'
    option acct_port '1813'
    option acct_secret 'RADIUS_SECRET_HERE'
    option dynamic_vlan '2'           # Required for VLAN assignment
    option vlan_tagged_interface 'eth0'
    option vlan_bridge 'br-lan'
    option ieee80211w '2'             # MFP required (WPA3)
```

### VLAN Bridge Setup

```bash
# /etc/config/network

# Base bridge (management)
config interface 'lan'
    option device 'br-lan'
    option proto 'static'
    option ipaddr '<LAN_IP>'    # AP management IP
    option netmask '255.255.255.0'
    option gateway '<LAN_IP>'

# Dynamic VLAN bridges created automatically by hostapd
# When RADIUS returns Tunnel-Private-Group-ID = 101:
# - Creates br-lan.101 tagged interface
# - Assigns client to that VLAN
```

## Authentik RADIUS Outpost

### Outpost Configuration

```yaml
authentik_radius_outpost:
  name: "radius-outpost"
  type: "radius"
  
  # Network settings
  host: "0.0.0.0"
  port_auth: 1812
  port_acct: 1813
  
  # Client definitions (APs, switches)
  clients:
    - name: "rt2980-1"
      ip: "<LAN_IP>"
      secret: "${RADIUS_SECRET}"
    - name: "rt2980-2"
      ip: "<LAN_IP>"
      secret: "${RADIUS_SECRET}"
    - name: "rt2980-3"
      ip: "<LAN_IP>"
      secret: "${RADIUS_SECRET}"
    - name: "rt2980-4"
      ip: "<LAN_IP>"
      secret: "${RADIUS_SECRET}"
    - name: "rt2980-5"
      ip: "<LAN_IP>"
      secret: "${RADIUS_SECRET}"
    - name: "zyxel-switch"
      ip: "<LAN_IP>"
      secret: "${RADIUS_SECRET}"
```

### VLAN Assignment Policy

Authentik expression policy for VLAN assignment:

```python
# Policy: assign-vlan-by-group

# Check group membership and return VLAN
if ak_is_group_member(request.user, name="admins"):
    request.context["radius"]["Tunnel-Type"] = "VLAN"
    request.context["radius"]["Tunnel-Medium-Type"] = "IEEE-802"
    request.context["radius"]["Tunnel-Private-Group-ID"] = "101"
    return True

if ak_is_group_member(request.user, name="privileged-users"):
    request.context["radius"]["Tunnel-Private-Group-ID"] = "101"
    return True

if ak_is_group_member(request.user, name="guests"):
    request.context["radius"]["Tunnel-Private-Group-ID"] = "110"
    return True

if ak_is_group_member(request.user, name="iot-devices"):
    request.context["radius"]["Tunnel-Private-Group-ID"] = "112"
    return True

if ak_is_group_member(request.user, name="work-devices"):
    request.context["radius"]["Tunnel-Private-Group-ID"] = "120"
    return True

# Default: quarantine
request.context["radius"]["Tunnel-Private-Group-ID"] = "100"
return True
```

## VPN Authentication

### WireGuard with Authentik

Option 1: User portal generates configs.

```
User logs into Authentik
        │
        ▼
┌────────────────────┐
│  User Portal       │
│  "Download VPN"    │
└──────────┬─────────┘
           │
           ▼
┌────────────────────┐
│  Generated config: │
│  - Personal keys   │
│  - Allowed IPs     │
│  - Based on groups │
└────────────────────┘
```

### OpenVPN with RADIUS

```bash
# /etc/openvpn/server.conf

plugin /usr/lib/openvpn/plugins/openvpn-plugin-auth-pam.so openvpn
# OR use radius plugin:
plugin /usr/lib/openvpn/radiusplugin.so /etc/openvpn/radiusplugin.cnf

# radiusplugin.cnf
NAS-Identifier=openvpn
Service-Type=5
Framed-Protocol=1
NAS-Port-Type=5
NAS-IP-Address=<LAN_IP>
server {
    acctport=1813
    authport=1812
    name=<LAN_IP>
    retry=1
    wait=1
    sharedsecret=RADIUS_SECRET_HERE
}
```

### Headscale with OIDC

Self-hosted Tailscale control server with Authentik SSO.

```yaml
# /etc/headscale/config.yaml

oidc:
  issuer: "https://auth.vos.home/application/o/headscale/"
  client_id: "headscale"
  client_secret: "${HEADSCALE_OIDC_SECRET}"
  scope: ["openid", "profile", "email", "groups"]
  
  # Map Authentik groups to Headscale ACL tags
  # Groups claim from Authentik
  allowed_groups:
    - "admins"
    - "privileged-users"
```

## Switch 802.1X (Zyxel)

```
Device plugs into port
        │
        │ 802.1X EAP
        ▼
┌────────────────────┐
│   Zyxel Switch     │
└──────────┬─────────┘
           │ RADIUS
           ▼
┌────────────────────┐
│  Authentik RADIUS  │
│  Returns VLAN ID   │
└────────────────────┘
           │
           ▼
   Port assigned to VLAN
```

### Zyxel RADIUS Configuration

```
# Via Zyxel web UI or CLI

radius-server host <LAN_IP>
radius-server key RADIUS_SECRET_HERE
radius-server timeout 3
radius-server retransmit 2

aaa authentication dot1x default radius
aaa authorization network default radius

interface ethernet 1/1-24
  dot1x port-control auto
  dot1x guest-vlan 100
  dot1x auth-fail-vlan 100
  dot1x radius-vlan enable
```

## MFA Enforcement

| Access Method | MFA | Notes |
|---------------|-----|-------|
| WiFi (WPA3-Enterprise) | Optional | Per-user setting in Authentik |
| VPN | Required | Always prompt for TOTP |
| Web SSO | Required | For admin accounts |
| 802.1X Switch | No | MAC/cert based |

## Troubleshooting

### Test RADIUS Auth

```bash
# From any Linux host with radtest
radtest user 'password' <LAN_IP> 0 RADIUS_SECRET_HERE

# Expected response:
Access-Accept
  Tunnel-Type = VLAN
  Tunnel-Medium-Type = IEEE-802
  Tunnel-Private-Group-ID = 101
```

### Debug OpenWRT

```bash
# Watch hostapd logs
logread -f | grep hostapd

# Check VLAN assignment
bridge vlan show

# Verify RADIUS communication
tcpdump -i br-lan port 1812
```

## Related

- [[homelab/identity/authentik]]
- [[homelab/identity/users-groups]]
- [[homelab/network/vlan-design]]
- [[homelab/network/security]]