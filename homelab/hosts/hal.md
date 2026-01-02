---
title: hal - OPNsense Router
description: Qotom Q20331G9 1U running OPNsense firewall with 4x 10G SFP+ and 5x 2.5GbE
published: true
date: 2025-01-02T14:45:00.000Z
tags: host, qotom, firewall, opnsense, 10gbe, router
editor: markdown
dateCreated: 2025-12-26T08:33:41.776Z
---

# hal - OPNsense Router

**Role:** Edge firewall, routing, VLAN management  
**Hostname:** OPNsense.localdomain  
**Management IP:** 192.168.2.1  
**OS:** OPNsense 24.7 "Thriving Tiger" (FreeBSD 14.1-RELEASE-p2)

## Overview

Qotom 1U rackmount with Intel Atom C3758R. Primary firewall running OPNsense with direct hardware access. 4x 10G SFP+ (Intel X553) for high-speed connectivity, 5x 2.5GbE (Intel I226-V) for standard networking, plus 1x virtual interface for Proxmox bridge.

## Hardware

| Component | Specification |
|-----------|---------------|
| Model | Qotom Q20331G9 1U |
| CPU | Intel Atom C3758R (8C/8T @ 2.4GHz) |
| RAM | 4 GB (assigned to OPNsense) |
| 10GbE | 4x SFP+ Intel X553 (ix driver) |
| 2.5GbE | 5x Intel I226-V (igc driver) |
| Virtual | 1x VirtIO (vtnet driver) |

## Network Interfaces

### Physical Port Mapping

Based on chassis layout (left to right, as viewed from rear):

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  12V   CONSOLE  USB3   ETH5   ETH3   ETH4   SFP3   SFP1   WiFi   WiFi      │
│   ⊚      ▣       ▣      ▣      ▣      ▣     ▣ ▣    ▣ ▣     ◯      ◯       │
│                         ▣      ▣      ▣     ▣ ▣    ▣ ▣                     │
│                        ETH1   ETH2   ETH2   SFP4   SFP2                     │
└─────────────────────────────────────────────────────────────────────────────┘
```

### SFP+ Ports (10GbE - Intel X553)

| Port | FreeBSD | MAC Address | Driver | PCI | Status | OPNsense Role |
|------|---------|-------------|--------|-----|--------|---------------|
| **S1** | ix1 | `20:7c:14:f4:78:77` | ix | pci0:2:0:0 | **ACTIVE** | **WAN** (62.163.217.49/24) |
| **S2** | ix0 | `20:7c:14:f4:78:76` | ix | pci0:1:0:0 | **ACTIVE** | **LAN** (192.168.2.1/24) |
| **S3** | ix2 | `20:7c:14:f4:78:78` | ix | pci0:3:0:0 | no carrier | Unassigned |
| **S4** | ix3 | `20:7c:14:f4:78:79` | ix | pci0:4:0:0 | no carrier | Unassigned |

### Ethernet Ports (2.5GbE - Intel I226-V)

| Port | FreeBSD | MAC Address | Driver | PCI | Status | OPNsense Role |
|------|---------|-------------|--------|-----|--------|---------------|
| **1** | igc0 | `20:7c:14:f4:78:71` | igc | pci0:5:0:0 | no carrier | Unassigned |
| **2** | igc1 | `20:7c:14:f4:78:72` | igc | pci0:6:0:0 | no carrier | Unassigned |
| **3** | igc2 | `20:7c:14:f4:78:73` | igc | pci0:7:0:0 | **ACTIVE** | (1GbE link up) |
| **4** | igc3 | `20:7c:14:f4:78:74` | igc | pci0:8:0:0 | no carrier | Unassigned |
| **5** | - | - | igc | - | **Proxmox** | Proxmox management (not passed through) |

> **Note:** Port 5 remains on Proxmox host for management. OPNsense connects to it via VirtualBridge (vtnet0 ↔ vmbr0).

### Virtual Interface

| Port | FreeBSD | MAC Address | Driver | Status | OPNsense Role |
|------|---------|-------------|--------|--------|---------------|
| **VIRT** | vtnet0 | `bc:24:11:88:c0:47` | vtnet | **ACTIVE** | **VirtualBridge** (192.168.254.254/24) |

> **Architecture:** vtnet0 connects to Proxmox vmbr0 bridge, which is attached to physical ETH5. This provides OPNsense access to the Proxmox management network without passing ETH5 through to the VM.

## Current Configuration

| Interface | Device | IPv4 | IPv6 | Purpose |
|-----------|--------|------|------|---------|
| **LAN** | ix0 (S2) | 192.168.2.1/24 | 2001:1c00:e00d:eb00::/56 | Local network |
| **WAN** | ix1 (S1) | 62.163.217.49/24 (DHCP) | 2001:1c00:e000::/128 | Internet uplink |
| **VirtualBridge** | vtnet0 | 192.168.254.254/24 | - | Proxmox management |

## Access

| Method | Address | Credentials |
|--------|---------|-------------|
| HTTPS | https://192.168.2.1 | root / NikonD90 |
| SSH | root@192.168.2.1:22 | root / NikonD90 |
| API | https://192.168.2.1/api | See API credentials |

### API Credentials

```
Key:    ucOl17sUVgGGxLUkmqlpYUNY5RYlPlxrliyy4Rm3R28zo/18Ny6yoHQ2sSvveiintsk7bwhGyvA+UJ5t
Secret: MeFjr/FrdCK2joLY9lEh72ps2odkOx22XalYSNco47ktuaP7DSKmtpn+5gpdX+Pde/5NrHRoGXoKv9NY
```

### SSH Fingerprints

```
ECDSA:   SHA256 7W3LEUrhYdy8R0IGEDmhywIMBvZChaEwp/Mq8ZruOKc
ED25519: SHA256 LMu2R6Qf/D3iuVqGb5CWxFRcUnTaytYTvsHOGxnYjA8
RSA:     SHA256 VW5mne5VaUaePcxSMJW2cs9KJxWuK055q3bO4V0sKkA
```

### HTTPS Certificate

```
SHA256: C9 2A 49 AE D2 3A 5F 1C 9A 7C 36 3F 38 E5 7E 52
        B8 49 2F D9 CD 63 D2 85 D8 B2 D9 EF 9B 35 44 9A
```

## Services

| Service | Type | Purpose |
|---------|------|---------|
| OPNsense | Firewall | Stateful firewall, NAT |
| DHCP | Service | IP assignment for LAN |
| DNS | Service | Local DNS resolution |
| SSH | Service | Remote management |

## MCP Integration

Claude Desktop MCP server configured at:
- Config: `~/.config/Claude/claude_desktop_config.json`
- Host: `https://192.168.2.1`
- SSL Verify: `false`

## Related

- [[homelab/hosts]]
- [[homelab/network/architecture]]
- [[components/nic-intel-x553]]
- [[components/nic-intel-i226-v]]
- [[howto/pci-passthrough-proxmox]]
