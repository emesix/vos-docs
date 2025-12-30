---
title: Backend VLAN Design
description: Backend VLAN design grouping services by function with strict inter-VLAN firewall control
published: false
date: 2025-12-28T18:16:14.577Z
tags: network, architecture, homelab, ssot, security, vlan
editor: markdown
dateCreated: 2025-12-27T01:25:37.651Z
---

---
title: "Backend VLAN Design"
uid: "infra-network-vlan-design"
kind: [infrastructure, network, architecture, security]
category: "01"
sub-category: "02"
visibility: 3.0
status: 4.0
tags: [network, vlan, security, architecture, homelab, ssot]
parent_uid: ""
---

# VLAN Design

**Category:** 01.02 - Network Infrastructure  
**Purpose:** Complete VLAN architecture with security zones  
**Updated:** 2025-12-28

## Security Model

```
┌───────────────────────────────────────────────────────────┐
│                      INTERNET                          │
│                          │                              │
│                          │ WAN                          │
│                          ▼                              │
│                    ┌──────────┐                        │
│                    │ OPNsense │                        │
│                    └────┬─────┘                        │
│                         │                              │
│          ┌──────────────┼──────────────┐               │
│          │              │              │               │
│          ▼              ▼              ▼               │
│     FRONTEND        BACKEND      MANAGEMENT           │
│     (100-150)       (200-250)      (254)              │
│          │              │              │               │
│          │              ╳ NO INTERNET  │               │
│          │              │              │               │
│     ┌────┴────┐    ┌────┴────┐   ┌────┴────┐          │
│     │Firewalled│    │ Trusted │   │  Admin  │          │
│     │ Internet │    │  Local  │   │  Only   │          │
│     └──────────┘    └─────────┘   └─────────┘          │
└───────────────────────────────────────────────────────────┘
```

| Zone | VLANs | Internet | Storage | Risk Level |
|------|-------|----------|---------|------------|
| **Frontend** | 100-150 | Firewalled | SMB | High (untrusted) |
| **Backend** | 200-250 | **BLOCKED** | NFS | Low (trusted) |
| **Management** | 254+ | Admin only | - | Controlled |

## Design Principles

**Backend = Trusted Zone**
- No internet access (air-gapped from WAN)
- Service-to-service traffic only
- NFS for storage (Linux native, fast)
- Monitored - bad actor = instant flag
- Low risk, high trust

**Frontend = Untrusted Zone**
- Firewalled internet per VLAN
- Experimental, client-facing
- SMB for storage (Windows/client compatible)
- Reverse proxy for service WebUIs
- MAC + domain auth for privileged access

**Management = Infrastructure Only**
- Physical hosts, switches, APs
- Proxmox/Unraid WebUI
- OPNsense WebUI at .254.254
- No .1 WebUI (gateway only)

## Traffic Rules

| From | To | Policy |
|------|-----|--------|
| Frontend → Backend | Allowed | Via reverse proxy (VLAN 150) |
| Backend → Frontend | **Blocked** | Backend never initiates |
| Backend → Internet | **Blocked** | Air-gapped |
| Frontend → Internet | Firewalled | Per-VLAN rules |
| Any → Management | Privileged only | Admin access required |

## Frontend VLANs (100-150)

### Client Access

| VLAN | Subnet | Name | Internet | Access |
|------|--------|------|----------|--------|
| 100 | <LAN_IP>/24 | Unprivileged LAN | 22/80/443 | Unknown MAC |
| 101 | <LAN_IP>/24 | Privileged LAN | Firewalled | Known MAC + auth |
| 110 | <LAN_IP>/24 | Guest WiFi | Captive | Time-limited |
| 111 | <LAN_IP>/24 | Home WiFi | Firewalled | Trusted devices |
| 112 | <LAN_IP>/24 | IoT WiFi | **Blocked** | Local only |
| 120 | <LAN_IP>/24 | Work | Firewalled | Isolated |

### Remote Sites

| VLAN | Subnet | Name | Connection |
|------|--------|------|------------|
| 130 | <LAN_IP>/24 | Basement | WireGuard tunnel |
| 131 | <LAN_IP>/24 | Garage | WireGuard over P2P |
| 140 | <LAN_IP>/24 | VvE | Isolated, no internet |

### Service Frontend

| VLAN | Subnet | Name | Purpose |
|------|--------|------|--------|
| 150 | <LAN_IP>/24 | Reverse Proxy | WebUI bridge to backend |

## Backend VLANs (200-250)

| VLAN | Subnet | Name | Purpose | Primary Host |
|------|--------|------|---------|-------------|
| 200 | <LAN_IP>/24 | AI | Inference, embeddings | skynet, watson |
| 210 | <LAN_IP>/24 | ARR | Media automation | napster |
| 220 | <LAN_IP>/24 | Database | PostgreSQL, Redis | deep-thought |
| 230 | <LAN_IP>/24 | Storage | NFS exports | alexandria |
| 240 | <LAN_IP>/24 | Docker | Container networks | mothership |

## Management VLAN (254)

| Subnet | Name | Purpose |
|--------|------|--------|
| <LAN_IP>/24 | Management | All physical infrastructure |

**Reserved Ranges:**
- .1 = Gateway (no WebUI)
- .2-.10 = Switches
- .11-.20 = WiFi APs
- .100-.109 = Proxmox Frontend NICs
- .110-.119 = NAS Frontend
- .200-.209 = Proxmox Backend NICs
- .210-.219 = NAS Backend
- .254 = OPNsense WebUI

## MAC-Based Access Control

```
New Device Connects
        │
        ▼
  ┌─────────────┐
  │ MAC Known?  │
  └──────┬──────┘
    NO  │  YES
        │
  ┌──────┴────────────┐
  ▼                  ▼
 VLAN 100         ┌─────────────┐
 Unprivileged     │ Domain Auth? │
 Setup only      └──────┬──────┘
 (22/80/443)        NO  │  YES
                        │
                  ┌──────┴──────┐
                  ▼            ▼
              VLAN 100     VLAN 101
              Unprivileged Privileged
                           Full access
```

## Storage Protocol Split

| Zone | Protocol | Use Case |
|------|----------|----------|
| Backend | NFS only | Linux service-to-service |
| Frontend | SMB only | Windows/Mac clients |

**NFS Exports (Backend VLAN 230):**
- `/mnt/media` → VLAN 210 (ARR)
- `/mnt/ai-models` → VLAN 200 (AI)
- `/mnt/backups` → VLAN 220 (DB)

**SMB Shares (Frontend via VLAN 101):**
- `\\nas\media` → Privileged clients
- `\\nas\documents` → Privileged clients
- `\\nas\backups` → Privileged clients

## Reverse Proxy Flow (VLAN 150)

```
Client (VLAN 101)
      │
      │ https://wiki.vos.local
      ▼
┌─────────────────┐
│ Reverse Proxy  │ VLAN 150
│ <LAN_IP> │
└────────┬────────┘
        │
        │ Proxy to backend
        ▼
┌─────────────────┐
│ Wiki.js        │ VLAN 220 (Database)
│ <SUBNET>.x  │
└─────────────────┘
```

**Rules:**
- Clients → VLAN 150: Allowed
- VLAN 150 → Backend: Allowed (specific ports)
- Clients → Backend: **Blocked** (must use proxy)

## WiFi SSIDs

| SSID | VLAN | Security | Access |
|------|------|----------|--------|
| VOS-Home | 111 | WPA3 | Trusted, firewalled internet |
| VOS-Guest | 110 | Captive Portal | Limited, time-based |
| VOS-IoT | 112 | WPA2/WPA3 (hidden?) | Local only, no internet |

## Open Questions

| Item | Options | Status |
|------|---------|--------|
| ARR Internet | Exception rule / Download proxy | TBD |
| Domain Auth | FreeIPA / Authentik / LLDAP | TBD |
| Reverse Proxy | Traefik / Caddy / nginx | TBD |
| Internal Domain | .home.arpa / .vos.local | TBD |

## Related

- [[homelab/network/ip-plan]]
- [[homelab/network/architecture]]
- [[homelab/network/topology]]
- [[homelab/hosts]]