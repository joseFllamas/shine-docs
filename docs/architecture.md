# Arquitectura del proyecto Shine

## Contexto y decisión

El proyecto parte de un Drupal 10 acoplado. Se ha decidido evolucionar a una arquitectura **headless** para:
- Soportar publicación como app nativa en iOS y Android (requisito por el uso del micrófono y distribución en stores)
- Separar las responsabilidades de contenido (Drupal) y experiencia de usuario (frontend nativo)
- Permitir escalar los tipos de actividad y métricas sin acoplar lógica de presentación al CMS

---

## Stack tecnológico

### Backend — Drupal 11 (Headless CMS)

| Componente | Tecnología | Justificación |
|---|---|---|
| CMS | Drupal 11 | Ya existe, entity system maduro, JSON:API nativo, multilingüe |
| API | JSON:API (core module) | Estándar REST con soporte nativo en Drupal, filtros potentes, sin código extra |
| Autenticación | `drupal/simple_oauth` | OAuth 2.0 / JWT, estándar para apps móviles |
| CORS | Config nativa D11 (`services.yml`) | Drupal 11 soporta CORS sin módulo adicional |
| Base de datos | MariaDB 10.11 | Sin cambios |
| Entorno dev | DDEV | Sin cambios |

**Por qué mantener Drupal y no cambiarlo:**
- El modelo de entidades ya está definido y validado
- JSON:API está incluido en el core (cero código extra para exponer entidades)
- El sistema de permisos por roles encaja con el modelo de usuario del proyecto
- Soporte nativo para multilingüe (relevante para accesibilidad)
- La migración a D11 desde D10 es incremental, no rupturista

---

### Frontend — Expo (React Native)

| Componente | Tecnología | Justificación |
|---|---|---|
| Framework | Expo SDK 51+ (React Native) | Un solo codebase para iOS, Android y Web |
| Routing | Expo Router (file-based) | Similar a Next.js App Router, estructura familiar |
| UI | React Native + NativeWind (Tailwind para RN) | Estilos familiares, accesibilidad buena |
| Gráficas | Victory Native | Compatible con React Native, accesible, responsive |
| Audio/Micrófono | `expo-av` + `expo-speech` | Acceso nativo fiable en iOS y Android |
| Build / Stores | EAS Build (Expo Application Services) | Genera `.ipa` y `.apk` listos para stores |
| API client | `@drupal/drupal-jsonapi-params` + fetch | Queries JSON:API tipadas |
| Estado global | Zustand o React Context | Ligero, suficiente para este dominio |

**Por qué Expo y no Next.js:**
- Next.js es web únicamente. Publicar en App Store de Apple requiere una app nativa; Apple no acepta PWAs en su store.
- El micrófono en iOS vía PWA (Safari) es inestable. `expo-av` da acceso nativo fiable en ambas plataformas.
- Expo Router tiene file-based routing idéntico en concepto a Next.js App Router: la curva de aprendizaje es mínima para alguien que conoce React.
- Un solo codebase cubre iOS, Android y web (Expo web).

**Por qué Expo y no Flutter:**
- Flutter usa Dart (nuevo lenguaje). El equipo ya tiene base JS/React.
- La integración con JSON:API de Drupal es más directa desde el ecosistema JS.

**Por qué Expo y no React Native puro:**
- Expo gestiona la configuración nativa (Xcode, Gradle) de forma transparente.
- EAS Build permite compilar para stores sin necesidad de Mac para iOS.

---

## Diagrama de arquitectura

```
┌─────────────────────────────────────────────────────────────┐
│                        STORES / WEB                          │
│   App Store (iOS)    Google Play (Android)    Web (PWA)     │
└──────────────┬──────────────┬───────────────────┬───────────┘
               │              │                   │
               └──────────────┴───────────────────┘
                              │
                    ┌─────────▼──────────┐
                    │   EXPO (React Native)│
                    │   iOS / Android / Web│
                    │                      │
                    │  ┌────────────────┐  │
                    │  │ Activity Player │  │
                    │  │ Statistics      │  │
                    │  │ Audio Reader    │  │
                    │  │ Auth (OAuth)    │  │
                    │  └────────────────┘  │
                    └─────────┬────────────┘
                              │ JSON:API + JWT (HTTPS)
                    ┌─────────▼────────────┐
                    │   DRUPAL 11 Headless  │
                    │                       │
                    │  JSON:API module      │
                    │  simple_oauth         │
                    │  Custom modules:      │
                    │  - activitylog        │
                    │  - activitysummary    │
                    │  - activity_message   │
                    └─────────┬────────────┘
                              │
                    ┌─────────▼────────────┐
                    │     MariaDB 10.11     │
                    └───────────────────────┘
```

---

## Principio de extensibilidad

Añadir un nuevo tipo de actividad implica:

```
Backend (Drupal)                    Frontend (Expo)
─────────────────────               ─────────────────────────
node.type.{tipo}                    src/components/activities/
  + campos específicos                {TipoPlayer}.tsx
  + bundle en activitylog             (extiende ActivityBase)
  + config export
```

Cada capa es independiente. El frontend no sabe qué tipos existen: los descubre desde la API (`field_activity_format` o el bundle del nodo).

---

## Estrategia de despliegue

| Entorno | Backend | Frontend |
|---|---|---|
| Desarrollo | DDEV (`ddev start`) | `npx expo start` |
| Preview | DDEV tunnel / ngrok | Expo Go app (QR code) |
| Producción iOS | Servidor PHP/Nginx | EAS Build → App Store |
| Producción Android | Servidor PHP/Nginx | EAS Build → Google Play |
| Producción Web | Servidor PHP/Nginx | EAS Build → Expo web (estático) |

---

## Decisiones pendientes de confirmar

- [ ] Nombre del paquete de la app (`com.oriose.shine` o similar)
- [ ] Si el usuario anónimo puede hacer actividades (sin login) o siempre requiere cuenta
- [ ] Idiomas iniciales a soportar (es, ca, en?)
- [ ] Si las actividades funcionan offline (requiere caché de contenido en el dispositivo)
