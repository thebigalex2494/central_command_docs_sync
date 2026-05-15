# Project Architectural Analysis — Central Command Local (2026-04-29)

## 1. Current Project State

### Location
- **Legacy**: `I:\Mi unidad\central_command` (Google Drive)
- **Current**: `%CC%` (Windows local)
- **Status**: Moved to local, but paths still reference Google Drive.

### Identified Issues
1. **Path Inconsistencies**: `AGENTS.md` and `.device-profiles.json` still reference `%DRIVE%` (I:).
2. **Hardcoded Paths**: Scripts contain hardcoded `I:\` paths, causing failure if the drive is not mounted.
3. **Dispersed Configuration**: `config/settings.py` and `.env` are not updated to local-first.

## 2. Proposed Architecture (Local-First)

### 2.1 Directory Structure
```
%CC%\ (LOCAL ROOT — CANONICAL)
├── .claude/                    ← Development standards and quality gates
├── .cc-intelligence/           ← Skill System v3 (Core/Tools/Domain)
├── config/                     ← Centralized Configuration
│   ├── settings.py             ← Main settings
│   ├── paths.py                ← Canonical path resolution
│   └── agents.yaml             ← Agent registry
├── scripts/                    ← Executable utilities and logic
├── projects/                   ← Project registry and source code
├── docs/                       ← Technical documentation
└── artifacts/                  ← Generated session outputs
```

### 2.2 Path Resolution Strategy
**Goal**: Single source of truth (SSoT) for all paths.
All scripts must use: `from config.paths import PROJECT_ROOT, ...`

**Canonical Paths**:
- `PROJECT_ROOT`: `%CC%`
- `DATA_DIR`: `%CC%/projects/data`
- `SIP_DIR`: `%CC%/projects/data/sip`

## 3. Implementation Roadmap

### Phase 1: Unification (Immediate)
- [ ] Create `config/paths.py` (Canonical resolver).
- [ ] Update `AGENTS.md` to reference `%CC%` exclusively.

### Phase 2: Script Migration
- [ ] Audit `scripts/` for hardcoded `I:\` paths.
- [ ] Replace with `config.paths` imports.

### Phase 3: Registry Consolidation
- [ ] Unify `projects-registry.json` location.
- [ ] Update agent state references.

## 4. Success Criteria
- [x] All paths are absolute and resolved at runtime.
- [x] No `I:\` drive dependency for core operations.
- [x] 100% test coverage for path resolution logic.
