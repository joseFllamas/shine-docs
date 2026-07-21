# 09d. App: manejo de errores, limpieza de código huérfano y base de calidad

- **Modelo recomendado: Opus.** Tareas acotadas: catch de errores, borrado de código muerto, configuración de tooling estándar.
- Fase: 9 (saneamiento técnico). Alcance: app (`shine-app/`).
- Dependencias: 09c (para no limpiar código que 09c acaba de tocar). Bugs: A7, A9.

## Prompt

Lee `plans/02-plan-ejecucion-v1.md` (FASE 9, bloque App) y `plans/01-auditoria.md` (A7, A9). Trabaja en `shine-app/`.

Tareas:

1. **Manejo de errores**:
   - Catch en la carga del detalle de sesión (`app/(auth)/session/[id].tsx`) con estado de error y botón reintentar (usa los componentes de `src/components/ui/` y los tokens de `src/theme/`).
   - `ActivityRouter` devuelve `null` con formatos desconocidos (pantalla en blanco): añadir fallback con mensaje amable "tipo de ejercicio no soportado" y botón de volver.
   - El login ya tiene estado de error (hecho el 2026-07-21); verifica que sigue cubierto.

2. **A7, código huérfano**: eliminar `SessionProgressBar`, `StandardComparison`, `getGroupSessions`, `getExplanationMessage` y las partes no usadas de `src/store/session.ts`. Antes de borrar cada uno, confirma con grep que no tiene usos, y contrasta con la FASE 12: si algo lo va a necesitar una pantalla nueva de forma inmediata y evidente, consérvalo y anótalo; en caso de duda, borra (git lo recuerda).

3. **A9, tooling**: añadir ESLint (config de Expo: `eslint-config-expo`) + Prettier + script `npm run lint`. Añadir Jest con `jest-expo` y un primer test trivial (por ejemplo de `src/lib/stripHtml.ts`) que sirva de andamiaje para los tests de stats de la fase 10. No instales los paquetes tú: deja los comandos `npm install ...` listados para que los ejecute el usuario con `nvm use 21`, y deja los ficheros de config creados.

Verificación:
- `npx tsc --noEmit` limpio.
- `npm run lint` y `npm test` funcionan (tras instalar el usuario las dependencias).
- Navegar a una actividad con formato desconocido muestra el fallback, no pantalla en blanco.

Al terminar: marca los checkboxes en `plans/02-plan-ejecucion-v1.md` y actualiza `docs/progress.md`.
