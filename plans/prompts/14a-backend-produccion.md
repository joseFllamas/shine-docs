# 14a. Backend de producción

- **Modelo recomendado: Fable.** Seguridad y despliegue: claves, CORS, settings de producción. Los errores aquí son incidentes, no bugs.
- Fase: 14 (publicación). Alcance: backend + infraestructura.
- Dependencias: FASE 13 completa. Requiere decisiones del usuario: dominio y hosting.

## Prompt

Lee `plans/02-plan-ejecucion-v1.md` (FASE 14) y `plans/01-auditoria.md` (B5). Antes de tocar nada, pregunta al usuario qué dominio y hosting de producción se van a usar (esto no se puede inferir).

Tareas:

1. **Checklist de despliegue** como documento `docs/deploy.md`: requisitos del servidor (PHP 8.3, MariaDB, HTTPS), proceso de deploy (git + composer install --no-dev + drush deploy), y el detalle de cada punto siguiente.
2. **Claves RSA nuevas de producción**: procedimiento documentado (nunca reutilizar las de desarrollo, nunca en git), rutas en `private/` fuera del web root.
3. **Consumer OAuth de producción**: verificar que sigue siendo cliente público sin secret (09a); redirect URIs SOLO las de producción (scheme nativo de la app + dominio web real); eliminar las URIs de desarrollo en el entorno de producción.
4. **CORS**: `allowedOrigins` restringido al dominio web real (valor preparado en 09a).
5. **settings.php de producción**: `settings.local.php` de prod con credenciales, `trusted_host_patterns`, caches de producción, agregación CSS/JS, y `config_exclude_modules` para módulos de desarrollo si los hay.
6. **Revisión de seguridad rápida**: `ddev composer audit` (y su equivalente en prod), estado de actualizaciones de core/contrib, permisos de ficheros de `private/`.
7. Si el hosting ya está accesible: ejecutar el deploy y verificar. Si no: dejar todo preparado y el checklist listo para ejecutarlo a mano.

Verificación (cuando exista el entorno):
- `https://DOMINIO/jsonapi` responde con SSL válido; el flujo OAuth completo funciona desde una build de la app apuntando a producción; CORS bloquea orígenes ajenos.
- Ningún secreto ni clave en git (`git log -p` de los ficheros tocados).

Al terminar: marca los checkboxes en `plans/02-plan-ejecucion-v1.md` y actualiza `docs/progress.md`.
