---
title: watson - AI Worker (GPU)
description: X99 dual Xeon E5-2686v4 with 128GB RAM for GPU inference workloads
published: true
date: 2025-12-29T12:00:00.000Z
tags: host, proxmox, xeon, ai, gpu, inference, 10gbe
editor: markdown
dateCreated: 2025-12-26T08:34:56.321Z
---

# watson - AI Worker (GPU)

**Role:** AI inference, GPU compute  
**Hostname:** pve-x2686-x2  
**Frontend IP:** <LAN_IP>  
**Backend IP:** <LAN_IP>

## Overview

X99 workstation with dual Intel Xeon E5-2686v4 (36 cores total) and 128GB RAM. Heavy AI inference workloads with GPU passthrough for vLLM. 10GbE backend via Intel X520-DA1.

## Hardware

| Component | Specification |
|-----------|---------------|
| Platform | X99 Dual Socket |
| CPU | 2x Intel Xeon E5-2686v4 (18C/36T each) |
| RAM | 128GB DDR4 ECC |
| Storage | 1TB NVMe |
| GPU | (GPU slot available for AI acceleration) |
| 10GbE | Intel X520-DA1 M.2 adapter (SFP+) |

## Network

| Interface | Switch | Speed | Purpose |
|-----------|--------|-------|---------|
| eth0 | Zyxel | 1GbE | Frontend |
| X520 SFP+ | ONTI P5 | 10GbE | Backend (AOC) |

## Services (VLAN 200)

| Service | IP | Purpose |
|---------|-----|---------|
| vLLM | <LAN_IP> | Inference engine |

## Related

- [[homelab/hosts]]
- [[homelab/network/ip-plan]]
- [[skynet]] (AI controller)
- [[components/nic-intel-x553]]
