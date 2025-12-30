---
title: skynet - AI Controller
description: CWWK 8845HS mini-PC running AI orchestration, LiteLLM, Open WebUI, and Haystack
published: true
date: 2025-12-29T12:00:00.000Z
tags: host, proxmox, amd, ai, llm, npu
editor: markdown
dateCreated: 2025-12-26T08:34:46.789Z
---

# skynet - AI Controller

**Role:** AI orchestration, LLM routing, RAG pipeline  
**Hostname:** pve-8845hs  
**Frontend IP:** <LAN_IP>  
**Backend IP:** <LAN_IP>

## Overview

CWWK mini-PC with AMD Ryzen 8845HS featuring integrated NPU. AI controller node running LiteLLM proxy, Open WebUI, Haystack RAG, and SearxNG. Backend via 10GbE RJ45.

## Hardware

| Component | Specification |
|-----------|---------------|
| Model | CWWK 8845HS |
| CPU | AMD Ryzen 8845HS (8C/16T @ 3.8-5.1GHz) |
| NPU | AMD XDNA (16 TOPS) |
| RAM | 64GB DDR5 |
| Storage | 1TB NVMe |
| 10GbE | 2x 10G RJ45 (built-in) |

## Network

| Interface | Switch | Speed | Purpose |
|-----------|--------|-------|---------|
| eth0 | Zyxel | 1GbE | Frontend |
| eth1 | Tenda P3 | 2.5GbE | Backend |

## Services (VLAN 200)

| Service | IP | Purpose |
|---------|-----|---------|
| Open WebUI | <LAN_IP> | Chat interface |
| LiteLLM | <LAN_IP> | LLM proxy/router |
| Haystack | <LAN_IP> | RAG pipeline |
| SearxNG | <LAN_IP> | Search aggregator |

## Related

- [[homelab/hosts]]
- [[homelab/network/ip-plan]]
- [[watson]] (AI worker with GPU)
