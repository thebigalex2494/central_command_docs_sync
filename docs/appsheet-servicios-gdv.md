# Trabajo realizado en AppSheet — "Control de servicios GDV"

**Fecha:** 10 de febrero de 2026
**App:** Control de servicios GDV
**URL Editor:** `https://www.appsheet.com/template/AppDef?appName=servicios010125-24894927`

---

## Exploración y análisis de la app

Se realizó una exploración completa del editor de AppSheet identificando:

- **11 tablas** en la aplicación: cambios, cat_clientes, cat_incidencias, cat_personal, incidencias, inicio, OPERADORES, pendientes_revision, PV, registro, RUTAS
- **Tabla principal analizada: "cambios"** con 39 columnas
- **Campo clave identificado: "Estatus"** — tipo Enum con 6 valores posibles:
  - En Ruta
  - Cancelados
  - Liberado
  - Programado en vivo
  - Terminados
  - Programado Por Contrato
- **4 vistas existentes** en la app: Incidencias Abiertas, Servicios del día, Historial de cambios, Reporte de cambios
- **2 reglas de formato preexistentes** en la tabla "incidencias": Prioridad Alta y Prioridad Critica

---

## Reglas de formato creadas

Se configuraron **6 reglas de formato condicional** en la sección **App > Format Rules** para la tabla "cambios", aplicadas a la columna "Estatus". El objetivo es que los usuarios identifiquen de un vistazo el estado de cada servicio sin necesidad de leer el texto.

| Regla | Condición | Color de fondo | Color de texto | Significado |
|---|---|---|---|---|
| Estatus Cancelados | `[Estatus]="Cancelados"` | `#FF0000` | `#FF0000` | Rojo — servicio cancelado |
| Estatus En Ruta | `[Estatus]="En Ruta"` | `#0000FF` | `#0000FF` | Azul — unidad en camino |
| Estatus Programado Vivo | `[Estatus]="Programado en vivo"` | `#FF8C00` | `#FF8C00` | Naranja — programado en vivo |
| Estatus Programado Contrato | `[Estatus]="Programado Por Contrato"` | `#800080` | `#800080` | Morado — programado por contrato |
| Estatus Liberado | `[Estatus]="Liberado"` | `#008000` | `#008000` | Verde — servicio liberado |
| Estatus Terminados | `[Estatus]="Terminados"` | `#006400` | `#006400` | Verde oscuro — servicio finalizado |

**Configuración de cada regla:**
- **Tabla aplicada:** cambios
- **Columna formateada:** Estatus
- **Tipo de condición:** Expresión Yes/No configurada mediante el Expression Assistant
- **Formato visual:** Color de highlight (fondo) y color de texto con valor hexadecimal personalizado

---

## Correcciones aplicadas

Durante la configuración se detectó que **4 de las 6 reglas** (Programado Vivo, Programado Contrato, Liberado y Terminados) tenían la columna objetivo incorrectamente asignada a "No Servicio" en lugar de "Estatus". Se corrigió la selección de columna en cada una de las 4 reglas antes de guardar, verificando individualmente que la opción "Estatus" quedara seleccionada.

---

## Estado final

- Las 6 reglas aparecen correctamente en el panel lateral bajo **"cambios (6)"**
- Cada regla fue verificada con: nombre correcto, tabla "cambios", condición con expresión válida, columna "Estatus" seleccionada, y colores hexadecimales configurados
- La app fue **guardada exitosamente** (indicador de checkmark verde confirmado)
- El editor muestra **"No issues found"** — sin errores de validación en la aplicación
- El total de reglas de formato en la app es **8**: 6 nuevas en "cambios" + 2 preexistentes en "incidencias"

---

## Próximas modificaciones programadas

### 1. Habilitar y formalizar la vista de Devoluciones

- **Objetivo:** Crear o habilitar una vista dedicada para el seguimiento de devoluciones dentro de la app
- **Alcance:**
  - Revisar si existe una tabla o datos de devoluciones actualmente en la estructura de la app
  - Definir los campos necesarios para la vista (fecha, cliente, servicio, motivo, estatus de devolución, monto)
  - Configurar la vista con filtros, ordenamiento y formato visual adecuado
  - Aplicar reglas de formato condicional si se requiere distinguir estados de devolución
  - Integrar la vista en la navegación principal de la app para que sea accesible desde el menú
- **Dependencia:** Evaluar si los datos de devoluciones pueden obtenerse de las tablas existentes (cambios, registro) o si se requiere crear una **nueva base de datos/hoja de cálculo** como fuente de datos adicional para la app

### 2. Habilitar usuario de Sandra Torres para vista previa

- **Objetivo:** Dar acceso a Sandra Torres para que pueda revisar y validar la vista de Devoluciones antes de su publicación general
- **Alcance:**
  - Configurar el correo de Sandra Torres como usuario autorizado en la sección **Security > Users** de la app
  - Asignar el rol adecuado con permisos de solo lectura o vista previa
  - Compartir el enlace de preview de la app para que Sandra pueda acceder y validar
  - Recopilar su retroalimentación antes de habilitar la vista para todos los usuarios

### 3. Actualizar documentación del proyecto (appsheet.md)

- **Objetivo:** Crear o actualizar el archivo `appsheet.md` como documento de referencia técnica que describa a detalle la configuración actual y las nuevas modificaciones de la app
- **Contenido a documentar:**
  - Estructura actual de la app: tablas, columnas, relaciones y tipos de datos
  - Catálogo completo de vistas con su configuración (filtros, ordenamiento, tipo de vista)
  - Reglas de formato existentes (las 8 actuales) con sus condiciones y colores
  - Configuración detallada de la nueva vista de Devoluciones (campos, filtros, permisos, formato)
  - **Evaluación de base de datos adicional:** documentar si se creó una nueva fuente de datos para devoluciones, su estructura de columnas, tipo de datos por campo, y la conexión con la app
  - Usuarios y roles configurados en Security
  - Historial de cambios de la app con fechas
- **Ubicación:** `Projects/docs/appsheet.md`
- **Importancia:** Este archivo servirá como fuente de verdad para cualquier modificación futura de la app, permitiendo que tanto el equipo como los agentes de IA tengan contexto completo de la configuración sin necesidad de explorar el editor manualmente cada vez

**Nota:** Estas modificaciones están organizadas como siguientes pasos. Se ejecutarán una vez confirmados los requerimientos específicos de la vista de Devoluciones, la necesidad de una base adicional, y el correo de acceso de Sandra Torres.
