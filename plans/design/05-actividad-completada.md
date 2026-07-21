# Prompt 05: Pantalla de actividad completada (celebración)

> Usa el design system de Shine ya definido en este proyecto (ver 00-sistema-diseno.md).

---

Diseña la pantalla de **actividad completada** de Shine, app de ejercicios para personas con dislexia. Aparece siempre al terminar cualquier ejercicio y es el momento emocional más importante de la app: refuerzo positivo puro.

## Layout (pantalla completa, sin navegación)

1. Confeti sutil de 3 segundos (respeta reduced motion: en ese caso, solo el check animado).
2. Icono de check grande en verde suave, animación scale-in con rebote (300ms spring).
3. "¡Actividad completada!" en H1 centrado.
4. Dato del intento en positivo, una sola línea: "Has practicado 42 segundos". Importante: se presenta como tiempo de práctica (esfuerzo), no como marca a batir.
5. Si el usuario ha mejorado su mejor marca personal: chip ámbar con estrella "¡Tu mejor marca!". Si no ha mejorado, NO se muestra ninguna comparación. Nunca "has sido más lento que la última vez".
6. Tarjeta de mensaje motivacional (aleatorio, del backend): borde teal, icono de corazón o estrella, texto tipo "¡Lo estás haciendo genial! Sigue así".
7. Botones:
   - Primario: "Siguiente actividad" (si quedan actividades en la sesión).
   - Secundario (borde): "Volver a la sesión".
   - Si era la última: primario "Terminar sesión" que lleva a una variante con celebración de sesión completa.

## Variante: sesión completada

Cuando la actividad completada era la última de la sesión:
- Confeti algo más generoso.
- "¡Has completado la sesión [nombre]!" en H1.
- Resumen amable: "Hoy: 8 actividades, 12 minutos de práctica".
- Botón primario "Volver al inicio".

## Estados

- Completada normal (con y sin mejor marca personal).
- Sesión completa.
- Guardado fallido: mostrar la celebración igualmente con nota discreta "Guardaremos tu progreso en cuanto haya conexión". La celebración nunca se sacrifica por un error técnico.

## Referencia previa

docs/stitch/activity-complete.png (evolucionar: añadir mejor marca personal y variante de sesión completa).
