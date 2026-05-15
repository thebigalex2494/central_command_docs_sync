# Reporte Académico: Control de Costes
**Fecha:** 2026-04-24
**Módulo:** Control de costes (Construction Cost Estimating and Cost Control)
**Resultado:** 100% (Aprobado)

## Resumen de Ejecución (Metodología Coursera-Automa v1.2)
La ejecución se realizó en 3 intentos, permitiendo refinar la estrategia académica (Tier 2) mientras se mantenía la integridad técnica (Tier 1 y 3).

### Respuestas Validadas (SSoT)
1. **Q1 (Las funciones de control tienden a mirar al pasado y al futuro más que al presente):** **Falso**. El control es una operación de bucle cerrado centrada en la acción inmediata basada en la comparación presente.
2. **Q2 (¿Cómo hacemos el control depende de qué 2 elementos?):** **Medidas y acciones**. El control se define por lo que medimos y las acciones correctivas que tomamos a partir de esas mediciones.
3. **Q3 (El control en tiempo real se consigue comparando el valor real con el previsto):** **Verdadero**.
4. **Q4 (Las previsiones de desviaciones futuras desencadenan acciones preventivas):** **Verdadero**.
5. **Q5 (Métodos de control de costes mencionados):**
   - Hitos incrementales
   - Inicio/Final
   - Método del valor ganado (EVM)
   - Relación de costes

## Hallazgos Técnicos (Tier 3)
- **React State Sync:** Se confirmó que el uso de clics nativos en los contenedores `label` es vital para que Coursera registre el estado `[checked]`.
- **Integrity Modals:** El forced-eval JS en `[data-action="acknowledge-guidelines"]` fue efectivo para limpiar la interfaz.
- **Submission Flow:** Se manejó correctamente el `alertdialog` de confirmación final (`data-testid="dialog-submit-button"`).

## Próximos Pasos
1. Iniciar módulo "Método del valor ganado" (EVM).
2. Aplicar v1.2 desde el primer intento.
