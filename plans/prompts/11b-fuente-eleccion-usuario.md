# 11b. App + backend: elección de fuente por el usuario (Nunito / OpenDyslexic)

- **Modelo recomendado: Opus.** Las decisiones de producto ya están tomadas (ver "Decisiones cerradas"); queda un refactor de tema con patrón claro, una pantalla mínima y un campo de usuario.
- Fase: 11 (design system en código). Alcance: app (`shine-app/`) + backend (`shine/`, un campo de usuario).
- Dependencias: 11a (pantallas ya migradas a tokens). Ejecutar ANTES de la FASE 12, para que las pantallas nuevas nazcan con la fuente dinámica.

## Decisiones cerradas (auditoría de producto, 2026-07-21)

- Es una **preferencia de confort, no una promesa terapéutica**: la evidencia sobre fuentes "para dislexia" es débil, pero la preferencia percibida y la agencia del niño sí aportan. El copy NUNCA dice que la fuente mejora la lectura ni menciona "dislexia" en la etiqueta.
- **Ajuste global de cuenta**, aplicado a TODA la app (no solo a los ejercicios), en una pantalla de Ajustes nueva. Nada de switch durante la ejecución de un ejercicio: distrae y mete ruido en la serie histórica de tiempos (las estadísticas comparan al usuario contra sí mismo).
- **Selector por vista previa**: dos tarjetas con la misma frase de ejemplo renderizada en cada fuente, pregunta "¿Cómo prefieres leer?". Se elige viendo, sin nombres técnicos ni etiquetas estigmatizantes.
- **Acceso rápido** desde la intro del ejercicio (pantalla "Comenzar" de ActivityBase): enlace discreto a ese mismo ajuste. El cambio ocurre entre intentos, nunca dentro de uno.
- **Persistencia en el servidor** (campo del usuario Drupal) con cache local, para que la elección viaje entre dispositivos.

## Prompt

Lee `plans/02-plan-ejecucion-v1.md` (FASE 11) y las decisiones cerradas de este fichero. Trabaja en `shine-app/` y, para el campo de usuario, en `shine/` con `ddev`.

Tareas backend:

1. Campo `field_dyslexic_font` (boolean, default false) en la entidad user, expuesto en JSON:API. Verificar que el usuario authenticated puede leer y modificar SU PROPIO usuario via `PATCH /jsonapi/user/user/{uuid}` (permiso de editar cuenta propia; no abrir nada más). Exportar config.

Tareas app:

2. **Fuente**: descargar OpenDyslexic (licencia SIL OFL, incluir el fichero de licencia) a `assets/fonts/` en los pesos disponibles (Regular, Bold, Italic) y cargarla con `expo-font` en el layout raíz junto a Nunito. No ejecutes `npx expo install`; si hace falta algún paquete, deja el comando para el usuario (`nvm use 21`).
3. **Mapeo de pesos**: OpenDyslexic solo tiene Regular y Bold. Mapear los 5 pesos del tema: regular/medium a Regular; semibold/bold/extrabold a Bold. Documentarlo en `src/theme/typography.ts`.
4. **Tema dinámico**: crear un store de ajustes (`src/store/settings.ts`, Zustand + persistencia en `@/lib/storage` + sync con el campo del usuario al hacer login y al cambiar). Exponer hooks `useFamilies()` y `useText()` en `@/theme` que devuelvan las familias/estilos según la preferencia, y migrar los componentes de `src/components/ui/` y las pantallas a los hooks (los usos estáticos de `families`/`text` quedan solo como base interna del tema).
5. **Pantalla de Ajustes** (`app/(auth)/settings.tsx`, acceso desde el dashboard con icono discreto, sin engordar el BottomNav): el selector por vista previa descrito arriba, y el botón de cerrar sesión (moverlo aquí si vive en otro sitio). Diseño con tokens y componentes ui.
6. **Enlace discreto en la intro** de ActivityBase ("Ajustar cómo se ven las letras" o similar) que abre Ajustes y vuelve.
7. **QA de layout**: OpenDyslexic tiene mayor x-height y anchura. Revisar con la fuente activa: botones, chips, StatTiles, BottomNav y la pantalla de ejercicio (`text.exercise` a 32px); corregir truncados o desbordes (numberOfLines, minWidth) sin bifurcar estilos por fuente salvo necesidad puntual.

Verificación:
- `npx tsc --noEmit` y `npm run lint` limpios.
- Cambiar la fuente en Ajustes cambia toda la app al instante y sobrevive a cerrar y reabrir (cache local) y a hacer login en otro navegador (sync servidor).
- Con OpenDyslexic activa, recorrido visual completo sin desbordes.
- En ningún texto de la UI aparece "dislexia" ni promesas de mejora.

Al terminar: marca el checkbox correspondiente de la FASE 11 en `plans/02-plan-ejecucion-v1.md` y actualiza `docs/progress.md`.
