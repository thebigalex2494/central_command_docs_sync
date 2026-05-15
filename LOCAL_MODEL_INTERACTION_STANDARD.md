# ⚡ Local Model Interaction Standard (v1.0 SAt)

> **SSoT**: Delegation protocol for local models (DeepSeek Coder v2, Qwen 2.5) within Central Command. Based on the optimization analysis of the `proj-001` migration.

## 🧠 Philosophy: Cognitive Load Reduction

Local models (DeepSeek, Qwen) are executor-tier agents operating in the **Intelligence Plane (WSL)**. They perform best when cognitive load is minimized. They are **Executors**, not Architects. The Orchestrator MUST separate planning, execution, and validation into distinct contracts.

### Core Principles
1. **Atomic Granularity**: One task per prompt. No mixed planning/execution.
2. **Artifact-Driven**: Every phase must produce or update a concrete artifact (Checkpoint, Handoff, SQL file). **No task is complete without a checkpoint or parity artifact.**
3. **Explicit Constraints**: Define memory barriers (DuckDB 4GB) and hardware limits (RTX 3060 12GB) upfront.
4. **Validation-First**: Success criteria must be measurable (Parity = $0.00, Diff check, Test pass).

---

## 🛠 Standard Input Pattern (The "Executor Contract")

Use this structure for every delegation to a local executor:

```text
Objective:
[One sentence only]

Inputs:
[Exact file names, tables, columns, or paths]

Constraints:
[Hardware limits, memory cap, plane restrictions, no-edit boundaries]

Task:
[One atomic action only - e.g., "Translate this pandas merge to DuckDB SQL"]

Expected Output:
[Exact artifact path and format]

Validation:
[Specific metric, diff, or test that must pass - e.g., "Analytical Parity Check"]

Rollback:
[Action to take if the task fails]
```

---

## 🔄 Standardized Workflow (T1-T4 Matrix)

| Phase | Owner | Deliverable |
| :--- | :--- | :--- |
| **1. Intake** | Gemini | Intent classification & Project Context |
| **2. Research** | Perplexity | Blueprint / Current State Analysis |
| **3. Planning** | Codex / Gemini | Task Graph / Sprint Plan |
| **4. Execution**| **DeepSeek / Qwen** | **Code / ETL Implementation** |
| **5. Audit** | Claude | Parity / Logic / Documentation Validation |
| **6. Closure** | Gemini | Handoff Report / Registry Update |

---

## ⚠️ Friction Avoidance (Anti-Patterns)

- ❌ **Do not** ask the local model to "modernize the whole pipeline" at once.
- ❌ **Do not** provide ambiguous column names; use the schema inventory.
- ❌ **Do not** skip the Audit phase; local models may introduce subtle logic drifts.
- ❌ **Do not** mix Intelligence Plane tasks (coordination) with Execution Plane tasks (Windows-native COM/GUI).
- ❌ **Do not** consider a task finished without a verifiable parity artifact.

## 💎 Operational Rule: The "Layer-Below" Principle
Always feed the local model **one layer below the final goal**. 
*Example*: Instead of "Complete Sprint 2", ask for "Execute the SQL parity script and write the results to `sprint_2_checkpoint.md`".

---
*Authorized by Central Command Intelligence Unit — 2026-05-15*
