---
title: Proxmox Helper Scripts - Automated LXC Deployment
description: How to automate Proxmox VE Helper Scripts deployments by bypassing the TUI using environment variables for unattended container creation.
published: true
date: 2025-12-26T06:08:26.411Z
tags: automation, howto, proxmox, lxc, deployment, helper-scripts
editor: markdown
dateCreated: 2025-12-26T06:08:23.685Z
---

---
title: "Proxmox Helper Scripts - Automated LXC Deployment"
uid: "howto-proxmox-helper-scripts-automation"
kind: [howto, procedure, automation]
category: "05"
sub-category: "05"
visibility: 0.0
status: 4.0
tags: [proxmox, automation, lxc, deployment, helper-scripts]
parent_uid: ""
---

# Proxmox Helper Scripts - Automated LXC Deployment

**Category:** 05.05 - Techniques & Methods  
**Purpose:** Bypass TUI for fully automated LXC container deployments

## Overview

Proxmox VE Helper Scripts (community-scripts.github.io/ProxmoxVE) provide one-command LXC container installations. By default, scripts use a TUI for configuration. Environment variable overrides enable fully automated, non-interactive deployments.

The scripts support a three-tier defaults system with priority order: Environment Variables → App Defaults → User Defaults → Built-in Defaults.

## Environment Variable Override Syntax

Prefix variables with `var_` and prepend to the bash command:

```bash
var_cpu=4 var_ram=4096 var_disk=20 \
  bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/debian.sh)"
```

## Available Variables

### Resources
| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `var_cpu` | Integer | 1 | CPU cores |
| `var_ram` | Integer | 1024 | RAM in MB |
| `var_disk` | Integer | 4 | Disk size in GB |
| `var_unprivileged` | Boolean | 1 | 1=unprivileged, 0=privileged |

### Network Configuration
| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `var_net` | String | dhcp | `dhcp` or `static` |
| `var_brg` | String | vmbr0 | Network bridge |
| `var_gateway` | IP | Auto | Gateway IP address |
| `var_ipv6_method` | String | none | `auto/dhcp/static/none/disable` |
| `var_vlan` | Integer | - | VLAN tag |
| `var_mtu` | Integer | 1500 | MTU size |
| `var_mac` | MAC | Auto | MAC address |
| `var_ns` | IP | Auto | DNS nameserver |

### Identity & SSH
| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `var_hostname` | String | App name | Container hostname |
| `var_pw` | String | Random | Root password (empty=auto-login) |
| `var_tags` | String | community-script | Tags (comma-separated) |
| `var_ssh` | Boolean | no | Enable SSH server |
| `var_ssh_authorized_key` | String | - | SSH public key |

### Container Features
| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `var_nesting` | Boolean | 1 | Container nesting (Docker) |
| `var_fuse` | Boolean | 0 | FUSE filesystem support |
| `var_keyctl` | Boolean | 0 | Keyctl support |
| `var_protection` | Boolean | no | Deletion protection |

## VOS ARR Stack Deployment Examples

### Sonarr
```bash
var_unprivileged=1 \
var_cpu=2 \
var_ram=2048 \
var_disk=16 \
var_hostname=sonarr \
var_brg=vmbr0 \
var_net=dhcp \
var_ipv6_method=disable \
var_ssh=yes \
var_tags=arr,media,vos \
  bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/sonarr.sh)"
```

### Radarr
```bash
var_unprivileged=1 \
var_cpu=2 \
var_ram=2048 \
var_disk=16 \
var_hostname=radarr \
var_brg=vmbr0 \
var_net=dhcp \
var_ipv6_method=disable \
var_ssh=yes \
var_tags=arr,media,vos \
  bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/radarr.sh)"
```

### Prowlarr
```bash
var_unprivileged=1 \
var_cpu=2 \
var_ram=1024 \
var_disk=8 \
var_hostname=prowlarr \
var_brg=vmbr0 \
var_net=dhcp \
var_ipv6_method=disable \
var_ssh=yes \
var_tags=arr,indexer,vos \
  bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/prowlarr.sh)"
```

## Batch Deployment Script

```bash
#!/bin/bash
# vos-arr-deploy.sh - Deploy ARR stack containers

arr_apps=(
  "prowlarr:2:1024:8"
  "sonarr:2:2048:16"
  "radarr:2:2048:16"
  "lidarr:2:2048:16"
  "readarr:2:2048:16"
)

for app_config in "${arr_apps[@]}"; do
  IFS=':' read -r app cpu ram disk <<< "$app_config"
  
  echo "Deploying $app..."
  var_cpu=$cpu \
  var_ram=$ram \
  var_disk=$disk \
  var_hostname=$app \
  var_net=dhcp \
  var_ipv6_method=disable \
  var_ssh=yes \
  var_tags=arr,media,vos \
    bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/${app}.sh)"
  
  echo "✓ $app deployed"
  sleep 5
done
```

## Saved Defaults (Alternative Method)

Create persistent defaults on Proxmox host:

**Global Defaults:**
```bash
mkdir -p /usr/local/community-scripts
cat > /usr/local/community-scripts/default.vars <<'EOF'
var_unprivileged=1
var_cpu=2
var_ram=2048
var_brg=vmbr0
var_net=dhcp
var_ipv6_method=disable
var_ssh=yes
var_tags=vos
EOF
```

**App-Specific Defaults:**
```bash
mkdir -p /usr/local/community-scripts/defaults
cat > /usr/local/community-scripts/defaults/sonarr.vars <<'EOF'
var_cpu=2
var_ram=2048
var_disk=16
var_hostname=sonarr
var_tags=arr,media,vos
EOF
```

## Priority System

```
1. Environment Variables (var_*)     ← HIGHEST
2. App-Specific Defaults (.vars)     
3. User Global Defaults (default.vars)
4. Built-in Script Defaults          ← LOWEST
```

Environment variables always override all other sources.

## Security Notes

The scripts validate variables against a whitelist. Command injection patterns (`$(...)`, backticks, semicolons) are rejected. Only whitelisted `var_*` variables are accepted.

## Available Scripts for VOS

Relevant scripts at community-scripts.github.io/ProxmoxVE:
- **Media:** sonarr, radarr, lidarr, readarr, bazarr, jellyfin, plex
- **Indexers:** prowlarr, jackett
- **Download:** qbittorrent, sabnzbd, nzbget, transmission
- **Networking:** nginx-proxy-manager, traefik, pihole, adguard
- **AI Stack:** ollama, open-webui, n8n
- **Infrastructure:** docker, postgresql, redis, wikijs

## Related
- [[infra-network-vlan-design]]
- [[howto-proxmox-lxc-best-practices]]
- [[05-01/app-arr-stack-design]]