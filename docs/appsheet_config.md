# AppSheet Config — Control de servicios GDV

> Technical reference for the AppSheet "Control de servicios GDV" application.
> Last audit: 2026-02-11

---

## App Identity

| Property | Value |
|----------|-------|
| **Name** | Control de servicios GDV |
| **App ID** | `5c17d482-59b2-4ff1-a8f7-62c7989a7733` |
| **Version** | 1.000257 |
| **Category** | Custom Apps |
| **Editor URL** | `https://www.appsheet.com/template/AppDef?appName=servicios010125-24894927&appId=5c17d482-59b2-4ff1-a8f7-62c7989a7733` |
| **App Folder** | `/appsheet/data/servicios010125-24894927` |
| **Data Source** | Google Sheets: `control_servicios` (principal) + 3 externos: `PERSONAL (BD)`, `Parque Vehicular (BD)`, `RUTAS (BD)` |
| **Auth Provider** | Google (sign-in required, user list) |

---

## Tables (11)

### cambios (38 columns) — Main services table

Source: `control_servicios/cambios`

| Column | Type | Notes |
|--------|------|-------|
| id_registro | Text | **Key** |
| Fecha | Text | |
| Usuario | Text | PII |
| No Servicio | Number | |
| Cita | DateTime | |
| Finaliza | DateTime | |
| Cliente | Text | |
| Unidad | Number | |
| Ruta | Text | |
| PAX | Number | |
| PAX Solicitado | Number | |
| Operador 1 | Text | |
| Operador 2 | Text | |
| Lugar de Cita | Text | |
| Incentivo | Text | |
| Monto Incentivo | Text | |
| Descuento | Text | |
| Tipo Comision | Text | |
| Porcentaje | Text | |
| Comision | Text | |
| IVA | Text | |
| Retencion | Text | |
| Tarifa | Text | |
| Moneda | Text | |
| Num Guia | Text | |
| Cupon | Text | |
| Observaciones | Text | |
| Capturado | Text | |
| Tipo de Operacion | Text | |
| Estatus | Enum | Values: En Ruta, Cancelados, Liberado, Programado en vivo, Terminados, Programado Por Contrato |
| Cupon Final | Text | |
| PAX Reales | Text | |
| Observaciones Operador | Text | |
| Tiempo | Text | |
| Coordinador | Text | |
| Tel Coordinador | Text | |
| Agente Venta | Text | |
| Itineratio | Text | |

**Actions:** Actualizar reporte, Call Phone (Tel Coordinador), Send SMS (Tel Coordinador)

---

### cat_clientes (8 columns) — Client catalog

| Column | Type | Notes |
|--------|------|-------|
| ID_Cliente | Number | **Key** |
| Cliente | Text | Label |
| Zona | Text | |
| Categoria | Enum | |
| Gerente | Ref → cat_personal | |
| Coordinador | Ref → cat_personal | |
| Supervisor | Ref → cat_personal | |

**Actions:** 6 (system-generated)

---

### cat_incidencias (8 columns) — Incident types catalog

| Column | Type | Notes |
|--------|------|-------|
| ID | Text | **Key** |
| Prioridad | Enum | |
| Tipo | Enum | |
| Clasificacion | Text | Label |
| Area_Responsable | Text | |
| Activo | Text | |
| Related incidencias | List (Virtual) | `=REF_ROWS("incidencias","Clasificacion")` |

**Actions:** 3 (system-generated)

---

### cat_personal (9 columns) — Staff catalog

| Column | Type | Notes |
|--------|------|-------|
| ID | Text | **Key** |
| Nombre | Text | Label |
| Rol | Text | |
| Activo | Text | |
| Correo | Email | Used in security filters |
| Related cat_clientes By Gerente | List (Virtual) | REF_ROWS |
| Related cat_clientes By Supervisor | List (Virtual) | REF_ROWS |
| Related cat_clientes By Coordinador | List (Virtual) | REF_ROWS |

**Actions:** 4 (system-generated)

---

### incidencias (21+ columns) — Incident tracking

| Column | Type | Notes |
|--------|------|-------|
| ID_Incidencia | Text | **Key** |
| No_Servicio | Ref → cambios | |
| Fecha | Date | `init=TODAY()` |
| Hora | Time | `init=NOW()` |
| Clasificacion | Ref → cat_incidencias | Label, Required |
| Area_Responsable | Text | `init=[Clasificacion].[Area_Responsable]` |
| Prioridad | Text | `init=[Clasificacion].[Prioridad]` |
| Operador_Nomina | Text | `init=[No_Servicio].[Operador 1].[ID NOMINAX]` |
| Observaciones | Text | |
| Plan_Accion | Text | Appended by "Agregar Seguimiento" action |
| Estatus | Enum | `init="Abierta"`, values: Abierta, Cerrada |
| Monto_Reembolso | Text | |
| Tipo_Reembolso | Text | |
| Supervisor | Text | |
| Resuelto | Text | Set by "Cerrar Incidencia" action |
| Cliente_Nombre | Text | |
| Prioridad_VC | Text | Used in format rules |
| Dias_Abierta | Text | |
| Resumen | Text | |
| Area_VC | Text | |

**Actions (8):**
- **Agregar Seguimiento** — Set `Plan_Accion = CONCATENATE([Plan_Accion], " | ", TEXT(NOW(), "DD/MM/YYYY HH:MM"), " - ", USEREMAIL(), ": [NOTA]")`
- **Cerrar Incidencia** — Set `Estatus="Cerrada"`, `Resuelto=true`
- **Escalar** — Set `Clasificacion="Critica"`
- **Llamar Coordinador** — Phone call to `[No_Servicio].[Tel Coordinador]`
- Add, Edit, View Ref (Clasificacion), View Ref (No_Servicio) — system

---

### inicio — Main data entry view

Mirrors `cambios` structure. This is the table users edit directly. Changes trigger the "Historial de cambios" bot.

**Actions:** 8

---

### registro — Audit log

Populated automatically by the bot when `inicio` is modified. Stores snapshots of service data at each change point.

| Key Fields | Expression |
|------------|------------|
| id_registro | `[Instance Id]` |
| Usuario | `useremail()` |
| Fecha | `NOW()` |
| All other fields | Copied from `inicio` row values |

**Actions (5):** Add, Call Phone (Tel Coordinador), Delete, Edit, Send SMS (Tel Coordinador)

---

### OPERADORES — Operators reference

**Source:** Google Sheet externo `PERSONAL (BD)`, tab `OPERADORES`
**Data Source:** google (Sheets)

Reference table for operator/driver data. Referenced by `inicio.Operador 1` and `inicio.Operador 2`.

---

### pendientes_revision — Pending review items (5 columns)

Source: `control_servicios/pendientes_revision`

| Column | Type | Notes |
|--------|------|-------|
| No_Servicio | Number | Reference to service |
| Campo_Erroneo | Text | Field with discrepancy |
| Valor_Actual (Registro) | Text | Current value in registro |
| Valor_Esperado (Inicio) | Text | Expected value from inicio |
| Estatus_Revision | Text | PENDIENTE / REVISADO |

**Actions:** 3

**Data volume:** ~156 active rows (as of 2026-03-10)

---

### PV — Parque Vehicular (14 columns)

**Source:** Google Sheet externo `Parque Vehicular (BD)`, tab `PV`
**Data Source:** google (Sheets)

| Column | Type | Notes |
|--------|------|-------|
| ESTATUS | Text | |
| LOCALIDAD | Text | |
| UNIDAD | Number | **Key**, Label |
| TIPO | Text | |
| CARROCERIA | Text | |
| CHASIS | Text | |
| AÑO | Number | |
| PASAJEROS | Number | |
| No. de Serie | Text | |
| No. De Motor | Number | |

Referenced by `inicio.Unidad` (Ref).

---

### RUTAS — Routes reference (10 columns)

**Source:** Google Sheet externo `RUTAS (BD)`, tab `RUTAS`
**Data Source:** google (Sheets)

| Column | Type | Notes |
|--------|------|-------|
| Ruta | Text | **Key** |
| Descripción | Text | **Key**, Label |
| Tipo Ruta | Enum | |
| Origen | Text | |
| Destino | Text | |
| Kms. Ruta | Number | |
| Tiempo | Decimal | |
| Estatus | Enum | |
| Related inicios | List (Virtual) | `=REF_ROWS("inicio", "Ruta")` |

Referenced by `inicio.Ruta` (Ref).

---

## Views

### Primary Navigation

| View | Data Source | Type | Notes |
|------|-----------|------|-------|
| **Servicios del dia** | inicio | — | Main daily services view |
| **Historial de cambios** | Historico de cambios | Table | Sort: Fecha, Group: Usuario + Cliente, 18 visible columns, Column width: Default |
| **Reporte de cambios** | — | — | Reporting view |

### Menu Navigation

| View | Data Source | Notes |
|------|-----------|-------|
| **Asignacion de clientes** | cat_clientes | Client assignment management |
| **Historial Incidencias** | incidencias | All incidents history |
| **Incidencias Abiertas** | Incidencias_Abiertas | Filtered to open incidents |
| **Nueva Incidencia** | incidencias | Form view for creating incidents |

### System Generated Views (by table)

| Table | Views |
|-------|-------|
| cambios | 1 (cambios_Detail) |
| cat_clientes | 3 |
| cat_incidencias | 2 |
| cat_personal | 2 |
| Historico de cambios | 1 |
| Hoja del dia | 2 |
| incidencias | 3 |
| Incidencias_Abiertas | 2 |
| Incidencias_Cerradas | 2 |
| inicio | 3 |
| OPERADORES | 1 |
| pendientes_revision | 2 |
| PV | 1 |
| registro | 2 |
| Reporte de cambios | 1 |
| RUTAS | 1 |

### Historial de cambios — View Detail

- **Type:** Table
- **Data:** Historico de cambios (slice)
- **Position:** Middle (primary nav)
- **Sort by:** Fecha
- **Group by:** Usuario, Cliente
- **Column order:** Manual
- **Visible columns (18):** No Servicio, Cita, Finaliza, Cliente, Unidad, Ruta, PAX, PAX Solicitado, Operador 1, Operador 2, Lugar de Cita, Num Guia, Cupon, Observaciones, Capturado, Tipo de Operacion, Estatus, Observaciones Operador
- **Column width:** Default
- **QuickEdit:** Disabled

---

## Format Rules (8 total)

### cambios table (6 rules)

All rules target the **Estatus** column.

| Rule | Condition | Highlight | Text Color |
|------|-----------|-----------|------------|
| Estatus Cancelados | `[Estatus]="Cancelados"` | `#FF0000` (red) | `#FF0000` |
| Estatus En Ruta | `[Estatus]="En Ruta"` | `#0000FF` (blue) | `#0000FF` |
| Estatus Programado Vivo | `[Estatus]="Programado en vivo"` | `#FF8C00` (orange) | `#FF8C00` |
| Estatus Programado Contrato | `[Estatus]="Programado Por Contrato"` | `#800080` (purple) | `#800080` |
| Estatus Liberado | `[Estatus]="Liberado"` | `#008000` (green) | `#008000` |
| Estatus Terminados | `[Estatus]="Terminados"` | `#006400` (dark green) | `#006400` |

### incidencias table (2 rules)

| Rule | Condition | Target Column | Highlight | Text |
|------|-----------|---------------|-----------|------|
| Prioridad Alta | `[Prioridad_VC]="Alta"` | Prioridad | — | — |
| Prioridad Critica | `[Prioridad_VC]="Critica"` | Prioridad | `red` | `red` |

---

## Automation (1 Bot)

### Bot: Historial de cambios

**Event: "Inicio modificado"**
- Source: App Change
- Table: `inicio`
- Triggers on: **Adds + Updates** (not Deletes)
- Condition: none
- Bypass Security Filters: No

**Process Step: "Registro"**
- Type: Run a data action → **Add new rows**
- Target table: `registro`
- Maps **33 fields** from `inicio` to `registro`:
  - `id_registro` = `[Instance Id]`
  - `Usuario` = `useremail()`
  - `Fecha` = `NOW()`
  - All other fields: direct copy from source row (`[No Servicio]`, `[Cita]`, `[Ruta]`, `[Unidad]`, `[Lugar de Cita]`, `[Estatus]`, `[Agente Venta]`, `[Capturado]`, `[Cliente]`, `[Itineratio]`, `[Observaciones]`, `[Observaciones Operador]`, `[Coordinador]`, `[PAX]`, `[PAX Reales]`, `[PAX Solicitado]`, `[Tarifa]`, `[Operador 1]`, `[Operador 2]`, `[Comision]`, `[Finaliza]`, `[Tel Coordinador]`, `[IVA]`, `[Moneda]`, `[Cupon]`, `[Tiempo]`, `[Cupon Final]`, `[Descuento]`, `[Incentivo]`, `[Monto Incentivo]`, `[Num Guia]`, `[Porcentaje]`, `[Retencion]`, `[Tipo Comision]`, `[Tipo de Operacion]`)

**Purpose:** Audit trail — every add/update to `inicio` creates a snapshot in `registro`.

---

## Security

### Authentication

| Setting | Value |
|---------|-------|
| Require sign-in | **Yes** |
| Provider | **Google** |
| Allow all signed-in users | **No** (managed user list) |

### Security Filters

**Filter applied to: `cambios`, `inicio`, `registro`**

```
=OR(
  NOT(IN(USEREMAIL(), cat_personal[Correo])),
  IN(
    [Cliente],
    SELECT(
      cat_clientes[Cliente],
      OR(
        [Gerente].[Correo] = USEREMAIL(),
        [Coordinador].[Correo] = USEREMAIL(),
        [Supervisor].[Correo] = USEREMAIL()
      )
    )
  )
)
```

**Logic:**
- If user is NOT in `cat_personal` (not staff) → sees all data (app creator / admin)
- If user IS in `cat_personal` → only sees clients where they are assigned as Gerente, Coordinador, or Supervisor

**Filter applied to: `incidencias`**

```
=OR(
  NOT(IN(USEREMAIL(), cat_personal[Correo])),
  IN(
    [No_Servicio].[Cliente],
    SELECT(
      cat_clientes[Cliente],
      OR(
        [Gerente].[Correo] = USEREMAIL(),
        [Coordinador].[Correo] = USEREMAIL(),
        [Supervisor].[Correo] = USEREMAIL()
      )
    )
  )
)
```

**Note:** Uses `[No_Servicio].[Cliente]` (dereference through Ref) instead of `[Cliente_Nombre]` because `Cliente_Nombre` is a virtual column and security filters don't support virtual columns.

**Tables WITHOUT filter:** cat_clientes, cat_incidencias, cat_personal, OPERADORES, pendientes_revision, PV, RUTAS

### Known Active Users

| Email | Role (inferred) |
|-------|-----------------|
| amendez@delvalletransportes.com.mx | Admin / Preview |
| lsoto@delvalletransportes.com.mx | Power user (27 entries) |
| gdv1super@gmail.com | Supervisor (8 entries) |
| yramirez@delvalletransportes.com.mx | User (2 entries) |
| gdv3super@gmail.com | Supervisor (1 entry) |

---

## Integrations

### Google Apps Script (GAS)

**Project:** "Reporte de cambios" (16 files)
**URL:** `https://script.google.com/u/0/home/projects/1PqppLSG_-zgHCbVtyBeWPvI4ffLnS05sN_l-4z-FCeqOTkC-zMOGDpnd/edit`

#### Web App Action
- **Action:** "Actualizar reporte" (on `cambios` table)
- **URL:** `https://script.google.com/macros/s/AKfycbyRRrV7IZSKnASTfD9sNP7af1LWjCj1sONLdmn23k6HVNuKHJ1NvjW7p18pifVeg55cxw/exec`
- **Launch:** External browser
- **Position:** Primary action
- **Purpose:** Triggers report update/generation via GAS web app

#### Installed Triggers (3)

| Function | Event | Purpose |
|----------|-------|---------|
| `ejecutarYEnviarReporteEmail` | Time-based (daily) | Generates and sends daily service report by email |
| `onEditAuditTrail` | Spreadsheet - On edit (inicio) | Copies edited rows from `inicio` to `registro` with timestamp — fallback audit trail independent of AppSheet bot |
| `onEditIncidentAlert` | Spreadsheet - On edit (incidencias) | Sends email alert when incident classification is changed to "Critica" |

#### GAS Project Files

| File | Purpose |
|------|---------|
| config.gs | Constants, sheet names (`NOMBRES_HOJAS`), settings |
| Code.gs | Main entry point |
| SincroInicio.gs | Sync logic for inicio sheet |
| ManejoHojas.gs | Sheet management utilities |
| LogicaNegocio.gs | Business logic |
| ValidacionDatos.gs | Data validation |
| ReporteEmail.gs | Email report generation |
| email_template.html | HTML template for email reports |
| UI.gs | Custom menu and UI dialogs |
| setup.gs | Initial setup functions |
| setup_data.gs | Setup data/configuration |
| ApiHandler.gs | API endpoint handling |
| TriggerManager.gs | Trigger management utilities |
| **AuditTrail.gs** | **Audit trail fallback + critical incident alerts (added 2026-02-11)** |
| index.html | Web app frontend |
| appsscript.json | Manifest |

#### AuditTrail.gs Details

**`onEditAuditTrail(e)`** — Fired on every edit in the spreadsheet:
- Only processes edits on the `inicio` sheet
- Copies the full edited row to `registro` sheet with: timestamp, user email, action type ("UPDATE")
- Deduplication: checks last 20 rows of `registro` within 60-second window to avoid duplicate entries
- Independent of AppSheet bot — works as fallback if trial expires

**`onEditIncidentAlert(e)`** — Fired on every edit in the spreadsheet:
- Only processes edits on the `incidencias` sheet where the `Clasificacion` column changes
- Looks up the priority of the new classification in `cat_incidencias`
- If priority is "Critica", sends email to `coordinacion@delvalletransportes.com.mx`
- Email includes: incident ID, service number, classification, observations

**`installAuditTrailTriggers()`** — Setup function:
- Creates both onEdit installed triggers
- Checks for existing triggers to avoid duplicates
- Run once to install; already executed on 2026-02-11

---

## Data Model Diagram

```
cat_personal (staff)
  ├── cat_clientes.Gerente (Ref)
  ├── cat_clientes.Coordinador (Ref)
  └── cat_clientes.Supervisor (Ref)

cat_clientes (clients)
  └── Used in security filters for row-level access

cat_incidencias (incident types)
  └── incidencias.Clasificacion (Ref)

cambios (services - read/historical)
  └── incidencias.No_Servicio (Ref)

inicio (services - editable)
  ├── [Bot: Historial de cambios] → registro (audit log)
  └── [GAS: onEditAuditTrail] → registro (fallback audit)

registro (audit trail)
  └── Auto-populated by bot on inicio Add/Update + GAS fallback on edit
```

---

## Pending Modifications

1. **Vista de Devoluciones** — Create/enable a dedicated returns tracking view
2. **Acceso Sandra Torres** — Configure user access for preview/validation
3. **Evaluate additional data source** — Determine if returns need a separate Google Sheet

---

## Audit History

| Date | Action | Details |
|------|--------|---------|
| 2026-02-10 | Format rules created | 6 conditional formatting rules for Estatus column on cambios |
| 2026-02-10 | Bug fixes | Corrected 4 rules targeting wrong column (No Servicio → Estatus) |
| 2026-02-11 | Full audit | Complete technical audit of all tables, views, actions, automation, security, settings |
| 2026-02-11 | Hardening MVP | Phase 1.2: Corrected data types (Tarifa→Price, Comision→Decimal, Porcentaje→Percent, Monto Incentivo→Price, Descuento→Decimal) across cambios/inicio/registro |
| 2026-02-11 | Hardening MVP | Phase 2.1: Verified offline mode enabled |
| 2026-02-11 | Table permissions | Configured table-level permissions (see Table Permissions section) |
| 2026-02-11 | Hardening MVP | Phase 1.3: Added Required validations — inicio: No Servicio, Cita, Cliente; incidencias: No_Servicio, Observaciones |
| 2026-02-11 | Hardening MVP | Phase 5.1: Added security filter to incidencias using [No_Servicio].[Cliente] dereference — same logic as cambios/inicio/registro |
| 2026-02-11 | Hardening MVP | Phase 4.1: Deployed AuditTrail.gs to existing GAS project — onEditAuditTrail (audit fallback) + onEditIncidentAlert (critical incident email) — 2 new triggers installed |
| 2026-02-11 | Hardening MVP | Phase 1.4: Added column descriptions (tooltips) — 5 in incidencias + 5 in inicio — for field operator guidance |

---

## Required Field Validations

| Table | Column | Required? | Notes |
|-------|--------|-----------|-------|
| **inicio** | No Servicio | ✓ | Service number must be provided |
| **inicio** | Cita | ✓ | Appointment datetime required |
| **inicio** | Cliente | ✓ | Client must be specified |
| **incidencias** | Clasificacion | ✓ | Already required (pre-existing) |
| **incidencias** | No_Servicio | ✓ | Must link to a service |
| **incidencias** | Observaciones | ✓ | Incident description required |

---

## Column Descriptions

Tooltips added to key columns to guide field operators.

### incidencias table (5 descriptions)

| Column | Description |
|--------|-------------|
| No_Servicio | "Selecciona el servicio relacionado con esta incidencia" |
| Clasificacion | "Selecciona el tipo de incidencia del catalogo" |
| Observaciones | "Describe detalladamente lo sucedido en la incidencia" |
| Plan_Accion | "Registro de seguimiento y acciones tomadas" |
| Estatus | "Estado actual de la incidencia: Abierta, En Proceso o Cerrada" |

### inicio table (4 descriptions)

| Column | Description |
|--------|-------------|
| No Servicio | "Numero unico que identifica el servicio" |
| Cita | "Fecha y hora programada del servicio" |
| Cliente | "Nombre del cliente del servicio" |
| Ruta | "Ruta asignada al servicio" |
| Estatus | "Estado actual del servicio" |

---

## Table Permissions

| Table | Updates | Adds | Deletes | Notes |
|-------|---------|------|---------|-------|
| **cambios** | - | - | - | **Read-Only** — historical data, no modifications |
| **inicio** | ✓ | ✓ | ✗ | Main editable table, no deletes to prevent data loss |
| **registro** | ✗ | ✓ | ✗ | Audit trail — bot adds rows, no edits/deletes allowed |
| **cat_clientes** | ✓ | ✓ | ✗ | Catalog — editable but no deletes |
| **cat_incidencias** | ✓ | ✓ | ✗ | Catalog — editable but no deletes |
| **cat_personal** | ✓ | ✓ | ✗ | Catalog — editable but no deletes |
| **incidencias** | ✓ | ✓ | ✗ | Incident tracking — no deletes |
| **OPERADORES** | ✓ | ✓ | ✓ | Reference table — default permissions |
| **pendientes_revision** | ✓ | ✓ | ✓ | Review queue — default permissions |
| **PV** | ✓ | ✓ | ✓ | Reference table — default permissions |
| **RUTAS** | ✓ | ✓ | ✓ | Reference table — default permissions |
