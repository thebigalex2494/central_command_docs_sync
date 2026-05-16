---
name: cc-context
version: 3.0.0
description: >
  Universal CC session initializer. Loads path conventions, device profiles,
  Email Golden Rule, 6-phase flow reference, and Gemini dispatch commands.
  Load at session start for ALL agents.
replaces:
  - ~/.gemini/skills/central-command-orchestrator/SKILL.md
  - ~/.gemini/skills/central-command/SKILL.md
  - .agents/skills/source-command-context/SKILL.md (dispatch section)
---

# CC Context — Universal Session Initializer (v3.0)

## 1. Identity & Path Convention

- **%CC%**: C:\Users\msi\central_command — PRIMARY WORKSPACE. All execution here.
- **%DRIVE%**: I:\Mi unidad\central_command — REFERENCE ONLY. Stable Google Drive mirror.
- **WSL Path**: /mnt/c/Users/msi/central_command

## 2. Device Detection (MANDATORY at session start)

```powershell
# Detect device profile
$hostname = $env:COMPUTERNAME
# Read: %CC%\.device-profiles.json
# Profiles: DESKTOP-UFH2S95 (Server/Executor) | QUIMERA (Supervisor)
# Determines: Ollama availability, model selection, drive letter
```

## 3. Session Initialization Workflow

1. **Read** .device-profiles.json → determine capabilities.
2. **Read** %CC%\Projects\data\projects-registry.json → load project list.
3. **Read** %CC%\AGENTS.md → internalize model matrix and 6-phase flow.
4. **Check** active project → load project skill from projects/ if applicable.
5. **Log** session start in data/observability.duckdb via tools/duckbase_logger.py.

## 4. The Email Golden Rule (CRITICAL)

**NEVER** implement win32com or Dispatch("Outlook.Application") in any project directly.
**ALWAYS** create/use an Adapter in outlook-mailer-hub/adapters/.

## 5. Standard Operational Rules

1. **Language**: Internal reasoning in Technical English. User communication in Spanish (MX).
2. **No Emojis**: All logs, dashboards, and technical documents — emoji-free.
3. **Local-First**: Active workspace is always %CC% on C:. Avoid heavy I/O on %DRIVE%.
4. **Context Loading**: Load only the required project skill after selecting it (ON DEMAND).
5. **Scanning**: When a project is selected, the agent MUST:
   - Read the project's README.md and any SKILL_*.md or project skill.
   - Output a "Key Module Map" (CLI table: FILE | RESPONSIBILITY).
   - Explain execution flow in 3 bullets.

## 6. Model Strategy (Quick Reference)

| Task | Model | Cost |
|------|-------|------|
| Architecture / Planning | Claude Opus 4.7 | Included Max |
| Implementation / Coding | Claude Sonnet 4.6 | Included Max |
| Python ETL / Pandas | DeepSeek Coder v2 (16B) | $0 local |
| Large codebase analysis | Gemini 2.5 Pro (1M ctx) | Included Student |
| Browser Automation | Playwright MCP (Tier 1-4) | $0 → $$ |

> Full matrix: %CC%\AGENTS.md — Model Expertise Matrix (Unified v4)

## 7. Gemini CLI Dispatch (for Antigravity / quota overflow)

```powershell
# Inline task delegation to Gemini CLI
python C:\Users\msi\central_command\tools\session-bridge\gemini_dispatch.py --task "task description"

# Task from file (complex tasks)
python C:\Users\msi\central_command\tools\session-bridge\gemini_dispatch.py --task-file C:\Users\msi\central_command\tasks\my_task.md

# Restore last remote session (post-SSH workflow)
python C:\Users\msi\central_command\tools\session-bridge\session_bridge.py --load gemini latest --output md
```

## 8. Skill Loading Protocol (v3.0)

All skills reside in %CC%\.cc-intelligence\. Do NOT load from legacy paths.
See .cc-intelligence/INDEX.md for the full routing table.

> Canonical reference: %CC%\AGENTS.md
