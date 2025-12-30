---
title: Emergency Access and Recovery
description: SSH key access and recovery procedures when Authentik or network is unavailable
published: true
date: 2025-12-28T19:16:02.441Z
tags: backup, security, ssh, emergency, recovery, maintenance
editor: markdown
dateCreated: 2025-12-28T19:15:59.921Z
---

---
title: "Emergency Access and Recovery"
uid: "infra-network-emergency"
kind: [infrastructure, security, operations]
category: "01"
sub-category: "02"
visibility: 3.0
status: 3.0
tags: [ssh, emergency, recovery, maintenance, security, backup]
parent_uid: ""
---

# Emergency Access and Recovery

**Category:** 01.02 - Network Infrastructure  
**Purpose:** Access methods when Authentik or network is down  
**Updated:** 2025-12-28

## Overview

Authentik dependency means identity system failure locks out everything. Emergency access via SSH keys bypasses Authentik entirely. Always works.

## Maintenance Hosts

| Host | Location | Purpose |
|------|----------|--------|
| iZombie | Office desk | Primary maintenance |
| (USB boot) | Drawer | Emergency recovery |

## SSH Key Access

```
┌─────────────────────────────────────────────────────────────┐
│                 EMERGENCY ACCESS PATH                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   iZombie (maintenance host)                                │
│   ~/.ssh/id_ed25519 (emergency key)                         │
│        │                                                    │
│        │  SSH with key auth                                 │
│        │  No LDAP, no Authentik, no network dependency      │
│        ▼                                                    │
│   ┌───────────────────────────────────────────┐              │
│   │  Proxmox hosts (as root)                  │              │
│   │  OPNsense (as root)                       │              │
│   │  LXC containers (as root)                 │              │
│   │  Unraid (as root)                         │              │
│   └───────────────────────────────────────────┘              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Generate Emergency Key

```bash
# On iZombie
ssh-keygen -t ed25519 -C "user@izombie-emergency" -f ~/.ssh/id_ed25519_emergency
```

### Deploy to All Hosts

```bash
# Proxmox hosts
ssh-copy-id -i ~/.ssh/id_ed25519_emergency root@<LAN_IP>
ssh-copy-id -i ~/.ssh/id_ed25519_emergency root@<LAN_IP>

# OPNsense
ssh-copy-id -i ~/.ssh/id_ed25519_emergency root@<LAN_IP>

# Unraid
ssh-copy-id -i ~/.ssh/id_ed25519_emergency root@<LAN_IP>
```

### SSH Config

```bash
# ~/.ssh/config on iZombie

Host pve-qotom
    HostName <LAN_IP>
    User root
    IdentityFile ~/.ssh/id_ed25519_emergency

Host pve-5800x
    HostName <LAN_IP>
    User root
    IdentityFile ~/.ssh/id_ed25519_emergency

Host opnsense
    HostName <LAN_IP>
    User root
    IdentityFile ~/.ssh/id_ed25519_emergency

Host nas
    HostName <LAN_IP>
    User root
    IdentityFile ~/.ssh/id_ed25519_emergency
```

## Backup Key Storage

Keep second copy of private key offline:

1. Export key: `cat ~/.ssh/id_ed25519_emergency`
2. Store on encrypted USB stick
3. Keep USB in secure location (drawer, safe)
4. Label: "VOS Emergency SSH Key"

## Recovery Scenarios

### Authentik Down

| Impact | Mitigation |
|--------|------------|
| Web SSO broken | Access services directly by IP |
| WiFi auth fails | Connect via ethernet |
| LDAP auth fails | SSH keys still work |

**Steps:**
1. SSH to Proxmox: `ssh pve-qotom`
2. Check Authentik container: `pct list`, `pct start <id>`
3. Check logs: `pct exec <id> -- journalctl -u authentik`

### OPNsense Down

| Impact | Mitigation |
|--------|------------|
| No routing | Connect directly to Proxmox IPMI/console |
| No DNS | Use IP addresses directly |
| No DHCP | Static IP on maintenance host |

**Steps:**
1. Console access via Proxmox web UI (if accessible)
2. Or connect monitor/keyboard to QOTOM directly
3. Boot OPNsense VM from Proxmox console

### QOTOM Down (Total Outage)

| Impact | Mitigation |
|--------|------------|
| Everything down | Physical access required |

**Steps:**
1. Physical access to QOTOM
2. Check power, monitor via HDMI
3. If hardware failure: boot from USB recovery
4. Restore from PBS backup to replacement hardware

## Database Recovery

### PostgreSQL Failover

Primary (on compute node) fails:

```bash
# On NAS, promote replica to primary
su - postgres
pg_ctl promote -D /var/lib/postgresql/data

# Update service connection strings to point to NAS IP
# Authentik, Wiki.js, etc.
```

### Authentik Database Restore

```bash
# From PBS backup
pg_restore -h localhost -U authentik -d authentik /backup/authentik.dump

# Or from replica
pg_dump -h nas-ip -U authentik authentik | psql -h new-primary -U authentik authentik
```

## Boot Priority (QOTOM)

Set in Proxmox to ensure correct startup order:

| Order | VM/CT | Delay | Purpose |
|-------|-------|-------|--------|
| 1 | OPNsense | 60s | Network first |
| 2 | PostgreSQL | 30s | Database ready |
| 3 | Authentik | 30s | Identity ready |
| 4 | Other services | 10s | Everything else |

```bash
# Set boot order
qm set 100 --startup order=1,up=60   # OPNsense
pct set 200 --startup order=2,up=30   # PostgreSQL
pct set 201 --startup order=3,up=30   # Authentik
```

## Checklist: Before Deployment

- [ ] Emergency SSH key generated on iZombie
- [ ] Key deployed to all Proxmox hosts
- [ ] Key deployed to OPNsense
- [ ] Key deployed to Unraid
- [ ] Backup key on encrypted USB
- [ ] USB stored securely
- [ ] SSH config file created
- [ ] Tested: SSH access works without LDAP
- [ ] Boot order configured on QOTOM
- [ ] PostgreSQL replication configured
- [ ] PBS backup schedule configured

## Related

- [[homelab/network/security]]
- [[homelab/identity/authentik]]
- [[homelab/services/pbs]]