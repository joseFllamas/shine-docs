# Estructura de Entidades — shine.test (Drupal 10)

> Generado a partir del config export en `config/sync/` y los módulos custom en `public_html/modules/custom/`.

---

## 1. Tipos de Contenido (Node)

### `node.activity` — Activity text list
Representa una actividad individual con su contenido y configuración.

| Campo | Tipo | Requerido | Relación |
|---|---|---|---|
| `title` | string | SI | — |
| `body` | text_long | NO | — |
| `field_activity_code` | string | NO | — |
| `field_activity_format` | list_string | **SI** | — |
| `field_seconds` | decimal | NO | — |
| `field_text` | text_long | NO | — |
| `field_custom_explanation` | text_long | NO | — |
| `field_activity_level` | entity_reference | **SI** | → `taxonomy_term.activity_level` |
| `field_explanation` | entity_reference | NO | → `activity_message.explanation_message` (cardinalidad: 1) |

---

### `node.session` — Session
Agrupa varias actividades en una sesión de entrenamiento.

| Campo | Tipo | Requerido | Relación |
|---|---|---|---|
| `title` | string | SI | — |
| `body` | text_long | NO | — |
| `field_activities` | entity_reference | **SI** | → `node.activity` (cardinalidad: ilimitada) |

---

### `node.group_session` — Group Session
Agrupa varias sesiones en un bloque de grupo.

| Campo | Tipo | Requerido | Relación |
|---|---|---|---|
| `title` | string | SI | — |
| `body` | text_long | NO | — |
| `field_ref_session` | entity_reference | **SI** | → `node.session` (cardinalidad: ilimitada) |

---

### `node.activity_image_position` — Activity image position
Variante de actividad con imágenes posicionables en lugar de texto.

| Campo | Tipo | Requerido | Relación |
|---|---|---|---|
| `title` | string | SI | — |
| `body` | text_long | NO | — |
| `field_activity_code` | string | NO | — |
| `field_custom_explanation` | text_long | NO | — |
| `field_activity_level` | entity_reference | **SI** | → `taxonomy_term.activity_level` |
| `field_explanation` | entity_reference | NO | → `activity_message.explanation_message` (cardinalidad: 1) |
| `field_items` | entity_reference_revisions | NO | → `paragraph.i1` (cardinalidad: ilimitada) |

---

### `node.page` — Basic page
Contenido genérico estándar de Drupal. Sin relaciones custom.

---

## 2. Entidades Custom

### `activity_message` — Activity Message
Entidad custom con dos bundles para mensajes de apoyo al usuario.

#### Bundle: `explanation_message`
| Campo | Tipo | Requerido | Relación |
|---|---|---|---|
| `label` | string | SI | — |
| `description` | text_long | NO | — |
| `field_media` | entity_reference | NO | → `media.image` |
| `uid` | entity_reference | — | → `user` (autor) |

#### Bundle: `motiviation_message`
| Campo | Tipo | Requerido | Relación |
|---|---|---|---|
| `label` | string | SI | — |
| `description` | text_long | NO | — |
| `field_media` | entity_reference | NO | → `media.image` |
| `field_activity_types` | entity_reference | NO | → `node_type` (cualquier tipo de contenido, ilimitado) |
| `uid` | entity_reference | — | → `user` (autor) |

> Los `motiviation_message` se inyectan dinámicamente en la plantilla de nodos `activity` a través del hook `preprocess_preprocess_node()` del módulo `preprocess`, cargando aleatoriamente uno que coincida con el tipo de actividad.

---

### `activitylog` — Activity Log
Registra cada interacción del usuario con una actividad.

#### Bundle: `logverticaltext`
| Campo | Tipo | Requerido | Relación |
|---|---|---|---|
| `label` | string | SI | — |
| `field_activityid` | entity_reference | NO | → `node.activity` (cardinalidad: 1) |
| `field_sessionid` | entity_reference | NO | → `node.session` (cardinalidad: 1) |
| `field_activity_type` | string | NO | — |
| `field_item` | string | NO | — |
| `field_iteration` | integer | NO | — |
| `field_time` | integer | NO | — |
| `field_weight` | integer | NO | — |
| `uid` | entity_reference | — | → `user` (autor) |

> Creado vía JavaScript (`activitylog_register/js/`) cuando el usuario interactúa con una actividad o sesión.

---

### `activitysummary` — Activity Summary
Agrega los registros de `activitylog` de una actividad/sesión completa.

#### Bundle: `summary_vertical_text`
| Campo | Tipo | Requerido | Relación |
|---|---|---|---|
| `label` | string | SI | — |
| `field_activitylogs` | entity_reference | NO | → `activitylog.logverticaltext` (cardinalidad: ilimitada) |
| `field_activityid` | entity_reference | NO | → `node.activity` (cardinalidad: 1) |
| `field_sessionid` | entity_reference | NO | → `node.session` (cardinalidad: 1) |
| `field_activity_type` | string | NO | — |
| `field_iteration` | integer | NO | — |
| `field_time` | integer | NO | — |
| `uid` | entity_reference | — | → `user` (autor) |

---

## 3. Párrafos (Paragraphs)

### `paragraph.i1` — Image group
Contenedor de un grupo de imágenes posicionables.

| Campo | Tipo | Requerido | Relación |
|---|---|---|---|
| `field_position_to_ask` | integer | NO | — |
| `field_items` | entity_reference_revisions | NO | → `paragraph.i1_item` (cardinalidad: ilimitada) |

### `paragraph.i1_item` — Image group item
Elemento individual de un grupo de imágenes.

| Campo | Tipo | Requerido | Relación |
|---|---|---|---|
| `field_image` | entity_reference | **SI** | → `media.image` |
| `field_text` | text_long | NO | — |

---

## 4. Taxonomías

| Vocabulario | Machine name | Usado por |
|---|---|---|
| Activity Level | `activity_level` | `node.activity.field_activity_level`, `node.activity_image_position.field_activity_level` |
| Explanations | `explanations` | Sin referencias de campo detectadas |
| Tags | `tags` | Estándar Drupal (sin relaciones custom) |

---

## 5. Media Types

| Tipo | Machine name | Usado por |
|---|---|---|
| Image | `image` | `activity_message.*.field_media`, `paragraph.i1_item.field_image` |
| Audio | `audio` | Sin referencias custom detectadas |
| Document | `document` | Sin referencias custom detectadas |
| Video | `video` | Sin referencias custom detectadas |
| Remote Video | `remote_video` | Sin referencias custom detectadas |

---

## 6. Diagrama de Relaciones

```
group_session
    └─[field_ref_session, M]──────────────► session
                                                └─[field_activities, M]──────► activity
                                                                                    ├─[field_activity_level, 1, REQ]──► taxonomy: activity_level
                                                                                    └─[field_explanation, 1]───────────► activity_message
                                                                                                                              └─[explanation_message]
                                                                                                                                   └─[field_media]──► media.image

activity_image_position
    ├─[field_activity_level, 1, REQ]────────► taxonomy: activity_level
    ├─[field_explanation, 1]────────────────► activity_message [explanation_message]
    └─[field_items, M]──────────────────────► paragraph.i1
                                                  └─[field_items, M]──────────► paragraph.i1_item
                                                                                    └─[field_image, REQ]──► media.image

activity_message [motiviation_message]
    ├─[field_media]─────────────────────────► media.image
    └─[field_activity_types, M]─────────────► node_type (cualquier tipo)

activitylog [logverticaltext]
    ├─[field_activityid, 1]─────────────────► node.activity
    └─[field_sessionid, 1]──────────────────► node.session

activitysummary [summary_vertical_text]
    ├─[field_activitylogs, M]───────────────► activitylog [logverticaltext]
    ├─[field_activityid, 1]─────────────────► node.activity
    └─[field_sessionid, 1]──────────────────► node.session

Todas las entidades custom ──[uid]──────────► user
```

---

## 7. Módulos Custom y sus Roles

| Módulo | Rol | Entidades involucradas |
|---|---|---|
| `activity_message` | Define la entidad `activity_message` y sus bundles; expone templates twig | `activity_message` |
| `activitylog` | Define la entidad `activitylog` | `activitylog` |
| `activitylog_register` | Registra interacciones via JS; hace preprocess en nodos para adjuntar sessionId/activityId | `node.session`, `node.activity`, `node.activity_image_position`, `activitylog` |
| `activitysummary` | Define la entidad `activitysummary` | `activitysummary`, `activitylog` |
| `preprocess` | Inyecta aleatoriamente un `motiviation_message` en la plantilla de nodos `activity` | `activity_message`, `node.activity` |

---

## 8. Flujo de datos principal

```
1. CONTENIDO:
   group_session → session → activity / activity_image_position

2. MENSAJES DE APOYO:
   activity_message[explanation_message] ← activity (campo fijo)
   activity_message[motiviation_message] → inyectado dinámicamente en template

3. TRACKING:
   usuario interactúa con activity dentro de session
       ↓ JS (activitylog_register)
   activitylog[logverticaltext] (registra cada item/peso/tiempo)
       ↓ agregación
   activitysummary[summary_vertical_text] (totales de la sesión)
```
