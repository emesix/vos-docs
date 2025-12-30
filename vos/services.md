---
title: VOS Services
description: Authoritative service assignments - what runs where
published: true
date: 2025-12-29T12:30:00.000Z
tags: vos, services, authoritative
editor: markdown
dateCreated: 2025-12-29T12:30:00.000Z
---

# VOS Services

Authoritative service placement. If it's not here, it doesn't exist.

## Control Plane (skynet)

AI orchestration and routing. No heavy inference.

| Service | IP | Purpose |
|---------|-----|---------|
| Open WebUI | <LAN_IP> | Chat interface |
| LiteLLM | <LAN_IP> | LLM router/proxy |
| Haystack | <LAN_IP> | RAG pipeline |
| SearxNG | <LAN_IP> | Search aggregator |

## Data Plane

### deep-thought (Databases)

| Service | IP | Purpose |
|---------|-----|---------|
| PostgreSQL | <LAN_IP> | Primary database |
| Redis | <LAN_IP> | Cache/sessions |
| Wiki.js | <LAN_IP> | Documentation |
| Gitea | <LAN_IP> | Git server |

### alexandria (Storage)

| Service | IP | Purpose |
|---------|-----|---------|
| NFS | <LAN_IP> | Backend storage |
| SMB | <LAN_IP> | Frontend shares |

## Compute Plane

### watson (GPU Inference)

| Service | IP | Purpose |
|---------|-----|---------|
| vLLM | <LAN_IP> | LLM inference |

### mothership (Containers)

| Service | IP | Purpose |
|---------|-----|---------|
| Traefik | <LAN_IP> | Reverse proxy |
| Authentik | <LAN_IP> | Identity/SSO |
| MCP Gateway | <LAN_IP> | MCP server proxy |

### napster (Media)

| Service | IP | Purpose |
|---------|-----|---------|
| Radarr | <LAN_IP> | Movies |
| Sonarr | <LAN_IP> | TV Shows |
| Lidarr | <LAN_IP> | Music |
| Prowlarr | <LAN_IP> | Indexers |
| qBittorrent | <LAN_IP> | Torrent |
| SABnzbd | <LAN_IP> | Usenet |
| Jellyfin | <LAN_IP> | Media server |

## Infrastructure (hal)

| Service | IP | Purpose |
|---------|-----|---------|
| OPNsense | <LAN_IP> | Firewall/Router |
| DHCP | <LAN_IP> | Address assignment |
| DNS (Unbound) | <LAN_IP> | Local resolution |

## Placement Rules

1. **skynet** routes, never computes heavy loads
2. **watson** handles all GPU inference
3. **mothership** runs stateless containers
4. **deep-thought** owns all databases
5. **napster** is media-only
6. **alexandria** is storage-only

## Related

- [[vos/README]]
- [[vos/authority]]
- [[homelab/network/ip-plan]]
