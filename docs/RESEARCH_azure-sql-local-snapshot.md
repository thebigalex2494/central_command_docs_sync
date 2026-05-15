# Investigación: Snapshot Periódico de Azure SQL a Servidor Local

> **Proyecto**: Retomar el control de datos alojados en Azure SQL Database gestionada por proveedor externo.
> **Fecha**: 2026-02-11
> **Estado**: Investigación completada — pendiente implementación
> **Referencia técnica**: `Projects/docs/azure-sql-snapshot-guide.md`

---

## 1. Contexto y Problema

### Situación actual

- La base de datos de producción es **Azure SQL Database** (PaaS), gestionada por un proveedor externo.
- El proveedor **entrega los datos** pero **no otorga acceso directo** a la infraestructura de Azure (portal, CLI, conexión libre al servidor SQL).
- Actualmente no se tiene autonomía para consultar, analizar ni explotar los datos fuera de lo que el proveedor permite.
- Los datos son propiedad de la organización, pero el control operativo está en manos del proveedor.

### Problema

Sin acceso autónomo a los datos:
- No se pueden generar reportes ad-hoc entre catálogos.
- No se puede cruzar información entre tablas de manera libre.
- No se puede alimentar proyectos de automatización, analytics o IA con datos frescos.
- Cualquier reporte depende del tiempo de respuesta del proveedor.
- No existe un respaldo propio fuera de la infraestructura del proveedor.

### Motivación

Retomar la **soberanía sobre los datos propios** para habilitar:
- Reportes automatizados entre catálogos sin depender del proveedor.
- Análisis de datos con herramientas propias (Python, DuckDB, Power BI, etc.).
- Proyectos de impacto que requieran acceso libre a la información.
- Respaldo independiente como estrategia de continuidad de negocio.

---

## 2. Objetivos

### Objetivo General

**Establecer un proceso automatizado que genere y descargue periódicamente un snapshot de la base de datos Azure SQL a un servidor local**, creando una copia consultable e independiente que permita la generación de reportes, análisis cruzado entre catálogos y alimentación de proyectos de impacto — sin depender del proveedor para el acceso a los datos.

### Objetivos Específicos

| # | Objetivo Específico | Entregable | Dependencia |
|---|---------------------|------------|-------------|
| **OE-1** | Obtener credenciales o mecanismo de acceso de lectura a la base Azure SQL | Credenciales SQL o connection string validados | Negociación con proveedor |
| **OE-2** | Instalar y configurar las herramientas CLI necesarias en el servidor local | Herramientas verificadas (`sqlpackage`, `bcp`, `sqlcmd`, `DuckDB`) | OE-1 |
| **OE-3** | Validar conectividad remota desde el servidor local hacia Azure SQL | Prueba de conexión exitosa documentada | OE-1, OE-2 |
| **OE-4** | Diseñar el esquema de snapshot (completo vs. por catálogos) | Matriz de tablas a exportar con frecuencia y método | OE-3 |
| **OE-5** | Implementar el script de exportación automatizada | Script PowerShell funcional con logging y retry | OE-4 |
| **OE-6** | Configurar la restauración/importación local del snapshot | Base de datos local consultable (SQL Express o DuckDB) | OE-5 |
| **OE-7** | Automatizar la ejecución periódica del proceso | Task Scheduler o servicio configurado | OE-5, OE-6 |
| **OE-8** | Establecer política de retención y limpieza de snapshots | Retención definida y limpieza automática operando | OE-7 |
| **OE-9** | Validar el pipeline completo end-to-end | Reporte generado desde snapshot local que coincida con datos en Azure | OE-6 |

### Diagrama de dependencias

```
OE-1 (Credenciales)
  ├──> OE-2 (Herramientas)
  │      └──> OE-3 (Conectividad)
  │             └──> OE-4 (Diseño de snapshot)
  │                    └──> OE-5 (Script de export)
  │                           ├──> OE-6 (Import local)
  │                           │      └──> OE-9 (Validación E2E)
  │                           └──> OE-7 (Automatización)
  │                                  └──> OE-8 (Retención)
  └──> OE-3
```

---

## 3. Escenarios de Acceso — Según lo que otorgue el proveedor

La estrategia técnica depende del **nivel de acceso** que el proveedor esté dispuesto a otorgar. Se documentan los 4 escenarios posibles, de más favorable a menos favorable.

### Escenario A: Credenciales SQL de solo lectura (ideal)

**Qué se obtiene del proveedor:**
- Usuario y contraseña SQL con permisos `db_datareader`
- Endpoint del servidor: `servidor.database.windows.net`
- IP del servidor local agregada al firewall de Azure SQL

**Qué habilita:**
- Acceso directo con `sqlpackage`, `bcp`, `sqlcmd`, DuckDB
- Snapshot completo (BACPAC) o por tablas (CSV/Parquet)
- Máxima flexibilidad y automatización total

**Herramientas aplicables:** Todas (sqlpackage, bcp, sqlcmd, DuckDB mssql extension)

**Solicitud al proveedor:**
```
Requerimos un usuario SQL de solo lectura (db_datareader) en la base de datos
[NOMBRE_DB] alojada en Azure SQL, junto con el endpoint del servidor y la
apertura de firewall para nuestra IP [IP_SERVIDOR_LOCAL].

El usuario será utilizado exclusivamente para consultas de lectura y generación
de respaldos periódicos de nuestros propios datos. No se requieren permisos de
escritura ni de administración.
```

### Escenario B: Connection string limitada (con restricciones)

**Qué se obtiene del proveedor:**
- Connection string con acceso a vistas específicas (no a todas las tablas)
- Posiblemente acceso solo a un subconjunto de esquemas

**Qué habilita:**
- Exportación de las vistas/tablas permitidas
- Snapshot parcial pero funcional

**Herramientas aplicables:** bcp (queries sobre vistas), sqlcmd, DuckDB

**Adaptación necesaria:**
- Mapear qué vistas/tablas están disponibles
- Ajustar el script de export para trabajar solo con lo accesible
- Si faltan catálogos críticos, negociar ampliación de acceso

### Escenario C: Proveedor entrega archivos de export (BACPAC o CSV)

**Qué se obtiene del proveedor:**
- Archivos BACPAC periódicos depositados en un Blob Storage compartido
- O archivos CSV/Excel enviados por correo o carpeta compartida

**Qué habilita:**
- Importación local del BACPAC a SQL Express
- O carga de CSVs a DuckDB/SQLite para consultas

**Herramientas aplicables:**
- `sqlpackage /Action:Import` (para BACPAC)
- DuckDB `read_csv_auto()` (para CSVs)
- Python pandas (para Excel)

**Adaptación necesaria:**
- Automatizar la descarga del Blob Storage (`az storage blob download` o `azcopy`)
- Establecer acuerdo de frecuencia de entrega con el proveedor
- Validar completitud de los archivos recibidos

**Solicitud al proveedor:**
```
Requerimos que se deposite un export periódico (BACPAC o CSV por tabla) de la
base de datos [NOMBRE_DB] en un contenedor de Azure Blob Storage al que
tengamos acceso de lectura, con frecuencia [diaria/semanal].

Alternativamente, solicitamos acceso de lectura al contenedor donde se
almacenan los backups existentes de la base de datos.
```

### Escenario D: Solo acceso vía API o aplicación del proveedor (peor caso)

**Qué se obtiene del proveedor:**
- Acceso solo a través de su aplicación o API REST
- No hay acceso SQL directo

**Qué habilita:**
- Extracción de datos vía API endpoint por endpoint
- Construcción del snapshot tabla por tabla vía requests HTTP

**Herramientas aplicables:**
- Python + `requests`/`httpx` para consumir la API
- DuckDB para consolidar los datos descargados

**Adaptación necesaria:**
- Mapear todos los endpoints de la API del proveedor
- Implementar paginación y rate limiting
- Manejar autenticación OAuth/API key
- Reconciliar datos entre endpoints (foreign keys manuales)

**Este escenario es el menos deseable** pero sigue siendo viable. La prioridad debe ser negociar al menos el Escenario B.

### Matriz de decisión

| Criterio | Escenario A | Escenario B | Escenario C | Escenario D |
|----------|:-----------:|:-----------:|:-----------:|:-----------:|
| Datos completos | Si | Parcial | Si* | Parcial |
| Automatización total | Si | Si | Parcial | Si (complejo) |
| Frescura de datos | Tiempo real | Tiempo real | Depende del proveedor | Tiempo real |
| Esfuerzo de implementación | Bajo | Bajo-Medio | Medio | Alto |
| Dependencia del proveedor | Solo firewall | Configura vistas | Genera exports | Mantiene API |
| Costo adicional Azure | $0 | $0 | Storage | $0 |

*Si el proveedor entrega export completo.

---

## 4. Arquitectura Técnica

### 4.1 Arquitectura objetivo (Escenario A — acceso SQL directo)

```
┌──────────────────────────────────────────────────────────────────┐
│                        AZURE (Proveedor)                         │
│                                                                  │
│  ┌─────────────────────────┐                                     │
│  │   Azure SQL Database    │◄── Firewall rule: IP servidor local │
│  │   (Producción)          │                                     │
│  └────────┬────────────────┘                                     │
│           │ TDS (puerto 1433, TLS)                               │
└───────────┼──────────────────────────────────────────────────────┘
            │
            │  Internet (conexión encriptada)
            │
┌───────────┼──────────────────────────────────────────────────────┐
│           ▼              SERVIDOR LOCAL                           │
│                                                                  │
│  ┌─────────────────────┐     ┌──────────────────────────────┐   │
│  │  Script PowerShell  │────►│  Archivos de Snapshot         │   │
│  │  (Task Scheduler)   │     │                               │   │
│  │                     │     │  ├─ BACPAC (DB completa)      │   │
│  │  • sqlpackage       │     │  ├─ CSVs (por tabla)          │   │
│  │  • bcp              │     │  └─ Parquet (por tabla)       │   │
│  │  • retry logic      │     └──────────┬───────────────────┘   │
│  │  • logging          │                │                        │
│  │  • notificaciones   │                ▼                        │
│  └─────────────────────┘     ┌──────────────────────────────┐   │
│                              │  Motor de Consulta Local      │   │
│  ┌─────────────────────┐     │                               │   │
│  │  Retención          │     │  Opción A: SQL Server Express │   │
│  │  • 48h snapshots    │     │  Opción B: DuckDB (Parquet)   │   │
│  │  • 7d diarios       │     │  Opción C: SQLite             │   │
│  │  • limpieza auto    │     └──────────┬───────────────────┘   │
│  └─────────────────────┘                │                        │
│                                         ▼                        │
│                              ┌──────────────────────────────┐   │
│                              │  Capa de Reportes             │   │
│                              │                               │   │
│                              │  • Python + pandas            │   │
│                              │  • DuckDB queries directas    │   │
│                              │  • Power BI / Excel           │   │
│                              │  • Scripts automatizados      │   │
│                              └──────────────────────────────┘   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### 4.2 Flujo de datos — proceso por ejecución

```
[Trigger: Task Scheduler cada N horas]
         │
         ▼
[1. Validar credenciales]
  • Leer de Credential Manager / Key Vault
  • Verificar que no hayan expirado
         │
         ▼
[2. Verificar conectividad]
  • Test de conexión TCP al puerto 1433
  • Query simple: SELECT 1
  • Si falla → retry con backoff → si persiste → notificar y abortar
         │
         ▼
[3. Exportar datos] ─── Dos estrategias posibles:
  │
  ├─ Estrategia BACPAC (snapshot completo)
  │   • sqlpackage /Action:Export
  │   • Genera archivo .bacpac (ZIP con schema + datos)
  │   • Pesado en DTU, pero captura todo
  │
  └─ Estrategia por tablas (recomendada para reportes)
      • bcp tabla1 out tabla1.csv (por cada tabla/catálogo)
      • O: DuckDB COPY desde Azure SQL directo a Parquet
      • Ligero en DTU, exporta solo lo necesario
         │
         ▼
[4. Importar a motor local]
  │
  ├─ BACPAC → sqlpackage /Action:Import → SQL Server Express
  │
  └─ CSVs/Parquet → DuckDB → base .duckdb consultable
         │
         ▼
[5. Validar integridad]
  • Comparar row counts: origen vs destino
  • Verificar que tablas clave no estén vacías
  • Checksum de tablas críticas
         │
         ▼
[6. Limpiar snapshots antiguos]
  • Aplicar política de retención
  • Eliminar archivos fuera de ventana
         │
         ▼
[7. Notificar resultado]
  • Log local con resultado
  • Webhook / email en caso de fallo
  • Event Log de Windows para monitoreo
```

---

## 5. Herramientas Requeridas

### 5.1 Instalación en servidor local (Windows)

| Herramienta | Propósito | Instalación | Verificación |
|-------------|-----------|-------------|--------------|
| **sqlpackage** | Export/Import BACPAC | `winget install Microsoft.SqlPackage` | `sqlpackage /version` |
| **sqlcmd** (go-sqlcmd) | Queries ad-hoc, test de conexión | `winget install sqlcmd` | `sqlcmd --version` |
| **bcp** | Export de tablas individuales a CSV | Viene con ODBC Driver 18 for SQL Server | `bcp -v` |
| **ODBC Driver 18** | Conectividad TDS a Azure SQL | `winget install Microsoft.msodbcsql` | `odbcconf /a {configsysdsn "ODBC Driver 18 for SQL Server"}` |
| **DuckDB** | Consultas locales sobre Parquet/CSV | `winget install DuckDB.cli` | `duckdb --version` |
| **Azure CLI** (opcional) | Si se usa `az sql db export` o Key Vault | `winget install Microsoft.AzureCLI` | `az version` |
| **SQL Server Express** (opcional) | Motor local para restaurar BACPAC | `winget install Microsoft.SQLServer.2022.Express` | `sqlcmd -S localhost\SQLEXPRESS -Q "SELECT @@VERSION"` |

### 5.2 Script de instalación unificado

```powershell
# install-snapshot-tools.ps1
# Ejecutar como Administrador

Write-Host "=== Instalando herramientas para Azure SQL Snapshot ===" -ForegroundColor Cyan

$tools = @(
    @{Name="Microsoft.SqlPackage";     Check="sqlpackage /version"},
    @{Name="sqlcmd";                   Check="sqlcmd --version"},
    @{Name="Microsoft.msodbcsql";      Check="bcp -v"},
    @{Name="DuckDB.cli";              Check="duckdb --version"}
)

foreach ($tool in $tools) {
    Write-Host "`nInstalando $($tool.Name)..." -ForegroundColor Yellow
    winget install $tool.Name --accept-package-agreements --accept-source-agreements

    try {
        Invoke-Expression $tool.Check 2>&1 | Out-Null
        Write-Host "  OK" -ForegroundColor Green
    } catch {
        Write-Host "  Verificar manualmente: $($tool.Check)" -ForegroundColor Red
    }
}

Write-Host "`n=== Instalación completada ===" -ForegroundColor Cyan
```

---

## 6. Detalle Técnico por Método de Export

### 6.1 Método BACPAC — Snapshot completo de la base de datos

**Cuándo usar:** Se requiere una copia idéntica de toda la base, incluyendo schema (tablas, vistas, stored procedures, índices, constraints).

**Comando base:**
```powershell
sqlpackage `
    /Action:Export `
    /TargetFile:"C:\Snapshots\AzureSQL\NOMBRE_DB_2026-02-11_0800.bacpac" `
    /SourceServerName:"tcp:SERVIDOR.database.windows.net,1433" `
    /SourceDatabaseName:"NOMBRE_DB" `
    /SourceUser:"USUARIO_READONLY" `
    /SourcePassword:"CONTRASEÑA" `
    /p:Storage=File `
    /p:CommandTimeout=1200 `
    /p:LongRunningCommandTimeout=0 `
    /p:CompressionOption=Fast `
    /p:MaxParallelism=8 `
    /SourceEncryptConnection:True
```

**Parámetros clave:**

| Parámetro | Valor | Justificación |
|-----------|-------|---------------|
| `/p:Storage=File` | File | Usa disco en vez de RAM — obligatorio para DBs > 1 GB |
| `/p:CommandTimeout=1200` | 1200 seg | Timeout por query individual — 20 min por tabla grande |
| `/p:LongRunningCommandTimeout=0` | 0 (infinito) | Sin timeout global — evita cortes en exports largos |
| `/p:CompressionOption=Fast` | Fast | Balance entre velocidad y tamaño del .bacpac |
| `/p:MaxParallelism=8` | 8 | Threads de extracción — ajustar según DTU disponible |

**Para exportar solo tablas específicas** (reduce tiempo y DTU):
```powershell
sqlpackage /Action:Export `
    /TargetFile:"C:\Snapshots\catalogos.bacpac" `
    /SourceServerName:"tcp:SERVIDOR.database.windows.net,1433" `
    /SourceDatabaseName:"NOMBRE_DB" `
    /SourceUser:"USUARIO" /SourcePassword:"PASS" `
    /p:Storage=File `
    /p:TableData="dbo.Clientes" `
    /p:TableData="dbo.Productos" `
    /p:TableData="dbo.Categorias" `
    /p:TableData="dbo.Pedidos" `
    /p:TableData="dbo.Regiones"
```

> **Nota:** `/p:TableData` exporta el schema de TODAS las tablas pero solo los datos de las especificadas.

**Restaurar BACPAC a SQL Server Express local:**
```powershell
sqlpackage `
    /Action:Import `
    /SourceFile:"C:\Snapshots\AzureSQL\NOMBRE_DB_2026-02-11_0800.bacpac" `
    /TargetServerName:"localhost\SQLEXPRESS" `
    /TargetDatabaseName:"NOMBRE_DB_Local" `
    /p:Storage=File `
    /p:CommandTimeout=0
```

**Estimaciones de tiempo:**

| Tamaño DB | Tiempo Export | Tiempo Import | Espacio en disco |
|-----------|--------------|---------------|------------------|
| 1 GB | 3-8 min | 2-5 min | ~300 MB (.bacpac) |
| 5 GB | 15-30 min | 10-20 min | ~1.5 GB |
| 10 GB | 30-60 min | 20-40 min | ~3 GB |
| 50 GB | 2-5 hrs | 1-3 hrs | ~15 GB |

### 6.2 Método bcp — Export de tablas individuales como CSV

**Cuándo usar:** Solo se necesitan catálogos/tablas específicas para reportes. Es más rápido y consume menos DTU que BACPAC.

**Comando base (una tabla):**
```powershell
bcp "dbo.Clientes" out "C:\Snapshots\csv\Clientes.csv" `
    -S "SERVIDOR.database.windows.net" `
    -d "NOMBRE_DB" `
    -U "USUARIO" -P "PASS" `
    -c -t "," -r "\n" -q
```

**Export de múltiples tablas (script):**
```powershell
$config = @{
    Server   = "SERVIDOR.database.windows.net"
    Database = "NOMBRE_DB"
    User     = "USUARIO"
    Password = "PASS"
}

$timestamp = Get-Date -Format "yyyy-MM-dd_HHmm"
$outDir = "C:\Snapshots\csv\$timestamp"
New-Item -ItemType Directory -Path $outDir -Force | Out-Null

# Definir tablas a exportar — con query personalizable
$exports = @(
    @{ Name="Clientes";    Query="SELECT * FROM dbo.Clientes" },
    @{ Name="Productos";   Query="SELECT * FROM dbo.Productos" },
    @{ Name="Categorias";  Query="SELECT * FROM dbo.Categorias" },
    @{ Name="Pedidos";     Query="SELECT * FROM dbo.Pedidos WHERE FechaPedido >= DATEADD(month, -6, GETDATE())" },
    @{ Name="Regiones";    Query="SELECT * FROM dbo.Regiones" },
    @{ Name="Inventario";  Query="SELECT * FROM dbo.Inventario" }
)

$results = @()
foreach ($export in $exports) {
    $csvPath = Join-Path $outDir "$($export.Name).csv"
    $start = Get-Date

    bcp "$($export.Query)" queryout $csvPath `
        -S $config.Server -d $config.Database `
        -U $config.User -P $config.Password `
        -c -t "," -r "\n" -q 2>&1

    $elapsed = (Get-Date) - $start
    $size = if (Test-Path $csvPath) { [math]::Round((Get-Item $csvPath).Length / 1KB, 1) } else { 0 }

    $results += [PSCustomObject]@{
        Tabla    = $export.Name
        Estado   = if ($LASTEXITCODE -eq 0) { "OK" } else { "ERROR" }
        Tamaño   = "$($size) KB"
        Tiempo   = "$([math]::Round($elapsed.TotalSeconds, 1))s"
    }
}

$results | Format-Table -AutoSize
```

**Limitación de bcp:** No incluye headers en el CSV. Workaround: generarlos con sqlcmd o agregarlos en el paso de importación a DuckDB.

### 6.3 Método DuckDB directo — Export Azure SQL a Parquet (sin intermediarios)

**Cuándo usar:** Se quiere ir directo de Azure SQL a archivos Parquet consultables, sin pasar por CSV ni BACPAC. Requiere DuckDB >= 1.1.0 con la extensión `mssql` (community, experimental).

```sql
-- Desde DuckDB CLI
INSTALL mssql FROM community;
LOAD mssql;

-- Conectar a Azure SQL
ATTACH 'mssql://USUARIO:PASS@SERVIDOR.database.windows.net:1433?database=NOMBRE_DB&encrypt=true' AS azure;

-- Exportar tablas a Parquet con compresión
COPY (SELECT * FROM azure.dbo.Clientes) TO 'C:/Snapshots/parquet/Clientes.parquet' (FORMAT PARQUET, COMPRESSION ZSTD);
COPY (SELECT * FROM azure.dbo.Productos) TO 'C:/Snapshots/parquet/Productos.parquet' (FORMAT PARQUET, COMPRESSION ZSTD);
COPY (SELECT * FROM azure.dbo.Pedidos WHERE FechaPedido >= '2025-08-01') TO 'C:/Snapshots/parquet/Pedidos.parquet' (FORMAT PARQUET, COMPRESSION ZSTD);

-- Crear base DuckDB local consolidada
CREATE TABLE clientes AS SELECT * FROM 'C:/Snapshots/parquet/Clientes.parquet';
CREATE TABLE productos AS SELECT * FROM 'C:/Snapshots/parquet/Productos.parquet';
CREATE TABLE pedidos AS SELECT * FROM 'C:/Snapshots/parquet/Pedidos.parquet';
```

**Ventajas:**
- Un solo paso: Azure SQL → Parquet (sin CSV intermedio)
- Parquet es columnar — más compacto y más rápido para analytics que CSV
- DuckDB puede hacer JOINs entre archivos Parquet sin importarlos

**Riesgo:** La extensión `mssql` es comunitaria/experimental. Si falla, usar el pipeline bcp → CSV → DuckDB como fallback.

---

## 7. Motor de Consulta Local — Comparativa

Una vez que los datos están en el servidor local, se necesita un motor para consultarlos.

| Criterio | SQL Server Express | DuckDB | SQLite |
|----------|:-----------------:|:------:|:------:|
| **Instalación** | Pesada (~1.5 GB, servicio Windows) | Ligera (~30 MB, sin servicio) | Ligera (~2 MB) |
| **Límite de tamaño** | 10 GB por DB | Sin límite práctico | ~281 TB teórico |
| **Rendimiento analítico** | Bueno | Excelente (columnar) | Aceptable |
| **Compatibilidad T-SQL** | 100% | Parcial (DuckDB SQL) | No |
| **JOINs complejos** | Excelente | Excelente | Bueno |
| **Import desde BACPAC** | Nativo (sqlpackage) | No directo | No |
| **Import desde CSV** | BULK INSERT | `read_csv_auto()` nativo | `.import` |
| **Import desde Parquet** | No nativo | Nativo | No |
| **Uso desde Python** | pyodbc / sqlalchemy | duckdb (pip) | sqlite3 (built-in) |
| **Concurrencia** | Multi-usuario | Mono-escritor | Mono-escritor |
| **Costo** | $0 (Express) | $0 | $0 |

### Recomendación

| Caso de uso | Motor recomendado |
|-------------|-------------------|
| Necesitas compatibilidad T-SQL exacta con la DB original | SQL Server Express |
| Reportes analíticos, cruces entre catálogos, Python | **DuckDB** (recomendado) |
| Aplicación simple, una sola tabla a la vez | SQLite |
| Necesitas que el mismo código SQL corra en Azure y local | SQL Server Express |

**DuckDB es la opción recomendada** para el caso de reportes entre catálogos porque:
- No requiere servicio corriendo (se abre como un archivo)
- Motor columnar optimizado para analytics y aggregations
- Lee Parquet directamente sin importar — acceso inmediato
- Integración nativa con Python (`import duckdb`)
- Permite JOINs entre archivos Parquet de diferentes snapshots

---

## 8. Seguridad y Manejo de Credenciales

### Principios

1. **Nunca** almacenar credenciales en texto plano en scripts.
2. **Nunca** incluir credenciales en repositorios de código.
3. Usar mecanismos de almacenamiento encriptado del sistema operativo.
4. Rotar credenciales periódicamente (acordar con proveedor).

### Opciones de almacenamiento seguro (Windows)

#### Opción 1: Windows Credential Manager (recomendada para Task Scheduler)

```powershell
# Setup inicial (ejecutar una vez, manualmente)
Install-Module -Name CredentialManager -Scope CurrentUser -Force

New-StoredCredential `
    -Target "AzureSQL_MiServidor" `
    -Type Generic `
    -UserName "USUARIO_READONLY" `
    -Password "CONTRASEÑA" `
    -Persist LocalMachine

# En el script automatizado:
Import-Module CredentialManager
$cred = Get-StoredCredential -Target "AzureSQL_MiServidor"
$user = $cred.UserName
$pass = $cred.GetNetworkCredential().Password
```

#### Opción 2: DPAPI con Export-Clixml (más simple)

```powershell
# Setup inicial (ejecutar como el usuario que correrá la tarea)
Get-Credential | Export-Clixml -Path "C:\Scripts\Snapshots\.credentials.xml"

# En el script:
$cred = Import-Clixml -Path "C:\Scripts\Snapshots\.credentials.xml"
$user = $cred.UserName
$pass = $cred.GetNetworkCredential().Password
```

> **Nota:** El archivo `.credentials.xml` está encriptado con DPAPI — solo puede ser leído por el mismo usuario en la misma máquina.

#### Opción 3: Variables de entorno del sistema

```powershell
# Configurar (como Administrador)
[System.Environment]::SetEnvironmentVariable("AZURE_SQL_USER", "USUARIO", "Machine")
[System.Environment]::SetEnvironmentVariable("AZURE_SQL_PASS", "CONTRASEÑA", "Machine")

# En el script:
$user = $env:AZURE_SQL_USER
$pass = $env:AZURE_SQL_PASS
```

---

## 9. Automatización — Ejecución Periódica

### Configuración de Task Scheduler

```powershell
# register-snapshot-task.ps1
# Ejecutar como Administrador

$taskName = "AzureSQL-Snapshot"
$scriptPath = "C:\Scripts\Snapshots\Export-AzureSnapshot.ps1"

# Acción
$action = New-ScheduledTaskAction `
    -Execute "powershell.exe" `
    -Argument "-NoProfile -ExecutionPolicy Bypass -File `"$scriptPath`""

# Trigger: cada hora, comenzando a las 00:00
$trigger = New-ScheduledTaskTrigger `
    -Once -At "00:00" `
    -RepetitionInterval (New-TimeSpan -Hours 1) `
    -RepetitionDuration ([TimeSpan]::MaxValue)

# Settings
$settings = New-ScheduledTaskSettingsSet `
    -AllowStartIfOnBatteries `
    -DontStopIfGoingOnBatteries `
    -StartWhenAvailable `
    -RunOnlyIfNetworkAvailable `
    -RestartCount 2 `
    -RestartInterval (New-TimeSpan -Minutes 5) `
    -ExecutionTimeLimit (New-TimeSpan -Hours 2) `
    -MultipleInstances IgnoreNew

# Registrar
Register-ScheduledTask `
    -TaskName $taskName `
    -Action $action `
    -Trigger $trigger `
    -Settings $settings `
    -User "SYSTEM" `
    -RunLevel Highest `
    -Description "Snapshot periódico de Azure SQL Database a servidor local"

Write-Host "Tarea '$taskName' registrada. Ejecutará cada hora."
```

### Política de retención

| Tipo de snapshot | Retención | Limpieza |
|-----------------|-----------|----------|
| Horarios | 48 horas | Automática por el script |
| Diarios (snapshot de las 00:00) | 30 días | Automática |
| Semanales (domingo 00:00) | 12 semanas | Automática |
| Mensuales (día 1, 00:00) | 12 meses | Manual / revisión |

```powershell
# Función de limpieza incluida en el script principal
function Remove-OldSnapshots {
    param(
        [string]$Path,
        [int]$RetainHours = 48
    )

    $cutoff = (Get-Date).AddHours(-$RetainHours)
    Get-ChildItem -Path $Path -Recurse -File |
        Where-Object { $_.LastWriteTime -lt $cutoff } |
        ForEach-Object {
            Remove-Item $_.FullName -Force
            Write-Log "Limpieza: eliminado $($_.Name)"
        }
}
```

---

## 10. Validación End-to-End (OE-9)

### Prueba de integridad del snapshot

```powershell
# Después de cada export + import, validar:

# 1. Row count comparison
$tablas = @("Clientes", "Productos", "Categorias", "Pedidos")

foreach ($tabla in $tablas) {
    # Count en Azure (fuente)
    $countAzure = sqlcmd -S "SERVIDOR.database.windows.net" -d "NOMBRE_DB" `
        -U "USUARIO" -P "PASS" `
        -Q "SELECT COUNT(*) FROM dbo.$tabla" -h -1 -W

    # Count en local (snapshot)
    $countLocal = duckdb "C:\Snapshots\snapshot.duckdb" `
        "SELECT COUNT(*) FROM $tabla"

    $match = $countAzure.Trim() -eq $countLocal.Trim()
    Write-Host "$tabla`: Azure=$countAzure Local=$countLocal $(if($match){'OK'}else{'MISMATCH'})"
}
```

### Criterios de aceptación

- [ ] Todas las tablas definidas se exportaron sin error
- [ ] Row count de cada tabla coincide entre Azure y local (tolerancia: 0 para catálogos, <1% para tablas transaccionales en movimiento)
- [ ] Se puede ejecutar un query JOIN entre 2+ tablas del snapshot local
- [ ] El proceso completo (export + import + validación) termina dentro de la ventana de tiempo (< 1 hora para frecuencia horaria)
- [ ] Los snapshots antiguos se eliminan según la política de retención
- [ ] En caso de fallo, se genera notificación y el snapshot anterior permanece disponible

---

## 11. Plan de Implementación — Fases

### Fase 0: Negociación con proveedor (prerequisito)

**Objetivo:** Obtener acceso de lectura a la base de datos.

**Acciones:**
1. Enviar solicitud formal al proveedor pidiendo usuario SQL `db_datareader` (Escenario A)
2. Si se niega, negociar al menos connection string con acceso a vistas (Escenario B)
3. Si se niega completamente, solicitar exports periódicos a Blob Storage (Escenario C)
4. Documentar el acuerdo alcanzado

**Entregable:** Credenciales validadas o mecanismo de acceso acordado.

### Fase 1: Setup de infraestructura local

**Objetivo:** OE-2 y OE-3.

**Acciones:**
1. Instalar herramientas CLI (`sqlpackage`, `sqlcmd`, `bcp`, `DuckDB`)
2. Configurar credenciales en Credential Manager
3. Validar conectividad TCP al servidor Azure SQL
4. Ejecutar query de prueba: `SELECT @@VERSION`

**Entregable:** Conexión funcional, herramientas instaladas.

### Fase 2: Diseño y primer export manual

**Objetivo:** OE-4 y OE-5 (parcial).

**Acciones:**
1. Catalogar tablas disponibles con `sqlcmd`:
   ```sql
   SELECT TABLE_SCHEMA, TABLE_NAME, TABLE_TYPE
   FROM INFORMATION_SCHEMA.TABLES
   ORDER BY TABLE_SCHEMA, TABLE_NAME;
   ```
2. Estimar tamaño de cada tabla:
   ```sql
   SELECT s.name + '.' + t.name AS tabla,
          SUM(p.rows) AS filas,
          SUM(a.total_pages) * 8 / 1024 AS tamano_MB
   FROM sys.tables t
   INNER JOIN sys.schemas s ON t.schema_id = s.schema_id
   INNER JOIN sys.indexes i ON t.object_id = i.object_id
   INNER JOIN sys.partitions p ON i.object_id = p.object_id AND i.index_id = p.index_id
   INNER JOIN sys.allocation_units a ON p.partition_id = a.container_id
   WHERE i.index_id <= 1
   GROUP BY s.name, t.name
   ORDER BY tamano_MB DESC;
   ```
3. Decidir estrategia: BACPAC completo vs. tablas selectas
4. Ejecutar primer export manualmente
5. Verificar resultado

**Entregable:** Primer snapshot local funcional, matriz de tablas documentada.

### Fase 3: Pipeline automatizado

**Objetivo:** OE-5 (completo), OE-6, OE-7.

**Acciones:**
1. Desarrollar script PowerShell completo con logging, retry y notificaciones
2. Configurar import a motor local (DuckDB o SQL Express)
3. Registrar tarea en Task Scheduler
4. Ejecutar 24 horas de prueba monitoreada

**Entregable:** Pipeline automatizado operando.

### Fase 4: Validación y hardening

**Objetivo:** OE-8, OE-9.

**Acciones:**
1. Implementar validación de integridad post-import
2. Configurar política de retención
3. Probar escenarios de fallo (red caída, credenciales expiradas, disco lleno)
4. Documentar runbook de operación
5. Generar primer reporte real desde el snapshot local

**Entregable:** Sistema validado y documentado, primer reporte de impacto generado.

---

## 12. Consideraciones Importantes

### Legales y contractuales

- **Verificar contrato:** Confirmar que se tiene derecho contractual a acceder y copiar los datos.
- **Propiedad de datos:** Los datos generados en la operación del negocio son propiedad de la organización, no del proveedor. Esto debe estar claro en el contrato.
- **SLA de acceso:** El proveedor debería tener obligación contractual de facilitar acceso a los datos propios.
- **Portabilidad:** Si el contrato no lo especifica, negociar cláusula de portabilidad de datos.

### Técnicas

- **Impacto en producción:** El export consume DTU de la base en Azure. Con el proveedor, acordar ventanas de export o que habiliten una read replica.
- **Ancho de banda:** Un export de 5 GB sobre internet puede tardar 10-30 minutos dependiendo del enlace. Considerar horarios de bajo tráfico.
- **Consistencia transaccional:** Un export con `bcp` tabla por tabla NO es transaccionalmente consistente entre tablas. Para reportes periódicos esto es aceptable. Para consistencia estricta, usar BACPAC o solicitar un DATABASE COPY.
- **Cambios de schema:** Si el proveedor modifica la estructura de tablas, el script de export puede fallar. Implementar alertas de detección de cambios de schema.

### De continuidad

- **Qué pasa si el proveedor revoca el acceso:** Los snapshots locales existentes permanecen disponibles como último recurso.
- **Qué pasa si el servidor local falla:** Configurar los snapshots en un segundo disco o ruta de backup.
- **Qué pasa si cambia la IP del servidor local:** Solicitar actualización de firewall al proveedor, o usar un servicio de IP fija / VPN.

---

## 13. Fuentes de Referencia

### Documentación oficial Microsoft
- [SqlPackage Export](https://learn.microsoft.com/en-us/sql/tools/sqlpackage/sqlpackage-export)
- [Export Azure SQL Database to BACPAC](https://learn.microsoft.com/en-us/azure/azure-sql/database/database-export)
- [Azure SQL Automated Backups](https://learn.microsoft.com/en-us/azure/azure-sql/database/automated-backups-overview)
- [bcp Utility](https://learn.microsoft.com/en-us/sql/tools/bcp-utility)
- [go-sqlcmd (modern CLI)](https://learn.microsoft.com/en-us/sql/tools/sqlcmd/sqlcmd-utility)
- [Azure SQL Firewall Rules](https://learn.microsoft.com/en-us/azure/azure-sql/database/firewall-configure)
- [Database Copy - Azure SQL](https://learn.microsoft.com/en-us/azure/azure-sql/database/database-copy)

### DuckDB
- [DuckDB mssql Extension](https://duckdb.org/community_extensions/extensions/mssql)
- [DuckDB Azure Extension](https://duckdb.org/docs/stable/core_extensions/azure)
- [DuckDB Parquet Import](https://duckdb.org/docs/stable/guides/file_formats/parquet_import)

### Guía técnica detallada (referencia interna)
- `%HOME%\Projects\docs\azure-sql-snapshot-guide.md` — 1800+ líneas con todos los comandos, flags, autenticación, Docker, restic, y optimización de costos.
