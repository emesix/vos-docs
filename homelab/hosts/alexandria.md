---
title: alexandria - NAS Storage
description: Unraid NAS with 117TB storage (8x16TB) providing NFS and SMB shares
published: true
date: 2025-12-29T12:00:00.000Z
tags: host, unraid, nas, storage, nfs, smb, 10gbe
editor: markdown
dateCreated: 2025-12-26T08:35:06.654Z
---

# alexandria - NAS Storage

**Role:** Primary NAS, NFS/SMB storage  
**Hostname:** nas-unraid  
**Frontend IP:** <LAN_IP>  
**Backend IP:** <LAN_IP>

## Overview

Unraid NAS with AMD Ryzen 5 3600 and 117TB raw storage (8x16TB). Provides NFS for backend services and SMB for frontend clients. 10GbE backend via Intel X520-DA1.

## Hardware

| Component | Specification |
|-----------|---------------|
| Platform | Custom build |
| CPU | AMD Ryzen 5 3600 (6C/12T @ 3.6GHz) |
| RAM | 32GB DDR4 |
| Array | 8x 16TB (117TB raw) |
| Cache | 2x 1TB NVMe |
| 10GbE | Intel X520-DA1 (SFP+) |

## Network

| Interface | Switch | Speed | Purpose |
|-----------|--------|-------|---------|
| eth0 | Zyxel | 1GbE | Frontend (SMB) |
| X520 SFP+ | ONTI P6 | 10GbE | Backend (NFS) |

## Storage Layout

| Share | Protocol | Zone | Access |
|-------|----------|------|--------|
| /mnt/user/media | NFS | Backend | arr-writers, media-readers |
| /mnt/user/downloads | NFS | Backend | arr-downloaders |
| /mnt/user/ai | NFS | Backend | ai-writers, ai-readers |
| /mnt/user/backups | NFS | Backend | backup-writers |
| Media | SMB | Frontend | Authenticated users |

## Services (VLAN 230)

| Service | IP | Purpose |
|---------|-----|---------|
| NFS Server | <LAN_IP> | Primary NFS exports |

## Related

- [[homelab/hosts]]
- [[homelab/network/ip-plan]]
- [[homelab/identity/nfs-permissions]]
