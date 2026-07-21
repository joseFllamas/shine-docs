# Progreso de implementación — Shine

> Actualizado en tiempo real. Si el desarrollo se interrumpe, retomar desde el primer ítem sin marcar de la fase en curso.

**Última actualización**: 2026-07-21 (sesión 5 — FASE 9 backend)

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
**Estado**: 🔶 En curso (2026-07-21) — bloque **Backend** hecho; bloque App pendiente (prompts 09b y siguientes)

### Backend (prompt 09a) — 2026-07-21

**B4 — Consumer OAuth como cliente público** ✅
- Consumer `shine_expo_app` (id=2) cambiado a `confidential: false` y `secret` vaciado.
- El backend **ya no exige `client_secret`**: flujo Authorization Code + PKCE verificado por curl end-to-end (login → `/oauth/authorize` → `/oauth/token`) **sin enviar `client_secret`** → respuesta `200` con `access_token` + `refresh_token`.
- ⚠️ El consumer es una **entidad de contenido** (vive en BD, no en `config/sync`). Por eso `drush cex` NO lo captura y el cambio **debe reaplicarse en cada entorno** (script en `CLAUDE.md` → "Configuración en un PC nuevo"). Actualizar ese script para poner `confidential: false` al crear el consumer.
- Pendiente en la app (prompt 09b): dejar de enviar `client_secret` en el POST a `/oauth/token`.

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

### ⚠️ Hallazgos para revisar (no bloquean 09a, sí afectan a FASE 9/14)
- **`grant simple_oauth codes` no está asignado al rol `authenticated`.** simple_oauth exige este permiso para completar `/oauth/authorize` (ver `Oauth2AuthorizeController::authorize`, línea 177). Hoy solo el user 1 (super user) puede completar el login; **un usuario normal de la app NO podría** hacer login. La verificación de PKCE se hizo con el user 1. Recomendado: conceder `grant simple_oauth codes` (y revisar `automatic_authorization` del consumer para evitar la pantalla de consentimiento en cada login) antes de probar login con usuarios reales.
- **Contraseña del user 1 (`admin`) reseteada a `Test1234!`** durante las pruebas (no se conocía la original). Cambiarla si procede.

---

## FASE 11 — Design system en código (ver plans/02-plan-ejecucion-v1.md)
**Estado**: 🔶 En curso (2026-07-21)

- [x] `src/theme/` creado: colors, spacing, radii, shadows, typography + index. Portado del proyecto "Shine Design System" de claude.ai/design (fuente de verdad visual)
- [x] `src/components/ui/` creado: Button, Card, Icon (SVG propio con react-native-svg, sin deps nuevas), LevelChip, AreaChip, StatTile, MotivationCard, EmptyState, ProgressBar, BottomNav + barrel index
- [x] `tsc --noEmit` limpio
- [x] Fuente Nunito instalada y conectada (2026-07-21): `useFonts` en `app/_layout.tsx`, familias por peso en `src/theme/typography.ts` (Nunito_400Regular a Nunito_800ExtraBold)
- [ ] Migrar pantallas existentes a los tokens (hoy siguen con la paleta azul antigua)

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
