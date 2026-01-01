# VOS Hardware Inventory

> Last updated: 2025-01-01
> Collected from live systems via SSH

## Host Summary

| IP | Hostname | Role | CPU | Cores/Threads | RAM | Primary MAC |
|---|---|---|---|---|---|---|
| 192.168.254.100 | pve-Qotom-1u | hal (Router) | Intel Atom C3758R @ 2.40GHz | 8C/8T | 16 GB | 20:7c:14:f4:78:75 |
| 192.168.254.101 | pve-hx310-db | deep-thought (DB) | Intel Pentium J6426 @ 2.00GHz | 4C/4T | 32 GB | 84:8b:cd:4d:b6:f0 |
| 192.168.254.102 | pve-hx310-arr | napster (ARR) | Intel Pentium J6426 @ 2.00GHz | 4C/4T | 64 GB | 84:8b:cd:4d:b6:4f |
| 192.168.254.103 | pve-5800x | mothership | AMD Ryzen 7 5800X | 8C/16T | 32 GB | 70:85:c2:f8:4b:85 |
| 192.168.254.104 | pve-8845hs | skynet | AMD Ryzen 7 PRO 8845HS | 8C/16T | 64 GB | a8:b8:e0:0a:2f:e8 |
| 192.168.254.110 | Tower | alexandria (NAS) | AMD Ryzen 5 3600 | 6C/12T | 32 GB | 70:85:c2:f8:4b:8b |

## Network Interfaces

### hal (192.168.254.100) - Qotom C3758R - 9 NICs

| NIC | MAC Address | State | Notes |
|---|---|---|---|
| nic0 | 20:7c:14:f4:78:71 | DOWN | |
| nic1 | 20:7c:14:f4:78:72 | DOWN | |
| nic2 | 20:7c:14:f4:78:73 | DOWN | |
| nic3 | 20:7c:14:f4:78:74 | DOWN | |
| nic4 | 20:7c:14:f4:78:75 | UP | vmbr0 - Management |
| nic5 | 20:7c:14:f4:78:76 | DOWN | |
| nic6 | 20:7c:14:f4:78:77 | UP | |
| nic7 | 20:7c:14:f4:78:78 | DOWN | |
| nic8 | 20:7c:14:f4:78:79 | DOWN | |

### deep-thought (192.168.254.101) - HX310 - 2 NICs

| NIC | MAC Address | State | Notes |
|---|---|---|---|
| nic0 | 84:8b:cd:4d:b6:f0 | UP | vmbr0 - Management |
| nic1 | 84:8b:cd:4d:bd:30 | DOWN | |

### napster (192.168.254.102) - HX310 - 2 NICs

| NIC | MAC Address | State | Notes |
|---|---|---|---|
| nic0 | 84:8b:cd:4d:b6:4f | UP | vmbr0 - Management |
| nic1 | 84:8b:cd:4d:bc:8f | DOWN | |

### mothership (192.168.254.103) - 5800X - 1 NIC

| NIC | MAC Address | State | Notes |
|---|---|---|---|
| nic0 | 70:85:c2:f8:4b:85 | UP | vmbr0 - Management |

### skynet (192.168.254.104) - 8845HS - 2 NICs

| NIC | MAC Address | State | Notes |
|---|---|---|---|
| nic0 | a8:b8:e0:0a:2f:e8 | UP | vmbr0 - Management |
| nic1 | a8:b8:e0:0a:2f:e9 | DOWN | |

### alexandria (192.168.254.110) - NAS - 2 NICs

| NIC | MAC Address | State | Notes |
|---|---|---|---|
| eth0 | 70:85:c2:f8:4b:8b | UP | bond0/br0 - Management |
| eth1 | 90:e3:ba:01:d7:b7 | DOWN | |

## Storage

### hal (192.168.254.100)

| Device | Size | Model | Serial |
|---|---|---|---|
| nvme0n1 | 1 TB | Crucial P3 (CT1000P3SSD8) | 24494CB8AB2B |

### deep-thought (192.168.254.101)

| Device | Size | Model | Serial |
|---|---|---|---|
| sda | 1 TB | Hikvision E100N | 30081939081 |
| nvme0n1 | 1 TB | Samsung 970 EVO Plus | S6P7NS0X529310M |

### napster (192.168.254.102)

| Device | Size | Model | Serial |
|---|---|---|---|
| nvme0n1 | 1 TB | Samsung 970 EVO Plus | S6P7NS0X611130B |

### mothership (192.168.254.103)

| Device | Size | Model | Serial |
|---|---|---|---|
| nvme0n1 | 512 GB | Samsung 960 PRO | S3EWNX0K213926T |
| nvme1n1 | 1 TB | Lexar NM790 | NKS515R002947P2202 |
| nvme2n1 | 1 TB | Lexar NM790 | NKS515R002968P2202 |
| nvme3n1 | 1 TB | Lexar NM790 | NKS515R002955P2202 |

### skynet (192.168.254.104)

| Device | Size | Model | Serial |
|---|---|---|---|
| nvme0n1 | 500 GB | Crucial P5 Plus | 22493D0CE416 |
| nvme1n1 | 500 GB | Crucial P5 Plus | 23123F86BF7B |

### alexandria (192.168.254.110)

| Device | Size | Model | Serial |
|---|---|---|---|
| sda | 64 GB | Samsung FIT Flash | 0377324010001218 |
| sdb | 14.6 TB | Toshiba MG08ACA16TE | 53U0A037FVGG |
| sdc | 14.6 TB | Toshiba MG08ACA16TE | 83W0A12MFVGG |
| sdd | 14.6 TB | Toshiba MG08ACA16TE | 83W0A1AYFVGG |
| sde | 14.6 TB | Toshiba MG08ACA16TE | 53U0A0HFFVGG |
| sdf | 14.6 TB | Toshiba MG08ACA16TE | 53U0A08VFVGG |
| sdg | 14.6 TB | Toshiba MG08ACA16TE | 53U0A09EFVGG |
| sdh | 14.6 TB | Toshiba MG08ACA16TE | 1190A2JKFVGG |
| sdi | 14.6 TB | Toshiba MG08ACA16TE | 1190A2P3FVGG |
| nvme0n1 | 1 TB | Corsair MP600 | 1927822900012855029C |

**Total raw NAS capacity:** ~117 TB (8 × 14.6 TB)

## Notes

- alexandria runs Unraid (hostname "Tower"), target for Proxmox Backup Server
- hal has 9 NICs available for OPNsense router configuration
- mothership has 4 NVMe slots populated (3 × 1TB Lexar + 512GB Samsung boot)
- All Proxmox hosts are running on VLAN 254 (Management)
