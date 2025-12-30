---
title: Network Topology Diagram
description: Mermaid diagrams showing complete VOS network topology, switch ports, WireGuard tunnels, and host interfaces
published: false
date: 2025-12-28T18:17:39.951Z
tags: mermaid, network, topology, diagram, switches, hosts, wifi, endpoints, vlan
editor: markdown
dateCreated: 2025-12-28T16:07:57.073Z
---

---
title: "Network Topology Diagram"
uid: "infra-network-topology"
kind: [infrastructure, network, diagram]
category: "01"
sub-category: "02"
visibility: 3.0
status: 4.0
tags: [network, topology, diagram, mermaid, switches, hosts, endpoints, wifi]
parent_uid: ""
---

# Network Topology Diagram

**Category:** 01.02 - Networking Infrastructure  
**Purpose:** Visual network topology for VOS homelab  
**Updated:** 2025-12-28

## Overview

Dual-zone architecture. Backend (200-250) has no internet, trusted. Frontend (100-150) has firewalled internet, untrusted. Management (254) for physical infrastructure.

## Security Zones

```mermaid
flowchart TB
    subgraph INTERNET["🌐 INTERNET"]
        WAN["WAN"]
    end

    subgraph OPNSENSE["🔥 OPNsense"]
        FW["Firewall & Router"]
    end

    subgraph FRONTEND["🟢 FRONTEND (100-150)"]
        direction TB
        F_DESC["Firewalled Internet
SMB Storage
Untrusted Zone"]
    end

    subgraph BACKEND["🔵 BACKEND (200-250)"]
        direction TB
        B_DESC["❌ NO INTERNET
NFS Storage
Trusted Zone"]
    end

    subgraph MGMT["🟡 MANAGEMENT (254)"]
        direction TB
        M_DESC["Admin Only
Infrastructure"]
    end

    WAN --> FW
    FW -->|"Firewalled"| FRONTEND
    FW -->|"BLOCKED"| BACKEND
    FW -->|"Admin"| MGMT

    FRONTEND -->|"Via Proxy"| BACKEND

    classDef internet fill:#ff6b6b,stroke:#cc5555,color:#fff
    classDef firewall fill:#ffa94d,stroke:#cc8844,color:#000
    classDef frontend fill:#69db7c,stroke:#55aa66,color:#000
    classDef backend fill:#4dabf7,stroke:#3388cc,color:#fff
    classDef mgmt fill:#ffd43b,stroke:#ccaa33,color:#000

    class WAN internet
    class FW firewall
    class FRONTEND,F_DESC frontend
    class BACKEND,B_DESC backend
    class MGMT,M_DESC mgmt
```

## Complete Physical Topology

```mermaid
flowchart TB
    subgraph INTERNET["🌐 INTERNET"]
        ZIGGO["Ziggo Modem<br/>Bridge Mode"]
    end

    subgraph EDGE["🔥 EDGE ROUTER"]
        OPNSENSE["hal<br/>Qotom Q20331G9<br/>OPNsense<br/>C3758R 8C | 64GB"]
    end

    subgraph SWITCHES["📡 SWITCHES"]
        subgraph FRONTEND_SW["Frontend 1GbE"]
            ZYXEL["Zyxel GS1900-24HP<br/>24× 1GbE PoE + 2× SFP<br/>170W Budget"]
        end
        subgraph BACKEND_SW["Backend 10GbE"]
            ONTI["ONTI S508CL-8S<br/>8× 10G SFP+"]
            TENDA["Tenda TEM2007<br/>7× 2.5G + 1× SFP+"]
        end
    end

    subgraph COMPUTE["💻 COMPUTE NODES"]
        BREIN["🧠 deep-thought<br/>HX310 J6426<br/>Core Services<br/>32GB"]
        DOWNLOAD["📥 napster<br/>HX310 J6426<br/>*ARR Stack<br/>32GB"]
        HOOFD["📖 skynet<br/>CWWK 8845HS<br/>AI Controller<br/>64GB"]
        KLUS["🔧 mothership<br/>B450M 5800X<br/>Docker Worker<br/>32GB"]
        DENK["🤖 watson<br/>X99 2×E5-2686v4<br/>AI GPU 2×A770<br/>128GB"]
    end

    subgraph STORAGE["🗄️ STORAGE"]
        NAS["alexandria<br/>Unraid NAS<br/>R5 3600 | 32GB<br/>8×16TB = 117TB"]
    end

    subgraph WIFI["📶 WIFI MESH"]
        subgraph INDOOR["Indoor APs"]
            AP1["RT-2980 #1<br/>Living Room"]
            AP2["RT-2980 #2<br/>Office"]
            AP3["RT-2980 #3<br/>Bedroom"]
        end
        subgraph REMOTE["Remote APs"]
            AP4["🔐 RT-2980 #4<br/>Basement<br/>WireGuard"]
            AP5["🔐 RT-2980 #5<br/>Garage<br/>WireGuard"]
        end
    end

    ZIGGO -->|WAN| OPNSENSE
    OPNSENSE -->|"LACP 2×1G<br/>P25-26"| ZYXEL
    OPNSENSE -->|"SFP+ 10G<br/>P1"| ONTI
    ONTI -->|"SFP+ 10G<br/>P2"| TENDA

    ZYXEL -->|"P1"| BREIN
    ZYXEL -->|"P2"| DOWNLOAD
    ZYXEL -->|"P3 PoE"| AP1
    ZYXEL -->|"P4 PoE"| AP2
    ZYXEL -->|"P5 PoE"| AP3

    ONTI -->|"P3 DAC"| KLUS
    ONTI -->|"P5 AOC"| DENK
    ONTI -->|"P6 AOC"| NAS

    TENDA -->|"P1"| BREIN
    TENDA -->|"P2"| DOWNLOAD
    TENDA -->|"P3"| HOOFD

    classDef internet fill:#ff6b6b,stroke:#cc5555,color:#fff
    classDef router fill:#ffa94d,stroke:#cc8844,color:#000
    classDef frontend fill:#69db7c,stroke:#55aa66,color:#000
    classDef backend fill:#4dabf7,stroke:#3388cc,color:#000
    classDef server fill:#748ffc,stroke:#5566cc,color:#fff
    classDef storage fill:#20c997,stroke:#188866,color:#fff
    classDef wifi fill:#da77f2,stroke:#aa55cc,color:#000
    classDef remote fill:#f783ac,stroke:#cc6688,color:#000

    class ZIGGO internet
    class OPNSENSE router
    class ZYXEL frontend
    class ONTI,TENDA backend
    class BREIN,DOWNLOAD,HOOFD,KLUS,DENK server
    class NAS storage
    class AP1,AP2,AP3 wifi
    class AP4,AP5 remote
```

## VLAN Zone Map

```mermaid
flowchart LR
    subgraph FRONTEND["🟢 FRONTEND VLANs (100-150)"]
        subgraph CLIENTS["Client Access"]
            V100["100: Unprivileged<br/>Unknown MAC"]
            V101["101: Privileged<br/>Known MAC + Auth"]
        end
        subgraph WIFI_V["WiFi"]
            V110["110: Guest<br/>Captive Portal"]
            V111["111: Home<br/>Trusted"]
            V112["112: IoT<br/>No Internet"]
        end
        subgraph OTHER_F["Other"]
            V120["120: Work<br/>Isolated"]
            V130["130: Basement<br/>WireGuard"]
            V131["131: Garage<br/>WireGuard"]
            V140["140: VvE<br/>Cameras"]
        end
        subgraph PROXY["Services"]
            V150["150: Reverse Proxy<br/>WebUI Bridge"]
        end
    end

    subgraph BACKEND["🔵 BACKEND VLANs (200-250)"]
        V200["200: AI<br/>vLLM, LiteLLM"]
        V210["210: ARR<br/>Radarr, Sonarr"]
        V220["220: Database<br/>PostgreSQL"]
        V230["230: Storage<br/>NFS Server"]
        V240["240: Docker<br/>Containers"]
    end

    subgraph MGMT["🟡 MANAGEMENT (254+)"]
        V254["254: Infrastructure<br/>Hosts, Switches, APs"]
    end

    V150 -->|"Proxy"| BACKEND

    classDef frontend fill:#69db7c,stroke:#55aa66,color:#000
    classDef backend fill:#4dabf7,stroke:#3388cc,color:#fff
    classDef mgmt fill:#ffd43b,stroke:#ccaa33,color:#000
    classDef proxy fill:#da77f2,stroke:#aa55cc,color:#000

    class V100,V101,V110,V111,V112,V120,V130,V131,V140 frontend
    class V150 proxy
    class V200,V210,V220,V230,V240 backend
    class V254 mgmt
```

## Management VLAN 254 Layout

```mermaid
flowchart TB
    subgraph VLAN254["VLAN 254 - Management (<LAN_IP>/24)"]
        subgraph GATEWAY["Gateway"]
            GW[".1 = Gateway<br/>NO WebUI!"]
            WEBUI[".254 = OPNsense WebUI"]
        end
        subgraph SWITCHES_M["Switches (.2-.10)"]
            SW2[".2 = Zyxel"]
            SW3[".3 = ONTI"]
            SW4[".4 = Tenda"]
        end
        subgraph APS["WiFi APs (.11-.20)"]
            AP11[".11 = RT-2980 #1"]
            AP12[".12 = RT-2980 #2"]
            AP13[".13 = RT-2980 #3"]
            AP14[".14 = RT-2980 #4"]
            AP15[".15 = RT-2980 #5"]
            AP16[".16 = CPE510 TX"]
            AP17[".17 = CPE510 RX"]
        end
        subgraph PVE_FE["Proxmox Frontend (.100-.109)"]
            PF100[".100 = hal"]
            PF101[".101 = deep-thought"]
            PF102[".102 = napster"]
            PF103[".103 = mothership"]
            PF104[".104 = skynet"]
            PF105[".105 = watson"]
        end
        subgraph NAS_FE["NAS Frontend (.110-.119)"]
            NF110[".110 = alexandria"]
        end
        subgraph PVE_BE["Proxmox Backend (.200-.209)"]
            PB200[".200 = hal"]
            PB201[".201 = deep-thought"]
            PB202[".202 = napster"]
            PB203[".203 = mothership"]
            PB204[".204 = skynet"]
            PB205[".205 = watson"]
        end
        subgraph NAS_BE["NAS Backend (.210-.219)"]
            NB210[".210 = alexandria"]
        end
    end

    classDef gw fill:#ffa94d,stroke:#cc8844
    classDef switch fill:#69db7c,stroke:#55aa66
    classDef ap fill:#da77f2,stroke:#aa55cc
    classDef pve_fe fill:#748ffc,stroke:#5566cc,color:#fff
    classDef pve_be fill:#4dabf7,stroke:#3388cc
    classDef nas fill:#20c997,stroke:#188866

    class GW,WEBUI gw
    class SW2,SW3,SW4 switch
    class AP11,AP12,AP13,AP14,AP15,AP16,AP17 ap
    class PF100,PF101,PF102,PF103,PF104,PF105 pve_fe
    class PB200,PB201,PB202,PB203,PB204,PB205 pve_be
    class NF110,NB210 nas
```

## Endpoint Devices

```mermaid
flowchart TB
    subgraph WIFI_MESH["📶 WiFi Mesh"]
        AP1["RT-2980 #1"]
        AP2["RT-2980 #2"]
        AP3["RT-2980 #3"]
    end

    subgraph ENDPOINTS["📱 ENDPOINTS"]
        subgraph DEV["Development (VLAN 101)"]
            IZOMBIE["🖥️ iZombie<br/>iMac 2014 Arch"]
            X300["🖥️ X300<br/>ASRock 5700G"]
        end
        subgraph MEDIA["Media (VLAN 112)"]
            H96["📺 H96 MAX V58<br/>RK3588"]
        end
        subgraph MOBILE["Mobile (VLAN 111)"]
            ARMOR["📱 Armor Pad 4"]
            A55_1["📱 Samsung A55-1"]
        end
        subgraph WORK["Work (VLAN 120)"]
            A55_2["📱 Samsung A55-2"]
            HP["💼 HP Laptop"]
        end
        subgraph IOT["IoT (VLAN 112)"]
            TUYA["🏠 Tuya Devices"]
        end
    end

    AP1 -.-> IZOMBIE
    AP1 -.-> H96
    AP2 -.-> X300
    AP2 -.-> HP
    AP1 -.-> ARMOR
    AP1 -.-> A55_1
    AP1 -.-> A55_2
    AP1 -.-> TUYA

    classDef ap fill:#da77f2,stroke:#aa55cc
    classDef dev fill:#4dabf7,stroke:#3388cc,color:#000
    classDef media fill:#ffa94d,stroke:#cc8844,color:#000
    classDef mobile fill:#69db7c,stroke:#55aa66,color:#000
    classDef work fill:#748ffc,stroke:#5566cc,color:#fff
    classDef iot fill:#ffd43b,stroke:#ccaa33,color:#000

    class AP1,AP2,AP3 ap
    class IZOMBIE,X300 dev
    class H96 media
    class ARMOR,A55_1 mobile
    class A55_2,HP work
    class TUYA iot
```

## Storage Access Model

```mermaid
flowchart LR
    subgraph NAS["alexandria (Unraid)"]
        subgraph BE_NIC["Backend NIC (.210)"]
            NFS["NFS Exports"]
        end
        subgraph FE_NIC["Frontend NIC (.110)"]
            SMB["SMB Shares"]
        end
    end

    subgraph BACKEND_C["Backend Consumers"]
        AI["VLAN 200 AI"]
        ARR["VLAN 210 ARR"]
        DB["VLAN 220 DB"]
    end

    subgraph FRONTEND_C["Frontend Consumers"]
        PRIV["VLAN 101 Privileged"]
    end

    NFS -->|"Linux Native"| AI
    NFS -->|"Linux Native"| ARR
    NFS -->|"Linux Native"| DB

    SMB -->|"Windows/Mac"| PRIV

    classDef nfs fill:#4dabf7,stroke:#3388cc
    classDef smb fill:#69db7c,stroke:#55aa66
    classDef backend fill:#748ffc,stroke:#5566cc,color:#fff
    classDef frontend fill:#69db7c,stroke:#55aa66

    class NFS,AI,ARR,DB nfs
    class SMB,PRIV smb
```

## Reverse Proxy Flow (VLAN 150)

```mermaid
flowchart LR
    subgraph CLIENT["Client (VLAN 101)"]
        USER["💻 User"]
    end

    subgraph PROXY["VLAN 150"]
        RP["Reverse Proxy<br/><LAN_IP>"]
    end

    subgraph BACKEND_SVC["Backend Services"]
        WIKI["Wiki.js<br/>VLAN 220"]
        WEBUI["Open WebUI<br/>VLAN 200"]
        JELLY["Jellyfin<br/>VLAN 210"]
    end

    USER -->|"wiki.vos.local"| RP
    USER -->|"ai.vos.local"| RP
    USER -->|"media.vos.local"| RP

    RP -->|"Proxy"| WIKI
    RP -->|"Proxy"| WEBUI
    RP -->|"Proxy"| JELLY

    USER -.->|"BLOCKED"| BACKEND_SVC

    classDef client fill:#69db7c,stroke:#55aa66
    classDef proxy fill:#da77f2,stroke:#aa55cc
    classDef backend fill:#4dabf7,stroke:#3388cc

    class USER client
    class RP proxy
    class WIKI,WEBUI,JELLY backend
```

## Switch Port Assignments

### Zyxel GS1900-24HP (Frontend)

| Port | Device | VLAN | Notes |
|------|--------|------|-------|
| P1 | deep-thought | 254 | Frontend NIC |
| P2 | napster | 254 | Frontend NIC |
| P3 | RT-2980 #1 | Trunk | PoE, Living Room |
| P4 | RT-2980 #2 | Trunk | PoE, Office |
| P5 | RT-2980 #3 | Trunk | PoE, Bedroom |
| P6 | Hisource | - | PoE, Balcony |
| P7 | Hasivo | - | PoE, Basement |
| P8 | iZombie | 101 | Wired dev |
| P9 | X300 | 101 | Wired dev |
| P10 | H96 | 112 | Media box |
| P25-26 | OPNsense | Trunk | LACP uplink |

### ONTI S508CL-8S (Backend 10G)

| Port | Device | Connection | Notes |
|------|--------|------------|-------|
| P1 | hal | SFP+ | OPNsense |
| P2 | Tenda | SFP+ | 2.5G bridge |
| P3 | mothership | DAC 1.0m | X520-DA1 |
| P4 | - | Reserved | skynet Phase 2 |
| P5 | watson | AOC 7m | X520-DA1 M.2 |
| P6 | alexandria | AOC 7m | X520-DA1 |

### Tenda TEM2007 (Backend 2.5G)

| Port | Device | Notes |
|------|--------|-------|
| SFP+ | ONTI P2 | 10G uplink |
| P1 | deep-thought | eth1 |
| P2 | napster | eth1 |
| P3 | skynet | eth1 2.5G |

## Related

- [[homelab/network/ip-plan]]
- [[homelab/network/vlan-design]]
- [[homelab/network/architecture]]
- [[homelab/hosts]]