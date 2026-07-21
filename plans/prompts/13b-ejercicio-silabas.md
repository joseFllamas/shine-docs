# 13b. Ejercicio nuevo: reordenamiento de sílabas (7.8 del docx)

- **Modelo recomendado: Opus.** Sigue el patrón completo ya establecido y documentado en 13a; trabajo de replicación cuidadosa, no de diseño.
- Fase: 13 (nuevos ejercicios, prioridad 2). Alcance: backend + app.
- Dependencias: 13a (patrón establecido).

## Prompt

Replica el patrón del ejercicio de matriz (13a) para el ejercicio 7.8 del docx "ejercicios - tipo.docx": reordenar sílabas desordenadas para formar la palabra correcta (decodificación y memoria de trabajo; sin audio obligatorio). Lee primero cómo quedó implementado 13a (`git log` reciente, `docs/activity-system.md` actualizado) y la especificación 7.8 del docx.

Tareas (mismo checklist que 13a):

1. Backend: node type `activity_syllable_order` (campos: palabra objetivo y sus sílabas; decide si las sílabas se derivan de la palabra en el cliente o se editan explícitamente, y sigue lo que diga el docx), bundles `logsyllableorder` + `summary_syllable_order`, permisos, config export, 1 o 2 actividades de ejemplo.
2. Término de taxonomía `areas` que falte (conciencia fonológica/decodificación según docx).
3. App: tipos TS, `SyllableOrderPlayer` (fichas de sílaba arrastrables o por taps ordenados: elige taps si el drag complica la accesibilidad), rama en ActivityRouter, tracking con log por intento de palabra (`field_correct`) y summary con contadores.
4. Interacción amable con el fallo: recolocar sin penalizar visualmente; contar el acierto al conseguirlo.

Verificación: igual que 13a (jugable en sesión mixta, métricas en Mi progreso, config idempotente, tsc y tests limpios).

Al terminar: marca los checkboxes en `plans/02-plan-ejecucion-v1.md` y actualiza `docs/progress.md`.
