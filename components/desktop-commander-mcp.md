---
title: Desktop Commander MCP Server
description: Off-the-shelf MCP server providing unrestricted terminal and filesystem access for Claude Desktop
published: true
date: 2025-12-25T20:28:47.260Z
tags: mcp, claude, automation, component, software
editor: markdown
dateCreated: 2025-12-25T20:28:44.901Z
---

---
title: "Desktop Commander MCP Server"
uid: "component-desktop-commander-mcp"
kind: [component, software, service]
category: "05"
sub-category: "01"
visibility: 0.0
status: 1.0
tags: [mcp, desktop-commander, claude-desktop, terminal, filesystem]
parent_uid: ""
---

# Desktop Commander MCP Server

**Category:** 05.01 - Core Services  
**Purpose:** Terminal and filesystem access for Claude Desktop via MCP

## Overview

Desktop Commander is an off-the-shelf MCP server that provides Claude Desktop with terminal control, filesystem access, and interactive process management. Built on the official MCP Filesystem server with extended capabilities.

## Source

**GitHub:** https://github.com/wonderwhy-er/DesktopCommanderMCP  
**NPM:** @wonderwhy-er/desktop-commander

## Capabilities

| Feature | Support |
|---------|--------|
| Local Files | Full access (configurable) |
| SUDO | Via terminal |
| SSH | Via terminal |
| SCP | Via terminal |
| Docker | Via terminal |
| Interactive Processes | Yes |
| Excel/PDF | Yes |
| Fuzzy Search | Yes |
| Git Diff Editing | Yes |

## Installation

```bash
npm install -g @wonderwhy-er/desktop-commander
```

Or use npx for automatic latest version.

## Configuration

```json
{
  "command": "npx",
  "args": ["-y", "@wonderwhy-er/desktop-commander@latest"],
  "env": {
    "ALLOWED_DIRECTORIES": "/",
    "BLOCKED_COMMANDS": "",
    "DEFAULT_SHELL": "bash"
  }
}
```

## Environment Variables

**ALLOWED_DIRECTORIES:** Filesystem paths accessible. Set to "/" for full access.

**BLOCKED_COMMANDS:** Commands to block. Empty string allows all.

**DEFAULT_SHELL:** Shell for command execution. Usually bash or zsh.

## Comparison to Official Servers

Desktop Commander combines and extends official MCP servers:

- @modelcontextprotocol/server-shell: Limited commands
- @modelcontextprotocol/server-filesystem: Limited to configured paths
- Desktop Commander: Both combined, plus Excel/PDF, fuzzy search

## Used By

- [[host-izombie]]
- VOS Pre-Staging environment

## Related

- [[vos/development/mcp-config]]
- [[vos/development/overview]]