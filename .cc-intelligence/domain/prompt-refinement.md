---
name: Claude Prompt Refinement
description: This skill should be used when creating or refining Mega-Prompts (structured XML instructions) for Claude agents. It ensures adherence to the v4.7 SAt framework, proper XML block ordering, executor anchoring, and the quality checklist.
version: 4.7.0
---

# Claude Prompt Refinement Skill (v4.6 — SAt)

## Overview

Official protocol for transforming abstract ideas into **Structured Execution Mega-Prompts** optimized for Claude (Opus 4.6 / Sonnet 4.6). Based on the v4.6 SAt (Structure-Aware Tasking) framework and the **Recursive Quality Assurance (RQA)** protocol.

**SSoT for Template**: `%CC%\.central-agent\docs\templates\claude_prompt_framework.md`
**MSI Sub-template**: `%CC%\.central-agent\docs\templates\multi_agent_coordination.md`

## When to Activate This Skill

Activate when any of the following are requested:
- "Refine instructions for Claude"
- "Create a mega-prompt"
- "Prepare instructions for an agent"
- "Refine prompt" / "Create mega-prompt"
- Any task involving the construction of structured XML instructions for Claude.

## Refinement Process (4 Phases)

```
         ┌─────────────┐
    ①    │  Inception   │  User provides requirements (audio, text, vague ideas).
         └──────┬───────┘
                │
         ┌──────▼───────┐
    ②    │ Gap Analysis │  Detect missing info:
         │              │  → Project ID (via projects-registry.json)
         │              │  → Absolute filesystem paths
         │              │  → Tech stack / dependencies
         │              │  → Constraints / limits
         │              │  → 2-3 targeted questions if strictly necessary.
         └──────┬───────┘
                │
         ┌──────▼───────┐
    ③    │  XML/Sprint  │  Structure with XML + Sprints + AGENTS.md:
         │  Translation │  → 18 XML blocks in correct sequence.
         │              │  → Guardrails + multi-domain examples.
         │              │  → Platform-agnostic discovery protocol.
         └──────┬───────┘
                │
    ③.₅ ┌──────▼───────┐
         │  Recursive   │  Silent self-audit against 18-point checklist.
         │  QA (RQA)    │  Auto-correction before final delivery.
         └──────┬───────┘
                │
         ┌──────▼───────┐
    ④    │   Delivery   │  Clean Mega-Prompt:
         │  Mega-Prompt │  → No internal HTML comments.
         │              │  → Ready to copy-paste into Claude.
         └──────────────┘
```

## XML Block Sequence (18 Blocks)

| Order | XML Block | Required | Purpose |
|:-----:|-----------|:--------:|---------|
| 1 | `<role_definition>` | ✅ | Specific role and mode (Autonomous/Delegated) |
| 2 | `<project_context>` | ✅ | Project ID, Purpose, and Entry Points |
| 3 | `<discovery_protocol>` | ✅ | Dynamic project mapping via filesystem |
| 4 | `<objectives>` | ✅ | Measurable goals for the current sprint |
| 5 | `<system_standards>` | ✅ | Reference to AGENTS.md + critical guardrails |
| 6 | `<output_format>` | ✅ | Expected delivery format |
| 7 | `<examples>` | ✅ | Few-shot (2+ multi-domain examples) |
| 8 | `<execution_guidelines>` | ✅ | Execution rules and Fail-Fast protocols |
| 9 | `<thinking_guidance>` | ✅ | Adaptive thinking (multi-step reasoning) |
| 9b | `<local_executor_anchor>` | ⚪ | Task state + constraints for Qwen/DeepSeek (if executor == local) |
| 10 | `<context_management>` | ✅ | Long-session persistence + mandatory state write-back |
| 11 | `<multi_agent_coordination>` | ⚪ | MSI Protocol (if applicable) |
| 12 | `<sprint_plan>` | ✅ | Sprints with `sprint_checkpoint_write` criteria |
| 13 | `<investigate_before_answering>` | ✅ | Anti-hallucination guardrail |
| 14 | `<scope_control>` | ✅ | Anti-over-engineering guardrail |
| 15 | `<safety_guardrails>` | ✅ | Circuit Breaker + Security |
| 16 | `<subagent_guidance>` | ✅ | Delegation vs. Direct Action logic |
| 17 | `<handoff_instructions>` | ✅ | Cold-start instructions for new sessions (TEP) |
| 18 | `<initial_instruction>` | ✅ | First action: Discovery and Mapping |

> **Note**: Blocks 13-16 (critical guardrails) are placed at the end for maximum adherence.

## Key Framework Rules

### R1: Structure-Aware Tasking (SAt)
Claude has filesystem access. `<project_context>` must be **minimalist**:
- Only Project ID, Purpose, and absolute entry point path.
- **DO NOT** include long file lists — the agent must discover them dynamically.
- `<discovery_protocol>` forces reading `projects-registry.json` and `AGENTS.md` first.

### R2: Absolute Paths
- **NEVER** use unresolved variables (`%DRIVE%`, `%CC%`) in exported mega-prompts.
- Always resolve to absolute paths: `%CC%\...`.

### R3: Tool Agnosticism
- **NEVER** use `ls -R`, `cat`, `find` or other Unix-specific commands.
- Use: "your filesystem exploration tools" (list_dir, find, grep, tree).

### R4: Mandatory Examples
- **ALWAYS** include 2+ examples from distinct domains relevant to the project.
- Each example must have `<input>` and `<expected_output>` with concrete deliverables.

### R5: Anti-Hallucination
- `<investigate_before_answering>` forces file reading before responding.
- `<initial_instruction>` requires a "Structural Project Map" based on real filesystem state.

### R6: Multi-Agent (Optional)
- Include `<multi_agent_coordination>` only when:
  - Multiple Claude Code instances run in parallel on the same repo.
  - Gemini orchestrates simultaneous tasks via `claude -p`.
  - Sprints have cross-module dependencies.

### R7: Transferable Execution Package (TEP)
To move work to a **Clean Session**, the Mega-Prompt must include a cold-start instruction block:
- **Step A**: Force `/session-report` (compact) in the new session to clear buffers.
- **Step B**: Load project context via ID (`/load proj-XXX`).
- **Step C**: Paste and execute the Mega-Prompt.
- These 3 steps must be explicitly defined in `<handoff_instructions>`.

### R8: Technical English Enforcement
- ALL XML blocks, objectives, sprints, and technical instructions MUST be in **Technical English**.
- Spanish (MX) is reserved EXCLUSIVELY for user-facing summaries.

### R9: WSL Orchestration Enforcement (T3 Protocol)
- Orchestration and coordination logic MUST be designed for the **WSL (Debian)** environment.
- Filesystem operations or tool invocations must assume the **WSL RPC Bridge** (`localhost:2112`) if running on Windows.
- Path translation (`C:\` -> `/mnt/c/`) is mandatory for script targets.

### R10: Recursive Quality Assurance (RQA)
- Before delivery, the agent MUST perform a **Silent Self-Audit** against the 18-point checklist.
- Missing blocks or incorrect sequence results in an automatic Grade F (Critical Integrity Failure).
- The agent must self-correct before presenting the final result.

### R11: Local Executor Anchoring (Block 9b)
- For local models (Qwen, DeepSeek), the prompt MUST inject mid-task re-grounding instructions.
- Force the model to state the current Task ID and Active Constraints before every major code generation.

### R12: Intra-session Sprint Checkpoints
- `<sprint_plan>` must include a mandatory instruction to write `sprint_N_state.md` at the end of each sprint.
- This creates an external memory anchor that prevents semantic drift across multiple implementation turns.

## Quality Checklist (Pre-Delivery)

- [ ] `<role_definition>` with specific (non-generic) role.
- [ ] `<project_context>` with absolute C: paths (no unresolved variables).
- [ ] `<discovery_protocol>` with platform-agnostic instructions.
- [ ] `<objectives>` with verbs + measurable outcomes.
- [ ] `<system_standards>` aligned with AGENTS.md.
- [ ] `<output_format>` specifying the expected format.
- [ ] `<examples>` with 2+ multi-domain examples.
- [ ] `<execution_guidelines>` with Fail-Fast and anti-hardcode rules.
- [ ] `<thinking_guidance>` (adaptive thinking).
- [ ] `<context_management>` for long sessions.
- [ ] Sprint plan with `<validation_criteria>` per sprint.
- [ ] `<investigate_before_answering>` (anti-hallucination).
- [ ] `<scope_control>` (anti-over-engineering).
- [ ] `<safety_guardrails>` with Circuit Breaker.
- [ ] `<subagent_guidance>` (anti-subagent-spam).
- [ ] `<handoff_instructions>` with Step A/B/C protocol for clean sessions.
- [ ] R8/R9/R10 Compliance: Technical English + WSL + Recursive Audit.
- [ ] **BLOCK COUNT VALIDATOR**: 18/18 blocks present in correct sequence.
- [ ] `<initial_instruction>` forcing discovery before action.
- [ ] No internal HTML comments in the export.
- [ ] Absolute paths resolved (no `%DRIVE%` or `%CC%`).

## Quick Usage Example

To generate a mega-prompt for `proj-001`:
1. Consult `projects-registry.json` → get path, type, frameworks, description.
2. Fill `<project_context>` with ID=proj-001 and absolute C: path.
3. Adapt `<examples>` to the project's ETL domain.
4. Define sprints based on user objectives.
5. Pass the 18-point RQA checklist.
6. Deliver clean XML to the user.

## References

- [claude_prompt_framework.md](file:///%CC%/.central-agent/docs/templates/claude_prompt_framework.md) — Full exportable template.
- [multi_agent_coordination.md](file:///%CC%/.central-agent/docs/templates/multi_agent_coordination.md) — MSI v1.1 Protocol.
- [AGENTS.md](file:///%CC%/AGENTS.md) — Shared Standards SSoT.
- [Anthropic Best Practices](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/extended-thinking-tips)
