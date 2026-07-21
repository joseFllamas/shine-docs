# 09a. Backend: cliente OAuth público, endpoint legacy y CORS

- **Modelo recomendado: Opus.** Cambios de configuración acotados con instrucciones claras; no hay diseño nuevo.
- Fase: 9 (saneamiento técnico). Alcance: backend (`shine/`).
- Dependencias: ninguna. Bugs de la auditoría: B1, B4, B5.

## Prompt

Lee `plans/02-plan-ejecucion-v1.md` (FASE 9, bloque Backend) y `plans/01-auditoria.md` (problemas B1, B4, B5). Trabaja en el backend Drupal (`shine/`, comandos con prefijo `ddev`).

Tareas:

1. **Convertir el consumer OAuth en cliente público** (B4). El consumer `shine_expo_app` tiene `confidential: true` y la app envía `client_secret`. Con PKCE el secreto es innecesario y en un bundle JS es público de facto:
   - Poner `confidential: false` en el consumer y vaciar el secret.
   - Verificar que el flujo Authorization Code + PKCE sigue funcionando SIN enviar `client_secret` (probar el POST a `/oauth/token` con curl; recuerda el gotcha de las comillas simples con `&` dentro de `ddev exec`).
   - Exportar la config con `ddev drush cex -y` si el cambio vive en config.
   - NO toques todavía el código de la app: eso es el prompt 09b. Documenta en `docs/progress.md` que el backend ya no exige secret.

2. **Endpoint legacy `/activitylog-register/add`** (B1): el módulo `activitylog_register` crea siempre bundles `logverticaltext`/`summary_vertical_text` ignorando el `activityType` recibido. Comprueba si el theme Drupal acoplado (módulo `preprocess` y plantillas del theme `shine`) sigue llamándolo. Si ya no se usa: desinstala el módulo `activitylog_register` y elimina su código. Si se usa: corrige el bundle hardcodeado mapeando `activityType` al bundle correcto. Decide con evidencia y deja la decisión anotada en `docs/progress.md`.

3. **CORS** (B5): en `services.yml` está `allowedOrigins: ['*']`. Déjalo así para desarrollo pero deja preparado y documentado (comentario en el propio fichero y nota en `docs/progress.md`) el valor de producción con los orígenes reales, para activarlo en la fase 14.

Verificación:
- Login completo desde la app (o con curl simulando el flujo) sin client_secret.
- `ddev drush watchdog:show --count=20 --severity=Error` sin errores nuevos.
- Config exportada y `ddev drush cim -y` idempotente.

Al terminar: marca los checkboxes correspondientes de la FASE 9 en `plans/02-plan-ejecucion-v1.md` y actualiza `docs/progress.md`.
