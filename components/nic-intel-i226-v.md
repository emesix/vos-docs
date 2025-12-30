---
title: Intel I226-V 2.5GbE NIC
description: Intel I226-V 2.5GbE network interface controller commonly found in mini-PCs and edge devices
published: true
date: 2025-12-28T10:16:10.344Z
tags: network, hardware, component, intel
editor: markdown
dateCreated: 2025-12-28T10:16:08.048Z
---

---
title: "Intel I226-V 2.5GbE NIC"
uid: "nic-intel-i226-v"
kind: [component, hardware, network]
category: "02"
sub-category: "52"
visibility: 0.0
status: 2.0
tags: [intel, network, hardware, component, nic, 2.5gbe]
parent_uid: ""
---

# Intel I226-V 2.5GbE NIC

**Category:** 02.52 - Network Components  
**Purpose:** 2.5GbE network interface for edge/mini-PC devices

## Overview

Intel I226-V is a 2.5GbE NIC commonly used in Qotom and similar mini-PCs.
Uses igc driver in Linux kernel 5.10+.
PCI ID: 8086:125c.

## Specifications

| Property | Value |
|----------|-------|
| Speed | 2.5 Gbps |
| Interface | PCIe 3.0 x1 |
| PCI Vendor:Device | 8086:125c |
| Linux Driver | igc |
| Kernel Support | 5.10+ |

## VFIO Passthrough

For PCI passthrough, blacklist igc driver and bind to vfio-pci:

**`/etc/modprobe.d/blacklist-igc.conf`:**
```
blacklist igc
```

**`/etc/modprobe.d/vfio.conf`:**
```bash
options vfio-pci ids=8086:125c
softdep igc pre: vfio-pci
```

## Used By

- [[host-qotom-c3758r]] (5x ports)

## Platform Notes

→ Proxmox passthrough: [[host-qotom-c3758r-pci-passthrough]]

## Related

- [[nic-intel-x553]]
- [[howto-pci-passthrough-proxmox]]