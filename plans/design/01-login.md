# Prompt 01: Pantalla de Login

> Usa el design system de Shine ya definido en este proyecto (ver 00-sistema-diseno.md).

---

Diseña la pantalla de **login** de Shine, app para personas con dislexia (Expo/React Native, mobile-first, versión web limitada a 480px centrados).

## Contexto funcional

La autenticación es OAuth2 con PKCE contra un backend Drupal: al pulsar el botón se abre el navegador del sistema con la página de login de Drupal y se vuelve a la app. Por tanto esta pantalla NO tiene campos de usuario y contraseña, solo un CTA que lanza el flujo.

## Layout

Pantalla completa, contenido centrado verticalmente, sin navegación ni footer. Totalmente libre de distracciones.

1. Logo o wordmark de Shine, grande, centrado en el tercio superior. Si no hay logo definido, proponer un wordmark amable con un detalle de brillo o estrella (shine = brillar).
2. Tagline corta debajo, en cuerpo 18sp, tono alentador: "Practicamos juntos" o similar.
3. Botón primario grande y ancho: "Entrar". Único CTA de la pantalla.
4. Debajo del botón, texto secundario pequeño (14sp) explicando que se abrirá el navegador para iniciar sesión de forma segura.

## Estados

- Normal.
- Cargando: el botón muestra spinner y texto "Abriendo sesión segura".
- Error: banner suave (no rojo agresivo) sobre el botón: "No hemos podido conectar. Vuelve a intentarlo." con botón de reintento.

## Notas

- Fondo blanco roto `#F7F5F2`, sin imágenes de fondo cargadas de detalle.
- Tipografía Nunito. Nada por debajo de 14sp.
- Referencia visual previa: docs/stitch/login.png (evolucionar, no copiar).
