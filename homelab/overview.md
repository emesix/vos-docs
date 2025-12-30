---
title: Homelab Infrastructure Overview
description: Single Source of Truth for VOS homelab infrastructure
published: true
date: 2025-12-29T12:00:00.000Z
tags: vos, infrastructure, proxmox, homelab, ssot
editor: markdown
dateCreated: 2025-12-26T08:32:54.414Z
---

# Homelab Infrastructure Overview

Single Source of Truth for VOS homelab infrastructure.

## Compute Summary

| Metric | Total |
|--------|-------|
| Cores | 74 |
| Threads | 132 |
| RAM | 384GB |
| 10GbE Hosts | 6 |
| NAS Storage | 117TB |

## Cluster Nodes

| Nickname | Hostname | IP | Role |
|----------|----------|-----|------|
| [[hal]] | pve-qotom01 | <LAN_IP> | OPNsense router |
| [[deep-thought]] | pve-hx310-db | <LAN_IP> | Core services |
| [[napster]] | pve-hx310-arr | <LAN_IP> | ARR stack |
| [[mothership]] | pve-5800x | <LAN_IP> | Docker worker |
| [[skynet]] | pve-8845hs | <LAN_IP> | AI controller |
| [[watson]] | pve-x2686-x2 | <LAN_IP> | AI worker (GPU) |
| [[alexandria]] | nas-unraid | <LAN_IP> | NAS (117TB) |

→ Host details: [[homelab/hosts]]

## Network Architecture

Dual-zone design: Frontend (clients, internet) and Backend (services, air-gapped).

| Zone | VLANs | Internet | Storage |
|------|-------|----------|---------|
| Frontend | 100-150 | Firewalled | SMB |
| Backend | 200-250 | **BLOCKED** | NFS |
| Management | 254 | Admin only | - |

→ Network details: [[homelab/network/architecture]]  
→ IP assignments: [[homelab/network/ip-plan]]  
→ VLAN design: [[homelab/network/vlan-design]]

## WiFi Mesh

5x EDUP RT-2980 with OpenWRT. 802.11r fast roaming.

| Location | Backhaul |
|----------|----------|
| Indoor (3x) | Wired |
| Basement | WireGuard |
| Garage | WireGuard via CPE510 |

→ OpenWRT devices: [[components/openwrt-device-compatibility]]

## Identity

Authentik provides centralized identity: LDAP, RADIUS, OIDC.

→ Identity overview: [[homelab/identity]]

## Current Phase

Pre-staging complete. Network infrastructure deployment in progress.

→ VOS Architecture: [[vos/architecture]]  
→ Development environment: [[vos/development/overview]]
