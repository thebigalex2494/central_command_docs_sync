---
name: cc-handoff
version: 1.0.0
description: >
  Automates the generation of Transferable Execution Packages (TEP) for handing off
  context between AI sessions.
---

# cc-handoff

Use this command when the user wants to "compact", "compress", or "transfer" the current session progress to a new dedicated session for a target project.

## 1. Trigger Syntax
- `/handoff [project_id]`
- "Generate TEP for [project_id]"
- "Compact current session for [project_id]"

## 2. Execution Flow (RQA-v4.8)

1. **Context Extraction**:
   - Analyze the active conversation.
   - Extract pending tasks, current blockers, and validated achievements.
   - Identify the target Project Root and Stack from `projects-registry.json`.

2. **Mega-Prompt Generation**:
   - Construct a full **18-block XML Mega-Prompt** (SAt v4.8).
   - Ensure **R8 (Technical English)** compliance.
   - Ensure **R9 (WSL Paths)** compliance using `${CC_ROOT_WSL}` variables.

3. **TEP Packaging**:
   - Output the final result as a structured Markdown artifact.
   - Include the **Step A/B/C** Cold-Start Protocol.

## 3. The TEP Template

```markdown
# 📦 Transferable Execution Package (TEP-v1)
> **Target Project**: {project_id}
> **Objective**: {short_description}
> **Standard**: SAt v4.8 / T3-WSL

### [STEP A: COMPACT]
Run `/session-report --compact` in the current session to close the turn and clear internal buffers.

### [STEP B: LOAD]
Open a NEW session and run:
`/load {project_id}`
*(This will invalidate residual memory and load project-specific skills).*

### [STEP C: EXECUTE]
Paste and execute the following Mega-Prompt:

[MEGA-PROMPT_XML_HERE]
```

## 4. Quality Guardrails (RQA)
- **MUST** contain exactly 18 XML blocks.
- **MUST** use absolute paths starting with `/mnt/c/Users/msi/central_command/`.
- **MUST** be written in Technical English.
- **FAIL-FAST**: Abort if the target Project ID is not found in the registry.

---
*Canonical Reference: %CC%\AGENTS.md (v3.2)*
