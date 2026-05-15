---
name: OneDrive Entregables GDV — Mapeo de destinos
description: Rutas canónicas en OneDrive corporativo para mover outputs de proyectos GDV por tipo/año/mes
type: reference
originSessionId: f26b0666-c01d-41d2-bc19-e1ef1113fa1e
---
# OneDrive Entregables GDV

**Raíz**: `C:\Users\msi\OneDrive - Autotransportes Ejecutivos Del Valle S.A. de C.V\entregables\`

Estructura aprobada 2026-04-14 (plan: `.claude/plans/witty-mixing-squid.md`). Convención: **snake_case sin espacios**, mes con formato `{MM}_{mes}` (ej: `04_abril`, `03_marzo`).

## Árbol

```
entregables/
├── ventas_nacional/{year}/{MM}_{mes}/
│   ├── extracciones/
│   ├── cancelaciones/
│   ├── reportes_operativos/
│   ├── servicios_clean/
│   ├── servicios_raw/
│   ├── seguros/
│   ├── validacion_limpieza/
│   └── servicios_cliente/
├── rentabilidad_nacional/{year}/{MM}_{mes}/
│   ├── gastos/
│   ├── utilidad_nacional/
│   ├── reportes_financieros/
│   └── clientes/
├── consolidados_anuales/{year}/
│   ├── ventas/
│   ├── utilidad/
│   └── operativos/
├── maestros/
│   ├── padron_flotilla/
│   ├── catalogos_clientes/
│   ├── tarifarios/
│   └── plantillas/
└── _legacy/   # Ventas_Nacional/, Rentabilidad_Nacional/, OTROS/ antiguos (no agregar nada)
```

## Mapeo output proj-001 → destino

| Output local (`reportes_ventas_nacional/data/output/...`) | Destino OneDrive (`entregables/...`) |
|---|---|
| `extracciones/extraccion_*.xlsx` | `ventas_nacional/{year}/{MM}_{mes}/extracciones/` |
| `cancelaciones/*.xlsx` | `ventas_nacional/{year}/{MM}_{mes}/cancelaciones/` |
| `reportes_operativos/consolidado_ron_*.xlsx` | `ventas_nacional/{year}/{MM}_{mes}/reportes_operativos/` |
| `servicios_clean/*` | `ventas_nacional/{year}/{MM}_{mes}/servicios_clean/` |
| `ventas_raw/*` | `ventas_nacional/{year}/{MM}_{mes}/servicios_raw/` |
| `seguros/*.xlsx` | `ventas_nacional/{year}/{MM}_{mes}/seguros/` |
| `validacion_limpieza/*.xlsx` | `ventas_nacional/{year}/{MM}_{mes}/validacion_limpieza/` |
| `servicios_cliente/*.xlsx` | `ventas_nacional/{year}/{MM}_{mes}/servicios_cliente/` |
| `gastos/*` | `rentabilidad_nacional/{year}/{MM}_{mes}/gastos/` |
| `utilidad_nacional/*.xlsx` | `rentabilidad_nacional/{year}/{MM}_{mes}/utilidad_nacional/` |
| `reportes_financieros/*` | `rentabilidad_nacional/{year}/{MM}_{mes}/reportes_financieros/` |
| `analisis_clientes/*` | `rentabilidad_nacional/{year}/{MM}_{mes}/clientes/` |
| `consolidados_historicos/*` | `consolidados_anuales/{year}/` (subcarpeta según tipo) |

## NO subir a OneDrive

- `dashboards/`, `dashboard/` (HTML/PNG) — solo se envían por email
- `cache/`, `test_results/`, `qa_*/`, `audits/`, `master_method_validation/`, `duckdb_validation/`
- `change_tracker/`, `snapshots/`, `preview_ron/`, `analisis_patrones/`, `reglas_mejoradas/`
- `horas_pico_can/`, `consumos_combustible_utilitarias/`, `auditoria_reglas/`, `validaciones/`

## Meses en español (snake_case)

`01_enero, 02_febrero, 03_marzo, 04_abril, 05_mayo, 06_junio, 07_julio, 08_agosto, 09_septiembre, 10_octubre, 11_noviembre, 12_diciembre`

## Uso por agente

Cuando el usuario pida "mover X a OneDrive", identificar: (1) tipo de output por nombre/carpeta origen, (2) año, (3) mes → resolver destino con esta tabla. Si el tipo no está listado, preguntar antes de mover. No tocar `_legacy/` ni `OTROS/`.
