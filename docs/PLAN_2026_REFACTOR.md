# Plan de Modernización y Automatización 2026

## 1. Automatización Operativa (Inmediata)
Se ha implementado el script `scripts/scheduler/auto_sync_sales.py` para eliminar la ejecución manual.

- **Frecuencia**: Cada Hora.
- **Flujo**:
  1.  Ejecutar Orquestador (`launcher_orchestrator.py`).
  2.  Validar éxito (Exit Code 0).
  3.  Disparar Webhook GAS (`GAS_SALES_WEBHOOK`).
  4.  Registrar en Logs (`sales_sync.log`).

## 2. Refactorización de Scripts "Consolidados" (Base Audit DeepSeek)
El análisis de `consolidado_servicios_aev.py` indica la necesidad de migrar a una arquitectura modular.

### Acciones Críticas:
1.  **Eliminar Rutas Hardcoded**: Mover `I:\Mi unidad\...` a un archivo `config.yaml`.
2.  **Manejo de Errores**: Envolver la lectura de Excel en bloques `try/except` robustos (actualmente falla silenciosamente si el archivo está abierto).
3.  **Tipo de Datos**: Usar `pandas` con tipos explícitos para evitar errores de conversión en columnas de moneda.

## 3. Integración AppSheet 2026
Para lograr la "Automatización Total 2026":
- Migrar el backend de Excel a **Google BigQuery** o una base de datos SQL ligera si el volumen crece.
- Usar la **API de AppSheet** directamente en lugar de solo actualizar la hoja de cálculo, para forzar la sincronización de datos en tiempo real.

## 4. Próximos Pasos
- Ejecutar: `python scripts/central_command/commander.py "Refactor consolidado_servicios_aev.py applying audit fixes"`
