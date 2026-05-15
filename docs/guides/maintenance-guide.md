# Cómo Ejecutar el Script de Mantenimiento (Viendo el Progreso)

## 🎯 Para Ver el Progreso en Consola

El script **DEBE** ejecutarse con privilegios de administrador para funcionar correctamente.

---

## ✅ MÉTODO RECOMENDADO (Ver progreso en tiempo real)

### **Paso 1: Abrir PowerShell como Administrador**

Tienes 3 opciones:

#### **Opción A: Atajo de teclado**
1. Presiona `Win + X`
2. Selecciona **"Windows PowerShell (Administrador)"** o **"Terminal (Admin)"**

#### **Opción B: Menú Inicio**
1. Busca "PowerShell" en el menú inicio
2. Clic derecho → **"Ejecutar como administrador"**

#### **Opción C: Desde PowerShell normal**
```powershell
Start-Process PowerShell -Verb RunAs
```

### **Paso 2: Navegar al directorio**

```powershell
cd %HOME%
```

### **Paso 3: Ejecutar el script**

**Si cargaste el perfil:**
```powershell
. $PROFILE
update-all
```

**O ejecuta directamente:**
```powershell
.\run-update.ps1
```

**O ejecuta el script principal:**
```powershell
.\SystemMaintenance.ps1
```

---

## 📺 ¿Qué Verás en Pantalla?

### **1. Inicio del Script**
```
================================================================================
           SCRIPT DE MANTENIMIENTO Y OPTIMIZACIÓN DEL SISTEMA
================================================================================
```

### **2. Progreso de Tareas**

Verás mensajes como:

```
[2025-10-12 14:32:15] [INFO] === Iniciando limpieza de archivos temporales ===
[2025-10-12 14:32:18] [SUCCESS] Se eliminaron 1,247 archivos (2.3 MB)
[2025-10-12 14:32:20] [INFO] === Iniciando limpieza de componentes de Windows ===
[2025-10-12 14:35:45] [SUCCESS] DISM completó correctamente
...
```

### **3. Resumen Final**

Al terminar verás el resumen ejecutivo detallado:

```
╔════════════════════════════════════════════════════════════════════════════╗
║                        RESUMEN EJECUTIVO DE CAMBIOS                       ║
╚════════════════════════════════════════════════════════════════════════════╝

  ⏱️  Duración Total: 00:08:32
  📅 Fecha: 2025-10-12 14:25:00

  📋 ESTADO DE TAREAS
  ────────────────────────────────────────────────────────────────────────────

  ✓ Limpieza Archivos Temporales                         [Éxito]
  ✓ Limpieza Componentes Windows                         [Éxito]
  ...

  📊 RESUMEN DE CAMBIOS
  ────────────────────────────────────────────────────────────────────────────

  🗑️  Archivos Temporales Eliminados:
      • Archivos: 1,247 archivos
      • Espacio liberado: 2.34 GB
  ...
```

---

## 🚫 SI NO ERES ADMINISTRADOR

Si ejecutas el script **SIN** privilegios de administrador, verás:

```
================================================================================
  ERROR: Este script requiere privilegios de administrador
================================================================================

Por favor, abre PowerShell como Administrador y ejecuta:
  cd %HOME%
  .\run-update.ps1
```

**Solución:** Abre PowerShell como Administrador (ver Paso 1 arriba)

---

## ⚡ FORMA RÁPIDA (Una sola línea)

Si ya estás en PowerShell como Administrador:

```powershell
%HOME%\SystemMaintenance.ps1
```

---

## 🔍 VERIFICAR QUE TIENES PRIVILEGIOS DE ADMIN

Antes de ejecutar, puedes verificar con:

```powershell
([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
```

Si retorna `True` → Tienes permisos de admin ✅
Si retorna `False` → NO tienes permisos de admin ❌

---

## 📝 NOTAS IMPORTANTES

### ✅ **Verás el progreso en tiempo real**
- Cada tarea muestra su estado mientras se ejecuta
- Los logs se escriben en pantalla con colores
- Al final verás el resumen completo con métricas

### ⏱️ **Duración Estimada**
- Primera ejecución: 8-15 minutos
- Ejecuciones posteriores: 3-8 minutos (si no hay actualizaciones)

### 💾 **Durante la Ejecución**
- NO cierres la ventana de PowerShell
- Puedes ver el progreso en tiempo real
- Algunas tareas pueden parecer "congeladas" (DISM tarda varios minutos)

### 🗂️ **Archivos Generados**
Al terminar, encontrarás en `C:\Scripts\`:
- `SystemMaintenance_YYYY-MM-DD.log` - Log detallado
- `SystemMaintenance_Status_YYYY-MM-DD.json` - Reporte JSON
- `BackupLog_YYYY-MM-DD.txt` - Log del respaldo

---

## 🐛 TROUBLESHOOTING

### **Problema: "No se puede cargar el archivo porque la ejecución de scripts está deshabilitada"**

**Solución:**
```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned -Force
```

### **Problema: "update-all no se reconoce"**

**Solución 1: Cargar el perfil**
```powershell
. $PROFILE
```

**Solución 2: Ejecutar directamente**
```powershell
.\run-update.ps1
```

### **Problema: DISM se queda "congelado"**

**No es un problema:** DISM puede tardar 5-10 minutos. Espera pacientemente.

### **Problema: Error en el respaldo**

**Causa común:** La unidad D:\ no está disponible o las rutas no existen.

**Solución:**
```powershell
# Verificar que las rutas existen
Test-Path "D:\Alex\Documents"
Test-Path "D:\Backups"

# Si no existen, créalas:
New-Item -Path "D:\Alex\Documents" -ItemType Directory -Force
New-Item -Path "D:\Backups" -ItemType Directory -Force
```

---

## 🎬 EJEMPLO COMPLETO

```powershell
# 1. Abrir PowerShell como Admin (Win + X → PowerShell Admin)

# 2. Navegar al directorio
PS C:\Windows\System32> cd %HOME%
PS %HOME%>

# 3. Ejecutar el script
PS %HOME%> .\run-update.ps1

Ejecutando mantenimiento del sistema...

================================================================================
           SCRIPT DE MANTENIMIENTO Y OPTIMIZACIÓN DEL SISTEMA
================================================================================

[2025-10-12 14:32:15] [INFO] Script iniciado por usuario: msi
[2025-10-12 14:32:15] [INFO] === Iniciando limpieza de archivos temporales ===
...

# 4. Esperar a que termine (verás el progreso)

# 5. Al final verás el resumen ejecutivo completo
```

---

## ✅ CHECKLIST ANTES DE EJECUTAR

- [ ] PowerShell está abierto **como Administrador**
- [ ] Estás en el directorio `%HOME%`
- [ ] Tienes al menos 5 GB de espacio libre en disco C:\
- [ ] La unidad D:\ está disponible (para respaldos)
- [ ] No hay actualizaciones de Windows pendientes de reinicio

---

**¡Listo para ejecutar!** 🚀

Simplemente abre PowerShell como Admin y ejecuta:
```powershell
cd %HOME%
.\run-update.ps1
```
