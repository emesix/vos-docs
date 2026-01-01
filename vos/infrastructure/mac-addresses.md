# VOS MAC Address Table

> Last updated: 2025-01-01

## Summary Table

| Host | Role | IP | Primary MAC | Interface |
|---|---|---|---|---|
| hal | Router | 192.168.254.100 | 20:7c:14:f4:78:75 | nic4 |
| deep-thought | Database | 192.168.254.101 | 84:8b:cd:4d:b6:f0 | nic0 |
| napster | ARR/Docker | 192.168.254.102 | 84:8b:cd:4d:b6:4f | nic0 |
| mothership | Compute | 192.168.254.103 | 70:85:c2:f8:4b:85 | nic0 |
| skynet | Compute | 192.168.254.104 | a8:b8:e0:0a:2f:e8 | nic0 |
| alexandria | NAS | 192.168.254.110 | 70:85:c2:f8:4b:8b | eth0 |

## Full NIC Inventory

### hal (Qotom C3758R) - 9 NICs

| NIC | MAC Address |
|---|---|
| nic0 | 20:7c:14:f4:78:71 |
| nic1 | 20:7c:14:f4:78:72 |
| nic2 | 20:7c:14:f4:78:73 |
| nic3 | 20:7c:14:f4:78:74 |
| nic4 | 20:7c:14:f4:78:75 |
| nic5 | 20:7c:14:f4:78:76 |
| nic6 | 20:7c:14:f4:78:77 |
| nic7 | 20:7c:14:f4:78:78 |
| nic8 | 20:7c:14:f4:78:79 |

### deep-thought (HX310) - 2 NICs

| NIC | MAC Address |
|---|---|
| nic0 | 84:8b:cd:4d:b6:f0 |
| nic1 | 84:8b:cd:4d:bd:30 |

### napster (HX310) - 2 NICs

| NIC | MAC Address |
|---|---|
| nic0 | 84:8b:cd:4d:b6:4f |
| nic1 | 84:8b:cd:4d:bc:8f |

### mothership (5800X) - 1 NIC (SFP card removed for VGA troubleshooting)

| NIC | MAC Address |
|---|---|
| nic0 | 70:85:c2:f8:4b:85 |

> **Note:** SFP network card temporarily removed. VGA card installed for troubleshooting.

### skynet (8845HS) - 2 NICs

| NIC | MAC Address |
|---|---|
| nic0 | a8:b8:e0:0a:2f:e8 |
| nic1 | a8:b8:e0:0a:2f:e9 |

### alexandria (Unraid NAS) - 2 NICs

| NIC | MAC Address |
|---|---|
| eth0 | 70:85:c2:f8:4b:8b |
| eth1 | 90:e3:ba:01:d7:b7 |
