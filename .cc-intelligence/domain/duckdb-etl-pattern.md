---
name: duckdb-etl-pattern
description: Standards for high-performance ETL using DuckDB and Parquet. Use when processing large CSV/Excel files (>100MB) or when complex SQL-like aggregations are needed.
---
# DuckDB & Parquet ETL Pattern

## Overview
DuckDB provides an in-process SQL OLAP engine that is 5-20x faster than pandas for large-scale data manipulation. Use this pattern to modernize legacy pandas pipelines.

## 1. Data Ingestion (Parquet First)
Always convert raw CSV/Excel to Parquet for intermediate storage. Parquet is compressed and columnar, allowing for projection pushdown.

```python
import duckdb
import pandas as pd

# Convert Excel to Parquet (Initial Load)
df = pd.read_excel("data.xlsx", sheet_name="ZAM")
df.to_parquet("data.parquet", compression="snappy")
```

## 2. High-Speed SQL Aggregation
Perform joins and group-bys directly on Parquet files using SQL.

```python
import duckdb

# Connect to in-memory DB or persistent file
con = duckdb.connect("analytics.db")

# Query Parquet directly (Zero-copy)
query = """
SELECT 
    localidad, 
    SUM(ventas) as total_ventas,
    AVG(margen) as avg_margen
FROM 'data.parquet'
WHERE fecha >= '2026-01-01'
GROUP BY localidad
ORDER BY total_ventas DESC
"""

results_df = con.execute(query).df()
```

## 3. Best Practices
- **Projection Pushdown**: Only select the columns you need in the SQL query.
- **Filter Pushdown**: Apply `WHERE` clauses in SQL to reduce memory footprint.
- **Chunking**: For extremely large files, use `duckdb.query(...).fetch_df_chunk()` to process in memory-safe batches.
- **Memory Limit**: Use `SET memory_limit = '4GB';` to prevent crashing on systems with limited RAM (like your RTX 3060 CPU-offload).

## 4. Integration with proj-001
Replace legacy `pd.merge` and `pd.groupby` operations in `reportes_ventas_nacional` with DuckDB SQL queries for a 10x performance gain.
