---
title: DNS and Certificate Strategy
description: Split-horizon DNS with Let's Encrypt wildcard certificates via Cloudflare DNS challenge
published: true
date: 2025-12-28T19:15:03.687Z
tags: opnsense, traefik, dns, certificates, ssl, cloudflare, letsencrypt
editor: markdown
dateCreated: 2025-12-28T19:15:00.745Z
---

---
title: "DNS and Certificate Strategy"
uid: "infra-network-dns-certs"
kind: [infrastructure, network, security]
category: "01"
sub-category: "02"
visibility: 3.0
status: 3.0
tags: [dns, certificates, ssl, cloudflare, letsencrypt, traefik, opnsense]
parent_uid: ""
---

# DNS and Certificate Strategy

**Category:** 01.02 - Network Infrastructure  
**Purpose:** Internal DNS resolution with trusted SSL certificates  
**Updated:** 2025-12-28

## Overview

Split-horizon DNS using `local.user.nl` subdomain for internal services. Wildcard SSL certificate from Let's Encrypt via Cloudflare DNS challenge. No self-signed certificate warnings.

## Domain Strategy

| Domain | Zone | Purpose |
|--------|------|--------|
| `user.nl` | Cloudflare (public) | Public DNS, cert validation |
| `local.user.nl` | OPNsense (internal) | All homelab services |
| `example.com` | Cloudflare (public) | Reserved for future use |

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    SPLIT-HORIZON DNS                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   EXTERNAL QUERY                 INTERNAL QUERY             │
│   (from internet)                (from LAN)                 │
│         │                              │                    │
│         ▼                              ▼                    │
│   ┌───────────────┐            ┌───────────────┐           │
│   │  Cloudflare   │            │   OPNsense    │           │
│   │  DNS          │            │   Unbound     │           │
│   └───────┬───────┘            └───────┬───────┘           │
│           │                            │                    │
│           ▼                            ▼                    │
│   local.user.nl                local.user.nl           │
│   → NXDOMAIN (not public)        → <LAN_IP>          │
│                                  (Traefik reverse proxy)    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Internal DNS Records

Managed in OPNsense Unbound as local zone overrides.

### Service Records

| Hostname | IP | Service |
|----------|-----|--------|
| `auth.local.user.nl` | <LAN_IP> | Authentik |
| `wiki.local.user.nl` | <LAN_IP> | Wiki.js |
| `proxmox.local.user.nl` | <LAN_IP> | Proxmox UI |
| `radarr.local.user.nl` | <LAN_IP> | Radarr |
| `sonarr.local.user.nl` | <LAN_IP> | Sonarr |
| `jellyfin.local.user.nl` | <LAN_IP> | Jellyfin |
| `grafana.local.user.nl` | <LAN_IP> | Grafana |
| `chat.local.user.nl` | <LAN_IP> | Open WebUI |

All point to Traefik (<LAN_IP>) which routes by hostname.

### Infrastructure Records

| Hostname | IP | Purpose |
|----------|-----|--------|
| `opnsense.local.user.nl` | <LAN_IP> | Firewall UI |
| `pve-qotom.local.user.nl` | <LAN_IP> | Proxmox node |
| `pve-5800x.local.user.nl` | <LAN_IP> | Proxmox node |
| `nas.local.user.nl` | <LAN_IP> | Unraid |

## Certificate Strategy

### Wildcard Certificate

Single certificate covers all `*.local.user.nl` hostnames.

| Property | Value |
|----------|-------|
| Domains | `*.local.user.nl`, `local.user.nl` |
| Issuer | Let's Encrypt |
| Challenge | DNS-01 via Cloudflare API |
| Validity | 90 days (auto-renewed) |
| Location | Traefik (VLAN 150) |

### How DNS Challenge Works

```
┌─────────────────────────────────────────────────────────────┐
│                 CERTIFICATE ISSUANCE                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. Traefik requests cert for *.local.user.nl             │
│                    │                                        │
│                    ▼                                        │
│  2. Let's Encrypt: "Prove you own user.nl"                │
│                    │                                        │
│                    ▼                                        │
│  3. Traefik creates DNS TXT record via Cloudflare API       │
│     _acme-challenge.local.user.nl = "random-token"        │
│                    │                                        │
│                    ▼                                        │
│  4. Let's Encrypt checks Cloudflare DNS                     │
│     "Token found, domain ownership verified"                │
│                    │                                        │
│                    ▼                                        │
│  5. Certificate issued, stored in Traefik                   │
│                    │                                        │
│                    ▼                                        │
│  6. All *.local.user.nl now have trusted HTTPS            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Key insight:** DNS challenge validates domain ownership. Doesn't matter that `local.user.nl` has no public IP. Cert is valid because you control the parent domain.

## Cloudflare Configuration

### API Token

Create scoped token for cert renewal:

```
Permissions:
  - Zone > DNS > Edit
  - Zone > Zone > Read

Zone Resources:
  - Include > Specific zone > user.nl
```

Store token securely for Traefik.

### DNS Records at Cloudflare

| Type | Name | Value | Proxy |
|------|------|-------|-------|
| A | `@` | (public IP if needed) | Yes/No |
| CNAME | `www` | `user.nl` | Yes |
| TXT | `_acme-challenge.local` | (auto-managed) | No |

No A record needed for `local.user.nl` - it's internal only.

## OPNsense Unbound Configuration

### Local Zone Override

```
Services > Unbound DNS > Overrides

Host Overrides:
┌────────────────┬─────────────────────┬─────────────────┐
│ Host           │ Domain              │ IP              │
├────────────────┼─────────────────────┼─────────────────┤
│ *              │ local.user.nl     │ <LAN_IP>  │
│ auth           │ local.user.nl     │ <LAN_IP>  │
│ wiki           │ local.user.nl     │ <LAN_IP>  │
│ opnsense       │ local.user.nl     │ <LAN_IP> │
│ nas            │ local.user.nl     │ <LAN_IP> │
└────────────────┴─────────────────────┴─────────────────┘
```

### DHCP DNS Settings

All VLANs use OPNsense as DNS server:

| VLAN | DNS Server |
|------|------------|
| All Frontend (100-150) | <LAN_IP> |
| All Backend (200-250) | <LAN_IP> |
| Management (254) | <LAN_IP> |

## Traefik Configuration

### Static Config (traefik.yml)

```yaml
entryPoints:
  web:
    address: ":80"
    http:
      redirections:
        entryPoint:
          to: websecure
  websecure:
    address: ":443"

certificatesResolvers:
  cloudflare:
    acme:
      email: "user@user.nl"
      storage: "/letsencrypt/acme.json"
      dnsChallenge:
        provider: cloudflare
        resolvers:
          - "1.1.1.1:53"
          - "8.8.8.8:53"
```

### Environment Variables

```bash
CF_API_EMAIL=user@user.nl
CF_DNS_API_TOKEN=your-scoped-api-token
```

### Dynamic Config (Wildcard Cert)

```yaml
tls:
  certificates:
    - certFile: /letsencrypt/certs/local.user.nl.crt
      keyFile: /letsencrypt/certs/local.user.nl.key
      stores:
        - default

  stores:
    default:
      defaultCertificate:
        certFile: /letsencrypt/certs/local.user.nl.crt
        keyFile: /letsencrypt/certs/local.user.nl.key
```

## Verification

### Test DNS Resolution

```bash
# From internal network
nslookup wiki.local.user.nl
# Should return: <LAN_IP>

# From external (should fail)
nslookup wiki.local.user.nl 8.8.8.8
# Should return: NXDOMAIN
```

### Test Certificate

```bash
curl -v https://wiki.local.user.nl 2>&1 | grep -A5 "Server certificate"
# Should show: issuer: Let's Encrypt
# No SSL errors
```

### Browser Test

Visit `https://wiki.local.user.nl` - green padlock, no warnings.

## Renewal

Traefik auto-renews certificates 30 days before expiry. No manual intervention needed.

Monitor via:
- Traefik dashboard
- Let's Encrypt expiry notifications to email

## Fallback: Cloudflare Tunnel

Alternative for external access without opening ports:

```
Browser → Cloudflare Edge → Tunnel → Traefik → Service
```

Cloudflare handles SSL termination at edge. Useful for remote access without VPN.

## Related

- [[homelab/network/architecture]]
- [[homelab/network/security]]
- [[homelab/services/traefik]]
- [[homelab/identity/authentik]]