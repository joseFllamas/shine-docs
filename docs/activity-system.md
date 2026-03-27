# Sistema de Actividades — Diseño y extensibilidad

## Concepto

Una **actividad** es la unidad básica de entrenamiento. Tiene partes comunes (siempre presentes) y partes específicas del tipo (varían según el tipo de actividad). Esta separación es lo que permite añadir nuevos tipos sin modificar la lógica existente.

---

## Anatomía de una actividad (partes comunes)

Toda actividad, independientemente del tipo, tiene estos elementos:

```
┌─────────────────────────────────────────┐
│  1. INTRO / DESCRIPCIÓN                  │
│     body + field_text (el enunciado)     │
│     field_custom_explanation (texto libre│
│     de apoyo adicional)                  │
├─────────────────────────────────────────┤
│  2. MENSAJE DE EXPLICACIÓN               │
│     field_explanation →                  │
│     activity_message.explanation_message │
│     (puede incluir imagen media)         │
├─────────────────────────────────────────┤
│  3. NIVEL DE DIFICULTAD                  │
│     field_activity_level →               │
│     taxonomy.activity_level              │
├─────────────────────────────────────────┤
│  4. CONTENIDO ESPECÍFICO DEL TIPO        │
│     (varía por bundle / node type)       │
├─────────────────────────────────────────┤
│  5. MENSAJE DE MOTIVACIÓN                │
│     activity_message.motivation_message  │
│     (inyectado aleatoriamente por tipo)  │
├─────────────────────────────────────────┤
│  6. TRACKING / LOG                       │
│     activitylog + activitysummary        │
│     (siempre presente en todo tipo)      │
└─────────────────────────────────────────┘
```

Los elementos 1, 2, 3, 5 y 6 son siempre los mismos. Solo el elemento 4 varía.

---

## Tipos de actividad actuales

### `activity` — Lista de texto vertical
**Node type**: `node.activity`
**Activitylog bundle**: `logverticaltext`

**Mecánica**: Se muestra una lista de palabras o frases en vertical. El usuario hace clic/tap para ir avanzando a la siguiente. Se registra el tiempo entre clics y el peso (posición) de cada elemento.

**Campos específicos**:
- `field_text` (text_long): el contenido de la lista (palabras/frases separadas)
- `field_activity_code` (string): código identificador de la actividad
- `field_activity_format` (list_string): formato de presentación (REQUERIDO)
- `field_seconds` (decimal): duración esperada en segundos

**Métricas capturadas** (en activitylog):
- `field_item`: texto del elemento
- `field_weight`: posición del elemento en la lista
- `field_time`: tiempo en ms desde el inicio o desde el elemento anterior
- `field_iteration`: número de intento

**Implementación JS actual**: clase `ActivityTextList` en `activitylog_register/js/logregister.js`

---

### `activity_image_position` — Posicionamiento de imágenes
**Node type**: `node.activity_image_position`
**Activitylog bundle**: *(sin bundle específico aún, comparte logverticaltext)*

**Mecánica**: Se muestran imágenes que el usuario debe posicionar o clasificar.

**Campos específicos**:
- `field_items` → `paragraph.i1` (grupos de imágenes)
  - `paragraph.i1.field_items` → `paragraph.i1_item` (imagen individual)
    - `field_image` → `media.image` (REQUERIDO)
    - `field_text` (text_long): etiqueta de la imagen
  - `field_position_to_ask` (integer): posición correcta esperada

**Estado**: Implementación JS incompleta (`prepareActivity()` y `playActivityItem()` son placeholders en `ActivityImagePosition`).

---

## Sesiones y grupos

Las actividades se organizan en sesiones:

```
group_session
  └── field_ref_session (M) → session
        └── field_activities (M) → activity / activity_image_position
```

El JavaScript de `activitylog_register` detecta si hay `session_id` para saber si mostrar la siguiente actividad o recargar la página al finalizar.

---

## Cómo añadir un nuevo tipo de actividad

### Checklist completo

**En Drupal (backend)**:
- [ ] Crear `node.type.activity_{tipo}.yml` en `config/install` del módulo correspondiente (o en config sync si se hace via UI)
- [ ] Reutilizar los campos comunes existentes: `field_activity_level`, `field_explanation`, `field_custom_explanation`, `field_activity_code`
- [ ] Añadir campos específicos del tipo (con prefijo `field_`, siguiendo convenciones)
- [ ] Crear bundle nuevo en `activitylog` si las métricas a capturar son diferentes a `logverticaltext` (ver `docs/metrics-tracking.md`)
- [ ] Si hay bundle nuevo en activitylog, crear bundle equivalente en `activitysummary`
- [ ] Añadir `activity_{tipo}` en la lógica de `activitylog_register.module` (hook_preprocess_node)
- [ ] Exportar config: `drush cex`

**En el frontend Expo**:
- [ ] Crear `src/components/activities/{Tipo}Player.tsx` extendiendo `ActivityBase`
- [ ] Implementar los métodos del ciclo de vida: `prepare()`, `play()`, `end()`
- [ ] Añadir el nuevo tipo al switch/map de `ActivityRouter` (componente que elige el player correcto según `field_activity_format`)
- [ ] Añadir tipo TypeScript en `src/types/activity.ts`
- [ ] Añadir tests básicos

---

## Jerarquía de clases JS (código acoplado actual)

```
Activity (clase base — activitylog_register/js/logregister.js)
│
│  Métodos base:
│  - constructor(element)
│  - initActivity()         → escucha click en startButton
│  - prepareActivity()      → configura timer, oculta intro
│  - startActivity()        → llama prepareActivity()
│  - playActivity()         → llama playActivityItem(), comprueba ended
│  - endActivity()          → prepara datos, llama sendLog()
│  - showMotivationMessage()→ muestra mensaje de motivación + confetti
│  - showConfetti()         → animación canvas-confetti (3 segundos)
│  - nextActivityProcess()  → siguiente actividad o recarga
│  - sendLog()              → POST a /activitylog-register/add via fetch
│  - prepareLogObject()     → serializa arrays a JSON
│
├── ActivityTextList (implementado)
│   Propiedades: counter, posicioItem, elements, activeElement, translateY
│   Sobreescribe: prepareActivity(), startActivity(), playActivityItem()
│   Añade: addRegisterLog() — registra tiempo, peso e ítem en arrays
│
└── ActivityImagePosition (parcialmente implementado)
    Propiedades: subActivities (array de SubActivity)
    Sobreescribe: constructor (crea SubActivity por cada .image-item)
    Pendiente: prepareActivity(), playActivityItem()

SubActivity (helper interno para image position)
  - getCurrentStep(), gotoNextStep(), gotoPrevStep()
  - Gestiona navegación entre pasos dentro de una imagen
```

---

## Jerarquía de componentes React Native (Expo, objetivo)

```
ActivityBase (src/components/activities/ActivityBase.tsx)
│
│  Props: activity, sessionId, onComplete
│  Estado: phase (intro | playing | completed)
│  Ciclo:
│  - render intro (explanation + nivel)
│  - usuario pulsa "Comenzar" → startTimer()
│  - lógica específica del tipo (implementada en subclase/composición)
│  - onComplete(logData) → padre envía a API
│
├── VerticalTextPlayer.tsx    (equivalente a ActivityTextList)
├── ImagePositionPlayer.tsx   (completar implementación)
└── AudioReadingPlayer.tsx    (nuevo, Fase 7)
```

El componente padre `ActivityRouter` selecciona qué player usar según `field_activity_format`:

```typescript
// src/components/activities/ActivityRouter.tsx
const players = {
  'vertical_text': VerticalTextPlayer,
  'image_position': ImagePositionPlayer,
  'audio_reading': AudioReadingPlayer,
};
```

---

## Mensajes de motivación — lógica de selección

Los `activity_message.motivation_message` tienen el campo `field_activity_types` que referencia tipos de contenido (node_type). Esto permite filtrar qué mensajes aplican a qué tipo de actividad.

**Lógica actual (acoplada, en `preprocess.module`)**:
1. Al hacer preprocess del nodo `activity`, carga todos los `motivation_message` cuyo `field_activity_types` incluye el tipo actual
2. Selecciona uno aleatoriamente
3. Lo inyecta en la variable `$vars['message']`

**Lógica objetivo (desacoplada, en Expo)**:
1. Al cargar una actividad, hacer `GET /jsonapi/activity_message/motivation_message?filter[field_activity_types.type]={activityFormat}`
2. Seleccionar uno aleatoriamente en el cliente
3. Mostrar al finalizar la actividad

> **Nota**: el bundle se llama `motiviation_message` (con typo) en la base de datos actual. Esto se corrige en Fase 0 del roadmap.
