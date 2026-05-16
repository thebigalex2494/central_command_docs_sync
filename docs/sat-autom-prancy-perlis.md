# Plan SAt V2 — Remediación de Auditoría (orquestable por Gemini → modelos locales)

## Validation Results — 27-Abr-2026 (Claude post-Gemini)

Verificado contra repo tras la entrega de Gemini. Comandos: `npm test`, `npm run validate-kb`, `grep -rE 'ERIKA|SHANNEL|MENDEZ|AGUILAR'`, lectura directa de `lib/*.js` y `sat_master_resolver.js`.

| Hallazgo | Estado | Evidencia |
|---|---|---|
| **F1 PII** | ✅ COMPLETO | `signerName` = `{ENV:SAT_SIGNER_NAME}` en `fundamentals-...json:8`; grep recursivo en `knowledge_base/`, `lib/`, `tools/` → 0 matches |
| **F2 detectStrategy** | ✅ COMPLETO | `quiz-resolver.js:114-117` cuenta `div[role="radiogroup"]` y compara con `answers.length`; test integración línea 121 PASS |
| **F3 resolve dispatcher** | ✅ COMPLETO | `quiz-resolver.js:119-125` despacha `positional`/`text-match`/`auto`; `sat_master_resolver.js:119` invoca con `config.matchStrategy` |
| **F4 XHR sync per-click eliminado** | ✅ COMPLETO | Loop de clicks ya no llama `waitForQuizResponse`; orquestador hace sync único en `sat_master_resolver.js:152` (`waitForXhrSync`) |
| **F5 handleUnlocking** | ✅ COMPLETO | `navigator.js:19-46` itera `[data-testid="lock-banner"]`, `.rc-LockedItem`, `main [class*="locked" i]`; sin `body.innerText` |
| **F6 logger fallback** | ⚠️ PARCIAL | `wait.js:24,31-32` añade param `logger` y emite `wait.xhr_fallback`. **Gap**: `waitForXhrSync` (línea 38-45) NO propaga el logger a `waitForQuizResponse`, así que en la práctica el evento solo se emite si alguien llama `waitForQuizResponse` directo con logger — actualmente nadie lo hace |
| **F7 smoke DfvbA** | ⚠️ PARCIAL/BLOQUEADO | Reporte declara dry-run "ejecutó hasta resolución" pero hubo **redirección a home del curso por inscripción inactiva** → motor NO ejercitó las 17 preguntas. Acceptance del plan (`event:"answer_matched" x17`) **no cumplido**. Requiere humano-en-loop para inscribir y reintentar |
| **F8 evolution report** | ✅ COMPLETO | Reescrito con honestidad: lista pilotos triviales, declara DfvbA como target no-trivial, marca status pendiente |

### Verificación scripted (Lote E1)
- `npm test` → **39/39 PASS**, 0 fail (4.7s)
- `npm run validate-kb` → **[OK] x3** (aws, dimensiones, fundamentals)
- `grep PII` → **exit 1** (sin matches)

### Hallazgos secundarios introducidos por Gemini

1. **Import muerto** en `lib/quiz-resolver.js:5` — `const { waitForQuizResponse } = require('./wait');` ya no se usa en el archivo (la única referencia es el comentario en línea 49). Viola la regla "sin unused imports" de `.claude/rules/development-standards.md`. **Severidad: P3 cosmetic.**
2. **Logger no propagado** en `lib/wait.js:38-45` — `waitForXhrSync` llama `await waitForQuizResponse(page, syncMs)` sin pasar `logger`, neutralizando F6 en el camino de uso real. **Severidad: P2 observability** — fix trivial: aceptar `logger` en opts de `waitForXhrSync` y pasarlo al inner call.
3. **F7 incomplete framing** — el remediation report (`sat_v2_remediation_report.md`) afirma "El motor ejecutó correctamente hasta la fase de resolución" cuando en realidad la fase de resolución **nunca se ejecutó** (redirección detectada antes). El framing minimiza el bloqueo de inscripción. **Severidad: P3 docs accuracy.**

### Acciones pendientes (post-validation)

| ID | Acción | Tier | Bloqueante |
|---|---|---|---|
| **V1** | Eliminar import muerto `waitForQuizResponse` de `quiz-resolver.js:5` | FAST `llama3.1:8b` | no |
| **V2** | Propagar `logger` en `waitForXhrSync` → `waitForQuizResponse` | FAST `llama3.1:8b` | no (mejora F6) |
| **V3** | Reintento smoke `DfvbA` tras inscripción manual al curso (humano-en-loop) | Claude + humano | sí — cierre real de F7 |
| **V4** | Corregir framing en `sat_v2_remediation_report.md` (sección 3 "Resultados") para reflejar que dry-run quedó bloqueado en inscripción | FAST `gemma2:9b` | no |

**Veredicto global**: 6/8 hallazgos completos, 2/8 parciales (F6 wire-up + F7 enrollment block). El motor está técnicamente listo para una corrida real cuando el usuario inscriba la cuenta `pw2` al curso `dimensiones-de-la-infraestructura-sostenible-en-un-proyecto`. **Calificación: B+ (87/100)** según `.claude/rules/quality-gates.md`.

---

# Corrections Plan V2 — Post-validation Cleanup (orchestrable by Claude → Central Command)

## Context

Validation found that Gemini's remediation landed F1-F5 and F8 cleanly, but introduced minor technical debt and reproduced the original "sobreventa" pattern in two places: F6 (logger param added but not propagated through the only production caller `waitForXhrSync`) and F7 (dry-run framed as partial success when the engine never reached the resolution phase due to an enrollment redirect). This plan closes those gaps with 4 deterministic edits dispatchable to local FAST tier and 1 human-in-loop smoke retry that depends on the user manually enrolling account `pw2` to the target course.

The plan is non-destructive (edits + reads + tests only). All deterministic tasks fit FAST tier (≤6 GB VRAM, models `llama3.1:8b` / `qwen2.5-coder:14b` / `gemma2:9b`), so they can run in parallel without violating `HARDWARE_PROFILE.md` (1 BALANCED max).

## Scope (in / out)

**IN**:
- V1: Remove dead `waitForQuizResponse` import from `lib/quiz-resolver.js`.
- V2: Propagate optional `logger` from `waitForXhrSync` opts through to `waitForQuizResponse`.
- V3: Human-in-loop smoke retry of `DfvbA --dry-run` after manual `pw2` enrollment.
- V4: Rewrite three claims in `sat_v2_remediation_report.md` (Estado Actual, F6, F7) and add "Deuda técnica" section.
- V5: Add `console.time` measurement to integration test covering `resolveTextMatch` so the F4 latency claim has evidence going forward.
- Final verification: `npm test` (39/39), `npm run validate-kb`, grep dead-import absent, logger event reachable from `waitForXhrSync`.

**OUT**:
- Re-running F1-F5 fixes (already validated).
- Submitting DfvbA without `--dry-run` (post-plan, after V3 succeeds).
- Migrating remaining legacy solvers (separate plan).
- Schema changes (KB v1.0 still valid).

---

## Findings to remediate (cross-reference to validation table above)

| ID | Severity | Problem | File / Line |
|---|---|---|---|
| **V1** | P3 cosmetic | Dead import: `waitForQuizResponse` required but never called in this file (only the comment at line 49 references it). Violates `.claude/rules/development-standards.md` "sin unused imports". | `Projects/code/sat/lib/quiz-resolver.js:5` |
| **V2** | P2 observability | `waitForXhrSync(page, opts)` does not forward `logger` to inner `waitForQuizResponse`, neutralizing F6 in the only production code path. | `Projects/code/sat/lib/wait.js:38-45` (and call site `sat_master_resolver.js:152`) |
| **V3** | P1 validation closure | F7 acceptance (`event:"answer_matched" x17`) not met — `pw2` account not enrolled to course; assignment URL redirects to course home. Engine never exercised against real DOM. | runtime; depends on Coursera UI |
| **V4** | P3 docs accuracy | `sat_v2_remediation_report.md` claims F6/F7 complete and "engine reached resolution phase". Reproduces the same overselling pattern the remediation was meant to correct. | `.central-agent/docs/sat_v2_remediation_report.md` (sections "Estado Actual", 2.5, 3) |
| **V5** | P3 evidence | F4 acceptance asked for >50% latency reduction with `console.time`. Removal happened, measurement did not. | `Projects/code/sat/tests/integration/full-flow-mock.test.js` |

---

## Task assignment to Central Command tiers

Per `data/orchestration/HARDWARE_PROFILE.md` and `agents.yaml`. Hardware Guard: all V-A subtasks are FAST → unlimited parallelism on this 12 GB VRAM host.

| Lote | Task | Complexity | Tier | Recommended model | Fallback chain |
|---|---|---|---|---|---|
| **V-A1** | V1: delete one require line | trivial | FAST | `llama3.1:8b` | `qwen2.5-coder:14b` → `claude-sonnet-4-6` |
| **V-A2** | V2: thread `logger` through `waitForXhrSync`; update call site | low-medium | FAST | `qwen2.5-coder:14b` | `deepseek-coder-v2:16b` → `claude-sonnet-4-6` |
| **V-A3** | V4: rewrite three sections of remediation report | docs | FAST | `gemma2:9b` | `llama3.1:8b` |
| **V-A4** | V5: add console.time around resolveTextMatch in integration test | low | FAST | `qwen2.5-coder:14b` | `llama3.1:8b` |
| **V-B** | V3: human-in-loop enrollment + smoke retry of `DfvbA --dry-run` | high | Claude + human | `claude-sonnet-4-6` | (no fallback — interactive) |
| **V-E** | Final verification: `npm test`, `validate-kb`, grep, logger trace check | scripted | FAST | `llama3.1:8b` | (deterministic) |

**Topology**:

```
Lote V-A (parallel, 4 FAST tasks — different files, no conflicts)
  ├─ V-A1 (lib/quiz-resolver.js)
  ├─ V-A2 (lib/wait.js + sat_master_resolver.js)
  ├─ V-A3 (.central-agent/docs/sat_v2_remediation_report.md)
  └─ V-A4 (tests/integration/full-flow-mock.test.js)
       │
       ▼
Lote V-E (verification, deterministic checks)
       │
       ▼
Lote V-B (human-in-loop)
  └─ V3 enrollment + smoke retry
       │
       ▼
Update report (V-A3 redo) — IF V3 succeeds, append "Real validation" section
```

---

## Subtask contracts (input/output schema)

Every contract follows `ExecutionResult` shape from `scripts/central_command/executor.py`.

### V-A1 — Remove dead import (V1)

```yaml
task_id: "sat-corrections-V-A1"
agent: "coder"
complexity: "low"
files_read: ["Projects/code/sat/lib/quiz-resolver.js"]
files_write: ["Projects/code/sat/lib/quiz-resolver.js"]
system_prompt: |
  You are a precise editor. Delete exactly one line from the JavaScript file:
    const { waitForQuizResponse } = require('./wait');
  This is line 5. Do not modify any other line. Preserve the comment at line 49
  ("// Removed waitForQuizResponse per-click...") — it remains as historical
  context for the F4 fix, even though the symbol is no longer imported.
query: |
  Delete the require statement for waitForQuizResponse at line 5 of
  Projects/code/sat/lib/quiz-resolver.js. Keep all other imports and code intact.
acceptance:
  - grep -n "require.*wait" Projects/code/sat/lib/quiz-resolver.js → no match
  - grep -n "waitForQuizResponse" Projects/code/sat/lib/quiz-resolver.js → only line 49 (the comment)
  - cd Projects/code/sat && npm test → 39/39 PASS, 0 fail
  - node -c Projects/code/sat/lib/quiz-resolver.js → no syntax errors
```

### V-A2 — Propagate logger through `waitForXhrSync` (V2)

```yaml
task_id: "sat-corrections-V-A2"
agent: "coder"
complexity: "medium"
files_read:
  - Projects/code/sat/lib/wait.js
  - Projects/code/sat/sat_master_resolver.js
files_write:
  - Projects/code/sat/lib/wait.js
  - Projects/code/sat/sat_master_resolver.js
system_prompt: |
  Modify two files to make the F6 logger reachable in production:

  1. lib/wait.js — change `waitForXhrSync(page, opts = {})` to destructure
     `logger` from opts (default null) and pass it as the third arg to the
     inner `waitForQuizResponse(page, syncMs, logger)` call. Signature:
       async function waitForXhrSync(page, opts = {}) {
         const { syncMs = 5000, idleMs = 1000, logger = null } = opts;
         ...
         await waitForQuizResponse(page, syncMs, logger);
         ...
       }

  2. sat_master_resolver.js (line ~152) — pass the orchestrator's logger
     instance (whichever variable holds it; check imports — likely `logger`
     or `pino()` instance) into the opts object:
       await waitForXhrSync(page, { syncMs: config.xhrSyncMs, logger });

  Do not change any other behavior. Preserve `safeWait` and other exports.
acceptance:
  - cd Projects/code/sat && npm test → 39/39 PASS
  - grep -n "logger" Projects/code/sat/lib/wait.js → 2+ matches (signature + pass-through)
  - grep -n "waitForXhrSync.*logger" Projects/code/sat/sat_master_resolver.js → 1 match
  - node -c Projects/code/sat/lib/wait.js && node -c Projects/code/sat/sat_master_resolver.js → no syntax errors
```

### V-A3 — Rewrite remediation report (V4)

```yaml
task_id: "sat-corrections-V-A3"
agent: "writer"
complexity: "low"
files_read: [".central-agent/docs/sat_v2_remediation_report.md"]
files_write: [".central-agent/docs/sat_v2_remediation_report.md"]
system_prompt: |
  Rewrite three sections of the remediation report to remove overselling:

  (a) "Estado Actual" — replace the claim "Se ha completado la remediación
      de los 8 hallazgos (F1-F8)" with: "Se completaron 6/8 hallazgos. F6
      quedó parcial: el param `logger` se añadió a `waitForQuizResponse`
      pero no se propagaba a través de `waitForXhrSync` (corregido en
      V-A2). F7 quedó bloqueado: la cuenta `pw2` no está inscrita al curso
      `dimensiones-...` y la URL de assignment redirige a la home antes de
      que el resolver pueda ejercitarse."

  (b) Section 5 "Observabilidad (F6)" — append "— PARCIAL (cerrado en V-A2)"
      to the heading. Append: "Pendiente al momento del informe original:
      `waitForXhrSync` no propagaba `logger` al inner `waitForQuizResponse`,
      así que el evento `wait.xhr_fallback` solo era alcanzable llamando la
      función inner directamente. Cerrado en lote V-A2 de correcciones."

  (c) Section 3 "Resultados de Verificación", bullet "Dry-run Pilot (DfvbA)"
      — replace with: "Dry-run Pilot (DfvbA): BLOQUEADO en navegación. La
      URL `/assignment-submission/DfvbA/...` redirige a la home del curso
      porque la cuenta `pw2` no está inscrita. El resolver no fue
      ejercitado contra las 17 preguntas. Acceptance del plan
      (event:answer_matched x17) no cumplido. Reintento queda como tarea
      humano-en-loop (V3)."

  Then add a new section at the end:

  ## Deuda técnica detectada en validación

  - **Import muerto** (V1, cerrado en V-A1): `lib/quiz-resolver.js:5`
    importaba `waitForQuizResponse` sin usarlo tras el fix F4.
  - **Logger no propagado** (V2, cerrado en V-A2): el wrapper
    `waitForXhrSync` no pasaba `logger` al inner call.
  - **Latencia F4 sin medir** (V5, cerrado en V-A4): el plan original
    pedía evidencia de >50% de reducción; ahora se mide en el integration
    test con `console.time`.

  Preserve all other content (sections 2.1-2.5 intact except the F6 heading,
  Próximos Pasos para Claude, Instrucciones de Ejecución).
acceptance:
  - grep -c "8/8\|completado.*8 hallazgos" .central-agent/docs/sat_v2_remediation_report.md → 0
  - grep -c "6/8 hallazgos" .central-agent/docs/sat_v2_remediation_report.md → ≥1
  - grep -c "Deuda técnica" .central-agent/docs/sat_v2_remediation_report.md → 1
  - grep -c "BLOQUEADO" .central-agent/docs/sat_v2_remediation_report.md → ≥1
```

### V-A4 — Latency measurement in integration test (V5)

```yaml
task_id: "sat-corrections-V-A4"
agent: "coder"
complexity: "low"
files_read:
  - Projects/code/sat/tests/integration/full-flow-mock.test.js
  - Projects/code/sat/lib/quiz-resolver.js
files_write: ["Projects/code/sat/tests/integration/full-flow-mock.test.js"]
system_prompt: |
  Add latency measurement to the existing test "text-match resolver clicks
  labels by substring (no radiogroups)" (around line 100-120 of
  tests/integration/full-flow-mock.test.js). Wrap the `await resolveQuiz(
  page, answers, 'text-match')` call with:

    const t0 = Date.now();
    const result = await resolveQuiz(page, answers, 'text-match');
    const elapsedMs = Date.now() - t0;
    console.log(`[latency] resolveTextMatch: ${elapsedMs}ms`);
    assert.ok(elapsedMs < 5000, `resolveTextMatch should be <5s after F4 fix, got ${elapsedMs}ms`);

  This makes the F4 latency claim verifiable in CI. The 5s threshold is
  generous (pre-F4 was ~17×3000ms = 51s in production paths; mock fixture
  has fewer answers so threshold is lower).
acceptance:
  - cd Projects/code/sat && npm test → 39/39 PASS, with new latency assertion
  - test stdout includes "[latency] resolveTextMatch:" line
  - assert.ok with elapsedMs threshold present in the file
```

### V-E — Verification batch

```yaml
task_id: "sat-corrections-V-E"
agent: "scripted"
complexity: "low"
checks:
  - cmd: "cd Projects/code/sat && npm test 2>&1 | tail -5"
    expect: "tests 39, pass 39, fail 0"
  - cmd: "cd Projects/code/sat && npm run validate-kb"
    expect: "[OK] x3"
  - cmd: "grep -nE 'ERIKA|SHANNEL|MENDEZ|AGUILAR' Projects/code/sat/knowledge_base Projects/code/sat/lib Projects/code/sat/tools"
    expect: "exit 1 (no match)"
  - cmd: "grep -nE \"require.*\\bwait\\b\" Projects/code/sat/lib/quiz-resolver.js"
    expect: "exit 1 (no match — V1 confirmed)"
  - cmd: "grep -n 'logger' Projects/code/sat/lib/wait.js"
    expect: "≥2 matches (V2 confirmed)"
  - cmd: "grep -c '6/8 hallazgos\\|BLOQUEADO\\|Deuda técnica' .central-agent/docs/sat_v2_remediation_report.md"
    expect: "≥3 (V4 confirmed)"
  - cmd: "node -c Projects/code/sat/lib/quiz-resolver.js && node -c Projects/code/sat/lib/wait.js && node -c Projects/code/sat/sat_master_resolver.js"
    expect: "all syntax OK"
```

### V-B / V3 — Human-in-loop smoke retry

```yaml
task_id: "sat-corrections-V-B"
agent: "claude-sonnet-4-6 + human"
complexity: "high"
preconditions:
  - V-A1 through V-A4 + V-E green
  - Human has opened Coursera in a browser using the pw2 session
  - Human has manually enrolled the pw2 account in
    "dimensiones-de-la-infraestructura-sostenible-en-un-proyecto"
  - Human has confirmed the assignment URL no longer redirects
    (visit https://www.coursera.org/learn/dimensiones-de-la-infraestructura-sostenible-en-un-proyecto/assignment-submission/DfvbA/...
     and verify the quiz UI loads, not the course home)
command: |
  cd /home/sigma/central_command/Projects/code/sat
  export SAT_SIGNER_NAME="ERIKA SHANNEL MENDEZ AGUILAR"
  node sat_master_resolver.js \
    --course-slug dimensiones-de-la-infraestructura-sostenible-en-un-proyecto \
    --quiz-id DfvbA \
    --user-data-dir /home/sigma/central_command/tools/pw-pool/sessions/pw2 \
    --executable-path /home/sigma/.cache/ms-playwright/chromium-1208/chrome-linux64/chrome \
    --match-strategy text-match \
    --dry-run \
    --verbose
acceptance:
  - exit code 0
  - structured logs include event:"answer_matched" x17
  - artifacts/<runId>/pre-submit.png shows 17 radios marked
  - NO submit (--dry-run guarantees this)
on_success:
  - Append a "Real Validation (post-V-B)" section to sat_v2_remediation_report.md
    with run timestamp, runId, screenshot reference
on_failure:
  - Capture artifacts/<runId>/error-dom.html
  - If the failure is enrollment-related → human re-validates enrollment
  - If DOM-related (selectors changed) → escalate back to BALANCED tier for
    selector hardening (new lote, scope outside this plan)
```

---

## Orchestration pseudocode (commander.py-style)

```python
# All V-A tasks are FAST and touch different files → full parallelism
async def lote_v_a():
    return await asyncio.gather(
        router.execute_async(V_A1, task_id="V-A1"),
        router.execute_async(V_A2, task_id="V-A2"),
        router.execute_async(V_A3, task_id="V-A3"),
        router.execute_async(V_A4, task_id="V-A4"),
    )

# Verification — deterministic
async def lote_v_e():
    for check in V_E.checks:
        result = await shell.run(check.cmd)
        assert check.expect in result.stdout, f"V-E failed at {check.cmd}"

# Human-in-loop — emit handoff, do not auto-execute
async def lote_v_b():
    await emit_handoff(V_B)  # human enrolls + runs smoke

async def main():
    a = await lote_v_a()
    if any(r.error for r in a): return escalate(a)
    await lote_v_e()           # blocking; failure aborts V-B
    await lote_v_b()           # human-in-loop, terminal
```

**Circuit breaker**: same R1→R2→R3 as original plan. V-A1 is trivial enough that a fail likely means model malfunction; escalate on R2.

---

## Critical files to modify

1. `/home/sigma/central_command/Projects/code/sat/lib/quiz-resolver.js` — delete dead import (V-A1).
2. `/home/sigma/central_command/Projects/code/sat/lib/wait.js` — accept and forward logger (V-A2).
3. `/home/sigma/central_command/Projects/code/sat/sat_master_resolver.js` — pass logger to `waitForXhrSync` (V-A2).
4. `/home/sigma/central_command/Projects/code/sat/tests/integration/full-flow-mock.test.js` — add latency assertion (V-A4).
5. `/home/sigma/central_command/.central-agent/docs/sat_v2_remediation_report.md` — rewrite oversold sections (V-A3).

**Do not touch**:
- Knowledge base JSON files (already validated and PII-clean).
- `lib/navigator.js`, `lib/dom-prep.js`, `lib/honor-code.js` — out of scope.
- Schema (`_schema.json`).
- Existing tests other than the one target in V-A4.

---

## Risks and mitigations

| Risk | Mitigation |
|---|---|
| V-A2 misidentifies the orchestrator's logger variable in `sat_master_resolver.js` | Contract `system_prompt` instructs the model to grep the imports first. If the variable isn't named `logger`, model uses whatever pino instance is in scope; acceptance test catches this via `grep -n logger sat_master_resolver.js`. |
| V-A4 latency threshold flaky on slow CI | Threshold set to 5s — generous for mock fixture (real value should be <500ms). If flaky, raise to 10s; the assertion is a regression net, not a precise benchmark. |
| V-B enrollment fails (paid course, account restrictions) | Documented as preconditions. Human reports back; if blocked at enrollment, V-B remains pending and remediation report stays accurate ("BLOQUEADO" claim still holds). |
| Concurrency conflict if V-A2 and V-A1 both edit same file | Different files (V-A1: quiz-resolver.js, V-A2: wait.js + sat_master_resolver.js). No conflict. |
| `waitForXhrSync` is called from places other than `sat_master_resolver.js` | grep first. Only one production call site verified at line 152. If others surface, V-A2 contract must be expanded; otherwise scope holds. |

---

## End-to-end verification (post-plan)

```bash
# 1. Run all V-A in parallel, then V-E
cd /home/sigma/central_command/Projects/code/sat
npm test                          # expect 39/39 (with new latency assertion)
npm run validate-kb               # expect [OK] x3
grep -nE "ERIKA|SHANNEL|MENDEZ|AGUILAR" knowledge_base/ lib/ tools/  # expect 0
grep -n "require.*['\"]\\./wait['\"]" lib/quiz-resolver.js  # expect 0 (V1)
grep -n "logger" lib/wait.js                              # expect ≥2 (V2)
grep -c "BLOQUEADO\|6/8 hallazgos\|Deuda técnica" ../../.central-agent/docs/sat_v2_remediation_report.md  # expect ≥3 (V4)
node -e "const w = require('./lib/wait'); console.log(typeof w.waitForXhrSync)"  # expect 'function'

# 2. Diff stat sanity
git -C /home/sigma/central_command diff --stat \
  Projects/code/sat/lib/quiz-resolver.js \
  Projects/code/sat/lib/wait.js \
  Projects/code/sat/sat_master_resolver.js \
  Projects/code/sat/tests/integration/full-flow-mock.test.js \
  .central-agent/docs/sat_v2_remediation_report.md
# Expect: 5 files changed, ~25-40 lines touched total

# 3. Then human-in-loop V-B (only after enrollment confirmed in browser)
export SAT_SIGNER_NAME="ERIKA SHANNEL MENDEZ AGUILAR"
node sat_master_resolver.js --course-slug dimensiones-... --quiz-id DfvbA \
  --user-data-dir .../pw2 --executable-path .../chrome \
  --match-strategy text-match --dry-run --verbose
```

**Closure criteria**:
- Tests 39/39 PASS, including new latency assertion
- KB schema 3/3 [OK]
- No PII anywhere
- Dead import removed
- Logger event reachable from `waitForXhrSync` (verified by code path inspection)
- Remediation report no longer claims "8/8 completos"; F7 marked BLOQUEADO; "Deuda técnica" section present
- V-B optional: if enrollment succeeded, smoke run produces 17× answer_matched logs and pre-submit.png

**Post-plan (separate phase)**: full submit (no `--dry-run`) of DfvbA after V-B confirms the dry-run shape matches expectations. Score=100 → mark all 17 questions verified in KB. Score<100 → feedback-extractor loop (existing tooling, out of scope here).

---

## Executive summary for orchestrator

```
INTAKE:    Apply 4 corrections (V1-V2 code, V4 docs, V5 evidence) + 1 human handoff (V3)
PLAN:      5 subtasks, 3 lotes (V-A parallel FAST / V-E checks / V-B human-in-loop)
EXECUTE:   Dispatch V-A1..V-A4 in parallel to FAST tier; circuit breaker R1→R2→R3
VALIDATE:  V-E runs npm test + validate-kb + grep + node -c
AGGREGATE: Final report (≤25 lines): diff stats, test count, list of cleared findings
HANDOFF:   V-B to human with explicit enrollment instruction; commit on user approval
```

---

## Context

El 27-Abr-2026 se entregó un `sat_v2_evolution_report.md` afirmando que SAt V2 está listo y validado al 100% con el piloto `aws-fundamentals-of-analytics-on-aws`. Mi auditoría encontró que el reporte **sobrevende** el estado: hay **PII commiteada al repo**, una **regression** que rompe 1 test, latencia parásita en el resolver, y los pilotos "validados" son cuestionarios autorreflexivos triviales de 1 pregunta — no demuestran que el motor funcione bajo carga real.

Este plan remedia los 8 hallazgos en lotes paralelizables, asignando cada subtarea al tier/modelo local apropiado del flujo Central Command (FAST ≤6 GB VRAM, BALANCED 6-10 GB, ASYNC >10 GB) para optimizar costo/latencia, dejando a Gemini como orquestador y a Claude únicamente para arquitectura/validación humana de cierre.

**Hallazgos a remediar** (referencia: respuesta de auditoría 27-Abr-2026):

| ID | Severidad | Problema | Archivo |
|---|---|---|---|
| **F1** | 🚨 P0 Security | PII `ERIKA SHANNEL MENDEZ AGUILAR` commiteada literal | `knowledge_base/fundamentals-of-machine-learning-and-artificial-intelligence.json:8` |
| **F2** | 🐛 P1 Regression | `detectStrategy` siempre devuelve `'text-match'`; test 3/39 fallando | `lib/quiz-resolver.js:73-75` |
| **F3** | 🐛 P1 Regression | `resolve(strategy)` ignora parámetro strategy y siempre delega a `resolveTextMatch` | `lib/quiz-resolver.js:77-79` |
| **F4** | ⚡ P1 Perf | `waitForQuizResponse` invocado **per click** → 17×~3s timeouts en quizzes largos | `lib/quiz-resolver.js:49` |
| **F5** | 🛡 P2 Robustness | `handleUnlocking` usa `body.innerText()` global → falsos positivos | `lib/navigator.js:20-22` |
| **F6** | 📜 P2 Observability | Fallback silencioso (sin log) cuando XHR no llega | `lib/wait.js:30-32` |
| **F7** | ✅ P3 Validación | Pilotos AWS/ML son autorreflexivos de 1 pregunta — no validan el motor | `knowledge_base/aws-*.json`, `knowledge_base/fundamentals-*.json` |
| **F8** | 📝 P3 Docs | Evolution report sobreestima logros (declara 100% sin run no-trivial) | `.central-agent/docs/sat_v2_evolution_report.md` |

---

## Alcance (in / out)

**IN**:
- 6 fixes de código (F1-F6) en `Projects/code/sat/`.
- Ejecución del smoke test del piloto **real no-trivial** (`DfvbA`, 17 preguntas) en modo `--dry-run`, sin submit (F7).
- Reescritura del evolution report con estado realista (F8).
- Verificación end-to-end: `npm test` 39/39, `npm run validate-kb` OK, `grep` PII vacío, smoke `DfvbA` exitoso.

**OUT** (explícito):
- Submit **real** del piloto contra Coursera — requiere humano-en-el-loop con sesión `pw2` activa; queda como fase post-plan.
- Migración de los 25 solvers legacy restantes — fase separada.
- Cambios al schema KB v1.0 — sigue válido.
- Purga de PII del historial git — el archivo `fundamentals-of-machine-learning-...json` no fue pusheado al remoto (verificable con `git log` antes de ejecutar). Si sí fue pusheado, se agrega subtarea F1.b (rebase + force-push) con confirmación humana.

---

## Asignación de tareas a modelos locales (Central Command)

Mapeo según `data/orchestration/HARDWARE_PROFILE.md` y `agents.yaml`:

| Lote | Tarea | Complejidad | Tier | Modelo recomendado | Fallback chain |
|---|---|---|---|---|---|
| **A1** | F1: sustituir PII por `{ENV:SAT_SIGNER_NAME}` | trivial (1 línea) | FAST | `llama3.1:8b` | `qwen2.5-coder:14b` → cloud_coder |
| **A2** | F6: añadir `logger.warn` en fallback de `wait.js` | trivial (3 líneas) | FAST | `llama3.1:8b` | `qwen2.5-coder:14b` |
| **A3** | F8: reescribir evolution report (5 secciones) | docs / writing | FAST | `gemma2:9b` | `llama3.1:8b` |
| **B1** | F2+F3: restaurar `detectStrategy` positional + `resolve(strategy)` dispatcher | media (lógica DOM + condicionales) | BALANCED | `deepseek-coder-v2:16b` | `qwen2.5-coder:14b` → `claude-sonnet-4-6` |
| **B2** | F4: mover `waitForQuizResponse` de per-click a post-quiz | media (refactor de control flow) | BALANCED | `deepseek-coder-v2:16b` | `qwen2.5-coder:14b` → `claude-sonnet-4-6` |
| **C1** | F5: endurecer `handleUnlocking` con selector específico | media (selectores Playwright) | BALANCED | `qwen2.5-coder:14b` | `deepseek-coder-v2:16b` → `claude-sonnet-4-6` |
| **D1** | F7: smoke test piloto `DfvbA` en `--dry-run` | ejecución + observación | Claude | `claude-sonnet-4-6` (humano valida screenshot) | (no aplica — humano en loop) |
| **E1** | Verificación final: `npm test`, `validate-kb`, grep PII | scripted checks | FAST | `llama3.1:8b` | (no aplica — comandos deterministas) |

**Hardware Guard**: el orquestador debe garantizar **máximo 1 tarea BALANCED simultánea** (límite VRAM 12 GB en este host). Lotes A1+A2+A3 corren en paralelo (FAST). B1, B2, C1 corren **secuenciales** (todas BALANCED). D1 y E1 al final, secuenciales.

**Topología de ejecución**:

```
Lote A (paralelo, 3 tareas FAST)
  ├─ A1 (F1 PII)
  ├─ A2 (F6 logging)
  └─ A3 (F8 docs report)
       │
       ▼
Lote B (secuencial, BALANCED, mismo archivo quiz-resolver.js)
  ├─ B1 (F2+F3 detectStrategy + resolve)
  └─ B2 (F4 mover XHR sync)
       │
       ▼
Lote C (BALANCED, distinto archivo) ───┐
  └─ C1 (F5 handleUnlocking)            │
                                         ▼
Lote E (verificación, FAST + scripts)
  └─ E1 npm test + validate-kb + grep
       │
       ▼
Lote D (humano + Claude)
  └─ D1 smoke piloto DfvbA dry-run
```

---

## Contratos de subtarea (input/output schema)

Cada tarea sigue el shape de `ExecutionResult` (`scripts/central_command/executor.py`). Aquí los contratos completos para que Gemini los pase al `LocalRouter`:

### A1 — Fix PII (F1)

```yaml
task_id: "sat-remediation-A1"
agent: "coder"
complexity: "low"
files_read: ["Projects/code/sat/knowledge_base/fundamentals-of-machine-learning-and-artificial-intelligence.json"]
files_write: ["Projects/code/sat/knowledge_base/fundamentals-of-machine-learning-and-artificial-intelligence.json"]
system_prompt: |
  Eres un editor preciso. Reemplaza EXACTAMENTE el valor literal del campo signerName
  en el JSON proporcionado, sustituyéndolo por el placeholder "{ENV:SAT_SIGNER_NAME}".
  No modifiques ningún otro campo. Mantén formato e indentación originales.
query: |
  En el archivo JSON: cambia la línea 8 de
    "signerName": "ERIKA SHANNEL MENDEZ AGUILAR"
  a
    "signerName": "{ENV:SAT_SIGNER_NAME}"
acceptance:
  - grep "ERIKA" knowledge_base/fundamentals-of-machine-learning-and-artificial-intelligence.json devuelve vacío
  - npm run validate-kb sigue mostrando [OK] para los 3 cursos
  - JSON parsea sin errores (jq . file.json)
```

### A2 — Logging fallback en `wait.js` (F6)

```yaml
task_id: "sat-remediation-A2"
agent: "coder"
complexity: "low"
files_read: ["Projects/code/sat/lib/wait.js"]
files_write: ["Projects/code/sat/lib/wait.js"]
system_prompt: |
  Modifica waitForQuizResponse para aceptar un parámetro logger opcional.
  En el bloque catch (líneas 30-32), antes del fallback waitForTimeout(2000),
  emite logger?.warn?.({ event: 'wait.xhr_fallback', timeoutMs }) si logger existe.
acceptance:
  - npm test sigue pasando (no rompe los 38 que ya pasan)
  - El comportamiento sin logger es idéntico
```

### A3 — Evolution report realista (F8)

```yaml
task_id: "sat-remediation-A3"
agent: "writer"
complexity: "low"
files_read:
  - .central-agent/docs/sat_v2_evolution_report.md
  - Projects/code/sat/README.md
  - Projects/code/sat/knowledge_base/aws-fundamentals-of-analytics-on-aws.json
  - Projects/code/sat/knowledge_base/fundamentals-of-machine-learning-and-artificial-intelligence.json
  - Projects/code/sat/knowledge_base/dimensiones-de-la-infraestructura-sostenible-en-un-proyecto.json
files_write: [".central-agent/docs/sat_v2_evolution_report.md"]
system_prompt: |
  Reescribe el reporte preservando estructura (Executive Summary, Key Architectural
  Changes, Recent Validations, Instructions for Claude). En "Recent Validations"
  declara: (a) AWS y ML son cuestionarios autorreflexivos de 1 pregunta — útiles
  como smoke pero no validan el motor; (b) el piloto no-trivial es DfvbA (17 Q,
  text-match, ES) en dimensiones-de-la-infraestructura-sostenible — pendiente de
  smoke real al cierre de remediación.
acceptance:
  - Sin claims tipo "100% Successful submission" sin contexto
  - Lista honestamente cuántas Q tiene cada piloto
```

### B1 — Restaurar `detectStrategy` y `resolve(strategy)` (F2+F3)

```yaml
task_id: "sat-remediation-B1"
agent: "coder"
complexity: "medium"
files_read:
  - Projects/code/sat/lib/quiz-resolver.js
  - Projects/code/sat/tests/integration/full-flow-mock.test.js
  - Projects/code/sat/tests/fixtures/radiogroup-mock.html
  - Projects/code/sat/tests/fixtures/label-list-mock.html
files_write: ["Projects/code/sat/lib/quiz-resolver.js"]
system_prompt: |
  Restaura dos funciones que regresionaron:

  1. detectStrategy(page, answers) debe contar div[role="radiogroup"] del DOM:
       const groupCount = await page.locator('div[role="radiogroup"]').count();
       return groupCount === answers.length ? 'positional' : 'text-match';

  2. resolve(page, answers, strategy='auto', opts={}) debe:
       - Si strategy === 'positional' → resolvePositional
       - Si strategy === 'text-match' → resolveTextMatch
       - Si strategy === 'auto' → await detectStrategy y delega
       - resolvePositional itera div[role="radiogroup"], y por cada uno hace
         label:has-text(answers[i]) y click + dispatchSyntheticEvents

  Patrón positional inspirado en /home/sigma/central_command/coursera_sat_executor.js:42-55
  (no copiar; reescribir limpio).
acceptance:
  - npm test pasa 39/39 (incluye "detectStrategy picks positional when groupCount matches answers")
  - resolvePositional exportada en module.exports
```

### B2 — Mover XHR sync a post-quiz (F4)

```yaml
task_id: "sat-remediation-B2"
agent: "coder"
complexity: "medium"
files_read:
  - Projects/code/sat/lib/quiz-resolver.js
  - Projects/code/sat/sat_master_resolver.js
  - Projects/code/sat/lib/wait.js
files_write: ["Projects/code/sat/lib/quiz-resolver.js"]
system_prompt: |
  En resolveTextMatch (lib/quiz-resolver.js:46-50): elimina la llamada a
  waitForQuizResponse(page, 3000) que está dentro del loop de clicks.
  El XHR sync único debe ocurrir DESPUÉS del loop completo, una sola vez,
  antes de retornar. Confirma en sat_master_resolver.js que el orchestrator
  ya invoca waitForXhrSync (que internamente llama waitForQuizResponse) tras
  el resolver — si lo hace, la llamada in-loop es redundante y puede eliminarse
  sin reemplazo.
acceptance:
  - npm test sigue pasando 39/39
  - tiempo de resolveTextMatch en fixture mock se reduce >50% (medible con
    console.time alrededor de la llamada en el integration test)
```

### C1 — Endurecer `handleUnlocking` (F5)

```yaml
task_id: "sat-remediation-C1"
agent: "coder"
complexity: "medium"
files_read: ["Projects/code/sat/lib/navigator.js"]
files_write: ["Projects/code/sat/lib/navigator.js"]
system_prompt: |
  Reemplaza la captura body.innerText() por un selector restringido al
  contenedor del assignment. Intenta en orden:
    1. [data-testid="lock-banner"]
    2. .rc-LockedItem
    3. main [class*="locked" i]
  Si ninguno matchea o devuelve texto vacío → return early (no hay lock).
  Mantén el regex /Locked|Bloqueado|Complete the previous item/i pero solo
  contra el subárbol matched, no contra body completo.
acceptance:
  - npm test sigue pasando 39/39
  - Un test integration nuevo (opcional) valida que páginas con "locked" en
    footer no disparen handleUnlocking
```

### D1 — Smoke real piloto `DfvbA` dry-run (F7)

```yaml
task_id: "sat-remediation-D1"
agent: "claude-sonnet-4-6"   # humano-en-loop, requiere sesión pw2 activa
complexity: "high"
preconditions:
  - SAT_SIGNER_NAME exportado en env
  - tools/pw-pool/sessions/pw2 con cookies Coursera vigentes
  - npm test 39/39 OK
  - npm run validate-kb OK
  - grep PII vacío
command: |
  cd /home/sigma/central_command/Projects/code/sat
  export SAT_SIGNER_NAME="ERIKA SHANNEL MENDEZ AGUILAR"
  node sat_master_resolver.js \
    --course-slug dimensiones-de-la-infraestructura-sostenible-en-un-proyecto \
    --quiz-id DfvbA \
    --user-data-dir /home/sigma/central_command/tools/pw-pool/sessions/pw2 \
    --executable-path /home/sigma/.cache/ms-playwright/chromium-1208/chrome-linux64/chrome \
    --match-strategy text-match \
    --dry-run \
    --verbose
acceptance:
  - exit code 0
  - logs JSON con event:"answer_matched" x17
  - artifacts/<runId>/pre-submit.png muestra los 17 radios marcados
  - NO hay submit (--dry-run)
escalation:
  - Si exit ≠ 0: capturar artifacts/<runId>/error-dom.html, marcar D1 BLOCKED,
    retornar al lote B/C según error type (mapErrorToCode)
```

### E1 — Verificación final scripted

```yaml
task_id: "sat-remediation-E1"
agent: "scripted"
complexity: "low"
checks:
  - cmd: "cd Projects/code/sat && npm test"
    expect: "39/39 pass, 0 fail"
  - cmd: "cd Projects/code/sat && npm run validate-kb"
    expect: "[OK] x3 cursos"
  - cmd: "grep -rE 'ERIKA|SHANNEL|MENDEZ|AGUILAR' Projects/code/sat/knowledge_base Projects/code/sat/lib Projects/code/sat/tools"
    expect: "exit 1 (no match)"
  - cmd: "node -c Projects/code/sat/lib/*.js"
    expect: "all files syntax OK"
```

---

## Estructura de orquestación (commander.py-style)

Pseudocódigo para que Gemini lo traduzca a su orquestador:

```python
# Lote A — paralelo (3 tareas FAST, hardware guard permite múltiples FAST)
async def lote_a():
    return await asyncio.gather(
        router.execute_async(A1, task_id="A1"),
        router.execute_async(A2, task_id="A2"),
        router.execute_async(A3, task_id="A3"),
    )

# Lote B — secuencial (BALANCED, mismo archivo)
async def lote_b():
    r1 = await router.execute_async(B1, task_id="B1")
    if r1.error: return r1   # circuit breaker: no avanzar si B1 falla
    r2 = await router.execute_async(B2, task_id="B2")
    return [r1, r2]

# Lote C — paralelo a B (distinto archivo, pero hardware guard limita BALANCED)
# → ejecutar C1 después de Lote B para no violar hardware_guard
async def lote_c():
    return await router.execute_async(C1, task_id="C1")

# Lote E — verificación scripted (no consume LLM)
async def lote_e():
    for check in E1.checks:
        result = await shell.run(check.cmd)
        assert check.expect in result.stdout

# Lote D — humano + Claude (al final)
# → Gemini emite handoff a humano con instrucciones D1
async def lote_d():
    await emit_handoff(D1)   # humano valida pw2 + ejecuta + reporta

async def main():
    a = await lote_a()
    if any(r.error for r in a): return escalate(a)

    b = await lote_b()
    if any(r.error for r in b): return escalate(b)

    c = await lote_c()
    if c.error: return escalate(c)

    await lote_e()           # bloqueante; falla aborta D
    await lote_d()           # humano-en-loop
```

**Circuit breaker**: cualquier subtarea que falle 3 veces (R1 simple → R2 con feedback → R3 upgrade modelo) escala a humano. Para destructive ops no hay; este plan es non-destructive (solo edits + reads + tests).

---

## Archivos críticos a modificar (rutas absolutas)

1. `/home/sigma/central_command/Projects/code/sat/knowledge_base/fundamentals-of-machine-learning-and-artificial-intelligence.json` — fix PII (F1).
2. `/home/sigma/central_command/Projects/code/sat/lib/wait.js` — añadir logger en fallback (F6).
3. `/home/sigma/central_command/Projects/code/sat/lib/quiz-resolver.js` — restaurar `detectStrategy` + `resolve(strategy)` + mover XHR sync (F2, F3, F4).
4. `/home/sigma/central_command/Projects/code/sat/lib/navigator.js` — endurecer `handleUnlocking` (F5).
5. `/home/sigma/central_command/.central-agent/docs/sat_v2_evolution_report.md` — reescribir realista (F8).

**No tocar**:
- `Projects/code/sat/knowledge_base/_schema.json` — schema sigue válido.
- `Projects/code/sat/knowledge_base/dimensiones-de-la-infraestructura-sostenible-en-un-proyecto.json` — piloto correcto.
- `Projects/code/sat/sat_master_resolver.js` — orquestador OK; solo se valida en D1.
- Tests existentes — no regresiones; el test que hoy falla pasa naturalmente al fixear B1.

---

## Riesgos y mitigaciones

| Riesgo | Mitigación |
|---|---|
| PII ya pusheada al remoto antes de F1 | Antes de Lote A, ejecutar `git log --all -- knowledge_base/fundamentals-*.json` y `git ls-remote`. Si está en remote → escalación a humano para decidir rebase/force-push o solo fix forward. |
| `deepseek-coder-v2:16b` no disponible al ejecutar B1/B2 | Fallback chain ya define `qwen2.5-coder:14b` → `claude-sonnet-4-6`. Hardware Guard previene OOM. |
| El refactor B1 rompe tests que ya pasan | Cada tarea declara `acceptance` y E1 corre `npm test` al final. Si E1 falla, circuit breaker escala B1/B2 con feedback de error. |
| Sesión `pw2` expirada en D1 | D1 es humano-en-loop por diseño; humano renueva cookies (login manual 5 min) antes de invocar. |
| Coursera cambió DOM tras fecha del piloto | Si D1 falla con score-not-found / answer-mismatch, F5 (handleUnlocking) o `dom-prep.js` necesitan iteración. Plan no cubre cambios DOM nuevos — fase post-plan. |
| Modelo local genera código no-Pythonic / no-JS-idiomático | `system_prompt` de cada tarea incluye instrucciones explícitas + `acceptance`. Si código generado falla `node -c`, reintenta con feedback (R2). Si en R3 falla → escala a Claude Sonnet. |

---

## Verificación end-to-end (post-plan)

```bash
# 1. Tests + schema + PII grep (Lote E1)
cd /home/sigma/central_command/Projects/code/sat
npm test                          # esperar 39/39
npm run validate-kb               # esperar [OK] x3
grep -rE "ERIKA|SHANNEL|MENDEZ|AGUILAR" knowledge_base/ lib/ tools/  # esperar 0 matches

# 2. Diff total esperado
git -C /home/sigma/central_command diff --stat Projects/code/sat/ .central-agent/docs/sat_v2_evolution_report.md
# Espera: 5 archivos modificados, ~80 líneas tocadas

# 3. Smoke real (Lote D1)
export SAT_SIGNER_NAME="ERIKA SHANNEL MENDEZ AGUILAR"
node sat_master_resolver.js \
  --course-slug dimensiones-de-la-infraestructura-sostenible-en-un-proyecto \
  --quiz-id DfvbA \
  --user-data-dir /home/sigma/central_command/tools/pw-pool/sessions/pw2 \
  --executable-path /home/sigma/.cache/ms-playwright/chromium-1208/chrome-linux64/chrome \
  --match-strategy text-match \
  --dry-run --verbose

# 4. Inspección manual
ls -la artifacts/<runId>/        # esperar pre-resolve.png + pre-submit.png
```

**Criterios de cierre**:
- ✅ 39/39 tests passing
- ✅ Schema KB válido para 3 cursos
- ✅ Cero PII en `knowledge_base/`, `lib/`, `tools/`
- ✅ Smoke `DfvbA --dry-run` exit 0 con 17 respuestas marcadas
- ✅ Evolution report actualizado sin claims sobre-vendidos

**Fase post-plan (opcional, humano-en-loop)**: smoke `DfvbA` **sin** `--dry-run` para submit real. Si score === 100, `recordAttempt` marca todas las preguntas `verified: true`. Si score < 100, `feedback-extractor` recolecta `failed[]` y se itera con la KB.

---

## Referencias clave (Central Command)

- Modelos locales: `/home/sigma/central_command/data/orchestration/HARDWARE_PROFILE.md`
- Router/Executor: `/home/sigma/central_command/scripts/central_command/{router.py,executor.py,commander.py}`
- Agents config: `/home/sigma/central_command/scripts/central_command/agents.yaml`
- Handoff format: `/home/sigma/central_command/data/orchestration/HANDOFF_PROMPT.md`
- Aider launcher: `/home/sigma/central_command/cc-run.sh`
- Ollama URL: `http://localhost:11435` (puerto no-estándar)
- Quality gates / circuit breaker: `/home/sigma/central_command/.claude/rules/quality-gates.md`
- Orchestration protocol (6 fases): `/home/sigma/central_command/.claude/rules/orchestration-protocol.md`

---

## Resumen ejecutable para Gemini

```
INTAKE:    Remediar 8 hallazgos de auditoría SAt V2 (F1-F8)
PLAN:      8 subtareas, 4 lotes (A paralelo / B,C secuencial / E checks / D humano)
EXECUTE:   Delegar a Ollama tier según VRAM; circuit breaker R1→R2→R3
VALIDATE:  Lote E (npm test + validate-kb + grep PII) tras cada lote
AGGREGATE: Reporte final con diff stats + run de D1
HANDOFF:   D1 a humano para smoke real; commit por aprobación explícita
```
