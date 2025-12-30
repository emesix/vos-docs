---
title: VOS Homelab Wiki
description: Single Source of Truth for user Operating System
published: true
date: 2025-12-29T12:30:00.000Z
tags: vos, homelab, wiki, index
editor: markdown
dateCreated: 2025-12-14T16:14:10.904Z
---

# VOS Homelab Wiki

Documentation for **user Operating System** - a unified operating environment.

## Start Here

→ **[[vos/README]]** - What VOS is, in one screen

## VOS Core

| Document | Purpose |
|----------|---------|
| [[vos/README]] | What VOS is |
| [[vos/architecture]] | Planes & responsibilities |
| [[vos/services]] | What runs where |
| [[vos/authority]] | What is authoritative vs derived |
| [[vos/document-pipeline]] | Knowledge flow as infrastructure |

## Infrastructure

| Document | Purpose |
|----------|---------|
| [[homelab/overview]] | Compute summary |
| [[homelab/hosts]] | All physical machines |
| [[homelab/network/architecture]] | Dual-zone network |
| [[homelab/network/ip-plan]] | IP assignments |
| [[homelab/identity]] | Authentik/LDAP/RADIUS |

## Reference

| Section | Contents |
|---------|----------|
| [[components/]] | Hardware/software specs |
| [[howto/]] | Step-by-step procedures |
| [[vos/development/overview]] | Pre-staging environment |

## Hosts

| Host | Role |
|------|------|
| [[hal]] | OPNsense router |
| [[deep-thought]] | Databases, Wiki |
| [[napster]] | ARR stack |
| [[mothership]] | Docker, MCP Gateway |
| [[skynet]] | AI controller |
| [[watson]] | GPU inference |
| [[alexandria]] | NAS (117TB) |

## Status

**Phase:** Network Foundation  
**Next:** VLAN deployment  
**Dev:** [[izombie]]
