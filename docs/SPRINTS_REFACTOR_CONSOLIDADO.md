# Ronda de Sprints: Refactorización `consolidado_servicios_aev.py`

> **Objetivo**: Elevar calificación de **C → A** aplicando estándares Python estándar + SIP
> **Entorno principal**: VS Code (Python estándar como plan B para modelos extra)
> **Archivo fuente**: `I:\Mi unidad\grupo_del_valle\scripts python\por_organizar\base_consolidado_ventas\consolidado_servicios_aev.py`
> **Líneas actuales**: ~434 (monolito con 6 módulos conceptuales embebidos)
> **Fecha de inicio**: 2026-02-12
> **Issues de auditoría**: 5 críticos

---

## Resumen Ejecutivo

| Sprint | Nombre | Enfoque | Issues que resuelve | Entregable |
|:------:|--------|---------|:-------------------:|------------|
| **S1** | Scaffold & Config | Estructura de proyecto + config externalizada | #1, #3 | Paquete Python con `config/` |
| **S2** | Modularización | Separar monolito en módulos reales | — (deuda estructural) | 6 archivos < 200 líneas c/u |
| **S3** | Hardening | Seguridad, logging dinámico, validación | #2, #4, #5 | Código robusto y seguro |
| **S4** | QA & Cierre | Tests, documentación, auditoría final | — (calidad) | Grade A confirmado |

---

## Sprint 1: Scaffold & Config
**Duración estimada**: 1 sesión de trabajo

### Objetivo
Crear la estructura de proyecto Python estándar y externalizar toda la configuración hardcodeada.

### Tareas

| # | Tarea | Detalle | Criterio de aceptación |
|---|-------|---------|----------------------|
| 1.1 | Crear estructura de directorios | Scaffold estándar Python estándar | Directorios `src/`, `config/`, `scripts/`, `tests/` creados |
| 1.2 | Crear `config/settings.py` | Cargar env vars con validación y defaults | Todas las env vars (`USER_ID`, `DOWNLOAD_ID`, `GOOGLE_CREDENTIALS_PATH`, `DRIVE_FOLDER_ID`) validadas al startup. `ConfigError` si faltan las obligatorias |
| 1.3 | Crear `.env.example` | Template documentado de variables de entorno | Todas las variables listadas con descripción y valores ejemplo |
| 1.4 | Crear `requirements.txt` con pinning | Fijar versiones exactas de dependencias | 7 dependencias con version pinning (`requests==2.31.0`, etc.) |
| 1.5 | Crear `pyproject.toml` | Metadata del paquete y dependencias | Nombre, versión, Python >=3.10, deps listadas |

### Estructura objetivo

```
consolidado_servicios_aev/
├── src/
│   ├── __init__.py
│   ├── config.py           ← S1 (validación de env vars)
│   ├── exceptions.py       ← S2
│   ├── downloader.py       ← S2
│   ├── uploader.py         ← S2
│   ├── ui.py               ← S2
│   └── logger.py           ← S2/S3
├── scripts/
│   └── run.py              ← S2 (entry point)
├── config/
│   └── settings.py         ← S1
├── tests/
│   ├── __init__.py
│   ├── test_config.py      ← S4
│   ├── test_downloader.py  ← S4
│   └── test_uploader.py    ← S4
├── .env.example             ← S1
├── requirements.txt         ← S1
├── pyproject.toml           ← S1
└── README.md                ← S4
```

### Issues de auditoría resueltos
- **#1** Variables de entorno sin validación → `config.py` con validación robusta
- **#3** Dependencias sin version pinning → `requirements.txt` con versiones fijas

### Definition of Done
- [ ] `python -c "from src.config import Settings; s = Settings(); print(s)"` funciona
- [ ] Sin env vars configuradas, el script lanza `ConfigError` claro
- [ ] `.env.example` documenta cada variable

---

## Sprint 2: Modularización
**Duración estimada**: 1 sesión de trabajo
**Dependencia**: Sprint 1 completado

### Objetivo
Separar el monolito en módulos reales siguiendo la regla de <200 líneas por archivo.

### Tareas

| # | Tarea | Detalle | Origen (líneas) |
|---|-------|---------|-----------------|
| 2.1 | Crear `src/exceptions.py` | 3 custom exceptions: `DownloadError`, `UploadError`, `ConfigError` | L114-120 |
| 2.2 | Crear `src/downloader.py` | `download_file(url)`, `build_download_url(params)` con type hints | L150-220 |
| 2.3 | Crear `src/uploader.py` | `upload_to_drive(stream, filename, folder_id)` con service account auth | L222-280 |
| 2.4 | Crear `src/ui.py` | `pick_dates()`, `show_progress()` con Rich + Questionary | L282-316 |
| 2.5 | Crear `src/logger.py` | `setup_logger(level)` con loguru dual output | L80-112 |
| 2.6 | Crear `scripts/run.py` | Entry point con `main()` que orquesta el flujo completo | L318-405 |
| 2.7 | Eliminar código global | Todo envuelto en funciones/clases. `if __name__ == "__main__"` como único global | Todo |

### Reglas de migración
- **No cambiar lógica de negocio** en este sprint — solo mover y conectar
- Cada módulo debe ser importable independientemente
- Type hints en todas las funciones públicas
- Imports absolutos: `from src.config import Settings`

### Definition of Done
- [ ] `python scripts/run.py` ejecuta el mismo flujo que el monolito original
- [ ] Cada archivo `src/*.py` tiene menos de 200 líneas
- [ ] No hay código suelto fuera de funciones/clases
- [ ] Todos los imports resuelven correctamente

---

## Sprint 3: Hardening
**Duración estimada**: 1 sesión de trabajo
**Dependencia**: Sprint 2 completado

### Objetivo
Resolver los issues de seguridad, logging y validación restantes de la auditoría.

### Tareas

| # | Tarea | Issue | Detalle |
|---|-------|-------|---------|
| 3.1 | Logging dinámico | **#2** | `LOG_LEVEL` env var (default `INFO`). Validar contra `DEBUG/INFO/WARNING/ERROR`. Implementar en `src/logger.py` |
| 3.2 | Validación de credenciales Google | **#4** | En `src/config.py`: verificar que el archivo de credenciales existe, es legible, y es JSON válido antes de continuar |
| 3.3 | Validación de inputs de fecha | **#5** | En `src/ui.py`: validar formato `YYYY-MM-DD`, rango lógico (inicio < fin), fecha no futura. Retry en input inválido |
| 3.4 | Retry con backoff en requests | — | En `src/downloader.py`: implementar retry (3 intentos, exponential backoff) para `download_file()` |
| 3.5 | Timeouts en HTTP | — | `requests.get(url, timeout=30)` en todas las llamadas HTTP |
| 3.6 | Manejo de archivo Excel abierto | — | En `src/downloader.py`: detectar y manejar `PermissionError` si el archivo está en uso |
| 3.7 | Graceful shutdown | — | Capturar `KeyboardInterrupt` en `main()` para cierre limpio con mensaje |

### Detalle técnico por issue

**3.1 — Logging dinámico**
```python
# src/logger.py
def setup_logger(level: str | None = None) -> None:
    log_level = level or os.getenv("LOG_LEVEL", "INFO")
    if log_level not in ("DEBUG", "INFO", "WARNING", "ERROR"):
        log_level = "INFO"
    logger.add(sys.stderr, level=log_level)
    logger.add("logs/app.log", rotation="1 MB", level=log_level)
```

**3.2 — Validación de credenciales**
```python
# src/config.py
def _validate_credentials(path: str) -> Path:
    cred_path = Path(path)
    if not cred_path.exists():
        raise ConfigError(f"Credentials file not found: {path}")
    if not cred_path.suffix == ".json":
        raise ConfigError(f"Credentials must be JSON: {path}")
    # Verify it's parseable
    import json
    with open(cred_path) as f:
        json.load(f)  # Raises JSONDecodeError if invalid
    return cred_path
```

**3.3 — Validación de fechas**
```python
# src/ui.py
def _validate_date(text: str) -> bool | str:
    try:
        dt = datetime.strptime(text, "%Y-%m-%d")
        if dt > datetime.now():
            return "Date cannot be in the future"
        return True
    except ValueError:
        return "Invalid format. Use YYYY-MM-DD"
```

### Issues de auditoría resueltos
- **#2** Logging estático → dinámico via env var
- **#4** Credenciales Google inseguras → validación de archivo al startup
- **#5** Inputs sin validación → validadores de formato + rango

### Definition of Done
- [ ] `LOG_LEVEL=DEBUG python scripts/run.py` muestra logs de debug
- [ ] Sin archivo de credenciales → error claro en consola
- [ ] Fecha `2099-01-01` → rechazada con mensaje
- [ ] Fecha `abc` → rechazada con mensaje
- [ ] Sin conexión de red → 3 reintentos con backoff + error claro
- [ ] Los 5 issues de auditoría están resueltos

---

## Sprint 4: QA & Cierre
**Duración estimada**: 1 sesión de trabajo
**Dependencia**: Sprint 3 completado

### Objetivo
Agregar tests, documentación, y ejecutar auditoría final para confirmar Grade A.

### Tareas

| # | Tarea | Detalle |
|---|-------|---------|
| 4.1 | Tests unitarios — config | `test_config.py`: env vars presentes/ausentes, validación de credenciales, defaults |
| 4.2 | Tests unitarios — downloader | `test_downloader.py`: URL building, retry logic (mock HTTP), timeout handling |
| 4.3 | Tests unitarios — uploader | `test_uploader.py`: auth flow (mock), upload success/failure |
| 4.4 | Tests unitarios — ui | `test_ui.py`: validación de fechas (formatos válidos/inválidos, rangos) |
| 4.5 | README.md | Propósito, instalación, configuración, uso, arquitectura |
| 4.6 | Migrar archivo original | Mover monolito a `_legacy/` y dejar la nueva estructura como principal |
| 4.7 | Auditoría final | Re-ejecutar checklist de auditoría contra los 5 issues originales |
| 4.8 | Actualizar registry | Actualizar `projects-registry.json` con nuevo estado del proyecto |

### Cobertura mínima de tests

| Módulo | Tests requeridos | Mocks necesarios |
|--------|-----------------|-----------------|
| `config.py` | 5 tests | `os.environ` (patch) |
| `downloader.py` | 4 tests | `requests.get` (mock responses) |
| `uploader.py` | 3 tests | `googleapiclient` (mock) |
| `ui.py` | 4 tests | `questionary` (mock input) |
| **Total** | **16 tests** | — |

### Checklist de auditoría final

| # | Issue original | Estado esperado | Verificación |
|---|---------------|-----------------|-------------|
| 1 | Env vars sin validación | RESUELTO (S1) | `Settings()` sin vars → `ConfigError` |
| 2 | Logging estático | RESUELTO (S3) | `LOG_LEVEL` env var funcional |
| 3 | Sin version pinning | RESUELTO (S1) | `requirements.txt` con `==` |
| 4 | Credenciales inseguras | RESUELTO (S3) | Validación de path + JSON |
| 5 | Inputs sin validación | RESUELTO (S3) | Validadores de fecha activos |

### Definition of Done
- [ ] 16 tests pasan (`pytest tests/ -v` → all green)
- [ ] README.md completo con secciones de instalación y uso
- [ ] Archivo original movido a `_legacy/`
- [ ] Auditoría final: 5/5 issues resueltos
- [ ] Grade objetivo: **A**
- [ ] `projects-registry.json` actualizado

---

## Timeline y Dependencias

```
S1 ─────► S2 ─────► S3 ─────► S4
Scaffold   Modular   Harden    QA
  Config     Split    Secure    Test
                                Audit
```

**Todas las dependencias son secuenciales** — cada sprint construye sobre el anterior.

### Criterios de avance entre sprints

| Transición | Gate |
|-----------|------|
| S1 → S2 | `Settings()` instanciable y validando env vars |
| S2 → S3 | `python scripts/run.py` ejecuta flujo completo |
| S3 → S4 | Los 5 issues de auditoría resueltos en código |

---

## Riesgos y Mitigaciones

| Riesgo | Impacto | Mitigación |
|--------|---------|------------|
| Archivo fuente en unidad I: (Google Drive) inaccesible | Bloquea todo | Copiar archivo a `%HOME%\Projects\` como primer paso |
| Cambio de lógica durante modularización (S2) | Regresiones | No cambiar lógica en S2 — solo mover código |
| Credenciales de Google no disponibles para testing | Bloquea S4 tests de uploader | Usar mocks exclusivamente |
| Excel en uso por otro proceso | Bloquea ejecución | Implementado en S3.6 con retry |

---

## Métricas de éxito

| Métrica | Antes (C) | Objetivo (A) |
|---------|-----------|-------------|
| Archivos | 1 monolito (434 líneas) | 8+ módulos (< 200 c/u) |
| Validación env vars | Ninguna | 100% validadas al startup |
| Logging | Fijo INFO | Dinámico via `LOG_LEVEL` |
| Tests | 0 | 16+ |
| Version pinning | No | Sí (`==` en requirements) |
| Input validation | No | Formato + rango + retry |
| Credencial validation | No | Path + JSON check |
| Timeouts HTTP | No | 30s con retry x3 |

---

> **Protocolo**: Al completar cada sprint, ejecutar Auto-QA antes de avanzar al siguiente.
> **Circuit Breaker**: Si un sprint falla 2 veces consecutivas → pausar, notificar, intervención manual.
