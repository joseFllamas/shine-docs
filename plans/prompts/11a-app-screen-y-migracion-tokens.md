# 11a. App: componente Screen y migración de pantallas a los tokens del tema

- **Modelo recomendado: Opus.** Refactor visual mecánico contra un design system ya portado; el criterio visual ya está decidido en el proyecto de Claude Design.
- Fase: 11 (design system en código). Alcance: app (`shine-app/`).
- Dependencias: ninguna (paralelizable con la fase 9/10, evitando tocar los mismos ficheros a la vez).

## Prompt

Lee `plans/02-plan-ejecucion-v1.md` (FASE 11) y `plans/01-auditoria.md` (A2). El design system ya está portado: tokens en `src/theme/` y componentes en `src/components/ui/` (fuente de verdad visual: proyecto "Shine Design System" en claude.ai/design; la pantalla de login ya está migrada y sirve de referencia de estilo). Trabaja en `shine-app/`.

Tareas:

1. **Componente `Screen`** en `src/components/ui/`: layout base que en web limita el ancho a `layout.appMaxWidth` (480) centrado sobre fondo `colors.shell`, y en móvil ocupa todo con `colors.bgScreen`; gestiona safe areas y el padding de gutter (`layout.screenGutter`). Exportarlo en `index.ts`.

2. **Migrar TODAS las pantallas existentes a los tokens**, eliminando cada hex duplicado y cada fontWeight/fontSize suelto: `app/(auth)/dashboard.tsx`, `app/(auth)/session/[id].tsx`, `app/(auth)/statistics/index.tsx`, y los componentes de `src/components/activities/` y `src/components/statistics/` que sigan vivos. Usar `colors`, `text`, `spacing`, `radii`, `shadows` de `@/theme` y los componentes de `@/components/ui` donde encajen (Button, Card, chips, ProgressBar, EmptyState...). Esta tarea es de sustitución visual: NO rediseñes las pantallas ni cambies su lógica (el rediseño funcional es la FASE 12); si el mapeo azul antiguo a teal cambia jerarquías raras, aplica el equivalente semántico más cercano.

3. La paleta azul de facto (`#1a5276`, `#E6F4FE`, etc.) desaparece por completo.

Verificación:
- `grep -rn "#[0-9A-Fa-f]\{3,6\}" shine-app/app shine-app/src --include="*.tsx" | grep -v src/theme | grep -v src/components/ui` no devuelve colores hardcodeados (los SVG de Icon.tsx y valores como 'transparent' son aceptables).
- `npx tsc --noEmit` limpio.
- Recorrido visual en web (usuario): todas las pantallas coherentes con el login ya migrado; en escritorio el contenido queda centrado a 480px.

Al terminar: marca los checkboxes en `plans/02-plan-ejecucion-v1.md` y actualiza `docs/progress.md`.
