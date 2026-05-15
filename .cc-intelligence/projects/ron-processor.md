---
name: "ron-processor"
version: 3.0.0
description: "Expert logic for the Reporte Operativo Nacional (RON) pipeline - proj-001"
scope: "project"
project_id: "proj-001"
---

# RON Processor Skill (proj-001)

This skill specializes the agent in the **Reporte Operativo Nacional (RON)** pipeline logic. It provides the architectural knowledge and business rules required to maintain, execute, and audit the national sales reports.

## Domain Knowledge: Business Rules

1.  **Pool de Gastos (Pool Calculation)**:
    -   **Objective**: Consolidate national expenses that must be shared across regions.
    -   **Formula**: `Total Pool = Impuestos + Mantenimiento + Combustible (Q.Roo/Yuc) + Nómina + Otros`.
    -   **Exclusions**: Specific units (e.g., Mahahual) are excluded from the pool despite being in the source data.
    -   **Source**: `src/modules/operational_reports/pool_calculation.py`.

2.  **Prorrateo (Prorration)**:
    -   **Logic**: Expenses are prorrated based on regional sales vs. national total sales.
    -   **Integrity**: The sum of prorrated expenses must match the total pool exactly (Zero-Sum validation).
    -   **Source**: `src/modules/operational_reports/post_processor.py`.

3.  **Dynamic Header Detection**:
    -   **Lean Files**: Headers on Row 1.
    -   **Corporate Files**: Headers on Row 5.
    -   **Mandate**: Always use `pd.read_excel(nrows=10)` to detect the header row before full processing.

## Pipeline Architecture (E2E)

The pipeline is orchestrated by `pipeline_ron_master.py` and follows these phases:

1.  **ETL (Unified Controller)**:
    -   Extraction from ZAM (SSoT for Services).
    -   Banxico API for daily exchange rates.
    -   Aggregating raw data into staging.
2.  **Post-Processing**:
    -   Applying Pool & Prorration logic.
    -   Cleaning up `MergedCells` to avoid `openpyxl` errors.
3.  **Formatting & Validation**:
    -   Applying corporate styles.
    -   Final audit check (Sales ZAM vs Nacional).

## Operational Standards

-   **Integrity Check**: Validate pool totals via Zero-Sum validation (prorrated sum must equal total pool). Reference values in STATUS.md or project DuckDB.
-   **Logger Usage**: Use `LoggerMX.critico()` for catastrophic failures.
-   **Drive Mapping**: Strictly use `paths_config.yaml` for `%DRIVE%` resolution.

## Reference Resources

-   `%CC%\Projects\reportes_ventas_nacional\src\modules\operational_reports\pipeline_ron_master.py`
-   `%CC%\Projects\reportes_ventas_nacional\src\modules\operational_reports\pool_calculation.py`
-   `%CC%\Projects\reportes_ventas_nacional\STATUS.md`
