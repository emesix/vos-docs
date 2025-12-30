---
title: Network Architecture
description: Dual-zone network architecture with VLAN segmentation
published: true
date: 2025-12-29T12:00:00.000Z
tags: network, architecture, vlan, opnsense, 10gbe
editor: markdown
dateCreated: 2025-12-28T15:45:25.958Z
---

# Network Architecture

Dual-zone architecture separating trusted backend (no internet) from untrusted frontend (firewalled).

## Security Zones

| Zone | VLANs | Internet | Storage | Trust |
|------|-------|----------|---------|-------|
| Frontend | 100-150 | Firewalled | SMB | Low |
| Backend | 200-250 | **BLOCKED** | NFS | High |
| Management | 254 | Admin only | - | Controlled |

## Topology

```
INTERNET
    │
    ▼
┌─────────────────────────────────┐
│  OPNsense (hal)                 │
│  Qotom Q20331G9 1U              │
└─────────────────────────────────┘
       │                    │
       │ 2x1GbE LACP        │ 10GbE
       ▼                    ▼
┌─────────────┐      ┌─────────────┐
│ Zyxel 24HP  │      │ ONTI 8-port │
│ FRONTEND    │      │ BACKEND 10G │
└──────┬──────┘      └──────┬──────┘
       │                    │
       │                    ▼
       │             ┌─────────────┐
       │             │ Tenda 2.5G  │
       │             │ BACKEND 1G  │
       │             └─────────────┘
       │
   ┌───┴───────────────┐
   │                   │
   ▼                   ▼
Balcony            Basement
(WiFi Bridge)      (PoE Extender)
   │                   │
   ~~~200m 5GHz~~~     │
   ▼                   ▼
Garage             VvE Cameras
(WireGuard)        (isolated)
```

## Switches

| Switch | Model | Ports | Role |
|--------|-------|-------|------|
| Zyxel | GS1900-24HPv1 | 24+2 SFP | Frontend PoE |
| ONTI | S508CL-8S | 8 SFP+ | Backend 10G |
| Tenda | TEM2007 | 7+1 SFP+ | Backend bridge |

## Traffic Rules

| From → To | Allowed? |
|-----------|----------|
| Frontend → Backend | Via proxy only (VLAN 150) |
| Backend → Frontend | **BLOCKED** |
| Backend → Internet | **BLOCKED** |
| Frontend → Internet | Firewalled |

## Storage Protocol Split

| Zone | Protocol | Purpose |
|------|----------|---------|
| Backend | NFS only | Linux service-to-service |
| Frontend | SMB only | Windows/Mac clients |

## Related

- [[homelab/network/ip-plan]]
- [[homelab/network/vlan-design]]
- [[homelab/network/security]]
- [[homelab/hosts]]
