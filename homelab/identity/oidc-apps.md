---
title: OIDC SSO Integration
description: Single Sign-On configuration for web applications using Authentik OIDC provider
published: true
date: 2025-12-28T18:48:38.544Z
tags: authentik, oidc, sso, authentication, oauth, traefik, forward-auth
editor: markdown
dateCreated: 2025-12-28T18:48:35.740Z
---

---
title: "OIDC SSO Integration"
uid: "identity-oidc-apps"
kind: [howto, identity, integration]
category: "03"
sub-category: "05"
visibility: 3.0
status: 3.0
tags: [oidc, sso, oauth, authentication, authentik, traefik, forward-auth]
parent_uid: "svc-authentik"
---

# OIDC SSO Integration

**Category:** 03.05 - Identity Services  
**Purpose:** Single Sign-On for all web applications  
**Updated:** 2025-12-28

## Overview

Authentik provides OIDC/OAuth2 for native app integration and Forward Auth for legacy apps. One login for all services.

## Integration Methods

| Method | Apps | How It Works |
|--------|------|-------------|
| OIDC Native | Wiki.js, Gitea, Grafana, Proxmox | App redirects to Authentik |
| Forward Auth | Radarr, Sonarr, legacy apps | Traefik checks auth first |
| LDAP Bind | Some enterprise apps | Direct LDAP authentication |

## SSO Flow

```
┌─────────────────────────────────────────────────────────────┐
│                     OIDC NATIVE FLOW                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. User visits wiki.vos.home                               │
│                │                                            │
│                ▼                                            │
│  2. Wiki.js redirects to Authentik                         │
│     https://auth.vos.home/application/o/authorize/...      │
│                │                                            │
│                ▼                                            │
│  3. User logs in + MFA                                     │
│                │                                            │
│                ▼                                            │
│  4. Authentik redirects back with code                     │
│     https://wiki.vos.home/callback?code=xxx                │
│                │                                            │
│                ▼                                            │
│  5. Wiki.js exchanges code for token                       │
│                │                                            │
│                ▼                                            │
│  6. User logged in with Authentik identity                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────────────────┐
│                    FORWARD AUTH FLOW                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. User visits radarr.vos.home                            │
│                │                                            │
│                ▼                                            │
│  2. Traefik intercepts, calls Authentik                    │
│     GET /outpost.goauthentik.io/auth/nginx                 │
│                │                                            │
│             ┌───┴───┐                                       │
│             │       │                                       │
│        Not auth'd  Auth'd                                  │
│             │       │                                       │
│             ▼       ▼                                       │
│         Redirect  Pass headers                             │
│         to login  X-authentik-username: user            │
│                   X-authentik-groups: admins               │
│                        │                                    │
│                        ▼                                    │
│                   Radarr receives request                   │
│                   with auth headers                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Application Configurations

### Wiki.js (OIDC Native)

```yaml
# Wiki.js Admin > Authentication > Add Strategy

strategy: oidc
display_name: "Login with VOS"
client_id: "wikijs"
client_secret: "${WIKIJS_OIDC_SECRET}"
authorization_url: "https://auth.vos.home/application/o/authorize/"
token_url: "https://auth.vos.home/application/o/token/"
userinfo_url: "https://auth.vos.home/application/o/userinfo/"
issuer: "https://auth.vos.home/application/o/wikijs/"
logout_url: "https://auth.vos.home/application/o/wikijs/end-session/"
scope: "openid profile email groups"
map_groups: true
allow_self_registration: true
```

Authentik Application:

```yaml
application:
  name: "Wiki.js"
  slug: "wikijs"
  provider: "wikijs-oidc"
  launch_url: "https://wiki.vos.home"
  
provider:
  name: "wikijs-oidc"
  type: "oauth2"
  client_type: "confidential"
  client_id: "wikijs"
  redirect_uris:
    - "https://wiki.vos.home/login/oidc/callback"
  signing_key: "authentik-self-signed"
  scopes:
    - "openid"
    - "profile"
    - "email"
    - "groups"
```

### Proxmox (OIDC Native)

```bash
# On Proxmox host
pveum realm add authentik --type openid \
  --issuer-url "https://auth.vos.home/application/o/proxmox/" \
  --client-id "proxmox" \
  --client-key "${PROXMOX_OIDC_SECRET}" \
  --username-claim "preferred_username" \
  --scopes "openid profile email" \
  --autocreate 1
```

### Grafana (OIDC Native)

```ini
# /etc/grafana/grafana.ini

[auth.generic_oauth]
enabled = true
name = VOS SSO
allow_sign_up = true
client_id = grafana
client_secret = ${GRAFANA_OIDC_SECRET}
scopes = openid profile email groups
auth_url = https://auth.vos.home/application/o/authorize/
token_url = https://auth.vos.home/application/o/token/
api_url = https://auth.vos.home/application/o/userinfo/
role_attribute_path = contains(groups[*], 'admins') && 'Admin' || 'Viewer'
```

### Open WebUI (OIDC Native)

```yaml
# docker-compose.yml environment
environment:
  - ENABLE_OAUTH_SIGNUP=true
  - OAUTH_CLIENT_ID=openwebui
  - OAUTH_CLIENT_SECRET=${OPENWEBUI_OIDC_SECRET}
  - OPENID_PROVIDER_URL=https://auth.vos.home/application/o/openwebui/.well-known/openid-configuration
  - OAUTH_SCOPES=openid profile email groups
```

### Jellyfin (OIDC Plugin)

```yaml
# Jellyfin > Plugins > SSO Authentication

OID Provider Name: VOS SSO
OID Endpoint: https://auth.vos.home/application/o/jellyfin/
OpenID Client ID: jellyfin
OpenID Client Secret: ${JELLYFIN_OIDC_SECRET}
Enabled Folders: [] # All libraries
Roles: [] # Map from Authentik groups
Admin Roles: ["admins"]
Enable Authorization by Plugin: true
```

### Gitea (OIDC Native)

```ini
# /etc/gitea/app.ini

[oauth2_client]
ENABLE_AUTO_REGISTRATION = true

# Then via Gitea Admin UI:
# Authentication Sources > Add > OAuth2
Authentication Name: VOS SSO
OAuth2 Provider: OpenID Connect
Client ID: gitea
Client Secret: ${GITEA_OIDC_SECRET}
OpenID Connect Auto Discovery URL: https://auth.vos.home/application/o/gitea/.well-known/openid-configuration
Group Claim Name: groups
Admin Group: admins
```

## Forward Auth (Legacy Apps)

### Traefik Configuration

```yaml
# /etc/traefik/dynamic/authentik.yml

http:
  middlewares:
    authentik:
      forwardAuth:
        address: "http://<LAN_IP>:9000/outpost.goauthentik.io/auth/traefik"
        trustForwardHeader: true
        authResponseHeaders:
          - X-authentik-username
          - X-authentik-groups
          - X-authentik-email
          - X-authentik-name
          - X-authentik-uid
```

### Radarr/Sonarr/Lidarr (Forward Auth)

```yaml
# Traefik router for Radarr

http:
  routers:
    radarr:
      rule: "Host(`radarr.vos.home`)"
      service: radarr
      middlewares:
        - authentik  # Forward auth middleware
      tls:
        certResolver: letsencrypt

  services:
    radarr:
      loadBalancer:
        servers:
          - url: "http://<LAN_IP>:7878"
```

Authentik Forward Auth Application:

```yaml
application:
  name: "Radarr"
  slug: "radarr"
  provider: "radarr-proxy"
  launch_url: "https://radarr.vos.home"

provider:
  name: "radarr-proxy"
  type: "proxy"
  mode: "forward_auth"  # or "forward_single" for single domain
  external_host: "https://radarr.vos.home"
  
  # Optional: restrict to specific groups
  authorization_flow: "default-provider-authorization-explicit-consent"
  access_control:
    require_groups:
      - "arr-users"
      - "admins"
```

## Group-Based Access Control

### Application Access Matrix

| Application | admins | arr-users | ai-users | guests |
|-------------|--------|-----------|----------|--------|
| Wiki.js | ✓ | ✓ | ✓ | ✓ (read) |
| Radarr | ✓ | ✓ | ✗ | ✗ |
| Sonarr | ✓ | ✓ | ✗ | ✗ |
| Jellyfin | ✓ | ✓ | ✗ | ✓ |
| Open WebUI | ✓ | ✗ | ✓ | ✗ |
| Proxmox | ✓ | ✗ | ✗ | ✗ |
| Grafana | ✓ | ✗ | ✗ | ✗ |
| Gitea | ✓ | ✓ | ✓ | ✗ |

### Authentik Policy Example

```python
# Policy: require-arr-users
# Bind to Radarr, Sonarr, Lidarr applications

return ak_is_group_member(request.user, name="arr-users") or \
       ak_is_group_member(request.user, name="admins")
```

## Session Management

### Settings

| Setting | Value | Purpose |
|---------|-------|--------|
| Session Duration | 24 hours | How long until re-auth |
| Remember Me | 30 days | Extended session |
| Idle Timeout | 1 hour | Inactive logout |
| MFA Grace Period | 0 | Always require MFA |

### Single Logout

When user logs out of one app, logs out everywhere:

```yaml
# Authentik settings
oidc_provider:
  backchannel_logout_url: "https://wiki.vos.home/logout"
  frontchannel_logout_url: "https://wiki.vos.home/logout"
```

## Troubleshooting

### Debug OIDC

```bash
# Check Authentik logs
docker logs authentik-server 2>&1 | grep -i oidc

# Test OIDC discovery
curl https://auth.vos.home/application/o/wikijs/.well-known/openid-configuration | jq

# Decode JWT token
echo "eyJhbGc..." | cut -d. -f2 | base64 -d | jq
```

### Common Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| Redirect loop | Wrong callback URL | Check redirect_uris |
| Invalid client | Wrong secret | Regenerate in Authentik |
| Groups missing | Scope not requested | Add "groups" to scopes |
| 403 after login | Policy denied | Check group membership |

## Related

- [[homelab/identity/authentik]]
- [[homelab/identity/users-groups]]
- [[homelab/services/traefik]]
- [[homelab/network/security]]