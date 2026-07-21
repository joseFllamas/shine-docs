# 13c. Ejercicio nuevo: segmentación léxica (7.5 del docx)

- **Modelo recomendado: Opus.** Tercera repetición del patrón de 13a; el contenido (frases sin espacios) es facilísimo de modelar.
- Fase: 13 (nuevos ejercicios, prioridad 3). Alcance: backend + app.
- Dependencias: 13a (patrón establecido). Paralelizable con 13b.

## Prompt

Replica el patrón del ejercicio de matriz (13a) para el ejercicio 7.5 del docx "ejercicios - tipo.docx": una frase escrita sin espacios que el usuario debe separar en palabras (conciencia léxica; sin audio). Lee cómo quedó 13a (`docs/activity-system.md` actualizado) y la especificación 7.5 del docx.

Tareas (mismo checklist que 13a):

1. Backend: node type `activity_word_split` (contenido: la frase correcta con espacios; la versión sin espacios se deriva en cliente), bundles `logwordsplit` + `summary_word_split`, permisos, config export, 2 o 3 actividades de ejemplo (el contenido es trivial de crear).
2. Término de taxonomía `areas` "conciencia léxica" si no existe.
3. App: tipos TS, `WordSplitPlayer` (la frase sin espacios; tap entre letras inserta/quita un corte; comprobación contra la segmentación correcta), rama en ActivityRouter, tracking (un log por frase con `field_correct`, summary con contadores).
4. Tipografía del ejercicio con `text.exercise` (mínimo 24) y zonas de tap generosas entre caracteres.

Verificación: igual que 13a (jugable en sesión mixta, métricas en Mi progreso con aciertos, config idempotente, tsc y tests limpios).

Al terminar: marca los checkboxes en `plans/02-plan-ejecucion-v1.md` y actualiza `docs/progress.md`. Con este ejercicio la FASE 13 queda completa: anótalo.
