---
title: VOS Architecture Overview
description: High-level architecture overview of VOS including layers, components, roadmap, and integration strategy
published: true
date: 2025-12-25T19:52:38.271Z
tags: vos, ai, architecture, knowledge-management, automation
editor: markdown
dateCreated: 2025-12-25T19:52:35.534Z
---

# VOS (user Operating System) - Architecture Overview

## Vision

VOS is a high-level personal operating system that integrates AI with a homelab-based productivity environment. It unifies ideas, tasks, knowledge, and tools into one cohesive system enhanced by AI capabilities.

## Core Philosophy

**"From AI as advisor to AI as integrated collaborator"**

- Self-hosted or open-source where possible
- Full control over data
- Feasible for solo developer
- Integration over invention
- Foundation stability before complexity

## System Architecture Layers

```
VOS (user Operating System)
├── AI Integration & Intelligence
│   ├── LLM Connector (Claude via OpenRouter, other models)
│   ├── AI Orchestrator/Agent (MCP-based tool use)
│   └── Session Manager (multi-session context)
├── Knowledge Management Module
│   ├── Personal Knowledge Base (Markdown vault)
│   ├── Knowledge Index & Search (full-text/embeddings)
│   └── Version Control (Git for notes)
├── Task & Project Management Module
│   ├── Task Tracker / To-Do Lists
│   ├── Project Planner
│   └── Calendar/Scheduling Integration
├── Automation & Scripting Module
│   ├── Workflow Orchestration (Node-RED/n8n)
│   ├── Script Runner
│   └── API Integrations
├── Core Services & Infrastructure
│   ├── Data Storage Layer (homelab storage)
│   ├── Access & Security
│   ├── Scheduling & Triggers
│   └── Sync & Backup
└── Interface Layer
    ├── Personal Dashboard UI
    ├── Chat Interface
    └── Mobile/Remote Access
```

## Key Components

### AI Integration Layer

**Purpose:** Cognitive capabilities powering the system

**Components:**
- **LLM Connector:** API calls to Claude/other models via OpenRouter
- **AI Orchestrator:** MCP-style agent with tool use and context management
- **Session Manager:** Multi-session support with persistent memory

**Technologies:**
- OpenRouter (unified API for multiple models)
- Model Context Protocol (MCP)
- Node-RED AI Agent nodes
- LangChain/LangGraph (advanced agent flows)

### Knowledge Management Module

**Purpose:** "Second brain" for persistent information

**Components:**
- **Markdown Vault:** Plain text notes (Obsidian-compatible)
- **Linking & Organization:** Wiki-style links, tags, folders
- **AI Integration:** Read/write vault via MCP tools
- **Search:** Full-text + semantic vector search

**Technologies:**
- Markdown files
- Git (version control)
- Obsidian (GUI)
- Vector database (Qdrant/pgvector)

**Key Feature:** AI can directly update knowledge base, eliminating copy-paste

### Task & Project Management Module

**Purpose:** Execution layer for goals and todos

**Components:**
- **Task Tracker:** To-do lists with status, dates, priorities
- **Project Planner:** Group tasks into projects/milestones
- **Calendar Integration:** Sync deadlines to calendar

**AI Enhancement:**
- Auto-generate action items from meetings
- Create project plans from brainstorming
- Update task status through conversation

### Automation & Scripting Module

**Purpose:** Glue and action engine connecting services

**Components:**
- **Node-RED/n8n:** Visual workflow orchestration
- **Custom Scripts:** Python/JavaScript for specialized logic
- **API Integrations:** External service connections
- **Scheduling:** Cron-like triggers and event-driven flows

**Technologies:**
- Node-RED with AI Agent nodes
- n8n workflow automation
- Custom Python/JS scripts
- Webhooks and API endpoints

### Core Services Layer

**Purpose:** Backend systems supporting VOS

**Components:**
- **Data Storage:** Files on homelab (local control)
- **Backend Services:** Node-RED, databases, containers
- **Security:** API key management, authentication
- **Version Control:** Git for vault and configs
- **Backup:** Regular snapshots, Git history

### Interface Layer

**Purpose:** User interaction with VOS

**Components:**
- **Unified Dashboard:** Overview of tasks, notes, AI suggestions
- **Knowledge Base GUI:** Obsidian for note editing
- **Chat/AI Interface:** Conversation with AI assistant
- **Mobile/Remote Access:** Secure access from anywhere

## The Two-Claude Architecture

**Cloud Claude (claude.ai web):**
- Planning and documentation
- Architecture design
- Template creation
- Cannot access iZombie directly

**Claude Desktop (on iZombie):**
- Executes actual work
- Full system access via MCP
- Real-time implementation
- Documents as it works

**Critical Issue:** System prompt doesn't update when chat migrates between environments!

## Implementation Roadmap

### Phase 1: Foundation - AI Core & Knowledge Base (Month 1-3)

**Goals:**
- LLM integration via OpenRouter API
- Basic chat interface with memory
- Markdown vault establishment
- AI vault read/write capability

**Deliverables:**
- Working AI assistant on homelab
- Personal knowledge base with AI access
- Initial tool use (ReadNote, WriteNote)

### Phase 2: Productivity Tools - Tasks & Projects (Month 3-5)

**Goals:**
- Task management integration
- Project planning structure
- AI-assisted task creation
- Conversation-to-action flow

**Deliverables:**
- Task tracker (file or database)
- Project pages in vault
- AI tools: AddTask, UpdateTaskStatus, ListTasks
- Automated project planning

### Phase 3: Automation & Advanced Orchestration (Month 5-7)

**Goals:**
- Workflow library creation
- AI tool integration for automation
- External service orchestration
- Scheduled jobs and triggers

**Deliverables:**
- Node-RED flows for common tasks
- AI-invoked automation tools
- Integration with external APIs
- Self-healing capabilities

### Phase 4: Refinement - Multi-Session, UI & Optimization (Month 7-9)

**Goals:**
- Multi-session context management
- UI/UX enhancements
- Performance optimization
- Security hardening

**Deliverables:**
- Session isolation and persistence
- Cohesive web interface
- Monitoring and logging
- Documentation and backup systems

## Integration with Homelab Infrastructure

### Pre-Staging: iZombie Development

**Environment:**
- 2014 iMac (Arch Linux, KDE Plasma)
- Claude Desktop + MCP servers
- Docker for service containers
- Local Git repositories

**Purpose:**
- Plan before implementing
- Test configurations
- Avoid cascade failures
- Document extensively

### Production: Distributed Deployment

**AI Services (VLAN 40):**
- Host: 8845HS (NPU acceleration)
- Services: LiteLLM, vLLM, Haystack, Open WebUI

**Automation (VLAN 40):**
- Host: 5800X or 8845HS
- Services: n8n, Node-RED, custom orchestrator

**Knowledge Base (VLAN 50):**
- Host: HX-310 #1
- Services: Wiki.js, PostgreSQL, vector database

**Storage:**
- unRAID NAS (centralized)
- Git repositories
- Database backups

## Priority Correction from Lessons Learned

**Previous Approach (Failed):**
- Complex AI orchestration first
- Basic infrastructure neglected
- WiFi/network broken
- No "default relaxation" services

**Corrected Approach:**
1. **Network Foundation** (WiFi, VLANs, proper segmentation)
2. ***ARR Stack** (media services for quality of life)
3. **AI Complexity** (after basics work)

**Rationale:** Foundation stability prevents motivation loss during development

## Key Design Decisions

### Why Off-the-Shelf Components First?

- Battle-tested reliability
- Fixed versions prevent "foundation churn"
- Known compatibility
- Faster deployment
- Custom services built on stable base

### Why VLAN Segmentation?

- Logical separation without physical isolation
- Containers get real IPs (can migrate hosts)
- Router as DHCP boss (central authority)
- Security through firewall rules
- Performance through backend 10GbE switch

### Why Markdown Vault?

- Plain text (future-proof)
- Git version control
- Obsidian GUI compatibility
- AI can read/write directly
- No vendor lock-in

### Why Node-RED/n8n?

- Visual workflow design
- Extensive integration library
- AI agent nodes available
- Self-hosted
- Accessible to non-programmers

## Success Metrics

**Foundation (Phase 1-2):**
- AI responds with vault context
- AI creates/updates notes automatically
- Tasks tracked and visible
- Projects organized

**Automation (Phase 3):**
- Workflows execute on triggers
- AI invokes tools successfully
- External services integrated
- Self-healing responds to alerts

**Production (Phase 4):**
- Multi-session context maintained
- No data loss (backups working)
- Secure remote access
- Daily usage without friction

## Technology Stack Summary

| Layer | Technology | Purpose |
|-------|-----------|----------|
| AI | OpenRouter, Claude, LiteLLM | LLM access |
| Agent | MCP, Node-RED AI Agent | Tool use |
| Knowledge | Markdown, Git, Obsidian | Notes |
| Search | Qdrant, pgvector | Vector DB |
| Tasks | Files or SQLite | To-do |
| Automation | n8n, Node-RED | Workflows |
| Infrastructure | Proxmox, Docker | Containers |
| Storage | unRAID, Git | Data |
| Frontend | Web UI, Obsidian | Interface |

## References

- [Network & Staging Environment](/vos/network-and-staging)
- [Off-the-Shelf AI Services](/vos/ai-services)
- [Staging Environment Design](/vos/staging-design)

---

*Last Updated: 2025-12-25*  
*Status: Architecture defined, pre-staging active*