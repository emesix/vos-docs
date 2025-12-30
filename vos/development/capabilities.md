---
title: Pre-Staging Claude Desktop Capabilities
description: Complete list of what Claude Desktop can do in the VOS Pre-Staging environment with unrestricted MCP access
published: true
date: 2025-12-25T20:26:29.441Z
tags: mcp, claude, vos, staging, capabilities
editor: markdown
dateCreated: 2025-12-25T20:26:27.166Z
---

---
title: "Pre-Staging Claude Desktop Capabilities"
uid: "prestaging-capabilities"
kind: [reference, specification]
category: "04"
sub-category: "01"
visibility: 0.0
status: 1.0
tags: [prestaging, mcp, capabilities, desktop-commander]
parent_uid: "infra-prestaging-overview"
---

# Pre-Staging Claude Desktop Capabilities

**Category:** 04.01 - Runtime Environments  
**Purpose:** Reference for available operations in Pre-Staging environment

## Overview

With Desktop Commander MCP configured for unrestricted access, Claude Desktop has full system administration capabilities on iZombie. This document catalogs available operations.

## System Administration

```bash
sudo systemctl start/stop/restart any-service
sudo pacman -S install-any-package
sudo mount -t nfs <LAN_IP>:/share /mnt/nas
sudo docker run -d any-container
sudo iptables -A any-firewall-rule
sudo useradd/usermod/passwd user-management
sudo chmod/chown permission-changes
```

## Network Operations

```bash
ssh root@<LAN_IP>    # SSH to Proxmox
ssh root@<LAN_IP>     # SSH to router
scp file.txt root@remote:/path/
rsync -av /local/ remote:/backup/
ping/traceroute/mtr network-diagnostics
nmcli/ip/route network-configuration
sudo tcpdump packet-capture
```

## Docker Operations

```bash
docker run/stop/start/rm containers
docker-compose up/down stacks
docker exec -it container bash
docker network create/rm networks
docker volume create/rm volumes
docker build/push/pull images
```

## File System

Complete access to all paths including /root, /etc, /var, /sys, /proc. Can mount NFS/SMB/SSHFS shares and modify system configurations.

## Package Management

```bash
sudo pacman -S package     # Install
sudo pacman -Syu           # System update
yay -S aur-package         # AUR packages
npm install -g package     # Node.js global
pip install --break-system-packages  # Python
```

## Limitations

None within iZombie. Production systems require SSH key setup for remote access.

## Related

- [[vos/development/overview]]
- [[vos/development/mcp-config]]