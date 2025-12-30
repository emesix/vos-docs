---
title: Authentik Identity Platform
description: Complete identity management with Authentik - LDAP, RADIUS, OIDC, SSO for all VOS services
published: true
date: 2025-12-28T18:45:27.258Z
tags: security, authentik, identity, ldap, radius, oidc, sso, authentication
editor: markdown
dateCreated: 2025-12-28T18:45:24.672Z
---

---
title: "Authentik Identity Platform"
uid: "svc-authentik"
kind: [service, identity, security]
category: "03"
sub-category: "05"
visibility: 3.0
status: 3.0
tags: [authentik, identity, ldap, radius, oidc, sso, security, authentication]
parent_uid: ""
---

# Authentik Identity Platform

**Category:** 03.05 - Identity Services  
**Purpose:** Unified identity management for all VOS infrastructure  
**Updated:** 2025-12-28

## Overview

Authentik provides centralized identity for users, devices, and services. Single source of truth for authentication across WiFi, VPN, web UIs, Linux systems, and network infrastructure.

## Why Authentik

| Feature | LLDAP | Authentik | FreeIPA |
|---------|-------|-----------|--------|
| LDAP | ✓ | ✓ | ✓ |
| RADIUS | ✗ | ✓ | ✓ |
| OIDC/OAuth2 | ✗ | ✓ | Limited |
| Web SSO | ✗ | ✓ | ✗ |
| MFA | ✗ | ✓ | ✓ |
| Forward Auth | ✗ | ✓ | ✗ |
| Complexity | Low | Medium | High |

Authentik covers all use cases in one platform.

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      AUTHENTIK                              │
│                   (Identity Hub)                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│         ┌──────────┬──────────┬──────────┐                 │
│         │   LDAP   │  RADIUS  │   OIDC   │                 │
│         │ Outpost  │  Outpost │ Provider │                 │
│         └────┬─────┴────┬─────┴────┬─────┘                 │
│              │          │          │                        │
│              ▼          ▼          ▼                        │
│         ┌────────┐ ┌────────┐ ┌────────┐                   │
│         │ Linux  │ │  WiFi  │ │ WebUIs │                   │
│         │  NFS   │ │  VPN   │ │  SSO   │                   │
│         │  SSH   │ │ 802.1X │ │  Apps  │                   │
│         └────────┘ └────────┘ └────────┘                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Deployment

### Components

| Component | VLAN | IP | Purpose |
|-----------|------|-----|--------|
| Authentik Server | 150 | <LAN_IP> | Core + Web UI |
| Authentik Worker | 150 | <LAN_IP> | Background tasks |
| LDAP Outpost | 254 | <LAN_IP> | Linux/NFS auth |
| RADIUS Outpost | 254 | <LAN_IP> | WiFi/VPN/802.1X |
| PostgreSQL | 220 | <LAN_IP> | Database |
| Redis | 220 | <LAN_IP> | Cache/sessions |

### Why Split Outposts?

- **LDAP Outpost on VLAN 254:** Infrastructure needs direct LDAP access
- **RADIUS Outpost on VLAN 254:** APs and switches need RADIUS on management network
- **Server on VLAN 150:** Web UI accessible via reverse proxy, can reach internet for updates

## Protocols Provided

### LDAP (Port 389/636)

Used by:
- Linux containers (SSSD/NSS/PAM)
- NFS user mapping
- SSH authentication
- Proxmox user sync
- Unraid user sync

### RADIUS (Port 1812/1813)

Used by:
- WiFi WPA3-Enterprise
- VPN authentication
- Switch 802.1X port auth
- Dynamic VLAN assignment

### OIDC/OAuth2 (Port 443)

Used by:
- Web UI SSO (Wiki.js, Gitea, Grafana)
- Open WebUI
- Jellyfin
- Proxmox
- Any OIDC-capable app

### Forward Auth (Port 443)

Used by:
- Legacy apps without native auth
- Radarr, Sonarr, Lidarr
- Any app behind Traefik

## Integration Points

```
┌─────────────────────────────────────────────────────────────┐
│                  WHAT AUTHENTIK CONTROLS                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  WIFI (RADIUS)           VPN (RADIUS/OIDC)                 │
│  ─────────────           ────────────────                  │
│  • VOS-Home → VLAN 111   • WireGuard configs               │
│  • VOS-Guest → VLAN 110  • OpenVPN auth                    │
│  • VOS-IoT → VLAN 112    • Headscale OIDC                  │
│  • Dynamic VLAN          • Per-user tunnels                │
│                                                             │
│  WEB UIS (OIDC/Forward)  LINUX (LDAP)                      │
│  ─────────────────────   ───────────                       │
│  • Single Sign-On        • SSH login                       │
│  • MFA enforcement       • sudo groups                     │
│  • RBAC per app          • NFS UID/GID                     │
│  • Session management    • Service accounts                │
│                                                             │
│  SWITCHES (RADIUS)       PROXMOX (OIDC)                    │
│  ────────────────        ─────────────                     │
│  • 802.1X port auth      • Admin SSO                       │
│  • MAC authentication    • Role mapping                    │
│  • Dynamic VLAN          • Audit logging                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Related

- [[homelab/identity/users-groups]]
- [[homelab/identity/radius-wifi]]
- [[homelab/identity/oidc-apps]]
- [[homelab/identity/ldap-linux]]
- [[homelab/network/security]]