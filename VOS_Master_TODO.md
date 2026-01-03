# VOS Master TODO List

**The Complete Implementation Checklist for user's Operating System**

---

```
Document:     VOS Master TODO List
Version:      1.0
Created:      2025-12-28
Purpose:      Single checklist for full VOS implementation
Status:       ACTIVE - Check off as you go
```

---

## Legend

- `[ ]` = Not started
- `[~]` = In progress
- `[x]` = Complete
- `[?]` = Needs decision
- `[!]` = Blocked

Priority: 🔴 Critical | 🟠 High | 🟡 Medium | 🟢 Nice-to-have

---

# Phase 0: Pre-Staging ✅ COMPLETE

- [x] Development environment on iZombie (Arch Linux)
- [x] Claude Desktop with MCP servers
- [x] Wiki.js documentation system
- [x] Planning documents created
- [x] Network Implementation Guide created

---

# Phase 1: Network Foundation 🟡 IN PROGRESS

## 1.1 OPNsense Base ✅ COMPLETE
- [x] Install OPNsense on hal (Qotom C3758R)
- [x] Configure WAN interface (igc0 → Ziggo)
- [x] Configure LAN interface (igc1 legacy, igc2 MGMT)
- [x] Set management IP: 192.168.254.254 (WebUI)
- [x] Set gateway IP: 192.168.254.1 (no WebUI)
- [x] Basic firewall rules (allow management access)
- [x] Enable SSH for emergency access

## 1.2 VLAN Creation ✅ COMPLETE
All 15 VLANs created on ix1 trunk via Python XML script (2025-01-03):
- [x] VLAN 254 (Management) - 192.168.254.0/24
- [x] VLAN 100 (Quarantine) - 192.168.100.0/24
- [x] VLAN 101 (Privileged) - 192.168.101.0/24
- [x] VLAN 110 (Guest WiFi) - 192.168.110.0/24
- [x] VLAN 111 (Home WiFi) - 192.168.111.0/24
- [x] VLAN 112 (IoT) - 192.168.112.0/24
- [x] VLAN 120 (Work) - 192.168.120.0/24
- [x] VLAN 130 (Basement) - 192.168.130.0/24
- [x] VLAN 131 (Garage) - 192.168.131.0/24
- [x] VLAN 140 (VvE) - 192.168.140.0/24
- [x] VLAN 150 (Proxy) - 192.168.150.0/24
- [x] VLAN 200 (AI) - 192.168.200.0/24
- [x] VLAN 210 (ARR) - 192.168.210.0/24
- [x] VLAN 220 (Database) - 192.168.220.0/24
- [x] VLAN 230 (Storage) - 192.168.230.0/24
- [x] VLAN 240 (Docker) - 192.168.240.0/24

## 1.3 Switch Configuration 🟡 IN PROGRESS
- [x] Zyxel GS1900-24HP: Management IP 192.168.254.2 (OpenWRT 24.10.1)
- [ ] Zyxel: Configure VLAN trunk ports
- [ ] Zyxel: Configure access ports per VLAN
- [x] ONTI S508CL-8S: Management IP 192.168.254.3 (OpenWRT SNAPSHOT r31056)
- [ ] ONTI: Configure VLAN trunk to OPNsense (lan4)
- [ ] ONTI: Configure backend access ports
- [x] BiDi fiber ONTI P8 ↔ Zyxel P26: 1Gbps UP (SFP-BX-U03D ↔ SFP-BX-D03D)
- [!] LACP/LAG: NOT POSSIBLE on OpenWRT RTL838x/RTL930x (no HW offload)
- [~] Tenda TEM2007: Connected to ONTI, config pending

## 1.4 Host Network Configuration
Proxmox hosts on VLAN 254 management:
- [x] hal: 192.168.254.100 (OPNsense VM host)
- [~] deep-thought: 192.168.254.101 (connected via Zyxel P17)
- [~] napster: 192.168.254.102 (connected via Zyxel P18)
- [ ] mothership: 192.168.254.103
- [ ] skynet: 192.168.254.104
- [ ] watson: 192.168.254.105 (offline)
- [ ] alexandria: 192.168.254.110

## 1.5 DHCP Configuration ✅ COMPLETE
- [x] Configure DHCP per VLAN (ranges .100-.250)
- [ ] Static reservations for known devices
- [x] DNS server = gateway (.1) per VLAN

## 1.6 Firewall Rules 🟡 PERMISSIVE (needs hardening)
Current: "Allow all" for testing. TODO: implement zone isolation.
- [ ] VLAN 100 rules (quarantine - limited internet)
- [ ] VLAN 101 rules (privileged - via proxy to backend)
- [ ] VLAN 150 rules (proxy - can reach backend services)
- [ ] Backend VLANs 200-250 (NO internet, inter-VLAN allowed)
- [x] Management VLAN 254 (full access)

## 1.7 Network Verification
- [x] Ping test: OPNsense ↔ ONTI ↔ Zyxel on VLAN 254
- [ ] VLAN isolation test (100 cannot reach 200)
- [ ] Backend internet block verified
- [ ] Inter-VLAN routing via OPNsense confirmed

---

# Phase 2: Identity Layer 🔴

## 2.1 Database Foundation (deep-thought)
- [ ] Deploy PostgreSQL LXC (<LAN_IP>)
- [ ] Deploy Redis LXC (<LAN_IP>)
- [ ] Create Authentik database
- [ ] Test database connectivity

## 2.2 Authentik Core (mothership)
- [ ] Deploy Authentik Server (<LAN_IP>)
- [ ] Deploy Authentik Worker (<LAN_IP>)
- [ ] Initial admin setup
- [ ] Configure PostgreSQL connection
- [ ] Configure Redis connection

## 2.3 LDAP Outpost
- [ ] Create LDAP provider in Authentik
- [ ] Deploy LDAP Outpost (<LAN_IP>)
- [ ] Configure LDAP schema (dc=vos,dc=home)
- [ ] Test LDAP bind from test client

## 2.4 RADIUS Outpost
- [ ] Create RADIUS provider in Authentik
- [ ] Deploy RADIUS Outpost (<LAN_IP>)
- [ ] Configure RADIUS clients (APs, switches)
- [ ] Create VLAN assignment policy
- [ ] Test RADIUS auth with radtest

## 2.5 Users and Groups
Access Control Groups:
- [ ] admins (GID 1000, VLAN 101)
- [ ] privileged-users (GID 1001, VLAN 101)
- [ ] guests (GID 1002, VLAN 110)
- [ ] home-devices (GID 1003, VLAN 111)
- [ ] iot-devices (GID 1004, VLAN 112)
- [ ] work-devices (GID 1005, VLAN 120)

Service Permission Groups:
- [ ] arr-writers (GID 20001)
- [ ] arr-downloaders (GID 20002)
- [ ] arr-indexers (GID 20003)
- [ ] media-readers (GID 20010)
- [ ] ai-readers (GID 20100)
- [ ] ai-writers (GID 20101)
- [ ] db-admins (GID 20200)
- [ ] backup-writers (GID 20201)

Human Users:
- [ ] user (UID 1000, all groups, MFA required)
- [ ] guest (UID 1001, guests only)

## 2.6 Service Accounts
ARR Stack:
- [ ] svc-radarr (UID 10001)
- [ ] svc-sonarr (UID 10002)
- [ ] svc-lidarr (UID 10003)
- [ ] svc-prowlarr (UID 10004)
- [ ] svc-sabnzbd (UID 10010)
- [ ] svc-qbittorrent (UID 10011)
- [ ] svc-jellyfin (UID 10015)

AI Stack:
- [ ] svc-openwebui (UID 10100)
- [ ] svc-litellm (UID 10101)
- [ ] svc-vllm (UID 10102)
- [ ] svc-haystack (UID 10103)
- [ ] svc-searxng (UID 10104)

Infrastructure:
- [ ] svc-postgres (UID 10200)
- [ ] svc-redis (UID 10201)
- [ ] svc-wiki (UID 10202)
- [ ] svc-gitea (UID 10203)
- [ ] svc-traefik (UID 10210)
- [ ] svc-authentik (UID 10211)

---

# Phase 3: DNS and Certificates 🔴

## 3.1 Internal DNS (OPNsense Unbound)
- [ ] Create local zone: local.user.nl
- [ ] Wildcard override: *.local.user.nl → <LAN_IP>
- [ ] Infrastructure overrides (opnsense, pve-*, nas)
- [ ] Test internal DNS resolution
- [ ] Verify external queries return NXDOMAIN

## 3.2 SSL Certificates
- [ ] Create Cloudflare API token (Zone DNS Edit)
- [ ] Store token securely
- [ ] Deploy Traefik on mothership (<LAN_IP>)
- [ ] Configure Let's Encrypt DNS challenge
- [ ] Request wildcard cert for *.local.user.nl
- [ ] Verify HTTPS works (green padlock)
- [ ] Test auto-renewal

---

# Phase 4: Core Services 🟠

## 4.1 Reverse Proxy (mothership)
- [ ] Configure Traefik entrypoints (80 → 443 redirect)
- [ ] Configure Authentik forward auth middleware
- [ ] Add route: auth.local.user.nl → Authentik
- [ ] Test SSO flow

## 4.2 Wiki.js (deep-thought)
- [ ] Deploy Wiki.js (<LAN_IP>)
- [ ] Configure PostgreSQL backend
- [ ] Configure OIDC auth with Authentik
- [ ] Add Traefik route: wiki.local.user.nl
- [ ] Test SSO login
- [ ] Configure Git sync (optional)

## 4.3 Gitea (deep-thought)
- [ ] Deploy Gitea (<LAN_IP>)
- [ ] Configure PostgreSQL backend
- [ ] Configure OIDC auth with Authentik
- [ ] Add Traefik route: git.local.user.nl
- [ ] Test SSO login
- [ ] Create infra repo for GitOps

## 4.4 Storage (alexandria)
- [ ] Configure NFS exports for backend VLANs
- [ ] Set LDAP group permissions (or all_squash)
- [ ] Test NFS mounts from backend hosts
- [ ] Configure SMB shares for frontend
- [ ] Test SMB access from privileged clients

---

# Phase 5: Observability (mothership) 🟠

*Gap identified by ChatGPT - essential for VOS "OS-like" behavior*

## 5.1 Metrics (Prometheus)
- [ ] Deploy Prometheus (<LAN_IP>)
- [ ] Configure node_exporter on all hosts
- [ ] Add OPNsense exporter/syslog metrics
- [ ] Add container metrics (cAdvisor or Docker metrics)
- [ ] Configure alerting rules
- [ ] Deploy Alertmanager
- [ ] Configure notification channel (email/Telegram)

## 5.2 Logging (Loki)
- [ ] Deploy Loki (<LAN_IP>)
- [ ] Deploy Promtail on key hosts
- [ ] Configure OPNsense syslog → Loki
- [ ] Configure Traefik access logs → Loki
- [ ] Configure Authentik logs → Loki

## 5.3 Dashboards (Grafana)
- [ ] Deploy Grafana (<LAN_IP>)
- [ ] Add Traefik route: grafana.local.user.nl
- [ ] Configure OIDC auth with Authentik
- [ ] Add Prometheus datasource
- [ ] Add Loki datasource
- [ ] Import dashboards (node, docker, OPNsense)
- [ ] Create VOS overview dashboard

---

# Phase 6: Backup and Recovery 🟠

*Gap identified by ChatGPT - essential for resilience*

## 6.1 Proxmox Backup Server (alexandria)
- [ ] Create PBS VM on alexandria
- [ ] Configure backup datastore on NAS storage
- [ ] Add PBS to Proxmox cluster
- [ ] Configure PBS web UI via Traefik (pbs.local.user.nl)

## 6.2 Backup Jobs
- [ ] VM/LXC backups (daily, 7-day retention)
- [ ] PostgreSQL dumps (daily, 30-day retention)
- [ ] Config snapshots (/etc, compose files)
- [ ] Wiki.js content backup
- [ ] Authentik blueprints export
- [ ] Secrets backup (encrypted)

## 6.3 Restore Testing
- [ ] Document restore procedures
- [ ] Test VM restore
- [ ] Test PostgreSQL restore
- [ ] Test config restore
- [ ] Schedule quarterly restore drills

---

# Phase 7: Secrets Management 🟠

*Vaultwarden for both human passwords AND machine secrets*

## 7.1 Vaultwarden Deployment (mothership)
- [ ] Deploy Vaultwarden (<LAN_IP>)
- [ ] Configure PostgreSQL backend
- [ ] Configure Traefik route (vault.local.user.nl)
- [ ] Configure OIDC auth with Authentik (optional)
- [ ] Set up admin account

## 7.2 Human Secrets
- [ ] Install browser extensions
- [ ] Import existing passwords
- [ ] Configure 2FA/TOTP

## 7.3 Machine Secrets (via Vaultwarden API)
- [ ] Create organization for "VOS Infrastructure"
- [ ] Create collections: cloudflare, database, radius, oidc
- [ ] Migrate secrets:
  - [ ] Cloudflare API token
  - [ ] OIDC client secrets
  - [ ] Database passwords
  - [ ] RADIUS shared secrets
- [ ] Document API access pattern for automation
- [ ] Backup encrypted vault

---

# Phase 8: Time and Updates 🟡

*Gaps identified by ChatGPT for air-gapped backend*

## 8.1 NTP for Backend
- [ ] Configure chrony on OPNsense or dedicated LXC
- [ ] Allow backend → NTP server only
- [ ] Block backend → public NTP
- [ ] Verify time sync on backend hosts

## 8.2 Update Proxy (mothership)
- [ ] Deploy apt-cacher-ng or similar
- [ ] Deploy container registry mirror (pull-through cache)
- [ ] Configure firewall: backend → update-proxy only
- [ ] Test container pulls from backend
- [ ] Test apt updates from backend

---

# Phase 9: Automation (mothership) 🟡

## 9.1 GitOps Repository
- [ ] Create vos-infra repo in Gitea
- [ ] Add VLAN definitions
- [ ] Add OPNsense config exports
- [ ] Add Traefik routes
- [ ] Add Authentik blueprints
- [ ] Add Docker compose stacks
- [ ] Add Ansible/scripts for provisioning

## 9.2 CI Runner
- [ ] Deploy Gitea Actions runner
- [ ] Configure runner for infra repo
- [ ] Create validation pipeline (lint, test)
- [ ] Create deployment pipeline (controlled)

---

# Phase 10: Application Services 🟡

## 10.1 ARR Stack (napster)
- [ ] Deploy Radarr (<LAN_IP>)
- [ ] Deploy Sonarr (<LAN_IP>)
- [ ] Deploy Lidarr (<LAN_IP>)
- [ ] Deploy Prowlarr (<LAN_IP>)
- [ ] Deploy qBittorrent (<LAN_IP>)
- [ ] Deploy SABnzbd (<LAN_IP>)
- [ ] Deploy Jellyfin (<LAN_IP>)
- [ ] Deploy Notifiarr (<LAN_IP>) - ARR notifications + Discord
- [ ] Configure NFS mounts (/media, /downloads)
- [ ] Configure forward auth via Traefik
- [ ] Add Traefik routes for each service
- [ ] Test full download → organize → stream flow

## 10.2 AI Stack (skynet/watson)
- [ ] Deploy Open WebUI (<LAN_IP>) - skynet
- [ ] Deploy LiteLLM (<LAN_IP>) - skynet
- [ ] Deploy vLLM (<LAN_IP>) - watson
- [ ] Deploy Haystack (<LAN_IP>) - skynet
- [ ] Deploy SearxNG (<LAN_IP>) - skynet
- [ ] Deploy MCP Gateway (<LAN_IP>) - mothership
- [ ] Configure NFS mounts (/ai)
- [ ] Configure OIDC auth
- [ ] Add Traefik routes
- [ ] Test inference flow

---

# Phase 11: WiFi and Remote Access 🟡

## 11.1 WiFi Configuration
- [ ] Configure RT-2980 #1-5 for WPA3-Enterprise
- [ ] Configure RADIUS client on each AP
- [ ] Test dynamic VLAN assignment
- [ ] Configure VOS-Guest (captive portal, VLAN 110)
- [ ] Configure VOS-IoT (WPA2-PSK, VLAN 112)
- [ ] Test roaming between APs

## 11.2 VPN (WireGuard)
- [ ] Configure WireGuard on OPNsense
- [ ] Integrate with Authentik (user portal)
- [ ] Generate user configs
- [ ] Test remote access to internal services

## 11.3 Remote Sites
- [ ] Configure Basement tunnel (VLAN 130)
- [ ] Configure Garage tunnel (VLAN 131)
- [ ] Configure CPE510 P2P link
- [ ] Test remote AP connectivity

---

# Phase 12: VOS Brain (skynet) 🟢

*The "more than sum of parts" layer*

## 12.1 VOS Registry
- [ ] Create VOS service registry (FastAPI or config file)
- [ ] Document all services:
  - Service name
  - URL (internal + proxy)
  - Auth method
  - Data classification
  - Host + VLAN + ports
  - Healthcheck endpoint

## 12.2 VOS Orchestrator
- [ ] Deploy orchestrator service
- [ ] Configure MCP client hub
- [ ] Connect to all tools/services
- [ ] Create policy engine (routing decisions)
- [ ] Log all actions to PostgreSQL

## 12.3 VOS Dashboard
- [ ] Create VOS overview panel
- [ ] Service health status
- [ ] Recent actions log
- [ ] Quick access to common tasks

---

# Phase 13: Document Pipeline 🟢

*Multi-agent document handling from ChatGPT analysis*

## 13.1 Pipeline Infrastructure
- [ ] Create drop zones on NFS:
  - /vos/inbox/public/
  - /vos/inbox/private/
  - /vos/inbox/ai-only/
  - /vos/inbox/quarantine/
- [ ] Create job state table in PostgreSQL
- [ ] Deploy Qdrant vector DB (<LAN_IP>) on skynet

## 13.2 Pipeline Agents
- [ ] Intake Agent (watches drop zones)
- [ ] Extractor Agent (PDF/DOCX/HTML → markdown)
- [ ] Classifier Agent (public/private/ai-only)
- [ ] Wiki Curator Agent (structure, dedup, links)
- [ ] RAG Indexer Agent (embeddings → Qdrant)
- [ ] QA Agent (broken links, contradictions)
- [ ] Publisher Agent (commits to Wiki.js)

## 13.3 Pipeline State Machine
States: NEW → EXTRACTED → CLASSIFIED → CURATED → INDEXED → PUBLISHED → VERIFIED → DONE
- [ ] Implement state transitions
- [ ] Implement retry logic
- [ ] Implement quarantine for failures

---

# Phase 14: Security Hardening 🟢

## 14.1 Internal PKI - Smallstep CA (mothership)
- [ ] Deploy Smallstep CA (<LAN_IP>)
- [ ] Configure root CA
- [ ] Create intermediate CA for services
- [ ] Issue certificates for backend services
- [ ] Configure auto-renewal (ACME or step-ca)
- [ ] Document cert request process
- [ ] Test: services work when internet is down

## 14.2 Vulnerability Scanning
- [ ] Deploy Trivy for container scanning
- [ ] Configure regular scans
- [ ] Alert on critical vulnerabilities

## 14.3 Audit and Compliance
- [ ] Configure "backend egress attempt" alerts
- [ ] Regular review of RADIUS VLAN assignments
- [ ] Regular review of firewall logs
- [ ] Document security policies

---

# Decisions ✅ FINALIZED

| ID | Decision | Final Answer | Location |
|----|----------|--------------|----------|
| D1 | Observability | **Full stack** (Prometheus+Loki+Grafana) + **Notifiarr** | mothership + napster |
| D2 | Secrets | **Vaultwarden** (humans + machines via API) | mothership VLAN 150 |
| D3 | PBS | **VM on alexandria** | alexandria VLAN 230 |
| D4 | PKI | **Let's Encrypt** (external) + **Smallstep CA** (internal) | mothership |
| D5 | Vector DB | **Qdrant** | skynet VLAN 200 |

---

# Quick Reference - Host Assignments

| Host | Role | Key Services |
|------|------|--------------|
| hal | Router | OPNsense, NTP, WireGuard |
| deep-thought | Data Plane | PostgreSQL, Redis, Wiki.js, Gitea |
| napster | ARR | Radarr, Sonarr, Lidarr, Prowlarr, Jellyfin, **Notifiarr** |
| mothership | Service Plane | Traefik, Authentik, Grafana/Loki/Prometheus, **Vaultwarden**, **Smallstep CA**, MCP Gateway |
| skynet | VOS Brain | Open WebUI, LiteLLM, Haystack, **Qdrant**, VOS Orchestrator |
| watson | AI Worker | vLLM (GPU inference) |
| alexandria | Storage | NFS, SMB, **PBS** |

---

# Progress Tracker

| Phase | Items | Complete | Progress |
|-------|-------|----------|----------|
| 0. Pre-Staging | 5 | 5 | ████████████████████ 100% |
| 1. Network | ~40 | ~25 | ████████████░░░░░░░░ 60% |
| 2. Identity | ~35 | 0 | ░░░░░░░░░░░░░░░░░░░░ 0% |
| 3. DNS/Certs | ~10 | 0 | ░░░░░░░░░░░░░░░░░░░░ 0% |
| 4. Core Services | ~20 | 0 | ░░░░░░░░░░░░░░░░░░░░ 0% |
| 5. Observability | ~15 | 0 | ░░░░░░░░░░░░░░░░░░░░ 0% |
| 6. Backup | ~15 | 0 | ░░░░░░░░░░░░░░░░░░░░ 0% |
| 7. Secrets | ~10 | 0 | ░░░░░░░░░░░░░░░░░░░░ 0% |
| 8. Time/Updates | ~8 | 0 | ░░░░░░░░░░░░░░░░░░░░ 0% |
| 9. Automation | ~10 | 0 | ░░░░░░░░░░░░░░░░░░░░ 0% |
| 10. Applications | ~25 | 0 | ░░░░░░░░░░░░░░░░░░░░ 0% |
| 11. WiFi/VPN | ~15 | 0 | ░░░░░░░░░░░░░░░░░░░░ 0% |
| 12. VOS Brain | ~10 | 0 | ░░░░░░░░░░░░░░░░░░░░ 0% |
| 13. Doc Pipeline | ~15 | 0 | ░░░░░░░░░░░░░░░░░░░░ 0% |
| 14. Hardening | ~10 | 0 | ░░░░░░░░░░░░░░░░░░░░ 0% |

**Total: ~243 items | ~30 complete**

---

**END OF TODO LIST**

*Update this document as you complete items. This is your implementation roadmap.*
