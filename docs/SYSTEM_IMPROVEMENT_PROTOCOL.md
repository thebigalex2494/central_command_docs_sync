# System Improvement Protocol (SIP)

> **Version**: 1.1 (English Standardization)
> **Status**: Active
> **Triggers**: "improve system", "update architecture", "optimize flow" (also accepts Spanish equivalents)

## Overview
This protocol defines the standardized workflow for evolving the system architecture. It utilizes a **Reflective Multi-Agent** approach where different AIs collaborate via a "Shared Context Blackboard" to evaluate, research, plan, audit, and execute improvements.

---

## Roles and Responsibilities

| Agent | Role (Pattern) | Primary Responsibility | Key Tools |
| :--- | :--- | :--- | :--- |
| **Gemini** | Orchestrator / Planner | Evaluate state, synthesize strategy, and coordinate phases. | `read_file`, `task_boundary` |
| **Perplexity** | Researcher | Search for best practices, updated documentation, and patterns. | `search_web` |
| **DeepSeek** | Auditor / Critic | Validate logic, security, and coherence before execution. | `audit_file` (Deep Logic) |
| **Claude** | Builder (Executor) | Complex technical implementation and code generation. Invoked by Gemini via `claude -p`. | `write_to_file`, `run_command` |
| **Ollama** | Support (Janitor) | Low-risk parallel tasks (docs, unit tests). | `run_local_model` (Future) |

---

## Workflow (The 6-Phase Loop)

### Phase 1: Context Evaluation (Gemini)
**Goal**: Understand current state and define improvement scope.
1.  Analyze `GEMINI.md`, `user_rules`, and active files.
2.  Generate artifact: `context_report.md` (Current vs. Desired State).

### Phase 2: Enrichment (Perplexity)
**Goal**: Import external knowledge to avoid local bias.
1.  Read `context_report.md`.
2.  Research design patterns, libraries, or recent methodologies.
3.  Generate artifact: `research_findings.md` (Executive Summary + Sources).

### Phase 3: Strategy Synthesis (Gemini)
**Goal**: Convert research into an actionable plan.
1.  Integrate `context_report.md` + `research_findings.md`.
2.  Create/Update: `implementation_plan.md` (or `strategy_draft.md`).

### Phase 3.5: Technical Audit (DeepSeek)
**Goal**: Safety "Veto". **CRITICAL**.
1.  Read `implementation_plan.md`.
2.  Evaluate against `GUIDELINES.md` and logical coherence.
3.  **Decision**:
    *   **APPROVED**: Proceed to Phase 4.
    *   **REJECTED**: Return to Phase 3 with feedback (`audit_report.md`).

### Phase 4: Execution (Claude)
**Goal**: "Heavy-Lifting" Implementation.
1.  Gemini invokes Claude via `claude -p "instruction"` with the approved plan.
2.  Claude executes code changes according to the approved plan.
3.  Keep `task.md` updated in real-time.

### Phase 5: Support & Cleanup (Ollama/Gemini)
**Goal**: Peripheral tasks.
1.  Update technical documentation.
2.  Generate unit tests for new code.
3.  Clean up temporary `handoff` files.

---

## Safety Mechanisms (Circuit Breakers)

### "Double Rejection" Rule
If **DeepSeek** rejects the strategy 2 consecutive times:
1.  The protocol **PAUSES**.
2.  A high-priority notification is sent to the user (`notify_user`).
3.  Human intervention is required to break the tie or correct the course.

### Rollback
Each execution phase must start with a `git` status check. If execution fails catastrophically:
1.  Execute `git reset --hard HEAD` (Prior consent/configuration).
2.  Log the incident in `system_failures.log`.

---

## System Artifacts (Blackboard)

All agents read and write to:
`%HOME%\Projects\data\sip\`

1.  `context_report.md`
2.  `research_findings.md`
3.  `implementation_plan.md`
4.  `audit_report.md`
