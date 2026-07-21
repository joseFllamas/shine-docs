# Prompts de ejecución del plan v1

Cada fichero es un prompt autocontenido para una sesión de Claude Code. Derivan de `plans/02-plan-ejecucion-v1.md` (fases 9 a 14) y de la auditoría `plans/01-auditoria.md`.

## Cómo usarlos

1. Abrir Claude Code en la raíz del proyecto con el modelo indicado.
2. Pegar el contenido de la sección "Prompt" del fichero (o pedir: "ejecuta plans/prompts/XX-....md").
3. Al terminar cada prompt: verificar según su sección "Verificación", marcar los checkboxes correspondientes en `plans/02-plan-ejecucion-v1.md` y actualizar `docs/progress.md`.

## Criterio de modelo

- **Fable**: tareas con decisiones de arquitectura o modelo de datos, lógica sutil de corrección (concurrencia, derivación de estado), criterio pedagógico o trabajo full-stack que cruza Drupal y la app.
- **Opus**: tareas acotadas con especificación clara, refactors mecánicos, implementación de pantallas con diseño ya validado y patrones ya establecidos.

## Orden de ejecución

| Orden | Fichero | Modelo | Depende de |
|---|---|---|---|
| 1 | `09a-backend-oauth-publico.md` | Opus | nada |
| 2 | `09b-app-config-entorno.md` | Opus | 09a |
| 3 | `09c-app-bugs-datos.md` | Fable | nada |
| 4 | `09d-app-errores-y-calidad.md` | Opus | 09c |
| 5 | `10a-backend-campos-summary.md` | Fable | 09a |
| 6 | `10b-app-stats-lib.md` | Fable | 10a, 09c |
| 7 | `11a-app-screen-y-migracion-tokens.md` | Opus | nada (en paralelo con 09/10) |
| 8 | `12a-dashboard.md` | Opus | 10b, 11a |
| 9 | `12b-sesion-detalle.md` | Opus | 10b, 11a |
| 10 | `12c-player.md` | Opus | 11a |
| 11 | `12d-actividad-completada.md` | Opus | 12c |
| 12 | `12e-mi-progreso.md` | Fable | 10b, 11a |
| 13 | `13a-ejercicio-matriz.md` | Fable | 12 completa |
| 14 | `13b-ejercicio-silabas.md` | Opus | 13a |
| 15 | `13c-ejercicio-segmentacion.md` | Opus | 13a |
| 16 | `14a-backend-produccion.md` | Fable | 13 completa |
| 17 | `14b-publicacion-stores.md` | Opus | 14a |

Hito de control tras el paso 12 (FASE 12 completa): demo con datos reales a 2 o 3 familias/terapeutas antes de invertir en los ejercicios nuevos.
