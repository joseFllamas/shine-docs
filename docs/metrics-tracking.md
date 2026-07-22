# Métricas y tracking — Diseño del sistema de logging

## Concepto

Cada vez que un usuario completa una actividad se generan dos tipos de registros:

1. **`activitylog`** — un registro por cada elemento completado dentro de la actividad (granular)
2. **`activitysummary`** — un registro único por intento completo de la actividad (agregado)

Ambos quedan enlazados entre sí y referenciados a la actividad y sesión correspondientes.

---

## Estructura actual de métricas

### `activitylog.logverticaltext` — Log granular

Un registro por cada elemento (palabra/frase) que el usuario completa:

| Campo | Tipo Drupal | Descripción | Ejemplo |
|---|---|---|---|
| `label` | string | ID único del log | `A-42-15-3-2` |
| `uid` | entity_ref → user | Usuario que realizó la actividad | — |
| `field_activityid` | entity_ref → node.activity | Actividad en la que ocurrió | nid: 15 |
| `field_sessionid` | entity_ref → node.session | Sesión en la que ocurrió | nid: 42 |
| `field_activity_type` | string | Bundle/tipo de actividad | `vertical_text` |
| `field_item` | string | Texto del elemento completado | `"elefante"` |
| `field_weight` | integer | Posición del elemento en la lista | `2` |
| `field_iteration` | integer | Número de intento (1 = primera vez) | `3` |
| `field_time` | integer | Tiempo en ms desde el inicio o desde el elemento anterior | `1450` |
| `created` | timestamp | Cuándo se creó el registro | — |

**Formato del label**: `A-{sessionId}-{activityId}-{iteration}-{weight}`

---

### `activitysummary.summary_vertical_text` — Resumen por intento

Un registro único por cada vez que el usuario completa la actividad entera:

| Campo | Tipo Drupal | Descripción | Ejemplo |
|---|---|---|---|
| `label` | string | ID único del resumen | `Summary-42-15-3` |
| `uid` | entity_ref → user | Usuario | — |
| `field_activitylogs` | entity_ref → activitylog (M) | Todos los logs de este intento | [log1, log2, log3] |
| `field_activityid` | entity_ref → node.activity | Actividad | nid: 15 |
| `field_sessionid` | entity_ref → node.session | Sesión | nid: 42 |
| `field_activity_type` | string | Tipo de actividad | `vertical_text` |
| `field_iteration` | integer | Número de intento | `3` |
| `field_time` | integer | Tiempo total en ms (suma de todos los field_time) | `12500` |
| `created` | timestamp | Cuándo se creó | — |

---

## Cómo se calcula `field_iteration`

El Controller actual (y el futuro cliente Expo) cuenta cuántos `activitysummary` existen para el mismo `uid` + `activityid` antes de crear el nuevo:

```php
// Lógica actual en ActivitylogRegisterController.php
$query = $entityTypeManager->getStorage('activitysummary')
  ->getQuery()
  ->condition('uid', $uid)
  ->condition('field_activityid', $activityId)
  ->count()
  ->execute();
$iteration = $query + 1;
```

---

## Flujo de creación de métricas

```
Usuario completa actividad
         │
         ▼
Frontend prepara arrays:
  items[]    = ["elefante", "mariposa", "casa"]
  times[]    = [1200, 980, 1500]    (ms por elemento)
  weights[]  = [1, 2, 3]           (posición)
         │
         ▼
Para cada elemento i:
  POST /jsonapi/activitylog/logverticaltext
  → field_item = items[i]
  → field_time = times[i]
  → field_weight = weights[i]
  → field_iteration = (consulta previa + 1)
  → field_activityid = {activity-uuid}
  → field_sessionid = {session-uuid}
         │
         ▼
Recoger los UUIDs de los logs creados
         │
         ▼
POST /jsonapi/activitysummary/summary_vertical_text
  → field_activitylogs = [uuid1, uuid2, uuid3]
  → field_time = sum(times[])
  → field_iteration = (mismo valor)
  → field_activityid + field_sessionid
```

---

## Campos denormalizados de estadísticas (FASE 10, 2026-07-22)

Para que TODA la pantalla de estadísticas se resuelva con una única query de summaries (ver `plans/02-plan-ejecucion-v1.md` § 2.2), los **3 bundles** de `activitysummary` (`summary_vertical_text`, `summary3_vertical_text_blink`, `summary_image_position`) comparten estos campos adicionales (storage compartido):

| Campo | Tipo Drupal | Descripción |
|---|---|---|
| `field_main_area` | entity_ref → taxonomy `areas` (1) | Área principal, copiada del nodo actividad al crear el summary |
| `field_activity_level` | entity_ref → taxonomy `activity_level` (M) | Nivel(es), copiados del nodo actividad |
| `field_correct_count` | integer | Aciertos del intento (solo ejercicios con respuesta correcta) |
| `field_items_count` | integer | Total de ítems del intento |

- **Quién los rellena**: la app, al crear el summary (ya tiene el nodo actividad cargado; coste cero). Los counts solo aplican a ejercicios con noción de acierto (hoy: imágenes, via `field_correct` de sus logs).
- **Backfill histórico**: `shine/scripts/backfill-summary-area-level.php` (drush php:script, idempotente) copia área/nivel desde el nodo actividad y deriva los counts de imágenes contando `field_activitylogs`.
- **Requisito de contenido**: los nodos actividad deben tener `field_main_area` asignado para que el área llegue a los summaries (el "required" del campo solo aplica en formulario, no a nodos creados antes o programáticamente). Las actividades existentes (A1, A2, A3) tienen área asignada desde el 2026-07-22; al crear actividades nuevas, asignarla siempre.

### Decisión B3: `activity_image_position` NO lleva `field_seconds`

Decidido el 2026-07-22 (auditoría B3, plan § FASE 10):

- La métrica del ejercicio de imágenes es la **precisión** (`field_correct_count` / `field_items_count`), no el tiempo: el tiempo de observación depende de la complejidad de cada imagen y no es comparable entre actividades.
- Una "zona objetivo" de tiempo incentivaría velocidad sobre acierto, contra el criterio de producto (§ 2.3 del plan: rendimiento suavizado, sin juicios).
- En la pantalla de progreso, las actividades de imágenes aparecen en la sección de **aciertos** (barras de % en positivo) y no en la de evolución temporal con zona objetivo.

Si en el futuro se necesitara referencia temporal para imágenes, se añadiría reutilizando el storage `node.field_seconds` existente.

### Índices (rendimiento de queries de estadísticas)

- `activitysummary` (tabla base; la entidad no es revisionable ni traducible, no existe `activitysummary_field_data`): índice compuesto `activitysummary__uid_created (uid, created)`, añadido en `activitysummary_update_10001()`.
- `activitysummary__field_activityid`: el índice `field_activityid_target_id` ya lo crea core para todo entity_reference; junto con el índice de uid de la tabla base cubre el patrón `filter[uid.id]` + `filter[field_activityid.id]`.

---

## Métricas propuestas para estadísticas futuras

Las siguientes métricas no existen aún pero son necesarias para el sistema de estadísticas de la Fase 6:

### Añadir a `node.activity` (estándar de referencia)

> **Corrección (auditoría B3)**: `field_standard_time` nunca se creó. El campo real de referencia temporal es **`field_seconds`** (decimal), presente en `activity` y `activity_blink`, y se presenta como "zona objetivo" (nunca como línea de aprobado). `activity_image_position` no lo lleva por decisión explícita (ver sección anterior).

| Campo nuevo | Tipo | Descripción |
|---|---|---|
| ~~`field_standard_time`~~ | — | Sustituido por `field_seconds` (ya existe) |
| `field_standard_item_time` | integer | Tiempo medio esperado por elemento (no creado; valorar en v1.1) |

### Añadir a `activitysummary.summary_vertical_text` (métricas derivadas)

| Campo nuevo | Tipo | Descripción |
|---|---|---|
| `field_score` | decimal | Puntuación normalizada (calculada en backend o frontend) |
| `field_errors` | integer | Número de correcciones realizadas (si se implementa botón "volver") |

> Estos campos se añadirán cuando se implemente la Fase 6 (estadísticas). No anticipar.

---

## Cómo añadir una nueva métrica a un tipo existente

1. **Drupal**: añadir campo al bundle de `activitylog` o `activitysummary` correspondiente via UI o config YAML
2. **Drupal**: exportar config: `drush cex`
3. **Controller** (si se mantiene endpoint legacy): leer el nuevo parámetro del POST y asignarlo al campo
4. **Frontend Expo**: capturar el dato durante la actividad y enviarlo en el payload JSON:API
5. **Estadísticas**: añadir el nuevo campo a las queries de `docs/statistics-design.md`

---

## Cómo añadir métricas para un nuevo tipo de actividad

Si el nuevo tipo tiene datos diferentes a capturar (no solo items/weights/times), crear un **nuevo bundle en activitylog**:

```
activitylog
  ├── logverticaltext   (actual — texto vertical)
  ├── logimageposition  (futuro — posicionamiento de imágenes)
  └── logaudioreading   (futuro — lectura en voz alta)
```

Cada bundle tiene sus propios campos específicos, pero **todos comparten**:
- `field_activityid` → node.activity
- `field_sessionid` → node.session
- `field_activity_type` (string)
- `field_iteration` (integer)
- `uid` (autor)

Igualmente, crear el bundle equivalente en `activitysummary`:
```
activitysummary
  ├── summary_vertical_text    (actual)
  ├── summary_image_position   (futuro)
  └── summary_audio_reading    (futuro)
```

---

## Consideraciones de rendimiento

- Un usuario activo puede generar decenas de `activitylog` por sesión
- Las queries de estadísticas filtran siempre por `uid` + opcionalmente `activityid` o `sessionid`
- Índices recomendados en la tabla `activitylog_field_data`: `(uid, field_activityid_target_id)` y `(uid, field_sessionid_target_id, field_iteration)`
- En JSON:API, usar `page[limit]` para limitar resultados en listas históricas
