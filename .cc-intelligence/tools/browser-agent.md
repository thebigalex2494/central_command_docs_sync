# Skill: BrowserAgent MCP

Esta skill otorga a Gemini CLI la capacidad de realizar navegación web avanzada, interacción con aplicaciones (AppSheet) y extracción de datos de sitios externos (Perplexity, DeepSeek) utilizando un agente especializado con persistencia de sesión.

## Identity

- **Name**: `browser-agent-mcp`
- **Engine**: Playwright MCP + Anthropic Claude (Reasoning)
- **Status**: EXPERIMENTAL (Tier 4)
- **Capability**: Google Auth Persistence via `pw-pool`

## Instrucciones de Operación

### 1. Selección de Modo
- **Modo Automatizado (Headless)**: Usar para tareas de extracción rutinarias donde ya existe una sesión activa.
- **Modo Login/Mantenimiento (Headed)**: Usar obligatoriamente cuando se necesite iniciar sesión manualmente o resolver captchas.

### 2. Gestión de Sesión (pw-pool)
La skill utiliza automáticamente el pool de Playwright. 
- Si no hay instancias libres, espera o solicita al usuario ejecutar `bash tools/pw-pool/pw-lock.sh cleanup`.
- Al finalizar, el lock se libera automáticamente.

## Comandos Disponibles

### Ejecución Estándar (Invisible)
```bash
python %CC%\tools\browser_agent_playwright_mcp.py "<objetivo>"
```

### Ejecución de Mantenimiento (Visible para Login)
Para forzar el modo visible, se debe pasar el parámetro interno a la herramienta o usar este comando CLI:
```bash
# Nota: Requiere que el script detecte la intención o se use el parámetro headless=False si se llama vía MCP
python %CC%\tools\browser_agent_playwright_mcp.py "Navega a <url> en modo visible para login"
```

## Configuración Requerida
- `ANTHROPIC_API_KEY`: Clave para el agente de razonamiento.
- `BROWSER_AGENT_ALLOW_AI_DOMAINS`: Debe ser `true` para pruebas con Perplexity/DeepSeek.

## Reglas de Seguridad
1. **No Loops**: El agente tiene un límite de 10 iteraciones por tarea.
2. **Dominio Restringido**: Bloqueo por defecto de dominios de IA para evitar recursión infinita, bypass controlado por variable de entorno.
3. **Privacidad**: Las sesiones se guardan localmente en `tools/pw-pool/sessions/`.
