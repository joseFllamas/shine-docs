# Auditoría del proyecto Shine

Fecha: 2026-07-21. Alcance: backend Drupal, app Expo, documentación y diseño. Objetivo: fotografía fiel de lo que hay para planificar la primera versión publicable (ver plan en `02-plan-ejecucion-v1.md`).

---

## 1. Resumen ejecutivo

El proyecto está en mejor estado del que sugiere su fase "en desarrollo": el flujo núcleo completo (login OAuth2 con PKCE, dashboard, sesión, ejecución de los 3 tipos de actividad, guardado de logs y resúmenes, estadísticas básicas) **funciona de extremo a extremo**. El backend Drupal 11 está sano, con modelo de datos bien pensado y extensible.

Lo que separa el estado actual de una primera versión publicable no es funcionalidad nueva sino cuatro carencias transversales:

1. **Estadísticas a medias y sin criterio pedagógico**: la pantalla actual muestra datos crudos (tiempo por intento) que pueden leerse como retroceso. Falta la capa de interpretación (suavizado, mejores marcas, esfuerzo) y la agregación por área, que es justo lo que pide el producto.
2. **Sin design system**: colores y estilos duplicados a mano en unos 10 ficheros, con una paleta de facto (azules `#1a5276`) distinta de la especificada en `docs/design.md` (teal `#3DBFA0`). No hay tema centralizado.
3. **Configuración de desarrollo incrustada**: dominio `shine.ddev.site`, `client_id` y `client_secret` OAuth hardcodeados en el bundle de la app. Inviable para producción y stores.
4. **Deuda de datos para estadísticas**: los logs y resúmenes no guardan área ni nivel (hay que navegar al nodo actividad), el filtro por usuario en las queries de estadísticas está sin aplicar en el cliente, y el endpoint legacy escribe siempre en los bundles de texto vertical aunque la actividad sea de otro tipo.

---

## 2. Backend Drupal

### 2.1 Lo que hay (verificado en config/sync y módulos custom)

**Stack**: Drupal 11.3.5, PHP 8.3, MariaDB 10.11, simple_oauth 6.1.0 (Authorization Code + PKCE), JSON:API core, CORS abierto (`*`), DDEV.

**Taxonomías**:
| Vocabulario | Uso |
|---|---|
| `areas` | `field_main_area` (requerido) y `field_secondary_area` (opcional) en los 3 tipos de actividad |
| `activity_level` | `field_activity_level` (requerido, múltiple) en los 3 tipos de actividad |
| `explanations`, `tags` | Sin uso funcional |

Las taxonomías cuelgan SOLO de los nodos de actividad. Ni `session`, ni `group_session`, ni `activitylog`, ni `activitysummary` las referencian.

**Content types**:
| Tipo | Contenido | Métrica de referencia |
|---|---|---|
| `activity` (Activity2-text list) | `field_text` múltiple, `field_activity_format` (vertical_list, blink) | `field_seconds` (decimal 10,2) |
| `activity_blink` | Como activity, sin field_activity_format | `field_seconds` |
| `activity_image_position` | `field_items` a paragraphs con imágenes y posición | Sin field_seconds |
| `session` | `field_activities` (múltiple, los 3 tipos) | Sin taxonomías ni métricas |
| `group_session` | `field_ref_session` (múltiple) | Sin uso en la app |

**Entidades custom de tracking** (la base de las estadísticas):
- `activitylog` (bundles `logverticaltext`, `log3_textlistblink`, `logimageposition`): un registro por ítem completado. Campos comunes: `field_activityid`, `field_sessionid`, `field_activity_type`, `field_item`, `field_iteration`, `field_weight`, `field_time` (integer, ms). El bundle de imágenes añade `field_answer` (integer) y `field_correct` (boolean).
- `activitysummary` (bundles `summary_vertical_text`, `summary3_vertical_text_blink`, `summary_image_position`): un registro por intento completo, con `field_activitylogs` (referencia múltiple a los logs), `field_time` (suma) y `field_iteration`. Los 3 bundles tienen los mismos campos.
- `activity_message` (bundles `explanation_message`, `motivation_message` con `field_activity_types` y `field_media`).

Permisos y hooks de acceso (`view own`, `create`) implementados y funcionando via JSON:API para el rol authenticated.

### 2.2 Problemas detectados

| # | Problema | Impacto | Severidad |
|---|---|---|---|
| B1 | El endpoint legacy `/activitylog-register/add` crea SIEMPRE bundles `logverticaltext` y `summary_vertical_text`, ignorando el `activityType` recibido | Si el Drupal acoplado sigue usándose, contamina las estadísticas por tipo | Media (baja si se retira ya) |
| B2 | Los logs y summaries no guardan área ni nivel; para agregar por área hay que resolver cada `field_activityid` contra su nodo | Cada pantalla de estadísticas por área multiplica queries o exige lógica de join en el cliente | Alta para el objetivo de producto |
| B3 | La documentación habla de `field_standard_time` (no existe); el campo real es `field_seconds` (decimal) en activity y activity_blink. `activity_image_position` no tiene ninguno | El componente StandardComparison no tiene fuente de datos conectada | Media |
| B4 | `client_secret` en un cliente público (app móvil). Con PKCE el secreto es innecesario y en el bundle JS es público de facto | Seguridad; además Apple/Google pueden rechazar secretos embebidos | Alta antes de publicar |
| B5 | Sin entorno de producción definido (dominio, SSL, claves RSA de producción, `allowedOrigins: '*'`) | Bloqueo de la Fase 8 | Alta antes de publicar |
| B6 | El cálculo de `field_iteration` se fía del cliente (cuenta summaries previos); con dos dispositivos puede duplicar iteraciones | Estadísticas con iteraciones repetidas | Baja |

### 2.3 Fortalezas a conservar

- El patrón bundle por tipo de ejercicio (log + summary paralelos) escala bien a los 10 ejercicios nuevos del docx.
- `field_correct`/`field_answer` en el bundle de imágenes ya modelan "precisión", generalizable a los ejercicios con respuesta correcta.
- Las vistas de gestión (`views.view.gestion_activitylogs`, `gestion_activitysummary`) sirven de backoffice para terapeutas sin desarrollo extra.

---

## 3. App Expo (shine-app)

### 3.1 Estado por pantalla

| Pantalla | Estado | Detalle |
|---|---|---|
| Login | Funcional | OAuth2 + PKCE completo. Falla en silencio si el token falla (sin feedback). Credenciales y dominio hardcodeados |
| Dashboard | Funcional | Saludo, lista de sesiones, estados de carga/vacío/error cubiertos. Sin progreso por sesión, sin resumen semanal, sin "continuar" |
| Sesión [id] | Funcional con salvedades | Carga y resuelve los 3 tipos de actividad. El estado "completada" es local y volátil (se pierde al navegar). Sin catch de errores |
| Player (3 tipos) | Funcional | ActivityBase (intro, playing, completed) + VerticalTextPlayer, AudioReadingPlayer (graba audio pero lo descarta), ImageGroupPlayer (con aciertos). ActivityRouter devuelve null en formatos desconocidos (pantalla en blanco) |
| Estadísticas | A medias | Stat cards + gráfica por actividad + imágenes + historial. Títulos placeholder ("Actividad 1"), nombres de sesión sin resolver, sección imágenes sin guard, y las queries ignoran el uuid de usuario recibido |

### 3.2 Problemas detectados

| # | Problema | Severidad |
|---|---|---|
| A1 | `getMySummaries`/`getMyImageSummaries` reciben el uuid del usuario y no lo aplican como filtro (dependen solo del access hook del servidor) | Alta |
| A2 | Sin design system: estilos inline duplicados, paleta de facto azul distinta de la especificada en docs/design.md | Alta (bloquea el trabajo de diseño) |
| A3 | `API_BASE`, `clientId`, `clientSecret` hardcodeados en 3 ficheros; sin variables de entorno | Alta antes de publicar |
| A4 | `iteration` global en el store: no se calcula por actividad, no se resetea entre sesiones | Media (corrompe la gráfica de evolución) |
| A5 | Estado de completado de sesión volátil (no se deriva de los summaries del servidor) | Media |
| A6 | Estadísticas: nombres reales de actividad y sesión sin resolver; `getMotivationMessages` no filtra por tipo | Media |
| A7 | Código huérfano: SessionProgressBar, StandardComparison, getGroupSessions, getExplanationMessage, la mayoría del store session.ts | Baja (ruido) |
| A8 | expo-av deprecado en SDK 54 (migrar a expo-audio); audio grabado que se descarta | Baja en v1 |
| A9 | Sin ESLint, sin tests, sin prettier | Media |

### 3.3 Fortalezas a conservar

- Arquitectura de players correcta y ya validada (ActivityBase + player por tipo + ActivityRouter): añadir un ejercicio nuevo es crear un componente y dos funciones de tracking.
- Cliente API con refresh automático de token, wrapper de storage multiplataforma, tipos TypeScript espejo de Drupal.
- victory-native y react-native-svg ya instalados y usados (el pendiente que figuraba en progress.md está resuelto).

---

## 4. Documentación y diseño

- `docs/` es completa y de calidad (arquitectura, convenciones, sistema de actividades, tracking, estadísticas). Desfases a corregir: `field_standard_time` inexistente, "pendiente instalar victory-native/expo-av" ya resuelto, y el diseño de estadísticas de `statistics-design.md` es anterior al criterio de "reportes no desmoralizantes" (muestra datos crudos por intento como titular).
- `docs/design.md` define un design system sólido (teal, Nunito, accesibilidad para dislexia) y `docs/stitch/` tiene 5 mockups generados. Nada de esto está aplicado en la app.
- "ejercicios - tipo.docx": 10 ejercicios tipo bien especificados (descripción, objetivo, interfaz, métricas), con los prompts para Claude Code aún vacíos. Encajan en el patrón bundle log/summary existente.

---

## 5. Conclusión y prioridades

Orden de ataque recomendado (desarrollado en `02-plan-ejecucion-v1.md`):

1. Saneamiento técnico (config por entorno, PKCE sin secret, filtro por usuario, iteración correcta, completado persistente).
2. Capa de datos para estadísticas (denormalizar área/nivel/aciertos en los summaries, unificar el concepto de tiempo de referencia).
3. Design system en código y aplicación a las 5 pantallas existentes (en paralelo, generación visual con los prompts de `plans/design/`).
4. Rediseño de la pantalla de progreso con el criterio de evolución no lineal.
5. Primeros 2 o 3 ejercicios nuevos del docx (los que no requieren locuciones de audio).
6. Preparación de publicación (EAS, stores, backend de producción).
