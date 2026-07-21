# 14b. Publicación: EAS Build, stores y web

- **Modelo recomendado: Opus.** Proceso guiado por la documentación de Expo/EAS; mucho es configuración y acompañamiento al usuario (los comandos EAS y las cuentas de stores son suyos).
- Fase: 14 (publicación). Alcance: app (`shine-app/`) + cuentas de stores del usuario.
- Dependencias: 14a (backend de producción funcionando).

## Prompt

Lee `plans/02-plan-ejecucion-v1.md` (FASE 14). Trabaja en `shine-app/`. Importante: los comandos `eas` y `npx expo` los ejecuta el usuario en su terminal (`nvm use 21`); tú preparas la configuración y le vas indicando cada comando y qué esperar.

Tareas:

1. **Identidad de la app** en `app.config.ts`: nombre definitivo, `bundleIdentifier` (iOS) y `package` (Android) acordados con el usuario, `scheme` definitivo para el redirect OAuth, versión 1.0.0.
2. **Iconos y splash** con la marca definitiva (círculo teal + sparkles del design system): genera los assets (icon 1024, adaptive icon Android, splash) coherentes con `src/theme`. Si hay logo fuente en el proyecto "Shine Design System" de claude.ai/design, úsalo de referencia.
3. **Variables de producción**: `EXPO_PUBLIC_API_BASE` y client id de producción via perfiles de EAS (`eas.json`), sin tocar los defaults de desarrollo.
4. **EAS Build**: `eas.json` con perfiles development, preview y production; guiar al usuario en `eas build --platform ios --profile preview` y el equivalente Android; probar las builds de preview en dispositivos reales (checklist de prueba: login OAuth con el scheme definitivo, los 6 tipos de ejercicio, Mi progreso).
5. **Redirect URIs nativas**: añadir al consumer OAuth de producción la URI del scheme definitivo.
6. **Stores**: guiar TestFlight (App Store Connect) y Google Play internal testing; textos de ficha mínimos (descripción, capturas desde las builds).
7. **Web**: `npx expo export --platform web` y despliegue estático (según el hosting decidido en 14a); verificar que el redirect URI web de producción está en el consumer.
8. **Piloto**: documento breve `docs/piloto.md` con el guion de feedback para las 3 a 5 familias (qué observar: comprensión de la pantalla de progreso, fricción en login, ejercicios).

Verificación:
- Build de preview instalada y flujo completo funcionando contra producción en iOS y Android.
- Web publicada con login funcional.
- Ficha de TestFlight/Play internal con testers invitados.

Al terminar: marca los checkboxes en `plans/02-plan-ejecucion-v1.md`, actualiza `docs/progress.md` y anota la fecha de inicio del piloto.
