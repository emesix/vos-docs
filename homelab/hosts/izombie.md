---
title: iZombie - Development Host
description: iMac 2014 running Arch Linux for VOS pre-staging development
published: true
date: 2025-12-29T12:00:00.000Z
tags: host, development, arch-linux, mcp
editor: markdown
dateCreated: 2025-12-25T20:27:40.370Z
---

# iZombie - Development Host

**Role:** VOS Pre-staging development  
**Location:** Desktop  
**Network:** <LAN_IP>

## Overview

iMac 2014 "rescued from Tim Cook" running Arch Linux. Isolated development environment for VOS infrastructure planning with Claude Desktop and MCP servers.

## Hardware

| Component | Specification |
|-----------|---------------|
| Model | iMac 2014 |
| CPU | Intel (integrated) |
| RAM | Integrated |
| Display | Built-in Retina |

## Software

| Component | Status |
|-----------|--------|
| OS | Arch Linux 6.18.2 |
| Desktop | KDE Plasma (Wayland) |
| Claude Desktop | Running |
| Docker | Running |

## MCP Servers

| Server | Purpose |
|--------|---------|
| Desktop Commander | System access |
| WikiJS | Documentation |
| Proxmox | Cluster management |
| Playwright | Browser automation |
| Strudel | Music synthesis |

## Security

Passwordless sudo enabled. Acceptable for isolated development.

## Related

- [[vos/development/overview]]
- [[vos/development/mcp-config]]
- [[homelab/hosts]]
