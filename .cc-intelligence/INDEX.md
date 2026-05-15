---
name: CC Intelligence Index
version: 3.0.0
description: Master index for all CC skills. Replaces the 3-directory skill system.
---

# .cc-intelligence/ — Master Skill Index (v3.0)

> SSoT: %CC%\.cc-intelligence\. Load skills from here only.
> Legacy paths (.gemini/skills/, .agents/skills/, .central-agent/skills/) are archived.

## Routing Table

| Trigger | Skill | Path | Agent Scope |
|---------|-------|------|-------------|
| Session start | context | core/context.md | All agents |
| Session end / report | session | core/session.md | All agents |
| /status command | status | commands/status.md | Claude Code / Gemini |
| /scan or sweep | sweep | commands/sweep.md | Claude Code / Gemini |
| /session-report | session-report | via core/session.md | All agents |
| Gemini dispatch | context (dispatch section) | commands/context.md | Gemini CLI |
| Project audit | project-architect | domain/project-architect.md | All agents |
| Mega-prompt creation | prompt-refinement | domain/prompt-refinement.md | All agents |
| File organization | file-classification | domain/file-classification.md | All agents |
| Registry query | project-registry | domain/project-registry.md | All agents |
| PS script generation | script-templates | domain/script-templates.md | All agents |
| proj-001 (RON pipeline) | ron-processor | projects/ron-processor.md | claude, gemini |
| Browser automation | browser-agent | tools/browser-agent.md | gemini |

## Load Protocol

1. ALL agents load core/context.md at session start.
2. Domain skills load ON DEMAND (not preloaded).
3. Project skills load ONLY when the project is active in projects-registry.json.
4. Skills in tools/ load ONLY when the specific tool is needed.

## Namespace Conventions

- core/* → Always-on system knowledge
- commands/* → Triggered by slash commands
- domain/* → On-demand domain knowledge
- projects/* → Per-project, loaded dynamically
- tools/* → Tool-specific, loaded on use
