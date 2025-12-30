---
title: VOS Document Pipeline
description: Knowledge flow as infrastructure - a first-class VOS subsystem
published: true
date: 2025-12-29T12:30:00.000Z
tags: vos, documents, pipeline, knowledge, mcp
editor: markdown
dateCreated: 2025-12-29T12:30:00.000Z
---

# VOS Document Pipeline

Knowledge flow is infrastructure. This pipeline is a first-class VOS subsystem.

## Why This Matters

For VOS, knowledge flow is as important as packet flow:

- Conversations become documentation
- Documentation becomes configuration
- Configuration becomes running services
- Services generate telemetry
- Telemetry informs conversations

## Pipeline Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    KNOWLEDGE SOURCES                     │
├──────────────┬──────────────┬──────────────┬────────────┤
│   Claude     │   ChatGPT    │   Manual     │  Metrics   │
│   Desktop    │   Sessions   │   Notes      │  Logs      │
└──────┬───────┴──────┬───────┴──────┬───────┴─────┬──────┘
       │              │              │             │
       ▼              ▼              ▼             ▼
┌─────────────────────────────────────────────────────────┐
│                   INGESTION LAYER                        │
│  MCP Servers → Haystack → Embeddings → Classification   │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                   PERSISTENCE LAYER                      │
│         Wiki.js (Markdown) + Qdrant (Vectors)           │
│                    PostgreSQL (Meta)                     │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                    RETRIEVAL LAYER                       │
│   RAG Queries ← Haystack ← LiteLLM ← User Requests      │
└─────────────────────────────────────────────────────────┘
```

## Components

| Component | Host | Role |
|-----------|------|------|
| Wiki.js | deep-thought | Markdown storage, UI |
| Qdrant | deep-thought | Vector embeddings |
| Haystack | skynet | RAG orchestration |
| WikiJS MCP | mothership | Claude ↔ Wiki bridge |
| LiteLLM | skynet | Query routing |

## Document Lifecycle

1. **Capture**: Conversation produces insight
2. **Classify**: Determine authority level (see [[vos/authority]])
3. **Store**: Wiki.js for text, Qdrant for vectors
4. **Link**: Cross-reference related documents
5. **Retrieve**: RAG finds relevant context
6. **Update**: New insights refine existing docs

## MCP Integration

| MCP Server | Function |
|------------|----------|
| WikiJS MCP | CRUD operations on wiki pages |
| Desktop Commander | File system access |
| Proxmox MCP | Infrastructure state |

## Quality Gates

Before a document is authoritative:

- [ ] Single topic per document
- [ ] Follows writing style guide
- [ ] Cross-referenced appropriately
- [ ] Authority level assigned
- [ ] No duplicate information

## Metrics (Future)

| Metric | Purpose |
|--------|---------|
| Docs created/day | Pipeline velocity |
| Stale doc count | Maintenance debt |
| RAG hit rate | Retrieval quality |
| Cross-ref density | Knowledge connectivity |

## Related

- [[vos/README]]
- [[vos/authority]]
- [[vos/services]]
- [[components/desktop-commander-mcp]]
