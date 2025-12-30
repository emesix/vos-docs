---
title: Pre-Staging Verification Tests
description: Test suite to verify Claude Desktop MCP configuration is working correctly in VOS Pre-Staging environment
published: true
date: 2025-12-25T20:27:24.063Z
tags: mcp, vos, staging, testing, verification
editor: markdown
dateCreated: 2025-12-25T20:27:21.808Z
---

---
title: "Pre-Staging Verification Tests"
uid: "prestaging-verification-tests"
kind: [howto, procedure, stable]
category: "04"
sub-category: "01"
visibility: 0.0
status: 1.0
tags: [prestaging, verification, testing, mcp, desktop-commander]
parent_uid: "infra-prestaging-overview"
---

# Pre-Staging Verification Tests

**Category:** 04.01 - Runtime Environments  
**Purpose:** Verify Claude Desktop MCP configuration works correctly

## Overview

Run these tests in Claude Desktop after setup to confirm all capabilities are functional. All tests should execute without password prompts.

## Test 1: Basic Sudo

**Prompt:** "Can you run 'sudo pacman -Q | head -5' for me?"

**Expected:** First 5 packages listed, no password prompt.

## Test 2: Root Filesystem

**Prompt:** "Can you run 'sudo ls -la /root' for me?"

**Expected:** Contents of /root directory.

## Test 3: System Service

**Prompt:** "Can you check if Docker is running with 'sudo systemctl status docker'?"

**Expected:** Docker service status output.

## Test 4: Docker Command

**Prompt:** "Can you run 'docker ps' for me?"

**Expected:** List of running containers or empty table.

## Test 5: Package Installation

**Prompt:** "Can you install 'htop' with pacman if it's not already installed?"

**Expected:** Installs htop or reports already installed.

## Test 6: File Creation in /etc

**Prompt:** "Can you create a test file at /etc/vos-test.txt with content 'VOS Pre-Staging Test'?"

**Expected:** File created successfully.

**Cleanup:** Remove test file after verification.

## Test 7: Configuration Check

**Prompt:** "Show me your Desktop Commander configuration"

**Expected:** Shows allowedDirectories: "/", blockedCommands: ""

## Test 8: Multi-Step Workflow

**Prompt:**
```
Can you:
1. Check if ~/vos-workspace exists
2. If not, create it
3. Create ~/vos-workspace/test.md
4. Write 'VOS Pre-Staging Ready!' to it
5. Show me the contents
```

**Expected:** All steps complete successfully.

## Test 9: SSH (Optional)

**Prompt:** "Can you SSH to root@<LAN_IP> and run 'hostname'?"

**Expected:** Proxmox hostname if SSH keys configured, or password prompt indication.

## Test 10: NFS Mount (Optional)

**Prompt:** "Can you try to mount NAS at <LAN_IP>:/share to /mnt/test?"

**Expected:** Mount succeeds or shows specific network error.

## Pass Criteria

Tests 1-8 must pass. Tests 9-10 depend on network configuration.

If tests fail, check:
- Passwordless sudo configuration
- MCP config file location
- Claude Desktop restart

## Related

- [[vos/development/overview]]
- [[vos/development/mcp-config]]
- [[vos/development/arch-setup]]