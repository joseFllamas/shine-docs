# Diseño de API — Shine

## Resumen

El sistema expone los datos de Drupal mediante **JSON:API** (módulo core de Drupal). La autenticación es **OAuth 2.0** via `drupal/simple_oauth`. El frontend Expo se comunica exclusivamente a través de estos endpoints.

---

## Autenticación

### Flujo OAuth 2.0 — Authorization Code + PKCE

simple_oauth v6 **elimina el Password Grant** (deprecado en OAuth 2.1). El flujo correcto para apps móviles es **Authorization Code con PKCE** (RFC 7636), que es seguro para clientes públicos.

#### Paso 1 — Abrir el navegador del dispositivo

```
GET /oauth/authorize
  ?client_id=shine_expo_app
  &response_type=code
  &redirect_uri=exp://localhost:19000/--/oauth2redirect
  &code_challenge={PKCE_CHALLENGE}
  &code_challenge_method=S256
  &scope=authenticated_user_access
```

- El usuario ve el formulario de login de Drupal
- Tras el login, acepta el acceso (o se autoriza automáticamente si `automatic_authorization=true`)
- El servidor redirige a `redirect_uri?code={AUTH_CODE}`

En Expo usar `expo-auth-session`:
```typescript
import * as AuthSession from 'expo-auth-session';

const request = new AuthSession.AuthRequest({
  clientId: 'shine_expo_app',
  scopes: ['authenticated_user_access'],
  redirectUri: AuthSession.makeRedirectUri({ scheme: 'shine' }),
  usePKCE: true,
});
```

#### Paso 2 — Intercambiar código por token

```
POST /oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
&client_id=shine_expo_app
&client_secret=shine_dev_secret_2026
&code={AUTH_CODE}
&redirect_uri=exp://localhost:19000/--/oauth2redirect
&code_verifier={PKCE_VERIFIER}
```

**Respuesta:**
```json
{
  "token_type": "Bearer",
  "expires_in": 86400,
  "access_token": "eyJ...",
  "refresh_token": "def50200..."
}
```

El `access_token` se incluye en todas las requests autenticadas:
```
Authorization: Bearer eyJ...
Content-Type: application/vnd.api+json
```

El token se almacena en `expo-secure-store` (nunca en AsyncStorage).

### Refresh del token

```
POST /oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token
&refresh_token={refresh_token}
&client_id=shine_expo_app
&client_secret=shine_dev_secret_2026
```

### Configuración del Consumer (Drupal admin)

- **consumer_id**: 2 (entity ID)
- **client_id**: `shine_expo_app`
- **client_secret**: `shine_dev_secret_2026` (**cambiar en producción**)
- **UUID**: `5393c7d5-0f7b-4482-a770-0e5ce00639c2`
- **grant_types**: `authorization_code`, `refresh_token`
- **scopes**: `authenticated_user_access`
- **redirect**: `exp://localhost:19000/--/oauth2redirect`
- **pkce**: false (PKCE optional — el cliente lo añade igualmente)
- **confidential**: true

### Scopes disponibles

| Scope ID | Descripción | Granularidad |
|---|---|---|
| `authenticated_user_access` | Acceso como usuario autenticado | Rol: authenticated |

### Notas de implementación

- El authorize endpoint redirige a `/user/login` si el usuario no está autenticado — esto es el comportamiento correcto en el flujo Authorization Code
- Para testing con curl, el `&` en la URL debe estar dentro de comillas simples: `curl -s 'https://shine.ddev.site/oauth/authorize?client_id=...&response_type=code'`
- `simple_oauth v6` no tiene Password Grant — requiere Authorization Code + PKCE
- Las claves RSA se encuentran en `/var/www/html/private/oauth-keys/` (dentro del contenedor DDEV)

---

## Headers globales

Todas las requests a JSON:API:
```
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json
Authorization: Bearer {access_token}
```

---

## Endpoints de lectura

### Actividades

```
GET /jsonapi/node/activity
GET /jsonapi/node/activity/{uuid}
GET /jsonapi/node/activity_blink
GET /jsonapi/node/activity_blink/{uuid}
GET /jsonapi/node/activity_image_position
GET /jsonapi/node/activity_image_position/{uuid}
```

**Parámetros de filtro útiles:**
```
?filter[field_activity_level.name]=Básico
?filter[status]=1
?include=field_activity_level,field_explanation
?sort=title
?page[limit]=10&page[offset]=0
```

**Include recomendado** (evita N+1 requests):
```
?include=field_activity_level,field_explanation.field_media
```

---

### Sesiones

```
GET /jsonapi/node/session
GET /jsonapi/node/session/{uuid}
```

**Con actividades incluidas:**
```
GET /jsonapi/node/session/{uuid}?include=field_activities,field_activities.field_activity_level
```

---

### Grupos de sesión

```
GET /jsonapi/node/group_session
GET /jsonapi/node/group_session/{uuid}?include=field_ref_session,field_ref_session.field_activities
```

---

### Mensajes de actividad

```
GET /jsonapi/activity_message/explanation_message/{uuid}?include=field_media
GET /jsonapi/activity_message/motivation_message
```

**Filtrar mensajes de motivación por tipo de actividad:**
```
GET /jsonapi/activity_message/motivation_message?filter[field_activity_types.type]=activity
```

> **Nota**: el typo `motiviation_message` fue corregido en Fase 0. El bundle correcto es `motivation_message`.

---

### Niveles de actividad (taxonomía)

```
GET /jsonapi/taxonomy_term/activity_level
GET /jsonapi/taxonomy_term/activity_level/{uuid}
```

---

## Endpoints de escritura

### Crear un activitylog (reemplaza POST /activitylog-register/add)

```
POST /jsonapi/activitylog/logverticaltext
```

**Body:**
```json
{
  "data": {
    "type": "activitylog--logverticaltext",
    "attributes": {
      "label": "A-{sessionId}-{activityId}-{iteration}-{weight}",
      "field_activity_type": "vertical_text",
      "field_item": "palabra",
      "field_weight": 1,
      "field_iteration": 3,
      "field_time": 1450
    },
    "relationships": {
      "field_activityid": {
        "data": { "type": "node--activity", "id": "{activity-uuid}" }
      },
      "field_sessionid": {
        "data": { "type": "node--session", "id": "{session-uuid}" }
      }
    }
  }
}
```

**Respuesta 201 Created:**
```json
{
  "data": {
    "type": "activitylog--logverticaltext",
    "id": "{nuevo-uuid}",
    ...
  }
}
```

---

### Crear un activitylog blink

```
POST /jsonapi/activitylog/log3_textlistblink
```

**Body:**
```json
{
  "data": {
    "type": "activitylog--log3_textlistblink",
    "attributes": {
      "label": "A-{sessionId}-{activityId}-{iteration}-{weight}",
      "field_activity_type": "activity_blink",
      "field_item": "ca",
      "field_weight": 1,
      "field_iteration": 1,
      "field_time": 1200
    },
    "relationships": {
      "field_activityid": {
        "data": { "type": "node--activity_blink", "id": "{activity-uuid}" }
      },
      "field_sessionid": {
        "data": { "type": "node--session", "id": "{session-uuid}" }
      }
    }
  }
}
```

---

### Crear un activitysummary blink

```
POST /jsonapi/activitysummary/summary3_vertical_text_blink
```

**Body:**
```json
{
  "data": {
    "type": "activitysummary--summary3_vertical_text_blink",
    "attributes": {
      "label": "Summary-{activityTitle}-intento {n}",
      "field_iteration": 1,
      "field_time": 12500
    },
    "relationships": {
      "field_activityid": {
        "data": { "type": "node--activity_blink", "id": "{activity-uuid}" }
      },
      "field_sessionid": {
        "data": { "type": "node--session", "id": "{session-uuid}" }
      },
      "field_activitylogs": {
        "data": [
          { "type": "activitylog--log3_textlistblink", "id": "{log-uuid-1}" }
        ]
      }
    }
  }
}
```

---

### Crear un activitysummary (vertical_list)

```
POST /jsonapi/activitysummary/summary_vertical_text
```

**Body:**
```json
{
  "data": {
    "type": "activitysummary--summary_vertical_text",
    "attributes": {
      "label": "Summary-{sessionId}-{activityId}-{iteration}",
      "field_activity_type": "vertical_text",
      "field_iteration": 3,
      "field_time": 12500
    },
    "relationships": {
      "field_activityid": {
        "data": { "type": "node--activity", "id": "{activity-uuid}" }
      },
      "field_sessionid": {
        "data": { "type": "node--session", "id": "{session-uuid}" }
      },
      "field_activitylogs": {
        "data": [
          { "type": "activitylog--logverticaltext", "id": "{log-uuid-1}" },
          { "type": "activitylog--logverticaltext", "id": "{log-uuid-2}" }
        ]
      }
    }
  }
}
```

---

### Consultar historial del usuario actual

```
GET /jsonapi/activitysummary/summary_vertical_text?filter[uid.id]={user-uuid}
GET /jsonapi/activitysummary/summary_vertical_text?filter[field_activityid.id]={activity-uuid}&filter[uid.id]={user-uuid}&sort=created
```

---

## Endpoint legacy (actual, acoplado)

Existe mientras se completa la migración a JSON:API:

```
POST /activitylog-register/add
Content-Type: application/x-www-form-urlencoded
(requiere sesión Drupal activa, no JWT)

activityId={nid}
activityType={bundle}
sessionId={nid}
items=[{"text":"palabra","weight":1}]   (JSON serializado)
times=[1200,980,1500]                   (JSON serializado)
weights=[1,2,3]                         (JSON serializado)
```

**Respuesta:** texto plano con mensaje de éxito.

> Este endpoint se eliminará en Fase 3 del roadmap.

---

## Endpoint de audio (futuro — Fase 7)

```
POST /jsonapi/activitylog/logaudioreading
```

```json
{
  "data": {
    "type": "activitylog--logaudioreading",
    "attributes": {
      "label": "Audio-{sessionId}-{activityId}-{iteration}",
      "field_activity_type": "audio_reading",
      "field_recorded_time": 8500,
      "field_iteration": 1
    },
    "relationships": {
      "field_activityid": {
        "data": { "type": "node--activity_audio_reading", "id": "{uuid}" }
      },
      "field_sessionid": {
        "data": { "type": "node--session", "id": "{uuid}" }
      }
    }
  }
}
```

---

## Configuración de CORS (Drupal)

En `sites/default/services.yml` (ya configurado en el proyecto):

```yaml
parameters:
  cors.config:
    enabled: true
    allowedHeaders: ['Content-Type', 'Authorization', 'Accept', 'X-CSRF-Token']
    allowedMethods: ['GET', 'POST', 'PATCH', 'DELETE', 'OPTIONS']
    allowedOrigins: ['*']
    exposedHeaders: false
    maxAge: 86400
    supportsCredentials: false
```

En producción, reemplazar `allowedOrigins: ['*']` por el dominio específico de la app.

---

## Errores comunes

| Código | Causa | Solución |
|---|---|---|
| 401 | Token expirado o inválido | Hacer refresh del token |
| 403 | Sin permisos para el recurso | Verificar permisos JSON:API en Drupal |
| 422 | Payload malformado | Verificar tipos y UUIDs en relationships |
| 404 | UUID no existe | Verificar que el recurso existe y está publicado |
