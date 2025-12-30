---
title: Pre-Staging Arch Linux Setup
description: Step-by-step Arch Linux setup guide for VOS Pre-Staging environment with Claude Desktop and MCP
published: true
date: 2025-12-25T20:27:04.925Z
tags: vos, staging, arch-linux, setup, howto
editor: markdown
dateCreated: 2025-12-25T20:27:02.638Z
---

---
title: "Pre-Staging Arch Linux Setup"
uid: "prestaging-arch-setup"
kind: [howto, procedure, stable]
category: "03"
sub-category: "01"
visibility: 0.0
status: 1.0
tags: [prestaging, arch-linux, setup, nodejs, claude-desktop]
parent_uid: "infra-prestaging-overview"
---

# Pre-Staging Arch Linux Setup

**Category:** 03.01 - Operating Systems  
**Purpose:** Install and configure Pre-Staging environment on Arch Linux

## Overview

iZombie runs Arch Linux, not macOS. Configuration paths and package management differ from Darwin systems. This guide covers the complete setup process.

## Prerequisites

Arch Linux with working network connection. User account with sudo privileges.

## Step 1: System Update and Dependencies

```bash
sudo pacman -Syu
sudo pacman -S nodejs npm openssh docker docker-compose nfs-utils git base-devel
```

## Step 2: Enable Docker

```bash
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER
```

Log out and back in for group membership to apply.

## Step 3: Install Claude Desktop

```bash
yay -S claude-desktop
```

Alternatively, download AppImage from claude.ai/download.

## Step 4: Install Desktop Commander

```bash
sudo npm install -g @wonderwhy-er/desktop-commander
```

## Step 5: Configure MCP

```bash
mkdir -p ~/.config/Claude
nano ~/.config/Claude/claude_desktop_config.json
```

Add configuration from [[vos/development/mcp-config]].

## Step 6: Enable Passwordless Sudo

```bash
sudo visudo
# Add line:
# user ALL=(ALL) NOPASSWD: ALL
```

Critical for Desktop Commander to execute sudo commands without prompts.

## Step 7: Create Workspace

```bash
mkdir -p ~/vos-workspace/{network-fixes,configs,scripts,docs}
cd ~/vos-workspace
git init
git config user.name "user"
git config user.email "user@example.com"
```

## Step 8: Restart and Verify

```bash
killall claude-desktop
claude-desktop &
```

Run verification tests from [[vos/development/verification-tests]].

## Path Differences from macOS

| Component | macOS | Arch Linux |
|-----------|-------|------------|
| Config | `~/Library/Application Support/Claude/` | `~/.config/Claude/` |
| Package Manager | `brew` | `pacman` / `yay` |
| Docker | Docker Desktop | `pacman -S docker` |

## Related

- [[vos/development/overview]]
- [[vos/development/mcp-config]]
- [[vos/development/verification-tests]]