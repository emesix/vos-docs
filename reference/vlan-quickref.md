---
title: "VLAN Quick Reference"
uid: "infra-network-vlan-quickref"
kind: [reference, network, operations]
category: "01"
sub-category: "02"
visibility: 3.0
status: 2.0
tags: [vlan, reference, cheatsheet, opnsense]
parent_uid: ""
---

# VLAN Quick Reference

Print this. Stick it on the wall.

## VLAN → Subnet Lookup

| VLAN | Subnet | Gateway | Name |
|------|--------|---------|------|
| 100 | 192.168.**100**.0/24 | .100.1 | Quarantine |
| 101 | 192.168.**101**.0/24 | .101.1 | Privileged |
| 110 | 192.168.**110**.0/24 | .110.1 | Guest WiFi |
| 111 | 192.168.**111**.0/24 | .111.1 | Home WiFi |
| 112 | 192.168.**112**.0/24 | .112.1 | IoT |
| 120 | 192.168.**120**.0/24 | .120.1 | Work |
| 130 | 192.168.**130**.0/24 | .130.1 | Basement |
| 131 | 192.168.**131**.0/24 | .131.1 | Garage |
| 140 | 192.168.**140**.0/24 | .140.1 | VvE |
| 150 | 192.168.**150**.0/24 | .150.1 | Reverse Proxy |
| 200 | 192.168.**200**.0/24 | .200.1 | AI |
| 210 | 192.168.**210**.0/24 | .210.1 | ARR |
| 220 | 192.168.**220**.0/24 | .220.1 | Database |
| 230 | 192.168.**230**.0/24 | .230.1 | Storage |
| 240 | 192.168.**240**.0/24 | .240.1 | Docker |
| 254 | 192.168.**254**.0/24 | .254.1 | Management |

**Pattern:** VLAN tag = third octet. Always.

## Zone Rules

```
FRONTEND (100-150)     BACKEND (200-250)
     │                      │
     │ Firewalled           │ NO INTERNET
     │ Internet             │ Air-gapped
     │                      │
     └───── Via VLAN 150 ───┘
           (Proxy only)
```

## IP Ranges Per VLAN

| Range | Purpose |
|-------|---------|
| .1 | Gateway (OPNsense) |
| .2-.10 | Network infra |
| .11-.20 | Access points |
| .21-.50 | Servers (static) |
| .51-.99 | Reserved |
| .100-.250 | DHCP pool |
| .251-.254 | Special |

## OPNsense Interface Names

| opt# | VLAN | Interface Name |
|------|------|----------------|
| opt4 | 100 | QUARANTINE |
| opt5 | 101 | PRIVILEGED |
| opt6 | 110 | GUEST_WIFI |
| opt7 | 111 | HOME_WIFI |
| opt8 | 112 | IOT |
| opt9 | 120 | WORK |
| opt10 | 130 | BASEMENT |
| opt11 | 131 | GARAGE |
| opt12 | 140 | VVE |
| opt13 | 150 | PROXY |
| opt14 | 200 | AI |
| opt15 | 210 | ARR |
| opt16 | 220 | DATABASE |
| opt17 | 230 | STORAGE |
| opt18 | 240 | DOCKER |

## Quick Commands

```bash
# SSH to OPNsense
ssh root@192.168.1.1

# Show all VLAN interfaces
ifconfig | grep ix1_vlan

# Check specific VLAN
ifconfig ix1_vlan200

# Reload config
/usr/local/etc/rc.reload_all

# Rollback
cp /conf/config.xml.pre-vlan-backup /conf/config.xml
/usr/local/etc/rc.reload_all
```

## Troubleshooting

| Symptom | Check |
|---------|-------|
| No DHCP | Is VLAN trunk tagged on switch? |
| Can't reach gateway | Is interface UP? `ifconfig ix1_vlanXXX` |
| Can reach gateway, no internet | Firewall rule blocking? |
| Wrong VLAN | Switch port config? Check tagged/untagged |

## Physical Path

```
Device → Switch Port → VLAN Tag → OPNsense ix1 → Firewall → Route
```

All VLANs trunk over single 10GbE DAC: OPNsense ix1 ↔ ONTI lan4
