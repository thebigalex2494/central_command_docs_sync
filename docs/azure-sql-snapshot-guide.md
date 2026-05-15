# Azure SQL Database: Snapshots Remotos via CLI - Guia Completa

> Investigacion exhaustiva sobre como generar y descargar snapshots periodicos desde Azure SQL Database (PaaS) de forma remota usando CLI.

---

## Tabla de Contenidos

1. [sqlpackage /Action:Export (BACPAC)](#1-sqlpackage-actionexport-bacpac)
2. [az sql db export (Azure CLI)](#2-az-sql-db-export-azure-cli)
3. [bcp utility](#3-bcp-utility)
4. [sqlcmd](#4-sqlcmd)
5. [Consideraciones Azure SQL Database](#5-consideraciones-azure-sql-database)
6. [Integracion DuckDB](#6-integracion-duckdb)
7. [Automatizacion practica (Windows)](#7-automatizacion-practica-windows)
8. [Restaurar BACPAC localmente](#8-restaurar-bacpac-localmente)
9. [Optimizacion de costos](#9-optimizacion-de-costos)

---

## 1. sqlpackage /Action:Export (BACPAC)

### Que es

`sqlpackage` es una herramienta CLI de Microsoft que exporta una Azure SQL Database como archivo `.bacpac` (ZIP con schema + datos). Es la **herramienta recomendada por Microsoft para produccion** por su rendimiento y control.

**Operacion client-side**: la herramienta corre en tu maquina y extrae datos via conexion SQL.

### Instalacion

```powershell
# Via winget (Windows)
winget install Microsoft.SqlPackage

# Via dotnet tool (cross-platform)
dotnet tool install -g microsoft.sqlpackage

# Verificar instalacion
sqlpackage /version
```

### Sintaxis Completa

```
sqlpackage /Action:Export {parametros de conexion} {propiedades} /TargetFile:{ruta.bacpac}
```

### Autenticacion: SQL Authentication

```powershell
sqlpackage `
    /Action:Export `
    /TargetFile:"C:\Backups\mydb_2026-02-11.bacpac" `
    /SourceServerName:"tcp:myserver.database.windows.net,1433" `
    /SourceDatabaseName:"MyDatabase" `
    /SourceUser:"sqladmin" `
    /SourcePassword:"MyStr0ngP@ss!" `
    /p:Storage=File `
    /p:CommandTimeout=1200 `
    /p:LongRunningCommandTimeout=0
```

### Autenticacion: Connection String

```powershell
sqlpackage `
    /Action:Export `
    /TargetFile:"C:\Backups\mydb.bacpac" `
    /SourceConnectionString:"Server=tcp:myserver.database.windows.net,1433;Initial Catalog=MyDatabase;Persist Security Info=False;User ID=sqladmin;Password=MyStr0ngP@ss!;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"
```

### Autenticacion: Microsoft Entra ID (Azure AD) - Managed Identity

```powershell
# Desde una Azure VM o servicio con Managed Identity configurada
sqlpackage `
    /Action:Export `
    /TargetFile:"C:\Backups\mydb.bacpac" `
    /SourceConnectionString:"Server=tcp:myserver.database.windows.net,1433;Initial Catalog=MyDatabase;Authentication=Active Directory Managed Identity;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"
```

### Autenticacion: Microsoft Entra ID - User-Assigned Managed Identity

```powershell
sqlpackage `
    /Action:Export `
    /TargetFile:"C:\Backups\mydb.bacpac" `
    /SourceConnectionString:"Server=tcp:myserver.database.windows.net,1433;Initial Catalog=MyDatabase;Authentication=Active Directory Default;User Id=<client-id-of-user-assigned-identity>;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"
```

### Autenticacion: Access Token (Entra ID)

```powershell
# Paso 1: Obtener access token via Azure CLI
$token = az account get-access-token --resource https://database.windows.net/ --query accessToken -o tsv

# Paso 2: Usar el token con sqlpackage
sqlpackage `
    /Action:Export `
    /TargetFile:"C:\Backups\mydb.bacpac" `
    /SourceServerName:"tcp:myserver.database.windows.net,1433" `
    /SourceDatabaseName:"MyDatabase" `
    /AccessToken:$token `
    /p:Storage=File
```

### Autenticacion: Universal (MFA interactivo)

```powershell
sqlpackage `
    /Action:Export `
    /TargetFile:"C:\Backups\mydb.bacpac" `
    /UniversalAuthentication:True `
    /SourceConnectionString:"Server=tcp:myserver.database.windows.net,1433;Initial Catalog=MyDatabase;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;" `
    /TenantId:"midominio.onmicrosoft.com"
```

### Propiedades Clave (/p:)

| Propiedad | Tipo | Default | Descripcion |
|-----------|------|---------|-------------|
| `CommandTimeout` | INT32 | 60 | Timeout en segundos para cada query SQL |
| `LongRunningCommandTimeout` | INT32 | 0 | Timeout para operaciones largas (0 = infinito) |
| `DatabaseLockTimeout` | INT32 | 60 | Timeout de lock (-1 = infinito) |
| `Storage` | Enum | Memory | `File` o `Memory`. Usar `File` para DBs grandes |
| `CompressionOption` | Enum | Normal | `Normal`, `Maximum`, `Fast`, `SuperFast`, `NotCompressed` |
| `TableData` | STRING | (todas) | Tabla especifica a exportar. Repetible |
| `TempDirectoryForTableData` | STRING | - | Directorio temporal para buffer de datos |
| `VerifyExtraction` | BOOLEAN | True | Validar schema extraido |
| `MaxParallelism` | INT | 8 | Grado de paralelismo |
| `HashObjectNamesInLogs` | BOOLEAN | False | Anonimizar nombres en logs |

### Exportar Solo Tablas Especificas

```powershell
# Exportar solo tablas de catalogo/dimension (schema + datos seleccionados)
sqlpackage `
    /Action:Export `
    /TargetFile:"C:\Backups\mydb_catalogs.bacpac" `
    /SourceServerName:"tcp:myserver.database.windows.net,1433" `
    /SourceDatabaseName:"MyDatabase" `
    /SourceUser:"sqladmin" `
    /SourcePassword:"MyStr0ngP@ss!" `
    /p:TableData="dbo.Customers" `
    /p:TableData="dbo.Products" `
    /p:TableData="dbo.Categories" `
    /p:TableData="dbo.Regions" `
    /p:Storage=File `
    /p:CommandTimeout=600
```

**Nota importante**: `/p:TableData` incluye el schema de TODAS las tablas, pero solo exporta datos de las tablas especificadas. No hay opcion inversa para excluir tablas.

### Configuracion de Timeouts para DBs Grandes

```powershell
# Para bases de datos > 50 GB
sqlpackage `
    /Action:Export `
    /TargetFile:"C:\Backups\large_db.bacpac" `
    /SourceServerName:"tcp:myserver.database.windows.net,1433" `
    /SourceDatabaseName:"LargeDB" `
    /SourceUser:"sqladmin" `
    /SourcePassword:"MyStr0ngP@ss!" `
    /p:Storage=File `
    /p:CommandTimeout=0 `
    /p:LongRunningCommandTimeout=0 `
    /p:DatabaseLockTimeout=-1 `
    /p:TempDirectoryForTableData="D:\Temp" `
    /p:MaxParallelism=16 `
    /p:CompressionOption=Fast `
    /SourceTimeout:300 `
    /Diagnostics:True `
    /DiagnosticsFile:"C:\Logs\export_diag.log"
```

### Limitaciones de sqlpackage Export

| Limitacion | Detalle |
|------------|---------|
| **Rendimiento optimo** | Bases de datos < 200 GB |
| **Consistencia transaccional** | No garantizada en DB activa (exporta objeto por objeto) |
| **Disco local** | Necesita hasta 3x el tamano de la DB en disco |
| **Tablas sin clustered index** | Extremadamente lentas, pueden fallar tras 6-12 hrs |
| **Paralelismo limitado** | Default 8 threads, puede no ser suficiente |
| **Network latency** | Ejecutar desde VM en la misma region que la DB |

### Best Practice: Exportar desde una Copia

```powershell
# 1. Crear copia transaccionalmente consistente
# (ver seccion de Database Copy)

# 2. Exportar desde la copia
sqlpackage /a:Export /tf:"C:\Backups\mydb.bacpac" `
    /ssn:"tcp:myserver.database.windows.net" /sdn:"MyDatabase_Copy" `
    /su:"sqladmin" /sp:"MyStr0ngP@ss!" /p:Storage=File

# 3. Eliminar la copia
az sql db delete -g MyRG -s myserver -n MyDatabase_Copy --yes
```

---

## 2. az sql db export (Azure CLI)

### Que es

`az sql db export` es una operacion **server-side**: Azure ejecuta el export en sus propios VMs y deposita el BACPAC directamente en Azure Blob Storage. Tu maquina local no participa en la transferencia de datos.

### Diferencias Clave: az sql db export vs sqlpackage

| Aspecto | `az sql db export` (server-side) | `sqlpackage` (client-side) |
|---------|----------------------------------|----------------------------|
| **Ejecucion** | Azure VMs internas | Tu maquina/VM |
| **Destino** | Azure Blob Storage (obligatorio) | Archivo local o Azure Blob |
| **Disco disponible** | 450 GB max | Sin limite (tu disco) |
| **Timeout** | 20 hrs max (puede cancelarse) | Sin limite |
| **DTU impact** | Menor (server-side) | Mayor (lecturas remotas) |
| **Paralelismo** | Limitado | Configurable |
| **Consistencia** | Misma (no transaccional de DB activa) | Misma |
| **DB > 150 GB** | Puede fallar por disco | Recomendado |
| **Prerequisito** | Public access habilitado en server | Firewall rule para tu IP |
| **MFA** | No soportado | Soportado (Universal Auth) |
| **Private Link** | Preview | No soportado |

### Prerequisitos

```powershell
# 1. Login a Azure CLI
az login

# 2. Crear Storage Account y Container (si no existe)
az storage account create `
    --name "mybackupstorage" `
    --resource-group "MyRG" `
    --location "eastus" `
    --sku "Standard_LRS"

az storage container create `
    --name "bacpacs" `
    --account-name "mybackupstorage"
```

### Sintaxis con Storage Account Key

```powershell
# Obtener storage key
$storageKey = az storage account keys list `
    --account-name "mybackupstorage" `
    --resource-group "MyRG" `
    --query "[0].value" -o tsv

# Exportar
az sql db export `
    --server "myserver" `
    --name "MyDatabase" `
    --resource-group "MyRG" `
    --admin-user "sqladmin" `
    --admin-password "MyStr0ngP@ss!" `
    --storage-key-type "StorageAccessKey" `
    --storage-key $storageKey `
    --storage-uri "https://mybackupstorage.blob.core.windows.net/bacpacs/mydb_$(Get-Date -Format 'yyyy-MM-dd_HHmm').bacpac"
```

### Sintaxis con SAS Token

```powershell
# Generar SAS token con permisos de lectura y escritura (AMBOS requeridos)
$expiry = (Get-Date).AddHours(4).ToUniversalTime().ToString("yyyy-MM-ddTHH:mm:ssZ")

$sasToken = az storage blob generate-sas `
    --account-name "mybackupstorage" `
    --container-name "bacpacs" `
    --name "mydb_export.bacpac" `
    --permissions rw `
    --expiry $expiry `
    --output tsv

# Exportar con SAS
az sql db export `
    --server "myserver" `
    --name "MyDatabase" `
    --resource-group "MyRG" `
    --admin-user "sqladmin" `
    --admin-password "MyStr0ngP@ss!" `
    --storage-key-type "SharedAccessKey" `
    --storage-key "?$sasToken" `
    --storage-uri "https://mybackupstorage.blob.core.windows.net/bacpacs/mydb_export.bacpac"
```

**IMPORTANTE**: El SAS token requiere permisos `rw` (read + write). Solo `w` causara error "storage account cannot be accessed".

### Monitorear Estado de Export

```powershell
# Listar operaciones en curso
az sql db operation list `
    --server "myserver" `
    --database "MyDatabase" `
    --resource-group "MyRG" `
    --output table

# El export retorna un operation status link
# Consultar status con PowerShell
$exportRequest = New-AzSqlDatabaseExport `
    -ResourceGroupName "MyRG" `
    -ServerName "myserver" `
    -DatabaseName "MyDatabase" `
    -StorageKeyType "StorageAccessKey" `
    -StorageKey $storageKey `
    -StorageUri "https://mybackupstorage.blob.core.windows.net/bacpacs/mydb.bacpac" `
    -AdministratorLogin "sqladmin" `
    -AdministratorLoginPassword (ConvertTo-SecureString "MyStr0ngP@ss!" -AsPlainText -Force)

# Polling loop
do {
    $status = Get-AzSqlDatabaseImportExportStatus -OperationStatusLink $exportRequest.OperationStatusLink
    Write-Host "Status: $($status.Status) - $($status.StatusMessage)"
    Start-Sleep -Seconds 10
} while ($status.Status -eq "InProgress")
```

### Cancelar Export

```powershell
# Via PowerShell
Stop-AzSqlDatabaseActivity `
    -ResourceGroupName "MyRG" `
    -ServerName "myserver" `
    -DatabaseName "MyDatabase" `
    -OperationId $operationId
```

### Descargar BACPAC desde Blob Storage

```powershell
# Despues del export server-side, descargar a local
az storage blob download `
    --account-name "mybackupstorage" `
    --container-name "bacpacs" `
    --name "mydb_export.bacpac" `
    --file "C:\Backups\mydb_export.bacpac"
```

### Limitaciones de az sql db export

| Limitacion | Detalle |
|------------|---------|
| **Requiere public access** | El SQL Server debe tener public network access habilitado |
| **450 GB disco max** | Las VMs de Azure tienen espacio limitado |
| **20 hrs timeout** | Operaciones mayores pueden ser canceladas automaticamente |
| **No MFA** | No soporta Microsoft Entra ID con MFA |
| **VMs compartidos** | Recursos compartidos por region; delays posibles |
| **Storage behind firewall** | No soportado actualmente |
| **Immutable storage** | No soportado |
| **Private Link** | En preview |

### Cuando Usar Cada Uno

- **`az sql db export`**: DBs pequenas/medianas, automatizacion simple, cuando ya tienes Blob Storage
- **`sqlpackage`**: DBs > 150 GB, produccion, necesitas control fino, paralelismo, tablas especificas

---

## 3. bcp utility

### Que es

`bcp` (Bulk Copy Program) exporta tablas/vistas individuales como archivos flat (CSV, TSV, etc). Ideal para extraer tablas de catalogo o dimensiones sin necesidad de un BACPAC completo.

### Instalacion

```powershell
# Viene con SQL Server tools, o instalar via:
# 1. Microsoft ODBC Driver 17/18 for SQL Server
# 2. Microsoft Command Line Utilities for SQL Server
# Verificar
bcp -v
```

### Exportar Tabla como CSV

```powershell
# Sintaxis basica: bcp {tabla|query} out {archivo} -S {server} -d {db} -U {user} -P {pass} -c -t ","
bcp "dbo.Customers" out "C:\Exports\customers.csv" `
    -S "myserver.database.windows.net" `
    -d "MyDatabase" `
    -U "sqladmin" `
    -P "MyStr0ngP@ss!" `
    -c -t "," -r "\n"
```

### Exportar con Query Personalizada

```powershell
# Usar queryout para queries personalizadas
bcp "SELECT CustomerID, Name, Email, Region FROM dbo.Customers WHERE IsActive = 1" queryout "C:\Exports\active_customers.csv" `
    -S "myserver.database.windows.net" `
    -d "MyDatabase" `
    -U "sqladmin" `
    -P "MyStr0ngP@ss!" `
    -c -t "," -r "\n"
```

### Exportar con Headers (truco)

```powershell
# bcp no incluye headers por defecto. Workaround:
# 1. Exportar headers primero
bcp "SELECT 'CustomerID','Name','Email','Region'" queryout "C:\Exports\headers.csv" `
    -S "myserver.database.windows.net" `
    -d "MyDatabase" `
    -U "sqladmin" `
    -P "MyStr0ngP@ss!" `
    -c -t ","

# 2. Exportar datos
bcp "SELECT CustomerID, Name, Email, Region FROM dbo.Customers" queryout "C:\Exports\data.csv" `
    -S "myserver.database.windows.net" `
    -d "MyDatabase" `
    -U "sqladmin" `
    -P "MyStr0ngP@ss!" `
    -c -t ","

# 3. Concatenar
Get-Content "C:\Exports\headers.csv", "C:\Exports\data.csv" | Set-Content "C:\Exports\customers_with_headers.csv"
```

### Autenticacion con Microsoft Entra ID (Azure AD)

```powershell
# Requiere Microsoft ODBC Driver 17+ (version 15.0.4298.1+)
# Flag -G habilita Azure AD auth

# Azure AD Password
bcp "dbo.Customers" out "C:\Exports\customers.csv" `
    -S "myserver.database.windows.net" `
    -d "MyDatabase" `
    -G `
    -U "user@domain.com" `
    -P "AzureADPassword!" `
    -c -t ","

# Azure AD Interactive (MFA)
bcp "dbo.Customers" out "C:\Exports\customers.csv" `
    -S "myserver.database.windows.net" `
    -d "MyDatabase" `
    -G `
    -c -t ","
```

### Exportar Multiples Tablas de Catalogo (script)

```powershell
$server = "myserver.database.windows.net"
$db = "MyDatabase"
$user = "sqladmin"
$pass = "MyStr0ngP@ss!"
$outDir = "C:\Exports\$(Get-Date -Format 'yyyy-MM-dd')"
New-Item -ItemType Directory -Path $outDir -Force | Out-Null

$tables = @(
    "dbo.Categories",
    "dbo.Regions",
    "dbo.Products",
    "dbo.Suppliers",
    "dbo.PaymentMethods",
    "dbo.StatusCodes"
)

foreach ($table in $tables) {
    $filename = ($table -replace '\.', '_') + ".csv"
    $filepath = Join-Path $outDir $filename
    Write-Host "Exporting $table to $filepath..."

    bcp $table out $filepath `
        -S $server -d $db -U $user -P $pass `
        -c -t "," -r "\n" -q

    if ($LASTEXITCODE -eq 0) {
        Write-Host "  OK - $(Get-Item $filepath | Select-Object -ExpandProperty Length) bytes"
    } else {
        Write-Warning "  FAILED to export $table"
    }
}
```

### Flags Importantes de bcp

| Flag | Descripcion |
|------|-------------|
| `-S` | Server name |
| `-d` | Database name |
| `-U` | Username (SQL auth) |
| `-P` | Password |
| `-G` | Azure AD authentication |
| `-c` | Character mode (text) |
| `-t ","` | Field terminator (comma for CSV) |
| `-r "\n"` | Row terminator |
| `-q` | Quoted identifiers |
| `-T` | Trusted connection (Windows Auth, no aplica para Azure SQL PaaS) |
| `-b {n}` | Batch size (filas por batch) |
| `-a {n}` | Network packet size (4096-65535) |
| `-e {file}` | Error file |

---

## 4. sqlcmd

### Legacy sqlcmd vs Modern go-sqlcmd

| Aspecto | Legacy sqlcmd (ODBC) | go-sqlcmd (moderno) |
|---------|---------------------|---------------------|
| **Lenguaje** | C/C++ | Go |
| **Instalacion** | SQL Server tools | `winget install sqlcmd` |
| **Plataformas** | Windows | Windows, macOS, Linux, containers |
| **Auth Azure AD** | Limitada | Completa (6 metodos) |
| **Formato output** | Texto | Texto, vertical |
| **Creacion local** | No | Si (`sqlcmd create mssql`) |
| **TDS 8.0** | SQL Server 2025 | Soportado |

### Instalar go-sqlcmd

```powershell
winget install sqlcmd
# o
choco install sqlcmd
```

### Conectar a Azure SQL Database

```powershell
# SQL Authentication
sqlcmd -S "myserver.database.windows.net" -d "MyDatabase" -U "sqladmin" -P "MyStr0ngP@ss!"

# Azure AD - Default (usa az login, VS Code, etc.)
sqlcmd -S "myserver.database.windows.net" -d "MyDatabase" --authentication-method ActiveDirectoryDefault

# Azure AD - Password
sqlcmd -S "myserver.database.windows.net" -d "MyDatabase" -G ActiveDirectoryPassword -U "user@domain.com" -P "pass"

# Azure AD - Managed Identity
sqlcmd -S "myserver.database.windows.net" -d "MyDatabase" -G ActiveDirectoryManagedIdentity

# Azure AD - Service Principal
sqlcmd -S "myserver.database.windows.net" -d "MyDatabase" -G ActiveDirectoryServicePrincipal -U "<client-id>" -P "<client-secret>"
```

### Metodos de Auth de go-sqlcmd

| Metodo `-G` | Descripcion |
|-------------|-------------|
| `SqlAuthentication` | SQL user/password clasico |
| `ActiveDirectoryDefault` | Usa DefaultAzureCredential (az login, env vars, etc.) |
| `ActiveDirectoryIntegrated` | Windows integrated + Azure AD |
| `ActiveDirectoryPassword` | Azure AD user + password |
| `ActiveDirectoryServicePrincipal` | Service Principal (client_id + secret) |
| `ActiveDirectoryManagedIdentity` | Managed Identity (system o user-assigned) |

### Ejecutar Query y Exportar Resultados

```powershell
# Query simple con output a archivo
sqlcmd -S "myserver.database.windows.net" -d "MyDatabase" `
    -U "sqladmin" -P "MyStr0ngP@ss!" `
    -Q "SELECT TOP 100 * FROM dbo.Customers" `
    -o "C:\Exports\customers.txt" `
    -s "," -W

# Ejecutar script SQL
sqlcmd -S "myserver.database.windows.net" -d "MyDatabase" `
    -U "sqladmin" -P "MyStr0ngP@ss!" `
    -i "C:\Scripts\export_query.sql" `
    -o "C:\Exports\results.csv" `
    -s "," -W -h -1
```

### Flags Utiles de sqlcmd

| Flag | Descripcion |
|------|-------------|
| `-S` | Server |
| `-d` | Database |
| `-U` | Username |
| `-P` | Password |
| `-G` | Auth method (go-sqlcmd) |
| `-Q` | Query inline (ejecuta y sale) |
| `-i` | Input file (script SQL) |
| `-o` | Output file |
| `-s` | Column separator |
| `-W` | Trim trailing spaces |
| `-h -1` | Sin headers |
| `-w {n}` | Screen width |
| `-k` | Remove control characters |
| `--vertical` | Formato vertical (go-sqlcmd) |

### Connection String Formats para Azure SQL

```
# ADO.NET style
Server=tcp:myserver.database.windows.net,1433;Initial Catalog=MyDatabase;Persist Security Info=False;User ID=sqladmin;Password={pass};MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;

# ODBC style
Driver={ODBC Driver 18 for SQL Server};Server=tcp:myserver.database.windows.net,1433;Database=MyDatabase;Uid=sqladmin;Pwd={pass};Encrypt=yes;TrustServerCertificate=no;Connection Timeout=30;

# Azure AD Default (passwordless)
Server=tcp:myserver.database.windows.net,1433;Initial Catalog=MyDatabase;Authentication=Active Directory Default;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;

# Managed Identity
Server=tcp:myserver.database.windows.net,1433;Initial Catalog=MyDatabase;Authentication=Active Directory Managed Identity;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;
```

---

## 5. Consideraciones Azure SQL Database

### 5.1 Reglas de Firewall

Para conectarte a Azure SQL Database desde tu maquina, necesitas una regla de firewall.

```powershell
# Obtener tu IP publica
$myIp = (Invoke-RestMethod -Uri "https://api.ipify.org")
Write-Host "Mi IP: $myIp"

# Crear regla de firewall
az sql server firewall-rule create `
    --resource-group "MyRG" `
    --server "myserver" `
    --name "MyDevMachine" `
    --start-ip-address $myIp `
    --end-ip-address $myIp

# Permitir acceso desde servicios Azure (para az sql db export)
az sql server firewall-rule create `
    --resource-group "MyRG" `
    --server "myserver" `
    --name "AllowAzureServices" `
    --start-ip-address "0.0.0.0" `
    --end-ip-address "0.0.0.0"

# Listar reglas existentes
az sql server firewall-rule list `
    --resource-group "MyRG" `
    --server "myserver" `
    --output table

# Eliminar regla
az sql server firewall-rule delete `
    --resource-group "MyRG" `
    --server "myserver" `
    --name "MyDevMachine"
```

### 5.2 Impacto DTU/vCore de Operaciones de Export

| Operacion | Impacto en DTU/CPU | Notas |
|-----------|-------------------|-------|
| `sqlpackage Export` (client) | **ALTO** | Lee toda la DB via SQL, consume DTU de lectura |
| `az sql db export` (server) | **MEDIO** | Azure gestiona internamente pero consume recursos |
| `bcp` (tabla individual) | **BAJO-MEDIO** | Solo lee la tabla especificada |
| `sqlcmd` (query) | **BAJO** | Depende de la query |
| `CREATE DATABASE ... AS COPY` | **BAJO** | Usa geo-replication technology internamente |

**Recomendaciones para minimizar impacto**:
- Escalar temporalmente la DB antes del export (e.g., S0 -> S3 o P1)
- Programar exports en horas no pico
- Usar `CREATE DATABASE ... AS COPY OF` y exportar desde la copia
- Exportar solo tablas necesarias con `/p:TableData`
- Asegurar que tablas grandes tengan clustered indexes

### 5.3 Microsoft Entra ID (Azure AD) desde CLI

```powershell
# 1. Login a Azure
az login

# 2. Obtener access token para Azure SQL
$token = az account get-access-token `
    --resource "https://database.windows.net/" `
    --query accessToken -o tsv

# 3. Usar con sqlpackage
sqlpackage /a:Export /tf:"backup.bacpac" `
    /ssn:"tcp:myserver.database.windows.net" /sdn:"MyDB" `
    /at:$token /p:Storage=File

# 4. O con sqlcmd (go-sqlcmd)
sqlcmd -S "myserver.database.windows.net" -d "MyDB" `
    --authentication-method ActiveDirectoryDefault `
    -Q "SELECT @@VERSION"

# Nota: Timeout recomendado >= 30 segundos para Entra ID
```

### 5.4 Managed Identity Authentication

Para automatizacion sin credenciales (Azure VM, Azure Functions, GitHub Actions):

```powershell
# Desde una Azure VM con System-Assigned Managed Identity

# sqlpackage con Managed Identity
sqlpackage /a:Export /tf:"backup.bacpac" `
    /scs:"Server=tcp:myserver.database.windows.net;Database=MyDB;Authentication=Active Directory Managed Identity;Encrypt=True;"

# Obtener token con Managed Identity (desde Azure VM)
$tokenResponse = Invoke-RestMethod -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fdatabase.windows.net%2F' `
    -Headers @{Metadata="true"}
$token = $tokenResponse.access_token

# Usar token con sqlpackage
sqlpackage /a:Export /tf:"backup.bacpac" `
    /ssn:"tcp:myserver.database.windows.net" /sdn:"MyDB" `
    /at:$token
```

### 5.5 Private Endpoints y Conectividad

| Metodo de Conexion | Soporte Export | Notas |
|-------------------|---------------|-------|
| **Public endpoint** | Completo | Requiere firewall rules |
| **Private endpoint** | Parcial | `az sql db export` en preview; sqlpackage funciona si hay ruta de red |
| **Service endpoint** | Si | Desde VNet con service endpoints habilitados |
| **VPN/ExpressRoute** | Si | sqlpackage desde on-prem via VPN |

### 5.6 Azure SQL Database Copy (Snapshot)

La alternativa mas ligera a un BACPAC completo: crear una copia transaccionalmente consistente.

```sql
-- T-SQL: Ejecutar en master database
-- Mismo servidor
CREATE DATABASE MyDB_Snapshot_20260211 AS COPY OF MyDatabase;

-- Diferente servidor (ejecutar en master del servidor destino)
CREATE DATABASE MyDB_Snapshot AS COPY OF sourceserver.MyDatabase;

-- Copia a elastic pool
CREATE DATABASE MyDB_Snapshot
AS COPY OF MyDatabase
(SERVICE_OBJECTIVE = ELASTIC_POOL(name = my_pool));
```

```powershell
# Azure CLI
az sql db copy `
    --name "MyDatabase" `
    --resource-group "MyRG" `
    --server "myserver" `
    --dest-name "MyDB_Snapshot_$(Get-Date -Format 'yyyyMMdd_HHmm')" `
    --dest-resource-group "MyRG" `
    --dest-server "myserver"

# PowerShell
New-AzSqlDatabaseCopy `
    -ResourceGroupName "MyRG" `
    -ServerName "myserver" `
    -DatabaseName "MyDatabase" `
    -CopyResourceGroupName "MyRG" `
    -CopyServerName "myserver" `
    -CopyDatabaseName "MyDB_Snapshot_$(Get-Date -Format 'yyyyMMdd_HHmm')"
```

**Monitorear progreso**:
```sql
-- En master database
SELECT * FROM sys.dm_database_copies;
SELECT name, state_desc FROM sys.databases WHERE name LIKE 'MyDB_Snapshot%';
-- state_desc = 'COPYING' -> en progreso
-- state_desc = 'ONLINE' -> completado
```

**Ventajas sobre BACPAC**:
- Mucho mas rapido (usa snapshots internos, especialmente en Hyperscale)
- Transaccionalmente consistente
- No requiere transferencia de datos
- Inmediatamente queryable

**Desventajas**:
- Genera costo (es una DB adicional en Azure)
- No es un archivo portable
- Solo 1 copia concurrente por source DB

### 5.7 Point-in-Time Restore (PITR) - Backups Automaticos

Azure SQL Database tiene backups automaticos integrados. **No necesitas hacer nada para habilitarlos**.

| Tipo de Backup | Frecuencia | Retencion Default |
|---------------|------------|-------------------|
| **Full** | Semanal | 7 dias (configurable 1-35) |
| **Differential** | Cada 12-24 horas | 7 dias (configurable 1-35) |
| **Transaction Log** | ~Cada 10 minutos | 7 dias (configurable 1-35) |

```powershell
# Restaurar a un punto en el tiempo (crea nueva DB)
az sql db restore `
    --resource-group "MyRG" `
    --server "myserver" `
    --name "MyDatabase" `
    --dest-name "MyDB_Restored_20260211" `
    --time "2026-02-11T08:00:00Z"

# Listar backups disponibles
az sql db ltr-backup list `
    --resource-group "MyRG" `
    --server "myserver" `
    --database "MyDatabase"
```

**Storage Redundancy Options**:
| Tipo | Descripcion | Geo-Restore |
|------|------------|-------------|
| LRS | 3 copias local | NO |
| ZRS | 3 copias cross-zone | NO |
| GRS (default) | 3 local + 3 region pareada | SI |
| GZRS (preview) | 3 cross-zone + 3 region pareada | SI |

**Long-Term Retention (LTR)**: Hasta 10 anos, backups full semanales/mensuales/anuales a Blob Storage.

**Limitacion principal de PITR**: Solo restaura como nueva DB en Azure. No puedes descargar el backup como archivo.

### 5.8 Comparativa de Metodos de Snapshot

| Metodo | Portable | Costo | Velocidad | Consistencia | Granularidad |
|--------|----------|-------|-----------|-------------|--------------|
| sqlpackage BACPAC | Si (archivo) | DTU uso | Lento (GB/hr) | Manual* | Full DB o tablas selectas |
| az sql db export | Si (Blob) | DTU + Storage | Lento | Manual* | Full DB |
| Database Copy | No (Azure) | DB adicional | Rapido | Transaccional | Full DB |
| PITR Restore | No (Azure) | DB adicional | Rapido | Transaccional | Full DB |
| bcp CSV | Si (archivos) | DTU minimo | Rapido/tabla | Per-statement | Tablas/queries |
| LTR Backup | No (Azure) | Storage | Automatico | Transaccional | Full DB |

*Para consistencia transaccional con BACPAC, exportar desde una Database Copy.

---

## 6. Integracion DuckDB

### 6.1 Conexion Directa DuckDB -> Azure SQL (mssql extension)

Desde 2025, existe una extension comunitaria `mssql` que conecta DuckDB directamente a SQL Server/Azure SQL via protocolo TDS nativo (sin ODBC).

```sql
-- Instalar y cargar extension
INSTALL mssql FROM community;
LOAD mssql;

-- Conectar a Azure SQL Database
ATTACH 'mssql://sqladmin:MyStr0ngP%40ss!@myserver.database.windows.net:1433?database=MyDatabase&use_encrypt=true' AS azuredb;

-- Consultar tablas
SELECT * FROM azuredb.dbo.Customers LIMIT 10;

-- Crear tabla local desde Azure SQL
CREATE TABLE local_customers AS SELECT * FROM azuredb.dbo.Customers;

-- Exportar a Parquet directamente desde Azure SQL
COPY (SELECT * FROM azuredb.dbo.Customers) TO 'customers.parquet' (FORMAT PARQUET);
```

**Autenticacion soportada**:
- SQL Authentication (user:password en URI)
- Azure Entra ID: Service Principal, CLI, Device Code, Environment Variables
- DuckDB secrets para gestion de credenciales

**Limitaciones**:
- Extension experimental (DuckDB >= 1.4.4)
- Soporte limitado de tipos de datos
- Named instances no soportadas
- Windows Authentication no soportada

### 6.2 DuckDB Azure Extension (Blob Storage)

Estrategia alternativa: exportar a Parquet en Azure Blob Storage, luego consultar con DuckDB.

```sql
-- Instalar extension Azure
INSTALL azure;
LOAD azure;

-- Configurar credenciales
CREATE SECRET azure_secret (
    TYPE AZURE,
    PROVIDER CREDENTIAL_CHAIN,
    ACCOUNT_NAME 'mybackupstorage'
);

-- O con connection string
CREATE SECRET azure_secret (
    TYPE AZURE,
    CONNECTION_STRING 'DefaultEndpointsProtocol=https;AccountName=mybackupstorage;AccountKey=xxx;EndpointSuffix=core.windows.net'
);

-- Consultar Parquet desde Blob Storage
SELECT * FROM 'az://mybackupstorage.blob.core.windows.net/exports/customers.parquet';

-- Consultar multiples archivos con glob
SELECT * FROM 'az://mybackupstorage.blob.core.windows.net/exports/*.parquet';

-- Crear tabla local desde Blob
CREATE TABLE customers AS
SELECT * FROM 'az://mybackupstorage.blob.core.windows.net/exports/customers.parquet';
```

**URI formats soportados**:
- `az://<storage_account>.blob.core.windows.net/<container>/<path>`
- `abfss://<container>@<storage_account>.dfs.core.windows.net/<path>` (ADLS Gen2)

### 6.3 Estrategia Recomendada: Export Parquet + DuckDB

```
Azure SQL Database
    |
    v  (bcp / sqlcmd / sqlpackage)
Archivos CSV/Parquet en local o Blob Storage
    |
    v  (DuckDB)
Queries analiticas locales sin conexion a Azure
```

**Script PowerShell para exportar tablas a Parquet via DuckDB**:

```powershell
# Requiere: DuckDB CLI instalado, mssql extension
# 1. Exportar multiples tablas a Parquet usando DuckDB
$tables = @("dbo.Customers", "dbo.Products", "dbo.Orders", "dbo.OrderItems")
$outDir = "C:\Exports\parquet\$(Get-Date -Format 'yyyy-MM-dd')"
New-Item -ItemType Directory -Path $outDir -Force | Out-Null

$connStr = "mssql://sqladmin:MyStr0ngP%40ss!@myserver.database.windows.net:1433?database=MyDatabase&use_encrypt=true"

foreach ($table in $tables) {
    $fileName = ($table -replace '\.', '_') + ".parquet"
    $filePath = Join-Path $outDir $fileName

    $sql = @"
INSTALL mssql FROM community;
LOAD mssql;
ATTACH '$connStr' AS src;
COPY (SELECT * FROM src.$table) TO '$filePath' (FORMAT PARQUET, COMPRESSION ZSTD);
"@

    Write-Host "Exporting $table -> $filePath"
    $sql | duckdb
}

# 2. Luego, queries locales sin Azure
# duckdb
# > SELECT * FROM 'C:\Exports\parquet\2026-02-11\dbo_Customers.parquet' LIMIT 10;
```

---

## 7. Automatizacion Practica (Windows)

### 7.1 Script PowerShell Completo para Export BACPAC Periodico

```powershell
<#
.SYNOPSIS
    Azure SQL Database BACPAC Export - Automated with retry logic
.DESCRIPTION
    Exports an Azure SQL Database to BACPAC file locally using sqlpackage.
    Designed for Windows Task Scheduler execution (hourly/daily).
.NOTES
    Requires: sqlpackage, CredentialManager module (or SecretManagement)
    Install: winget install Microsoft.SqlPackage
    Install: Install-Module -Name CredentialManager -Scope CurrentUser
#>

param(
    [string]$ConfigPath = "C:\Scripts\AzureSqlExport\config.json"
)

# ============================================================
# CONFIGURATION
# ============================================================

$ErrorActionPreference = "Stop"
$timestamp = Get-Date -Format "yyyy-MM-dd_HHmm"
$logDir = "C:\Scripts\AzureSqlExport\logs"
$backupDir = "C:\Backups\AzureSQL"
$logFile = Join-Path $logDir "export_$timestamp.log"
$maxRetries = 3
$retryDelaySeconds = 30
$retentionDays = 7  # Dias a mantener backups locales

# Ensure directories exist
New-Item -ItemType Directory -Path $logDir -Force | Out-Null
New-Item -ItemType Directory -Path $backupDir -Force | Out-Null

# ============================================================
# LOGGING
# ============================================================

function Write-Log {
    param([string]$Message, [string]$Level = "INFO")
    $entry = "$(Get-Date -Format 'yyyy-MM-dd HH:mm:ss') [$Level] $Message"
    Add-Content -Path $logFile -Value $entry
    if ($Level -eq "ERROR") {
        Write-Error $Message
    } else {
        Write-Host $entry
    }
}

# ============================================================
# CREDENTIAL MANAGEMENT
# ============================================================

function Get-AzureSqlCredentials {
    <#
    .SYNOPSIS
        Retrieves Azure SQL credentials from Windows Credential Manager.

    To store credentials initially, run once:
        Install-Module -Name CredentialManager -Scope CurrentUser
        New-StoredCredential -Target "AzureSql_MyServer" -Type Generic `
            -UserName "sqladmin" -Password "MyStr0ngP@ss!" -Persist LocalMachine
    #>
    try {
        # Option 1: Windows Credential Manager
        Import-Module CredentialManager -ErrorAction Stop
        $cred = Get-StoredCredential -Target "AzureSql_MyServer"
        if ($cred) {
            return @{
                User = $cred.UserName
                Password = $cred.GetNetworkCredential().Password
            }
        }
    } catch {
        Write-Log "CredentialManager not available, trying SecretManagement..." "WARN"
    }

    try {
        # Option 2: Microsoft SecretManagement
        Import-Module Microsoft.PowerShell.SecretManagement -ErrorAction Stop
        $user = Get-Secret -Name "AzureSql_User" -AsPlainText
        $pass = Get-Secret -Name "AzureSql_Password" -AsPlainText
        return @{ User = $user; Password = $pass }
    } catch {
        Write-Log "SecretManagement not available, trying Azure Key Vault..." "WARN"
    }

    try {
        # Option 3: Azure Key Vault (requires az login)
        $user = az keyvault secret show --vault-name "MyKeyVault" --name "sql-admin-user" --query value -o tsv
        $pass = az keyvault secret show --vault-name "MyKeyVault" --name "sql-admin-pass" --query value -o tsv
        return @{ User = $user; Password = $pass }
    } catch {
        throw "No credential source available. Configure CredentialManager, SecretManagement, or Azure Key Vault."
    }
}

# ============================================================
# EXPORT FUNCTION
# ============================================================

function Export-AzureSqlBacpac {
    param(
        [string]$Server,
        [string]$Database,
        [string]$User,
        [string]$Password,
        [string]$OutputPath,
        [string[]]$Tables = @()  # Empty = all tables
    )

    $args = @(
        "/Action:Export"
        "/TargetFile:`"$OutputPath`""
        "/SourceServerName:`"tcp:$Server,1433`""
        "/SourceDatabaseName:`"$Database`""
        "/SourceUser:`"$User`""
        "/SourcePassword:`"$Password`""
        "/p:Storage=File"
        "/p:CommandTimeout=1200"
        "/p:LongRunningCommandTimeout=0"
        "/p:DatabaseLockTimeout=-1"
        "/p:CompressionOption=Fast"
        "/p:MaxParallelism=8"
        "/p:VerifyExtraction=True"
        "/SourceEncryptConnection:True"
        "/SourceTrustServerCertificate:False"
        "/Diagnostics:True"
        "/DiagnosticsFile:`"$logFile.diag`""
    )

    # Add specific tables if provided
    foreach ($table in $Tables) {
        $args += "/p:TableData=`"$table`""
    }

    Write-Log "Starting sqlpackage export..."
    Write-Log "Target: $OutputPath"
    Write-Log "Database: $Server/$Database"
    if ($Tables.Count -gt 0) {
        Write-Log "Tables: $($Tables -join ', ')"
    } else {
        Write-Log "Tables: ALL"
    }

    $process = Start-Process -FilePath "sqlpackage" `
        -ArgumentList $args `
        -NoNewWindow -Wait -PassThru `
        -RedirectStandardOutput "$logFile.stdout" `
        -RedirectStandardError "$logFile.stderr"

    $exitCode = $process.ExitCode

    if ($exitCode -eq 0 -and (Test-Path $OutputPath)) {
        $size = [math]::Round((Get-Item $OutputPath).Length / 1MB, 2)
        Write-Log "Export completed successfully. File size: $size MB"
        return $true
    } else {
        $stderr = Get-Content "$logFile.stderr" -ErrorAction SilentlyContinue | Out-String
        Write-Log "Export failed. Exit code: $exitCode. Error: $stderr" "ERROR"
        return $false
    }
}

# ============================================================
# RETRY LOGIC
# ============================================================

function Invoke-WithRetry {
    param(
        [scriptblock]$Action,
        [int]$MaxRetries = 3,
        [int]$DelaySeconds = 30
    )

    for ($attempt = 1; $attempt -le $MaxRetries; $attempt++) {
        try {
            Write-Log "Attempt $attempt of $MaxRetries..."
            $result = & $Action
            if ($result) { return $true }
        } catch {
            Write-Log "Attempt $attempt failed: $($_.Exception.Message)" "WARN"
        }

        if ($attempt -lt $MaxRetries) {
            $delay = $DelaySeconds * $attempt  # Exponential-ish backoff
            Write-Log "Waiting $delay seconds before retry..."
            Start-Sleep -Seconds $delay
        }
    }
    return $false
}

# ============================================================
# NOTIFICATION
# ============================================================

function Send-Notification {
    param(
        [string]$Subject,
        [string]$Body,
        [string]$Level = "INFO"
    )

    # Option 1: Webhook (Teams/Slack/Discord)
    $webhookUrl = $env:EXPORT_WEBHOOK_URL
    if ($webhookUrl) {
        try {
            $payload = @{
                text = "[$Level] $Subject`n$Body"
            } | ConvertTo-Json

            Invoke-RestMethod -Uri $webhookUrl -Method Post -Body $payload -ContentType "application/json"
            Write-Log "Webhook notification sent"
        } catch {
            Write-Log "Failed to send webhook: $($_.Exception.Message)" "WARN"
        }
    }

    # Option 2: Email via SMTP
    # Uncomment and configure:
    # $smtpParams = @{
    #     From       = "alerts@mydomain.com"
    #     To         = "dba@mydomain.com"
    #     Subject    = "Azure SQL Export: $Subject"
    #     Body       = $Body
    #     SmtpServer = "smtp.office365.com"
    #     Port       = 587
    #     UseSsl     = $true
    #     Credential = Get-StoredCredential -Target "SMTP_Alerts"
    # }
    # Send-MailMessage @smtpParams

    # Option 3: Windows Event Log
    $source = "AzureSqlExport"
    if (-not [System.Diagnostics.EventLog]::SourceExists($source)) {
        New-EventLog -LogName Application -Source $source
    }
    $eventType = if ($Level -eq "ERROR") { "Error" } else { "Information" }
    Write-EventLog -LogName Application -Source $source -EventId 1000 -EntryType $eventType -Message "$Subject`n$Body"
}

# ============================================================
# RETENTION MANAGEMENT
# ============================================================

function Remove-OldBackups {
    param(
        [string]$BackupDir,
        [int]$RetentionDays
    )

    $cutoff = (Get-Date).AddDays(-$RetentionDays)
    $oldFiles = Get-ChildItem -Path $BackupDir -Filter "*.bacpac" -File | Where-Object { $_.LastWriteTime -lt $cutoff }

    foreach ($file in $oldFiles) {
        Write-Log "Removing old backup: $($file.Name) (created $($file.LastWriteTime))"
        Remove-Item $file.FullName -Force
    }

    if ($oldFiles.Count -gt 0) {
        Write-Log "Removed $($oldFiles.Count) old backup(s)"
    }
}

# ============================================================
# MAIN EXECUTION
# ============================================================

try {
    Write-Log "============================================"
    Write-Log "Azure SQL Database BACPAC Export - Starting"
    Write-Log "============================================"

    # Configuration
    $serverName = "myserver.database.windows.net"
    $databaseName = "MyDatabase"
    $bacpacPath = Join-Path $backupDir "$databaseName`_$timestamp.bacpac"

    # Get credentials
    Write-Log "Retrieving credentials..."
    $creds = Get-AzureSqlCredentials

    # Execute export with retry
    $success = Invoke-WithRetry -MaxRetries $maxRetries -DelaySeconds $retryDelaySeconds -Action {
        Export-AzureSqlBacpac `
            -Server $serverName `
            -Database $databaseName `
            -User $creds.User `
            -Password $creds.Password `
            -OutputPath $bacpacPath
            # Para exportar solo tablas especificas, agregar:
            # -Tables @("dbo.Customers", "dbo.Products", "dbo.Orders")
    }

    if ($success) {
        $fileSize = [math]::Round((Get-Item $bacpacPath).Length / 1MB, 2)
        Write-Log "SUCCESS: Export completed. Size: $fileSize MB"
        Send-Notification -Subject "Export OK: $databaseName" `
            -Body "BACPAC: $bacpacPath ($fileSize MB)" -Level "INFO"

        # Cleanup old backups
        Remove-OldBackups -BackupDir $backupDir -RetentionDays $retentionDays
    } else {
        Write-Log "FAILURE: Export failed after $maxRetries attempts" "ERROR"
        Send-Notification -Subject "EXPORT FAILED: $databaseName" `
            -Body "All $maxRetries attempts failed. Check logs: $logFile" -Level "ERROR"
        exit 1
    }

    Write-Log "============================================"
    Write-Log "Export process completed"
    Write-Log "============================================"

} catch {
    Write-Log "FATAL ERROR: $($_.Exception.Message)" "ERROR"
    Write-Log $_.ScriptStackTrace "ERROR"
    Send-Notification -Subject "FATAL: Azure SQL Export" `
        -Body "Error: $($_.Exception.Message)`nStack: $($_.ScriptStackTrace)" -Level "ERROR"
    exit 1
}
```

### 7.2 Almacenar Credenciales de Forma Segura

```powershell
# ============================================
# OPCION 1: Windows Credential Manager
# ============================================
Install-Module -Name CredentialManager -Scope CurrentUser -Force

# Guardar credenciales (ejecutar una vez manualmente)
New-StoredCredential -Target "AzureSql_MyServer" -Type Generic `
    -UserName "sqladmin" `
    -Password "MyStr0ngP@ss!" `
    -Persist LocalMachine

# Verificar
Get-StoredCredential -Target "AzureSql_MyServer"

# ============================================
# OPCION 2: Microsoft SecretManagement
# ============================================
Install-Module -Name Microsoft.PowerShell.SecretManagement -Scope CurrentUser -Force
Install-Module -Name Microsoft.PowerShell.SecretStore -Scope CurrentUser -Force

# Configurar vault (una vez)
Register-SecretVault -Name "LocalVault" -ModuleName Microsoft.PowerShell.SecretStore
Set-SecretStoreConfiguration -Authentication None  # Para Task Scheduler (sin prompt)

# Guardar secretos
Set-Secret -Name "AzureSql_User" -Secret "sqladmin"
Set-Secret -Name "AzureSql_Password" -Secret "MyStr0ngP@ss!"

# Recuperar
$user = Get-Secret -Name "AzureSql_User" -AsPlainText
$pass = Get-Secret -Name "AzureSql_Password" -AsPlainText

# ============================================
# OPCION 3: Azure Key Vault
# ============================================
# Requiere: az login (o Managed Identity)
az keyvault secret set --vault-name "MyKeyVault" --name "sql-admin-user" --value "sqladmin"
az keyvault secret set --vault-name "MyKeyVault" --name "sql-admin-pass" --value "MyStr0ngP@ss!"

# Recuperar
$user = az keyvault secret show --vault-name "MyKeyVault" --name "sql-admin-user" --query value -o tsv
$pass = az keyvault secret show --vault-name "MyKeyVault" --name "sql-admin-pass" --query value -o tsv

# ============================================
# OPCION 4: DPAPI encrypted file (simplest)
# ============================================
# Guardar (una vez, como el usuario que ejecutara el task)
$cred = Get-Credential -Message "Azure SQL credentials"
$cred | Export-Clixml -Path "C:\Scripts\AzureSqlExport\creds.xml"

# Recuperar (solo funciona con el mismo usuario en la misma maquina)
$cred = Import-Clixml -Path "C:\Scripts\AzureSqlExport\creds.xml"
$user = $cred.UserName
$pass = $cred.GetNetworkCredential().Password
```

### 7.3 Configurar Windows Task Scheduler

```powershell
# ============================================
# Crear Scheduled Task via PowerShell
# ============================================

$taskName = "AzureSqlBackup_Hourly"
$scriptPath = "C:\Scripts\AzureSqlExport\Export-AzureSqlBacpac.ps1"

# Accion: ejecutar PowerShell con el script
$action = New-ScheduledTaskAction `
    -Execute "powershell.exe" `
    -Argument "-NoProfile -ExecutionPolicy Bypass -File `"$scriptPath`""

# Trigger: cada hora
$trigger = New-ScheduledTaskTrigger `
    -Once `
    -At (Get-Date -Hour 0 -Minute 0 -Second 0) `
    -RepetitionInterval (New-TimeSpan -Hours 1) `
    -RepetitionDuration (New-TimeSpan -Days 365)

# Alternativa: trigger diario a las 2 AM
# $trigger = New-ScheduledTaskTrigger -Daily -At "02:00"

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

# Principal: ejecutar como usuario especifico con "logon or not"
$principal = New-ScheduledTaskPrincipal `
    -UserId "DOMAIN\ServiceAccount" `
    -LogonType Password `
    -RunLevel Highest

# Registrar task
Register-ScheduledTask `
    -TaskName $taskName `
    -Action $action `
    -Trigger $trigger `
    -Settings $settings `
    -Principal $principal `
    -Description "Hourly BACPAC export from Azure SQL Database"

# ============================================
# Verificar / gestionar
# ============================================

# Ver estado
Get-ScheduledTask -TaskName $taskName | Format-List

# Ejecutar manualmente
Start-ScheduledTask -TaskName $taskName

# Ver historial
Get-ScheduledTask -TaskName $taskName | Get-ScheduledTaskInfo

# Deshabilitar
Disable-ScheduledTask -TaskName $taskName

# Eliminar
Unregister-ScheduledTask -TaskName $taskName -Confirm:$false
```

### 7.4 Script Alternativo: Export con Access Token (sin password)

```powershell
<#
.SYNOPSIS
    Export usando Azure AD / Entra ID (sin SQL password)
    Requiere: az login previo o Managed Identity
#>

$timestamp = Get-Date -Format "yyyy-MM-dd_HHmm"
$backupDir = "C:\Backups\AzureSQL"
$logFile = "$backupDir\export_$timestamp.log"

# Obtener token fresco
try {
    $token = az account get-access-token --resource "https://database.windows.net/" --query accessToken -o tsv
    if (-not $token) { throw "Failed to get access token" }
} catch {
    # Si az login ha expirado, re-login con service principal
    az login --service-principal `
        -u $env:AZURE_CLIENT_ID `
        -p $env:AZURE_CLIENT_SECRET `
        --tenant $env:AZURE_TENANT_ID

    $token = az account get-access-token --resource "https://database.windows.net/" --query accessToken -o tsv
}

$bacpacPath = Join-Path $backupDir "MyDatabase_$timestamp.bacpac"

sqlpackage `
    /Action:Export `
    /TargetFile:$bacpacPath `
    /SourceServerName:"tcp:myserver.database.windows.net,1433" `
    /SourceDatabaseName:"MyDatabase" `
    /AccessToken:$token `
    /p:Storage=File `
    /p:CommandTimeout=1200 `
    /p:LongRunningCommandTimeout=0 `
    /Diagnostics:True `
    /DiagnosticsFile:$logFile

if ($LASTEXITCODE -eq 0) {
    Write-Host "Export OK: $bacpacPath"
} else {
    Write-Error "Export FAILED (exit code $LASTEXITCODE)"
    exit 1
}
```

---

## 8. Restaurar BACPAC Localmente

### 8.1 sqlpackage /Action:Import a SQL Server Express

```powershell
# Instalar SQL Server Express (si no lo tienes)
# winget install Microsoft.SQLServer.2022.Express

# Import BACPAC a SQL Server Express local
sqlpackage `
    /Action:Import `
    /SourceFile:"C:\Backups\AzureSQL\MyDatabase_2026-02-11_0800.bacpac" `
    /TargetServerName:"localhost\SQLEXPRESS" `
    /TargetDatabaseName:"MyDatabase_Local" `
    /p:CommandTimeout=0 `
    /p:LongRunningCommandTimeout=0 `
    /p:Storage=File

# Con autenticacion SQL
sqlpackage `
    /Action:Import `
    /SourceFile:"C:\Backups\AzureSQL\MyDatabase.bacpac" `
    /TargetServerName:"localhost\SQLEXPRESS" `
    /TargetDatabaseName:"MyDatabase_Local" `
    /TargetUser:"sa" `
    /TargetPassword:"LocalP@ss!" `
    /p:Storage=File
```

### 8.2 sqlpackage /Action:Import a LocalDB

```powershell
# LocalDB viene con Visual Studio o SQL Server Express
# Verificar instancias
sqllocaldb info

# Crear instancia si no existe
sqllocaldb create "MSSQLLocalDB"
sqllocaldb start "MSSQLLocalDB"

# Import a LocalDB
sqlpackage `
    /Action:Import `
    /SourceFile:"C:\Backups\AzureSQL\MyDatabase.bacpac" `
    /TargetServerName:"(localdb)\MSSQLLocalDB" `
    /TargetDatabaseName:"MyDatabase_Local" `
    /p:Storage=File

# Prerequisito: habilitar contained database authentication
sqlcmd -S "(localdb)\MSSQLLocalDB" -Q "EXEC sp_configure 'contained database authentication', 1; RECONFIGURE;"
```

### 8.3 Alternativa: Import BACPAC -> Queries con DuckDB (sin SQL Server)

Si no quieres instalar SQL Server, puedes extraer los datos del BACPAC y consultarlos con DuckDB:

```powershell
# Un BACPAC es un ZIP. Contiene:
# - model.xml (schema)
# - Data/<schema>.<table>/TableData-*.BCP (datos en formato BCP)

# Estrategia:
# 1. Extraer el BACPAC (ZIP)
# 2. Convertir BCP files a CSV/Parquet
# 3. Consultar con DuckDB

# Paso 1: Extraer
$bacpacPath = "C:\Backups\AzureSQL\MyDatabase.bacpac"
$extractDir = "C:\Backups\AzureSQL\MyDatabase_extracted"
Expand-Archive -Path $bacpacPath -DestinationPath $extractDir -Force

# Paso 2 & 3: Mejor usar sqlpackage import a SQL Express y luego exportar tablas con bcp
# O usar la estrategia directa: exportar tablas como CSV/Parquet desde Azure SQL originalmente
```

### 8.4 Estrategia Recomendada: Export Tablas -> Parquet -> DuckDB

En vez de BACPAC completo, exportar tablas especificas directamente a Parquet:

```powershell
# ============================================
# ESTRATEGIA: bcp -> CSV -> DuckDB -> Parquet
# ============================================

$server = "myserver.database.windows.net"
$db = "MyDatabase"
$user = "sqladmin"
$pass = "MyStr0ngP@ss!"
$exportDir = "C:\Exports\$(Get-Date -Format 'yyyy-MM-dd')"
New-Item -ItemType Directory -Path $exportDir -Force | Out-Null

# 1. Export tables as CSV via bcp
$tables = @(
    @{Schema="dbo"; Table="Customers"; Query="SELECT * FROM dbo.Customers"},
    @{Schema="dbo"; Table="Products"; Query="SELECT * FROM dbo.Products"},
    @{Schema="dbo"; Table="Orders"; Query="SELECT * FROM dbo.Orders WHERE OrderDate >= DATEADD(month, -3, GETDATE())"},
    @{Schema="dbo"; Table="Categories"; Query="SELECT * FROM dbo.Categories"}
)

foreach ($t in $tables) {
    $csvPath = Join-Path $exportDir "$($t.Schema)_$($t.Table).csv"
    Write-Host "Exporting $($t.Schema).$($t.Table)..."

    bcp "$($t.Query)" queryout $csvPath `
        -S $server -d $db -U $user -P $pass `
        -c -t "," -r "\n" -q
}

# 2. Convert CSV to Parquet and create DuckDB database
$duckdbPath = Join-Path $exportDir "snapshot.duckdb"

$sql = @"
-- Create tables from CSV files
CREATE TABLE customers AS SELECT * FROM read_csv_auto('$exportDir/dbo_Customers.csv');
CREATE TABLE products AS SELECT * FROM read_csv_auto('$exportDir/dbo_Products.csv');
CREATE TABLE orders AS SELECT * FROM read_csv_auto('$exportDir/dbo_Orders.csv');
CREATE TABLE categories AS SELECT * FROM read_csv_auto('$exportDir/dbo_Categories.csv');

-- Also export as individual Parquet files
COPY customers TO '$exportDir/customers.parquet' (FORMAT PARQUET, COMPRESSION ZSTD);
COPY products TO '$exportDir/products.parquet' (FORMAT PARQUET, COMPRESSION ZSTD);
COPY orders TO '$exportDir/orders.parquet' (FORMAT PARQUET, COMPRESSION ZSTD);
COPY categories TO '$exportDir/categories.parquet' (FORMAT PARQUET, COMPRESSION ZSTD);

-- Verify
SELECT 'customers' as tbl, count(*) as rows FROM customers
UNION ALL SELECT 'products', count(*) FROM products
UNION ALL SELECT 'orders', count(*) FROM orders
UNION ALL SELECT 'categories', count(*) FROM categories;
"@

$sql | duckdb $duckdbPath

Write-Host "DuckDB snapshot ready: $duckdbPath"
Write-Host "Parquet files in: $exportDir"
```

### 8.5 Queries Locales con DuckDB

```sql
-- Abrir el snapshot
-- duckdb C:\Exports\2026-02-11\snapshot.duckdb

-- Consultar directamente
SELECT c.Name, COUNT(o.OrderID) as TotalOrders, SUM(o.Amount) as Revenue
FROM customers c
JOIN orders o ON c.CustomerID = o.CustomerID
GROUP BY c.Name
ORDER BY Revenue DESC
LIMIT 20;

-- O consultar Parquet files directamente (sin importar)
SELECT * FROM 'C:\Exports\2026-02-11\customers.parquet'
WHERE Region = 'LATAM';

-- Cross-file join
SELECT *
FROM 'C:\Exports\2026-02-11\orders.parquet' o
JOIN 'C:\Exports\2026-02-11\customers.parquet' c
    ON o.CustomerID = c.CustomerID
WHERE o.OrderDate >= '2026-01-01';

-- Exportar resultados a Excel-friendly CSV
COPY (
    SELECT * FROM orders WHERE Amount > 1000
) TO 'C:\Reports\high_value_orders.csv' (HEADER, DELIMITER ',');
```

---

## 9. Optimizacion de Costos

### 9.1 Impacto en DTU

```
Export Method              DTU Impact    Estimated Time (10GB)
-----------------------------------------------------------------
sqlpackage (all tables)    HIGH (60-90%) 15-45 min
sqlpackage (select tables) MEDIUM        5-15 min
az sql db export           MEDIUM        20-60 min
bcp (per table)            LOW-MEDIUM    1-5 min/table
DATABASE COPY              LOW           2-10 min
PITR Restore               NONE*         Instant
```
*PITR usa backups internos, no consume DTU de produccion.

### 9.2 Usar Read Replicas

Si tienes Premium/Business Critical o geo-replication configurada:

```powershell
# Exportar desde una read replica en vez de produccion
# Opcion 1: Usar ApplicationIntent=ReadOnly en connection string
sqlpackage `
    /Action:Export `
    /TargetFile:"C:\Backups\mydb.bacpac" `
    /SourceConnectionString:"Server=tcp:myserver.database.windows.net;Database=MyDB;User ID=sqladmin;Password=pass;ApplicationIntent=ReadOnly;Encrypt=True;" `
    /p:Storage=File

# Opcion 2: Exportar desde geo-secondary
sqlpackage `
    /Action:Export `
    /TargetFile:"C:\Backups\mydb.bacpac" `
    /SourceServerName:"tcp:myserver-secondary.database.windows.net" `
    /SourceDatabaseName:"MyDatabase" `
    /SourceUser:"sqladmin" `
    /SourcePassword:"pass" `
    /p:Storage=File
```

### 9.3 Database Copy como Alternativa Economica

```powershell
# Estrategia: Copy -> Export from Copy -> Delete Copy
# Beneficio: produccion no se impacta durante el export

# 1. Crear copia (rapido, bajo impacto)
$copyName = "MyDB_ExportCopy"
az sql db copy `
    --name "MyDatabase" `
    --resource-group "MyRG" `
    --server "myserver" `
    --dest-name $copyName

# 2. Esperar a que termine
do {
    $state = az sql db show -g "MyRG" -s "myserver" -n $copyName --query "status" -o tsv
    Write-Host "Copy state: $state"
    Start-Sleep -Seconds 10
} while ($state -ne "Online")

# 3. Exportar desde la copia (no afecta produccion)
sqlpackage /a:Export /tf:"C:\Backups\mydb.bacpac" `
    /ssn:"tcp:myserver.database.windows.net" /sdn:$copyName `
    /su:"sqladmin" /sp:"pass" /p:Storage=File

# 4. Eliminar la copia inmediatamente
az sql db delete -g "MyRG" -s "myserver" -n $copyName --yes

# Costo: solo pagas por la DB copia durante el tiempo que exista
# Si el export tarda 30 min y la copia es S0 ($15/mes): ~$0.01 por export
```

### 9.4 Programar Exports en Horas No Pico

```powershell
# Configurar Task Scheduler para 2 AM (horario de menos actividad)
$trigger = New-ScheduledTaskTrigger -Daily -At "02:00"

# O para dias laborales a medianoche
$trigger = New-ScheduledTaskTrigger -Weekly `
    -DaysOfWeek Monday,Tuesday,Wednesday,Thursday,Friday `
    -At "00:00"
```

### 9.5 Escalar Temporalmente para Export

```powershell
# Escalar UP antes del export
az sql db update -g "MyRG" -s "myserver" -n "MyDatabase" --service-objective S3

# ... ejecutar export ...

# Escalar DOWN despues
az sql db update -g "MyRG" -s "myserver" -n "MyDatabase" --service-objective S0

# Script completo con escala temporal
$originalTier = az sql db show -g "MyRG" -s "myserver" -n "MyDatabase" --query "currentServiceObjectiveName" -o tsv

# Scale up
Write-Host "Scaling up to S3..."
az sql db update -g "MyRG" -s "myserver" -n "MyDatabase" --service-objective S3
Start-Sleep -Seconds 60  # Esperar a que aplique

# Export
sqlpackage /a:Export /tf:"C:\Backups\mydb.bacpac" `
    /ssn:"tcp:myserver.database.windows.net" /sdn:"MyDatabase" `
    /su:"sqladmin" /sp:"pass" /p:Storage=File /p:CommandTimeout=0

# Scale back
Write-Host "Scaling back to $originalTier..."
az sql db update -g "MyRG" -s "myserver" -n "MyDatabase" --service-objective $originalTier
```

### 9.6 Resumen de Costos por Estrategia

| Estrategia | Costo Azure Adicional | Costo DTU | Mejor Para |
|------------|----------------------|-----------|------------|
| sqlpackage directo | $0 | ALTO | DBs pequenas, off-peak |
| sqlpackage + DB Copy | ~$0.01/export | BAJO | Produccion, consistencia |
| az sql db export | Storage cost | MEDIO | Automatizacion simple |
| bcp tablas selectas | $0 | BAJO | Tablas de catalogo |
| PITR (built-in) | Incluido | ZERO | DR, no para download |
| LTR backups | Storage cost | ZERO | Compliance, largo plazo |
| Scale up + export | Tier difference | MEDIO | DBs grandes |
| Read replica export | Replica cost | ZERO (prod) | Premium/BC tier |

---

## Fuentes y Referencias

### Microsoft Documentation
- [SqlPackage Export](https://learn.microsoft.com/en-us/sql/tools/sqlpackage/sqlpackage-export?view=sql-server-ver17)
- [Export Azure SQL Database to BACPAC](https://learn.microsoft.com/en-us/azure/azure-sql/database/database-export?view=azuresql)
- [Azure SQL Database Automated Backups](https://learn.microsoft.com/en-us/azure/azure-sql/database/automated-backups-overview?view=azuresql)
- [Copy a Database - Azure SQL Database](https://learn.microsoft.com/en-us/azure/azure-sql/database/database-copy?view=azuresql)
- [Import/Export Performance Troubleshooting](https://learn.microsoft.com/en-us/azure/azure-sql/database/database-import-export-hang?view=azuresql)
- [az sql db CLI Reference](https://learn.microsoft.com/en-us/cli/azure/sql/db?view=azure-cli-latest)
- [IP Firewall Rules - Azure SQL Database](https://learn.microsoft.com/en-us/azure/azure-sql/database/firewall-configure?view=azuresql)
- [Read Queries on Replicas](https://learn.microsoft.com/en-us/azure/azure-sql/database/read-scale-out?view=azuresql)
- [Active Geo-Replication](https://learn.microsoft.com/en-us/azure/azure-sql/database/active-geo-replication-overview?view=azuresql)
- [SqlPackage with Managed Identity](https://techcommunity.microsoft.com/t5/azure-database-support-blog/how-to-use-sqlpackage-with-managed-identity/ba-p/3642942)
- [SqlPackage with Access Token](https://techcommunity.microsoft.com/t5/azure-database-support-blog/step-by-step-how-to-use-sqlpackage-with-access-token/ba-p/1407819)
- [bcp Utility](https://learn.microsoft.com/en-us/sql/tools/bcp-utility?view=sql-server-ver16)
- [go-sqlcmd](https://github.com/microsoft/go-sqlcmd)
- [Download sqlcmd](https://learn.microsoft.com/en-us/sql/tools/sqlcmd/sqlcmd-download-install?view=sql-server-ver17)
- [Restore from Backup - Azure SQL](https://learn.microsoft.com/en-us/azure/azure-sql/database/recovery-using-backups?view=azuresql)

### DuckDB
- [DuckDB Azure Extension](https://duckdb.org/docs/stable/core_extensions/azure)
- [DuckDB mssql Extension](https://duckdb.org/community_extensions/extensions/mssql)
- [DuckDB Parquet Import](https://duckdb.org/docs/stable/guides/file_formats/parquet_import)
- [DuckDB SQLite Extension](https://duckdb.org/docs/stable/core_extensions/sqlite)
- [Analyze Data in Azure with DuckDB](https://motherduck.com/blog/analyze-data-in-azure-with-duckdb/)

### Community / Tutorials
- [SQLPackage Export Azure SQL Databases](https://www.sqlshack.com/sqlpackage-utility-to-export-azure-sql-databases/)
- [bcp Import/Export Azure SQL](https://www.sqlshack.com/bcp-for-import-and-export-data-in-azure-sql-database/)
- [4 Ways to Export Data from Azure SQL](https://blog.devart.com/export-azure-sql-database.html)
- [Automate Azure SQL BACPAC with PowerShell](https://www.mssqltips.com/sqlservertip/3606/automate-retrieving-sql-azure-bacpacs-with-powershell/)
- [Azure SQL BACPAC Large Database Guide](https://azureops.org/articles/bacpac-large-sql-server-database/)
- [Azure SQL Optimizing BACPAC Imports](https://azurefeeds.com/2025/11/21/azure-sql-optimizing-bacpac-imports-sqlpackage-done-right/)
