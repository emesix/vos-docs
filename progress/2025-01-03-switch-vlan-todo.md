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
**Status:** Not started  
**Depends on:** OPNsense VLAN implementation (DONE)

## Current State

OPNsense has 15 VLANs configured on ix1. Switches don't know about them yet.

```
OPNsense ix1 (trunk, all VLANs)
      │
      │ DAC 10GbE
      ▼
ONTI lan4 ← Currently: untagged VLAN 1 only
      │
      ├── lan1 → Tenda (not configured)
      └── lan8 → Zyxel (not configured)
```

## Target State

```
OPNsense ix1 (trunk: 100-150, 200-240, 254)
      │
      ▼
ONTI lan4 (trunk: all VLANs)
      │
      ├── lan1 → Tenda (trunk: backend 200-240, 254)
      ├── lan3 → mothership (untagged: 240)
      ├── lan5 → watson (untagged: 200)
      ├── lan6 → alexandria (untagged: 230)
      └── lan8 → Zyxel (trunk: frontend 100-150, 254)
                    │
                    ├── P1-2 → deep-thought, napster (untagged: 254)
                    ├── P3-5 → APs (trunk: 110,111,112)
                    └── P8-10 → Clients (untagged: 101)
```

## ONTI S508CL-8S Configuration

SSH: `ssh root@192.168.254.3`

### Step 1: Backup Current Config

```bash
# On iZombie
scp root@192.168.254.3:/etc/config/network /tmp/onti-network-backup
scp root@192.168.254.3:/etc/config/system /tmp/onti-system-backup
```

### Step 2: Create VLANs via UCI

```bash
ssh root@192.168.254.3

# Frontend VLANs
for vid in 100 101 110 111 112 120 130 131 140 150; do
  uci add network switch_vlan
  uci set network.@switch_vlan[-1].device='switch0'
  uci set network.@switch_vlan[-1].vlan="$vid"
  uci set network.@switch_vlan[-1].vid="$vid"
done

# Backend VLANs
for vid in 200 210 220 230 240; do
  uci add network switch_vlan
  uci set network.@switch_vlan[-1].device='switch0'
  uci set network.@switch_vlan[-1].vlan="$vid"
  uci set network.@switch_vlan[-1].vid="$vid"
done

# Management VLAN
uci add network switch_vlan
uci set network.@switch_vlan[-1].device='switch0'
uci set network.@switch_vlan[-1].vlan='254'
uci set network.@switch_vlan[-1].vid='254'

uci commit network
```

### Step 3: Assign Ports

ONTI port mapping (from previous session):
- lan1 = port 0 (CPU)
- lan2-8 = ports 1-7

```bash
# Example: lan4 (port 3) as trunk to OPNsense
# All VLANs tagged

# This needs more research on ONTI's switch_vlan syntax
# OpenWrt DSA vs swconfig differences
```

**TODO:** Research exact UCI commands for ONTI's RTL9303 switch.

### Step 4: Test

```bash
# From OPNsense, ping ONTI on each VLAN
ping 192.168.254.3  # VLAN 254 (should already work)

# After trunk config:
# Tagged frames should pass through
```

## Zyxel GS1900-24HPv1 Configuration

SSH: `ssh root@192.168.254.2`

### Step 1: Backup

```bash
scp root@192.168.254.2:/etc/config/network /tmp/zyxel-network-backup
```

### Step 2: Create VLANs

```bash
ssh root@192.168.254.2

# Frontend VLANs only (100-150)
for vid in 100 101 110 111 112 120 130 131 140 150; do
  uci add network switch_vlan
  uci set network.@switch_vlan[-1].device='switch0'
  uci set network.@switch_vlan[-1].vlan="$vid"
  uci set network.@switch_vlan[-1].vid="$vid"
done

# Management VLAN
uci add network switch_vlan
uci set network.@switch_vlan[-1].device='switch0'
uci set network.@switch_vlan[-1].vlan='254'
uci set network.@switch_vlan[-1].vid='254'

uci commit network
```

### Step 3: Port Assignments

| Port | Device | VLAN Config |
|------|--------|-------------|
| P1 | deep-thought | Untagged 254 |
| P2 | napster | Untagged 254 |
| P3 | AP Living | Tagged 110,111,112 |
| P4 | AP Office | Tagged 110,111,112 |
| P5 | AP Bedroom | Tagged 110,111,112 |
| P8 | iZombie | Untagged 101 |
| P9 | X300 | Untagged 101 |
| P25-26 | (future LACP) | Trunk all |

### Step 4: Uplink to ONTI

Port connecting to ONTI (via SFP) needs trunk of 100-150, 254.

## Testing Plan

### Test 1: VLAN 254 Still Works

After any change, verify management access:
```bash
ping 192.168.254.2  # Zyxel
ping 192.168.254.3  # ONTI
```

### Test 2: New VLAN Reachable

Connect device to untagged port, should get DHCP:
```bash
# Device on VLAN 101 port
ip addr  # Should show 192.168.101.x
ping 192.168.101.1  # Gateway
```

### Test 3: VLAN Isolation

From VLAN 101, should NOT reach VLAN 200:
```bash
ping 192.168.200.1  # Should fail (firewall blocks)
```

## Rollback

If switch becomes unreachable:

1. Console cable to switch (if available)
2. Or: Factory reset button
3. Or: OPNsense still accessible, reconfigure via VLAN 254

## Notes

- ONTI runs OpenWrt SNAPSHOT r31056 - don't upgrade, LUCI broken in newer
- Zyxel runs OpenWrt 24.10.1 stable
- Both use DSA (not swconfig) - syntax may differ from old tutorials
- VLAN 1 is default untagged on all ports initially

## Related

- [[progress/2025-01-03-opnsense-vlan-implementation]]
- [[reference/vlan-quickref]]
- [[homelab/network/topology]]
