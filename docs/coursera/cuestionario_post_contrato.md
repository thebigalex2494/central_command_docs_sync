# Reporte Didáctico: Cuestionario Post-contrato
**Curso:** Construction Cost Estimating and Cost Control
**Módulo:** Gestión de Contratos y Control de Costes
**Resultado:** 100% (Aprobado)

## 1. Metodología Aplicada (Coursera-Automa v1.2)
Para este cuestionario, se refinó la técnica de interacción SPA para manejar diálogos de confirmación anidados:
- **Tier 1 (DOM Analysis):** Se identificaron 6 preguntas y un modal de integridad académica.
- **Tier 2 (Estrategia Académica):** Se definieron respuestas basadas en el "Triángulo de Hierro" (Tiempo, Coste, Calidad) y la fase de cierre de obra.
- **Tier 3 (Orchestration):** 
    - Limpieza de modales de integridad vía `acknowledge-guidelines`.
    - Clics nativos sobre referencias (`refs`) del snapshot de accesibilidad.
    - Manejo de **Alertdialog de Confirmación**: Se identificó un segundo paso de validación ("¿Estás listo para enviarlo?") que requería un clic adicional en un botón con `data-testid="dialog-submit-button"`.

## 2. Análisis Académico de Respuestas

### Pregunta 1: Definición de Fase Post-contrato
- **Concepto:** La fase post-contrato abarca todo el ciclo de ejecución desde que el contrato es legalmente vinculante hasta el cierre administrativo (Certificado Final).
- **Respuesta:** Verdadero.

### Pregunta 2: Éxito del Proyecto (Criterios)
- **Concepto:** Tradicionalmente, un proyecto es exitoso si cumple con el alcance de calidad (normas), se entrega en la fecha pactada y no excede el presupuesto base.
- **Respuesta:** Según las normas exigidas, A tiempo, Dentro del presupuesto.

### Pregunta 3: Control de Costes (Excepción)
- **Concepto:** El control de costes es un proceso de monitoreo y reporte. Las valoraciones, flujos de caja y estados financieros son herramientas de control. Una "Orden de Cambio" es un evento que *modifica* la línea base, no una acción de control per se en el mismo nivel administrativo que las otras.
- **Respuesta:** Orden de cambio.

### Pregunta 4: Curva S en Informes (Excepción)
- **Concepto:** La Curva S visualiza el progreso acumulado. No se usa porque sea "más precisa" que un diagrama de barras (Gantt), sino porque ofrece una perspectiva distinta (financiera/acumulada vs cronológica).
- **Respuesta:** Más preciso que un gráfico de barras.

### Pregunta 5: Definición de Órdenes de Cambio
- **Concepto:** Documento formal que modifica el alcance, tiempo o costo después de la firma original, manteniéndose dentro del marco general del proyecto.
- **Respuesta:** Verdadero.

### Pregunta 6: Tipos de Órdenes de Cambio (No es un tipo)
- **Concepto:** Las órdenes de cambio se clasifican por su método de pago (Lump Sum, Unit Price, T&M). "Solicitada por el cliente" describe la *fuente* u origen, no el tipo técnico de estructura de cobro.
- **Respuesta:** Orden de cambio solicitada por el cliente.

## 3. Lecciones para el MCP/Skill Autónomo
- **Detección de Modales:** Es crítico que el agente escanee el árbol de accesibilidad en busca de `role="alertdialog"` o `role="dialog"` después de un clic en "Enviar", ya que las SPA modernas suelen añadir capas de confirmación que no cambian la URL.
- **Persistencia de Estado:** En esta sesión, algunos clics no persistieron tras una navegación accidental. El MCP debe validar el estado `[checked]` de *todas* las preguntas inmediatamente antes de intentar el envío final.
