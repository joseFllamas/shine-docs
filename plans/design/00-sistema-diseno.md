# Prompt 00: Design system de Shine

> Usar este prompt primero. Genera la base visual que heredan todas las pantallas.

---

Diseña el design system completo de **Shine**, una aplicación móvil (iOS, Android y web con la misma base de código Expo/React Native) para personas con dislexia, principalmente niños de 6 a 14 años acompañados de familias y terapeutas. La app propone ejercicios cortos de lectura y escritura organizados en sesiones, y muestra el progreso del usuario de forma motivadora.

## Personalidad de marca

Cálida, tranquila y alentadora. Nunca infantil en exceso ni clínica. El usuario tiene que sentir que la app está de su lado: cada pantalla celebra el esfuerzo y ninguna transmite juicio o fracaso. Referencias de tono: Headspace (calma), Duolingo (progreso visible) pero con mucho menos ruido visual.

## Restricciones de accesibilidad (innegociables)

- WCAG AA mínimo en todos los contrastes.
- Tipografía sans-serif redondeada apta para dislexia: **Nunito** como fuente base, con opción de usuario para cambiar a OpenDyslexic.
- Cuerpo de texto mínimo 18sp, nada por debajo de 14sp. Interlineado 1.6 o superior. Letter-spacing ligeramente abierto. Texto nunca justificado.
- Áreas táctiles mínimas de 48x48 px.
- Ninguna información comunicada solo por color: acompañar siempre con icono, forma o etiqueta.
- Sin sonido ni animación automática sin acción del usuario. Respetar prefers-reduced-motion.
- Una sola acción primaria por pantalla.

## Paleta

| Rol | Color | Uso |
|---|---|---|
| Primario | Teal cálido `#3DBFA0` | CTA principal, estados activos |
| Primario oscuro | `#2A8C74` | Estados pressed, texto sobre fondos claros |
| Acento | Ámbar cálido `#F5A623` | Barras de progreso, rachas, destacados |
| Fondo | Blanco roto `#F7F5F2` | Fondo de pantalla, nunca blanco puro |
| Superficie | Blanco `#FFFFFF` | Tarjetas y modales |
| Texto primario | Carbón `#1E1E2E` | Contenido principal, nunca negro puro |
| Texto secundario | Gris medio `#6B7280` | Etiquetas y captions |
| Éxito | Verde suave `#4CAF7D` | Completados, celebraciones |
| Error | Rojo cálido `#E05A5A` | Solo errores de sistema, nunca para resultados del usuario |

Importante: el rojo no se usa jamás para señalar respuestas incorrectas en los ejercicios. Un fallo del usuario se marca en gris neutro o ámbar suave con mensaje amable.

## Tipografía

- H1: 28sp Bold, letter-spacing 0.3px
- H2: 22sp SemiBold
- Cuerpo: 18sp Regular, line-height 1.7
- Etiqueta: 14sp Medium, gris secundario
- Texto de ejercicio (palabras y frases durante la actividad): 24 a 36sp, Bold o SemiBold, centrado, alto contraste

## Componentes a definir

1. **Botón**: primario (relleno teal, texto blanco, alto 48px, radio 12px, ancho completo en móvil), secundario (borde teal, fondo blanco), destructivo (solo para cerrar sesión). Estados: normal, pressed (escala 0.97), disabled, loading.
2. **Tarjeta**: fondo blanco, radio 16px, sombra suave `0 2px 8px rgba(0,0,0,0.08)`, padding 16px, estado pressed con escala 0.98.
3. **Chip de nivel**: píldora. Inicial (verde claro/verde oscuro), Intermedio (ámbar claro/ámbar oscuro), Avanzado (morado claro/morado oscuro).
4. **Chip de área**: píldora neutra con icono, para las áreas de trabajo (conciencia fonológica, discriminación visual, memoria de trabajo, ortografía, fluidez lectora).
5. **Barra de progreso**: fondo `#E5E7EB`, relleno ámbar, alto 12px, extremos redondeados, siempre con etiqueta de texto (por ejemplo "7 de 10 actividades").
6. **Tarjeta de mensaje motivacional**: borde izquierdo teal de 4px, fondo teal muy claro, icono de estrella o corazón, texto en cuerpo.
7. **Stat tile**: tarjeta pequeña con número grande (32sp Bold), etiqueta debajo y un icono. Para el dashboard y estadísticas.
8. **Estado vacío**: ilustración simple y amable (libros, lápices, bombillas, estrellas) más un mensaje corto alentador. Nunca una pantalla en blanco.
9. **Navegación**: barra inferior con solo dos elementos (Inicio, Mi progreso). Sin menús hamburguesa. Navegación push con flecha atrás arriba a la izquierda. El player de actividades oculta toda la navegación (modo foco a pantalla completa).

## Adaptación web

Mobile-first. En navegador: layout limitado a 480px de ancho máximo, centrado, con fondo exterior neutro `#EEECE9`. La barra inferior pasa a ser cabecera superior con logo a la izquierda. Nada más cambia.

## Entregable

Genera: hoja de estilos con tokens (colores, tipografía, espaciado en escala de 4px, radios, sombras), y una lámina con todos los componentes en sus estados. Modo claro solamente en v1.
