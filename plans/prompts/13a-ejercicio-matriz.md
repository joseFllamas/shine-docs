# 13a. Ejercicio nuevo: matriz de discriminación visual (7.6 del docx)

- **Modelo recomendado: Fable.** Primer ejercicio full-stack de la fase: fija el patrón (bundles Drupal + tipos TS + player + tracking) que 13b y 13c copiarán. Cruza backend y app con decisiones de modelado.
- Fase: 13 (nuevos ejercicios, prioridad 1). Alcance: backend + app.
- Dependencias: FASE 12 completa (y el hito de demo con familias, si se respeta).

## Prompt

Lee `plans/02-plan-ejecucion-v1.md` (FASE 13), la especificación del ejercicio 7.6 en "ejercicios - tipo.docx" (raíz del repo; si no puedes leer el docx, pide al usuario que exporte esa sección a texto), el diseño `plans/design/07-ejercicios-nuevos.md`, y sobre todo el checklist de `docs/activity-system.md` y las convenciones de nombres de `docs/conventions.md`.

Resumen del ejercicio: matriz de letras/símbolos donde el usuario marca todas las apariciones del objetivo (discriminación visual). Sin audio. Métricas: aciertos y tiempo.

Tareas backend (`shine/`, con `ddev`, config export siempre):

1. Node type `activity_letter_matrix` siguiendo el patrón de los existentes: `field_main_area` (requerido), `field_secondary_area`, `field_activity_level` (requerido, múltiple), más los campos propios del contenido de la matriz (decide el modelado leyendo el docx: dimensiones, objetivo, celdas; prefiere campos simples o paragraphs según el patrón de `activity_image_position`).
2. Bundles de tracking paralelos: `logletermatrix` en `activitylog` y `summary_letter_matrix` en `activitysummary`, con los campos comunes del storage compartido (incluidos `field_main_area`, `field_activity_level`, `field_correct_count`, `field_items_count` de la fase 10). El log usa `field_correct` por ítem como el bundle de imágenes.
3. Término de taxonomía `areas` para discriminación visual si no existe.
4. Permisos JSON:API para el rol authenticated (mismo patrón que los bundles existentes) y export de config.
5. Crear 1 o 2 actividades de ejemplo con contenido real para poder probar.

Tareas app (`shine-app/`):

6. Tipos TS espejo (`src/types/`), player `LetterMatrixPlayer` en `src/components/activities/` sobre `ActivityBase` (intro, playing, completed), rama en `ActivityRouter`, y funciones de tracking en `src/lib/api/tracking.ts` que creen logs + summary del bundle correcto con área/nivel/aciertos e iteración por actividad.
7. Diseño con los tokens y componentes del design system; celdas con área táctil mínima 48px; feedback de acierto en positivo (sin marcar "fallos" en rojo).

Verificación:
- El ejercicio es jugable dentro de una sesión mixta con los tipos antiguos.
- Sus métricas aparecen en Mi progreso (incluida la sección de aciertos).
- `ddev drush cim -y` idempotente; `npx tsc --noEmit` y `npm test` limpios.

Al terminar: marca los checkboxes en `plans/02-plan-ejecucion-v1.md`, actualiza `docs/progress.md` y `docs/activity-system.md` con lo aprendido del patrón (13b y 13c lo reutilizarán).
