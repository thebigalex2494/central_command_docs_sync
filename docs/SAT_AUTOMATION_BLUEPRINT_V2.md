# 🏗️ Blueprint: Strategic Automation Tool (SAt v4.1)

> **Core Orchestration Document for Audit and Refactoring.**
> Date: April 26, 2026.
> Status: Operational / Scaling Phase.

## 1. Executive Summary
The **SAt (Strategic Automation Tool)** is a Playwright-based engine engineered to automate the Coursera learning lifecycle. Its primary focus is bypassing third-party portal UI restrictions (e.g., UVM empty state bugs) and resolving assessments via external Knowledge Bases (KB).

---

## 2. Asset Inventory (Current Stack)

### 📂 Automation Core
- **`coursera_sat_executor.js`**: Low-level orchestrator handling navigation logic and bypass parameters.
- **`coursera_login_final.js`**: Session persistence management and enrollment bypass logic.
- **`cc-run.sh`**: Bash entrypoint for CLI task execution.

### 🧠 Knowledge Engine (KB)
- **`solve_sustainable_m1_final_v2.js`**: Proof of Concept (PoC) for Course 4. Contains hardcoded `Question -> Answer` mapping.
- **`knowledge_base/*.json` (Proposed)**: Target directory for decoupled JSON answer keys.

### 📊 Audit & QA Tools
- **`audit_grades_detailed.js`**: Progress scanner generating `all_grades_text.txt`.
- **`extract_course_links.js`**: Discovery tool for extracting task and quiz IDs from new courses.

---

## 3. Operational Workflow: "Bypass & Solve"
1. **Auth Bypass**: Injecting `?authProvider=uvmmx` parameter into the attempt URL to override portal UI errors.
2. **Selector Identification**: DOM scanning for `labels` matching the Knowledge Base text patterns.
3. **Error Handling**: Automatic retry logic triggered by "Didn't Pass" status detection.
4. **Honor Code Automation**: Programmatic execution of the `checkbox -> identity injection -> submit` chain.

---

## 4. Technical Debt & Refactoring Requirements
1. **Data Decoupling**: Current scripts have hardcoded answers. These must migrate to a schema-based `./knowledge_base/` directory.
2. **State Synchronization**: Over-reliance on static `setTimeout` calls. Refactor to use `waitForSelector` and event-driven state transitions.
3. **Generic Orchestration**: Implement a single `master_resolver.js` that accepts `--course-slug` and `--quiz-id` as CLI arguments.

---

## 5. Proposed V2 Architecture (Master System)
- **`sat_master_resolver.js`**: The unified execution engine.
- **`knowledge_base/`**: Structured JSON files indexed by `course-slug.json`.
- **`lib/`**: Shared modules for (a) Auth Bypass, (b) DOM Marking, (c) Honor Signature.

---

## 🚀 Mega-Prompt for Claude Opus

**Context:**
Act as a Senior Software Architect. You are tasked with refactoring a Coursera automation suite (SAt v4.1) located at `/home/sigma/central_command`.

**Mission:**
Analyze `coursera_sat_executor.js`, `solve_sustainable_m1_final_v2.js`, and `audit_grades_detailed.js`. Design a production-grade script named `sat_master_resolver.js` that implements the following:
1.  **JSON Data Loading**: Dynamically load answer keys from `./knowledge_base/`.
2.  **Resilient Synchronization**: Replace all static sleeps with robust `Promises` and `waitForSelector` to handle Coursera's dynamic modals (Start Attempt, Confirm Submit, Feedback View).
3.  **Modular Honor Code**: Standardize the "ERIKA SHANNEL MENDEZ AGUILAR" signature as a reusable module.
4.  **CLI Interface**: Support command-line arguments to target specific courses and assessment IDs.

**Constraints:**
- Do not modify existing PoC files; create a new directory structure for V2.
- Maintain Playwright usage with current session persistence paths.
- Ensure the `?authProvider=uvmmx` bypass is integrated as a default navigation standard.
