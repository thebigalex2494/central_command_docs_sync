# Plan: Integración Multi-Tier AI Stack en Central Command

**Fecha:** 2026-04-08  
**Estado:** En diseño — pendiente evaluación post-merge  
**Autor:** amendez + Claude Code  

---

## Contexto

Central Command v3 es el sistema de orquestación multi-agente de Grupo del Valle. Actualmente usa Ollama-first routing con fallback a Claude CLI. Este plan extiende la arquitectura para maximizar uso $0 integrando cloud APIs gratuitas, Aider como coding agent automatizado, OpenClaw como tool orchestrator, y la knowledge base de plugins de Claude Code para potenciar agentes locales.

---

## Arquitectura: 5 Tiers

```
Tier 0: $0 Local Ollama (RTX 3060 12GB)
  ├── qwen3:14b      — razonamiento, chat, auditor
  ├── qwen2.5-coder  — código, refactoring
  └── gemma3          — tareas ligeras

Tier 1: $0 Cloud APIs
  ├── Gemini 2.5 Flash API  — 500 req/día, contexto largo
  ├── Gemini 2.5 Pro API    — 25 req/día, razonamiento
  ├── Groq llama-3.3-70b    — 30 req/min, inferencia rápida
  └── Groq deepseek-r1-70b  — 30 req/min, razonamiento

Tier 2: $0 Code Editing (Aider)
  ├── Architect mode: Tier 0/1 modelo planea, otro ejecuta
  ├── Git-native: auto-commits
  ├── Headless: invocable desde CC como tool
  └── Knowledge base: .aider.conf.yml → CLAUDE_SUPERPOWERS.md

Tier 3: Tools & Orchestration
  ├── OpenClaw — gh-issues, coding-agent, skills (gateway local)
  └── cc chat  — agente conversacional DuckDB/pipeline

Tier 4: Interactive Agents ($$)
  ├── Gemini CLI (Pro 3) — PRIMERA OPCIÓN (gratis via OAuth personal)
  └── Claude Code          — fallback $$ (solo tareas críticas/complejas)
```

---

## Enfoque Aprobado

### Arquitectura: Enfoque C — CC como cerebro

CC permanece como orquestador central. OpenClaw y Aider son **tools invocables**, no middleware. CC ya tiene 247+ tests, 9 agentes, token tracking, y routing maduro.

**Justificación:** No tiene sentido agregar capas intermedias cuando CC ya orquesta. Aider y OpenClaw se integran como providers/tools adicionales, igual que Ollama o Claude CLI.

### Knowledge Base: Enfoque A — System Prompt Injection

Inyectar skills/conocimiento directamente en system prompts de los modelos locales. No RAG, no portable skills.

**Justificación:**
- Skills promedio: ~3,300 tokens — caben en cualquier modelo
- Zero dependencias (no vector DB, no embeddings)
- Patrón ya validado con `.aider.conf.yml` → `read: CLAUDE_SUPERPOWERS.md`
- Simplicidad > complejidad para el volumen actual

---

## Knowledge Base: 3 Fuentes

### Fuente 1: Claude Plugins Cache
- **Ubicación:** `~/.claude/plugins/cache/claude-plugins-official/`
- **Contenido:** 16 skills, 82 archivos .md, ~102K tokens total
- **Skills clave:** systematic-debugging (5K words), brainstorming (3.3K), TDD (2.7K), subagent-driven-dev (2.6K)

### Fuente 2: Claude Code Main (Google Drive)
- **Ubicación:** Google Drive → central_command → claude-code-main
- **Contenido:** 13 plugins con agentes especializados
- **Plugins clave:** code-review (5 agentes paralelos), feature-dev (7 fases, 3 agentes), security-guidance (9 patrones), pr-review-toolkit (6 agentes)

### Fuente 3: Adaptaciones Aider
- **CLAUDE_SUPERPOWERS.md** — workflow 7 fases adaptado para Aider
- **CONVENTIONS.md** — reglas de sistema para modelos locales (Senior SW Engineer role)
- **aider_command_validator.py** — bloqueador regex de comandos peligrosos (rm -rf, sudo, git push, etc.)

---

## Estado Actual del Servidor (BlueDragon)

| Componente | Estado | Notas |
|------------|--------|-------|
| Ollama | OPERATIVO | qwen3:14b, qwen2.5-coder, gemma3 |
| GPU | RTX 3060 12GB | 11.8 GB libres |
| OpenClaw v2026.4.5 | INSTALADO | Gateway stopped, config apunta a modelo inexistente |
| Aider | NO INSTALADO | Pendiente |
| Gemini CLI v0.36.0 | OPERATIVO | OAuth personal configurado |
| Python 3.13.5 | OPERATIVO | venv activo |
| Central Command | OPERATIVO | 247 tests, 9 agentes |

---

## Tareas Pendientes

### Fase 0: Pre-requisitos
- [ ] Completar merge de branches en Central Command
- [ ] Evaluar proyecto actualizado post-merge
- [ ] Configurar API key de Groq (crear cuenta en console.groq.com)
- [ ] Configurar API key de Gemini (Google AI Studio → aistudio.google.com)

### Fase 1: Infraestructura
- [ ] Instalar Aider en BlueDragon (`pip install aider-chat` en .venv)
- [ ] Fixear OpenClaw config: cambiar `kimi-k2.5:cloud` → `qwen3:14b`
- [ ] Arrancar OpenClaw gateway
- [ ] Clonar/sincronizar claude-code-main a BlueDragon (actualmente solo en Google Drive)

### Fase 2: Providers en Central Command
- [ ] Crear `providers/gemini_provider.py` — API 2.5 Flash/Pro
- [ ] Crear `providers/groq_provider.py` — llama-3.3-70b, deepseek-r1-70b
- [ ] Crear `providers/aider_provider.py` — wrapper CLI para coding tasks
- [ ] Actualizar `router.py` — agregar Tier 1 cloud al routing logic
- [ ] Agregar API keys a `config/settings.yaml`

### Fase 3: Knowledge Base
- [ ] Curar skills de Fuente 1 (plugins cache) → extraer top 5-8 más útiles
- [ ] Adaptar skills de Fuente 2 (claude-code-main) → system prompts compactos
- [ ] Crear directorio `scripts/central_command/knowledge/` con prompts inyectables
- [ ] Integrar inyección en `OllamaToolProvider.chat()` y cloud providers

### Fase 4: Integración & Tests
- [ ] Tests para cada nuevo provider
- [ ] Tests de routing con 5 tiers
- [ ] Test E2E: CC → Aider → commit automático
- [ ] Test E2E: CC → Groq/Gemini API fallback
- [ ] Benchmark: latencia y calidad por tier

### Fase 5: Documentación
- [ ] Actualizar CLAUDE.md con nueva arquitectura
- [ ] Documentar rate limits y fallback logic
- [ ] Actualizar memory files

---

## Comparativa: Aider vs VS Code Extensions

| Criterio | Aider | VS Code ext |
|----------|-------|-------------|
| Automatizable desde CC | **Sí** | No |
| Corre headless (BlueDragon) | **Sí** | No (necesita GUI) |
| Multi-LLM ($0 routing) | **Sí** | Limitado |
| Knowledge base inyectable | **Sí** | Parcial |
| Inline completions | No | **Sí** |
| Git-native auto-commits | **Sí** | No |

**Conclusión:** Aider es para automated coding (Tier 2). Extensiones de VS Code son herramientas personales de IDE, no componentes de pipeline.

---

## Decisiones de Diseño

1. **CC como cerebro** — no agregar capas middleware (OpenClaw/Aider son tools, no orquestadores)
2. **System prompt injection** — no RAG (skills son pequeñas, ~3.3K tokens promedio)
3. **Gemini CLI (Pro 3) antes que Claude Code** — gratis via OAuth, modelo superior a 2.5
4. **Ollama-first extendido** — 80%+ tareas locales, cloud solo cuando local no alcanza
5. **Unidad 9994 (Vallarta)** — registro de ajuste conocido, no es error en analytics

---

## Verificación

```bash
# Post-implementación
cd ~/central_command && .venv/bin/python -m pytest tests/ -q  # todos pasan
cc status                                                       # health check
cc chat                                                         # agente local funcional
aider --model ollama/qwen3:14b --message "test"                # Aider operativo
curl -s http://localhost:18789/health                           # OpenClaw gateway up
```
