---
title: Identity Management
description: Centralized identity with Authentik - LDAP, RADIUS, OIDC
published: true
date: 2025-12-29T12:00:00.000Z
tags: identity, authentik, ldap, radius, oidc, sso
editor: markdown
dateCreated: 2025-12-26T08:35:30.000Z
---

# Identity Management

Authentik provides single identity for all services.

## Architecture

```
┌─────────────────────────────────────────┐
│              AUTHENTIK                  │
│         (Single Source of Truth)        │
├─────────────────────────────────────────┤
│    ┌──────────┬──────────┬──────────┐  │
│    │   LDAP   │  RADIUS  │   OIDC   │  │
│    │ Outpost  │  Outpost │ Provider │  │
│    └────┬─────┴────┬─────┴────┬─────┘  │
│         │          │          │         │
│         ▼          ▼          ▼         │
│    ┌────────┐ ┌────────┐ ┌────────┐    │
│    │ Linux  │ │  WiFi  │ │ WebUIs │    │
│    │  NFS   │ │  VPN   │ │  SSO   │    │
│    │  sudo  │ │ 802.1X │ │  Apps  │    │
│    └────────┘ └────────┘ └────────┘    │
└─────────────────────────────────────────┘
```

## Deployment

| Component | VLAN | IP | Purpose |
|-----------|------|-----|---------|
| Authentik Server | 150 | <LAN_IP> | Core + WebUI |
| LDAP Outpost | 254 | <LAN_IP> | Linux/NFS auth |
| RADIUS Outpost | 254 | <LAN_IP> | WiFi/VPN auth |
| PostgreSQL | 220 | <LAN_IP> | Database |
| Redis | 220 | <LAN_IP> | Sessions |

## Protocols

| Protocol | Port | Used For |
|----------|------|----------|
| LDAP | 389/636 | Linux, NFS, SSH, sudo |
| RADIUS | 1812/1813 | WiFi, VPN, 802.1X, dynamic VLAN |
| OIDC | 443 | Web SSO, Proxmox, apps |

## Related

- [[homelab/identity/authentik]]
- [[homelab/identity/ldap-linux]]
- [[homelab/identity/radius-wifi]]
- [[homelab/identity/users-groups]]
- [[homelab/identity/nfs-permissions]]
- [[homelab/identity/oidc-apps]]
