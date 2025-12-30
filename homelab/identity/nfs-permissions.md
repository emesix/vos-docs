---
title: NFS Permissions with LDAP
description: NFS folder structure and LDAP group permissions for access control
published: true
date: 2025-12-28T18:53:41.117Z
tags: unraid, storage, authentik, ldap, groups, nfs, permissions
editor: markdown
dateCreated: 2025-12-28T18:53:38.437Z
---

---
title: "NFS Permissions with LDAP"
uid: "identity-nfs-permissions"
kind: [reference, identity, storage]
category: "03"
sub-category: "05"
visibility: 3.0
status: 3.0
tags: [nfs, ldap, permissions, storage, authentik, unraid, groups]
parent_uid: "svc-authentik"
---

# NFS Permissions with LDAP

**Category:** 03.05 - Identity Services  
**Purpose:** NFS access control using LDAP groups  
**Updated:** 2025-12-28

## Overview

NFS exports on alexandria (Unraid) use Unix permissions with LDAP groups. Service accounts access folders based on group membership. Replaces IP-based trust with identity-based access.

## Security Improvement

| Before (IP Trust) | After (LDAP Groups) |
|-------------------|--------------------|
| Any process on IP has access | Only correct UID/GID has access |
| Hacked container = full NFS access | Hacked container = limited to its user |
| No audit trail | All access logged with identity |

## Folder Structure

```
/mnt/user/
├── downloads/              # arr-downloaders (GID 20002)
│   ├── incomplete/
│   ├── complete/
│   └── watch/
├── media/                  # arr-writers (GID 20001)
│   ├── movies/
│   ├── tv/
│   ├── music/
│   └── books/
├── ai/                     # ai-writers (GID 20101)
│   ├── models/
│   ├── embeddings/
│   └── cache/
└── backups/                # backup-writers (GID 20201)
    ├── databases/
    ├── configs/
    └── vms/
```

## Group Permissions

| Folder | Group | Mode | Access |
|--------|-------|------|--------|
| /downloads | arr-downloaders | 775 | SABnzbd, qBittorrent |
| /media | arr-writers | 775 | Radarr, Sonarr, Lidarr |
| /media | media-readers | ACL | Jellyfin (read-only) |
| /ai | ai-writers | 775 | vLLM, Haystack |
| /ai | ai-readers | ACL | Open WebUI (read-only) |
| /backups | backup-writers | 770 | Backup services |

## Service Account Matrix

| Service | UID | Groups | Access |
|---------|-----|--------|--------|
| svc-radarr | 10001 | arr-writers, arr-downloaders | /downloads RW, /media RW |
| svc-sonarr | 10002 | arr-writers, arr-downloaders | /downloads RW, /media RW |
| svc-sabnzbd | 10010 | arr-downloaders | /downloads RW |
| svc-jellyfin | 10015 | media-readers | /media RO |
| svc-vllm | 10102 | ai-readers, ai-writers | /ai RW |
| svc-openwebui | 10100 | ai-readers | /ai RO |

## NFS Exports

### Simple: All Squash

```bash
# /etc/exports
/mnt/user/downloads  <LAN_IP>/24(rw,all_squash,anonuid=10010,anongid=20002)
/mnt/user/media      <LAN_IP>/24(rw,all_squash,anonuid=10001,anongid=20001)
/mnt/user/ai         <LAN_IP>/24(rw,all_squash,anonuid=10102,anongid=20101)
```

### Proper: SSSD + ACLs

```bash
# /etc/exports (no squash)
/mnt/user/downloads  <LAN_IP>/24(rw,no_root_squash)
/mnt/user/media      <LAN_IP>/24(rw,no_root_squash)
/mnt/user/ai         <LAN_IP>/24(rw,no_root_squash)

# Folder permissions
chown root:arr-writers /mnt/user/media
chmod 775 /mnt/user/media
setfacl -R -m g:media-readers:rx /mnt/user/media
```

## Security Boundaries

| If Compromised | Can Access | Cannot Access |
|----------------|------------|---------------|
| Radarr | /media RW, /downloads RW | /ai, /backups |
| Jellyfin | /media RO | /downloads, /ai, /backups |
| vLLM | /ai RW | /media, /downloads |
| SABnzbd | /downloads RW | /media directly |

## Related

- [[homelab/identity/users-groups]]
- [[homelab/identity/ldap-linux]]
- [[homelab/hosts/alexandria]]