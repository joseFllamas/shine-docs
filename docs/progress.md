# Progreso de implementación — Shine

> Actualizado en tiempo real. Si el desarrollo se interrumpe, retomar desde el primer ítem sin marcar de la fase en curso.

**Última actualización**: 2026-07-22 (sesión 9 — FASE 12 en curso: pantalla Dashboard, prompt 12a)

---

## FASE 0 — Limpieza y prerequisitos
**Estado**: ✅ Completada

- [x] Eliminar `/structure.md` de la raíz (ya no existía)
- [x] Corregir typo `motiviation_message` → `motivation_message` (DB + config YAMLs + preprocess.module)
- [x] Actualizar `.info.yml` de módulos custom (descripción real + `package: shine` + `^10 || ^11`)
- [x] Verificar que `docs/` contiene todos los documentos de referencia
- [x] Eliminado config huérfano `odiseanb5.settings.yml`

---

## FASE 1 — Upgrade Drupal 10 → 11
**Estado**: ✅ Completada

- [x] Verificar módulos incompatibles con D11 (eliminados ~80 módulos no usados)
- [x] Actualizar `composer.json` a `^11` (+ quitar parches D8/D9/D10, actualizar constraints)
- [x] Ejecutar `composer update` (Drupal 11.3.5 instalado)
- [x] Ejecutar `drush updb` (40+ updates aplicados, sin pendientes)
- [x] Verificar estado del sitio post-upgrade (Drupal 11.3.5 activo, bootstrap exitoso)

---

## FASE 2 — Habilitar JSON:API + Auth
**Estado**: ✅ Completada

- [x] Habilitar módulo `jsonapi` (ya estaba habilitado en D11)
- [x] Instalar `drupal/simple_oauth` v6.1.0 (D11 compatible) + `drupal/consumers`
- [x] Configurar CORS en `services.yml` (allowedOrigins: *, métodos GET/POST/PATCH/DELETE/OPTIONS)
- [x] Crear Consumer OAuth2 "Shine Expo App" (client_id: shine_expo_app, UUID: 5393c7d5-0f7b-4482-a770-0e5ce00639c2)
- [x] Configurar permisos JSON:API (create/view activitylog, activitysummary, activity_message para authenticated)
- [x] Exportar config (`drush cex`)

---

## FASE 3 — Refactorizar endpoint de registro
**Estado**: ✅ Completada

- [x] Verificar viabilidad de escritura directa a JSON:API (viable — activitylog/activitysummary expuestos)
- [x] Actualizar permisos para escritura de activitylog/activitysummary (create/view own para `authenticated`)
- [x] Documentar nuevo flujo en api-design.md (OAuth2 Authorization Code + PKCE, simple_oauth v6)
- [x] Mantener endpoint legacy hasta migración frontend (sigue activo en `/activitylog-register/add`)

---

## FASE 4 — Proyecto Expo (estructura base)
**Estado**: ✅ Completada

- [x] Crear proyecto Expo con TypeScript (Expo SDK 55, React Native 0.83, Node >=20)
- [x] Instalar dependencias: expo-router, expo-auth-session, expo-web-browser, expo-secure-store, expo-crypto, zustand
- [x] Configurar `app.json` (scheme: "shine") y `tsconfig.json` (path alias @/*)
- [x] Crear estructura de directorios: src/types, src/lib/api, src/store, src/hooks, src/components
- [x] Tipos TypeScript: jsonapi.ts, activity.ts, session.ts, activitylog.ts, activitysummary.ts
- [x] Cliente API (`src/lib/api/client.ts`): fetch autorizado + refresh automático de token
- [x] Módulos API: activities.ts, sessions.ts, tracking.ts
- [x] Stores Zustand: auth.ts (isAuthenticated, login, logout, initialize), session.ts
- [x] Expo Router: _layout.tsx (root con guard de auth), (auth)/_layout.tsx, login.tsx, dashboard.tsx
- [x] Flujo OAuth2 Authorization Code + PKCE implementado en login.tsx (expo-auth-session)

---

## FASE 5 — Activity Player
**Estado**: ✅ Completada

- [x] `ActivityBase` — intro + playing + completed con animación fade
- [x] `VerticalTextPlayer` — maneja formatos `blink` (auto-avance) y `vertical_list` (tap)
- [x] `ActivityRouter` — selecciona el player según `field_activity_format`
- [x] Tracking: crea logs individuales (activitylog) + resumen (activitysummary) via JSON:API
- [x] Mensaje de motivación aleatorio al finalizar
- [x] Pantalla de sesión `app/(auth)/session/[id].tsx` con lista de actividades y estado completado
- [x] Dashboard enlaza a sesión (tap en tarjeta → `/session/[id]`)
- [ ] `ImagePositionPlayer` (pospuesto, sin bundle activitylog específico aún)

---

## FASE 6 — Estadísticas y gráficas
**Estado**: ✅ Completada (pendiente instalar victory-native)

- [x] `ActivityEvolutionChart` (gráfica de líneas — victory-native)
- [x] `StandardComparison` (gráfica de barras vs tiempo estándar)
- [x] `SessionProgressBar` (barra de progreso accesible)
- [x] `SessionComparisonTable` (tabla de sesiones con agrupación en cliente)
- [x] `StatsDataTable` (tabla accesible bajo cada gráfica — WCAG)
- [x] `statistics/index.tsx` — pantalla con resumen + historial por sesión
- [x] Enlace "Estadísticas" en dashboard
- [x] `src/lib/api/statistics.ts` — queries JSON:API + helpers
- [ ] Instalar: `npx expo install victory-native react-native-svg`
- [ ] Añadir `field_standard_time` a `node.activity` en Drupal (para comparativa)

---

## FASE 7 — Soporte de audio y micrófono
**Estado**: ✅ Completada (pendiente instalar expo-av)

- [x] Permisos de micrófono en `app.json` (iOS infoPlist + plugin expo-av)
- [x] `AudioReadingPlayer` — muestra texto, mide tiempo, graba audio (fallback si falla)
- [x] Formato `audio_reading` añadido a `ActivityFormat` y `ActivityRouter`
- [x] Funciona en web (solo tiempo) y móvil (graba + tiempo)
- [ ] Instalar: `npx expo install expo-av`
- [ ] Crear node type `activity_audio_reading` en Drupal

---

## FASE 7b — Separación activity_blink
**Estado**: ✅ Completada

### Drupal
- [x] Nuevo node type `activity_blink` ("Activity3-text list blink") — sin `field_activity_format` (el formato es implícitamente blink)
- [x] Nuevo activitylog bundle `log3_textlistblink` ("Log3-textlistblink")
- [x] Nuevo activitysummary bundle `summary3_vertical_text_blink` ("Summary3-Vertical text list blink")
- [x] 29 YAMLs de config generados e importados con `drush cim --partial -y`
- [x] `field.field.node.session.field_activities.yml` actualizado para aceptar `activity_blink`
- [x] Endpoints JSON:API verificados: `/jsonapi/node/activity_blink`, `/jsonapi/activitylog/log3_textlistblink`, `/jsonapi/activitysummary/summary3_vertical_text_blink`

### App
- [x] `src/types/activity.ts` — añadido `BlinkActivityAttributes`, `BlinkActivityResource`; `AnyActivityResource` actualizado
- [x] `src/types/activitylog.ts` — añadido `BlinkLogResource`, `CreateBlinkLogPayload`
- [x] `src/types/activitysummary.ts` — añadido `BlinkSummaryResource`, `CreateBlinkSummaryPayload`
- [x] `src/lib/api/tracking.ts` — añadido `createBlinkLog`, `createBlinkSummary`
- [x] `src/lib/api/activities.ts` — añadido `getBlinkActivities`, `getBlinkActivity`
- [x] `ActivityBase.tsx` — `handleEnd` ramifica por `activity.type`: blink usa bundles `log3_textlistblink`/`summary3_vertical_text_blink`; vertical_list sigue usando los originales
- [x] `VerticalTextPlayer.tsx` — prop `activity` ampliado a `ActivityResource | BlinkActivityResource`
- [x] `ActivityRouter.tsx` — rama `node--activity_blink` añadida antes del cast a `ActivityResource`
- [x] `app/(auth)/session/[id].tsx` — carga actividades blink con `getBlinkActivity()` en lugar de `getActivity()`

---

## FASE 9 — Saneamiento técnico (ver plans/02-plan-ejecucion-v1.md)
**Estado**: ✅ Completada (2026-07-21) — **Backend** (09a) + **App**: config por entorno (09b), bugs de datos A1/A4/A5 (09c), manejo de errores + limpieza A7 + tooling A9 (09d). Único pendiente diferido: restringir CORS a orígenes reales (FASE 14).

### Backend (prompt 09a) — 2026-07-21

**B4 — Consumer OAuth como cliente público** ✅
- Consumer `shine_expo_app` (id=2) cambiado a `confidential: false` y `secret` vaciado.
- El backend **ya no exige `client_secret`**: flujo Authorization Code + PKCE verificado por curl end-to-end **con un usuario normal** (no super user): login → `/oauth/authorize` (con pantalla de consentimiento "Allow") → `/oauth/token` **sin enviar `client_secret`** → respuesta `200` con `access_token` + `refresh_token`.
- ⚠️ El consumer es una **entidad de contenido** (vive en BD, no en `config/sync`). Por eso `drush cex` NO lo captura y el cambio **debe reaplicarse en cada entorno** (script en `CLAUDE.md` → "Configuración en un PC nuevo"). Actualizar ese script para poner `confidential: false` al crear el consumer.
- Pendiente en la app (prompt 09b): dejar de enviar `client_secret` en el POST a `/oauth/token`.

**Permiso `grant simple_oauth codes`** ✅ (concedido 2026-07-21)
- Se detectó que el rol `authenticated` NO tenía `grant simple_oauth codes`, permiso que simple_oauth exige para completar `/oauth/authorize` (`Oauth2AuthorizeController::authorize`). Sin él, solo el user 1 (super user) podía completar el login.
- Concedido al rol `authenticated` y **exportado a config** (`config/sync/user.role.authenticated.yml` + dependencia `simple_oauth`). `config:status` sin diff → idempotente.
- Verificado: un usuario normal (rol authenticated) completa el flujo completo sin secret (token `200`).
- Nota UX: `automatic_authorization` del consumer sigue en `off`, así que el usuario ve una pantalla de consentimiento "Allow" en cada login. Si se quiere login sin fricción (recomendable para una app propia), activar `automatic_authorization` en el consumer (vive en BD, como el resto de sus ajustes).

**B1 — Endpoint legacy `/activitylog-register/add`: RETIRADO** ✅
- Decisión con evidencia: **retirar** (módulo `activitylog_register` desinstalado + código eliminado).
- Evidencia:
  - La app Expo escribe el tracking **solo por JSON:API** (`shine-app/src/lib/api/tracking.ts`: POST a `/activitylog/*` y `/activitysummary/*` por bundle). No hay ninguna referencia a `/activitylog-register/add` en la app.
  - El único consumidor del endpoint era el **frontend Drupal acoplado** (`activitylog_register_preprocess_node` adjuntaba `logregister.js` a las plantillas `node--activity`/`node--session`/`node--activity-image-position`, con `startButton`), sustituido por la app Expo en el pivote de 2026-03-27.
  - El endpoint tenía el bug B1 (creaba siempre bundles `logverticaltext`/`summary_vertical_text` ignorando `activityType`): retirarlo lo elimina de raíz.
- Acciones: `drush pm:uninstall activitylog_register`; borrado `public_html/modules/custom/activitylog_register/`; línea eliminada de `config/sync/core.extension.yml`; `drush cr`.
- Verificado: `GET /activitylog-register/add` → **404**; `GET /jsonapi/activitylog/logverticaltext` → **200** (write path de la app intacto). `activitylog` y `activitysummary` siguen habilitados.
- Nota: las plantillas del frontend acoplado (`node--activity.html.twig` con `startButton`) siguen en los themes `shine`/`stablechild`; sin el módulo su `startButton` queda inerte (no es parte de la v1). El módulo `preprocess` (inyecta mensaje de motivación en esas plantillas Drupal) también es solo del frontend acoplado; se deja intacto por estar fuera de alcance.

**B5 — CORS** ✅ (preparado)
- `services.yml` mantiene `allowedOrigins: ['*']` para desarrollo.
- Añadido comentario en el propio `services.yml` con el valor de producción (orígenes reales del build web; las apps nativas no envían `Origin`). **Activar en FASE 14.**

**Verificación general**
- `drush watchdog:show --severity=Error`: sin errores nuevos atribuibles a estos cambios (las entradas de la sesión son el 404 esperado del endpoint retirado y un "user denied" de una iteración del test PKCE previa a activar la autorización automática).
- Config: mi único cambio en `config/sync` (quitar `activitylog_register` de `core.extension.yml`) es idempotente — active y sync coinciden respecto a ese módulo. El resto de diffs de `drush config:status` (`search_help`/`search_node`, `backup_migrate`, `olivero`, `system.performance`) son **drift pre-existente ajeno** a esta tarea; NO se ejecutó un `cim` global para no arrastrarlos.

### App (prompt 09b) — 2026-07-21

**A3 — Configuración por entorno + eliminación de `client_secret`** ✅
- **Único punto de verdad**: nuevo módulo `shine-app/src/lib/config.ts` que lee de `expo-constants` (`Constants.expoConfig.extra`) con defaults de desarrollo y exporta `API_BASE`, `OAUTH_CLIENT_ID`, `JSONAPI_BASE`, `OAUTH` (endpoints `authorize`/`token`/`userinfo` derivados) y `OAUTH_SCOPE`.
- **`app.config.ts`** creado: extiende `app.json` (que sigue siendo la base estática) e inyecta en `extra` las variables `EXPO_PUBLIC_API_BASE` y `EXPO_PUBLIC_OAUTH_CLIENT_ID`.
- **`client_secret` eliminado por completo de la app**: quitado del `useAuthRequest` y del intercambio de token en `app/login.tsx`, y del refresh en `src/lib/api/client.ts`. El backend ya es cliente público con PKCE (ver 09a). No queda ningún secret en el bundle.
- Sustituidos todos los hardcodeos (`https://shine.ddev.site`, `shine_expo_app`) en `login.tsx`, `src/lib/api/client.ts` (ahora re-exporta `API_BASE`/`JSONAPI_BASE` desde config para consumidores existentes) y `src/store/auth.ts` (userinfo).
- **`.env.example`** creado con las dos variables `EXPO_PUBLIC_*` y nota de que no existe secret. `.env*.local` ya estaba en `.gitignore`.
- Verificación: `npx tsc --noEmit` limpio; `grep` de `shine.ddev.site`/`shine_dev_secret`/`shine_expo_app` en `src`+`app` solo aparece en `src/lib/config.ts` (defaults), sin ningún secret. Login web pendiente de probar por el usuario (`npm run web`).

### App (prompt 09c) — 2026-07-21 · bugs de datos A1 / A4 / A5

**A1 — Filtro por usuario en la capa API** ✅
- `getMySummaries` y `getMyImageSummaries` (`src/lib/api/statistics.ts`) ignoraban el `userUuid` que recibían (dependían solo del access hook del servidor). Ahora ambas aplican `filter[uid.id]={uuid}`.
- Revisado el resto de la capa API: `getMyActivitySummaries` (tracking.ts) ya filtraba; los helpers nuevos también. No queda ninguna función que reciba uuid y no lo aplique.

**A4/B6 — Iteración por actividad calculada desde el servidor** ✅
- Antes `iteration` era un contador global en `src/store/session.ts` que se incrementaba en cada actividad y no se reseteaba entre sesiones ni distinguía actividades → corrompía la gráfica de evolución.
- Nuevo `getNextIteration(bundle, uid, activityId)` en statistics.ts: consulta el último summary del usuario para esa actividad (`filter[uid.id]` + `filter[field_activityid.id]` + `sort=-created` + `page[limit]=1`) y devuelve `field_iteration + 1` (o 1 si no hay). Se calcula **justo antes de crear el summary** en `ActivityBase` (bundles `summary_vertical_text` y `summary3_vertical_text_blink`) e `ImageGroupPlayer` (`summary_image_position`).
- Eliminado el prop `iteration` de `ActivityRouter`/`ActivityBase`/`ImageGroupPlayer` y el consumo del contador global en `session/[id].tsx`.
- **Limitación conocida (B6, severidad baja)**: el cálculo se fía del cliente; con dos dispositivos a la vez pueden duplicarse iteraciones. Documentado en el JSDoc de `getNextIteration`; no se resuelve aquí (requeriría cálculo atómico en servidor).

**A5 — Estado "completada" derivado del servidor** ✅
- El completado era local (`completedIds` en `session/[id].tsx`) y se perdía al navegar.
- Nuevo `getCompletedActivityIds(uid, sessionId)` en statistics.ts: consulta los 3 bundles de summary filtrando por `uid` + `field_sessionid` y devuelve el `Set` de `field_activityid` (una actividad con summary en esa sesión ⇒ completada). Un fallo en un bundle no tumba el resto.
- `session/[id].tsx` deriva `completedIds` del servidor al cargar (efecto con deps `[id, user?.sub]`) y mantiene el estado local solo como **cache optimista** tras completar.

**Store huérfano** — `src/store/session.ts` ya no lo consume nadie tras estos cambios. Marcado en el propio fichero como candidato a eliminación en el prompt 09d (limpieza A7); no se borra aquí.

**Verificación** — `npx tsc --noEmit` limpio. Recorrido manual (completar → salir → volver y ver completada; dos navegadores sin duplicar iteraciones) pendiente de probar por el usuario.

### App (prompt 09d) — 2026-07-21 · errores + limpieza A7 + tooling A9

**Manejo de errores** ✅
- `session/[id].tsx`: `loadSession` envuelto en try/catch/finally con estado `error`; vista de error con `EmptyState` (icono `cloud-off`) + botones **Reintentar** (relanza la carga) y **Volver**, usando los componentes de `src/components/ui` y tokens de `src/theme`.
- `ActivityRouter`: nuevo fallback `UnsupportedActivity` (EmptyState + botón Volver) para `field_activity_format` fuera de `['blink','vertical_list','audio_reading']` y para tipos de nodo desconocidos. Se añadió el prop `onExit` (la pantalla de sesión lo conecta a volver a la lista). Ya no devuelve `null` → no hay pantalla en blanco.
- Login: el estado de error (alerta + botón "Reintentar") de 09b sigue cubierto (verificado).

**A7 — Código huérfano eliminado** ✅ (confirmado sin usos por grep antes de borrar)
- Borrados: `src/components/statistics/SessionProgressBar.tsx`, `src/components/statistics/StandardComparison.tsx`, `src/store/session.ts` (ya huérfano tras 09c).
- Funciones borradas: `getGroupSessions` (+ import `GroupSessionResource`) en `sessions.ts`; `getExplanationMessage` + interfaz `ExplanationMessageResource` en `messages.ts`.
- Contraste con FASE 12: la "barra de progreso real" del detalle de sesión usará el `ProgressBar` del design system (`src/components/ui/ProgressBar.tsx`), no el `SessionProgressBar` legacy; FASE 12 explícitamente evita comparaciones negativas, así que `StandardComparison` no se necesita. `git` conserva ambos si hicieran falta.

**A9 — Tooling** ✅ (ficheros creados; el usuario instala las dev-deps)
- `eslint.config.js` (flat config: `eslint-config-expo/flat` + `eslint-config-prettier`), `.prettierrc`, `.prettierignore`.
- `jest.config.js` (preset `jest-expo`, alias `@/*`), primer test `src/lib/__tests__/stripHtml.test.ts` (andamiaje para los tests de stats de FASE 10).
- Scripts en `package.json`: `lint`, `format`, `format:check`, `test`.
- Tests aislados del typecheck principal: `tsconfig.json` excluye `**/*.test.ts(x)` y `**/__tests__/**` (así `npx tsc --noEmit` sigue limpio sin `@types/jest`).

- Dev-deps ya declaradas en `package.json` (instaladas por el usuario 2026-07-21): `eslint ^9`, `eslint-config-expo ~10`, `eslint-config-prettier ^10`, `prettier ^3`, `jest-expo ~54`, `jest ^29.7`, `@types/jest ^29.5`.
- ⚠️ **`jest` fijado a `^29` (no 30)**: `jest-expo@54` depende del ecosistema Jest 29 (`@jest/globals ^29`, `jest-environment-jsdom ^29`…). Con Jest 30 el runner peta con `this._moduleMocker.clearMocksOnScope is not a function`. No subir Jest a 30 hasta que jest-expo lo soporte.
- ⚠️ Los tests requieren **Node ≥ 18** (el preset carga el winter runtime de Expo, que usa `FormData` global). Con Node 16 fallan con `FormData is not defined`. Ejecutar siempre con `nvm use 21`.

**Verificación (2026-07-21, Node 21)** ✅ — `npx tsc --noEmit` limpio; `npm test` → **4/4 pasan** (`stripHtml`); `npm run lint` → **0 errores** (15 warnings preexistentes: `react-hooks/exhaustive-deps`, `@typescript-eslint/array-type`, algún unused var; no bloquean). Sin referencias colgantes a lo eliminado. Fallback de formato desconocido y estado de error de sesión: pendientes de prueba manual visual por el usuario.

### ⚠️ Hallazgos para revisar
- **Contraseña del user 1 (`admin`) reseteada a `Test1234!`** durante las pruebas (no se conocía la original). Cambiarla si procede.
- **`automatic_authorization` del consumer en `off`**: el usuario ve una pantalla de consentimiento "Allow" en cada login. Valorar activarla para una experiencia de login sin fricción (decisión de producto, no bloqueante).

---

## FASE 10 — Capa de datos de estadísticas (ver plans/02-plan-ejecucion-v1.md)
**Estado**: ✅ Completada (2026-07-22) — Backend (10a) + App (10b).

### Backend (prompt 10a) — 2026-07-22

**Campos denormalizados en los 3 bundles de activitysummary** ✅
- Creados con storage compartido (entity_type `activitysummary`): `field_main_area` (entity_ref → taxonomy `areas`, cardinalidad 1), `field_activity_level` (entity_ref → taxonomy `activity_level`, múltiple como en los nodos), `field_correct_count` y `field_items_count` (integers, min 0). Instancias en `summary_vertical_text`, `summary3_vertical_text_blink` y `summary_image_position`, con widgets/formatters en form y view displays.
- Script de creación (one-off, idempotente) guardado en `shine/scripts/create-summary-stats-fields.php`; la fuente de verdad es la config exportada (16 YAMLs nuevos + 6 displays en `config/sync`).
- Verificado en JSON:API: `GET /jsonapi/activitysummary/summary_vertical_text` expone `field_correct_count`/`field_items_count` (attributes) y `field_main_area`/`field_activity_level` (relationships).

**Backfill de summaries históricos** ✅
- Script idempotente `shine/scripts/backfill-summary-area-level.php` (`ddev drush php:script`): copia área/nivel desde el nodo actividad (via `field_activityid`) solo si el campo está vacío; para `summary_image_position` deriva `field_correct_count`/`field_items_count` contando sus `field_activitylogs` (`field_correct`).
- Resultado: 15 procesados, **8 actualizados**, 7 sin actividad resoluble (todos apuntan al nodo 2, borrado — datos de prueba de mayo, irrecuperables). Segunda pasada: 0 cambios (idempotencia verificada).
- Muestreo verificado: niveles del summary = niveles del nodo; counts de imágenes = conteo real de logs (correct 1/1, items 2/2).
- En la primera pasada `field_main_area` quedó vacío porque ningún nodo actividad tenía área asignada (el "required + default" del campo solo aplica en formulario). **Resuelto el 2026-07-22**: se asignó área a A1 (Rendimiento y Velocidad), A2 (Competencia Lingüística y Fonológica) y A3 (Memoria de Trabajo) y se re-ejecutó el backfill: los 9 summaries con actividad resoluble tienen área verificada contra su nodo (✓ en los 9).

**B3 — `field_seconds` en `activity_image_position`: NO se añade** ✅
- La métrica de imágenes es precisión (correct/items), no tiempo; una zona objetivo temporal incentivaría velocidad sobre acierto (contra § 2.3 del plan). Documentado con el razonamiento completo en `docs/metrics-tracking.md` (incluida la corrección del inexistente `field_standard_time` → `field_seconds`).

**Índices** ✅
- Nota de esquema: la entidad no es revisionable ni traducible → la tabla base es `activitysummary` (no existe `activitysummary_field_data` como decía el plan).
- `activitysummary_update_10001()` (nuevo `activitysummary.install`, ejecutado con `drush updb`): índice compuesto `activitysummary__uid_created (uid, created)` en la tabla base. Verificado con `SHOW INDEX`.
- `(uid, field_activityid)`: `field_activityid` vive en tabla dedicada `activitysummary__field_activityid`, que ya tiene el índice `field_activityid_target_id` creado por core; junto al índice de uid de la base cubre el patrón de filtros de la app.

### App (prompt 10b) — 2026-07-22

**Campos denormalizados rellenados al crear summaries (los 3 tipos)** ✅
- Nuevo helper `summaryStatsRelationships(activity)` en `src/lib/api/tracking.ts`: extrae `field_main_area` y `field_activity_level` del nodo actividad que la app ya tiene cargado (JSON:API siempre devuelve el linkage de relaciones sin `include`; coste cero) y tolera nodos sin área/nivel.
- `ActivityBase` (bundles `summary_vertical_text` y `summary3_vertical_text_blink`) e `ImageGroupPlayer` (`summary_image_position`) añaden esas relaciones al payload; el de imágenes añade además `field_correct_count` (respuestas acertadas) y `field_items_count` (total) como attributes.
- Tipos TS actualizados: `SummaryStatsRelationships`/`SummaryStatsAttributes` compartidos por los 3 payloads en `src/types/activitysummary.ts` (+ campos en los resources), `field_main_area` en los relationships de `src/types/activity.ts`.
- Verificado contra el backend real: POST del payload exacto de la app → **201** con área, 3 niveles y contadores persistidos; el summary de prueba se borró después (queda pendiente una pasada manual desde la app por el usuario).

**Módulo `src/lib/stats/`** ✅ (puro: sin React ni imports de `src/lib/api`)
- Modelo propio `StatsSummary` + adaptador `toStatsSummary(resource)` (solo importa tipos): una única descarga de summaries alimenta todas las funciones.
- Funciones (plan § 2.4): `movingAverage` (ventana 5, serie suavizada de la misma longitud), `personalBest` (mayor precisión si el ejercicio la tiene, menor tiempo si no), `baselineVsRecent` (3 primeros vs 5 últimos; solo devuelve mejora > 10%, nunca negativa, null con < 5 intentos), `streak` (vigente si hoy o ayer; `today` inyectable), `weeklyBuckets` (lunes a domingo local, actividades y minutos por día + total), `areaAggregation` (dedicación por área ordenada ascendente — la menos trabajada primero, para "Para seguir practicando"), `accuracy` (% desde contadores, null si no aplica).
- **44 tests nuevos** en `src/lib/stats/__tests__/` (uno por función + adaptador): listas vacías, 1 y 4 intentos, racha rota, summaries sin área/fecha, empates, ventanas parciales. `npm test` → **45/45**; `npx tsc --noEmit` limpio; lint 0 errores.

**Nombres reales de actividad y sesión** ✅
- Elección: `include=field_activityid,field_sessionid` en la propia query de summaries (1 request; JSON:API lo soporta también en referencias multi-bundle) frente a fetch complementario (N requests). Verificado contra el backend: `included` trae los títulos reales.
- `getMySummaries` ahora devuelve `{ summaries, included }` y `getMyImageSummaries` amplía su include; nuevo helper `titlesFromIncluded()`. La pantalla de estadísticas usa títulos reales; eliminados los placeholders `Actividad ${idx+1}` y `Sesión ${uuid...}` (fallbacks humanos para nodos borrados: "Actividad ya no disponible" / "Sin sesión").
- Nota: los nodos referenciados borrados no vienen en `included`; el fallback cubre ese caso (hay 7 summaries históricos así).

**Config y drift** ✅
- `drush cex -y` + commit de TODO el diff: además de mis 22 ficheros, la exportación **reconcilia el drift preexistente del upgrade a core 11.4.4** (split del módulo search → `search_help`/`search_node` en `core.extension` y `search.page.*`, `gzip`→`compress` en `system.performance`, `field.settings` eliminado de core, schema nuevo de `backup_migrate`, `olivero.settings`) y recupera el permiso `grant simple_oauth codes` en `user.role.authenticated.yml` que en 09a no llegó a commitearse.
- Verificado: `drush config:status` → "No differences"; `drush cim -y` → "no changes" (idempotente).
- Sin errores en watchdog atribuibles a la sesión (2026-07-22 limpio).

---

## FASE 11 — Design system en código (ver plans/02-plan-ejecucion-v1.md)
**Estado**: ✅ Completada (2026-07-22)

- [x] `src/theme/` creado: colors, spacing, radii, shadows, typography + index. Portado del proyecto "Shine Design System" de claude.ai/design (fuente de verdad visual)
- [x] `src/components/ui/` creado: Button, Card, Icon (SVG propio con react-native-svg, sin deps nuevas), LevelChip, AreaChip, StatTile, MotivationCard, EmptyState, ProgressBar, BottomNav + barrel index
- [x] `tsc --noEmit` limpio
- [x] Fuente Nunito instalada y conectada (2026-07-21): `useFonts` en `app/_layout.tsx`, familias por peso en `src/theme/typography.ts` (Nunito_400Regular a Nunito_800ExtraBold)
- [x] Componente `Screen` (2026-07-22, prompt 11a): `src/components/ui/Screen.tsx`. Marco exterior `colors.shell` en web con columna centrada a `layout.appMaxWidth` (480), `colors.bgScreen` a pantalla completa en móvil; safe areas configurables (`edges`) y gutter opcional (`layout.screenGutter`). Exportado en el barrel.
- [x] Migrar pantallas existentes a los tokens (2026-07-22, prompt 11a): `dashboard.tsx` reescrito por completo con tokens + componentes DS (Card, Button, Icon, Screen); eliminados los objetos locales `C`/`FONT`/`Icon` Material y la paleta azul de facto (`#006b57`, `#3dbfa0`, `rgba(61,191,160,…)`). `session/[id].tsx` y `statistics/index.tsx` envueltos en `Screen` (centrado 480 en web) y sus `styles.screen`/`errorScreen` redundantes limpiados. Los componentes de `activities/` y `statistics/` ya estaban tokenizados (sin cambios). Añadido el icono `log-out` al set de `Icon.tsx`. Verificado: grep de hex/rgba sin resultados fuera de `theme`/`ui`, `tsc --noEmit` limpio (exit 0), sin unused locals. Falta el recorrido visual en web por el usuario.
- [x] Elección de fuente por el usuario (Nunito / OpenDyslexic) (2026-07-22, prompt 11b). Backend: campo `field_dyslexic_font` (boolean, default false) en la entidad user, expuesto en JSON:API; el usuario edita SU propia cuenta vía `PATCH /jsonapi/user/user/{uuid}` con el acceso self-edit de core (verificado: update propio ALLOWED, otro usuario DENIED, campo edit/view ALLOWED) — no se abrió ningún permiso extra; config exportada. App: OpenDyslexic (SIL OFL, `OFL.txt` incluido) en `assets/fonts/` en 3 pesos, cargada con `expo-font` junto a Nunito en el layout raíz; solo tiene Regular y Bold, mapeo de los 5 pesos del tema documentado en `src/theme/typography.ts` (regular/medium→Regular, semibold/bold/extrabold→Bold). Store `src/store/settings.ts` (Zustand + cache local en `@/lib/storage` + sync con el servidor al hacer login y al cambiar, best-effort). Hooks `useFamilies()`/`useText()` en `@/theme` (los `families`/`text` estáticos quedan como base interna Nunito); migrados a los hooks TODOS los componentes de `ui/`, las pantallas (`dashboard`, `session/[id]`, `statistics`, `login`), los players (`ActivityBase`, `VerticalTextPlayer`, `AudioReadingPlayer`, `ImageGroupPlayer`) y las tablas/gráficas de `statistics/`. Pantalla nueva `app/(auth)/settings.tsx`: selector por vista previa (dos tarjetas con la misma frase en cada fuente, "¿Cómo prefieres leer?", sin nombres técnicos ni "dislexia") + botón de cerrar sesión (movido aquí desde el BottomNav del dashboard). Acceso: avatar del dashboard → Ajustes (icono `settings` nuevo) y enlace discreto "Ajustar cómo se ven las letras" en la intro de ActivityBase. QA: `numberOfLines`/`flexShrink`/`adjustsFontSizeToFit` en Button, StatTile y el BottomNav para la mayor anchura de OpenDyslexic; sin bifurcar estilos por fuente. `tsc --noEmit` limpio, `npm run lint` sin errores (solo warnings pre-existentes), 45/45 tests verdes. Falta el recorrido visual en dispositivo por el usuario.

---

## FASE 12 — Pantallas v1 (ver plans/02-plan-ejecucion-v1.md)
**Estado**: 🔄 En curso (2026-07-22)

- [x] Dashboard (prompt 12a): `app/(auth)/dashboard.tsx` reescrito portando el mockup aprobado `HomeScreen.jsx` del proyecto "Shine Design System" (claude.ai/design). Muestra: saludo con nombre y momento del día (buenos días/tardes/noches) + fecha larga; resumen semanal (actividades y minutos) con `StatTile`, solo si hubo práctica esta semana (sin ceros tristes; minutos sub-minuto se muestran como "<1"); racha en positivo con `MotivationCard` (omitida si es 0); tarjeta "Continuar" con la sesión a medias más reciente (0 < progreso < 100) y CTA directo; lista de sesiones con descripción, chips `AreaChip`/`LevelChip` derivados de sus actividades, `ProgressBar` real (X de Y actividades) y badge "Completada"; `EmptyState` amable sin sesiones; skeletons de carga (no spinner); `BottomNav` con Inicio/Mi progreso. Datos: **2 queries** — `getSessions` (con include anidado `field_activities.field_main_area,field_activities.field_activity_level` para traer área/nivel en la misma petición) + `getMySummaries`; todo lo demás derivado en cliente con `@/lib/stats` (`weeklyBuckets`, `streak`, `toStatsSummary`) y el helper puro nuevo `src/lib/sessionCards.ts` (progreso por sesión, mapeo de área a etiqueta+icono por palabra clave, nivel Level0/1/2→beginner/intermediate/advanced tomando el mínimo). Reglas de producto respetadas (esfuerzo, sin comparaciones, sin "error"). Verificación: `tsc --noEmit` limpio, `npm run lint` sin errores (solo warnings pre-existentes); derivaciones (semana=2, racha=1, progreso por sesión) contrastadas contra los summaries reales de user 1. Falta el recorrido visual con usuario real por el usuario.
- [ ] Detalle de sesión: barra de progreso real, estados por actividad desde servidor, CTA continuar/empezar/repetir, celebración de sesión completa.
- [ ] Player: cuenta atrás en blink, permiso de micro denegado con fallback amable, guardado resiliente.
- [ ] Actividad completada: mejor marca personal, variante sesión completa, sin comparaciones negativas.
- [ ] Mi progreso: las 5 secciones del prompt 06.
- [ ] Mensajes de motivación filtrados por tipo de actividad (arreglo A6).

---

## FASE 8 — PWA completa y publicación en stores
**Estado**: ⏳ Pendiente

- [ ] Configurar `app.json` (bundle ID, iconos, splash)
- [ ] Configurar EAS Build
- [ ] Build de preview para pruebas
- [ ] Build de producción
- [ ] Publicar en Google Play
- [ ] Publicar en App Store
- [ ] Deploy versión web

---

## Notas técnicas y decisiones

| Fecha | Nota |
|---|---|
| 2026-03-27 | Frontend cambiado de Next.js a Expo por requisito de publicación en App Store iOS y acceso nativo al micrófono |
| 2026-03-27 | El typo `motiviation_message` no se puede renombrar con un simple find/replace: requiere migración de bundle en BD + rename de config. Ver sección de notas en Fase 0. |
| 2026-03-27 | `simple_oauth v6` NO tiene Password Grant (eliminado por OAuth 2.1). Usar Authorization Code + PKCE. Ver `docs/api-design.md`. |
| 2026-03-27 | Al testear endpoints OAuth con curl dentro de ddev exec, el `&` en URLs debe estar en comillas simples: `ddev exec bash -c 'curl -s "...?a=1&b=2"'` |
| 2026-03-27 | Consumer OAuth2: client_id=`shine_expo_app`, secret=`shine_dev_secret_2026` (cambiar en prod), scope=`authenticated_user_access` |
| 2026-03-27 | Claves RSA OAuth: `/var/www/html/private/oauth-keys/` (dentro del contenedor DDEV) |
| 2026-05-20 | `activity_blink` es un node type propio (no usa `field_activity_format`). La sesión debe cargar sus actividades con `getBlinkActivity()` — si se usa `getActivity()` devuelve 404 y el tipo queda mal resuelto. |
| 2026-05-20 | `drush cim` falló con sync divergente (gin theme, devel huérfano). Solución: usar `--partial` + neutralizar conflictos de theme/extension escribiendo la config activa en sync con `$sync->write()`. |
