# Prompts de diseño: Shine

Prompts preparados para trabajar el diseño visual del aplicativo con Claude Design (o cualquier herramienta de generación de UI como Stitch).

## Cómo usarlos

1. Empezar SIEMPRE por `00-sistema-diseno.md`. Es el prompt maestro: define marca, paleta, tipografía y componentes compartidos. Genera primero el design system y guárdalo como referencia del proyecto.
2. Generar las pantallas en orden (01 a 07). Cada prompt es autocontenido: incluye el contexto de usuario, las restricciones de accesibilidad para dislexia y la descripción pantalla a pantalla.
3. Al pegar cada prompt de pantalla, añadir al principio: "Usa el design system de Shine ya definido en este proyecto".
4. Para variantes, pedir siempre: estado vacío, estado con datos y estado de error/carga.

## Orden recomendado

| Fichero | Pantalla | Prioridad v1 |
|---|---|---|
| 00-sistema-diseno.md | Design system (base de todo) | Obligatorio |
| 01-login.md | Login | Alta |
| 02-dashboard.md | Dashboard (home) | Alta |
| 03-sesion-detalle.md | Detalle de sesión | Alta |
| 04-activity-player.md | Player de actividades (intro, texto vertical, blink, audio) | Alta |
| 05-actividad-completada.md | Pantalla de celebración | Alta |
| 06-progreso-estadisticas.md | Mi progreso (estadísticas no desmoralizantes) | Alta |
| 07-ejercicios-nuevos.md | Players de los 10 ejercicios tipo del docx (spec) | Media (v1.1) |
| 07-fase-ejecucion/ | Mirror del diseño visual definitivo de la fase de ejecución (importado de claude.ai/design). Ver su README. | — |

## Referencias previas

- `docs/design.md` contiene la especificación original hecha para Google Stitch. Estos prompts la evolucionan.
- `docs/stitch/` contiene los mockups HTML/PNG ya generados con Stitch (login, dashboard, session-detail, activity-player, activity-complete). Sirven como punto de partida visual; se pueden adjuntar como imagen de referencia al prompt correspondiente.

## Principio rector del diseño de estadísticas

La evolución de la dislexia no es lineal. Ninguna pantalla debe mostrar datos que puedan leerse como "estás empeorando". Ver el detalle en `06-progreso-estadisticas.md`: se prioriza esfuerzo y constancia (siempre crecientes) sobre rendimiento puntual, y las gráficas de tiempo usan tendencia suavizada y mejores marcas personales, nunca el dato crudo aislado.
