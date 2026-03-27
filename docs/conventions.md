# Convenciones de desarrollo — Shine

Este documento establece los estándares que deben seguirse en todos los desarrollos del proyecto para garantizar consistencia.

---

## 1. Convenciones Drupal / PHP

### Módulos

- **Prefijo**: todos los módulos custom usan prefijo `shine_` para los nuevos. Los existentes (activitylog, activitysummary, activity_message, activitylog_register, preprocess) mantienen su nombre por compatibilidad con la base de datos.
- **Nombrado**: `snake_case`
- **`.info.yml`** obligatorio:
  ```yaml
  name: 'Shine Mi Módulo'
  type: module
  description: 'Descripción real, no @todo.'
  package: shine
  core_version_requirement: ^11
  ```

### Entidades custom

| Elemento | Convención | Ejemplo |
|---|---|---|
| Tipo de entidad (machine name) | `snake_case` | `activitylog` |
| Bundle (machine name) | `snake_case` | `logverticaltext` |
| Clase PHP | `PascalCase` | `ActivityLog` |
| Clase bundle PHP | `PascalCaseType` | `ActivityLogType` |
| Tabla base de datos | igual al tipo | `activitylog` |

### Campos

- **Siempre** con prefijo `field_`
- `snake_case`
- Nombres descriptivos, no abreviaturas: `field_activity_level`, no `field_act_lvl`
- Los campos reutilizados entre tipos de contenido comparten el mismo `field.storage`: si `field_activity_level` ya existe, se reutiliza en el nuevo content type, no se crea uno nuevo

### Permisos

El patrón estándar para entidades custom es:
```
view {entity_type}
create {entity_type}
edit {entity_type}
delete {entity_type}
administer {entity_type} types
```

### Configuración

- **Obligatorio** hacer `drush cex` después de cualquier cambio de configuración en el sitio antes de hacer commit
- Los archivos `.yml` exportados son la fuente de verdad, no la base de datos
- Nunca editar manualmente un archivo `.yml` de config sin aplicarlo después con `drush cim`

### Nomenclatura de tipos de actividad (crítico para escalabilidad)

Cuando se crea un nuevo tipo de actividad, todos los elementos deben seguir el mismo patrón de nombre:

| Elemento | Patrón | Ejemplo: lectura de audio |
|---|---|---|
| Node type (Drupal) | `activity_{tipo}` | `activity_audio_reading` |
| Activitylog bundle | `log{tipo}` | `logaudioreading` |
| Activitysummary bundle | `summary_{tipo}` | `summary_audio_reading` |
| React Native component | `{Tipo}Player` | `AudioReadingPlayer` |
| TypeScript type | `Activity{Tipo}` | `ActivityAudioReading` |

---

## 2. Convenciones JavaScript (código acoplado Drupal)

> Aplica al código en `activitylog_register/js/` mientras se mantiene la versión acoplada.

- **Patrón de actividades**: clase base `Activity` con subclases por tipo
- Métodos obligatorios en cada subclase: `prepareActivity()`, `playActivityItem()`, `addRegisterLog()`
- Inicialización via `Drupal.behaviors.{moduleName}`, nunca en DOMContentLoaded
- **No usar jQuery AJAX** en código nuevo — usar `fetch` API
- Los datos del log siempre se envían juntos: `activityId`, `sessionId`, `activityType`, `items[]`, `times[]`, `weights[]`
- Registrar el tipo de actividad en `activitylog_register.module` vía `hook_preprocess_node()`

---

## 3. Convenciones React Native / Expo

### Estructura de carpetas

```
shine-app/
├── app/                        # Expo Router (file-based routing)
│   ├── (auth)/                 # Rutas protegidas
│   │   ├── dashboard.tsx
│   │   ├── session/[id].tsx
│   │   └── activity/[id].tsx
│   ├── login.tsx
│   └── _layout.tsx
├── src/
│   ├── components/
│   │   ├── activities/         # Un archivo por tipo de actividad
│   │   │   ├── ActivityBase.tsx
│   │   │   ├── VerticalTextPlayer.tsx
│   │   │   ├── ImagePositionPlayer.tsx
│   │   │   └── AudioReadingPlayer.tsx
│   │   ├── statistics/         # Componentes de gráficas
│   │   │   ├── ActivityEvolutionChart.tsx
│   │   │   ├── StandardComparison.tsx
│   │   │   └── SessionProgressBar.tsx
│   │   └── ui/                 # Componentes reutilizables
│   ├── hooks/                  # Custom hooks (prefijo `use`)
│   │   ├── useActivityLog.ts
│   │   └── useSession.ts
│   ├── lib/
│   │   └── api/
│   │       ├── client.ts       # Fetch + JWT interceptor
│   │       ├── activities.ts   # Queries de actividades
│   │       ├── sessions.ts     # Queries de sesiones
│   │       └── tracking.ts     # Crear activitylog/summary
│   ├── store/                  # Zustand stores
│   │   ├── auth.ts
│   │   └── session.ts
│   └── types/                  # TypeScript interfaces (mirroring Drupal)
│       ├── activity.ts
│       ├── session.ts
│       ├── activitylog.ts
│       └── activitysummary.ts
```

### Nombrado

| Elemento | Convención | Ejemplo |
|---|---|---|
| Componentes | `PascalCase` | `VerticalTextPlayer` |
| Hooks | `use` + PascalCase | `useActivityLog` |
| Stores Zustand | `use` + PascalCase + `Store` | `useAuthStore` |
| Funciones de API | camelCase descriptivo | `fetchActivitiesBySession` |
| Archivos de componente | `PascalCase.tsx` | `VerticalTextPlayer.tsx` |
| Archivos de hook/util | `camelCase.ts` | `useActivityLog.ts` |

### Tipos TypeScript

Cada entidad Drupal tiene su tipo TypeScript correspondiente en `src/types/`. Los campos JSON:API usan el prefijo `field_` igual que en Drupal:

```typescript
// src/types/activity.ts
export interface ActivityNode {
  id: string;
  attributes: {
    title: string;
    field_activity_format: string;
    field_activity_code: string | null;
    field_seconds: number | null;
    field_text: string | null;
  };
  relationships: {
    field_activity_level: { data: { id: string } };
    field_explanation: { data: { id: string } | null };
  };
}
```

### Patrón de componentes de actividad

Todos los players de actividad siguen la misma estructura:

```typescript
interface ActivityPlayerProps {
  activity: ActivityNode;
  sessionId: string;
  onComplete: (logData: ActivityLogData) => void;
}

// Ciclo de vida obligatorio:
// 1. Render intro (explanation + motivational message)
// 2. Usuario pulsa "Comenzar"
// 3. Timer inicia
// 4. Lógica específica del tipo
// 5. Al completar: llamar onComplete() con los datos
// 6. El padre se encarga de enviar el log a la API
```

---

## 4. Convenciones de API (JSON:API)

- Los headers de toda request autenticada: `Authorization: Bearer {token}`, `Content-Type: application/vnd.api+json`
- Incluir siempre `include` para evitar N+1 requests: `?include=field_activities,field_explanation`
- Filtros siguiendo el spec JSON:API: `?filter[field_activity_level.name]=Básico`
- Ver `docs/api-design.md` para los endpoints concretos

---

## 5. Convenciones de Git

- **Idioma de commits**: inglés
- **Prefijos obligatorios**:
  - `feat:` nueva funcionalidad
  - `fix:` corrección de bug
  - `refactor:` refactorización sin cambio funcional
  - `docs:` cambios solo en documentación
  - `chore:` configuración, dependencias, tareas de mantenimiento
  - `config:` cambios de configuración Drupal (config export)
- **Ejemplos correctos**:
  - `feat: add AudioReadingPlayer component`
  - `config: add field_standard_time to node.activity`
  - `fix: correct iteration counter in activitylog controller`
- **Ramas**: `feature/{nombre}`, `fix/{nombre}`, `release/{versión}`
