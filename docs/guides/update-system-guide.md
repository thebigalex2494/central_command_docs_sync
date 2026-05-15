# Sistema de Mantenimiento Automatizado - `update-all`

## Descripción

Sistema completo de mantenimiento y optimización de Windows que integra:
- Actualización de herramientas de desarrollo (Node.js, npm)
- Limpieza de archivos temporales
- Optimización de componentes del sistema
- Deshabilitación de programas de inicio innecesarios
- Respaldo incremental automático
- Actualización de métricas de rendimiento

## Archivos del Sistema

| Archivo | Ubicación | Descripción |
|---------|-----------|-------------|
| **SystemMaintenance.ps1** | `%HOME%\` | Script principal de mantenimiento |
| **Microsoft.PowerShell_profile.ps1** | `%HOME%\Documents\WindowsPowerShell\` | Perfil de PowerShell con comando global |
| **setup-update-all.ps1** | `%HOME%\` | Script de configuración inicial (ejecutar una vez) |
| **README-UpdateAll.md** | `%HOME%\` | Este archivo de documentación |

## Instalación y Configuración

### Primera vez - Configuración Inicial

1. **Abrir PowerShell como Administrador**
   - Presiona `Win + X` → Selecciona "Windows PowerShell (Admin)"

2. **Ejecutar el script de configuración**
   ```powershell
   cd %HOME%
   .\setup-update-all.ps1
   ```

3. **Cerrar y reabrir PowerShell**
   - Esto asegura que el perfil se cargue correctamente

### Verificación

Para verificar que el comando está disponible:

```powershell
Get-Command update-all
```

Deberías ver:

```
Name       CommandType Source
----       ----------- ------
update-all       Alias
```

## Uso del Comando `update-all`

### Ejecución Básica

Simplemente abre **cualquier terminal de PowerShell** y ejecuta:

```powershell
update-all
```

### Comportamiento Automático

- **Si NO tienes privilegios de administrador**: El comando automáticamente abrirá una nueva ventana elevada de PowerShell y ejecutará el script
- **Si YA eres administrador**: El script se ejecuta directamente en la ventana actual

### Desde Terminal Normal (sin elevación)

```powershell
# Desde cualquier directorio
C:\> update-all

# El sistema pedirá elevación automáticamente (UAC)
```

### Desde PowerShell como Administrador

```powershell
# Se ejecuta directamente sin pedir permisos adicionales
PS C:\> update-all
```

## Tareas Ejecutadas

El comando `update-all` ejecuta las siguientes tareas en orden:

### 1. Limpieza de Archivos Temporales
- Limpia: `%SystemRoot%\Temp`, `%TEMP%`, `%LOCALAPPDATA%\Temp`
- Registra archivos eliminados y espacio liberado
- Continúa aunque algunos archivos estén en uso

### 2. Limpieza de Componentes de Windows
- Ejecuta: `DISM.exe /Online /Cleanup-Image /StartComponentCleanup /ResetBase`
- Elimina versiones antiguas de componentes de Windows
- Libera espacio significativo en disco

### 3. Liberador de Espacio en Disco
- Ejecuta: `cleanmgr.exe /sagerun:1`
- Limpia archivos temporales del sistema, caché, etc.

### 4. Deshabilitación de Programas de Inicio
- Remueve de la lista de inicio (registro):
  - OneDrive
  - Spotify
  - Teams
  - Discord
  - SomeUnwantedApp (personalizable)

### 5. Actualización de Métricas de Rendimiento
- Ejecuta: `winsat formal -restart clean`
- Recalcula el índice de rendimiento de Windows

### 6. Actualización de Node.js
- Verifica versión instalada vs. última disponible
- Descarga e instala automáticamente si hay actualización
- Instalación silenciosa (sin interacción)

### 7. Actualización de Paquetes npm
- Actualiza npm a la última versión
- Detecta paquetes globales obsoletos
- Actualiza solo los que necesitan actualización

### 8. Respaldo Incremental
- **Origen**: `D:\Alex\Documents`
- **Destino**: `D:\Backups\AlexDocuments`
- **Método**: Robocopy con modo espejo (/MIR)
- **Características**:
  - Copia solo archivos nuevos o modificados
  - Elimina archivos en destino que ya no existen en origen
  - Modo reiniciable en caso de interrupciones
  - Excluye archivos de sistema y ocultos

## Archivos de Log y Reportes

Después de cada ejecución, se generan los siguientes archivos en `C:\Scripts\`:

### Log Completo
```
C:\Scripts\SystemMaintenance_YYYY-MM-DD.log
```

Contiene registro detallado de todas las operaciones con timestamps.

**Ejemplo:**
```
[2025-10-12 14:32:15] [INFO] Script iniciado por usuario: msi
[2025-10-12 14:32:20] [SUCCESS] Se eliminaron 1,247 archivos (2.3 MB)
[2025-10-12 14:41:03] [SUCCESS] DISM completó correctamente
```

### Reporte JSON
```
C:\Scripts\SystemMaintenance_Status_YYYY-MM-DD.json
```

Reporte estructurado en formato JSON con el estado de cada tarea.

**Ejemplo:**
```json
{
  "limpieza_archivos_temporales": {
    "estado": "Éxito",
    "detalles": "Se eliminaron 1,247 archivos (2.3 MB). 3 archivos omitidos.",
    "timestamp": "2025-10-12T14:32:15-06:00",
    "codigo_salida": 0
  },
  "actualizacion_nodejs": {
    "estado": "Éxito",
    "detalles": "Node.js actualizado de 18.17.0 a 20.10.0.",
    "timestamp": "2025-10-12T14:45:30-06:00",
    "codigo_salida": 0
  }
}
```

### Log de Respaldo
```
C:\Scripts\BackupLog_YYYY-MM-DD.txt
```

Log detallado de Robocopy con estadísticas de archivos copiados.

## Personalización

### Cambiar Rutas de Respaldo

Edita el archivo `%HOME%\SystemMaintenance.ps1` (líneas 36-37):

```powershell
# Rutas de respaldo
$BackupSource = "D:\Alex\Documents"           # Cambiar aquí
$BackupDestination = "D:\Backups\AlexDocuments"  # Cambiar aquí
```

### Modificar Programas de Inicio a Deshabilitar

Edita el archivo `%HOME%\SystemMaintenance.ps1` (línea 33):

```powershell
$StartupBlacklist = @('OneDrive', 'Spotify', 'SomeUnwantedApp', 'Teams', 'Discord')
# Agregar o quitar aplicaciones según necesites
```

### Cambiar Ubicación de Logs

Edita el archivo `%HOME%\SystemMaintenance.ps1` (línea 26):

```powershell
$ScriptDir = "C:\Scripts"  # Cambiar a tu directorio preferido
```

## Programación Automática (Opcional)

### Crear Tarea Programada Semanal

Ejecuta como Administrador:

```powershell
$action = New-ScheduledTaskAction -Execute 'PowerShell.exe' `
    -Argument '-ExecutionPolicy Bypass -File "%HOME%\SystemMaintenance.ps1"'

$trigger = New-ScheduledTaskTrigger -Weekly -DaysOfWeek Sunday -At 2:00AM

$principal = New-ScheduledTaskPrincipal -UserId "SYSTEM" `
    -LogonType ServiceAccount -RunLevel Highest

Register-ScheduledTask -Action $action -Trigger $trigger `
    -Principal $principal -TaskName "Mantenimiento del Sistema" `
    -Description "Ejecución automática de limpieza, optimización y respaldo"
```

### Ver Tareas Programadas

```powershell
Get-ScheduledTask -TaskName "Mantenimiento del Sistema"
```

### Eliminar Tarea Programada

```powershell
Unregister-ScheduledTask -TaskName "Mantenimiento del Sistema" -Confirm:$false
```

## Resolución de Problemas

### El comando `update-all` no se reconoce

**Causa**: El perfil de PowerShell no se ha cargado.

**Solución**:
1. Cierra todas las ventanas de PowerShell
2. Abre una nueva ventana de PowerShell
3. Verifica que veas el mensaje: "PowerShell Profile cargado - Comando 'update-all' disponible"

### Error de política de ejecución

**Síntoma**:
```
El archivo no se puede cargar porque la ejecución de scripts está deshabilitada
```

**Solución**:
```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned -Force
```

### El respaldo falla con "unidad no disponible"

**Causa**: La unidad D:\ no está conectada o la ruta no existe.

**Solución**:
1. Verifica que la unidad D:\ esté disponible
2. Crea manualmente las carpetas si no existen:
   ```powershell
   New-Item -Path "D:\Alex\Documents" -ItemType Directory -Force
   New-Item -Path "D:\Backups" -ItemType Directory -Force
   ```

### DISM falla con error

**Síntoma**: DISM retorna código de error != 0

**Solución**:
1. Verifica que tienes privilegios de administrador
2. Ejecuta manualmente:
   ```cmd
   DISM.exe /Online /Cleanup-Image /RestoreHealth
   ```
3. Reinicia el sistema y vuelve a intentar

## Notas de Seguridad

- ✅ **El script requiere privilegios administrativos** - Esto es necesario para operaciones de sistema
- ✅ **Los archivos están firmados y controlados** - Solo ejecuta scripts de fuentes confiables
- ✅ **Respaldo con modo espejo (/MIR)** - Archivos eliminados en origen se eliminan en destino
- ⚠️ **Revisa el log antes de respaldos críticos** - Verifica que las rutas sean correctas

## Estructura de Código

```
%HOME%\
├── SystemMaintenance.ps1          # Script principal
├── setup-update-all.ps1           # Configuración inicial
├── update_tools.bat               # Legacy (obsoleto)
├── update_tools.ps1               # Legacy (integrado)
├── README-UpdateAll.md            # Esta documentación
└── Documents\
    └── WindowsPowerShell\
        └── Microsoft.PowerShell_profile.ps1  # Perfil con función global

C:\Scripts\                        # Generado automáticamente
├── SystemMaintenance_YYYY-MM-DD.log
├── SystemMaintenance_Status_YYYY-MM-DD.json
├── BackupLog_YYYY-MM-DD.txt
├── dism_output.txt
├── dism_error.txt
├── winsat_output.txt
└── winsat_error.txt
```

## Contribución y Soporte

Este script fue generado por **AI System Automation Engineer** basado en análisis del sistema.

Para reportar problemas o sugerencias:
- Revisa los logs en `C:\Scripts\`
- Verifica los códigos de salida en el reporte JSON
- Consulta la sección de Resolución de Problemas

## Changelog

### Versión 1.0 - 2025-10-12
- ✅ Implementación inicial completa
- ✅ Integración de funciones de update_tools.ps1
- ✅ Sistema de logging robusto con JSON
- ✅ Comando global `update-all`
- ✅ Elevación automática de privilegios
- ✅ Respaldo incremental con Robocopy
- ✅ Rutas personalizadas para D:\Alex\Documents

---

**¡Listo para usar!** Ejecuta `update-all` desde cualquier terminal de PowerShell.
