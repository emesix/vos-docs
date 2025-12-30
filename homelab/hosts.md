---
title: Homelab Hosts Index
description: Index of all Proxmox nodes and infrastructure hosts
published: true
date: 2025-12-29T12:00:00.000Z
tags: infrastructure, host, proxmox, homelab, index
editor: markdown
dateCreated: 2025-12-26T08:36:01.418Z
---

# Homelab Hosts Index

Quick reference to all homelab hosts.

## Compute Summary

| Metric | Total |
|--------|-------|
| Cores | 74 |
| Threads | 132 |
| RAM | 384GB |
| 10GbE Hosts | 6 |
| Storage | 117TB NAS |

## Proxmox Cluster

| Host | Hostname | CPU | RAM | 10G | Role |
|------|----------|-----|-----|-----|------|
| [[hal]] | pve-qotom01 | C3758R 8C | 64GB | 4x SFP+ | OPNsense router |
| [[deep-thought]] | pve-hx310-db | J6426 4C | 32GB | - | Core services/DB |
| [[napster]] | pve-hx310-arr | J6426 4C | 32GB | - | ARR stack |
| [[mothership]] | pve-5800x | 5800X 8C/16T | 32GB | X520 | Docker/MCP |
| [[skynet]] | pve-8845hs | 8845HS 8C/16T | 64GB | 2x 10G | AI controller |
| [[watson]] | pve-x2686-x2 | 2xE5-2686v4 | 128GB | X520 | AI worker (GPU) |

## Storage

| Host | Hostname | CPU | RAM | 10G | Capacity |
|------|----------|-----|-----|-----|----------|
| [[alexandria]] | nas-unraid | R5 3600 | 32GB | X520 | 117TB |

## Development

| Host | Role |
|------|------|
| [[izombie]] | Pre-staging development |

## Network Backbone

| Switch | Ports | Purpose |
|--------|-------|---------|
| Zyxel GS1900-24HP | 24x 1G + 2 SFP | Frontend + PoE |
| ONTI S508CL-8S | 8x 10G SFP+ | Backend 10G |
| Tenda TEM2007 | 7x 2.5G + 1 SFP+ | Backend bridge |

## Related

- [[homelab/overview]]
- [[homelab/network/ip-plan]]
