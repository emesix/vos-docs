---
title: LDAP Linux Integration
description: SSSD configuration for Linux containers and VMs to authenticate against Authentik LDAP
published: true
date: 2025-12-28T18:50:02.149Z
tags: authentik, ldap, authentication, linux, sssd, pam, nss, nfs
editor: markdown
dateCreated: 2025-12-28T18:49:59.290Z
---

---
title: "LDAP Linux Integration"
uid: "identity-ldap-linux"
kind: [howto, identity, linux]
category: "03"
sub-category: "05"
visibility: 3.0
status: 3.0
tags: [ldap, linux, sssd, pam, nss, authentication, authentik, nfs]
parent_uid: "svc-authentik"
---

# LDAP Linux Integration

**Category:** 03.05 - Identity Services  
**Purpose:** Linux authentication against Authentik LDAP  
**Updated:** 2025-12-28

## Overview

Linux containers and VMs use SSSD to authenticate users and resolve UIDs/GIDs from Authentik LDAP. Enables centralized user management, SSH access control, and NFS permission mapping.

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    LINUX LDAP AUTH                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Linux Container/VM                                        │
│  ┌────────────────────────────────────────────┐            │
│  │  Application (runs as svc-radarr)         │            │
│  └──────────────────────┬─────────────────────┘            │
│                         │                                   │
│                         ▼                                   │
│  ┌────────────────────────────────────────────┐            │
│  │  NSS (Name Service Switch)                │            │
│  │  "Who is UID 10001?" → svc-radarr         │            │
│  └──────────────────────┬─────────────────────┘            │
│                         │                                   │
│                         ▼                                   │
│  ┌────────────────────────────────────────────┐            │
│  │  SSSD                                     │            │
│  │  Caches LDAP lookups                      │            │
│  └──────────────────────┬─────────────────────┘            │
│                         │                                   │
│                         ▼                                   │
│  ┌────────────────────────────────────────────┐            │
│  │  Authentik LDAP Outpost (<LAN_IP>) │            │
│  └────────────────────────────────────────────┘            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Use Cases

| Use Case | How LDAP Helps |
|----------|---------------|
| SSH login | Human users authenticate via LDAP |
| sudo access | Groups determine sudo permissions |
| NFS permissions | UID/GID mapped from LDAP |
| Service identity | Containers run as LDAP service accounts |
| Audit | All access tied to named identity |

## SSSD Configuration

### Installation

```bash
# Debian/Ubuntu
apt install sssd sssd-ldap sssd-tools libnss-sss libpam-sss

# Alpine (LXC containers)
apk add sssd sssd-ldap
```

### /etc/sssd/sssd.conf

```ini
[sssd]
domains = vos
config_file_version = 2
services = nss, pam, ssh

[domain/vos]
id_provider = ldap
auth_provider = ldap
chpass_provider = ldap
access_provider = ldap

# Authentik LDAP Outpost
ldap_uri = ldap://<LAN_IP>
ldap_search_base = dc=vos,dc=home
ldap_default_bind_dn = cn=ldap-service,ou=users,dc=vos,dc=home
ldap_default_authtok_type = password
ldap_default_authtok = ${LDAP_BIND_PASSWORD}

# User/Group search bases
ldap_user_search_base = ou=users,dc=vos,dc=home
ldap_group_search_base = ou=groups,dc=vos,dc=home

# Schema mapping (Authentik uses RFC2307bis)
ldap_schema = rfc2307bis
ldap_user_object_class = user
ldap_user_name = cn
ldap_user_uid_number = uidNumber
ldap_user_gid_number = gidNumber
ldap_user_home_directory = homeDirectory
ldap_user_shell = loginShell

ldap_group_object_class = group
ldap_group_name = cn
ldap_group_gid_number = gidNumber
ldap_group_member = member

# Access control
ldap_access_filter = (memberOf=cn=linux-users,ou=groups,dc=vos,dc=home)

# Caching
cache_credentials = true
entry_cache_timeout = 300

# Enumeration (disable for large directories)
enumerate = false
```

### File Permissions

```bash
chmod 600 /etc/sssd/sssd.conf
chown root:root /etc/sssd/sssd.conf
```

### /etc/nsswitch.conf

```
passwd:     files sss
group:      files sss
shadow:     files sss

hosts:      files dns
networks:   files

protocols:  files
services:   files
ethers:     files
rpc:        files

netgroup:   sss
automount:  sss
```

### /etc/pam.d/common-auth (Debian)

```
auth    [success=2 default=ignore]  pam_sss.so forward_pass
auth    [success=1 default=ignore]  pam_unix.so nullok try_first_pass
auth    requisite                   pam_deny.so
auth    required                    pam_permit.so
```

## Service Account Containers

### Running Container as LDAP User

For LXC containers, set the user in container config:

```bash
# Proxmox LXC config
# /etc/pve/lxc/210.conf (Radarr container)

lxc.init.uid = 10001
lxc.init.gid = 10001
```

For Docker:

```yaml
# docker-compose.yml
services:
  radarr:
    image: linuxserver/radarr
    user: "10001:10001"  # svc-radarr UID:GID
    environment:
      - PUID=10001
      - PGID=10001
```

### Verify User Resolution

```bash
# Test LDAP user lookup
getent passwd svc-radarr
# Output: svc-radarr:*:10001:10001:Radarr Service:/var/lib/radarr:/usr/sbin/nologin

# Test group lookup
getent group arr-writers
# Output: arr-writers:*:20001:svc-radarr,svc-sonarr,svc-lidarr

# Test with id command
id svc-radarr
# Output: uid=10001(svc-radarr) gid=10001(svc-radarr) groups=20001(arr-writers),20002(arr-downloaders)
```

## SSH Access Control

### Allow Only LDAP Groups

```bash
# /etc/ssh/sshd_config

# Allow specific groups
AllowGroups admins privileged-users

# Use SSSD for authorized keys (optional)
AuthorizedKeysCommand /usr/bin/sss_ssh_authorizedkeys
AuthorizedKeysCommandUser nobody
```

### sudo Configuration

```bash
# /etc/sudoers.d/ldap-groups

# Admins get full sudo
%admins ALL=(ALL:ALL) ALL

# Privileged users get limited sudo
%privileged-users ALL=(ALL:ALL) /usr/bin/systemctl, /usr/bin/journalctl
```

## NFS Permission Mapping

### How It Works

```
┌─────────────────────────────────────────────────────────────┐
│                   NFS + LDAP                                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  NFS Client (Radarr container)                              │
│  Running as UID 10001 (svc-radarr)                          │
│         │                                                   │
│         │ Write to /mnt/media/movies/                      │
│         ▼                                                   │
│  NFS Server (alexandria)                                    │
│  Sees request from UID 10001                                │
│         │                                                   │
│         │ Check permissions                                │
│         ▼                                                   │
│  /mnt/media/movies/ owned by:                               │
│  - Owner: root (UID 0)                                      │
│  - Group: arr-writers (GID 20001)                           │
│  - Mode: 775                                                │
│         │                                                   │
│         ▼                                                   │
│  SSSD on NFS server resolves:                               │
│  UID 10001 = svc-radarr                                     │
│  svc-radarr is member of GID 20001 (arr-writers)            │
│         │                                                   │
│         ▼                                                   │
│  ACCESS GRANTED (group write permission)                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### NFS Server Setup (Unraid/alexandria)

Unraid needs SSSD or direct LDAP support. Alternative: use fixed UID/GID.

```bash
# /etc/exports on NFS server
/mnt/media     <LAN_IP>/24(rw,sync,no_subtree_check,all_squash,anonuid=10001,anongid=20001)
/mnt/downloads <LAN_IP>/24(rw,sync,no_subtree_check,all_squash,anonuid=10010,anongid=20002)
```

Or with SSSD on NFS server (proper identity mapping):

```bash
# /etc/exports
/mnt/media     <LAN_IP>/24(rw,sync,no_subtree_check,no_root_squash)
/mnt/downloads <LAN_IP>/24(rw,sync,no_subtree_check,no_root_squash)

# Folder permissions
chown -R root:arr-writers /mnt/user/media
chmod -R 775 /mnt/user/media

chown -R root:arr-downloaders /mnt/user/downloads
chmod -R 775 /mnt/user/downloads
```

## Proxmox Integration

Proxmox can sync users from LDAP for Web UI access.

```bash
# Add LDAP realm
pveum realm add authentik --type ldap \
  --base_dn "ou=users,dc=vos,dc=home" \
  --user_attr "cn" \
  --server1 "<LAN_IP>" \
  --port 389 \
  --bind_dn "cn=ldap-service,ou=users,dc=vos,dc=home" \
  --password "${LDAP_BIND_PASSWORD}" \
  --sync_defaults_options "scope=sub,enable-new=1"

# Sync users
pveum realm sync authentik

# Map LDAP group to Proxmox role
pveum acl modify / --roles Administrator --groups admins@authentik
```

## Troubleshooting

### Test LDAP Connection

```bash
# Test bind
ldapsearch -x -H ldap://<LAN_IP> \
  -D "cn=ldap-service,ou=users,dc=vos,dc=home" \
  -w "${LDAP_BIND_PASSWORD}" \
  -b "dc=vos,dc=home" "(cn=svc-radarr)"
```

### Clear SSSD Cache

```bash
systemctl stop sssd
rm -rf /var/lib/sss/db/*
systemctl start sssd
```

### Debug Mode

```bash
# /etc/sssd/sssd.conf
[sssd]
debug_level = 9

# Then check logs
journalctl -u sssd -f
```

### Common Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| User not found | SSSD not running | `systemctl start sssd` |
| Auth fails | Wrong bind password | Check sssd.conf |
| Groups missing | Wrong search base | Verify ldap_group_search_base |
| UID mismatch | Different LDAP servers | Ensure same Authentik outpost |

## Related

- [[homelab/identity/authentik]]
- [[homelab/identity/users-groups]]
- [[homelab/identity/nfs-permissions]]
- [[homelab/network/security]]