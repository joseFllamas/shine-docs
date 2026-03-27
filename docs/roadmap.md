# Roadmap de implementación — Shine

Cada fase tiene prerequisitos, entregables y criterios de verificación claros.

---

## Fase 0 — Limpieza y prerequisitos
**Estado**: Pendiente
**Prerequisito**: Ninguno

### Tareas
- [ ] Eliminar `/structure.md` de la raíz (el correcto está en `docs/structure.md`)
- [ ] Corregir el typo `motiviation_message` → `motivation_message` en todos los archivos de configuración y código (bundle name, entity class, config YAMLs, preprocess.module)
- [ ] Actualizar descripciones `@todo` en los `.info.yml` de módulos custom
- [ ] Añadir `package: shine` a todos los módulos custom en sus `.info.yml`

### Verificación
- `docs/` contiene todos los archivos de documentación
- `grep -r "motiviation" shine/` no devuelve resultados

---

## Fase 1 — Upgrade Drupal 10 → 11
**Estado**: Pendiente
**Prerequisito**: Fase 0 completa

### Tareas
- [ ] Verificar compatibilidad de módulos contrib con D11: `ddev drush pm:security`
- [ ] Actualizar `composer.json`: `"drupal/core-recommended": "^11"`
- [ ] Ejecutar: `ddev composer update drupal/core-recommended drupal/core-composer-scaffold --with-all-dependencies`
- [ ] Ejecutar: `ddev drush updb` (actualizaciones de base de datos)
- [ ] Ejecutar: `ddev drush cr` (limpiar caché)
- [ ] Resolver incompatibilidades de módulos si las hay

### Verificación
- `ddev drush status` muestra Drupal 11.x
- Todos los módulos habilitados sin errores en `/admin/reports/status`
- Las entidades custom (activitylog, activitysummary, activity_message) funcionan correctamente

---

## Fase 2 — Habilitación de JSON:API + Autenticación
**Estado**: Pendiente
**Prerequisito**: Fase 1 completa

### Tareas
- [ ] Habilitar módulo `jsonapi`: `ddev drush en jsonapi`
- [ ] Instalar `drupal/simple_oauth`: `ddev composer require drupal/simple_oauth`
- [ ] Configurar CORS en `sites/default/services.yml` (allowedOrigins para el frontend Expo)
- [ ] Crear Consumer OAuth2 en `/admin/config/services/consumer`
- [ ] Configurar permisos JSON:API para las entidades (qué puede leer/escribir un usuario autenticado)
- [ ] Exportar config: `ddev drush cex`

### Verificación
- `GET https://shine.ddev.site/jsonapi/node/activity` devuelve JSON válido
- Un token OAuth2 se puede obtener via `POST /oauth/token`
- Con Bearer token, se puede crear un `activitylog` via `POST /jsonapi/activitylog/logverticaltext`

---

## Fase 3 — Refactorización del endpoint de registro
**Estado**: Pendiente
**Prerequisito**: Fase 2 completa

### Objetivo
Reemplazar el endpoint custom `POST /activitylog-register/add` por escritura directa a JSON:API. Esto elimina el Controller custom y simplifica la arquitectura.

### Tareas
- [ ] Verificar que crear `activitylog` + `activitysummary` via JSON:API es viable en una sola secuencia de llamadas
- [ ] Actualizar permisos: usuario autenticado puede crear `activitylog` y `activitysummary`
- [ ] Documentar el nuevo flujo en `docs/api-design.md`
- [ ] Mantener el endpoint legacy `/activitylog-register/add` hasta que el frontend esté migrado

### Verificación
- El frontend puede crear un `activitylog` y un `activitysummary` via JSON:API con token JWT
- Los datos quedan correctamente enlazados (field_activitylogs en summary apunta a los logs creados)

---

## Fase 4 — Proyecto Expo (estructura base)
**Estado**: Pendiente
**Prerequisito**: Fase 2 completa (la API debe estar disponible)

### Tareas
- [ ] Crear proyecto: `npx create-expo-app@latest shine-app --template tabs`
- [ ] Configurar TypeScript
- [ ] Instalar dependencias base:
  - `@drupal/drupal-jsonapi-params` — queries JSON:API
  - `nativewind` — Tailwind para React Native
  - `zustand` — estado global
  - `expo-secure-store` — almacenamiento seguro del JWT
- [ ] Configurar cliente API (`src/lib/api/client.ts`) con interceptor de token
- [ ] Implementar flujo de autenticación (login, token refresh, logout)
- [ ] Estructura de carpetas (ver `docs/conventions.md`)

### Verificación
- La app arranca en iOS Simulator, Android Emulator y web (`npx expo start --web`)
- El login contra Drupal funciona y devuelve un JWT
- Una llamada `GET /jsonapi/node/activity` autenticada funciona desde la app

---

## Fase 5 — Activity Player
**Estado**: Pendiente
**Prerequisito**: Fase 4 completa

### Objetivo
Implementar el reproductor de actividades en React Native, replicando la funcionalidad actual del JavaScript acoplado.

### Tareas
- [ ] Componente base `ActivityBase` (abstract, con métodos `prepare`, `play`, `end`, `sendLog`)
- [ ] Componente `VerticalTextPlayer` (equivalente a `ActivityTextList`)
- [ ] Componente `ImagePositionPlayer` (completar la implementación parcial actual)
- [ ] Integración con tracking: al finalizar, crear `activitylog` + `activitysummary` via JSON:API
- [ ] Mensaje de motivación: cargar `activity_message.motivation_message` al inicio de cada actividad
- [ ] Mensaje de explicación: mostrar `activity_message.explanation_message` en la intro
- [ ] Animación de celebración al completar (reemplazar confetti web por animación nativa)
- [ ] Navegación entre actividades de una sesión

### Verificación
- Un usuario puede completar una sesión completa de actividades de tipo texto
- Los datos quedan registrados en Drupal (activitylog + activitysummary con los campos correctos)
- La pantalla de motivación aparece al finalizar

---

## Fase 6 — Estadísticas y gráficas
**Estado**: Pendiente
**Prerequisito**: Fase 5 completa (necesita datos reales de activitysummary)

### Tareas
- [ ] Instalar Victory Native: `npx expo install victory-native`
- [ ] Pantalla de evolución por actividad (`ActivityEvolutionScreen`):
  - Gráfica de líneas: tiempo total por iteración
  - Query: `GET /jsonapi/activitysummary/summary_vertical_text?filter[uid]=me&filter[activityid]={id}`
- [ ] Pantalla de comparativa con estándar (`StandardComparisonScreen`):
  - Añadir campo `field_standard_time` a `node.activity` en Drupal
  - Gráfica de barras con línea de referencia
- [ ] Pantalla de progreso de sesión (`SessionProgressScreen`):
  - Porcentaje de actividades completadas
  - Query: activitysummary filtrado por sessionid
- [ ] Dashboard principal con resumen de todas las estadísticas

### Verificación
- Las gráficas muestran datos reales del usuario logueado
- La comparativa con el estándar funciona cuando `field_standard_time` está definido

---

## Fase 7 — Soporte de audio y micrófono
**Estado**: Pendiente
**Prerequisito**: Fase 5 completa

### Tareas
- [ ] Instalar `expo-av` y solicitar permisos de micrófono (`expo-modules-core`)
- [ ] Nuevo tipo de actividad en Drupal: `audio_reading` (node type)
- [ ] Bundle `logaudioreading` en `activitylog` con campos: `field_recorded_time` (integer, ms)
- [ ] Componente `AudioReadingPlayer`:
  - Muestra el texto a leer
  - Botón "Start reading" → inicia grabación de tiempo (no necesariamente graba audio)
  - Botón "Done" → registra el tiempo total
  - Opcionalmente: reproducción de audio de ejemplo (`expo-av`)
- [ ] Registrar tipo en `activitylog_register` (si aún existe el endpoint legacy)

### Verificación
- Un usuario puede completar una actividad de lectura en voz alta
- El tiempo registrado es consistente con el tiempo real de lectura

---

## Fase 8 — PWA completa y publicación en stores
**Estado**: Pendiente
**Prerequisito**: Fases 5, 6 y 7 completas

### Tareas
- [ ] Configurar `app.json` con bundle ID, iconos, splash screen
- [ ] Configurar EAS Build: `eas build:configure`
- [ ] Build de preview para pruebas: `eas build --platform all --profile preview`
- [ ] Testing en dispositivos reales
- [ ] Build de producción: `eas build --platform all --profile production`
- [ ] Publicar en Google Play (internal testing primero)
- [ ] Publicar en App Store (TestFlight primero)
- [ ] Para web: `npx expo export --platform web` → desplegar en servidor estático

### Verificación
- La app funciona correctamente en iPhone real y Android real
- Se puede instalar desde las stores
- La versión web es accesible y funcional como fallback

---

## Resumen visual

```
Fase 0 ──► Fase 1 ──► Fase 2 ──► Fase 3
                          │
                          ├──► Fase 4 ──► Fase 5 ──► Fase 6
                                               │
                                               └──► Fase 7 ──► Fase 8
```
