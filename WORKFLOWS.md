# Central Command — Standard Operating Procedures (SOP v4)
**Status**: Canonical | **Target**: AI Agents & Human Operators

This document defines mandatory workflows to ensure every project in the ecosystem maintains the **A+ (AI-Ready)** standard.

---

## 1. Development Lifecycle

All work in Central Command must follow these 6 phases without exception:

| Phase | Name | Mandatory Action | Tool |
| :--- | :--- | :--- | :--- |
| **1** | **INTAKE** | Verify system health and load master context. | `/status`, `/load [id]` |
| **2** | **PLAN** | Decompose tasks and generate execution graph. | `ACTION_PLAN_[ID].md` |
| **3** | **EXECUTE** | Modular, type-annotated implementation. | Claude Code / Gemini |
| **4** | **AUDIT** | Validate logic, security, and **documentation**. | `modo-dev` |
| **5** | **AGGREGATE** | Consolidate changes and update registries. | `projects-registry.json` |
| **6** | **HANDOFF** | Session report and context closure. | `/session-report` |

---

## 2. Documentation Standard (Mandatory)

For a project to be considered **Healthy**, it must satisfy:

1.  **Modules (.py)**:
    - Descriptive docstring header at the top of each file.
    - Type Hints on all function/method signatures.
    - Comments on complex logic blocks.
2.  **Directory Structure**:
    - Every root directory must have an up-to-date `README.md`.
    - Tool directories must include a `SKILL.md` if they are integrable.
3.  **Evolution Log**:
    - Structural changes must be reflected in `projects-registry.json`.
    - Implementation plans (`implementation_plan.md`) must be marked as completed.

---

## 3. Audit Protocol (Quality Gate)

Before closing any task, the agent must activate `modo-dev` to perform the **Quality Sweep**:

- **Is the code AI-Ready?** (Self-descriptive names, modularity < 200 lines per file).
- **Are there context leaks?** (Hardcoded variables, non-standardized relative paths).
- **Is documentation synchronized?** (If logic changed, README was updated).

---

## 4. Resilience Flow (Cross-Agent)

When working across different agents (e.g., SSH from mobile vs. local PC):

1.  **Exit**: Generate `/session-report` when finishing in environment A.
2.  **Entry**: Run `/session-sync` when starting in environment B to recover logical thread.

> [!IMPORTANT]
> **No documentation audit = no session closure.** Readability for future agents is priority #1.
