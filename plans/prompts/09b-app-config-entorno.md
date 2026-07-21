# 09b. App: configuración por entorno (sin secretos ni dominios hardcodeados)

- **Modelo recomendado: Opus.** Refactor mecánico con un patrón estándar de Expo.
- Fase: 9 (saneamiento técnico). Alcance: app (`shine-app/`).
- Dependencias: 09a (el backend ya no exige client_secret). Bugs: A3, B4.

## Prompt

Lee `plans/02-plan-ejecucion-v1.md` (FASE 9, bloque App) y `plans/01-auditoria.md` (A3). Trabaja en `shine-app/`.

Contexto: `API_BASE` (`https://shine.ddev.site`), `clientId` y `clientSecret` están hardcodeados en al menos 3 ficheros (`app/login.tsx`, `src/lib/api/client.ts` y otros; búscalos todos con grep).

Tareas:

1. Crear `app.config.ts` (migrando lo necesario de `app.json`) que lea variables `EXPO_PUBLIC_API_BASE` y `EXPO_PUBLIC_OAUTH_CLIENT_ID` con defaults de desarrollo, y las exponga en `extra`.
2. Crear un módulo único `src/lib/config.ts` que lea de `expo-constants` y exporte `API_BASE`, `OAUTH_CLIENT_ID` y los endpoints OAuth derivados. Un solo punto de verdad.
3. Sustituir TODOS los usos hardcodeados por imports de ese módulo.
4. **Eliminar `client_secret` por completo de la app** (el backend ya es cliente público con PKCE, ver 09a). Quitarlo del authRequest y del intercambio de token en `app/login.tsx`.
5. Crear `.env.example` con las variables y añadir `.env.local` al `.gitignore` si no está.
6. Actualizar la tabla de "archivos que no están en git" de `CLAUDE.md` si aplica.

Restricciones:
- No ejecutes `npx expo install` (falla desde Claude Code por la versión de Node del sistema); si hace falta instalar algo, deja el comando indicado para que lo ejecute el usuario con `nvm use 21`.
- No rompas el flujo OAuth actual: la redirect URI y el scope no cambian.

Verificación:
- `npx tsc --noEmit` limpio.
- `grep -r "shine.ddev.site\|shine_dev_secret\|shine_expo_app" shine-app/src shine-app/app` solo debe encontrar el módulo de config (y ningún secret en ninguna parte).
- Login funcional en web (`npm run web`, lo prueba el usuario).

Al terminar: marca los checkboxes en `plans/02-plan-ejecucion-v1.md` y actualiza `docs/progress.md`.
