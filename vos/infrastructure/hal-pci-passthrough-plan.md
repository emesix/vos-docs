# HAL (Qotom) PCI Passthrough Planning

> Created: 2025-01-01
> Status: PLANNING
> Host: 192.168.254.100 (pve-Qotom-1u)

## Objective

Configure PCI passthrough on hal (Qotom C3758R) to pass 4× I226-V NICs to OPNsense VM while retaining 1× I226-V for Proxmox management.

## Current State

### Network Interfaces

| NIC | PCI Address | Type | MAC | Current Use | Target Use |
|---|---|---|---|---|---|
| nic0 | 0000:04:00.0 | I226-V | 20:7c:14:f4:78:71 | DOWN | → VM (OPNsense) |
| nic1 | 0000:05:00.0 | I226-V | 20:7c:14:f4:78:72 | DOWN | → VM (OPNsense) |
| nic2 | 0000:06:00.0 | I226-V | 20:7c:14:f4:78:73 | DOWN | → VM (OPNsense) |
| nic3 | 0000:07:00.0 | I226-V | 20:7c:14:f4:78:74 | DOWN | → VM (OPNsense) |
| nic4 | 0000:08:00.0 | I226-V | 20:7c:14:f4:78:75 | vmbr0 (mgmt) | **KEEP for Proxmox** |
| nic5 | 0000:0b:00.0 | X553 SFP+ | 20:7c:14:f4:78:76 | DOWN | → VM (OPNsense) |
| nic6 | 0000:0b:00.1 | X553 SFP+ | 20:7c:14:f4:78:77 | UP (backup) | → VM (OPNsense) |
| nic7 | 0000:0c:00.0 | X553 SFP+ | 20:7c:14:f4:78:78 | DOWN | → VM (OPNsense) |
| nic8 | 0000:0c:00.1 | X553 SFP+ | 20:7c:14:f4:78:79 | DOWN | → VM (OPNsense) |

### IOMMU Groups (All separate - good!)

| IOMMU Group | PCI Device | NIC |
|---|---|---|
| 23 | 0000:04:00.0 | nic0 |
| 24 | 0000:05:00.0 | nic1 |
| 25 | 0000:06:00.0 | nic2 |
| 26 | 0000:07:00.0 | nic3 |
| 27 | 0000:08:00.0 | nic4 |
| 29 | 0000:0b:00.0 | nic5 |
| 30 | 0000:0b:00.1 | nic6 |
| 31 | 0000:0c:00.0 | nic7 |
| 32 | 0000:0c:00.1 | nic8 |

### Current GRUB Configuration

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet"
```

IOMMU is already enabled (visible in dmesg), but not explicitly set in GRUB.

## Implementation Plan

### Phase 1: Enable IOMMU in GRUB (Safe)

**Purpose:** Explicitly enable IOMMU and prepare for VFIO.

1. Edit `/etc/default/grub`:
   ```
   GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"
   ```

2. Update GRUB:
   ```bash
   update-grub
   ```

3. Reboot and verify:
   ```bash
   dmesg | grep -e DMAR -e IOMMU
   ```

**Rollback:** Remove parameters from GRUB, update-grub, reboot.

### Phase 2: Load VFIO Modules

**Purpose:** Load required kernel modules for PCI passthrough.

1. Create `/etc/modules-load.d/vfio.conf`:
   ```
   vfio
   vfio_iommu_type1
   vfio_pci
   ```

2. Reboot and verify:
   ```bash
   lsmod | grep vfio
   ```

**Rollback:** Remove the file, reboot.

### Phase 3: Bind NICs to VFIO (CRITICAL)

> ⚠️ **WARNING:** This phase can cause loss of network connectivity.
> nic6 (SFP+) is currently connected as backup. Do NOT proceed until nic4 is confirmed working.

**Pre-flight Checklist:**
- [ ] nic4 is UP and responding on 192.168.254.100
- [ ] SSH session confirmed over nic4 (vmbr0)
- [ ] nic6 can be safely released

**Devices to bind to VFIO:**

| PCI ID | Device ID | NIC | Purpose |
|---|---|---|---|
| 0000:04:00.0 | 8086:125c | nic0 | OPNsense LAN |
| 0000:05:00.0 | 8086:125c | nic1 | OPNsense WAN |
| 0000:06:00.0 | 8086:125c | nic2 | OPNsense OPT1 |
| 0000:07:00.0 | 8086:125c | nic3 | OPNsense OPT2 |
| 0000:0b:00.0 | 8086:15c4 | nic5 | OPNsense SFP1 |
| 0000:0b:00.1 | 8086:15c4 | nic6 | OPNsense SFP2 |
| 0000:0c:00.0 | 8086:15c4 | nic7 | OPNsense SFP3 |
| 0000:0c:00.1 | 8086:15c4 | nic8 | OPNsense SFP4 |

**NOT binding (Proxmox management):**
- 0000:08:00.0 (nic4) - stays with Proxmox for vmbr0

1. Create `/etc/modprobe.d/vfio.conf`:
   ```
   options vfio-pci ids=8086:125c,8086:15c4
   softdep igc pre: vfio-pci
   softdep ixgbe pre: vfio-pci
   ```

   > **Problem:** This binds ALL I226-V NICs including nic4!
   > **Solution:** Bind by PCI address instead of device ID.

2. Alternative - bind specific PCI addresses via script at boot:
   Create `/etc/initramfs-tools/scripts/init-top/vfio-bind.sh`

3. Or use Proxmox UI to assign PCI devices to VM after creation.

**Recommended approach:** Do NOT pre-bind. Let Proxmox handle passthrough at VM level.

### Phase 4: Verify and Test

1. Confirm nic4/vmbr0 still works:
   ```bash
   ping -c 3 192.168.254.1
   ip addr show vmbr0
   ```

2. Verify VFIO ready:
   ```bash
   dmesg | grep vfio
   ```

3. Check PCI devices available for passthrough:
   ```bash
   pvesh get /nodes/pve-Qotom-1u/hardware/pci --pci-class-blacklist ""
   ```

## Execution Checklist

### Before Starting

- [ ] Physical access available (crash recovery)
- [ ] IPMI/iKVM available? **NO - Qotom has no IPMI**
- [ ] Backup current config:
  ```bash
  cp /etc/default/grub /etc/default/grub.bak
  cp /etc/network/interfaces /etc/network/interfaces.bak
  ```

### Go/No-Go Decision Points

| Checkpoint | Condition | If FAIL |
|---|---|---|
| After Phase 1 | Host boots, SSH works via nic4 | Restore grub.bak, reboot |
| After Phase 2 | vfio modules loaded | Remove modules file, reboot |
| After Phase 3 | vmbr0 on nic4 responds | Physical console recovery |

### Recovery Procedure

If host becomes unreachable:

1. Connect monitor + keyboard to Qotom
2. Boot to recovery/single-user mode
3. Restore backup files:
   ```bash
   cp /etc/default/grub.bak /etc/default/grub
   update-grub
   rm /etc/modprobe.d/vfio.conf
   rm /etc/modules-load.d/vfio.conf
   update-initramfs -u
   reboot
   ```

## Final Target State

```
┌─────────────────────────────────────────────────────────┐
│ Proxmox Host (pve-Qotom-1u)                             │
│                                                         │
│   vmbr0 ─── nic4 (08:00.0) ─── 192.168.254.100         │
│                                                         │
│   ┌─────────────────────────────────────────────────┐   │
│   │ OPNsense VM                                     │   │
│   │                                                 │   │
│   │   PCI Passthrough:                              │   │
│   │   - nic0 (04:00.0) → LAN                       │   │
│   │   - nic1 (05:00.0) → WAN                       │   │
│   │   - nic2 (06:00.0) → OPT1                      │   │
│   │   - nic3 (07:00.0) → OPT2                      │   │
│   │   - nic5 (0b:00.0) → SFP1                      │   │
│   │   - nic6 (0b:00.1) → SFP2                      │   │
│   │   - nic7 (0c:00.0) → SFP3                      │   │
│   │   - nic8 (0c:00.1) → SFP4                      │   │
│   └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

## References

- Proxmox PCI Passthrough: https://pve.proxmox.com/wiki/PCI_Passthrough
- Intel IOMMU: https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF
