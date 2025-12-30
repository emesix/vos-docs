# VOS Network Implementation Guide

**The Complete "Dummies Guide" to Building user's Operating System Network**

---

```
Document:     VOS Network Implementation Guide
Version:      1.0
Created:      2025-12-28
Purpose:      Single Source of Truth for network implementation
Audience:     user (human) and Claude (AI) - both dummies
Status:       AUTHORITATIVE - Follow this, not memory
```

---

## Table of Contents

1. [What We're Building](#1-what-were-building)
2. [Hardware Inventory](#2-hardware-inventory)
3. [Network Architecture](#3-network-architecture)
4. [VLAN Reference](#4-vlan-reference)
5. [IP Address Plan](#5-ip-address-plan)
6. [Identity Management](#6-identity-management)
7. [Users and Groups](#7-users-and-groups)
8. [Service Accounts](#8-service-accounts)
9. [DNS Configuration](#9-dns-configuration)
10. [SSL Certificates](#10-ssl-certificates)
11. [Firewall Rules](#11-firewall-rules)
12. [WiFi Configuration](#12-wifi-configuration)
13. [NFS Storage Permissions](#13-nfs-storage-permissions)
14. [Implementation Order](#14-implementation-order)
15. [Verification Checklist](#15-verification-checklist)

---

# 1. What We're Building

## The Goal

A secure, identity-aware homelab network with:

- **Dual-zone architecture**: Frontend (internet) / Backend (air-gapped)
- **Centralized identity**: One login for WiFi, VPN, web apps, Linux
- **VLAN isolation**: Services grouped by function
- **MAC lockdown**: Unknown devices quarantined
- **Trusted SSL**: No certificate warnings

## The Big Picture

```
                            INTERNET
                                │
                                ▼
                        ┌───────────────┐
                        │   OPNsense    │  ← hal (Qotom)
                        │   Firewall    │
                        └───────┬───────┘
                                │
            ┌───────────────────┼───────────────────┐
            │                   │                   │
            ▼                   ▼                   ▼
      ┌──────────┐        ┌──────────┐        ┌──────────┐
      │ FRONTEND │        │ BACKEND  │        │   MGMT   │
      │ 100-150  │        │ 200-250  │        │   254    │
      │          │        │          │        │          │
      │ Internet │        │ NO       │        │ Physical │
      │ Allowed  │        │ Internet │        │ Hosts    │
      └──────────┘        └──────────┘        └──────────┘
```

## Security Zones Summary

| Zone | VLANs | Internet | Storage | Trust Level |
|------|-------|----------|---------|-------------|
| **Frontend** | 100-150 | Firewalled | SMB | Low (untrusted) |
| **Backend** | 200-250 | **BLOCKED** | NFS | High (trusted) |
| **Management** | 254 | Admin only | None | Controlled |

---

# 2. Hardware Inventory

## Compute Hosts

| Hostname | Nickname | CPU | RAM | 10GbE | Role | Frontend IP | Backend IP |
|----------|----------|-----|-----|-------|------|-------------|------------|
| pve-qotom01 | hal | C3758R 8C | 64GB | 4x SFP+ | OPNsense Router | <LAN_IP> | <LAN_IP> |
| pve-hx310-db | deep-thought | J6426 4C | 32GB | No | Core Services/DB | <LAN_IP> | <LAN_IP> |
| pve-hx310-arr | napster | J6426 4C | 32GB | No | ARR Stack | <LAN_IP> | <LAN_IP> |
| pve-5800x | mothership | 5800X 8C/16T | 32GB | X520 SFP+ | Docker/MCP Gateway | <LAN_IP> | <LAN_IP> |
| pve-8845hs | skynet | 8845HS 8C/16T | 64GB | 2x 2.5G | AI Controller | <LAN_IP> | <LAN_IP> |
| pve-x2686-x2 | watson | 2xE5-2686v4 36C | 128GB | X520 | AI Worker (GPU) | <LAN_IP> | <LAN_IP> |
| nas-unraid | alexandria | R5 3600 6C | 32GB | X520 | NAS (117TB) | <LAN_IP> | <LAN_IP> |

**Note:** HX310 units (deep-thought, napster) have 2x 1GbE only. Backend uses 1GbE, not 10GbE.

## Network Switches

| IP | Device | Model | Ports | Role |
|----|--------|-------|-------|------|
| <LAN_IP> | Zyxel | GS1900-24HP | 24x 1G + 2x SFP | Frontend 1GbE + PoE |
| <LAN_IP> | ONTI | S508CL-8S | 8x 10G SFP+ | Backend 10GbE |
| <LAN_IP> | Tenda | TEM2007 | 7x 2.5G + 1x SFP+ | Backend 2.5GbE bridge |

## WiFi Access Points

| IP | Device | Location | Backhaul |
|----|--------|----------|----------|
| <LAN_IP> | RT-2980 #1 | Living Room | Wired (Zyxel P3) |
| <LAN_IP> | RT-2980 #2 | Office | Wired (Zyxel P4) |
| <LAN_IP> | RT-2980 #3 | Bedroom | Wired (Zyxel P5) |
| <LAN_IP> | RT-2980 #4 | Basement | WireGuard tunnel |
| <LAN_IP> | RT-2980 #5 | Garage | WireGuard tunnel |
| <LAN_IP> | CPE510 TX | Balcony | Wired (Zyxel P6) |
| <LAN_IP> | CPE510 RX | Garage | P2P from TX |

---

# 3. Network Architecture

## Dual Interface Design

Every host gets two network connections:

| Interface | Speed | Purpose |
|-----------|-------|---------|
| **Frontend NIC** | 1GbE | WebUI, internet, user-facing traffic |
| **Backend NIC** | 10GbE (or 2.5G/1G) | Inter-service, databases, storage |

## Traffic Flow Rules

| From → To | Allowed? | How |
|-----------|----------|-----|
| Frontend → Backend | Via proxy only | Through VLAN 150 reverse proxy |
| Backend → Frontend | **BLOCKED** | Backend never initiates to frontend |
| Backend → Internet | **BLOCKED** | Air-gapped zone |
| Frontend → Internet | Firewalled | Per-VLAN rules |
| Any → Management | Admin only | Requires privileged access |

## Why This Design?

**Backend = Trusted Zone**
- No internet = no exfiltration
- Service-to-service only
- Compromise contained
- NFS for fast Linux storage

**Frontend = Untrusted Zone**
- Client-facing
- Firewalled internet
- SMB for Windows/Mac
- Reverse proxy for WebUI access

---

# 4. VLAN Reference

## Complete VLAN Table

### Frontend VLANs (100-150) - Firewalled Internet

| VLAN | Subnet | Name | Internet | Purpose |
|------|--------|------|----------|---------|
| 100 | <LAN_IP>/24 | Unprivileged LAN | 22/80/443 only | Unknown MAC, quarantine |
| 101 | <LAN_IP>/24 | Privileged LAN | Firewalled | Known MAC + authenticated |
| 110 | <LAN_IP>/24 | Guest WiFi | Captive portal | Time-limited guests |
| 111 | <LAN_IP>/24 | Home WiFi | Firewalled | Trusted home devices |
| 112 | <LAN_IP>/24 | IoT WiFi | **BLOCKED** | Local only, no internet |
| 120 | <LAN_IP>/24 | Work | Firewalled | Isolated work devices |
| 130 | <LAN_IP>/24 | Basement | WireGuard | Remote site |
| 131 | <LAN_IP>/24 | Garage | WireGuard | Remote site |
| 140 | <LAN_IP>/24 | VvE | Isolated | Cameras, no internet |
| 150 | <LAN_IP>/24 | Reverse Proxy | Firewalled | WebUI bridge to backend |

### Backend VLANs (200-250) - NO INTERNET

| VLAN | Subnet | Name | Purpose | Primary Host |
|------|--------|------|---------|--------------|
| 200 | <LAN_IP>/24 | AI | Inference, embeddings, MCP | skynet, watson |
| 210 | <LAN_IP>/24 | ARR | Media automation | napster |
| 220 | <LAN_IP>/24 | Database | PostgreSQL, Redis | deep-thought |
| 230 | <LAN_IP>/24 | Storage | NFS exports | alexandria |
| 240 | <LAN_IP>/24 | Docker | Container networks | mothership |

### Management VLAN (254)

| Subnet | Name | Purpose |
|--------|------|---------|
| <LAN_IP>/24 | Management | Physical infrastructure only |

## VLAN Access Matrix

```
                    Internet    Backend    Storage    Management
                    Access      Access     (NFS)      Access
                    ─────────   ─────────  ─────────  ─────────
VLAN 100 (Unpriv)      ▪          ✗          ✗          ✗
VLAN 101 (Priv)        ✓          proxy      SMB        ✗
VLAN 110 (Guest)       ▪          ✗          ✗          ✗
VLAN 111 (Home)        ✓          proxy      SMB        ✗
VLAN 112 (IoT)         ✗          ✗          ✗          ✗
VLAN 120 (Work)        ✓          ✗          ✗          ✗
VLAN 150 (Proxy)       ✓          ✓          ✗          ✗
VLAN 200 (AI)          ✗          ✓          NFS        ✗
VLAN 210 (ARR)         ▪          ✓          NFS        ✗
VLAN 220 (DB)          ✗          ✓          NFS        ✗
VLAN 230 (Storage)     ✗          ✓          source     ✗
VLAN 240 (Docker)      ✗          ✓          NFS        ✗
VLAN 254 (Mgmt)        admin      admin      admin      ✓

Legend: ✓ = Full, ▪ = Limited, ✗ = Blocked, proxy = via VLAN 150
```

---

# 5. IP Address Plan

## Management VLAN 254 - Static IPs

### Network Infrastructure

| IP | Device | Role |
|----|--------|------|
| <LAN_IP> | Gateway | Routing only, NO WebUI |
| <LAN_IP> | Zyxel GS1900-24HP | Frontend switch |
| <LAN_IP> | ONTI S508CL-8S | Backend 10GbE switch |
| <LAN_IP> | Tenda TEM2007 | Backend 2.5GbE switch |
| <LAN_IP> | OPNsense WebUI | Admin interface |

### WiFi Access Points

| IP | Device | Location |
|----|--------|----------|
| <LAN_IP> | RT-2980 #1 | Living Room |
| <LAN_IP> | RT-2980 #2 | Office |
| <LAN_IP> | RT-2980 #3 | Bedroom |
| <LAN_IP> | RT-2980 #4 | Basement |
| <LAN_IP> | RT-2980 #5 | Garage |
| <LAN_IP> | CPE510 TX | Balcony |
| <LAN_IP> | CPE510 RX | Garage |

### Proxmox Hosts - Frontend NICs

| IP | Hostname | Nickname |
|----|----------|----------|
| <LAN_IP> | pve-qotom01 | hal |
| <LAN_IP> | pve-hx310-db | deep-thought |
| <LAN_IP> | pve-hx310-arr | napster |
| <LAN_IP> | pve-5800x | mothership |
| <LAN_IP> | pve-8845hs | skynet |
| <LAN_IP> | pve-x2686-x2 | watson |
| <LAN_IP> | nas-unraid | alexandria |

### Proxmox Hosts - Backend NICs

| IP | Hostname | Connection |
|----|----------|------------|
| <LAN_IP> | pve-qotom01 | ONTI P1 (SFP+) |
| <LAN_IP> | pve-hx310-db | Tenda P1 (2.5G) |
| <LAN_IP> | pve-hx310-arr | Tenda P2 (2.5G) |
| <LAN_IP> | pve-5800x | ONTI P3 (DAC) |
| <LAN_IP> | pve-8845hs | Tenda P3 (2.5G) |
| <LAN_IP> | pve-x2686-x2 | ONTI P5 (AOC) |
| <LAN_IP> | nas-unraid | ONTI P6 (AOC) |

### Identity Infrastructure (VLAN 254)

| IP | Service | Purpose |
|----|---------|---------|
| <LAN_IP> | Authentik LDAP Outpost | Linux/NFS authentication |
| <LAN_IP> | Authentik RADIUS Outpost | WiFi/VPN/802.1X |

## Frontend VLAN 150 - Service Frontend

| IP | Service | Host | Purpose |
|----|---------|------|---------|
| <LAN_IP> | Gateway | - | VLAN gateway |
| <LAN_IP> | Traefik | mothership | Reverse proxy |
| <LAN_IP> | Authentik Server | mothership | Core + Web UI |
| <LAN_IP> | Authentik Worker | mothership | Background tasks |

## Backend VLAN 200 - AI Services

| IP | Service | Host | Purpose |
|----|---------|------|---------|
| <LAN_IP> | Gateway | - | VLAN gateway |
| <LAN_IP> | Open WebUI | skynet | Chat interface |
| <LAN_IP> | LiteLLM | skynet | LLM proxy |
| <LAN_IP> | vLLM | watson | Inference engine |
| <LAN_IP> | Haystack | skynet | RAG pipeline |
| <LAN_IP> | SearxNG | skynet | Search aggregator |
| <LAN_IP> | MCP Gateway | mothership | MCP server proxy |

## Backend VLAN 210 - ARR Stack

| IP | Service | Host | Purpose |
|----|---------|------|---------|
| <LAN_IP> | Gateway | - | VLAN gateway |
| <LAN_IP> | Radarr | napster | Movies |
| <LAN_IP> | Sonarr | napster | TV Shows |
| <LAN_IP> | Lidarr | napster | Music |
| <LAN_IP> | Prowlarr | napster | Indexer proxy |
| <LAN_IP> | qBittorrent | napster | Torrent client |
| <LAN_IP> | SABnzbd | napster | Usenet client |
| <LAN_IP> | Jellyfin | napster | Media server |

## Backend VLAN 220 - Database Services

| IP | Service | Host | Purpose |
|----|---------|------|---------|
| <LAN_IP> | Gateway | - | VLAN gateway |
| <LAN_IP> | PostgreSQL | deep-thought | Main database |
| <LAN_IP> | Redis | deep-thought | Cache |
| <LAN_IP> | Valkey | deep-thought | Redis alternative |
| <LAN_IP> | Wiki.js | deep-thought | Documentation |
| <LAN_IP> | Gitea | deep-thought | Git server |

## Backend VLAN 230 - Storage

| IP | Service | Host | Purpose |
|----|---------|------|---------|
| <LAN_IP> | Gateway | - | VLAN gateway |
| <LAN_IP> | NFS Server | alexandria | Primary NFS |

## Backend VLAN 240 - Docker

| IP | Service | Host | Purpose |
|----|---------|------|---------|
| <LAN_IP> | Gateway | - | VLAN gateway |
| <LAN_IP> | Docker Bridge | mothership | Container networks |

## DHCP Ranges

| VLAN | Static Range | DHCP Range |
|------|--------------|------------|
| 100 | .1-.50 | .100-.250 |
| 101 | .1-.50 | .100-.250 |
| 110 | .1-.20 | .100-.250 |
| 111 | .1-.50 | .100-.250 |
| 112 | .1-.50 | .100-.250 |
| 150 | .1-.50 | .100-.250 |
| 200-240 | .1-.50 | .100-.250 |
| 254 | ALL | None (static only) |

---

# 6. Identity Management

## Authentik Overview

Authentik is the single identity system for everything:

```
┌─────────────────────────────────────────────────────────────────┐
│                        AUTHENTIK                                │
│                   (Single Source of Truth)                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│         ┌──────────┬──────────┬──────────┐                     │
│         │   LDAP   │  RADIUS  │   OIDC   │                     │
│         │ Outpost  │  Outpost │ Provider │                     │
│         └────┬─────┴────┬─────┴────┬─────┘                     │
│              │          │          │                            │
│              ▼          ▼          ▼                            │
│         ┌────────┐ ┌────────┐ ┌────────┐                       │
│         │ Linux  │ │  WiFi  │ │ WebUIs │                       │
│         │  NFS   │ │  VPN   │ │  SSO   │                       │
│         │  SSH   │ │ 802.1X │ │  Apps  │                       │
│         │  sudo  │ │ VLAN   │ │  MFA   │                       │
│         └────────┘ └────────┘ └────────┘                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## What Authentik Controls

| Protocol | Port | Used For |
|----------|------|----------|
| **LDAP** | 389/636 | Linux auth, NFS permissions, SSH, sudo groups |
| **RADIUS** | 1812/1813 | WiFi WPA3-Enterprise, VPN, switch 802.1X, dynamic VLAN |
| **OIDC** | 443 | Web UI SSO, Proxmox, all web apps |
| **Forward Auth** | 443 | Legacy apps via Traefik (Radarr, Sonarr, etc.) |

## Authentik Deployment

| Component | VLAN | IP | Purpose |
|-----------|------|-----|---------|
| Authentik Server | 150 | <LAN_IP> | Core + Web UI |
| Authentik Worker | 150 | <LAN_IP> | Background tasks |
| LDAP Outpost | 254 | <LAN_IP> | Linux/NFS auth |
| RADIUS Outpost | 254 | <LAN_IP> | WiFi/VPN auth |
| PostgreSQL | 220 | <LAN_IP> | Database |
| Redis | 220 | <LAN_IP> | Cache/sessions |

## Why Split Outposts?

- **LDAP on VLAN 254**: Infrastructure needs direct LDAP access
- **RADIUS on VLAN 254**: APs and switches need RADIUS on management network
- **Server on VLAN 150**: Web UI via reverse proxy, can reach internet for updates

---

# 7. Users and Groups

## Account Types

| Type | UID Range | Purpose | MFA |
|------|-----------|---------|-----|
| Human | 1000-1999 | Interactive login | Required |
| Service | 10000-19999 | Container identity | None |
| Device | 20000-29999 | MAC/802.1X auth | None |

## Human Users

| Username | UID | Email | Groups | MFA |
|----------|-----|-------|--------|-----|
| user | 1000 | user@vos.home | admins, privileged-users, arr-users, ai-users | Required |
| guest | 1001 | guest@vos.home | guests | Optional |

## Access Control Groups

| Group | GID | RADIUS VLAN | Purpose |
|-------|-----|-------------|---------|
| admins | 1000 | 101 | Full system access |
| privileged-users | 1001 | 101 | LAN + services |
| guests | 1002 | 110 | Internet only |
| home-devices | 1003 | 111 | Home network |
| iot-devices | 1004 | 112 | Local only |
| work-devices | 1005 | 120 | Isolated work |

## Service Permission Groups

| Group | GID | NFS Access | Purpose |
|-------|-----|------------|---------|
| arr-writers | 20001 | /mnt/media:rw | Write media files |
| arr-downloaders | 20002 | /mnt/downloads:rw | Write downloads |
| arr-indexers | 20003 | None | Query indexers |
| media-readers | 20010 | /mnt/media:ro | Read media |
| ai-readers | 20100 | /mnt/ai:ro | Read AI models |
| ai-writers | 20101 | /mnt/ai:rw | Write embeddings |
| db-admins | 20200 | /mnt/backups:rw | Database backup |
| backup-writers | 20201 | /mnt/backups:rw | System backups |

## LDAP Structure

```
dc=vos,dc=home
├── ou=users
│   ├── ou=humans
│   │   ├── uid=user
│   │   └── uid=guest
│   ├── ou=services
│   │   ├── uid=svc-radarr
│   │   ├── uid=svc-sonarr
│   │   └── ...
│   └── ou=devices
│       ├── uid=device-imac-izombie
│       └── ...
└── ou=groups
    ├── cn=admins
    ├── cn=arr-writers
    ├── cn=arr-downloaders
    ├── cn=media-readers
    ├── cn=ai-readers
    └── ...
```

---

# 8. Service Accounts

## ARR Stack (UID 10001-10019)

| Username | UID | Groups | Purpose |
|----------|-----|--------|---------|
| svc-radarr | 10001 | arr-writers, arr-downloaders | Movie management |
| svc-sonarr | 10002 | arr-writers, arr-downloaders | TV management |
| svc-lidarr | 10003 | arr-writers, arr-downloaders | Music management |
| svc-prowlarr | 10004 | arr-indexers | Indexer proxy |
| svc-sabnzbd | 10010 | arr-downloaders | Usenet client |
| svc-qbittorrent | 10011 | arr-downloaders | Torrent client |
| svc-jellyfin | 10015 | media-readers | Media server (read-only) |

## AI Stack (UID 10100-10119)

| Username | UID | Groups | Purpose |
|----------|-----|--------|---------|
| svc-openwebui | 10100 | ai-readers | Chat interface |
| svc-litellm | 10101 | ai-readers | LLM proxy |
| svc-vllm | 10102 | ai-readers, ai-writers | Inference engine |
| svc-haystack | 10103 | ai-readers, ai-writers | RAG pipeline |
| svc-searxng | 10104 | ai-readers | Search aggregator |

## Infrastructure (UID 10200-10219)

| Username | UID | Groups | Purpose |
|----------|-----|--------|---------|
| svc-postgres | 10200 | db-admins | Database server |
| svc-redis | 10201 | db-admins | Cache server |
| svc-wiki | 10202 | wiki-editors | Wiki.js |
| svc-gitea | 10203 | git-users | Git server |
| svc-traefik | 10210 | proxy-admins | Reverse proxy |
| svc-authentik | 10211 | identity-admins | Identity server |

## Group Membership Matrix

```
                    arr-  arr-   media- ai-   ai-
Service Account     write downl  read   read  write
──────────────────  ────  ─────  ─────  ────  ─────
svc-radarr           ✓     ✓      -      -     -
svc-sonarr           ✓     ✓      -      -     -
svc-lidarr           ✓     ✓      -      -     -
svc-sabnzbd          -     ✓      -      -     -
svc-qbittorrent      -     ✓      -      -     -
svc-jellyfin         -     -      ✓      -     -
svc-openwebui        -     -      -      ✓     -
svc-vllm             -     -      -      ✓     ✓
svc-haystack         -     -      -      ✓     ✓

Human Users
──────────────────
user (admin)      ✓     ✓      ✓      ✓     ✓
guest                -     -      ✓      -     -
```

## Security Boundaries

If a container is compromised:

| Compromised Service | Can Access | Cannot Access |
|---------------------|------------|---------------|
| Radarr | /media RW, /downloads RW | /ai, /backups |
| Jellyfin | /media RO | /downloads, /ai, /backups |
| vLLM | /ai RW | /media, /downloads |
| SABnzbd | /downloads RW | /media directly |

---

# 9. DNS Configuration

## Domain Strategy

| Domain | Zone | Purpose |
|--------|------|---------|
| `user.nl` | Cloudflare (public) | Public DNS, cert validation |
| `local.user.nl` | OPNsense (internal) | All homelab services |

## Split-Horizon DNS

```
EXTERNAL QUERY                 INTERNAL QUERY
(from internet)                (from LAN)
      │                              │
      ▼                              ▼
┌───────────────┐            ┌───────────────┐
│  Cloudflare   │            │   OPNsense    │
│  DNS          │            │   Unbound     │
└───────┬───────┘            └───────┬───────┘
        │                            │
        ▼                            ▼
local.user.nl              local.user.nl
→ NXDOMAIN                   → <LAN_IP>
(not public)                 (Traefik proxy)
```

## Internal DNS Records

All service records point to Traefik (<LAN_IP>), which routes by hostname:

| Hostname | IP | Service |
|----------|-----|---------|
| `auth.local.user.nl` | <LAN_IP> | Authentik |
| `wiki.local.user.nl` | <LAN_IP> | Wiki.js |
| `proxmox.local.user.nl` | <LAN_IP> | Proxmox UI |
| `radarr.local.user.nl` | <LAN_IP> | Radarr |
| `sonarr.local.user.nl` | <LAN_IP> | Sonarr |
| `jellyfin.local.user.nl` | <LAN_IP> | Jellyfin |
| `grafana.local.user.nl` | <LAN_IP> | Grafana |
| `chat.local.user.nl` | <LAN_IP> | Open WebUI |

## Infrastructure DNS Records (Direct)

| Hostname | IP | Purpose |
|----------|-----|---------|
| `opnsense.local.user.nl` | <LAN_IP> | Firewall UI |
| `pve-qotom.local.user.nl` | <LAN_IP> | Proxmox node |
| `pve-5800x.local.user.nl` | <LAN_IP> | Proxmox node |
| `nas.local.user.nl` | <LAN_IP> | Unraid |

## OPNsense Unbound Configuration

```
Services > Unbound DNS > Overrides

Host Overrides:
┌────────────────┬─────────────────────┬─────────────────┐
│ Host           │ Domain              │ IP              │
├────────────────┼─────────────────────┼─────────────────┤
│ *              │ local.user.nl     │ <LAN_IP>  │
│ auth           │ local.user.nl     │ <LAN_IP>  │
│ wiki           │ local.user.nl     │ <LAN_IP>  │
│ opnsense       │ local.user.nl     │ <LAN_IP> │
│ nas            │ local.user.nl     │ <LAN_IP> │
└────────────────┴─────────────────────┴─────────────────┘
```

---

# 10. SSL Certificates

## Strategy

Single wildcard certificate covers all `*.local.user.nl` using Let's Encrypt with Cloudflare DNS challenge.

| Property | Value |
|----------|-------|
| Domains | `*.local.user.nl`, `local.user.nl` |
| Issuer | Let's Encrypt |
| Challenge | DNS-01 via Cloudflare API |
| Validity | 90 days (auto-renewed) |
| Location | Traefik (VLAN 150) |

## How DNS Challenge Works

```
1. Traefik requests cert for *.local.user.nl
                 │
                 ▼
2. Let's Encrypt: "Prove you own user.nl"
                 │
                 ▼
3. Traefik creates DNS TXT record via Cloudflare API
   _acme-challenge.local.user.nl = "random-token"
                 │
                 ▼
4. Let's Encrypt checks Cloudflare DNS
   "Token found, domain ownership verified"
                 │
                 ▼
5. Certificate issued, stored in Traefik
                 │
                 ▼
6. All *.local.user.nl now have trusted HTTPS
```

**Key insight:** DNS challenge validates domain ownership. Doesn't matter that `local.user.nl` has no public IP.

## Cloudflare API Token

Create scoped token for cert renewal:

```
Permissions:
  - Zone > DNS > Edit
  - Zone > Zone > Read

Zone Resources:
  - Include > Specific zone > user.nl
```

## Traefik Configuration

### traefik.yml

```yaml
entryPoints:
  web:
    address: ":80"
    http:
      redirections:
        entryPoint:
          to: websecure
  websecure:
    address: ":443"

certificatesResolvers:
  cloudflare:
    acme:
      email: "user@user.nl"
      storage: "/letsencrypt/acme.json"
      dnsChallenge:
        provider: cloudflare
        resolvers:
          - "1.1.1.1:53"
          - "8.8.8.8:53"
```

### Environment Variables

```bash
CF_API_EMAIL=user@user.nl
CF_DNS_API_TOKEN=your-scoped-api-token
```

---

# 11. Firewall Rules

## OPNsense Rule Structure

Rules are processed top-to-bottom, first match wins.

### VLAN 100 (Unprivileged/Quarantine)

| Order | Action | Source | Dest | Port | Purpose |
|-------|--------|--------|------|------|---------|
| 1 | Allow | VLAN100 | Any | 22,80,443 | Basic web only |
| 2 | Block | VLAN100 | RFC1918 | Any | No internal access |
| 3 | Block | VLAN100 | Any | Any | Default deny |

### VLAN 101 (Privileged LAN)

| Order | Action | Source | Dest | Port | Purpose |
|-------|--------|--------|------|------|---------|
| 1 | Allow | VLAN101 | VLAN150 | 80,443 | Access reverse proxy |
| 2 | Allow | VLAN101 | VLAN254 | 53,67,68 | DNS, DHCP |
| 3 | Block | VLAN101 | RFC1918 | Any | No direct backend |
| 4 | Allow | VLAN101 | Any | Any | Internet access |

### VLAN 110 (Guest WiFi)

| Order | Action | Source | Dest | Port | Purpose |
|-------|--------|--------|------|------|---------|
| 1 | Allow | VLAN110 | VLAN254 | 53 | DNS only |
| 2 | Block | VLAN110 | RFC1918 | Any | No internal |
| 3 | Allow | VLAN110 | Any | 80,443 | Web only |
| 4 | Block | VLAN110 | Any | Any | Default deny |

### VLAN 112 (IoT)

| Order | Action | Source | Dest | Port | Purpose |
|-------|--------|--------|------|------|---------|
| 1 | Allow | VLAN112 | VLAN254 | 53 | DNS only |
| 2 | Allow | VLAN112 | VLAN112 | Any | Local discovery |
| 3 | Block | VLAN112 | Any | Any | No internet |

### VLAN 150 (Reverse Proxy)

| Order | Action | Source | Dest | Port | Purpose |
|-------|--------|--------|------|------|---------|
| 1 | Allow | VLAN150 | VLAN200 | 8080,8000 | AI services |
| 2 | Allow | VLAN150 | VLAN210 | 7878,8989,8686,9696 | ARR stack |
| 3 | Allow | VLAN150 | VLAN220 | 3000,5432 | Wiki.js, PostgreSQL |
| 4 | Allow | VLAN150 | Any | 80,443 | Let's Encrypt ACME |
| 5 | Block | VLAN150 | Any | Any | Default deny |

### Backend VLANs (200-250)

| Order | Action | Source | Dest | Port | Purpose |
|-------|--------|--------|------|------|---------|
| 1 | Allow | Backend | VLAN230 | 2049,111 | NFS access |
| 2 | Allow | Backend | VLAN220 | 5432,6379 | Database |
| 3 | Allow | Backend | Same VLAN | Any | Intra-VLAN |
| 4 | Block | Backend | Any | Any | **NO INTERNET** |

### VLAN 254 (Management)

| Order | Action | Source | Dest | Port | Purpose |
|-------|--------|--------|------|------|---------|
| 1 | Allow | VLAN254 | Any | Any | Full admin access |

## Inter-VLAN Rules Summary

```
ALLOWED FLOWS:
─────────────────────────────────────────────────
Frontend (101,111) → Proxy (150) → Backend (200-250)
Backend (200-250) ↔ Storage (230)
Backend (200-250) ↔ Database (220)
Management (254) → Anywhere

BLOCKED FLOWS:
─────────────────────────────────────────────────
Frontend → Backend directly (must use proxy)
Backend → Internet (air-gapped)
Backend → Frontend (never initiates)
Guest/IoT → Internal (isolated)
```

---

# 12. WiFi Configuration

## SSIDs

| SSID | Auth | VLAN | Security | Purpose |
|------|------|------|----------|---------|
| VOS-Home | WPA3-Enterprise | Dynamic | RADIUS | Trusted users, VLAN by identity |
| VOS-Guest | Captive Portal | 110 | WPA2 | Time-limited guests |
| VOS-IoT | WPA2-PSK | 112 | Hidden | IoT devices, local only |

## Dynamic VLAN Assignment

RADIUS returns VLAN based on user/group:

| User/Group | VLAN | Subnet | Access |
|------------|------|--------|--------|
| admins | 101 | <LAN_IP>/24 | Full privileged |
| privileged-users | 101 | <LAN_IP>/24 | LAN + services |
| guests | 110 | <LAN_IP>/24 | Internet only |
| home-devices | 111 | <LAN_IP>/24 | Home network |
| iot-devices | 112 | <LAN_IP>/24 | Local only |
| work-devices | 120 | <LAN_IP>/24 | Isolated |
| unknown | 100 | <LAN_IP>/24 | Quarantine |

## OpenWRT AP Configuration (RT-2980)

### /etc/config/wireless

```bash
config wifi-iface 'wlan0_enterprise'
    option device 'radio0'
    option mode 'ap'
    option ssid 'VOS-Home'
    option encryption 'wpa3'
    option auth_server '<LAN_IP>'
    option auth_port '1812'
    option auth_secret 'RADIUS_SECRET_HERE'
    option acct_server '<LAN_IP>'
    option acct_port '1813'
    option acct_secret 'RADIUS_SECRET_HERE'
    option dynamic_vlan '2'
    option vlan_tagged_interface 'eth0'
    option vlan_bridge 'br-lan'
    option ieee80211w '2'  # MFP required (WPA3)
```

## Authentik RADIUS Outpost

### VLAN Assignment Policy

```python
# Policy: assign-vlan-by-group

if ak_is_group_member(request.user, name="admins"):
    request.context["radius"]["Tunnel-Type"] = "VLAN"
    request.context["radius"]["Tunnel-Medium-Type"] = "IEEE-802"
    request.context["radius"]["Tunnel-Private-Group-ID"] = "101"
    return True

if ak_is_group_member(request.user, name="privileged-users"):
    request.context["radius"]["Tunnel-Private-Group-ID"] = "101"
    return True

if ak_is_group_member(request.user, name="guests"):
    request.context["radius"]["Tunnel-Private-Group-ID"] = "110"
    return True

if ak_is_group_member(request.user, name="iot-devices"):
    request.context["radius"]["Tunnel-Private-Group-ID"] = "112"
    return True

if ak_is_group_member(request.user, name="work-devices"):
    request.context["radius"]["Tunnel-Private-Group-ID"] = "120"
    return True

# Default: quarantine
request.context["radius"]["Tunnel-Private-Group-ID"] = "100"
return True
```

---

# 13. NFS Storage Permissions

## Folder Structure

```
/mnt/user/
├── downloads/              # arr-downloaders (GID 20002)
│   ├── incomplete/
│   ├── complete/
│   └── watch/
├── media/                  # arr-writers (GID 20001)
│   ├── movies/
│   ├── tv/
│   ├── music/
│   └── books/
├── ai/                     # ai-writers (GID 20101)
│   ├── models/
│   ├── embeddings/
│   └── cache/
└── backups/                # backup-writers (GID 20201)
    ├── databases/
    ├── configs/
    └── vms/
```

## NFS Exports

### Option A: All Squash (Simple)

```bash
# /etc/exports
/mnt/user/downloads  <LAN_IP>/24(rw,all_squash,anonuid=10010,anongid=20002)
/mnt/user/media      <LAN_IP>/24(rw,all_squash,anonuid=10001,anongid=20001)
/mnt/user/ai         <LAN_IP>/24(rw,all_squash,anonuid=10102,anongid=20101)
```

### Option B: SSSD + ACLs (Proper)

```bash
# /etc/exports (no squash)
/mnt/user/downloads  <LAN_IP>/24(rw,no_root_squash)
/mnt/user/media      <LAN_IP>/24(rw,no_root_squash)
/mnt/user/ai         <LAN_IP>/24(rw,no_root_squash)

# Folder permissions
chown root:arr-writers /mnt/user/media
chmod 775 /mnt/user/media
setfacl -R -m g:media-readers:rx /mnt/user/media
```

## Storage Protocol by Zone

| Zone | Protocol | Use Case |
|------|----------|----------|
| Backend | NFS only | Linux service-to-service |
| Frontend | SMB only | Windows/Mac clients |

---

# 14. Implementation Order

## Phase 0: Pre-Staging (DONE)

- [x] Development environment on iZombie
- [x] Claude Desktop with MCP servers
- [x] Wiki.js documentation system
- [x] Planning documents

## Phase 1: Network Foundation

### Step 1.1: OPNsense Base

1. Install OPNsense on hal
2. Configure WAN interface
3. Configure LAN interface (VLAN 254)
4. Set management IP: <LAN_IP>
5. Basic firewall rules

### Step 1.2: VLAN Creation

Create VLANs in order:

```
1. VLAN 254 (Management) - First, for infrastructure
2. VLAN 100 (Unprivileged) - Quarantine network
3. VLAN 101 (Privileged) - Main LAN
4. VLAN 150 (Proxy) - Reverse proxy
5. VLAN 200-240 (Backend) - Service VLANs
6. VLAN 110-112 (WiFi) - After RADIUS works
7. VLAN 120-140 (Remote) - After VPN works
```

### Step 1.3: Switch Configuration

1. Zyxel GS1900-24HP: Configure trunk ports, VLAN tagging
2. ONTI S508CL-8S: Configure backend trunk
3. Tenda TEM2007: Bridge to ONTI

### Step 1.4: Host Network Configuration

For each Proxmox host:

1. Configure frontend NIC on VLAN 254
2. Configure backend NIC on VLAN 254 (backend side)
3. Create VLAN interfaces for services
4. Test connectivity

## Phase 2: Identity Layer

### Step 2.1: Authentik Deployment

1. Deploy PostgreSQL on deep-thought (<LAN_IP>)
2. Deploy Redis on deep-thought (<LAN_IP>)
3. Deploy Authentik Server on mothership (<LAN_IP>)
4. Deploy Authentik Worker on mothership (<LAN_IP>)
5. Configure initial admin user

### Step 2.2: LDAP Setup

1. Create LDAP Outpost
2. Deploy to VLAN 254 (<LAN_IP>)
3. Configure LDAP schema
4. Create service accounts
5. Test LDAP bind

### Step 2.3: RADIUS Setup

1. Create RADIUS Outpost
2. Deploy to VLAN 254 (<LAN_IP>)
3. Configure RADIUS clients (APs, switches)
4. Create VLAN assignment policy
5. Test RADIUS auth

### Step 2.4: Groups and Users

1. Create access control groups
2. Create service permission groups
3. Create user user account
4. Create service accounts
5. Assign group memberships

## Phase 3: DNS and Certificates

### Step 3.1: Internal DNS

1. Configure OPNsense Unbound
2. Create local zone: local.user.nl
3. Add host overrides
4. Test internal resolution

### Step 3.2: SSL Certificates

1. Create Cloudflare API token
2. Deploy Traefik on mothership
3. Configure Let's Encrypt DNS challenge
4. Request wildcard certificate
5. Verify HTTPS works

## Phase 4: Core Services

### Step 4.1: Reverse Proxy

1. Configure Traefik routes
2. Add Authentik forward auth
3. Test SSO flow

### Step 4.2: Database Services

1. Deploy Wiki.js (<LAN_IP>)
2. Deploy Gitea (<LAN_IP>)
3. Configure OIDC auth
4. Test SSO

### Step 4.3: Storage Services

1. Configure NFS exports on alexandria
2. Set LDAP group permissions
3. Test NFS mounts
4. Configure SMB for frontend

## Phase 5: Application Services

### Step 5.1: ARR Stack

1. Deploy services on napster
2. Configure service accounts
3. Configure NFS mounts
4. Configure forward auth
5. Test full flow

### Step 5.2: AI Stack

1. Deploy services on skynet/watson
2. Configure service accounts
3. Configure NFS mounts
4. Configure OIDC auth
5. Test full flow

## Phase 6: WiFi and Remote Access

### Step 6.1: WiFi

1. Configure APs for WPA3-Enterprise
2. Test RADIUS authentication
3. Test dynamic VLAN assignment
4. Configure Guest and IoT SSIDs

### Step 6.2: VPN

1. Configure WireGuard on OPNsense
2. Integrate with Authentik
3. Test remote access
4. Configure remote site tunnels

---

# 15. Verification Checklist

## Network Verification

```bash
# From management host
ping <LAN_IP>       # Gateway
ping <LAN_IP>       # Zyxel
ping <LAN_IP>     # Proxmox host

# VLAN isolation test
# From VLAN 100, should FAIL:
ping <LAN_IP>      # Backend should be blocked

# From VLAN 101 via proxy, should WORK:
curl https://wiki.local.user.nl
```

## DNS Verification

```bash
# Internal DNS
nslookup wiki.local.user.nl
# Should return: <LAN_IP>

# External DNS (should fail)
nslookup wiki.local.user.nl 8.8.8.8
# Should return: NXDOMAIN
```

## Certificate Verification

```bash
curl -v https://wiki.local.user.nl 2>&1 | grep -A5 "Server certificate"
# Should show: issuer: Let's Encrypt
```

## LDAP Verification

```bash
# Test LDAP bind
ldapsearch -x -H ldap://<LAN_IP> \
  -D "cn=ldap-service,ou=services,dc=vos,dc=home" \
  -w "password" \
  -b "ou=users,dc=vos,dc=home"
```

## RADIUS Verification

```bash
# Test RADIUS auth
radtest user 'password' <LAN_IP> 0 RADIUS_SECRET

# Expected response:
# Access-Accept
#   Tunnel-Type = VLAN
#   Tunnel-Medium-Type = IEEE-802
#   Tunnel-Private-Group-ID = 101
```

## NFS Verification

```bash
# From container as svc-radarr
touch /mnt/media/test     # Should work
touch /mnt/ai/test        # Should FAIL (wrong group)
```

## SSO Verification

1. Visit https://wiki.local.user.nl
2. Should redirect to Authentik login
3. Login with user
4. Should redirect back, logged in
5. Visit https://radarr.local.user.nl
6. Should be logged in automatically (SSO)

---

# Quick Reference Card

## Key IPs

| Service | IP |
|---------|-----|
| OPNsense WebUI | <LAN_IP> |
| Authentik | https://auth.local.user.nl |
| Wiki.js | https://wiki.local.user.nl |
| Traefik | <LAN_IP> |
| LDAP Outpost | <LAN_IP> |
| RADIUS Outpost | <LAN_IP> |

## Key Ports

| Service | Port |
|---------|------|
| LDAP | 389, 636 (TLS) |
| RADIUS Auth | 1812 |
| RADIUS Acct | 1813 |
| PostgreSQL | 5432 |
| Redis | 6379 |
| NFS | 2049, 111 |

## Key Users

| Username | Purpose |
|----------|---------|
| user | Admin user |
| svc-* | Service accounts |
| device-* | Device accounts |

## Key Groups

| Group | VLAN | Purpose |
|-------|------|---------|
| admins | 101 | Full access |
| arr-writers | - | Media write |
| ai-writers | - | AI data write |
| media-readers | - | Media read-only |

---

**END OF DOCUMENT**

*This document is the Single Source of Truth. When in doubt, follow this guide.*
