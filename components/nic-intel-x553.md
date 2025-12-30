---
title: Intel X553 10GbE SFP+ NIC
description: Intel X553 10GbE SFP+ network interface controller for high-speed server connectivity
published: true
date: 2025-12-28T10:16:26.642Z
tags: network, hardware, component, intel
editor: markdown
dateCreated: 2025-12-28T10:16:24.158Z
---

---
title: "Intel X553 10GbE SFP+ NIC"
uid: "nic-intel-x553"
kind: [component, hardware, network]
category: "02"
sub-category: "52"
visibility: 0.0
status: 2.0
tags: [intel, network, hardware, component, nic, 10gbe, sfp]
parent_uid: ""
---

# Intel X553 10GbE SFP+ NIC

**Category:** 02.52 - Network Components  
**Purpose:** 10GbE SFP+ network interface for server connectivity

## Overview

Intel X553 is a 10GbE SFP+ NIC used in enterprise and homelab servers.
Uses ixgbe driver in Linux.
PCI ID: 8086:15c4.

## Specifications

| Property | Value |
|----------|-------|
| Speed | 10 Gbps |
| Connector | SFP+ |
| Interface | PCIe 3.0 |
| PCI Vendor:Device | 8086:15c4 |
| Linux Driver | ixgbe |

## VFIO Passthrough

For PCI passthrough, blacklist ixgbe driver and bind to vfio-pci:

**`/etc/modprobe.d/blacklist-ixgbe.conf`:**
```
blacklist ixgbe
```

**`/etc/modprobe.d/vfio.conf`:**
```bash
options vfio-pci ids=8086:15c4
softdep ixgbe pre: vfio-pci
```

## Used By

- [[host-qotom-c3758r]] (4x ports)

## Platform Notes

→ Proxmox passthrough: [[host-qotom-c3758r-pci-passthrough]]

## Related

- [[nic-intel-i226-v]]
- [[howto-pci-passthrough-proxmox]]