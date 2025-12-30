---
title: VOS Authority Map
description: What is authoritative vs derived vs historical
published: true
date: 2025-12-29T12:30:00.000Z
tags: vos, authority, ssot
editor: markdown
dateCreated: 2025-12-29T12:30:00.000Z
---

# VOS Authority Map

Defines what documents are source-of-truth vs generated vs historical.

## Authority Levels

| Level | Meaning | Example |
|-------|---------|---------|
| **Authoritative** | Defines reality. Change here first. | services.md, ip-plan.md |
| **Derived** | Generated from authoritative sources | Host status, metrics |
| **Reference** | Stable knowledge, rarely changes | Component specs, howtos |
| **Historical** | Past decisions, context only | Design notes, archives |

## Authoritative Documents

These define truth. If reality differs, reality is wrong.

| Document | Scope |
|----------|-------|
| [[vos/services]] | What runs where |
| [[homelab/network/ip-plan]] | IP assignments |
| [[homelab/network/vlan-design]] | VLAN structure |
| [[homelab/identity/users-groups]] | Identity definitions |
| [[homelab/hosts]] | Host inventory |

## Derived Documents

Generated or updated from authoritative sources.

| Document | Source |
|----------|--------|
| Host status pages | Proxmox API |
| Service health | Monitoring stack |
| Backup reports | PBS logs |

## Reference Documents

Stable knowledge. Update when specs change.

| Document | Type |
|----------|------|
| [[components/*]] | Hardware/software specs |
| [[howto/*]] | Procedures |
| [[homelab/network/architecture]] | Design patterns |
| [[homelab/identity]] | Protocol overview |

## Historical Documents

Context and reasoning. Safe to archive.

| Document | Purpose |
|----------|---------|
| Design notes | Why decisions were made |
| Migration logs | What changed when |
| Exploration docs | Options considered |

## Regeneration Rules

| If deleted... | Can regenerate from... |
|---------------|------------------------|
| Host pages | Proxmox API + ip-plan.md |
| Service status | Running containers |
| Component specs | Vendor docs |
| Howtos | Must be rewritten |
| Authoritative docs | **Cannot regenerate** |

## Update Flow

```
Reality changes
      ↓
Update AUTHORITATIVE doc
      ↓
Derived docs auto-update (or manual refresh)
      ↓
Reference docs: check if affected
      ↓
Historical: no action needed
```

## Related

- [[vos/README]]
- [[vos/services]]
- [[vos/document-pipeline]]
