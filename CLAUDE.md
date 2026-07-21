# CLAUDE.md — Shine Project Reference

Guía de referencia para Claude Code. Actualizar con cada descubrimiento técnico relevante.

---

## Configuración en un PC nuevo

Pasos completos para arrancar el proyecto desde cero en una máquina nueva.

### 1. Requisitos previos

Instalar estas herramientas antes de empezar:

- **Docker Desktop** — requerido por DDEV
- **DDEV** — [docs.ddev.com/install](https://ddev.readthedocs.io/en/stable/users/install/)
- **nvm** — para gestionar versiones de Node
- **Node 21** — instalar y fijar como default:
  ```bash
  nvm install 21
  nvm alias default 21
  ```
- **Git**

### 2. Clonar el repositorio

```bash
git clone <URL_DEL_REPO> shine.test
cd shine.test
```

### 3. Configurar Drupal con DDEV

```bash
cd shine/

# Arrancar el entorno Docker
ddev start

# Instalar dependencias PHP
ddev composer install

# Importar la configuración (crea tablas, módulos, etc.)
ddev drush site:install --existing-config -y

# Importar configuración exportada (si ya existe BD)
# ddev drush cim -y

# Reconstruir cache
ddev drush cr

# Verificar que todo va bien
ddev drush status
```

> La URL local será `https://shine.ddev.site` (DDEV gestiona el SSL automáticamente).

### 4. Generar las claves RSA para OAuth2

Las claves **no se guardan en git** (están en `.gitignore`). Hay que regenerarlas:

```bash
ddev exec "mkdir -p /var/www/html/private/oauth-keys"
ddev exec "openssl genrsa -out /var/www/html/private/oauth-keys/private.key 2048"
ddev exec "openssl rsa -in /var/www/html/private/oauth-keys/private.key -pubout -out /var/www/html/private/oauth-keys/public.key"
ddev exec "chmod 600 /var/www/html/private/oauth-keys/private.key"
```

### 5. Verificar el scope OAuth2

Después de `drush cim`, comprobar que el nombre del scope es correcto (ver gotcha #3):

```bash
ddev drush php-eval "
\$scope = \Drupal::entityTypeManager()->getStorage('oauth2_scope')->load('authenticated_user_access');
echo \$scope->getName();
"
# Debe imprimir exactamente: authenticated_user_access
# Si no, corregir:
ddev drush php-eval "
\$s = \Drupal::entityTypeManager()->getStorage('oauth2_scope')->load('authenticated_user_access');
\$s->set('name', 'authenticated_user_access')->save();
"
```

### 6. Crear usuario administrador

```bash
ddev drush user:create admin --mail="admin@example.com" --password="password"
ddev drush user:role:add administrator admin
# O resetear contraseña de user 1:
ddev drush user:password admin "nueva_password"
```

### 7. Instalar dependencias de la app Expo

```bash
cd ../shine-app/
nvm use 21
npm install --legacy-peer-deps
npx expo install --fix
```

### 8. Añadir redirect URI para el entorno local

Si el entorno local tiene una IP o puerto diferente, añadirlo al consumer OAuth2:

```bash
cd ../shine/
ddev drush php-eval "
\$consumers = \Drupal\consumers\Entity\Consumer::loadMultiple();
foreach (\$consumers as \$c) {
  if (\$c->get('client_id')->value === 'shine_expo_app') {
    \$uris = \$c->get('redirect')->getValue();
    \$uris[] = ['value' => 'http://localhost:8081'];
    \$c->set('redirect', \$uris)->save();
    echo 'OK';
  }
}
"
```

### 9. Arrancar y probar

```bash
# Terminal 1 — Drupal ya está corriendo con DDEV
# Verificar: https://shine.ddev.site

# Terminal 2 — App Expo
cd shine-app/
nvm use 21
npm run web    # abre http://localhost:8081
```

### Archivos que NO están en git y hay que crear/configurar manualmente

| Archivo/Directorio | Motivo | Cómo obtenerlo |
|---|---|---|
| `shine/public_html/sites/default/settings.local.php` | Credenciales locales | Copiar de `settings.php` y ajustar |
| `shine/private/oauth-keys/private.key` | Clave privada RSA | Regenerar (paso 4) |
| `shine/private/oauth-keys/public.key` | Clave pública RSA | Regenerar (paso 4) |
| `shine-app/.env.local` | Variables de entorno locales | Crear si existe `.env.example` |

---

## Estructura del proyecto

```
shine.test/
├── shine/                    ← raíz del proyecto Drupal (Composer root)
│   ├── composer.json
│   ├── public_html/          ← web root (Drupal)
│   │   ├── core/
│   │   ├── modules/
│   │   │   └── custom/       ← 5 módulos custom
│   │   ├── themes/
│   │   │   └── custom/shine/ ← tema activo
│   │   └── sites/default/
│   └── config/sync/          ← config exportada (drush cex)
├── shine-app/                ← proyecto Expo (React Native + web)
│   ├── app/                  ← rutas Expo Router
│   │   ├── _layout.tsx       ← root layout con auth guard
│   │   ├── index.tsx         ← redirect inicial
│   │   ├── login.tsx         ← OAuth2 + PKCE
│   │   └── (auth)/
│   │       ├── dashboard.tsx
│   │       ├── session/[id].tsx
│   │       └── statistics/index.tsx
│   ├── src/
│   │   ├── types/            ← jsonapi.ts, activity.ts, session.ts, activitylog.ts, activitysummary.ts
│   │   ├── lib/
│   │   │   ├── storage.ts    ← SecureStore (móvil) / localStorage (web)
│   │   │   ├── stripHtml.ts
│   │   │   └── api/          ← client.ts, activities.ts, sessions.ts, tracking.ts, statistics.ts, messages.ts
│   │   ├── store/            ← auth.ts, session.ts (Zustand)
│   │   └── components/
│   │       ├── activities/   ← ActivityBase, ActivityRouter, VerticalTextPlayer, AudioReadingPlayer
│   │       └── statistics/   ← ActivityEvolutionChart, StandardComparison, SessionProgressBar, SessionComparisonTable, StatsDataTable
│   ├── package.json
│   ├── app.json
│   └── README.md
├── docs/                     ← documentación del proyecto
│   ├── description.md
│   ├── structure.md
│   ├── architecture.md
│   ├── roadmap.md
│   ├── progress.md           ← ESTADO ACTUAL — leer primero al reanudar
│   ├── conventions.md
│   ├── activity-system.md
│   ├── api-design.md
│   ├── metrics-tracking.md
│   └── statistics-design.md
└── CLAUDE.md                 ← este fichero
```

> **Antes de trabajar**: leer `docs/progress.md` para saber en qué fase estamos.

---

## Entorno de desarrollo

- **DDEV** (Docker). Todos los comandos Drupal se ejecutan con prefijo `ddev` desde `shine/`.
- **Web root**: `shine/public_html/`
- **Config sync**: `shine/config/sync/`
- **URL local**: `https://shine.ddev.site`
- **Node**: se requiere >= 20. Usar `nvm use 21`. Default: `nvm alias default 21`.

Comandos frecuentes:
```bash
ddev start / ddev stop
ddev drush <comando>
ddev composer <comando>
ddev exec bash -c '<comando-dentro-del-contenedor>'
```

> **Importante**: para URLs con `&` en curl dentro de `ddev exec`, usar comillas simples:
> ```bash
> ddev exec bash -c 'curl -s "https://shine.ddev.site/ruta?a=1&b=2"'
> ```
> Sin comillas simples, el shell interpreta `&` como "ejecutar en background" y se pierde el resto de la URL.

### Arrancar la app Expo

```bash
cd shine-app
nvm use 21          # o nvm alias default 21 para hacerlo permanente
npm run web         # navegador en localhost:8081
npm start           # QR para Expo Go (SDK 54)
```

---

## Stack actual (2026-03-27)

| Componente | Versión | Notas |
|---|---|---|
| Drupal | 11.3.5 | Upgrade completado desde D10 |
| PHP | 8.3 | |
| MariaDB | 10.11 | |
| Drush | 13.7.2 | |
| simple_oauth | 6.1.0 | OAuth2 Authorization Code + PKCE |
| JSON:API | core | Habilitado |
| CORS | services.yml | Habilitado, `allowedOrigins: ['*']` |
| Expo SDK | 54 | SDK 55 incompatible con Expo Go actual |
| React Native | 0.76.7 | |
| Expo Router | 4.x | File-based routing |
| Zustand | 5.x | Estado global |

---

## OAuth2 — Cómo funciona en este proyecto

### Versión y flujo

`simple_oauth v6` **no tiene Password Grant** (eliminado siguiendo OAuth 2.1). El flujo para la app Expo es **Authorization Code + PKCE**.

### Endpoints OAuth2

```
GET  /oauth/authorize     ← inicia el flujo (browser/WebView)
POST /oauth/token         ← intercambia código por token / refresh
GET  /oauth/userinfo      ← info del usuario autenticado (JWT)
GET  /oauth/jwks          ← claves públicas JWT
```

### Consumer configurado

- **client_id**: `shine_expo_app`
- **client_secret**: (vacío) — desde 2026-07-21 el consumer es **cliente público** (B4). Con PKCE el secreto es innecesario y en un bundle JS sería público de facto. NO reintroducir secret.
- **UUID**: `5393c7d5-0f7b-4482-a770-0e5ce00639c2`
- **grant_types**: `authorization_code`, `refresh_token`
- **scope**: `authenticated_user_access`
- **redirect_uris configuradas**:
  - `exp://localhost:19000/--/oauth2redirect` — Expo Go (móvil)
  - `http://localhost:8081` — web dev
- **confidential**: false (cliente público)

> El consumer es una **entidad de contenido** (vive en BD, no en `config/sync`): `drush cex` no lo captura y hay que recrearlo/ajustarlo en cada entorno (ver "Configuración en un PC nuevo"). Al crearlo, poner `confidential: false` y no asignar secret.
>
> **Pendiente conocido**: el rol `authenticated` no tiene el permiso `grant simple_oauth codes`, que simple_oauth exige para completar `/oauth/authorize`. Hoy solo el user 1 (super user) puede completar el login. Conceder ese permiso antes de probar login con usuarios reales.

### Scope — BUG CONOCIDO Y CORREGIDO

`simple_oauth` v6 busca el scope con `loadByName()` → `loadByProperties(['name' => $identifier])`.
El campo `name` es el label humano, **NO** el machine name. Por tanto, el `name` del scope entity
**debe ser idéntico al identifier enviado en el request OAuth**.

**Correcto**: scope con `name = 'authenticated_user_access'` (sin mayúsculas, con guiones bajos).
**Incorrecto**: scope con `name = 'Authenticated user access'` → devuelve `invalid_scope`.

Si aparece `invalid_scope`, verificar con:
```bash
ddev drush php-eval "
\$scope = \Drupal::entityTypeManager()->getStorage('oauth2_scope')->load('authenticated_user_access');
echo \$scope->getName();  // debe ser exactamente 'authenticated_user_access'
"
```
Para corregir:
```bash
ddev drush php-eval "
\$s = \Drupal::entityTypeManager()->getStorage('oauth2_scope')->load('authenticated_user_access');
\$s->set('name', 'authenticated_user_access')->save();
"
```

### Scope configurado

- **ID**: `authenticated_user_access`
- **name**: `authenticated_user_access` (igual que el ID — ver bug arriba)
- **Granularidad**: Rol `authenticated`
- **grant_types habilitados**: `authorization_code`, `refresh_token`

### Claves RSA

- Dentro del contenedor DDEV: `/var/www/html/private/oauth-keys/private.key` y `public.key`
- Re-generar con:
  ```bash
  ddev exec "openssl genrsa -out /var/www/html/private/oauth-keys/private.key 2048"
  ddev exec "openssl rsa -in /var/www/html/private/oauth-keys/private.key -pubout > /var/www/html/private/oauth-keys/public.key"
  ```

### Añadir redirect URI al consumer (desarrollo)

```bash
ddev drush php-eval "
\$consumers = \Drupal\consumers\Entity\Consumer::loadMultiple();
foreach (\$consumers as \$c) {
  if (\$c->get('client_id')->value === 'shine_expo_app') {
    \$uris = \$c->get('redirect')->getValue();
    \$uris[] = ['value' => 'TU_URI_AQUI'];
    \$c->set('redirect', \$uris)->save();
    echo 'OK';
  }
}
"
```

---

## Expo App — Puntos clave

### expo-secure-store no funciona en web

`expo-secure-store` lanza error en web. Usar el wrapper en `src/lib/storage.ts`:
```typescript
// usa SecureStore en móvil, localStorage en web
import { getItem, setItem, deleteItem } from '@/lib/storage';
```
**Nunca importar expo-secure-store directamente** en código compartido móvil/web.

### Expo SDK y Expo Go

- **SDK 54** → compatible con Expo Go actual en App Store.
- **SDK 55** → requiere development build (Expo Go dice "incompatible").
- Si se quiere SDK 55: `eas build --platform ios --profile development`.

### Instalación desde cero

```bash
nvm use 21
npm install --legacy-peer-deps
npx expo install --fix   # ajusta versiones exactas para el SDK
npm run web              # o npm start para móvil
```

### Dependencias que hay que instalar manualmente

Estas no están en `package.json` porque no se puede ejecutar expo install desde Claude (Node 16 del sistema vs Node 21 de nvm):

```bash
# Web support (ya instalado si npm install funcionó)
npx expo install react-dom react-native-web @expo/metro-runtime

# Fase 6 — gráficas
npx expo install victory-native react-native-svg

# Fase 7 — audio
npx expo install expo-av
```

### Formatos de actividad reales (field_activity_format)

| Valor | Descripción | Player |
|---|---|---|
| `blink` | Auto-avance cada `field_seconds` segundos | `VerticalTextPlayer` |
| `vertical_list` | Tap para avanzar, animación slide | `VerticalTextPlayer` |
| `audio_reading` | Lectura en voz alta con cronómetro | `AudioReadingPlayer` |

### field_text — formato HTML

`field_text` devuelve array de `{ value: "<p>CA SA</p>", processed: "<p>CA SA</p>" }`.
Usar `stripHtml()` de `src/lib/stripHtml.ts` para obtener texto plano.

---

## JSON:API — Puntos clave

### Headers siempre requeridos

```
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json
Authorization: Bearer {access_token}
```

### Entidades expuestas

- `GET /jsonapi/node/activity` — actividades (`field_activity_format`, `field_text`, `field_seconds`)
- `GET /jsonapi/node/session?include=field_activities` — sesiones
- `GET /jsonapi/node/group_session` — grupos de sesiones
- `GET /jsonapi/activity_message/explanation_message` — requiere auth
- `GET /jsonapi/activity_message/motivation_message` — requiere auth
- `POST /jsonapi/activitylog/logverticaltext` — crear log
- `POST /jsonapi/activitysummary/summary_vertical_text` — crear resumen
- `GET /jsonapi/activitysummary/summary_vertical_text?filter[uid.id]={uuid}` — mis resúmenes

### Filtros útiles para estadísticas

```
?filter[uid.id]={user-uuid}
&filter[field_activityid.id]={activity-uuid}
&filter[field_sessionid.id]={session-uuid}
&sort=created
&page[limit]=50
```

### Permisos JSON:API (rol `authenticated`)

- `create activitylog entities`
- `view own activitylog entities`
- `create activitysummary entities`
- `view own activitysummary entities`
- `view activity_message`
- `access content`

### Acceso implementado mediante hooks

Los módulos `activitylog` y `activitysummary` tienen implementados en sus `.module`:
- `hook_activitylog_access()` / `hook_activitysummary_access()` — permite `view own`
- `hook_activitylog_create_access()` / `hook_activitysummary_create_access()` — permite `create`

---

## Módulos custom

| Módulo | Función |
|---|---|
| `activity_message` | Entidad `activity_message` (bundles: `explanation_message`, `motivation_message`) |
| `activitylog` | Entidad `activitylog` (bundle: `logverticaltext`) |
| `activitylog_register` | Endpoint legacy `POST /activitylog-register/add` (mantener hasta migración completa) |
| `activitysummary` | Entidad `activitysummary` (bundle: `summary_vertical_text`) |
| `preprocess` | Hooks `preprocess_node` (inyecta mensaje motivación en plantillas Drupal acopladas) |

---

## Comandos útiles

```bash
# Estado general
ddev drush status

# Reconstruir cache
ddev drush cr

# Importar/exportar config
ddev drush cim -y
ddev drush cex -y

# Permisos
ddev drush role:perm:add authenticated "permission1,permission2"

# Ver módulos habilitados
ddev drush pm:list --status=enabled

# Habilitar módulo
ddev drush pm:enable nombre_modulo -y

# PHP eval (útil para debug)
ddev drush php-eval "echo Drupal::VERSION;"

# Acceder a la BD
ddev mysql

# Ver logs de error recientes
ddev drush watchdog:show --count=20 --severity=Error
```

---

## Gotchas conocidos

1. **`&` en curl dentro de ddev exec** → siempre usar `ddev exec bash -c 'curl -s "...?a=1&b=2"'`

2. **simple_oauth v6 no tiene Password Grant** → solo Authorization Code + PKCE

3. **simple_oauth `invalid_scope`** → el campo `name` del scope entity debe ser igual al identifier OAuth (ver sección OAuth2 arriba)

4. **expo-secure-store en web** → lanza error. Usar `src/lib/storage.ts` como wrapper

5. **Claude Code usa Node 16 del sistema** → los comandos `npx expo install` fallan desde Claude. Ejecutar siempre en terminal del usuario con `nvm use 21`

6. **Expo SDK 55 incompatible con Expo Go** → usar SDK 54 para desarrollo con Expo Go. SDK 55 requiere development build

7. **Renombrar bundles de entidades custom** → NO se puede con `drush cim`. Requiere actualizar BD + recrear bundle programáticamente

8. **Parches D8/D9/D10 en composer.json** → NO funcionan en D11. Eliminar todos antes del upgrade

9. **Módulo `devel` en system.schema sin estar instalado** → puede bloquear `pm:enable`. Limpiar con:
   ```php
   ddev drush php-eval "\Drupal::keyValue('system.schema')->delete('devel');"
   ddev drush php-eval "\$c = \Drupal::configFactory()->getEditable('core.extension'); \$m = \$c->get('module'); unset(\$m['devel']); \$c->set('module', \$m)->save();"
   ```

10. **Temas con `core_version_requirement: ^10`** → no cargan en D11. Actualizar a `^10 || ^11`.

11. **`drush core:requirements --severity=2`** → no funciona en Drush 13. Usar `ddev drush status`.

12. **`npm install` en shine-app** → siempre con `--legacy-peer-deps` por conflictos de peer deps de Expo.

13. **JSON:API POST a activitylog/activitysummary devuelve 422** → el campo `label` (base field) es **required** aunque no aparece en los campos `field_*`. Siempre incluirlo en `attributes`: `{ label: "...", field_item: ..., ... }`.
