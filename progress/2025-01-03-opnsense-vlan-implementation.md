---
title: "OPNsense VLAN Implementation"
uid: "infra-network-vlan-implementation"
kind: [infrastructure, network, implementation, procedure]
category: "01"
sub-category: "02"
visibility: 3.0
status: 2.0
tags: [opnsense, vlan, implementation, network, firewall, dhcp]
parent_uid: ""
---

# OPNsense VLAN Implementation

**Category:** 01.02 - Network Infrastructure  
**Purpose:** Complete VLAN setup on OPNsense firewall  
**Implemented:** 2025-01-03  
**Author:** Vincent + Claude

## What We Did

Created 15 VLANs on OPNsense to segment the network into security zones. All VLANs run over a single 10GbE DAC cable from OPNsense to the ONTI switch.

## Why This Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    SECURITY MODEL                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   FRONTEND (100-150)          BACKEND (200-250)            │
│   ├─ Untrusted zone           ├─ Trusted zone              │
│   ├─ Firewalled internet      ├─ NO INTERNET (air-gapped)  │
│   ├─ Client devices           ├─ Service-to-service only   │
│   └─ SMB for storage          └─ NFS for storage           │
│                                                             │
│   Traffic rules:                                            │
│   • Frontend → Backend: Via reverse proxy only (VLAN 150)  │
│   • Backend → Frontend: BLOCKED                            │
│   • Backend → Internet: BLOCKED                            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

Backend services (databases, AI, media automation) have no internet access. If a container gets compromised, it can't phone home. Frontend (clients, WiFi, guests) has firewalled internet per VLAN.

## Physical Topology

```
OPNsense (hal)
    │
    │ ix1 ←── 10GbE DAC cable (Intel X553 SFP+)
    │
    ▼
ONTI S508CL-8S (lan4)
    │
    ├── lan1 → Tenda (10G, backend 2.5G bridge)
    ├── lan3 → mothership (10G DAC, Docker)
    ├── lan5 → watson (10G AOC, AI GPU)
    ├── lan6 → alexandria (10G AOC, NAS)
    └── lan8 → Zyxel (1G SFP, frontend bridge)
```

All 15 VLANs are tagged on the ix1 trunk. The ONTI switch untags frames to the correct port based on VLAN membership.

## VLAN Assignments

### Frontend VLANs (100-150) - Firewalled Internet

| VLAN | Subnet | Name | Purpose |
|------|--------|------|---------|
| 100 | 192.168.100.0/24 | QUARANTINE | Unknown MAC, setup only |
| 101 | 192.168.101.0/24 | PRIVILEGED | Known MAC + domain auth |
| 110 | 192.168.110.0/24 | GUEST_WIFI | Captive portal, time-limited |
| 111 | 192.168.111.0/24 | HOME_WIFI | Trusted home devices |
| 112 | 192.168.112.0/24 | IOT | Smart home, NO internet |
| 120 | 192.168.120.0/24 | WORK | Work laptop, isolated |
| 130 | 192.168.130.0/24 | BASEMENT | Remote site via WireGuard |
| 131 | 192.168.131.0/24 | GARAGE | Remote site via WireGuard |
| 140 | 192.168.140.0/24 | VVE | Building cameras, isolated |
| 150 | 192.168.150.0/24 | PROXY | Reverse proxy to backend |

### Backend VLANs (200-250) - NO Internet

| VLAN | Subnet | Name | Purpose |
|------|--------|------|---------|
| 200 | 192.168.200.0/24 | AI | vLLM, LiteLLM, embeddings |
| 210 | 192.168.210.0/24 | ARR | Radarr, Sonarr, Prowlarr |
| 220 | 192.168.220.0/24 | DATABASE | PostgreSQL, Redis, Wiki.js |
| 230 | 192.168.230.0/24 | STORAGE | NFS server exports |
| 240 | 192.168.240.0/24 | DOCKER | Container networks |

### Management VLAN (254) - Admin Only

| Subnet | Purpose |
|--------|---------|
| 192.168.254.0/24 | Physical infrastructure (switches, APs, Proxmox hosts) |

## OPNsense Interface Mapping

| Interface | VLAN | Physical | IP Address |
|-----------|------|----------|------------|
| opt4 | 100 | ix1_vlan100 | 192.168.100.1/24 |
| opt5 | 101 | ix1_vlan101 | 192.168.101.1/24 |
| opt6 | 110 | ix1_vlan110 | 192.168.110.1/24 |
| opt7 | 111 | ix1_vlan111 | 192.168.111.1/24 |
| opt8 | 112 | ix1_vlan112 | 192.168.112.1/24 |
| opt9 | 120 | ix1_vlan120 | 192.168.120.1/24 |
| opt10 | 130 | ix1_vlan130 | 192.168.130.1/24 |
| opt11 | 131 | ix1_vlan131 | 192.168.131.1/24 |
| opt12 | 140 | ix1_vlan140 | 192.168.140.1/24 |
| opt13 | 150 | ix1_vlan150 | 192.168.150.1/24 |
| opt14 | 200 | ix1_vlan200 | 192.168.200.1/24 |
| opt15 | 210 | ix1_vlan210 | 192.168.210.1/24 |
| opt16 | 220 | ix1_vlan220 | 192.168.220.1/24 |
| opt17 | 230 | ix1_vlan230 | 192.168.230.1/24 |
| opt18 | 240 | ix1_vlan240 | 192.168.240.1/24 |

Pre-existing interfaces (not touched):
- WAN: igc0 (Ziggo modem)
- LAN: igc1 (legacy, 192.168.1.0/24)
- opt1 (MGMT): igc2 (192.168.254.1/24)

## DHCP Configuration

All VLANs have DHCP enabled:
- Static range: .1 - .99 (reserved for servers, infrastructure)
- DHCP range: .100 - .250
- Gateway: .1 (OPNsense)
- DNS: .1 (OPNsense Unbound)

## Firewall Rules (Current State)

Current rules are permissive "allow all" for initial testing:

```
QUARANTINE: Allow to any (will restrict to 22/80/443 only)
PRIVILEGED: Allow to any (will add zone rules)
GUEST_WIFI: Allow to any (will add captive portal + throttle)
HOME_WIFI: Allow to any (will firewalled internet)
IOT: Allow to any (will BLOCK internet)
WORK: Allow to any (will isolate)
BASEMENT: Allow to any (WireGuard site)
GARAGE: Allow to any (WireGuard site)
VVE: Allow to any (will BLOCK internet)
PROXY: Allow to any (only allow to backend services)
AI: Allow to any (will BLOCK internet)
ARR: Allow to any (will BLOCK internet, except download proxy)
DATABASE: Allow to any (will BLOCK internet)
STORAGE: Allow to any (will BLOCK internet)
DOCKER: Allow to any (will BLOCK internet)
```

**TODO:** Implement proper zone isolation rules per vlan-design.md.

## How It Was Implemented

### Method: Python XML Manipulation

OPNsense config is stored in `/conf/config.xml`. We used Python to bulk-add VLANs, interfaces, DHCP, and firewall rules.

### The Script

Location on OPNsense: `/tmp/clean_config.py`

```python
import xml.etree.ElementTree as ET

tree = ET.parse("/conf/config.xml")
root = tree.getroot()

# === VLANS ===
vlans = root.find("vlans")
for v in list(vlans):
    vlans.remove(v)

vlan_defs = [
    (100, "Quarantine"), (101, "Privileged"), (110, "Guest_WiFi"),
    (111, "Home_WiFi"), (112, "IoT"), (120, "Work"),
    (130, "Basement"), (131, "Garage"), (140, "VvE"), (150, "Proxy"),
    (200, "AI"), (210, "ARR"), (220, "Database"), (230, "Storage"), (240, "Docker"),
]

for tag, desc in vlan_defs:
    vlan = ET.SubElement(vlans, "vlan")
    ET.SubElement(vlan, "if").text = "ix1"
    ET.SubElement(vlan, "tag").text = str(tag)
    ET.SubElement(vlan, "descr").text = desc
    ET.SubElement(vlan, "vlanif").text = "ix1_vlan" + str(tag)

# === INTERFACES ===
interfaces = root.find("interfaces")
for child in list(interfaces):
    if child.tag.startswith("opt"):
        try:
            num = int(child.tag[3:])
            if num > 3:
                interfaces.remove(child)
        except:
            pass

iface_defs = [
    (4, 100, "QUARANTINE"), (5, 101, "PRIVILEGED"), (6, 110, "GUEST_WIFI"),
    (7, 111, "HOME_WIFI"), (8, 112, "IOT"), (9, 120, "WORK"),
    (10, 130, "BASEMENT"), (11, 131, "GARAGE"), (12, 140, "VVE"), (13, 150, "PROXY"),
    (14, 200, "AI"), (15, 210, "ARR"), (16, 220, "DATABASE"), (17, 230, "STORAGE"), (18, 240, "DOCKER"),
]

for opt_num, tag, name in iface_defs:
    iface = ET.SubElement(interfaces, "opt" + str(opt_num))
    ET.SubElement(iface, "if").text = "ix1_vlan" + str(tag)
    ET.SubElement(iface, "enable").text = "1"
    ET.SubElement(iface, "descr").text = name
    ET.SubElement(iface, "ipaddr").text = "192.168." + str(tag) + ".1"
    ET.SubElement(iface, "subnet").text = "24"

# === DHCP ===
dhcpd = root.find("dhcpd")
dhcp_map = [
    ("opt4", 100), ("opt5", 101), ("opt6", 110), ("opt7", 111), ("opt8", 112),
    ("opt9", 120), ("opt10", 130), ("opt11", 131), ("opt12", 140), ("opt13", 150),
    ("opt14", 200), ("opt15", 210), ("opt16", 220), ("opt17", 230), ("opt18", 240),
]

for opt, tag in dhcp_map:
    existing = dhcpd.find(opt)
    if existing is not None:
        dhcpd.remove(existing)
    
    dhcp = ET.SubElement(dhcpd, opt)
    ET.SubElement(dhcp, "enable").text = "1"
    rng = ET.SubElement(dhcp, "range")
    ET.SubElement(rng, "from").text = "192.168." + str(tag) + ".100"
    ET.SubElement(rng, "to").text = "192.168." + str(tag) + ".250"
    ET.SubElement(dhcp, "gateway").text = "192.168." + str(tag) + ".1"
    ET.SubElement(dhcp, "dnsserver").text = "192.168." + str(tag) + ".1"

# === FIREWALL RULES ===
filt = root.find("filter")
fw_defs = [
    ("opt4", "QUARANTINE"), ("opt5", "PRIVILEGED"), ("opt6", "GUEST_WIFI"),
    ("opt7", "HOME_WIFI"), ("opt8", "IOT"), ("opt9", "WORK"),
    ("opt10", "BASEMENT"), ("opt11", "GARAGE"), ("opt12", "VVE"), ("opt13", "PROXY"),
    ("opt14", "AI"), ("opt15", "ARR"), ("opt16", "DATABASE"), ("opt17", "STORAGE"), ("opt18", "DOCKER"),
]

for i, (opt, name) in enumerate(fw_defs):
    rule = ET.SubElement(filt, "rule")
    ET.SubElement(rule, "type").text = "pass"
    ET.SubElement(rule, "interface").text = opt
    ET.SubElement(rule, "ipprotocol").text = "inet"
    ET.SubElement(rule, "descr").text = "Allow " + name + " to any"
    source = ET.SubElement(rule, "source")
    ET.SubElement(source, "network").text = opt
    dest = ET.SubElement(rule, "destination")
    ET.SubElement(dest, "any")
    ET.SubElement(rule, "tracker").text = str(2000000 + i)

tree.write("/conf/config.xml.vlan", encoding="utf-8", xml_declaration=True)
```

### Activation Steps

```bash
# 1. Create backup
cp /conf/config.xml /conf/config.xml.pre-vlan-backup

# 2. Run script
python3 /tmp/clean_config.py

# 3. Validate XML
python3 -c 'import xml.etree.ElementTree as ET; ET.parse("/conf/config.xml.vlan")'

# 4. Apply config
cp /conf/config.xml.vlan /conf/config.xml

# 5. Reload all services
/usr/local/etc/rc.reload_all
```

## Verification

### Check Interfaces Are Up

```bash
ifconfig | grep -E '^ix1_vlan|inet 192.168\.(1[0-5][0-9]|2[0-4][0-9])\.'
```

Expected output: 15 interfaces, each with correct IP.

### Verify From OPNsense WebUI

1. Interfaces → Assignments: See all 15 VLAN interfaces
2. Interfaces → [Each VLAN]: Verify IP and enabled
3. Services → DHCPv4: See 15 DHCP servers
4. Firewall → Rules: See rules per interface

### Test VLAN Connectivity (After Switch Config)

From a device on VLAN 101:
```bash
ping 192.168.101.1  # Gateway
ping 192.168.254.1  # Should fail (different VLAN, no route yet)
```

## Recovery Procedures

### Rollback to Pre-VLAN Config

```bash
ssh root@192.168.1.1
cp /conf/config.xml.pre-vlan-backup /conf/config.xml
/usr/local/etc/rc.reload_all
```

### If Locked Out

1. Connect monitor/keyboard to Qotom
2. Option 8: Shell
3. `cp /conf/config.xml.pre-vlan-backup /conf/config.xml`
4. `reboot`

### Config Backup Locations

| File | Purpose |
|------|---------|
| `/conf/config.xml` | Active config |
| `/conf/config.xml.pre-vlan-backup` | Before VLAN implementation |
| `/conf/config.xml.vlan` | Generated VLAN config |

## Next Steps

1. **Switch VLAN Config**: Configure ONTI and Zyxel to trunk/untag VLANs
2. **Firewall Hardening**: Block backend internet, zone isolation
3. **Test End-to-End**: Connect device, verify DHCP, verify isolation
4. **DNS Setup**: Local zone for `local.emesix.nl`
5. **Authentik**: RADIUS for WiFi, LDAP for services

## Credentials

| Device | Address | User | Password |
|--------|---------|------|----------|
| OPNsense | 192.168.1.1 | root | NikonD90 |
| OPNsense WebUI | 192.168.254.254 | root | NikonD90 |
| Zyxel | 192.168.254.2 | root | (empty) |
| ONTI | 192.168.254.3 | root | (unknown) |

## Related Documentation

- [[homelab/network/vlan-design]] - Security model and VLAN purpose
- [[homelab/network/ip-plan]] - Complete IP assignments
- [[homelab/network/topology]] - Physical switch connections
- [[homelab/network/architecture]] - High-level design

## Session Transcript

Full implementation log: `/mnt/transcripts/2026-01-02-23-02-44-opnsense-vlan-bulk-configuration.txt`
