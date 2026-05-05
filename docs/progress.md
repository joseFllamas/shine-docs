# Progreso de implementación — Shine

> Actualizado en tiempo real. Si el desarrollo se interrumpe, retomar desde el primer ítem sin marcar de la fase en curso.

**Última actualización**: 2026-03-27 (sesión 3)

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
