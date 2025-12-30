---
title: mothership - Docker/MCP Gateway
description: B450M with AMD Ryzen 7 5800X running Docker workloads, MCP Gateway, and Traefik
published: true
date: 2025-12-29T12:00:00.000Z
tags: host, proxmox, amd, docker, mcp, traefik, 10gbe
editor: markdown
dateCreated: 2025-12-26T08:34:36.762Z
---

# mothership - Docker/MCP Gateway

**Role:** Docker worker, MCP Gateway, Traefik reverse proxy  
**Hostname:** pve-5800x  
**Frontend IP:** <LAN_IP>  
**Backend IP:** <LAN_IP>

## Overview

ASRock B450M Pro4 with AMD Ryzen 7 5800X. Heavy Docker workloads, MCP Gateway for multi-client AI access, Traefik reverse proxy. 10GbE backend via Intel X520-DA1.

## Hardware

| Component | Specification |
|-----------|---------------|
| Motherboard | ASRock B450M Pro4 |
| CPU | AMD Ryzen 7 5800X (8C/16T @ 3.8-4.7GHz) |
| RAM | 32GB DDR4 |
| Boot | Samsung 960 PRO 512GB NVMe |
| Data | 4x 1TB NVMe RAID0 (via PCIe bifurcation) |
| 10GbE | Intel X520-DA1 (PCIe SFP+) |

## Network

| Interface | Switch | Speed | Purpose |
|-----------|--------|-------|---------|
| RTL8111 | Zyxel | 1GbE | Frontend |
| X520 SFP+ | ONTI P3 | 10GbE | Backend (DAC) |

## Services

**VLAN 150 (Proxy):**

| Service | IP | Purpose |
|---------|-----|---------|
| Traefik | <LAN_IP> | Reverse proxy |
| Authentik | <LAN_IP> | Identity/SSO |

**VLAN 200 (AI):**

| Service | IP | Purpose |
|---------|-----|---------|
| MCP Gateway | <LAN_IP> | MCP server proxy |

**VLAN 240 (Docker):**

| Service | IP | Purpose |
|---------|-----|---------|
| Docker Bridge | <LAN_IP> | Container networks |

## Bifurcation Note

Upgraded from 5700G to 5800X. The 5700G APU could not bifurcate PCIe x16 to x4x4x4x4. The 5800X supports bifurcation, enabling 4x NVMe via single adapter.

## Related

- [[homelab/hosts]]
- [[homelab/network/ip-plan]]
- [[components/nic-intel-x553]]
