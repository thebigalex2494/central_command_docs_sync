# Orchestration Guide — Local-First Stack ($0)

This document describes the orchestration strategy for local models (Ollama) using Claude Code as the primary orchestration director, following the canonical 6-Phase Flow defined in AGENTS.md (Unified v4).

## 1. Initialization

The local inference server must be running before any orchestration activity begins:

```bash
./cc-init-ai.sh
```

> Mounts Drive D and starts the Ollama daemon on port **11435**.

## 2. Available Local Models (Ollama)

| Model | Ollama Tag | Primary Use Case |
|-------|-----------|-----------------|
| **Qwen 3** | `qwen3:latest` | Structured tool-use, JSONL editing, general-purpose chat |
| **Qwen 2.5 Coder** | `qwen2.5-coder` | Python ETL, pandas, precise coding |
| **DeepSeek Coder v2** | `deepseek-coder-v2:16b` | Logic audit, security review, complex ETL workloads |
| **Llama 3.1** | `llama3.1:8b` | Fast classification, general chat, indexing |

> Model names are canonical per `AGENTS.md § Model Expertise Matrix`. Do not alias or rename them in scripts.

## 3. Orchestration Commands

All local model communication is routed through the `local-ai` alias or `opencode-dispatch.py`.

### A. Logic Audit

```bash
cat file.py | local-ai run deepseek-coder-v2 "Identify logic errors and security issues"
```

### B. Code Generation

```bash
local-ai run qwen2.5-coder "Write a pandas ETL script that..." > script.py
```

### C. Dispatcher (Preferred)

```bash
python tools/opencode-dispatch.py --agent coder --task "implement feature X"
```

## 4. Canonical 6-Phase Flow

| Phase | Name | Description | Owner |
|-------|------|-------------|-------|
| 1 | **INTAKE** | Intent classification, context filtering, mode routing | Orchestrator (Gemini) |
| 2 | **PLAN** | Task decomposition, subtask graph, model assignment | Claude Sonnet 4.6 (Opus 4.7 fallback) |
| 3 | **EXECUTE** | Parallel execution via local dispatcher (MAX_STEPS=10) | Gemini → Local Models (DeepSeek Coder v2 / Qwen 2.5 Coder) |
| 4 | **VALIDATE** | Deterministic checks → DeepSeek Coder v2 audit → QA | DeepSeek Coder v2 + Claude Haiku 4.5 |
| 5 | **AGGREGATE** | Synthesis → executive summary → Spanish MX output | Claude Sonnet 4.6 |
| 6 | **HANDOFF** | Structured contracts, audit trail, update STATUS.md | Claude Haiku 4.5 + contracts.py |

> Full protocol details: `.claude/rules/orchestration-protocol.md`

### Mode Router (Phase 1 — INTAKE)

| Condition | Active Mode |
|-----------|-------------|
| User interacts directly | **Autonomous Mode** — full 6-phase orchestration |
| Task arrives via `claude -p` from Gemini CLI | **Delegated Mode** — focused executor, no decomposition |
| User says "orquesta con Gemini" | **Delegated Mode** |

## 5. Hybrid Workflow (Recommended)

The canonical execution path for maximum throughput and quality:

1. **INTAKE → PLAN** — Claude classifies intent, filters context, and produces the task decomposition graph with model assignments.
2. **EXECUTE** — Claude delegates heavy-generation subtasks to Ollama via `opencode-dispatch.py` (DeepSeek Coder v2 for ETL/audit; Qwen 2.5 Coder for precision scripting).
3. **VALIDATE → AGGREGATE → HANDOFF** — Claude validates Ollama output, applies deterministic QA checks, resolves errors, synthesizes the executive summary, and commits the final artifact.

This pattern minimizes cloud token cost while preserving audit integrity on the orchestrator side.

## 6. Local Agent Roster

| Agent ID | Role | Model | Invocation |
|----------|------|-------|------------|
| `tools_coder` | Structured tool-use, JSONL editing | `qwen3:latest` | `ollama run qwen3` |
| `coder` | Python ETL, pandas, precise coding | `deepseek-coder-v2:16b` | `ollama run deepseek-coder-v2` |
| `auditor` | Logic audit, security review | `deepseek-coder-v2:16b` | `ollama run deepseek-coder-v2` |
| `chat` | Fast classification, indexing | `llama3.1:8b` | `ollama run llama3.1` |

## 7. Circuit Breaker

| Retry | Action |
|-------|--------|
| R1 | Same agent, same prompt |
| R2 | Same agent + error context |
| R3 | Upgrade model (Haiku → Sonnet → Opus) |
| Fail | Escalate to human with full context |

**Fast-fail override**: destructive operations — 2 failures → PAUSE + manual intervention.

---

*SSoT reference: [AGENTS.md](AGENTS.md) · Protocol detail: [.claude/rules/orchestration-protocol.md](.claude/rules/orchestration-protocol.md)*
