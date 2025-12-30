---
title: PCI Passthrough on Proxmox VE
description: Configure IOMMU and VFIO for PCI device passthrough to VMs on Intel/AMD Proxmox hosts
published: true
date: 2025-12-28T10:15:16.601Z
tags: hardware, configuration, howto, proxmox
editor: markdown
dateCreated: 2025-12-28T10:15:14.125Z
---

---
title: "PCI Passthrough on Proxmox VE"
uid: "howto-pci-passthrough-proxmox"
kind: [howto, procedure, stable]
category: "03"
sub-category: "01"
visibility: 0.0
status: 2.0
tags: [proxmox, howto, hardware, configuration, vfio, iommu]
parent_uid: ""
---

# PCI Passthrough on Proxmox VE

**Category:** 03.01 - Operating Systems  
**Purpose:** Enable direct hardware access for VMs via IOMMU/VFIO

## Overview

PCI passthrough allows VMs to directly access physical hardware devices.
Requires IOMMU (VT-d on Intel, AMD-Vi on AMD) support in CPU and motherboard.
Devices must be in isolated IOMMU groups for clean passthrough.

## Requirements

- CPU with IOMMU support (Intel VT-d or AMD-Vi)
- Motherboard with IOMMU enabled in BIOS
- Proxmox VE 7.x or 8.x/9.x
- Q35 machine type for PCIe passthrough

## Step 1: Enable IOMMU in GRUB

Edit `/etc/default/grub`:

```bash
# Intel
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"

# AMD
GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on iommu=pt"
```

Apply changes:

```bash
update-grub
reboot
```

## Step 2: Load VFIO Modules

Add to `/etc/modules`:

```
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```

Regenerate initramfs:

```bash
update-initramfs -u -k all
```

## Step 3: Blacklist Host Drivers

Create `/etc/modprobe.d/blacklist-device.conf`:

```bash
blacklist <driver_name>
```

Bind devices to VFIO in `/etc/modprobe.d/vfio.conf`:

```bash
options vfio-pci ids=XXXX:XXXX,YYYY:YYYY
softdep <driver_name> pre: vfio-pci
```

Replace `XXXX:XXXX` with PCI vendor:device IDs from `lspci -nn`.

## Step 4: Verify IOMMU Groups

Check IOMMU is active:

```bash
dmesg | grep -i iommu
```

List IOMMU groups:

```bash
for d in /sys/kernel/iommu_groups/*/devices/*; do
  n=$(basename $d)
  g=$(basename $(dirname $(dirname $d)))
  echo "IOMMU Group $g: $(lspci -nns $n)"
done | sort -V
```

→ Each passthrough device should be in its own IOMMU group.

## Step 5: Configure VM

In VM config (`/etc/pve/qemu-server/<vmid>.conf`):

```
machine: q35
hostpci0: XX:XX.X,pcie=1
```

For GPU passthrough, add `x-vga=1`.

## Verification

- `lspci -nnk` shows vfio-pci driver bound to devices
- VM starts without errors
- Device visible inside VM

## Related

- [[host-qotom-c3758r-pci-passthrough]]
- [[03-01/os-proxmox-8]]