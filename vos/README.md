---
title: VOS - user Operating System
description: What VOS is, in one screen
published: true
date: 2025-12-29T12:30:00.000Z
tags: vos, readme, index
editor: markdown
dateCreated: 2025-12-29T12:30:00.000Z
---

# VOS - user Operating System

A unified operating environment for homelab infrastructure, AI services, and knowledge management.

## What VOS Is

VOS is not a Linux distro. It's an **integration layer** that:

- Unifies 7 compute nodes under one identity system
- Routes AI requests through a central brain (skynet)
- Maintains a living knowledge base (this wiki)
- Treats documentation as infrastructure

## The Three Planes

| Plane | Purpose | Owner |
|-------|---------|-------|
| **Control** | AI orchestration, routing, decisions | skynet (8845HS) |
| **Data** | Persistence, databases, knowledge | deep-thought + alexandria |
| **Compute** | Workloads, containers, inference | mothership, watson, napster |

## Core Subsystems

| Subsystem | Description | Entry Point |
|-----------|-------------|-------------|
| Network | Dual-zone VLAN architecture | [[homelab/network/architecture]] |
| Identity | Authentik (LDAP/RADIUS/OIDC) | [[homelab/identity]] |
| Knowledge | Wiki.js + document pipeline | [[vos/document-pipeline]] |
| AI | LiteLLM → vLLM/Haystack/MCP | [[vos/services]] |

## Where Things Live

| Question | Answer |
|----------|--------|
| What runs where? | [[vos/services]] |
| What is authoritative? | [[vos/authority]] |
| How is it connected? | [[homelab/network/ip-plan]] |
| How do I access it? | [[homelab/identity]] |

## Current State

**Phase:** Network Foundation  
**Next:** VLAN deployment on hal (OPNsense)  
**Development:** Active on [[izombie]]

## Related

- [[vos/architecture]] - Detailed layer breakdown
- [[vos/services]] - Service assignments
- [[vos/authority]] - Document authority map
- [[homelab/overview]] - Infrastructure summary
