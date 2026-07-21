# 10a. Backend: denormalizar área/nivel/aciertos en activitysummary + backfill

- **Modelo recomendado: Fable.** Cambio de modelo de datos con backfill de históricos e índices: las decisiones equivocadas aquí son caras de revertir.
- Fase: 10 (capa de datos de estadísticas). Alcance: backend (`shine/`).
- Dependencias: 09a. Bugs: B2, B3.

## Prompt

Lee `plans/02-plan-ejecucion-v1.md` (secciones 2.1, 2.2 y FASE 10 Backend) y `plans/01-auditoria.md` (B2, B3). Trabaja en el backend Drupal (`shine/`, comandos con `ddev`). Sigue las convenciones de `docs/conventions.md` y el patrón de campos existente en los bundles de `activitysummary` (los 3 comparten storage). Preferir siempre APIs de core y config export sobre código custom.

Tareas:

1. **Campos nuevos en los 3 bundles de `activitysummary`** (`summary_vertical_text`, `summary3_vertical_text_blink`, `summary_image_position`), con storage compartido:
   - `field_main_area`: entity_reference a taxonomy `areas`.
   - `field_activity_level`: entity_reference a taxonomy `activity_level` (múltiple, como en los nodos de actividad).
   - `field_correct_count` y `field_items_count`: integers.
   Crearlos por config (UI o config YAML directo en `config/sync`) y exportar con `ddev drush cex -y`. Verificar que aparecen en JSON:API.

2. **Backfill de summaries históricos**: script idempotente (hook_update_N en el módulo `activitysummary`, o si es un one-off, un script drush php:script guardado en el repo) que recorra todos los summaries, cargue su nodo actividad via `field_activityid` y copie `field_main_area` y `field_activity_level`. Para el bundle de imágenes, derivar `field_correct_count` y `field_items_count` contando sus `field_activitylogs` (campo `field_correct`). Registrar cuántos se actualizan y cuántos quedan sin actividad resoluble.

3. **`field_seconds` en `activity_image_position`** (B3): decidir explícitamente. Recomendación del plan: las imágenes no tienen zona objetivo de tiempo (su métrica es precisión); si lo confirmas, NO añadas el campo y documenta la decisión en `docs/metrics-tracking.md`. Si encuentras razones en contra, añade el campo y documenta.

4. **Índices**: en `activitysummary_field_data`, índices para (uid, created) y (uid, field_activityid si es columna de esa tabla; si es tabla de campo aparte, indexar ahí). Vía hook_update con la API de schema de Drupal. Comprobar con `ddev mysql` que existen.

Verificación:
- `GET /jsonapi/activitysummary/summary_vertical_text` expone los campos nuevos.
- Backfill ejecutado: muestrear 3 summaries antiguos y comprobar área/nivel correctos frente a su nodo actividad.
- `ddev drush cim -y` idempotente tras el export.
- Sin errores en watchdog.

Al terminar: marca los checkboxes en `plans/02-plan-ejecucion-v1.md` y actualiza `docs/progress.md` y `docs/metrics-tracking.md`.
