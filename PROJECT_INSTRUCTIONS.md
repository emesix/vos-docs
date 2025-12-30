# VOS Project Instructions

## Single Source of Truth

**Repository:** https://github.com/emesix/vos-docs  
**Local copy:** `/home/emesix/VOS-Docs/`

All documentation lives here. All updates go here first.

---

## Authority Rules

### Rule 1: Documentation Before Implementation
If documentation and a running system disagree, **documentation is updated first** — never "just fix it live". This prevents undocumented drift.

### Rule 2: No Silent Changes
Every change to infrastructure must be recorded. Undocumented changes do not exist.

### Rule 3: Negative Testing Required
No step is "complete" unless at least one negative test is performed (e.g., access blocked where it should be, service unreachable from wrong VLAN).

---

## What Memory Is (and Is Not)

**Memory contains:**
- Stable project invariants
- Finalized architectural decisions
- Hardware assignments
- Network topology

**Memory does NOT contain:**
- Transient decisions
- Experiments and hypotheses
- Troubleshooting notes
- Work-in-progress

→ These belong in `progress/` reports or implementation notes.

---

## Implementation Workflow

### Before Starting Any Step

1. **Check VOS_Master_TODO.md** - Confirm current phase/task
2. **Validate prerequisites** - Are prior steps complete?
3. **Review dependencies** - Does anything block this?
4. **Verify documentation matches reality** - If not, update docs first

### During Implementation

1. **Create progress report** in `progress/YYYY-MM-DD-<task>.md`
2. **Document as you go** - Don't rely on memory
3. **Test both positive AND negative** cases
4. **Update source of truth** when configuration is finalized

### After Completing a Step

1. Mark complete in `VOS_Master_TODO.md`
2. Commit and push:
   ```bash
   cd ~/VOS-Docs
   git add .
   git commit -m "Complete: <task>"
   git push
   ```
3. Verify next step is unblocked

---

## Progress Report Template

Save as: `progress/YYYY-MM-DD-<task-slug>.md`

```markdown
---
date: YYYY-MM-DD
task: "<Task Name>"
status: complete | in-progress | blocked
phase: <Phase Number>
host: <hostname if applicable>
---

## Summary
<One-line description of what was done>

## Packages Installed
| Package | Version | Purpose |
|---------|---------|---------|
| ... | ... | ... |

## Configuration Changes

### /path/to/config.file
```diff
- old line
+ new line
```

## Issues & Solutions
| Issue | Cause | Solution |
|-------|-------|----------|
| ... | ... | ... |

## Verification
- [ ] Positive test: <expected behavior works>
- [ ] Negative test: <unauthorized access blocked>

## Rollback Notes
- Command(s) to revert: `...`
- Config backup location: `...`
- Restore procedure: `...`

## Time Spent
<X hours/minutes>

## Next Steps
- ...
```

---

## Repository Structure

```
VOS-Docs/
├── README.md                              # What this repo is
├── PROJECT_INSTRUCTIONS.md                # This file
├── VOS_Master_TODO.md                     # Execution order
├── VOS_Network_Implementation_Guide.md   # Network spec
├── VOS_Foundation_Implementation_Guide.md # OPNsense + Traefik
│
├── progress/                              # Implementation logs
│   └── YYYY-MM-DD-<task>.md
│
├── vos/                                   # VOS architecture docs
│   ├── README.md
│   ├── architecture.md
│   └── services.md
│
├── homelab/                               # Infrastructure docs
│   ├── hosts/                             # Per-host configs
│   ├── network/                           # Network topology
│   └── identity/                          # Auth/identity
│
├── components/                            # Hardware/software specs
│
└── howto/                                 # Procedures
```

---

## Finalized Decisions (Reference)

| ID | Decision | Choice |
|----|----------|--------|
| D1 | Observability | Full stack + Notifiarr |
| D2 | Secrets | Vaultwarden |
| D3 | Backups | PBS on alexandria |
| D4 | Certificates | Let's Encrypt + Smallstep CA |
| D5 | Vector DB | Qdrant |

---

## Current Phase

**Phase 1: Network Foundation**

See `VOS_Master_TODO.md` for detailed task list.
