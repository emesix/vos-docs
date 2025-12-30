---
title: Proxmox Helper Scripts - Browser Automation for Script Analysis
description: Documents the web_fetch limitation encountered during Proxmox Helper Scripts research and the MCP browser plugin solution for unrestricted URL access.
published: true
date: 2025-12-26T07:41:42.169Z
tags: mcp, automation, proxmox, browser, workaround, playwright
editor: markdown
dateCreated: 2025-12-26T07:41:39.883Z
---

---
title: "Proxmox Helper Scripts - Browser Automation for Script Analysis"
uid: "howto-proxmox-helper-scripts-browser-automation"
kind: [howto, workaround, reference]
category: "05"
sub-category: "05"
visibility: 3.0
status: 4.0
tags: [proxmox, automation, browser, mcp, workaround, playwright]
parent_uid: ""
---

# Proxmox Helper Scripts - Browser Automation for Script Analysis

**Category:** 05.05 - Techniques & Methods  
**Purpose:** Document the web_fetch limitation and browser MCP solution for script analysis

## Overview

During VOS Phase 1 planning, we needed to analyze the actual Proxmox Helper Scripts source code to understand their internal structure. Claude's built-in `web_fetch` tool has a provenance restriction that blocked direct access to raw GitHub URLs. This document captures the problem, root cause, and the MCP browser plugin solution.

## The Problem

When attempting to fetch raw script files from GitHub, the `web_fetch` tool returned permission errors, preventing analysis of the actual script implementation.

### Failed Fetch Attempt

```
URL: https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/sonarr.sh

Error: PERMISSIONS_ERROR
Message: "This URL cannot be fetched because it was not provided 
         by the user nor did it appear in any search/fetch results"
```

### What Worked vs. What Failed

| URL Type | Example | Result |
|----------|---------|--------|
| Search result URLs | `github.com/community-scripts/ProxmoxVE` | Success |
| GitHub discussion pages | `github.com/.../discussions/9590` | Success |
| Raw GitHub content | `raw.githubusercontent.com/...` | Blocked |
| Constructed URLs | Any URL not from search/user | Blocked |

## Root Cause Analysis

The `web_fetch` tool enforces a **provenance check** that requires URLs to originate from one of two authorized sources. The first authorized source is explicit user input, meaning URLs the user pastes directly into the conversation. The second authorized source is search results, meaning URLs that appeared in a previous `web_search` response.

This is a security constraint, not a blacklist. The system verifies where each URL came from before allowing the fetch. URLs constructed programmatically or inferred from patterns are rejected regardless of the target domain.

### Why This Constraint Exists

The provenance check serves three security purposes. First, it prevents arbitrary web scraping by ensuring Claude can only fetch content contextually relevant to the conversation. Second, it protects against prompt injection via URLs by blocking fetches of pages that might contain adversarial instructions. Third, it maintains conversation relevance by keeping web access tightly scoped.

### Limitation Scope

This constraint applies only to Anthropic's built-in `web_fetch` tool. It does not affect MCP servers, command-line tools like curl, or browser automation plugins, all of which operate outside this restriction.

## The Solution: MCP Browser Plugin

Adding a Playwright-based browser MCP server bypasses the `web_fetch` provenance restriction entirely. The browser operates as a separate tool with its own permissions model.

### Capabilities Gained

The MCP browser plugin provides unrestricted URL navigation via `playwright:browser_navigate` to any URL. It also enables full page interaction including clicking, typing, form filling, and JavaScript execution. Visual debugging through screenshots and accessibility snapshots becomes available. Finally, it handles JavaScript-heavy pages that render dynamically, which simple fetch cannot process.

### Configuration

The Playwright MCP server integrates with Claude Desktop's MCP configuration. Once enabled, the `playwright:*` tools become available for browser automation tasks.

### Example Usage

To fetch a raw GitHub script that `web_fetch` would block, you can use the browser navigation approach. Navigate to the raw script URL, then capture the page content via snapshot or screenshot.

```
# Navigate to blocked URL
playwright:browser_navigate -> https://raw.githubusercontent.com/.../sonarr.sh

# Capture content
playwright:browser_snapshot -> Returns page content
```

## Alternative Workarounds

If the browser MCP is unavailable, two other approaches exist.

### Option 1: User Provides URL

When a user pastes a URL directly in their message, it becomes "user-provided" and `web_fetch` will accept it. This requires manual intervention for each URL.

### Option 2: Desktop Commander curl

Since Desktop Commander has unrestricted system access, curl can fetch any URL directly on the local machine.

```bash
# Fetch via Desktop Commander process
curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/sonarr.sh
```

This approach works because the constraint applies to Claude's `web_fetch` tool specifically, not to system commands executed via MCP.

## Implications for VOS Planning

When analyzing external scripts, configurations, or documentation during VOS planning sessions, the team should be aware of these access patterns.

### Recommended Approach

For quick URL checks, have the user paste the URL directly. For systematic script analysis, use the Playwright browser MCP. For downloading files to the local system, use Desktop Commander with curl or wget. For searching and discovering URLs, the built-in `web_search` works fine since discovered URLs become fetchable.

### Documentation Pattern

When documenting external resources in wiki pages, include both the human-readable URL and note whether it requires browser access. This helps future sessions understand access requirements.

## Related

- [[howto-proxmox-helper-scripts-automation]]
- [[plan-vos-phase-1-network-arr]]
- [[vos-prestaging-claude-desktop-mcp-setup]]