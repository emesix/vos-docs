---
title: napster - ARR Stack
description: OnLogic HX310 running Radarr, Sonarr, Lidarr, Prowlarr, Jellyfin media automation
published: true
date: 2025-12-29T12:00:00.000Z
tags: host, proxmox, hx310, arr, media, jellyfin
editor: markdown
dateCreated: 2025-12-26T08:34:26.456Z
---

# napster - ARR Stack

**Role:** Media automation (*ARR stack)  
**Hostname:** pve-hx310-arr  
**Frontend IP:** <LAN_IP>  
**Backend IP:** <LAN_IP>

## Overview

OnLogic HX310 fanless mini-PC dedicated to media automation. Runs complete *ARR stack plus Jellyfin media server. Backend via Tenda 2.5GbE bridge.

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
| eth0 | Zyxel P2 | 1GbE | Frontend |
| eth1 | Tenda P2 | 1GbE | Backend |

## Services (VLAN 210)

| Service | IP | Purpose |
|---------|-----|---------|
| Radarr | <LAN_IP> | Movies |
| Sonarr | <LAN_IP> | TV Shows |
| Lidarr | <LAN_IP> | Music |
| Prowlarr | <LAN_IP> | Indexer proxy |
| qBittorrent | <LAN_IP> | Torrent client |
| SABnzbd | <LAN_IP> | Usenet client |
| Jellyfin | <LAN_IP> | Media server |

## Related

- [[homelab/hosts]]
- [[homelab/network/ip-plan]]
- [[alexandria]] (NFS storage)
