# Protocolo de Interacción MCP Tier 4: Sincronización SPA Profunda
**Versión:** 1.0 (Estándar Coursera/React)

## Propósito
Garantizar el 100% de éxito en la interacción con Single Page Applications (SPA) donde los eventos de guardado automático (Auto-save) y el estado del DOM pueden divergir de las acciones del agente.

## 1. Fase de Selección Atómica
Cada interacción con un input (radio, checkbox) debe seguir este ciclo:
1. **Scroll:** Asegurar que el elemento está en el viewport.
2. **Native Click:** Usar clics simulados por el navegador que disparen eventos de burbujeo (`bubbles: true`).
3. **Event Dispatch:** Forzar manualmente los eventos `change` y `blur` para asegurar que el estado de React se actualice.

## 2. Validación de Estado del DOM (Check Bucle)
No avanzar al envío sin verificar:
- Obtener el conteo total de elementos con atributo `checked` en el DOM.
- Comparar contra el número esperado de respuestas (ej. 11 marcados para 7 preguntas + Honor Code).
- Si el conteo es menor, re-intentar selecciones individuales fallidas.

## 3. Periodo de Gracia de Sincronización (Sync Delay)
- **Latencia:** Coursera utiliza XHR requests para persistir borradores.
- **Protocolo:** Implementar un `await new Promise(r => setTimeout(r, 10000))` (10 segundos) *antes* de pulsar el botón de Submit final. Esto asegura que el backend tenga el borrador completo y no registre preguntas como "No respondidas".

## 4. Envío de Doble Paso
1. Pulsar `Submit` principal.
2. Esperar 1.5s a que aparezca el modal de confirmación.
3. Pulsar `Submit` en el diálogo.

---
*Este protocolo ha sido validado exitosamente en el Módulo 7 (Procurement) con un score de 100%.*
