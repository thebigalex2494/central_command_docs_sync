# 📘 Central Command — Session Blueprint (v4.5)
**Status**: Canonical | **Orchestrator**: Gemini Antigravity

Este documento define el flujo táctico y las mejores prácticas para ejecutar una sesión de desarrollo de alto nivel, integrando los últimos insights de **Perplexity Pro** sobre coordinación multi-agente.

---

## 🛰️ 1. Flujo Táctico de Sesión

### Paso 1: INTAKE (/status)
- **Acción**: Ejecutar `/status` inmediatamente al iniciar.
- **Justificación**: Proporciona el "Radar" del sistema. Sin esto, podrías trabajar sobre infraestructura fallida.

### Paso 2: RESTORE (/sessions)
- **Acción**: `/sessions` -> `/load-session gemini latest`.
- **Justificación**: Recupera el hilo lógico de sesiones móviles (SSH) o delegadas. Evita la "amnesia del agente".

### Paso 3: FOCUS (/load [proj-id])
- **Acción**: Activa el contexto maestro del proyecto.
- **Justificación**: Carga el `SESSION_STATUS.md`. Es el prerrequisito para cualquier plan.

### Paso 4: AUDIT (/audit-structure)
- **Acción**: Limpieza "Clean Root".
- **Justificación**: Elimina el ruido visual y de contexto. Menos archivos en el root = Menos errores de búsqueda.

### Paso 5: PLAN (ACTION_PLAN)
- **Acción**: Generar un plan de acción detallado antes de codificar.
- **Justificación**: El plan es el contrato. Permite delegar a modelos locales con precisión.

---

## 🧠 2. Patrones de Coordinación Avanzada (AI-Pro)

### 2.1 Anchor Files (Anclas de la Verdad)
Para evitar el **Context Drift** en sesiones largas:
- Mantener un `PROJECT_CONTEXT.md` en la raíz.
- Antes de cada fase de ejecución, sincronizar el progreso con el Ancla.

### 2.2 Deterministic Handoff
Al delegar tareas a Claude o Qwen, usar este formato de encabezado:
```markdown
---
task_id: [id] | phase: [EXECUTE|AUDIT] | scope: [files]
constraints: [from PROJECT_CONTEXT.md]
validation: [tests/modo-dev]
---
```

### 2.3 RAG-Lite Local
No cargar logs masivos (>1000 líneas). Usar `grep` o herramientas de filtrado para inyectar solo fragmentos de error específicos.

---

## 🎓 3. Daily Training: El Camino a Data Engineering

Para evolucionar el sistema, sigue este protocolo de aprendizaje diario:

1.  **Observabilidad Duckbase**: Registrar el rendimiento de cada sesión en `observability.duckdb`.
    - **Por qué**: Permite realizar auditorías SQL sobre el gasto de tokens, la duración de las tareas y la tasa de éxito de los agentes locales.
    - **Acción**: Ejecutar `python tools/duckbase_logger.py` al final de tareas críticas.
2.  **Hardening de Python**: Refactorizar módulos usando Type Hints estrictos y `pydantic` para validación de contratos.
3.  **Local-First Optimization**: Perfeccionar el uso de **Qwen3** (Ollama) para reducir la dependencia de nubes pagas.

---

## 🛡️ 4. Reglas de Oro para el Dev
- **Atomicidad**: Archivos < 200 líneas siempre.
- **AI-Ready**: Si un humano no entiende el código a primera vista, una IA tampoco. Comenta el "Por Qué".
- **Validation-First**: No consideres una tarea terminada hasta que `/modo-dev` dé su aprobación.
