---
name: cc-session
version: 3.0.0
description: >
  Session lifecycle management: pre-session audit and post-session report generation.
  Invoke at session start for context integrity check, and at session end for reporting.
replaces:
  - ~/.gemini/skills/modo-dev/SKILL.md
  - .agents/skills/source-command-session-report/SKILL.md
---

# CC Session Manager (v3.0)

## A) Pre-Session Audit (invoke at START)

### Phase 1: Context Integrity
- Did the agent load `core/context.md` and the correct project skill?
- Is active context consistent with `projects-registry.json`?
- Were unnecessary skills loaded (context window bloat)?

### Phase 2: Execution Quality (A+ Standard)
- Does the implementation follow the project profile standards?
- Were tests executed? Behavioral correctness verified?
- Is the System Improvement Protocol (SIP) being followed?

### Audit Output Template

```markdown
## AUDITORIA CC-SESSION

| Criterio | Calificacion | Observacion |
|---|---|---|
| Carga de Contexto | [A/B/C/F] | [Detalle tecnico] |
| Flujo de Trabajo | [A/B/C/F] | [Cumplimiento de 6 fases] |
| Integridad de Datos | [A/B/C/F] | [Validacion de archivos/registros] |

**Evaluacion de Efectividad:** [Breve parrafo sobre eficiencia de tokens y precision].
**Recomendacion de Ajuste:** [Accion concreta para mejorar la siguiente fase].
```

## B) Post-Session Report (invoke at END or `/session-report`)

### Step 1: Documentation Audit (Mandatory)
- Scan modified files for missing docstrings.
- Rate "AI-Readiness" of the session (A+ Standard).
- Generate missing docs before proceeding.

### Step 2: Identify Changes
```bash
# Git-based (preferred)
git diff --stat
git log --oneline -20

# Filesystem-based (no git)
find . -newer /tmp/session_start_marker -name "*.py" -o -name "*.md" 2>/dev/null
```

### Step 3: Categorize Changes
- **Code**: .py, .js, .ts, .ps1
- **Config**: .json, .yaml, .toml, .env
- **Docs**: .md, README, AGENTS.md
- **Data**: .csv, .xlsx, registros

### Step 4: Generate Report

Save to `artifacts/session_reports/SESSION_{YYYY-MM-DD}_{HH-MM}.md`:

```markdown
# Session Report — {YYYY-MM-DD HH:MM}

## Resumen
{1-3 oraciones describiendo objetivo principal y resultado}

## Cambios Realizados

### Codigo
- `archivo.py` — descripcion del cambio

### Configuracion
- `config.json` — descripcion del cambio

### Documentacion
- `README.md` — descripcion del cambio

## Decisiones Tomadas
1. {Decision} — {Justificacion breve}

## Pendientes / TODOs
- [ ] {Pendiente 1}

## Metricas
- Archivos modificados: N
- Archivos creados: N
- Lineas anadidas/eliminadas: +N / -N
```

### Step 5: Log to DuckDB (Observability)

```python
# Execute after generating report
python %CC%\tools\duckbase_logger.py
# Or direct insert into data/observability.duckdb:
# Tables: session_history (session_id, project_id, resumen, resultado)
#         agent_performance (metrics estimated for the session)
```

Comunicar siempre en **Espanol (MX)**.
