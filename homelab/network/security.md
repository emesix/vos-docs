---
title: Network Security Architecture
description: Security model, access control, zones, and traffic policies for VOS network
published: false
date: 2025-12-28T18:54:44.707Z
tags: network, architecture, firewall, security, vlan, mac, authentik, identity
editor: markdown
dateCreated: 2025-12-28T18:18:35.069Z
---

---
title: "Network Security Architecture"
uid: "infra-network-security"
kind: [infrastructure, network, security]
category: "01"
sub-category: "02"
visibility: 3.0
status: 4.0
tags: [network, security, architecture, vlan, firewall, mac, authentik, identity]
parent_uid: ""
---

# Network Security Architecture

**Category:** 01.02 - Network Infrastructure  
**Purpose:** Security model, access control, and zone isolation  
**Updated:** 2025-12-28

## Overview

Dual-zone architecture with Authentik identity layer. MAC-based network access. LDAP for service identity. RADIUS for WiFi/VPN. All inter-VLAN traffic through OPNsense firewall.

## Security Layers

```
┌─────────────────────────────────────────────────────────────┐
│                   SECURITY STACK                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  L7: Application    Authentik SSO + Forward Auth           │
│                     Per-app authorization policies          │
│                                                             │
│  L4: Service        NFS/SMB with LDAP groups               │
│                     UID/GID based file permissions          │
│                                                             │
│  L3: Network        OPNsense firewall                      │
│                     Inter-VLAN rules, no backend internet   │
│                                                             │
│  L2: Access         RADIUS dynamic VLAN                    │
│                     MAC + user auth determines VLAN         │
│                                                             │
│  L1: Physical       802.1X port auth (optional)            │
│                     Unknown devices quarantined             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Security Zones

| Zone | VLANs | Internet | Storage | Trust |
|------|-------|----------|---------|-------|
| **Backend** | 200-250 | **BLOCKED** | NFS | High (trusted) |
| **Frontend** | 100-150 | Firewalled | SMB | Low (untrusted) |
| **Management** | 254+ | Admin only | - | Controlled |

## Identity Layer (Authentik)

Centralized identity management for all access.

| Protocol | Use Case |
|----------|----------|
| LDAP | Linux auth, NFS permissions, service accounts |
| RADIUS | WiFi WPA3-Enterprise, VPN, switch 802.1X |
| OIDC | Web UI SSO, application authorization |
| Forward Auth | Legacy apps via reverse proxy |

→ Details: [[homelab/identity/authentik]]

## Traffic Flow Rules

| From | To | Policy |
|------|-----|--------|
| Frontend → Backend | Via proxy only | VLAN 150 |
| Backend → Frontend | **Blocked** | Never initiates |
| Backend → Internet | **Blocked** | Air-gapped |
| Frontend → Internet | Firewalled | Per-VLAN rules |
| Any → Management | Privileged only | Admin required |

## Access Control Flow

```
Device/User Connects
        │
        ▼
┌───────────────┐
│  WiFi/Wired?  │
└───────┬───────┘
        │
        ▼
┌───────────────┐    RADIUS     ┌───────────────┐
│  AP/Switch   │ ──────────▶ │   Authentik   │
└───────────────┘              └───────┬───────┘
                                    │
                             Return VLAN ID
                                    │
                    ┌───────────────┴───────────────┐
                    │                               │
                    ▼                               ▼
              admins → VLAN 101            guests → VLAN 110
              privileged → VLAN 101        iot → VLAN 112
              work → VLAN 120              unknown → VLAN 100
```

→ Details: [[homelab/identity/radius-wifi]]

## Storage Security

| Protocol | Zone | Auth | Access Control |
|----------|------|------|---------------|
| NFS | Backend | LDAP UID/GID | Group permissions |
| SMB | Frontend | Authentik | User shares |

**Key principle:** Service accounts have minimum required permissions.

| Service | Can Write | Cannot Write |
|---------|-----------|-------------|
| svc-radarr | /media, /downloads | /ai, /backups |
| svc-jellyfin | Nothing | Everything (read-only) |
| svc-vllm | /ai | /media, /downloads |

→ Details: [[homelab/identity/nfs-permissions]]

## Web Application Security

All web UIs protected by Authentik.

| Method | Apps | Protection |
|--------|------|------------|
| OIDC Native | Wiki.js, Grafana, Proxmox | Direct SSO |
| Forward Auth | Radarr, Sonarr, legacy | Proxy auth |

**Features:**
- Single Sign-On across all apps
- MFA enforcement for admin accounts
- Group-based authorization
- Session management

→ Details: [[homelab/identity/oidc-apps]]

## WiFi Security

| SSID | Auth | VLAN | Internet |
|------|------|------|----------|
| VOS-Home | WPA3-Enterprise (RADIUS) | Dynamic | Firewalled |
| VOS-Guest | Captive Portal | 110 | Throttled |
| VOS-IoT | WPA2-PSK | 112 | **Blocked** |

**Dynamic VLAN:** User identity determines network access, not just SSID.

## VPN Security

| Method | Auth | Use Case |
|--------|------|----------|
| WireGuard | Authentik portal | Personal configs |
| OpenVPN | RADIUS + MFA | Legacy clients |
| Headscale | OIDC | Mesh networking |

**Remote sites (Basement, Garage):** All traffic encrypted via WireGuard.

## Attack Mitigation

| Attack | Mitigation |
|--------|------------|
| Compromised container | Limited to service account permissions |
| WiFi eavesdropping | WPA3 encryption |
| Lateral movement | VLAN isolation + firewall rules |
| Credential theft | Unique service accounts, rotation |
| Data exfiltration | Backend has no internet |
| Man-in-middle | WireGuard for remote sites |

## Monitoring

| What | How |
|------|-----|
| Auth failures | Authentik logs |
| Unusual traffic | OPNsense IDS |
| Service access | LDAP audit logs |
| WiFi associations | OpenWRT logs |

## Related

- [[homelab/identity/authentik]]
- [[homelab/identity/users-groups]]
- [[homelab/identity/radius-wifi]]
- [[homelab/identity/oidc-apps]]
- [[homelab/identity/ldap-linux]]
- [[homelab/identity/nfs-permissions]]
- [[homelab/network/vlan-design]]
- [[homelab/network/ip-plan]]