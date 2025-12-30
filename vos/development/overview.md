---
title: VOS Development Environment
description: Pre-staging environment on iZombie with Claude Desktop and MCP servers
published: true
date: 2025-12-29T12:00:00.000Z
tags: vos, development, mcp, claude-desktop
editor: markdown
dateCreated: 2025-12-25T20:26:08.707Z
---

# VOS Development Environment

AI-assisted development environment for infrastructure planning and documentation.

## Host

| Property | Value |
|----------|-------|
| Machine | iZombie ([[izombie]]) |
| Hardware | iMac 2014 |
| OS | Arch Linux |
| Desktop | KDE Plasma (Wayland) |

## Core Components

| Component | Purpose |
|-----------|---------|
| Claude Desktop | AI development interface |
| Desktop Commander MCP | System access (passwordless sudo) |
| WikiJS MCP | Documentation automation |
| Proxmox MCP | Cluster management |
| Playwright MCP | Browser automation |
| Strudel MCP | Music synthesis |

## Security Note

Passwordless sudo enabled. Acceptable for isolated development machine.

## Capabilities

- Full system access via Desktop Commander
- Wiki.js page creation and editing
- Proxmox VM/LXC management
- Network documentation
- Configuration validation

## Workflow

1. Claude Desktop receives task
2. Executes via MCP servers
3. Documents findings in Wiki.js
4. Version controls with Git

## Status

**Phase:** Complete ✅  
**Next:** Network infrastructure deployment

## Related

- [[izombie]]
- [[vos/architecture]]
- [[vos/development/mcp-config]]
- [[vos/development/capabilities]]
