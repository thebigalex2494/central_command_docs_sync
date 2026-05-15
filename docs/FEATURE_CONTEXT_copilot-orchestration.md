# FEATURE_CONTEXT — copilot-orchestration
# Volatile. Update at every session checkpoint.
# This file is the source of truth for the current state of this feature.

***

## 1. Feature Identity

- **Name:** Copilot Orchestration with DeepSeek via Ollama
- **Status:** IN PROGRESS
- **Owner:** central_command (orchestrator) + GitHub Copilot (VS Code frontend)
- **Goal:** Establish GitHub Copilot (VS Code native extension) + deepseek-v3.1:671b-cloud (via Ollama) as the primary tactical orchestration layer for evolving central_command. Enable consistent, token-efficient, context-persistent workflows regardless of which model or agent is active.

***

## 2. Session Objective (current iteration)

Apply all context files and architectural decisions from the design session to the live central_command repository:

1. Create/update `DECISIONS.md` with the three new closed decisions.
2. Create/update `PROJECT_CONTEXT.md` with the full stable architecture context.
3. Create `.github/copilot-instructions.md` with the finalized Copilot workspace system prompt.
4. Create this `FEATURE_CONTEXT_copilot-orchestration.md` file.
5. Commit all changes with a structured message.

***

## 3. Files in Scope

```
central_command/
├── .github/
│   └── copilot-instructions.md       ← CREATE (Copilot workspace system prompt)
├── DECISIONS.md                       ← CREATE or APPEND (3 new decisions)
├── PROJECT_CONTEXT.md                 ← CREATE (stable architecture context)
└── FEATURE_CONTEXT_copilot-orchestration.md  ← CREATE (this file)
```

***

## 4. Decisions Already Made (for this feature)

- Copilot + Ollama native integration is the primary VS Code AI interface. [CLOSED 2026-05-03]
- central_command remains the global orchestrator. Copilot is a frontend surface only. [CLOSED 2026-05-03]
- DeepSeek v3.1:671b-cloud via Ollama API is the primary Copilot reasoning model. [CLOSED 2026-05-03]
- Claude Haiku (native free) is the fallback model for reduced-scope sessions. [CLOSED 2026-05-03]
- Continue is the secondary lab/fallback. Not a replacement for Copilot. [CLOSED 2026-05-03]
- Context must live in .md files. Chat history is never the source of truth. [CLOSED 2026-05-03]
- copilot-instructions.md does NOT auto-inject PROJECT_CONTEXT. Deep context is manually included in prompts. [CLOSED 2026-05-03]
- Experimental BrowserAgent via Playwright MCP is a local test utility and does not replace the primary orchestration routing. [CLOSED 2026-05-03]

***

## 5. Pending Work

- [ ] Apply all 4 context files to live repo (this session objective).
- [ ] Verify `.github/copilot-instructions.md` is picked up by Copilot Chat in VS Code.
- [ ] Confirm `deepseek-v3.1:671b-cloud` appears in Copilot Chat model picker on BLUEDRAGON.
- [ ] Run a first end-to-end session using the 5-phase protocol (Context → Plan → Execute → Validate → Checkpoint).
- [ ] Validate Continue local HTTP API behavior before enabling central_command → Continue integration.
- [ ] Add `deepseek-v3.1:671b-cloud` to `.device-profiles.json` BLUEDRAGON entry as preferred Copilot model.

***

## 6. Risks and Constraints

- Copilot Free (Claude Haiku base) has strict monthly limits. Sessions must stay narrow and produce checkpoints.
- `deepseek-v3.1:671b-cloud` is a large model via Ollama. Monitor VRAM usage on BLUEDRAGON during Copilot sessions.
- If Ollama model does not appear in Copilot Chat model picker: check VS Code version (requires 1.99+) and Ollama integration settings.
- Do not use Copilot as a planner for large multi-file changes. Escalate to Claude for anything touching >8 files or routing architecture.

***

## 7. Session Checkpoint

**Last updated:** 2026-05-03 (design session via Perplexity)
**Accomplished:**
- Full Copilot workspace system prompt designed and audited.
- DECISIONS.md entries drafted and validated.
- PROJECT_CONTEXT.md drafted and validated.
- This FEATURE_CONTEXT file drafted.

**Next recommended action:**
- Local agent applies all 4 files to repo, commits with message: `feat: add Copilot orchestration context and architecture decisions`
- After commit: open VS Code, verify copilot-instructions.md is active, confirm DeepSeek model in picker.