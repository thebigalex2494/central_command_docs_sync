# Skill: Project Architect & Health Audit (v1.0)

> Specialized skill for analyzing, standardizing, and maintaining clean project architectures across the Central Command ecosystem.

## Goal
To eliminate "project bloat" and "root garbage" by enforcing a strict structural standard. It uses local models for exhaustive scanning and high-level models for strategic reorganization.

## Project Taxonomy (Profiles)

Before auditing, classify the project into one of these profiles to prevent false positives:

1. **`python-etl`**: Standard Python data pipelines. Requires `src/`, `tests/`, `pyproject.toml` (or `requirements.txt`), and `data/` directories.
2. **`gas-appsheet`**: Google Apps Script / AppSheet projects. Does NOT require `src/`, `tests/`, or `pyproject.toml`. Often contains `.gs` files, `.html`, and `appsheet/` configs.
3. **`automation-win`**: Windows-specific automations (e.g., COM automation, PowerShell). May lack `tests/` but requires `scripts/` and `bin/`.
4. **`data-analysis`**: Notebook-heavy projects. Requires `notebooks/` and `data/`, but might not have a formal `src/`.

## Structural Standards

### 1. Root Directory Rules (The "Clean Root" Policy)
Depending on the project profile, the following files/folders are permitted in a project root:
- **Folders**: `src/` (Python), `data/`, `docs/`, `tests/` (Python), `config/`, `scripts/`, `bin/`, `notebooks/` (Data Analysis).
- **Metadata**: `.git/`, `.gitignore`, `README.md`, `LICENSE`, `pyproject.toml` / `requirements.txt` (Python).
- **Context**: `GEMINI.md`, `PROJECT_CONTEXT.md`, `.central-agent/`.
- **Environment**: `.env.example` (Actual `.env` should be ignored but is allowed).

**Forbidden in Root**:
- Temporary files (`*.tmp`, `*.log`, `*.bak`).
- Ad-hoc scripts (should be in `scripts/`).
- Random data exports (should be in `data/output/`).
- Legacy folders (`old_version/`, `backup_copy/`).
- Tool-specific caches (`.mypy_cache/`, `__pycache__/`, `htmlcov/`) — unless required by the tool and gitignored.

### 2. Mandatory Subdirectories (If applicable to profile)
- `data/`: Subdivided into `input/`, `output/`, `processed/`, `raw/`.
- `docs/`: All architectural diagrams, manuals, and meeting notes.
- `scripts/`: Utility scripts, maintenance tasks, and automation.
- `src/` (or project name folder): The core logic only.

## Workflow: Structural Health Audit

### Phase 1: Local Scan (Ollama - Zero Cost)
Use a local model (e.g., `qwen2.5-coder`) to perform a recursive tree scan and identify anomalies.
- **Command**: `tree /F` or `wsl find . -maxdepth 3`
- **Audit Logic**: Check every file against the "Clean Root" policy and "Mandatory Subdirectories" list.

### Phase 2: Intelligence Processing (Claude/Gemini)
Feed the anomaly list to a high-level model to:
1. Classify the "garbage" (Temporary, Legacy, Misplaced, Redundant).
2. Generate a **Structural Improvement Plan (SIP)**.
3. Map ad-hoc files to their correct destination.

### Phase 3: Execution & Validation
1. Move/Delete files according to the SIP.
2. Update references in code (if paths changed).
3. Validate that the project still runs.

## Operational Directives
- **Rule of Symmetry**: If a project exists in `C:` (active) and `I:` (reference), their structures MUST match, but `I:` must not contain the `data/output/` or `data/raw/` bloat.
- **No Orphan Scripts**: Every script in `scripts/` must have a 1-line comment explaining its purpose.
- **Environment Isolation**: No virtual environments (`venv/`, `.venv/`) inside the project folder if they can be centralized or managed at a higher level (unless project-specific dependencies require it).
