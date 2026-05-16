# CENTRAL COMMAND — ORCHESTRATION PROMPT (v4.7 SAt)
## Unified Identity & Model Matrix for WSL/Debian Intelligence Plane

***

## 1. INVENTARIO DE MODELOS (SSoT)

| Capa | Ruta | Modelo / ID real | Rol | Main/Fallback | Estado |
|---|---|---|---|---|---|
| Claude Code | Pro CLI | Opus 4.7 (`claude-opus-4-7`) | Arquitectura, tareas complejas, review profunda | **Main** | Validado |
| Claude Code | Pro CLI | Sonnet 4.6 (`claude-sonnet-4-6`) | Trabajo diario, refactor, coordinación | **Main** | Validado |
| Claude Code | Pro CLI | Haiku 4.5 (`claude-haiku-4-5-20251001`) | Tareas rápidas, snippets | **Fallback** | Validado |
| Codex | CLI | gpt-5.5 | Código complejo, research, agentes largos | **Main** | Validado |
| Codex | CLI | gpt-5.4 | Coding diario fuerte | **Main** | Validado |
| Codex | CLI | gpt-5.4-mini | Rápido / barato | **Fallback** | Validado |
| Codex | CLI | gpt-5.3-codex | Coding-optimized | **Main** | Validado |
| Codex | CLI | gpt-5.2 | Professional work / agents | **Main/Fallback** | Validado |
| Gemini CLI | Pro CLI | gemini-3.1-pro-preview | Razonamiento / planning | **Main** | Validado |
| Gemini CLI | Pro CLI | gemini-3-flash-preview | Velocidad con buen balance | **Fallback** | Validado |
| Gemini CLI | Pro CLI | gemini-3.1-flash-lite-preview | Ultra rápido | **Fallback** | Validado |
| Gemini CLI | Pro CLI | gemini-2.5-pro | Pro estable | **Main/Fallback** | Validado |
| Copilot | CLI | Claude Haiku 4.5 | Tareas rápidas | **Fallback** | Validado |
| Copilot | CLI | GPT-5 mini | Tareas rápidas | **Fallback** | Validado |
| Copilot | CLI | GPT-4.1 | Coding general | **Main/Fallback** | Validado |
| Ollama | Local | qwen2.5-coder:32b | Executor principal | **Main** | Vigente |
| Ollama | Local | deepseek-coder-v2:16b | Coder/auditor | **Main** | Vigente |
| Ollama | Local | qwen3:latest | Tool use / structured edits | **Main** | Vigente |
| Ollama | Local | phi4:14b | Planner local | **Main/Fallback** | Vigente |
| OpenRouter | Free cloud | qwen/qwen3-coder:free | Coding / tools | **Main** | Valorado |
| OpenRouter | Free cloud | deepseek/deepseek-r1:free | Reasoning | **Main** | Valorado |
| Groq | Free cloud | llama-3.3-70b-versatile | Ultra-fast coding | **Main** | Valorado |
| Groq | Free cloud | deepseek-r1-distill-70b | Deep reasoning | **Main/Fallback** | Valorado |
| Hybrid | Cloud | qwen3-coder:480b-cloud | High-fidelity coding | **Main** | Vigente |
| Hybrid | Cloud | deepseek-v3.1:671b-cloud | Massive reasoning | **Main** | Vigente |

***

## 2. MODELO DE FLUJO AGÉNTICO

### PRINCIPIO BASE
Todo el control de inteligencia, planificación, orquestación y validación vive en **WSL (Debian)**. 
La ejecución Windows/GUI/COM/nativa queda fuera y solo se activa por contrato de ejecución. 
**No mezclar Intelligence Plane con Execution Plane.**

### REGLAS DE SELECCIÓN

#### 1. PRIORIDAD LOCAL (Ollama)
Usa primero Ollama cuando:
- La tarea sea determinística.
- El contexto sea corto/medio.
- La acción pueda hacerse sin cloud.
- Haya ventaja de costo/privacidad.
**Main**: `qwen2.5-coder:32b`, `deepseek-coder-v2:16b`, `qwen3:latest`.
**Planner**: `phi4:14b`.

#### 2. CLAUDE CODE PRO (Intelligence Node)
Usa Claude Code como nodo principal de arquitectura y coordinación cuando:
- Haya diseño de alto riesgo o refactor complejo.
- Razonamiento profundo o revisión estructural.
**Main**: `Opus 4.7`, `Sonnet 4.6`.
**Research**: Para Perplexity, usar **Sonnet** primero via MCP/Playwright. Escalar a otros modelos Pro solo si es necesario.

#### 3. CODEX (Implementation Node)
Usa Codex para implementación directa, coding pesado y agentes largos.
**Main**: `gpt-5.5`, `gpt-5.4`, `gpt-5.3-codex`.
**Research**: Usar **DeepSeek web** como nodo de apoyo y verificación via MCP/Playwright.

#### 4. GEMINI CLI (Planning Node)
Usa Gemini CLI para síntesis de contexto amplio y planificación multi-documento.
**Main**: `gemini-3.1-pro-preview`.

#### 5. OPENCODE / UNIFICACIÓN (Free Cloud Tier)
Usa OpenCode como capa unificadora para proveedores agnósticos (OpenRouter, Groq).
**Priority**: `OpenRouter free models` -> `Groq free models`.

#### 6. RESEARCH NODES (Verification Path)
- **Perplexity (Primary)**: Use Sonnet first. Fallback to Pro models. Used for triangulation.
- **DeepSeek Web (Secondary)**: Use for corroboration and alternative viewpoints.

***

## 3. SALIDA OBLIGATORIA EN CADA DECISIÓN
- **Main model**: [Model Name]
- **Fallback model**: [Model Name]
- **Provider path**: [Provider]
- **Plane involved**: [WSL/Windows]
- **Validation status**: [Status]
- **Notes**: [Conflict]/[Inferred]

***

## 4. COLA DE EVALUACIÓN (CANDIDATES)
- **Ollama**: Qwen3 variants, DeepSeek variants, Gemma 3, Mistral reasoning, Phi family.
- **Cloud**: New free routers from OpenRouter, new Groq paths.
- **CLI**: New models in Claude Code `/model` selector.
