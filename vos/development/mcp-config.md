---
title: Pre-Staging MCP Server Configuration
description: Claude Desktop MCP server configuration for VOS Pre-Staging environment on Arch Linux
published: true
date: 2025-12-25T20:26:47.192Z
tags: mcp, claude, vos, staging, configuration
editor: markdown
dateCreated: 2025-12-25T20:26:45.012Z
---

---
title: "Pre-Staging MCP Server Configuration"
uid: "prestaging-mcp-config"
kind: [component, software, configuration]
category: "05"
sub-category: "01"
visibility: 3.0
status: 1.0
tags: [prestaging, mcp, configuration, desktop-commander, arch-linux]
parent_uid: "infra-prestaging-overview"
---

# Pre-Staging MCP Server Configuration

**Category:** 05.01 - Core Services  
**Purpose:** MCP server setup for unrestricted Claude Desktop access

## Overview

Claude Desktop on Arch Linux uses ~/.config/Claude/claude_desktop_config.json for MCP server configuration. The Pre-Staging environment uses Desktop Commander with no restrictions.

## Configuration File

**Location:** `~/.config/Claude/claude_desktop_config.json`

```json
{
  "mcpServers": {
    "desktop-commander": {
      "command": "npx",
      "args": ["-y", "@wonderwhy-er/desktop-commander@latest"],
      "env": {
        "ALLOWED_DIRECTORIES": "/",
        "BLOCKED_COMMANDS": "",
        "DEFAULT_SHELL": "bash"
      }
    },
    "wikijs": {
      "command": "python",
      "args": ["/path/to/wikijs-mcp-server.py"],
      "env": {
        "WIKIJS_URL": "http://localhost:3000",
        "WIKIJS_API_KEY": "your-api-key"
      }
    },
    "strudel": {
      "command": "node",
      "args": ["/path/to/strudel-mcp-server.js"]
    }
  }
}
```

## Configuration Parameters

| Parameter | Value | Effect |
|-----------|-------|--------|
| ALLOWED_DIRECTORIES | `/` | Full filesystem access |
| BLOCKED_COMMANDS | `""` | No command restrictions |
| DEFAULT_SHELL | `bash` | Bash as default shell |

## Verification

After configuration changes, restart Claude Desktop:

```bash
killall claude-desktop
claude-desktop &
```

Test with: "Can you run 'sudo pacman -Q | head -5'?"

Expected: Package list, no password prompt.

## Production Note

This unrestricted configuration is only for Pre-Staging. Production environments will use restricted whitelists:

```json
{
  "ALLOWED_COMMANDS": "docker,systemctl,ls,cat,ip,ping,ssh",
  "ALLOWED_DIRECTORIES": ["/etc/", "/var/", "/home/"]
}
```

## Related

- [[vos/development/overview]]
- [[vos/development/arch-setup]]
- [[vos/development/verification-tests]]