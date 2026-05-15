# Central Command — Shared Agent Knowledge (v4.7 SAt)

> SSoT for shared knowledge across all agents (Claude Code, Gemini CLI, local models). Agent-specific behavior resides in CLAUDE.md and GEMINI.md respectively.

## Path Convention

Path standardization for portability and consistency:

| Variable | Value | Purpose |
|----------|-------|-----------|
| `%DRIVE%` | `I:\Mi unidad` | **REFERENCE ONLY** — Stable Google Drive mirror (SSoT for releases) |
| `%CC%` | `%CC%` | **PRIMARY WORKSPACE** — Local-First active execution & data |

### Hybrid Execution (WSL ↔ Windows)

Central Command operates on a **Dual-Plane Architecture**: **Development/Automation (Linux/WSL)** and **Execution/GUI (Windows)**.

1.  **Orchestration Host**: WSL (Debian) is the primary environment for agent coordination and CLI interaction.
2.  **Execution Target**: All Windows-native code (Excel, Outlook, win32com) MUST be executed in the Windows host.
3.  **Persistent RPC Bridge (MANDATORY)**: For agents running natively in Windows (like Antigravity), use the **WSL RPC Bridge** (`localhost:2112`) for all Linux-level operations (grep, bash, AST parsing).
    -   **Bridge Client**: `%CC%/tools/wsl_bridge/wsl_rpc_client.py`
    -   **Usage**: Prepend Linux commands through the RPC client instead of one-off `wsl.exe` calls to avoid I/O bottlenecks.
4.  **Legacy Bridge**: Use `%CC%/tools/wsl2win.sh` only for delegating commands from WSL back to Windows.
5.  **Path Translation**: The RPC Bridge performs automatic `C:\` → `/mnt/c/` translation. For other tools, use `wslpath -w`.

### Device Detection

Agents MUST read `.device-profiles.json` (at `%CC%` root) at session start to adapt behavior per device.

```
Detection:  $env:COMPUTERNAME → lookup in .device-profiles.json → profile with capabilities
Profiles:   DESKTOP-UFH2S95 (Server, executor) | QUIMERA (Quimera, supervisor)
Fallback:   DESKTOP-UFH2S95
```

Use profile to determine: Ollama availability, model selection, role (supervisor vs executor), and drive letter for path resolution. See `.device-profiles.json` for full schema.

### Connectivity & Remote Access (SSH & Tailscale)

To maintain a secure and identified connection across devices, Central Command utilizes **Tailscale** as its primary mesh VPN.

1.  **Tailscale SSH**: Enabled for all nodes. Access is governed by `config/dotfiles/tailscale_acl_v1.json`.
2.  **Android Integration (Mosh Standard)**: For all PC <-> Android interactions, **Mosh (Mobile Shell)** is the mandatory standard to ensure roaming resilience and local-echo over Tailscale.
    -   **Standard Command**: `mosh --ssh="ssh -p 8022" <user>@<magicdns>`
    -   **Verified Nodes**: 
        - `kingkong-power` (100.77.209.43) - Termux
        - `realme-c63` (100.69.225.42) - Termux
3.  **Identification**: Devices are identified via their Tailscale IP or MagicDNS name.
4.  **Security**: Handshakes use Tailscale SSH. No manual authorized_keys management is required for internal mesh access.

## Model Expertise Matrix (Unified v4)

> **SSoT**: `scripts\central_command\local_router\agents.yaml`. This table is a quick reference.

| Domain | Recommended | Fallback | Cost |
|:---|:---|:---|:---|
| **Complex Orchestration** | Claude Opus 4.7 (Thinking enabled) | — | Included Max |
| **Architecture / Planning** | Claude Opus 4.7 | groq_reasoner (70B) | Included Max / $0 |
| **Implementation / Coding** | Claude Sonnet 4.6 (Thinking enabled) | Claude Opus 4.7 | Included Max |
| **Review / Documentation** | Claude Haiku 4.5 | Claude Sonnet 4.6 | Included Max |
| **Python ETL / Pandas** | DeepSeek Coder v2 (16B) | Qwen 2.5 Coder | **$0 local** |
| **Logic / Auditing** | DeepSeek Coder v2 | Claude step-by-step | **$0 local** |
| **Research (web)** | Perplexity sonar-pro | Claude/Gemini + search | Included Pro |
| **Research (deep)** | Perplexity sonar-deep | — | Included Pro |
| **Large codebase analysis** | Gemini 2.5 Pro (1M ctx) | Claude + subagents | Included Student |
| **Extensive documentation** | Gemini 2.5 Pro | Claude + subagents | Included Student |
| **Multi-file refactoring** | Gemini 2.5 Pro | Claude + subagents | Included Student |
| **File editing / Filesystem** | Claude Code | — | Included Max |
| **Browser Automation** | Tier 1→2→3→4 | Claude + Playwright MCP | $0→$0→$$→$$ |
| **AppSheet / GAS** | Claude (Playwright MCP) | — | Included Max |
| **HTML / CSS (Email)** | Claude (Sonnet 4.6) | — | Included Max |
| **Classification / Indexing** | Ollama llama3.1 (8B) | Claude Haiku | **$0 local** |
| **Docs / Tests / Low-risk** | Ollama (janitor) | Claude Haiku | **$0 local** |

### Local Agents ($0)

| Agent | Role | Invocation | Model |
|-------|------|------------|-------|
| **tools_coder** | Structured tool-use, JSONL editing | `ollama run qwen3` | qwen3:latest |
| **coder** | Python ETL, pandas, precise coding | `ollama run deepseek-coder-v2` | deepseek-coder-v2:16b |
| **auditor** | Logic audit, security review | `ollama run deepseek-coder-v2` | deepseek-coder-v2:16b |
| **chat** | Fast general chat, classification | `ollama run llama3.1` | llama3.1:8b |

### Routing Rules

> **Implemented**: `scripts\central_command\local_router\` — deterministic-first classification.

4 phases: **Tool fast-path** → **Keyword scoring** (prefer $0 tier) → **LLM brain** (llama3.1, cached) → **Fallback** (DeepSeek Coder v2).

## Orchestration Flow (6 Phases — Refined v5 · T1-T4)

> **Role Assignment (SSoT)**:
> - **Gemini** = Orchestrator (INTAKE + routing + coordination)
> - **Codex** = Planner (task decomposition + subtask graph)
> - **Claude** = Auditor (VALIDATE + AGGREGATE + HANDOFF)
> - **Qwen / DeepSeek** = Executor (implementation + ETL + coding)

| Phase | Name | Description | Owner |
|-------|------|-------------|-------|
| 1 | **INTAKE** | Intent classification, context filtering, mode routing, dynamic skill loading | **Gemini** (Orchestrator) |
| 2 | **PLAN** | Task decomposition, subtask graph, model assignment | **Codex** (Planner) / **groq_reasoner** (Free-tier Fallback) |
| 3 | **EXECUTE** | Parallel execution via local dispatcher (MAX_STEPS=10) | **Qwen 2.5 Coder / DeepSeek Coder v2** (Executor) |
| 4 | **VALIDATE** | Deterministic checks → logic audit → QA | **Claude** (Auditor) |
| 5 | **AGGREGATE** | Synthesis → executive summary → Español MX output | **Claude** (Auditor) |
| 6 | **HANDOFF** | Structured contracts, audit trail, TEP generation | **Claude** (Auditor) + TEP-v1 |

### 6.1 Transferable Execution Package (TEP v1.0)
Standard for moving context between clean AI sessions:
- **Step A**: `/session-report --compact` (Clear turn buffers).
- **Step B**: `/load [Project_ID]` (Invalidate residual memory + load target skills).
- **Step C**: Execute 18-block Mega-Prompt (v4.7 SAt).

### 6.2 Recursive Quality Assurance (RQA v4.7)
Mandatory self-audit before any Mega-Prompt delivery:
- **Block Count**: 18/18 structured XML blocks required.
- **Language**: 100% Technical English for internal logic.
- **Environment**: WSL-Native (T3 Protocol) enforcement.


> Full details: see `.claude/rules/orchestration-protocol.md` and `ORCHESTRATION_REFINED_V4.md`.

### WSL/Win Dual-Plane Enforcement (T3 — MANDATORY)

| Plane | Host | Responsibilities |
|-------|------|-----------------|
| **Intelligence Plane** | WSL (Debian) | Gemini CLI, Codex, Ollama (Qwen/DeepSeek), Claude Code — all AI coordination |
| **Execution Plane** | Windows Host | COM/win32 (Outlook, Excel), Playwright GUI, PowerShell native scripts |

**Rules**:
1. ALL AI agent invocations MUST run in WSL — no exceptions.
2. Windows-native code (COM, win32) MUST cross via `tools/wsl2win.sh` or RPC Bridge (`localhost:2112`).
3. Agents in Windows MUST use `tools/wsl_bridge/wsl_rpc_client.py` for Linux ops.
4. Drive `C:` = primary active workspace. `I:` = reference/release mirror only.

## Dynamic Skill System (Hot-Loading)

To optimize context and scale to dozens of projects, Central Command uses a **Dynamic Skill System**.

1.  **Registry Mapping**: Projects in `projects-registry.json` can define a `"skills"` array.
2.  **Skill Location**: Skills are stored in `%CC%\.cc-intelligence\[namespace]\[skill-name].md`.
3.  **Hot-Loading Logic**: During the **INTAKE** phase, the Orchestrator:
    -   Checks the registry for the active project's skills.
    -   Reads the corresponding files from `.cc-intelligence/`.
    -   Adopts the domain-specific business rules and architectural knowledge into its current context.

### Skill Loading Protocol (v3.0)

**SSoT**: `%CC%\.cc-intelligence\` — All skills live here from v3.0 onward.

| Load Phase | Skills | Trigger |
|-----------|--------|---------|
| Session start (always) | `core/context.md` | Automatic |
| Command invoked | `commands/{command}.md` | User runs `/status`, `/scan`, etc. |
| Domain knowledge needed | `domain/{skill}.md` | On demand |
| Project activated | `projects/{project-id}.md` | Project selected in registry |
| Tool needed | `tools/{tool}.md` | Specific tool invoked |

**Legacy paths** (`.gemini/skills/`, `.agents/skills/`, `.central-agent/skills/`) are archived.
See `.cc-intelligence/INDEX.md` for the full routing table.

This mechanism reduces the need for constant "Discovery-by-Reading" of source files, saving up to 90% of token usage for domain-specific tasks.

## Standard Development Lifecycle (SOP v4.7)

Every work cycle in Central Command must follow this flow to guarantee integrity and AI-readability:

1.  **INTAKE & STATUS**: 
    - Identify the active project.
    - Run `/status` to verify health, blockers, and registry state.
    - Load master context via `/load [id]`.
2.  **PRIORITIZATION & PLAN**:
    - Determine session objective.
    - Generate a detailed `ACTION_PLAN_[ID].md`.
    - Delegate complex planning to **Claude/Gemini** per expertise matrix.
3.  **EXECUTION**:
    - Modular implementation (files < 200 lines).
    - Comments and Type Hints mandatory.
    - Unit/functional tests integrated.
4.  **AUDIT (modo-dev)**:
    - **Mandatory** before closing: activate `modo-dev`.
    - Validate documentation (docstrings, headers, README).
    - Evaluate coherence with Central Command A+ standard.
5.  **REPORT & HANDOFF**:
    - Generate final report via `/session-report`.
    - Update `SESSION_STATUS.md` and project registry if structural changes occurred.

---

### Claude Code & Gemini CLI Invocation (WSL Native)

To ensure consistency and leverage the optimized Linux environment, all AI tools MUST be executed natively in WSL.

1.  **Mandatory Tooling Host**: The WSL Debian environment is the SSoT for all AI tool execution (Claude Code, Gemini CLI, Ollama).
2.  **In Windows Host (Antigravity)**: Use the RPC Bridge to invoke the tools.
    -   **Claude**: `python %CC%/tools/wsl_bridge/wsl_rpc_client.py "claude -p '...'"`.
    -   **Gemini**: `python %CC%/tools/wsl_bridge/wsl_rpc_client.py "gemini --load-context '...'"`.
3.  **In WSL Host**: Call the tools directly (natively installed).
4.  **Fallback (Windows)**: Only use Windows-native versions (`claude.exe`, `gemini.exe`) if the WSL bridge is offline or the task requires direct Windows filesystem hooks that `wslpath` cannot handle.

### Ollama Standard (Unified Models)

Ollama models are centralized in `D:\moved_from_c\ollama\models` and accessed by the WSL daemon via the `/mnt/d/...` junction to ensure a single model store across planes. Use `ollama run` natively in WSL via the bridge.

### Claude Code Invocation (Non-Interactive)

To ensure autonomous delegation, always use:
```bash
claude --print --permission-mode bypassPermissions -p "instruction"
```

## Language & Protocol

1. **User Interaction**: ALWAYS communicate with the user in **Spanish (MX)**.
2. **Internal Processing**: All reasoning in **Technical English** for maximum performance.
3. **Translation**: Silently translate user requests from Spanish → English for processing.
4. **Code**: Comments and variables according to project standards, generally in English.
5. **SIP Artifacts**: In **English** (context_report.md, implementation_plan.md, etc.).
6. **Inter-Agent Interaction**: ALWAYS in **Technical English** (orchestration, delegation, internal logs).
7. **Caveman Mode**: MANDATORY (FULL intensity) for internal reasoning and inter-agent communication to ensure token efficiency.

## Development Standards (Unified v3)

> Full detail: `.claude/rules/development-standards.md` (auto-loaded in Claude Code sessions).

Summary: PEP 8, type hints 3.10+, UTF-8, pathlib, modularity, files <200 lines, tests required.

## SIP + Circuit Breaker

> Full detail: `.claude/rules/quality-gates.md` (auto-loaded in Claude Code sessions).

- **SIP**: Audit → Identify → Propose → Execute → Validate. Blackboard: `%CC%\Projects\data\sip\`
- **Circuit Breaker**: R1 (retry) → R2 (+ error ctx) → R3 (upgrade model) → Fail (escalate human)
- **Fast-fail**: Destructive ops: 2 failures → PAUSE + manual intervention

## Key Paths

```
Registry:       %CC%\Projects\data\projects-registry.json
Agent State:    %CC%\Projects\data\agent-state\
Action Plans:   %CC%\Projects\[PROJECT_ID]\ACTION_PLAN_[ID].md
Status Template:%CC%\Projects\STATUS_TEMPLATE.md
Tools:          %CC%\tools\
Dispatcher:     %CC%\tools\opencode-dispatch.py
Audits:         %CC%\artifacts\agent_reports\
Sessions:       %CC%\data\agent-state\sessions\
Screenshots:    %CC%\data\output\screenshots\
Artifacts:      %CC%\artifacts\
CLAUDE.md:      %CC%\CLAUDE.md
GEMINI.md:      %CC%\GEMINI.md
```

## Documentation & Standards Index

| Context | Document |
|---------|----------|
| **Project Standards** | `.cc-intelligence/domain/project-architect.md` |
| System config | `.cc-intelligence/core/context.md` |
| File classification | `.cc-intelligence/domain/file-classification.md` |
| Project tracking | `.cc-intelligence/domain/project-registry.md` |
| Agent coordination | `.cc-intelligence/domain/agent-coordination.md` |
| Prompt framework | `.cc-intelligence/domain/prompt-refinement.md` |

## Operational Rules

1. **Default Environment**: The orchestration and coordination (Gemini CLI) is performed via Bash (WSL).
2. **Hybrid Execution**: Code that requires Windows dependencies (Excel, Outlook) MUST be executed in the Windows host using `tools/wsl2win.sh` or `python.exe` (Windows binary).
3. **PowerShell Usage**: Restricted exclusively to final acceptance validations or Windows-specific system management.
3. **Documentation First**: Read documentation before starting complex tasks.
4. **Structured Data**: Use JSON for storage; do not rely on agent memory.
5. **Registry Synchronization**: Update the project registry upon detecting structure changes.
6. **Traceability**: Log all file operations and agent delegations.
7. **Safe Operations**: Confirm with the user before moving or deleting files in active projects.
8. **Automated QA**: Validate all changes before delivery using established test suites.
9. **Task Decomposition**: Always decompose complex requests into discrete subtasks.
10. **Executive Summaries**: Provide a technical summary after multi-step tasks.
11. **Survival Mode (T-400)**: When active, prioritize local models and minimize cloud tokens.
12. **Core Maintenance**: Mandatory Weekly FS Hygiene (auto-008) and Monthly DB Compaction (auto-009).
13. **RQA Enforcement**: Never deliver prompts without a silent 18-point structural audit.
