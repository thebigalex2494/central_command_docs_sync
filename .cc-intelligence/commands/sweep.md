---
name: "source-command-sweep"
description: "Holistic Project Audit: Data Integrity, Architecture (A+), and Workspace Hygiene (SIP v3.0)"
---

# Universal Project Sweep (SIP v3.0)

This is the unified protocol for holistic system maintenance. It combines data integrity audits, architectural standardization, and workspace hygiene into a single execution flow.

## 1. HYGIENE & DETECTION (Recycler Phase)
Identify and remove "Technical Noise":
- **Junk Files**: .bak, .old, orphaned test scripts, and redundant logs.
- **Sanitization**: Remove temporary artifacts and ensure `.gitignore` compliance.
- **Cognitive Guardrail**: Identify undocumented modules or vague naming.

## 2. DATA INTEGRITY (Gold Sweep Phase)
Audit the project's data lifecycle:
- **Extraction Integrity**: Compare raw sources (XLSX) vs staging (CSV).
- **Persistence Integrity**: Ensure `CSV_COUNT == DUCKDB_COUNT`.
- **Database Standard**: Verify that all persistence is centralized in DuckDB. No local CSVs for final reporting.

## 3. ARCHITECTURAL AUDIT (A+ Standard)
Evaluate against Central Command Gold Standards:
- **Knowledge Anchor**: Verify `SKILL.md` (must have Mermaid diagram and v1.1 standards).
- **AI-Readiness**: Evaluate README and module headers (Purpose, Dependencies, AI-Hint).
- **Code Excellence**: Enforce strict typing (Python 3.10+) and PEP 257 docstrings.
- **Hardcoded Zero**: Detect and eliminate hardcoded paths/credentials.

## 4. RECYCLING & POLISH (Refactor Phase)
- **Refactor**: Simplify complex logic using the "Single Responsibility" principle.
- **Pattern Extraction**: Save successful logic patterns to `config/templates/`.
- **Knowledge Injection**: Update the project's `SKILL.md` with newly optimized patterns.

---

## OUTPUT: Universal Sweep Report

### A) Executive Summary (SIP Grade)
> [!IMPORTANT]
> **SIP Grade**: [A/B/C/D/F] - holistic score.

| Domain | Status | Key Metrics / Findings |
| :--- | :--- | :--- |
| **Hygiene** | [emoji] | [X] files removed / [Y] cleaned |
| **Data** | [emoji] | [Z] records synced (Drift: 0.0%) |
| **Architecture** | [emoji] | [A+ Standard] Mermaid [OK] |
| **Code** | [emoji] | [Type Hints / Docs] |

### B) Drift & Gap Analysis
List any discrepancies found (Data drift, Logic gaps, or Tech Debt).

### C) Remediation & Recycling
- **Fixed**: Actions taken during the sweep.
- **Recycled**: Patterns extracted for the ecosystem.

---

## OPERATIONAL RULES
- Technical English for all internal audits and reports.
- No emojis in technical files (allowed in final user report only).
- ALWAYS use the `modo-dev` audit table after a sweep.
