---
title: deep-thought - Core Services
description: OnLogic HX310 running PostgreSQL, Wiki.js, Gitea and core database services
published: true
date: 2025-12-29T12:00:00.000Z
tags: host, proxmox, hx310, database, wiki, gitea
editor: markdown
dateCreated: 2025-12-26T08:34:16.123Z
---

# deep-thought - Core Services

**Role:** Database services, Wiki.js, Gitea  
**Hostname:** pve-hx310-db  
**Frontend IP:** <LAN_IP>  
**Backend IP:** <LAN_IP>

## Overview

OnLogic HX310 fanless mini-PC. Runs core infrastructure services: PostgreSQL, Redis, Wiki.js, Gitea. Backend via Tenda 2.5GbE bridge.

## Hardware

| Component | Specification |
|-----------|---------------|
| Model | OnLogic HX310 |
| CPU | Intel J6426 (4C @ 2.0GHz) |
| RAM | 32GB DDR4 |
| Storage | 512GB NVMe |
| Network | 2x 1GbE Intel I226-V |

## Network

| Interface | Switch | Speed | Purpose |
|-----------|--------|-------|---------|
| eth0 | Zyxel P1 | 1GbE | Frontend |
| eth1 | Tenda P1 | 1GbE | Backend |

## Services (VLAN 220)

| Service | IP | Purpose |
|---------|-----|---------|
| PostgreSQL | <LAN_IP> | Primary database |
| Redis | <LAN_IP> | Cache/sessions |
| Wiki.js | <LAN_IP> | Documentation |
| Gitea | <LAN_IP> | Git server |

## Related

- [[homelab/hosts]]
- [[homelab/network/ip-plan]]
