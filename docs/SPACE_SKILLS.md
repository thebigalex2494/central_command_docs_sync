---
name: central-command-space-skills
description: Specialized skills available to the Perplexity Research Agent
version: 4.8
---
# Central Command Space Skills (v4.8 SAt)

This document defines the specialized skills available to the Perplexity Research Agent.

## 1. Skill: Local Research (pplx-local)
- **Objective**: Execute research tasks using the local Playwright scraper to bypass API costs and context limits.
- **Trigger**: When the user asks for "investigación local" or uses the `pplx-local` alias.
- **Protocol**: 
  1. Use `perplexity_browser_scraper.py` via CDP (port 9222).
  2. Pass raw data to Ollama (`qwen2.5-coder:latest`) for synthesis.
  3. Return the "FINAL INTEGRATED SUMMARY".

## 2. Skill: Project Architect (modo-dev)
- **Objective**: Ensure all code follows the Central Command A+ standard.
- **Rules**:
  - Modularity: Files must be < 200 lines.
  - Documentation: Mandatory docstrings and type hints.
  - SSoT: Always refer to `projects-registry.json` before proposing architectural changes.

## 3. Skill: Logic Auditor
- **Objective**: Perform high-fidelity validation of logic and security.
- **Flow**: SIP (Audit -> Identify -> Propose -> Execute -> Validate).
- **Tooling**: Coordinate with Claude Auditor for final quality gates.

## 4. Skill: Data Ingestion (proj-001)
- **Objective**: Process and normalize sales data for the National Sales Pipeline.
- **Standard**: EBITDA v5.3 consistency across all financial reports.
