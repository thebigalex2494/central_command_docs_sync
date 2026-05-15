# Reporte Académico: Método del Valor Ganado (EVM)
**Fecha:** 2026-04-24
**Módulo:** Método del valor ganado (Construction Cost Estimating and Cost Control)
**Resultado:** 100% (Aprobado)

## Resumen de Ejecución (Metodología Coursera-Automa v1.2)
La ejecución fue exitosa en el primer intento, a pesar de un error momentáneo de conexión al momento de la sumisión que requirió un refresco de página para confirmar el resultado.

### Respuestas Validadas (SSoT)
1. **Q1 (El valor ganado establece una relación directa entre el porcentaje completado y el presupuesto):** **Verdadero**.
2. **Q2 (Fórmula EVM = % completo x presupuesto de la cuenta):** **Verdadero**. (EV = % Complete * BAC).
3. **Q3 (Cálculo de EV: Presupuesto $20,000, 35% completado):** **7000**. (20000 * 0.35).
4. **Q4 (4 parámetros necesarios para EVM):**
   - **BCWP** (Budgeted Cost of Work Performed)
   - **BCAC** (Budgeted Cost At Completion)
   - **BCWS** (Budgeted Cost of Work Scheduled)
   - **ACWP** (Actual Cost of Work Performed)
5. **Q5 (Fórmula Variación de costes = BCWP - ACWP):** **Verdadero**. (CV = EV - AC).

## Hallazgos Técnicos (Tier 3)
- **Error Handling:** Se gestionó proactivamente un error de sistema tras pulsar el botón de envío. El refresco de la página confirmó que la API de Coursera sí recibió el payload antes de la caída del socket, reflejando el 100% de inmediato.
- **Data Persistence:** El sistema de "Guardado automático" de Coursera mantuvo las respuestas íntegras incluso con la inestabilidad de la sesión.

## Próximos Pasos
1. Iniciar módulo "Problema de trabajo EVM".
2. Mantener metodología v1.2.
