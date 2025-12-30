---
title: OpenWrt Device Compatibility - VOS Network
description: OpenWrt support status for VOS network hardware including switches, routers, and wireless bridges
published: false
date: 2025-12-28T15:46:37.005Z
tags: networking, router, openwrt, switch, wireless, compatibility, wireguard, mesh
editor: markdown
dateCreated: 2025-12-28T12:32:03.295Z
---

---
title: "OpenWrt Device Compatibility - VOS Network"
uid: "openwrt-device-compatibility"
kind: [reference, software, network]
category: "03"
sub-category: "02"
visibility: 3.0
status: 2.0
tags: [openwrt, networking, switch, router, wireless, compatibility, wireguard, mesh]
parent_uid: ""
---

# OpenWrt Device Compatibility - VOS Network

**Category:** 03.02 - Protocols & Standards  
**Purpose:** Track OpenWrt support status for VOS network devices  
**Updated:** 2025-12-28

## Overview

Four device types in VOS inventory require OpenWrt. Only official and semi-official (same-board) firmware acceptable. No unofficial community builds.

## Device Inventory

| Device | Qty | Support | Use Case |
|--------|-----|---------|----------|
| ZyXEL GS1900-24HP v1 | 1 | Official | Frontend switch |
| ONTi S508CL-S8 | 1 | Official | Backend 10G switch |
| TP-Link Pharos CPE510 | 2 | Official | Garage P2P bridge |
| EDUP RT2980 | 5 | Official | WiFi mesh + remote APs |

## RT2980 Deployment Plan

| Unit | Location | Purpose | Backhaul |
|------|----------|---------|----------|
| #1 | Living room | Indoor AP | Wired (Zyxel P3) |
| #2 | Office | Indoor AP | Wired (Zyxel P4) |
| #3 | Bedroom | Indoor AP | Wired (Zyxel P5) |
| #4 | Basement | Remote AP | WireGuard tunnel |
| #5 | Garage | Remote AP | WireGuard over CPE510 |

---

## ZyXEL GS1900-24HP v1

**Type:** 24-port PoE Managed Switch  
**Support:** Official OpenWrt  
**Target:** realtek/rtl838x

### Specifications

- SoC: Realtek RTL8382M 500 MHz MIPS 4KEc
- Flash: 16 MiB
- RAM: 64 MiB DDR2 (v1) / 128 MiB DDR3 (v2)
- Ethernet: 24x GbE + 2x SFP
- PoE: 170W budget, 802.3at

### Install Method

U-Boot TFTP method. Serial access needed.

```
> rtk network on
> setsys bootpartition 0
> savesys
> tftpboot 0x81f00000 <LAN_IP>:openwrt-...-initramfs-kernel.bin
> bootm
```

Then sysupgrade from running OpenWrt.

---

## ONTi S508CL-S8 (ONT-S508CL-8S)

**Type:** 8-port 10G SFP+ L3 Managed Switch  
**Support:** Official (via XikeStor)  
**Target:** realtek/rtl930x

### Specifications

- SoC: Realtek RTL9303 800 MHz MIPS 34Kc
- Flash: SPI-NOR
- RAM: 512 MiB DDR3
- Ethernet: 8x 10G SFP+ (1G/2.5G/10G)
- Switching: 160Gbps non-blocking

### Important

ONTi S508CL-S8 = relabeled XikeStor SKS8300-8X. Use XikeStor images.

---

## TP-Link Pharos CPE510

**Type:** 5GHz Outdoor Wireless CPE  
**Support:** Official OpenWrt  
**Target:** ath79/generic (v3) or ar71xx (v1/v2)

### Specifications

- SoC: Atheros AR9344 (v1/v2) or AR934x (v3)
- Flash: 8 MiB
- RAM: 64 MiB
- Wireless: 5GHz 802.11a/n, 13dBi antenna
- Range: Up to 15km point-to-point

### VOS Usage

2x CPE510 for 200m balcony-to-garage P2P bridge.

| Unit | Location | Mode | Power |
|------|----------|------|-------|
| TX | Balcony | AP | 48V->24V converter |
| RX | Garage | Client | 48V->24V converter |

**Note:** CPE510 requires 24V passive PoE, not standard 48V 802.3af.

---

## EDUP RT2980

**Type:** WiFi 6 AX3000 Dual-Band Router  
**Support:** Official (via CreatLentem)  
**Target:** mediatek/filogic

### Specifications

- SoC: MediaTek MT7981B dual-core ARM Cortex-A53 1.3 GHz
- Flash: 128 MiB SPI-NAND
- RAM: 256 MiB DDR3
- Wireless: MT7976CN WiFi 6 (2x2 2.4GHz + 3x3 5GHz)
- Ethernet: 3x GbE LAN (MT7531AE) + 1x GbE WAN

### Firmware

Use CreatLentem CLT-R30B1 images. Same board as Dragonglass DXG21.

| Variant | UBI Size | Notes |
|---------|----------|-------|
| 64M | 64 MiB | Stock OpenWrt-based |
| 112M | 112 MiB | ImmortalWrt with LuCI |

### OpenWrt Mesh Configuration

```
# /etc/config/wireless (example)
config wifi-iface 'mesh0'
    option device 'radio0'
    option network 'mesh'
    option mode 'mesh'
    option mesh_id 'vos-mesh'
    option encryption 'sae'
    option key 'meshpassword'
    option mesh_fwding '1'
    option mesh_rssi_threshold '-80'
```

### WireGuard for Remote APs

Basement and garage APs use WireGuard to encrypt backhaul.

```
# /etc/config/network (remote AP)
config interface 'wg0'
    option proto 'wireguard'
    option private_key '<generated>'
    list addresses '10.200.0.X/32'

config wireguard_wg0
    option public_key '<opnsense-pubkey>'
    option endpoint_host '<LAN_IP>'
    option endpoint_port '51820'
    list allowed_ips '0.0.0.0/0'
    option persistent_keepalive '25'
```

### 802.11r Fast Roaming

```
# /etc/config/wireless (all APs)
config wifi-iface 'default_radio0'
    option ieee80211r '1'
    option mobility_domain 'v05a'
    option ft_over_ds '0'
    option ft_psk_generate_local '1'
```

---

## Cross-Vendor Firmware Compatibility

| Brand A | Brand B | OpenWrt Image |
|---------|---------|---------------|
| ONTi S508CL-S8 | XikeStor SKS8300-8X | xikestor_sks8300-8x |
| EDUP RT2980 | CreatLentem CLT-R30B1 | creatlentem_clt-r30b1 |
| EDUP RT2980 | Dragonglass DXG21 | creatlentem_clt-r30b1 |

## Resources

- [OpenWrt Table of Hardware](https://openwrt.org/toh/start)
- [OpenWrt Forum](https://forum.openwrt.org/)
- [OpenWrt Downloads](https://downloads.openwrt.org/)
- [EDUP RT2980 Thread](https://forum.openwrt.org/t/edup-rt2980-openwrt-version/226930)

## Related

- [[homelab/network/architecture]]
- [[homelab/network/vlan-design]]
- [[homelab/hosts/hal]]