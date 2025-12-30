---
title: IP Address Plan
description: Authoritative IP assignments for all homelab infrastructure
published: true
date: 2025-12-29T12:00:00.000Z
tags: network, infrastructure, ip-addressing, vlan
editor: markdown
dateCreated: 2025-12-26T08:33:22.465Z
---

# IP Address Plan

Authoritative IP assignments. All physical infrastructure on VLAN 254.

## VLAN 254 - Management

### Gateway & Network

| IP | Device | Role |
|----|--------|------|
| <LAN_IP> | Gateway | Routing only (NO WebUI) |
| <LAN_IP> | Zyxel GS1900-24HP | Frontend switch |
| <LAN_IP> | ONTI S508CL-8S | Backend 10G |
| <LAN_IP> | Tenda TEM2007 | Backend 2.5G |
| <LAN_IP> | OPNsense WebUI | Admin interface |

### WiFi Access Points

| IP | Device | Location |
|----|--------|----------|
| <LAN_IP> | RT-2980 #1 | Living Room |
| <LAN_IP> | RT-2980 #2 | Office |
| <LAN_IP> | RT-2980 #3 | Bedroom |
| <LAN_IP> | RT-2980 #4 | Basement (WireGuard) |
| <LAN_IP> | RT-2980 #5 | Garage (WireGuard) |
| <LAN_IP> | CPE510 TX | Balcony |
| <LAN_IP> | CPE510 RX | Garage |

### Proxmox Hosts - Frontend

| IP | Hostname | Nickname |
|----|----------|----------|
| <LAN_IP> | pve-qotom01 | [[hal]] |
| <LAN_IP> | pve-hx310-db | [[deep-thought]] |
| <LAN_IP> | pve-hx310-arr | [[napster]] |
| <LAN_IP> | pve-5800x | [[mothership]] |
| <LAN_IP> | pve-8845hs | [[skynet]] |
| <LAN_IP> | pve-x2686-x2 | [[watson]] |
| <LAN_IP> | nas-unraid | [[alexandria]] |

### Proxmox Hosts - Backend

| IP | Hostname | Connection |
|----|----------|------------|
| <LAN_IP> | pve-qotom01 | ONTI P1 (SFP+) |
| <LAN_IP> | pve-hx310-db | Tenda P1 |
| <LAN_IP> | pve-hx310-arr | Tenda P2 |
| <LAN_IP> | pve-5800x | ONTI P3 (DAC) |
| <LAN_IP> | pve-8845hs | Tenda P3 |
| <LAN_IP> | pve-x2686-x2 | ONTI P5 (AOC) |
| <LAN_IP> | nas-unraid | ONTI P6 (AOC) |

### Identity Infrastructure

| IP | Service | Purpose |
|----|---------|---------|
| <LAN_IP> | LDAP Outpost | Linux/NFS auth |
| <LAN_IP> | RADIUS Outpost | WiFi/VPN auth |

## Frontend VLANs (100-150)

| VLAN | Subnet | Name | Internet |
|------|--------|------|----------|
| 100 | <LAN_IP>/24 | Quarantine | 22/80/443 only |
| 101 | <LAN_IP>/24 | Privileged | Firewalled |
| 110 | <LAN_IP>/24 | Guest WiFi | Captive portal |
| 111 | <LAN_IP>/24 | Home WiFi | Firewalled |
| 112 | <LAN_IP>/24 | IoT | **BLOCKED** |
| 120 | <LAN_IP>/24 | Work | Firewalled |
| 130 | <LAN_IP>/24 | Basement | WireGuard |
| 131 | <LAN_IP>/24 | Garage | WireGuard |
| 140 | <LAN_IP>/24 | VvE | **BLOCKED** |
| 150 | <LAN_IP>/24 | Reverse Proxy | Outbound only |

### VLAN 150 Services

| IP | Service | Host |
|----|---------|------|
| <LAN_IP> | Traefik | mothership |
| <LAN_IP> | Authentik | mothership |

## Backend VLANs (200-250) - NO INTERNET

| VLAN | Subnet | Name | Primary Host |
|------|--------|------|--------------|
| 200 | <LAN_IP>/24 | AI | skynet, watson |
| 210 | <LAN_IP>/24 | ARR | napster |
| 220 | <LAN_IP>/24 | Database | deep-thought |
| 230 | <LAN_IP>/24 | Storage | alexandria |
| 240 | <LAN_IP>/24 | Docker | mothership |

### VLAN 200 - AI Services

| IP | Service | Host |
|----|---------|------|
| <LAN_IP> | Open WebUI | skynet |
| <LAN_IP> | LiteLLM | skynet |
| <LAN_IP> | vLLM | watson |
| <LAN_IP> | Haystack | skynet |
| <LAN_IP> | SearxNG | skynet |
| <LAN_IP> | MCP Gateway | mothership |

### VLAN 210 - ARR Stack

| IP | Service |
|----|---------|
| <LAN_IP> | Radarr |
| <LAN_IP> | Sonarr |
| <LAN_IP> | Lidarr |
| <LAN_IP> | Prowlarr |
| <LAN_IP> | qBittorrent |
| <LAN_IP> | SABnzbd |
| <LAN_IP> | Jellyfin |

### VLAN 220 - Database

| IP | Service |
|----|---------|
| <LAN_IP> | PostgreSQL |
| <LAN_IP> | Redis |
| <LAN_IP> | Wiki.js |
| <LAN_IP> | Gitea |

### VLAN 230 - Storage

| IP | Service |
|----|---------|
| <LAN_IP> | NFS Server |

## DHCP Ranges

All VLANs: Static .1-.50, DHCP .100-.250  
VLAN 254: Static only (no DHCP)

## Related

- [[homelab/network/vlan-design]]
- [[homelab/network/architecture]]
- [[homelab/hosts]]
