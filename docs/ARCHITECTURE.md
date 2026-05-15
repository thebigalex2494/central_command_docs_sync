# Análisis Arquitectónico del Proyecto — Central Command Local (2026-04-29)

## 1. Estado Actual del Proyecto

### Ubicación
- **Anterior (Legacy)**: `I:\Mi unidad\central_command` (Google Drive)
- **Actual (Local)**: `%CC%` (Windows local)
- **Estado**: Movido a local, pero paths aún referencias Google Drive

### Problemas Identificados

#### 1.1 Path Inconsistencies
```
❌ PROBLEMA 1: Variables de Entorno Obsoletas
   - AGENTS.md referencia: %DRIVE% = "I:\Mi unidad"
   - AGENTS.md referencia: %CC% = "I:\Mi unidad\central_command"
   - Realidad: Proyecto está en %CC%

❌ PROBLEMA 2: Symlinks Legacy
   - %HOME%\Projects es symlink hacia %CC%\Projects
   - Pero %HOME% probablemente no resuelve correctamente post-movimiento

❌ PROBLEMA 3: Rutas Hardcoded en Código
   - Scripts probablemente contienen I:\ hardcodeado
   - Fallarán si el drive no está montado

❌ PROBLEMA 4: Configuración Dispersa
   - config/settings.py (si existe) referencia I:\
   - .env no actualizado
   - .device-profiles.json usa I:\ como baseline
```

### Estructura Actual
```
%CC%\
├── .claude/                      ← Config de Claude Agent
├── .central-agent/               ← Config de Central Agent
├── .device-profiles.json         ← Profiles por device (USES I:\)
├── .env                          ← Environment (legacy paths?)
├── AGENTS.md                     ← SSoT (BUT references I:\)
├── CLAUDE.md                     ← Claude instructions
├── GEMINI.md                     ← Gemini instructions
├── Projects/                     ← Project registry & data
│   ├── data/                     ← projects-registry.json, agent-state
│   ├── scripts/                  ← Project scripts
│   └── code/                     ← Project code
├── scripts/                      ← Utilities & central_command
│   ├── central_command/          ← Main package
│   │   ├── local_router/         ← NEW: Local router (tests OK ✓)
│   │   ├── tests/                ← Test suite (63 tests passing ✓)
│   │   └── ...
│   └── ...
├── config/                       ← Configuration files
├── artifacts/                    ← Generated outputs
├── data/                         ← Data storage
├── docs/                         ← Documentation
├── logs/                         ← Logs
└── ...
```

## 2. Arquitectura Propuesta

### 2.1 Nueva Estructura (LOCAL-FIRST)

```
%CC%\  (LOCAL ROOT — CANONICAL)
│
├── .claude/                    ← VS Code Copilot config
│   ├── rules/                  ← Development standards
│   │   ├── development-standards.md
│   │   ├── orchestration-protocol.md
│   │   └── quality-gates.md
│   ├── skills/                 ← Reusable skills
│   └── hooks.json              ← Path guards
│
├── .central-agent/             ← Central Agent config
│   ├── docs/                   ← System documentation
│   ├── templates/              ← Prompt templates
│   └── skills/                 ← Agent skills
│
├── config/                     ← Configuration (LOCAL-FIRST)
│   ├── settings.py             ← [NEW] Centralized settings
│   ├── paths.py                ← [NEW] Path resolution (CANONICAL)
│   ├── agents.yaml             ← Agent registry
│   └── .env.local              ← [NEW] Local environment variables
│
├── scripts/                    ← Executable scripts
│   ├── central_command/        ← Python package (LOCAL root reference)
│   │   ├── __init__.py
│   │   ├── local_router/       ← Local orchestration (ready ✓)
│   │   │   ├── agents.yaml
│   │   │   ├── classifier.py
│   │   │   ├── executor.py
│   │   │   └── registry.py
│   │   ├── providers/          ← Provider implementations
│   │   ├── tests/              ← Test suite (63 passing ✓)
│   │   └── main.py             ← [NEW] Entry point
│   └── [OTHER SCRIPTS]
│
├── projects/                   ← [RENAMED from Projects] Project registry & code
│   ├── registry.json           ← [NEW] Centralized registry
│   ├── data/                   ← Shared data
│   │   ├── sip/                ← System Improvement Protocol
│   │   ├── agent-state/        ← Agent runtime state
│   │   └── logs/               ← Audit logs
│   └── code/                   ← Project source code
│
├── docs/                       ← Central documentation
│   ├── ARCHITECTURE.md         ← [NEW] This document
│   ├── PATH_RESOLUTION.md      ← [NEW] Path mapping guide
│   ├── API.md                  ← API documentation
│   └── [OTHER DOCS]
│
├── artifacts/                  ← Generated outputs (ephemeral)
│   ├── agent_reports/          ← Test/audit reports
│   ├── screenshots/            ← Browser automation
│   └── [TEMPORARY]
│
├── AGENTS.md                   ← [UPDATE] Update path references
├── CLAUDE.md                   ← [UPDATE] Update path references
├── GEMINI.md                   ← [NEW] Gemini-specific instructions
├── pyproject.toml              ← Python project config
└── .device-profiles.json       ← [UPDATE] LOCAL paths, not I:\
```

### 2.2 Path Resolution Strategy (CANONICAL)

**Goal**: Single source of truth for all paths. All scripts use `from config.paths import ...`

**File**: `config/paths.py` (NEW)
```python
"""
Central path resolution for local-first architecture.
CANONICAL SOURCE FOR ALL PATHS — update here, updates everywhere.

Rules:
1. All paths are absolute (no relative ../../..)
2. All paths use pathlib.Path (cross-platform)
3. All paths resolved at RUNTIME from config/settings.py
4. Backward-compatible with legacy I:\ references (if Google Drive mounted)
"""

from pathlib import Path
import os

# Root paths — CANONICAL
PROJECT_ROOT = Path(__file__).parent.parent.resolve()  # %CC%
SCRIPTS_DIR = PROJECT_ROOT / "scripts"
CONFIG_DIR = PROJECT_ROOT / "config"
PROJECTS_DIR = PROJECT_ROOT / "projects"
DOCS_DIR = PROJECT_ROOT / "docs"
ARTIFACTS_DIR = PROJECT_ROOT / "artifacts"

# Subdirectories
LOCAL_ROUTER_DIR = SCRIPTS_DIR / "central_command" / "local_router"
TESTS_DIR = SCRIPTS_DIR / "central_command" / "tests"
DATA_DIR = PROJECTS_DIR / "data"
SIP_DIR = DATA_DIR / "sip"
AGENT_STATE_DIR = DATA_DIR / "agent-state"
LOGS_DIR = DATA_DIR / "logs"

# Legacy (if Google Drive mounted)
try:
    GOOGLE_DRIVE_ROOT = Path("I:\\Mi unidad")
    if GOOGLE_DRIVE_ROOT.exists():
        LEGACY_DRIVE = GOOGLE_DRIVE_ROOT
        LEGACY_CC = GOOGLE_DRIVE_ROOT / "central_command"
    else:
        LEGACY_DRIVE = None
        LEGACY_CC = None
except Exception:
    LEGACY_DRIVE = None
    LEGACY_CC = None

# Validation
assert PROJECT_ROOT.exists(), f"Project root missing: {PROJECT_ROOT}"
assert SCRIPTS_DIR.exists(), f"Scripts dir missing: {SCRIPTS_DIR}"
```

**File**: `config/settings.py` (NEW)
```python
"""
Centralized settings for Central Command local architecture.
Uses config/paths.py for all path resolution.
"""

from pathlib import Path
import os
from . import paths

# Environment
ENVIRONMENT = os.getenv("CC_ENV", "local")  # local, staging, production
DEBUG = ENVIRONMENT == "local"

# Paths (from canonical resolution)
PROJECT_ROOT = paths.PROJECT_ROOT
SCRIPTS_DIR = paths.SCRIPTS_DIR
PROJECTS_DIR = paths.PROJECTS_DIR
TESTS_DIR = paths.TESTS_DIR
DATA_DIR = paths.DATA_DIR

# Agent registry
AGENT_REGISTRY_PATH = paths.CONFIG_DIR / "agents.yaml"
PROJECTS_REGISTRY_PATH = PROJECTS_DIR / "registry.json"
AGENT_STATE_PATH = paths.AGENT_STATE_DIR

# Logging
LOGS_DIR = paths.LOGS_DIR
LOGS_DIR.mkdir(parents=True, exist_ok=True)

# Default values
MAX_RETRIES = 3
CIRCUIT_BREAKER_TIMEOUT = 300  # seconds
OLLAMA_BASE_URL = os.getenv("OLLAMA_BASE_URL", "http://localhost:11434")
```

**File**: `.env.local` (NEW)
```
# Local environment for %CC%

CC_ENV=local
CC_ROOT=%CC%
OLLAMA_BASE_URL=http://localhost:11434

# Optional: Google Drive (if mounted)
GOOGLE_DRIVE_ROOT=I:\Mi unidad

# Python
PYTHONPATH=${CC_ROOT}/scripts
```

### 2.3 Key Changes

#### Phase 1: Path Unification (IMMEDIATE)
- [ ] Create `config/paths.py` (CANONICAL path resolver)
- [ ] Create `config/settings.py` (centralized config)
- [ ] Create `.env.local` (local environment)
- [ ] Update `.device-profiles.json` to use local paths
- [ ] Update AGENTS.md, CLAUDE.md, GEMINI.md to reference config/paths

#### Phase 2: Script Migration (SHORT-TERM)
- [ ] Audit `scripts/**/*.py` for hardcoded `I:\` paths
- [ ] Replace with `from config.paths import PROJECT_ROOT, ...`
- [ ] Update `pyproject.toml` entry points to use config/settings
- [ ] Test all scripts locally

#### Phase 3: Registry Consolidation (SHORT-TERM)
- [ ] Consolidate `Projects/data/projects-registry.json` → `projects/registry.json`
- [ ] Create symbolic link for backward compatibility (if needed)
- [ ] Update all scripts to reference new location

#### Phase 4: Testing & Validation (ONGOING)
- [ ] Run local router tests (63 passing ✓)
- [ ] Run integration tests against scripts
- [ ] Validate path resolution on target devices
- [ ] Test fallback to Google Drive if mounted

## 3. Implementation Checklist

### 3.1 Configuration (Phase 1)

```
[ ] Create config/paths.py
    └─ Canonical path resolution, PROJECT_ROOT = %CC%
    
[ ] Create config/settings.py
    └─ Uses config/paths.py, centralized configuration
    
[ ] Create .env.local
    └─ CC_ROOT, OLLAMA_BASE_URL, GOOGLE_DRIVE_ROOT (optional)
    
[ ] Update .device-profiles.json
    └─ Replace I:\ with %CC%
    
[ ] Update AGENTS.md
    └─ Reference config/paths.py instead of %DRIVE%, %CC%
```

### 3.2 Scripts (Phase 2)

```
[ ] Audit scripts/central_command/*.py for hardcoded paths
    ├─ local_router/ (already using relative imports)
    ├─ providers/ 
    ├─ autonomy/
    └─ [OTHER MODULES]
    
[ ] Replace hardcoded paths with config.paths imports
    
[ ] Test import resolution:
    python -c "from config.paths import PROJECT_ROOT; print(PROJECT_ROOT)"
```

### 3.3 Registry (Phase 3)

```
[ ] Consolidate Projects/data/projects-registry.json → projects/registry.json
    
[ ] Update all references:
    ├─ central_command/project_manager.py
    ├─ Tests
    └─ Agent state
```

### 3.4 Documentation (Phase 4)

```
[ ] Create docs/ARCHITECTURE.md (THIS FILE)
    
[ ] Create docs/PATH_RESOLUTION.md
    └─ Detailed guide to path resolution strategy
    
[ ] Update AGENTS.md with new path variables
    
[ ] Update README with setup instructions
```

## 4. Testing Strategy

### 4.1 Local Validation (First Run)
```bash
# Test path resolution
python -c "from config.paths import *; print(f'PROJECT_ROOT={PROJECT_ROOT}')"

# Test settings import
python -c "from config import settings; print(f'ENV={settings.ENVIRONMENT}')"

# Run existing tests (should all pass)
pytest scripts/central_command/tests/ -v
```

### 4.2 Integration Testing
```bash
# Test local router with new path resolution
pytest scripts/central_command/tests/test_local_router.py -v

# Test orchestration scenarios
pytest scripts/central_command/tests/test_orchestration_scenarios.py -v

# Test on-demand:
python scripts/central_command/orchestration_demo.py --no-ollama
```

### 4.3 Fallback Testing
```bash
# If Google Drive mounted:
# 1. Set GOOGLE_DRIVE_ROOT in .env
# 2. Run tests with legacy path detection
# 3. Verify config/paths.py resolves I:\ correctly
```

## 5. Migration Timeline

| Phase | Task | Timeline | Blocker? |
|-------|------|----------|----------|
| 1 | Create config/paths.py + settings.py | 1h | No |
| 1 | Update .device-profiles.json | 30m | No |
| 1 | Update AGENTS.md references | 30m | No |
| 2 | Audit scripts for hardcoded paths | 2h | No |
| 2 | Replace hardcoded paths with config imports | 3h | No |
| 2 | Test local router (63 tests) | 30m | No |
| 3 | Consolidate projects registry | 1h | No |
| 3 | Update all registry references | 2h | No |
| 4 | Create docs/ARCHITECTURE.md | 1h | No |
| 4 | Create docs/PATH_RESOLUTION.md | 1h | No |
| **TOTAL** | | **~12 hours** | None |

## 6. Success Criteria

✓ All paths are absolute (no ../../..)  
✓ All paths resolve from `config/paths.py` (single source of truth)  
✓ All tests pass locally (63/63 ✓)  
✓ Scripts run without `I:\` drive dependency  
✓ Backward compatible with Google Drive (if mounted)  
✓ Documentation updated (ARCHITECTURE.md, PATH_RESOLUTION.md)  
✓ New projects can be added without hardcoding paths  

## 7. Next Steps

1. **Immediate**: Review this architecture proposal
2. **Approve**: Confirm path strategy (config/paths.py)
3. **Implement**: Create config/paths.py + settings.py
4. **Test**: Run local router tests to validate
5. **Extend**: Audit and migrate scripts
6. **Consolidate**: Unify registry and data directories
7. **Document**: Create ARCHITECTURE.md + PATH_RESOLUTION.md
8. **Validate**: Test on multiple devices (DESKTOP-UFH2S95, QUIMERA, etc.)

