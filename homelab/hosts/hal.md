---
title: hal - OPNsense Router
description: Qotom Q20331G9 1U running OPNsense firewall with 4x 10G SFP+ and 5x 2.5GbE
published: true
date: 2025-12-29T12:00:00.000Z
tags: host, proxmox, qotom, firewall, opnsense, 10gbe, router
editor: markdown
dateCreated: 2025-12-26T08:33:41.776Z
---

# hal - OPNsense Router

**Role:** Edge firewall, routing, VLAN management  
**Hostname:** pve-qotom01  
**Frontend IP:** <LAN_IP>  
**Backend IP:** <LAN_IP>

## Overview

Qotom 1U rackmount with Intel Atom C3758R. Primary firewall running OPNsense VM with full NIC passthrough. 4x 10G SFP+ for backend, 5x 2.5GbE for frontend. Central DHCP, DNS, and VLAN routing.

## Hardware

| Component | Specification |
|-----------|---------------|
| Model | Qotom Q20331G9 1U |
| CPU | Intel Atom C3758R (8C/8T @ 2.4GHz) |
| RAM | 64GB DDR4 |
| 10GbE | 4x SFP+ (built-in) |
| 2.5GbE | 5x Intel I226-V (built-in) |

## Network

| Interface | Switch | Speed | Purpose |
|-----------|--------|-------|---------|
| WAN | Ziggo | 1GbE | Internet uplink |
| SFP 1+2 | Zyxel P25-26 | 1GbE LACP | Frontend trunk |
| SFP+ 1 | ONTI P1 | 10GbE | Backend trunk |

## Services

| Service | Type | Purpose |
|---------|------|---------|
| OPNsense | VM | Firewall, DHCP, DNS, RADIUS |

## Related

- [[homelab/hosts]]
- [[homelab/network/architecture]]
- [[howto/pci-passthrough-proxmox]]
