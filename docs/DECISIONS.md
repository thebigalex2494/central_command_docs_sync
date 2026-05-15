# central_command — Architectural Decisions

## Decision Log Convention
- **Format**: Date | Decision | Rationale | Impact | Owner
- **Scope**: Architectural changes, policy updates, major conventions
- **Append-only**: Never delete or modify existing decisions
- **Reference**: Use `[DECISION:YYYY-MM-DD-X]` format in other docs

---

## 2026-05-03 | Copilot Context System Foundation

### [DECISION:2026-05-03-01] Context File Structure
**Decision**: Establish mandatory PROJECT_CONTEXT.md + FEATURE_CONTEXT_*.md pattern
**Rationale**: Prevent session-state loss, ensure continuity across model changes and IDE resets
**Impact**: All future sessions must validate context before implementation
**Owner**: Copilot Orchestration Team

### [DECISION:2026-05-03-02] Language Protocol  
**Decision**: Technical English for internal processing, Spanish (MX) for user communication
**Rationale**: Maximize reasoning performance while maintaining user-friendly interaction
**Impact**: Code/comments in English, user summaries in Spanish, artifacts in English
**Owner**: Copilot Orchestration Team

### [DECISION:2026-05-03-03] Model Routing Strategy
**Decision**: DeepSeek-v3.1 as primary reasoning, Claude Haiku as tactical fallback
**Rationale**: Balance high-quality processing with token efficiency and continuity preservation
**Impact**: Structured delegation between expensive reasoning and lightweight execution
**Owner**: Copilot Orchestration Team

### [DECISION:2026-05-03-04] Token Efficiency Rules
**Decision**: Implement strict token conservation policies (minimize context, compress early, externalize memory)
**Rationale**: Prevent quota exhaustion and maintain session continuity under constraints
**Impact**: Required behavior for all Copilot interactions
**Owner**: Copilot Orchestration Team

### [DECISION:2026-05-03-05] 5-Phase Session Protocol
**Decision**: Mandatory Context Validation → Planning → Scoped Execution → Validation → Checkpoint
**Rationale**: Structured approach to prevent drift, ensure quality, and maintain traceability
**Impact**: All sessions must follow this sequence with documented checkpoints
**Owner**: Copilot Orchestration Team

### [DECISION:2026-05-03-06] Phase 2 OpenCode Activation
**Decision**: Activate OpenCode as the primary autonomous execution layer for `code_execution` + `single_repo` tasks, using `qwen3-tools` as the sole validated model.
**Rationale**: OpenCode provides a native tool-use loop (read/edit/grep/bash) that direct Ollama API calls lack, improving autonomous implementation success in local environments.
**Impact**: Gemini Phase 3 orchestration now delegates repo-scoped tasks to OpenCode via `opencode-dispatch.py`.
**Owner**: Architecture Team

---

## Previous Decisions (Pre-2026-05-03)
*Note: These represent existing architectural decisions discovered during context establishment*

### [DECISION:LEGACY-01] 6-Phase Orchestration Protocol
**Decision**: INTAKE → PLAN → EXECUTE → VALIDATE → AGGREGATE → HANDOFF
**Rationale**: Structured multi-agent workflow with clear phase ownership and quality gates
**Impact**: Foundation of central_command orchestration system
**Owner**: Architecture Team

### [DECISION:LEGACY-02] Agent Matrix Specialization  
**Decision**: Specialized agents (coder, auditor, architect, implementer, reviewer, researcher, analyzer)
**Rationale**: Optimal task distribution based on model capabilities and cost efficiency
**Impact**: Clear routing rules and fallback chains between agents
**Owner**: Architecture Team

### [DECISION:LEGACY-03] Local Router Implementation
**Decision**: Deterministic routing with health checks: Tool fast-path → Keyword scoring → LLM brain → Fallback
**Rationale**: Reliability and performance optimization for agent coordination
**Impact**: 3 retries, fast-fail=2, health check 60s, metrics enabled
**Owner**: Local Router Team

### [DECISION:LEGACY-04] Quality Grading System
**Decision**: A+ (95-100) to F (<50) scale with circuit breaker protocol
**Rationale**: Objective quality measurement and failure handling
**Impact**: R1 (retry) → R2 (+error ctx) → R3 (upgrade) → Fail (escalate)
**Owner**: Quality Team

### [DECISION:LEGACY-05] Device Profile Detection
**Decision**: `$env:COMPUTERNAME` → lookup in `.device-profiles.json` for capability adaptation
**Rationale**: Environment-aware behavior based on hardware and role capabilities
**Impact**: Different behavior for DESKTOP-UFH2S95 (executor) vs QUIMERA (supervisor)
**Owner**: System Configuration Team

---

## Decision Review Process

### When to Add a Decision
- Architectural pattern changes
- New technology adoption  
- Policy or convention updates
- Major workflow modifications
- Cross-cutting concerns affecting multiple components

### Decision Format Compliance
```
[DECISION:YYYY-MM-DD-X] Brief title
Date: YYYY-MM-DD
Decision: Clear statement of what was decided
Rationale: Why this decision was made
Impact: What areas/components are affected
Owner: Who is responsible for this decision
```

### Review Cycle
- Monthly review of recent decisions
- Quarterly architectural alignment check
- Annual comprehensive review and pruning (mark obsolete, never delete)

---
*This log is append-only. Record decisions here to maintain architectural consistency and traceability.*

---

## [2026-05-03] Phase 2 OpenCode Activation

**Decision:**
- Designate `qwen3-tools` as the sole validated model for OpenCode tool usage.
- Disqualify all `qwen2.5-coder` variants from OpenCode tool-use paths (generate plain-text tool responses, not structured JSONL).
- Deprecate `ollama-master-orchestrator` agent to eliminate Phase 3 routing conflicts.
- Map Phase 3 `code_execution + single_repo` tasks directly to OpenCode via `opencode-dispatch.py`.

**Rationale:**
- `qwen2.5-coder` variants (7B, 14B, 32B) were disqualified after validation testing: they output tool calls as plain-text JSON strings instead of executing them as structured JSONL `tool_use` events.
- `qwen3-tools` (FROM qwen3:latest, num_ctx 32768) is the only local model confirmed to produce correct JSONL tool-use behavior.
- Legacy `ollama-master-orchestrator` was conflicting with the new OpenCode dispatch layer for `code_execution` tasks on `single_repo` scope.

**Files affected:**
- `.central-agent/agents/ollama-master-orchestrator.md` (deprecated)
- `.central-agent/agents/opencode-repo-executor.md` (created)
- `tools/opencode.json` (configured)
- `tools/opencode-dispatch.py` (created)
- `tools/modelfiles/qwen3-tools.Modelfile` (created)
- `ORCHESTRATION_REFINED_V4.md` (updated: added `code_execution + single_repo → OpenCode` routing condition)

**Status:** CLOSED

***

## [2026-05-03] VS Code AI Integration Strategy

**Decision:**
- Use GitHub Copilot + Ollama native integration as the primary AI coding interface inside VS Code.
- Keep `central_command` as the global orchestrator (planning, routing, context management) outside the editor.
- Use Continue as a secondary lab / fallback for advanced multi-model experiments and custom context pipelines.
- Reserve Claude/Gemini APIs for high-value planning, auditing, and complex reasoning that exceed local model capabilities.

**Rationale:**
- Copilot's native support for Ollama models (via the model picker, VS Code 1.99+) provides the best UX, inline completions, and chat integration without external tooling.
- Repository-level `copilot-instructions.md` gives consistent guidance aligned with `central_command` principles across all sessions.
- Continue offers flexibility (multiple models, RAG, custom providers) without replacing Copilot as the primary editor experience.
- Cloud models remain optional, preserving privacy and local-first execution for sensitive or proprietary code.

**Notes:**
- The ability for `central_command` to invoke Continue via its local HTTP API is a PLANNED CAPABILITY. Exact endpoint and authentication require validation before implementation.
- `copilot-instructions.md` influences Copilot behavior, but deep context (PROJECT_CONTEXT.md, FEATURE_CONTEXT_*.md) must be explicitly injected into prompts when needed. Copilot does not auto-read all markdown files in the repo.

**Status:** CLOSED

***

## [2026-05-03] Context Anchoring Strategy — Cross-Agent Continuity

**Decision:**
- All project continuity must live in versioned `.md` files, not in ephemeral chat history.
- Every coding session (regardless of agent: Copilot, Claude, Gemini, OpenCode) must start from `PROJECT_CONTEXT.md` and `FEATURE_CONTEXT_<name>.md`.
- Every meaningful session must end with a written checkpoint update to the active `FEATURE_CONTEXT_<name>.md`.
- Architectural and policy decisions are recorded append-only in `DECISIONS.md`.

**Rationale:**
- Repeated context loss was observed when sessions hit token limits in Claude Code and Gemini (chats not persisted correctly).
- Externalizing context to files eliminates dependence on conversational memory across agent and model switches.
- This pattern is consistent with context anchoring methodology (Martin Fowler, 2026) and memory-first conversational architecture patterns.

**Files to maintain:**
- `PROJECT_CONTEXT.md` — stable, updated only on architecture changes.
- `FEATURE_CONTEXT_<name>.md` — one per active feature, updated every session.
- `DECISIONS.md` — append-only, permanent record.

**Status:** CLOSED

***

## [2026-05-03] Experimental BrowserAgent via Playwright MCP

**Decision:**
- Introduce `tools/browser_agent_playwright_mcp.py` as an experimental browser-extraction utility.
- Limit scope to external website extraction and local browser tests.
- Explicitly block LLM chat domains and hosted AI web UIs (e.g., Perplexity, DeepSeek, Claude, ChatGPT).
- Implement a `BROWSER_AGENT_ALLOW_AI_DOMAINS` environment variable as a 'DEVELOPMENT_BYPASS' for testing phases.
- Keep feature outside primary orchestration routing until validated.

**Rationale:**
- Provide a controlled mechanism for browser-based tests and data extraction without compromising the primary orchestration logic or exposing the system to external LLM chat interfaces.
- Use Playwright MCP as a standardized tool-use interface for browser control.

**Files affected:**
- `tools/browser_agent_playwright_mcp.py` (created)
- `.env.example` (created/updated)
- `DECISIONS.md` (updated)
- `PROJECT_CONTEXT.md` (updated)
- `FEATURE_CONTEXT_copilot-orchestration.md` (updated)

**Status:** CLOSED