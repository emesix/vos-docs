# VOS Foundation Implementation Guide

**Phase 1: OPNsense + Traefik - The Network Bones**

---

```
Document:     VOS Foundation Implementation Guide
Version:      1.0
Created:      2025-12-28
Scope:        OPNsense + Traefik ONLY
Audience:     user and Claude - both dummies
```

---

## What This Guide Covers

✅ OPNsense installation and base config
✅ All 17 VLANs created and configured
✅ DHCP per VLAN
✅ Firewall rules (frontend/backend/management)
✅ Internal DNS (Unbound)
✅ Traefik reverse proxy
✅ Let's Encrypt wildcard certificate

❌ NOT covered: Authentik, databases, ARR, AI, WiFi, VPN

---

# Part 1: OPNsense on hal

## 1.1 Hardware Setup

| Item | Value |
|------|-------|
| Host | hal (Qotom C3758R) |
| CPU | Intel Atom C3758R 8-core |
| RAM | 64GB |
| NICs | 4x SFP+ (multi-port) |
| Storage | 2x NVMe (ZFS mirror recommended) |

### NIC Assignment Plan

| Port | Interface | Role |
|------|-----------|------|
| SFP+ Port 1 | igb0 | WAN (Internet) |
| SFP+ Port 2 | igb1 | LAN Trunk (to Zyxel) |
| SFP+ Port 3 | igb2 | Backend Trunk (to ONTI 10GbE) |
| SFP+ Port 4 | igb3 | Spare / HA |

---

## 1.2 OPNsense Installation

### Download
```
https://opnsense.org/download/
Architecture: amd64
Image type: dvd (for USB install)
```

### Installation Steps

1. Boot from USB
2. Login: `installer` / `opnsense`
3. Select "Install (ZFS)" if using mirror, otherwise "Install (UFS)"
4. Select target disk(s)
5. Set root password
6. Reboot, remove USB

### Initial Console Setup

After reboot, at console:

```
1) Assign interfaces

WAN  -> igb0
LAN  -> igb1

(Skip optional interfaces for now - we'll add VLANs via WebUI)
```

```
2) Set interface IP address

LAN:
  IPv4: <LAN_IP>/24
  DHCP: No
  
(This is your management IP)
```

### First WebUI Access

From a computer connected to LAN port:

1. Set static IP: `<LAN_IP>/24`
2. Gateway: `<LAN_IP>`
3. Browse to: `https://<LAN_IP>`
4. Login: `root` / (your password)
5. Complete wizard

---

## 1.3 VLAN Creation

### Navigate to: Interfaces → Other Types → VLAN

Create each VLAN on the **LAN parent interface** (igb1):

| VLAN Tag | Description | Parent |
|----------|-------------|--------|
| 100 | Unprivileged LAN | igb1 |
| 101 | Privileged LAN | igb1 |
| 110 | Guest WiFi | igb1 |
| 111 | Home WiFi | igb1 |
| 112 | IoT WiFi | igb1 |
| 120 | Work | igb1 |
| 130 | Basement | igb1 |
| 131 | Garage | igb1 |
| 140 | VvE | igb1 |
| 150 | Reverse Proxy | igb1 |
| 200 | AI | igb1 |
| 210 | ARR | igb1 |
| 220 | Database | igb1 |
| 230 | Storage | igb1 |
| 240 | Docker | igb1 |

**Note:** VLAN 254 is the native/untagged LAN - no VLAN interface needed.

---

## 1.4 Interface Assignment

### Navigate to: Interfaces → Assignments

For each VLAN created, click **+** to add as interface.

Then configure each:

### VLAN 100 - Unprivileged
```
Interfaces → [VLAN100]
  Enable: ✓
  Description: VLAN100_Unprivileged
  IPv4 Type: Static IPv4
  IPv4 Address: <LAN_IP>/24
```

### VLAN 101 - Privileged
```
  Description: VLAN101_Privileged
  IPv4 Address: <LAN_IP>/24
```

### VLAN 110 - Guest WiFi
```
  Description: VLAN110_Guest
  IPv4 Address: <LAN_IP>/24
```

### VLAN 111 - Home WiFi
```
  Description: VLAN111_Home
  IPv4 Address: <LAN_IP>/24
```

### VLAN 112 - IoT
```
  Description: VLAN112_IoT
  IPv4 Address: <LAN_IP>/24
```

### VLAN 120 - Work
```
  Description: VLAN120_Work
  IPv4 Address: <LAN_IP>/24
```

### VLAN 130 - Basement
```
  Description: VLAN130_Basement
  IPv4 Address: <LAN_IP>/24
```

### VLAN 131 - Garage
```
  Description: VLAN131_Garage
  IPv4 Address: <LAN_IP>/24
```

### VLAN 140 - VvE
```
  Description: VLAN140_VvE
  IPv4 Address: <LAN_IP>/24
```

### VLAN 150 - Reverse Proxy
```
  Description: VLAN150_Proxy
  IPv4 Address: <LAN_IP>/24
```

### VLAN 200 - AI
```
  Description: VLAN200_AI
  IPv4 Address: <LAN_IP>/24
```

### VLAN 210 - ARR
```
  Description: VLAN210_ARR
  IPv4 Address: <LAN_IP>/24
```

### VLAN 220 - Database
```
  Description: VLAN220_Database
  IPv4 Address: <LAN_IP>/24
```

### VLAN 230 - Storage
```
  Description: VLAN230_Storage
  IPv4 Address: <LAN_IP>/24
```

### VLAN 240 - Docker
```
  Description: VLAN240_Docker
  IPv4 Address: <LAN_IP>/24
```

### LAN (Native VLAN 254)
```
  Description: LAN_Management
  IPv4 Address: <LAN_IP>/24
  (Keep <LAN_IP> as secondary for WebUI)
```

**After all interfaces configured:** Apply Changes

---

## 1.5 DHCP Server

### Navigate to: Services → DHCPv4

Configure for each VLAN:

### VLAN 100 - Unprivileged
```
Enable: ✓
Range: <LAN_IP> - <LAN_IP>
DNS: <LAN_IP>
Gateway: <LAN_IP>
```

### VLAN 101 - Privileged
```
Enable: ✓
Range: <LAN_IP> - <LAN_IP>
DNS: <LAN_IP>
Gateway: <LAN_IP>
```

### VLAN 110 - Guest
```
Enable: ✓
Range: <LAN_IP> - <LAN_IP>
DNS: <LAN_IP>
Gateway: <LAN_IP>
Lease time: 3600 (1 hour - guests are temporary)
```

### VLAN 111 - Home
```
Enable: ✓
Range: <LAN_IP> - <LAN_IP>
DNS: <LAN_IP>
Gateway: <LAN_IP>
```

### VLAN 112 - IoT
```
Enable: ✓
Range: <LAN_IP> - <LAN_IP>
DNS: <LAN_IP>
Gateway: <LAN_IP>
```

### VLAN 150 - Proxy
```
Enable: ✓
Range: <LAN_IP> - <LAN_IP>
DNS: <LAN_IP>
Gateway: <LAN_IP>
```

### Backend VLANs (200, 210, 220, 230, 240)
```
Enable: ✓
Range: 192.168.X.100 - 192.168.X.250
DNS: 192.168.X.1
Gateway: 192.168.X.1
```

### VLAN 254 - Management
```
Enable: ✗ (Static only - no DHCP)
```

---

## 1.6 Firewall Rules

### Navigate to: Firewall → Rules

**IMPORTANT:** Rules are processed top-to-bottom, first match wins.

---

### LAN (VLAN 254 - Management)

| # | Action | Source | Dest | Port | Description |
|---|--------|--------|------|------|-------------|
| 1 | Allow | LAN net | Any | Any | Management full access |

---

### VLAN 100 - Unprivileged (Quarantine)

| # | Action | Source | Dest | Port | Description |
|---|--------|--------|------|------|-------------|
| 1 | Allow | VLAN100 net | VLAN100 address | 53 | DNS to gateway |
| 2 | Block | VLAN100 net | RFC1918 | Any | Block internal |
| 3 | Allow | VLAN100 net | Any | 22,80,443 | Limited internet |
| 4 | Block | VLAN100 net | Any | Any | Default deny |

---

### VLAN 101 - Privileged

| # | Action | Source | Dest | Port | Description |
|---|--------|--------|------|------|-------------|
| 1 | Allow | VLAN101 net | VLAN101 address | 53 | DNS |
| 2 | Allow | VLAN101 net | VLAN150 net | 80,443 | Access proxy |
| 3 | Allow | VLAN101 net | VLAN230 net | 445 | SMB to storage |
| 4 | Block | VLAN101 net | RFC1918 | Any | Block other internal |
| 5 | Allow | VLAN101 net | Any | Any | Internet access |

---

### VLAN 110 - Guest

| # | Action | Source | Dest | Port | Description |
|---|--------|--------|------|------|-------------|
| 1 | Allow | VLAN110 net | VLAN110 address | 53 | DNS only |
| 2 | Block | VLAN110 net | RFC1918 | Any | No internal |
| 3 | Allow | VLAN110 net | Any | 80,443 | Web only |
| 4 | Block | VLAN110 net | Any | Any | Default deny |

---

### VLAN 111 - Home

| # | Action | Source | Dest | Port | Description |
|---|--------|--------|------|------|-------------|
| 1 | Allow | VLAN111 net | VLAN111 address | 53 | DNS |
| 2 | Allow | VLAN111 net | VLAN150 net | 80,443 | Access proxy |
| 3 | Block | VLAN111 net | RFC1918 | Any | Block other internal |
| 4 | Allow | VLAN111 net | Any | Any | Internet access |

---

### VLAN 112 - IoT (NO INTERNET)

| # | Action | Source | Dest | Port | Description |
|---|--------|--------|------|------|-------------|
| 1 | Allow | VLAN112 net | VLAN112 address | 53 | DNS |
| 2 | Allow | VLAN112 net | VLAN112 net | Any | Local discovery |
| 3 | Block | VLAN112 net | Any | Any | No internet |

---

### VLAN 150 - Reverse Proxy

| # | Action | Source | Dest | Port | Description |
|---|--------|--------|------|------|-------------|
| 1 | Allow | VLAN150 net | VLAN200 net | Any | Access AI |
| 2 | Allow | VLAN150 net | VLAN210 net | Any | Access ARR |
| 3 | Allow | VLAN150 net | VLAN220 net | Any | Access Database |
| 4 | Allow | VLAN150 net | VLAN240 net | Any | Access Docker |
| 5 | Allow | VLAN150 net | Any | 80,443 | Let's Encrypt |
| 6 | Block | VLAN150 net | Any | Any | Default deny |

---

### Backend VLANs (200, 210, 220, 240) - NO INTERNET

| # | Action | Source | Dest | Port | Description |
|---|--------|--------|------|------|-------------|
| 1 | Allow | VLANxxx net | VLANxxx address | 53 | DNS to gateway |
| 2 | Allow | VLANxxx net | VLAN230 net | 2049,111 | NFS to storage |
| 3 | Allow | VLANxxx net | VLAN220 net | 5432,6379 | Database access |
| 4 | Allow | VLANxxx net | VLANxxx net | Any | Intra-VLAN |
| 5 | Block | VLANxxx net | Any | Any | NO INTERNET |

---

### VLAN 230 - Storage (NO INTERNET)

| # | Action | Source | Dest | Port | Description |
|---|--------|--------|------|------|-------------|
| 1 | Allow | VLAN230 net | VLAN230 address | 53 | DNS |
| 2 | Allow | VLAN230 net | VLAN230 net | Any | Intra-VLAN |
| 3 | Block | VLAN230 net | Any | Any | NO INTERNET |

---

### Floating Rules (Apply to multiple interfaces)

Create alias first:

**Firewall → Aliases:**
```
Name: RFC1918
Type: Network(s)
Content:
  <LAN_IP>/8
  <LAN_IP>/12
  <LAN_IP>/16
```

```
Name: Backend_VLANs
Type: Network(s)
Content:
  <LAN_IP>/24
  <LAN_IP>/24
  <LAN_IP>/24
  <LAN_IP>/24
  <LAN_IP>/24
```

---

## 1.7 DNS (Unbound)

### Navigate to: Services → Unbound DNS → General

```
Enable: ✓
Listen Port: 53
Network Interfaces: All
DNSSEC: ✓
DNS Query Forwarding: (optional - to upstream like 1.1.1.1)
```

### Host Overrides (Services → Unbound DNS → Overrides)

**Wildcard for services (all go to Traefik):**

| Host | Domain | IP |
|------|--------|-----|
| * | local.user.nl | <LAN_IP> |

**Specific infrastructure hosts:**

| Host | Domain | IP |
|------|--------|-----|
| opnsense | local.user.nl | <LAN_IP> |
| hal | local.user.nl | <LAN_IP> |
| deep-thought | local.user.nl | <LAN_IP> |
| napster | local.user.nl | <LAN_IP> |
| mothership | local.user.nl | <LAN_IP> |
| skynet | local.user.nl | <LAN_IP> |
| watson | local.user.nl | <LAN_IP> |
| alexandria | local.user.nl | <LAN_IP> |

---

## 1.8 Verification Checklist

```bash
# From management network (VLAN 254)
ping <LAN_IP>      # Gateway
ping <LAN_IP>      # VLAN 100 gateway
ping <LAN_IP>      # VLAN 200 gateway

# DNS test
nslookup wiki.local.user.nl <LAN_IP>
# Should return: <LAN_IP>

# VLAN isolation test (from VLAN 100 client)
ping <LAN_IP>      # Should FAIL (blocked)
ping 8.8.8.8            # Should work (limited internet)
curl https://google.com # Should work

# Backend isolation test (from VLAN 200 client)
ping 8.8.8.8            # Should FAIL (no internet)
ping <LAN_IP>      # Should work (storage access)
```

---

# Part 2: Traefik on mothership

## 2.1 Prerequisites

- mothership (5800X) running with Proxmox
- LXC or VM for Traefik
- IP: <LAN_IP> (VLAN 150)
- Docker installed

---

## 2.2 Directory Structure

```bash
mkdir -p /opt/traefik/{config,letsencrypt}
cd /opt/traefik
```

---

## 2.3 Docker Compose

### /opt/traefik/docker-compose.yml

```yaml
version: "3.8"

services:
  traefik:
    image: traefik:v3.0
    container_name: traefik
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    ports:
      - "80:80"
      - "443:443"
    environment:
      - CF_API_EMAIL=${CF_API_EMAIL}
      - CF_DNS_API_TOKEN=${CF_DNS_API_TOKEN}
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./config/traefik.yml:/traefik.yml:ro
      - ./config/dynamic:/dynamic:ro
      - ./letsencrypt:/letsencrypt
    networks:
      - proxy
    labels:
      - "traefik.enable=true"
      # Dashboard (optional, remove in production)
      - "traefik.http.routers.traefik.rule=Host(`traefik.local.user.nl`)"
      - "traefik.http.routers.traefik.entrypoints=websecure"
      - "traefik.http.routers.traefik.tls.certresolver=cloudflare"
      - "traefik.http.routers.traefik.service=api@internal"
      # Basic auth for dashboard (generate with: htpasswd -nb admin password)
      - "traefik.http.routers.traefik.middlewares=traefik-auth"
      - "traefik.http.middlewares.traefik-auth.basicauth.users=admin:$$apr1$$xxxxx"

networks:
  proxy:
    name: proxy
    driver: bridge
```

### /opt/traefik/.env

```bash
CF_API_EMAIL=user@user.nl
CF_DNS_API_TOKEN=your-cloudflare-api-token-here
```

**IMPORTANT:** Protect this file!
```bash
chmod 600 /opt/traefik/.env
```

---

## 2.4 Traefik Static Configuration

### /opt/traefik/config/traefik.yml

```yaml
# API and Dashboard
api:
  dashboard: true
  insecure: false

# Logging
log:
  level: INFO
  # level: DEBUG  # Uncomment for troubleshooting

# Access logs (optional)
accessLog:
  filePath: "/letsencrypt/access.log"
  bufferingSize: 100

# Entrypoints
entryPoints:
  web:
    address: ":80"
    http:
      redirections:
        entryPoint:
          to: websecure
          scheme: https
  websecure:
    address: ":443"
    http:
      tls:
        certResolver: cloudflare
        domains:
          - main: "local.user.nl"
            sans:
              - "*.local.user.nl"

# Certificate Resolvers
certificatesResolvers:
  cloudflare:
    acme:
      email: user@user.nl
      storage: /letsencrypt/acme.json
      # Use staging for testing (uncomment next line)
      # caServer: https://acme-staging-v02.api.letsencrypt.org/directory
      dnsChallenge:
        provider: cloudflare
        resolvers:
          - "1.1.1.1:53"
          - "8.8.8.8:53"
        delayBeforeCheck: 10

# Providers
providers:
  docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false
    network: proxy
  file:
    directory: /dynamic
    watch: true
```

---

## 2.5 Dynamic Configuration (File Provider)

For services NOT running in Docker on mothership, use file-based routing.

### /opt/traefik/config/dynamic/middlewares.yml

```yaml
http:
  middlewares:
    # Redirect HTTP to HTTPS
    https-redirect:
      redirectScheme:
        scheme: https
        permanent: true
    
    # Security headers
    secure-headers:
      headers:
        frameDeny: true
        sslRedirect: true
        browserXssFilter: true
        contentTypeNosniff: true
        forceSTSHeader: true
        stsIncludeSubdomains: true
        stsPreload: true
        stsSeconds: 31536000
    
    # Rate limiting
    rate-limit:
      rateLimit:
        average: 100
        burst: 50
```

### /opt/traefik/config/dynamic/services.yml

```yaml
http:
  routers:
    # Wiki.js (on deep-thought)
    wiki:
      rule: "Host(`wiki.local.user.nl`)"
      entryPoints:
        - websecure
      service: wiki
      tls:
        certResolver: cloudflare
    
    # Proxmox (on hal - example)
    proxmox-hal:
      rule: "Host(`pve-hal.local.user.nl`)"
      entryPoints:
        - websecure
      service: proxmox-hal
      tls:
        certResolver: cloudflare

  services:
    wiki:
      loadBalancer:
        servers:
          - url: "http://<LAN_IP>:3000"
    
    proxmox-hal:
      loadBalancer:
        servers:
          - url: "https://<LAN_IP>:8006"
        serversTransport: insecure

  serversTransports:
    insecure:
      insecureSkipVerify: true
```

---

## 2.6 Cloudflare API Token

### Create Token at: https://dash.cloudflare.com/profile/api-tokens

```
Token name: VOS Traefik DNS
Permissions:
  - Zone → DNS → Edit
  - Zone → Zone → Read
Zone Resources:
  - Include → Specific zone → user.nl
```

Copy token to `/opt/traefik/.env`

---

## 2.7 Start Traefik

```bash
cd /opt/traefik

# First run - test with staging certs
# Edit traefik.yml, uncomment caServer staging line

docker compose up -d
docker logs -f traefik

# Watch for certificate acquisition
# Should see: "Certificate obtained successfully"

# If staging works, comment out caServer line
# Then remove acme.json and restart:
rm letsencrypt/acme.json
docker compose restart traefik
```

---

## 2.8 Verification

### Check Traefik Dashboard

```
https://traefik.local.user.nl
(requires basic auth configured in docker-compose)
```

### Test Certificate

```bash
# From any internal host
curl -v https://wiki.local.user.nl 2>&1 | grep -A5 "Server certificate"

# Should show:
# * Server certificate:
# *  subject: CN=*.local.user.nl
# *  issuer: C=US; O=Let's Encrypt; CN=R3
```

### Test Routing

```bash
# Should reach Wiki.js
curl -I https://wiki.local.user.nl

# Should get 200 OK or redirect to login
```

---

## 2.9 Adding New Services

### For Docker containers on mothership:

Add labels to the container:

```yaml
services:
  myapp:
    image: myapp:latest
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.myapp.rule=Host(`myapp.local.user.nl`)"
      - "traefik.http.routers.myapp.entrypoints=websecure"
      - "traefik.http.routers.myapp.tls.certresolver=cloudflare"
      - "traefik.http.services.myapp.loadbalancer.server.port=8080"
    networks:
      - proxy

networks:
  proxy:
    external: true
```

### For services on other hosts:

Add to `/opt/traefik/config/dynamic/services.yml`:

```yaml
http:
  routers:
    newservice:
      rule: "Host(`newservice.local.user.nl`)"
      entryPoints:
        - websecure
      service: newservice
      tls:
        certResolver: cloudflare

  services:
    newservice:
      loadBalancer:
        servers:
          - url: "http://192.168.xxx.xxx:port"
```

Traefik will auto-reload (file watcher enabled).

---

# Quick Reference

## Key IPs

| Service | IP | Port |
|---------|-----|------|
| OPNsense WebUI | <LAN_IP> | 443 |
| OPNsense Gateway | <LAN_IP> | - |
| Traefik | <LAN_IP> | 80, 443 |

## VLAN Summary

| VLAN | Subnet | Gateway | Internet |
|------|--------|---------|----------|
| 100 | <LAN_IP>/24 | .1 | Limited |
| 101 | <LAN_IP>/24 | .1 | Yes |
| 110 | <LAN_IP>/24 | .1 | Web only |
| 111 | <LAN_IP>/24 | .1 | Yes |
| 112 | <LAN_IP>/24 | .1 | **NO** |
| 150 | <LAN_IP>/24 | .1 | ACME only |
| 200-240 | 192.168.X.0/24 | .1 | **NO** |
| 254 | <LAN_IP>/24 | .1 | Admin |

## DNS Wildcard

All `*.local.user.nl` → <LAN_IP> (Traefik)

## Certificate

- Domain: `*.local.user.nl`
- Issuer: Let's Encrypt
- Resolver: Cloudflare DNS challenge
- Auto-renewal: Yes

---

# Troubleshooting

## OPNsense

### Can't reach WebUI
```bash
# Connect directly to LAN port
# Set static IP in same subnet
ip addr add <LAN_IP>/24 dev eth0
ping <LAN_IP>
```

### VLAN not working
1. Check switch trunk port configuration
2. Verify VLAN tag matches on both ends
3. Check interface is enabled in OPNsense

### DNS not resolving
```bash
# Test Unbound directly
dig @<LAN_IP> wiki.local.user.nl
# Check Services → Unbound DNS → General → Status
```

## Traefik

### Certificate not issued
```bash
docker logs traefik | grep -i acme
# Common issues:
# - Wrong API token permissions
# - DNS propagation delay (increase delayBeforeCheck)
# - Rate limited (wait or use staging)
```

### 404 on service
```bash
# Check router is active
curl http://localhost:8080/api/http/routers
# Check service is reachable from Traefik container
docker exec traefik wget -O- http://<LAN_IP>:3000
```

### 502 Bad Gateway
- Service not running
- Wrong port in service config
- Firewall blocking VLAN 150 → target VLAN

---

**END OF FOUNDATION GUIDE**

*Complete this before moving to Phase 2 (Identity) or any other services.*
