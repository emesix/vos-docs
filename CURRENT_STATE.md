# VOS Current State

**Updated:** 2025-01-03 09:00 CET
**Phase:** 1 - Network Foundation (60%)

## What Works RIGHT NOW

### OPNsense (hal)
- WebUI: https://192.168.254.254
- SSH: root@192.168.254.254
- API: Configured (see staging-credentials.md)
- WAN: igc0 → Ziggo (DHCP)
- MGMT: igc2 → 192.168.254.1
- Trunk: ix1 → all 15 VLANs configured
- DHCP: Running on all VLANs (.100-.250)
- Firewall: PERMISSIVE (allow all for testing)

### Switches
| Device | IP | Status | OpenWRT |
|--------|-----|--------|---------|
| Zyxel GS1900-24HP | 192.168.254.2 | UP | 24.10.1 |
| ONTI S508CL-8S | 192.168.254.3 | UP | SNAPSHOT r31056 |
| Tenda TEM2007 | .4 (pending) | Connected | N/A |

### Fiber Link
- ONTI P8 ↔ Zyxel P26: **1Gbps UP** (BiDi SFP)
- ONTI P7 ↔ Zyxel P25: **DOWN** (rx_los, abandoned)
- LACP: **NOT POSSIBLE** (OpenWRT RTL838x has no HW offload)

### VLANs (all on ix1 trunk)
Frontend: 100, 101, 110, 111, 112, 120, 130, 131, 140, 150
Backend: 200, 210, 220, 230, 240
Management: 254

## What's Next

1. Configure VLAN trunking on Zyxel (P26 trunk, access ports per device)
2. Configure VLAN trunking on ONTI (lan4 trunk to OPNsense)
3. Connect Proxmox hosts to correct VLANs
4. Harden firewall rules (zone isolation)

## Quick Access

```bash
# OPNsense
ssh root@192.168.254.254

# Zyxel (OpenWRT)
ssh root@192.168.254.2

# ONTI (OpenWRT)  
ssh root@192.168.254.3
```

## Blocked

- watson: Offline (hardware issue?)
- LACP: Hardware limitation, use single 1G link
- Second BiDi fiber: rx_los on P7/P25, not investigated further
