# CLAUDE.md — Shine Project Reference

Guía de referencia para Claude Code. Actualizar con cada descubrimiento técnico relevante.

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
├── docs/                     ← documentación del proyecto
│   ├── description.md        ← qué es el proyecto
│   ├── structure.md          ← mapa de entidades Drupal
│   ├── architecture.md       ← stack tecnológico
│   ├── roadmap.md            ← fases de implementación
│   ├── progress.md           ← ESTADO ACTUAL — leer primero al reanudar
│   ├── conventions.md        ← convenciones de código
│   ├── activity-system.md    ← cómo funciona el sistema de actividades
│   ├── api-design.md         ← contratos de API (OAuth2, JSON:API endpoints)
│   ├── metrics-tracking.md   ← qué datos se capturan y cómo
│   └── statistics-design.md  ← diseño de estadísticas y gráficas
└── CLAUDE.md                 ← este fichero
```

> **Antes de trabajar**: leer `docs/progress.md` para saber en qué fase estamos.

---

## Entorno de desarrollo

- **DDEV** (Docker). Todos los comandos Drupal se ejecutan con prefijo `ddev`.
- **Web root**: `shine/public_html/`
- **Config sync**: `shine/config/sync/`
- **URL local**: `https://shine.ddev.site`

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
- **client_secret**: `shine_dev_secret_2026` ← **CAMBIAR EN PRODUCCIÓN**
- **UUID**: `5393c7d5-0f7b-4482-a770-0e5ce00639c2`
- **grant_types**: `authorization_code`, `refresh_token`
- **scope**: `authenticated_user_access`
- **redirect_uri**: `exp://localhost:19000/--/oauth2redirect`
- **pkce**: false (PKCE optional en el servidor, pero el cliente siempre lo usa)
- **confidential**: true

### Scope configurado

- **ID**: `authenticated_user_access`
- **Granularidad**: Rol `authenticated`

### Claves RSA

- Dentro del contenedor DDEV: `/var/www/html/private/oauth-keys/private.key` y `public.key`
- Fuera del contenedor: `shine/.ddev/homeadditions/private/oauth-keys/` (si existe) o re-generar con:
  ```bash
  ddev exec "openssl genrsa -out /var/www/html/private/oauth-keys/private.key 2048"
  ddev exec "openssl rsa -in /var/www/html/private/oauth-keys/private.key -pubout > /var/www/html/private/oauth-keys/public.key"
  ```

### Flujo completo Authorization Code + PKCE

```
1. App genera code_verifier (random string, min 43 chars)
2. App calcula code_challenge = BASE64URL(SHA256(code_verifier))
3. App abre navegador:
   GET /oauth/authorize?client_id=shine_expo_app
     &response_type=code
     &redirect_uri=exp://localhost:19000/--/oauth2redirect
     &code_challenge={code_challenge}
     &code_challenge_method=S256
     &scope=authenticated_user_access
4. Drupal muestra login → usuario se autentica → Drupal redirige:
   exp://localhost:19000/--/oauth2redirect?code={AUTH_CODE}
5. App intercambia código:
   POST /oauth/token
   grant_type=authorization_code
   &client_id=shine_expo_app
   &client_secret=shine_dev_secret_2026
   &code={AUTH_CODE}
   &redirect_uri=exp://localhost:19000/--/oauth2redirect
   &code_verifier={code_verifier}
6. Respuesta: { access_token, refresh_token, expires_in }
7. App guarda tokens en expo-secure-store
```

### Biblioteca Expo recomendada

```typescript
import * as AuthSession from 'expo-auth-session';
import * as WebBrowser from 'expo-web-browser';

WebBrowser.maybeCompleteAuthSession();

const discovery = {
  authorizationEndpoint: 'https://shine.ddev.site/oauth/authorize',
  tokenEndpoint: 'https://shine.ddev.site/oauth/token',
};

const [request, response, promptAsync] = AuthSession.useAuthRequest(
  {
    clientId: 'shine_expo_app',
    scopes: ['authenticated_user_access'],
    redirectUri: AuthSession.makeRedirectUri({ scheme: 'shine' }),
    usePKCE: true,
  },
  discovery
);
```

---

## JSON:API — Puntos clave

### Headers siempre requeridos

```
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json
Authorization: Bearer {access_token}
```

### Entidades expuestas

- `GET /jsonapi/node/activity` — actividades
- `GET /jsonapi/node/session` — sesiones
- `GET /jsonapi/node/group_session` — grupos de sesiones
- `GET /jsonapi/activity_message/explanation_message` — mensajes explicación
- `GET /jsonapi/activity_message/motivation_message` — mensajes motivación
- `POST /jsonapi/activitylog/logverticaltext` — crear log (requiere `create activitylog entities`)
- `POST /jsonapi/activitysummary/summary_vertical_text` — crear resumen (requiere `create activitysummary entities`)
- `GET /jsonapi/activitysummary/summary_vertical_text?filter[uid.id]={uuid}` — mis resúmenes

### Permisos JSON:API (rol `authenticated`)

Configurados en el módulo y a través de `drush role:perm:add`:
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
| `activitylog_register` | Endpoint legacy `POST /activitylog-register/add` |
| `activitysummary` | Entidad `activitysummary` (bundle: `summary_vertical_text`) |
| `preprocess` | Hooks `preprocess_node` (inyecta mensaje motivación en plantillas) |

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
```

---

## Gotchas conocidos

1. **`&` en curl dentro de ddev exec** → siempre usar `ddev exec bash -c 'curl -s "...?a=1&b=2"'`

2. **simple_oauth v6 no tiene Password Grant** → solo Authorization Code + PKCE

3. **Renombrar bundles de entidades custom** → NO se puede con `drush cim`. Requiere:
   - Actualizar tabla en BD
   - Recrear bundle programáticamente
   - `drush cex` para sincronizar

4. **Parches D8/D9/D10 en composer.json** → NO funcionan en D11. Eliminar todos antes del upgrade.

5. **Módulo `devel` en system.schema sin estar instalado** → puede bloquear `pm:enable`. Limpiar con:
   ```php
   ddev drush php-eval "\Drupal::keyValue('system.schema')->delete('devel');"
   ddev drush php-eval "\$c = \Drupal::configFactory()->getEditable('core.extension'); \$m = \$c->get('module'); unset(\$m['devel']); \$c->set('module', \$m)->save();"
   ```

6. **Temas con `core_version_requirement: ^10`** → no cargan en D11. Actualizar a `^10 || ^11`.

7. **`drush core:requirements --severity=2`** → no funciona en Drush 13. Usar `ddev drush status` y revisar `/admin/reports/status` en el navegador.
