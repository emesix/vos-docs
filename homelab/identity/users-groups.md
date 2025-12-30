---
title: Users and Groups Design
description: LDAP user and group structure for VOS identity management - service accounts, human users, device accounts
published: true
date: 2025-12-28T18:46:18.495Z
tags: authentik, identity, ldap, users, groups, service-accounts, rbac
editor: markdown
dateCreated: 2025-12-28T18:46:15.914Z
---

---
title: "Users and Groups Design"
uid: "identity-users-groups"
kind: [reference, identity, security]
category: "03"
sub-category: "05"
visibility: 3.0
status: 3.0
tags: [identity, ldap, users, groups, service-accounts, rbac, authentik]
parent_uid: "svc-authentik"
---

# Users and Groups Design

**Category:** 03.05 - Identity Services  
**Purpose:** User and group structure for Authentik LDAP  
**Updated:** 2025-12-28

## Overview

Three account types: human users, service accounts, device accounts. Group membership determines access to resources. UID/GID ranges prevent conflicts.

## Account Types

| Type | UID Range | Purpose | MFA |
|------|-----------|---------|-----|
| Human | 1000-1999 | Interactive login | Required |
| Service | 10000-19999 | Container/app identity | None |
| Device | 20000-29999 | 802.1X/MAC auth | None |

## Human Users

```yaml
users:
  - username: user
    uid: 1000
    email: user@vos.home
    groups:
      - admins
      - privileged-users
      - arr-users
      - ai-users
    mfa: required
    shell: /bin/bash
    
  - username: guest
    uid: 1001
    email: guest@vos.home
    groups:
      - guests
    mfa: optional
    expiry: 24h  # Temporary accounts
```

## Service Accounts

### ARR Stack (UID 10001-10019)

| Username | UID | Groups | Purpose |
|----------|-----|--------|--------|
| svc-radarr | 10001 | arr-writers, arr-downloaders | Movie management |
| svc-sonarr | 10002 | arr-writers, arr-downloaders | TV management |
| svc-lidarr | 10003 | arr-writers, arr-downloaders | Music management |
| svc-prowlarr | 10004 | arr-indexers | Indexer proxy |
| svc-sabnzbd | 10010 | arr-downloaders | Usenet client |
| svc-qbittorrent | 10011 | arr-downloaders | Torrent client |
| svc-jellyfin | 10015 | media-readers | Media server |

### AI Stack (UID 10100-10119)

| Username | UID | Groups | Purpose |
|----------|-----|--------|--------|
| svc-openwebui | 10100 | ai-readers | Chat interface |
| svc-litellm | 10101 | ai-readers | LLM proxy |
| svc-vllm | 10102 | ai-readers, ai-writers | Inference engine |
| svc-haystack | 10103 | ai-readers, ai-writers | RAG pipeline |
| svc-searxng | 10104 | ai-readers | Search aggregator |

### Infrastructure (UID 10200-10219)

| Username | UID | Groups | Purpose |
|----------|-----|--------|--------|
| svc-postgres | 10200 | db-admins | Database server |
| svc-redis | 10201 | db-admins | Cache server |
| svc-wiki | 10202 | wiki-editors | Wiki.js |
| svc-gitea | 10203 | git-users | Git server |
| svc-traefik | 10210 | proxy-admins | Reverse proxy |
| svc-authentik | 10211 | identity-admins | Identity server |

## Device Accounts

For 802.1X wired/wireless authentication.

```yaml
devices:
  - username: device-imac-izombie
    uid: 20001
    mac: "AA:BB:CC:DD:EE:FF"
    groups: [privileged-devices]
    vlan: 101
    
  - username: device-laptop-work
    uid: 20002
    mac: "11:22:33:44:55:66"
    groups: [work-devices]
    vlan: 120
    
  - username: device-tv-h96
    uid: 20003
    mac: "AA:11:BB:22:CC:33"
    groups: [media-devices]
    vlan: 112
```

## Group Structure

### Access Control Groups

| Group | GID | RADIUS VLAN | Purpose |
|-------|-----|-------------|--------|
| admins | 1000 | 101 | Full system access |
| privileged-users | 1001 | 101 | LAN + services |
| guests | 1002 | 110 | Internet only |
| work-devices | 1003 | 120 | Isolated work |
| iot-devices | 1004 | 112 | Local only |

### Service Permission Groups

| Group | GID | NFS Access | Purpose |
|-------|-----|------------|--------|
| arr-writers | 20001 | /mnt/media:rw | Write media files |
| arr-downloaders | 20002 | /mnt/downloads:rw | Write downloads |
| arr-indexers | 20003 | None | Query indexers |
| media-readers | 20010 | /mnt/media:ro | Read media |
| ai-readers | 20100 | /mnt/ai:ro | Read AI models |
| ai-writers | 20101 | /mnt/ai:rw | Write embeddings |
| db-admins | 20200 | /mnt/backups:rw | Database backup |
| backup-writers | 20201 | /mnt/backups:rw | System backups |

## Group Membership Matrix

```
                    arr-  arr-   media- ai-   ai-
Service Account     write downl  read   read  write
──────────────────  ────  ─────  ─────  ────  ─────
svc-radarr           ✓     ✓      -      -     -
svc-sonarr           ✓     ✓      -      -     -
svc-lidarr           ✓     ✓      -      -     -
svc-sabnzbd          -     ✓      -      -     -
svc-qbittorrent      -     ✓      -      -     -
svc-jellyfin         -     -      ✓      -     -
svc-openwebui        -     -      -      ✓     -
svc-vllm             -     -      -      ✓     ✓
svc-haystack         -     -      -      ✓     ✓

Human Users
──────────────────
user (admin)      ✓     ✓      ✓      ✓     ✓
guest                -     -      ✓      -     -
```

## LDAP Structure

```
dc=vos,dc=home
├── ou=users
│   ├── ou=humans
│   │   ├── uid=user
│   │   └── uid=guest
│   ├── ou=services
│   │   ├── uid=svc-radarr
│   │   ├── uid=svc-sonarr
│   │   └── ...
│   └── ou=devices
│       ├── uid=device-imac-izombie
│       └── ...
└── ou=groups
    ├── cn=admins
    ├── cn=arr-writers
    ├── cn=arr-downloaders
    ├── cn=media-readers
    ├── cn=ai-readers
    └── ...
```

## Authentik Configuration

### LDAP Outpost Settings

```yaml
ldap_outpost:
  base_dn: "dc=vos,dc=home"
  bind_dn: "cn=ldap-service,ou=services,dc=vos,dc=home"
  search_base: "ou=users,dc=vos,dc=home"
  group_base: "ou=groups,dc=vos,dc=home"
  
  uid_start: 1000      # Humans
  gid_start: 1000      # Groups
  
  # Map Authentik attributes to LDAP
  attribute_mapping:
    uid: username
    uidNumber: ak_uid
    gidNumber: ak_gid
    homeDirectory: "/home/{username}"
    loginShell: "/bin/bash"
```

### Service Account Template

```yaml
# Create via Authentik Admin UI or API
service_account:
  username: "svc-{service}"
  type: "service_account"
  attributes:
    uid: {auto-assigned from range}
    gid: {auto-assigned}
    home: "/var/lib/{service}"
    shell: "/usr/sbin/nologin"
  groups: ["{service-specific groups}"]
  tokens:
    - name: "container-token"
      expiry: "never"  # Service accounts use tokens, not passwords
```

## Security Policies

### Password Policy (Humans)

- Minimum 12 characters
- Complexity: upper, lower, number, special
- No password reuse (last 10)
- Expiry: 90 days
- MFA: Required for all human accounts

### Service Account Policy

- No interactive login (`/usr/sbin/nologin`)
- Token-based authentication
- Tokens never expire (rotated manually)
- Bound to specific container IP (optional)

### Device Account Policy

- MAC address bound
- Certificate optional (802.1X EAP-TLS)
- Auto-expire after 365 days of inactivity

## Related

- [[homelab/identity/authentik]]
- [[homelab/identity/nfs-permissions]]
- [[homelab/identity/ldap-linux]]
- [[homelab/network/security]]