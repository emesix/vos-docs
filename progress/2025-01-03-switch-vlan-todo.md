---
title: "Switch VLAN Configuration TODO"
uid: "infra-network-switch-vlan-todo"
kind: [infrastructure, network, procedure]
category: "01"
sub-category: "02"
visibility: 3.0
status: 4.0
tags: [switch, vlan, openwrt, onti, zyxel, todo]
parent_uid: ""
---

# Switch VLAN Configuration TODO

**Category:** 01.02 - Network Infrastructure  
**Purpose:** Configure switches to carry OPNsense VLANs  
**Status:** In progress  
**Updated:** 2025-01-03
**Depends on:** OPNsense VLAN implementation (DONE)

## Current State (2025-01-03)

### What's Working
- OPNsense: 15 VLANs on ix1 trunk ✅
- ONTI management: 192.168.254.3 on VLAN 254 ✅
- Zyxel management: 192.168.254.2 on VLAN 254 ✅
- BiDi fiber link: ONTI P8 ↔ Zyxel P26 @ 1Gbps ✅

### What's NOT Working
- LACP/LAG: NOT POSSIBLE (OpenWRT RTL838x/RTL930x lacks HW offload)
- Second fiber: P7↔P25 has rx_los, abandoned
- VLAN trunking: Not yet configured on switches

### Topology
```
OPNsense ix1 (trunk, all VLANs)
      │
      │ DAC 10GbE (lan4)
      ▼
ONTI S508CL-8S (.3) ← VLAN trunk needed here
      │
      │ BiDi 1Gbps (lan8 ↔ lan26)
      ▼
Zyxel GS1900-24HP (.2) ← VLAN trunk + access ports needed
      │
      ├── P17: deep-thought
      ├── P18: napster  
      ├── P19-22: Reserved Proxmox
      └── P1-16: Access ports TBD
```

## TODO

### Phase 1: ONTI Trunk (Priority)
- [ ] Configure lan4 as VLAN trunk (tagged: 100,101,110-112,120,130,131,140,150,200-240,254)
- [ ] Configure lan8 as VLAN trunk to Zyxel
- [ ] Test: ping from OPNsense VLAN interfaces through ONTI

### Phase 2: Zyxel Trunk + Access
- [ ] Configure lan26 as VLAN trunk from ONTI
- [ ] Configure P17-22 as VLAN 254 access (Proxmox hosts)
- [ ] Configure P1-16 access ports per device needs

### Phase 3: Verification
- [ ] Test VLAN isolation (client on 100 cannot reach 200)
- [ ] Test inter-VLAN routing via OPNsense
- [ ] Document final port assignments

## Blocked Items

| Item | Reason | Workaround |
|------|--------|------------|
| LACP/LAG | OpenWRT RTL838x/RTL930x no HW offload | Single 1Gbps link |
| P7↔P25 fiber | rx_los error, cause unknown | Use P8↔P26 only |

## Reference

OpenWRT VLAN config location: `/etc/config/network`

Example trunk port (conceptual):
```
config bridge-vlan
    option device 'br-lan'
    option vlan '254'
    list ports 'lan4:t'
    list ports 'lan8:t'
```

## Related
- [[progress/2025-01-03-opnsense-vlan-implementation]]
- [[homelab/network/vlan-design]]
