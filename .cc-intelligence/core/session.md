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
## CC-SESSION AUDIT

| Criterion | Rating | Observation |
|---|---|---|
| Context Load | [A/B/C/F] | [Technical detail] |
| Workflow Flow | [A/B/C/F] | [6-phase compliance] |
| Data Integrity | [A/B/C/F] | [File/registry validation] |

**Effectiveness Evaluation:** [Brief paragraph on token efficiency and precision].
**Adjustment Recommendation:** [Concrete action to improve the next phase].
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
- **Data**: .csv, .xlsx, records

### Step 4: Generate Report

Save to `artifacts/session_reports/SESSION_{YYYY-MM-DD}_{HH-MM}.md`:

```markdown
# Session Report — {YYYY-MM-DD HH:MM}

## Summary
{1-3 sentences describing main objective and outcome}

## Changes Performed

### Code
- `file.py` — change description

### Configuration
- `config.json` — change description

### Documentation
- `README.md` — change description

## Decisions Taken
1. {Decision} — {Brief justification}

## Pending / TODOs
- [ ] {Pending 1}

## Metrics
- Files modified: N
- Files created: N
- Lines added/deleted: +N / -N
```

### Step 5: Log to DuckDB (Observability)

```python
# Execute after generating report
python %CC%\tools\duckbase_logger.py
# Or direct insert into data/observability.duckdb:
# Tables: session_history (session_id, project_id, summary, result)
#         agent_performance (metrics estimated for the session)
```

Always communicate in **Spanish (MX)** with the user.
