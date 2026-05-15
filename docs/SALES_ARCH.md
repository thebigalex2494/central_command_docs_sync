# Architecture Mapping: Reportes Ventas Nacional

> SSoT para la estructura técnica del proyecto de ventas ubicado en Drive I:

## 📂 Project Structure

- **`/data`**: Pipeline de datos (Input -> Processed -> Output).
  - `paths_config.yaml`: Gestión centralizada de rutas.
- **`/docs`**: Documentación técnica exhaustiva (Arquitectura, Decisiones, Guías).
- **`/examples`**: Scripts standalone para validación, limpieza y auditoría.
- **`/logs`**: Registro histórico de ejecuciones del pipeline.
- **`/htmlcov`**: Reportes de cobertura de tests.
- **`/templates`**: Plantillas para resultados y presentaciones HTML.

## 🐍 Key Python Modules

### Core & Orchestration
- `agent_daemon.py`: Orquestador principal (Daemon).
- `direct_sync.py`: Sincronización directa de datos.

### Quality & Audit
- `auditoria_consolidada.py`: Auditoría integral del proceso.
- `auditoria_calidad_datos.py`: Validación de integridad de registros.

### External Integrations
- `banxico_module.py`: Conexión con APIs financieras para tipos de cambio.
- `call_apps_script.py`: Puente con Google Apps Script.

## 🛠️ Pipeline Architecture Summary

1. **Extraction**: Los datos entran por `/data/input`.
2. **Transformation**: Scripts en `/examples` y módulos core limpian y procesan (reglas de 6 meses, validación de tarifas).
3. **Audit**: Se ejecutan los validadores de calidad.
4. **Loading**: Los resultados se mueven a `/data/output` y se generan reportes visuales en `/templates`.

---
*Mapeado automáticamente por Central Command Orchestrator el 2026-04-23.*
