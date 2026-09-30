# termixCustom — Upstream Synchronization Guide

## 1. Objetivo

Este documento define cómo incorporar cambios del proyecto oficial Termix dentro de `termixCustom` sin perder nuestras personalizaciones.

Repositorio personalizado:

```text
origin
https://github.com/arturinho16/termixCustom.git
```

Repositorio oficial:

```text
upstream
https://github.com/Termix-SSH/Termix.git
```

---

# 2. Remotos

La configuración correcta debe ser:

```bash
git remote -v
```

Resultado esperado:

```text
origin    https://github.com/arturinho16/termixCustom.git
upstream  https://github.com/Termix-SSH/Termix.git
```

Regla:

```text
origin
```

es nuestro repositorio.

```text
upstream
```

es exclusivamente Termix oficial.

Nunca realizar:

```bash
git push upstream
```

---

# 3. Ramas principales

Usaremos:

```text
main
develop
```

`main`:

```text
versión estable/integrada de termixCustom
```

`develop`:

```text
rama donde se integran nuevas funciones
```

Nunca desarrollar directamente en `main`.

---

# 4. Feature branches

Cada función debe desarrollarse desde `develop`.

Ejemplo Browser:

```bash
git switch develop
git pull origin develop
git switch -c feature/advanced-browser
```

Ejemplo Parsec:

```bash
git switch develop
git pull origin develop
git switch -c feature/parsec-remote
```

Ejemplo corrección:

```bash
git switch -c fix/browser-certificates
```

---

# 5. Flujo normal

Arquitectura Git:

```text
                    Termix official
                    upstream/dev-2.9.0
                           │
                           │
                           ▼
                     upstream-sync
                           │
                           ▼
                        develop
                      /         \
                     /           \
        feature/browser    feature/parsec
                     \           /
                      \         /
                        develop
                           │
                           ▼
                          main
```

---

# 6. Nunca actualizar directamente develop

Para incorporar cambios importantes de upstream se utilizará una rama temporal.

Ejemplo:

```bash
git switch develop

git pull origin develop

git fetch upstream
```

Crear rama:

```bash
git switch -c chore/sync-upstream-YYYY-MM-DD
```

Ejemplo:

```bash
git switch -c chore/sync-upstream-2026-10-01
```

---

# 7. Crear backup antes de actualizar

Antes de sincronizar:

```bash
git status
```

Debe mostrar:

```text
working tree clean
```

Crear tag:

```bash
git tag backup-before-upstream-YYYY-MM-DD
```

Ejemplo:

```bash
git tag backup-before-upstream-2026-10-01
```

Subir tag:

```bash
git push origin backup-before-upstream-2026-10-01
```

Esto permite volver rápidamente al estado anterior.

---

# 8. Obtener cambios upstream

Ejecutar:

```bash
git fetch upstream
```

Comprobar ramas:

```bash
git branch -r
```

Ejemplo:

```text
upstream/dev-2.9.0
upstream/main
```

Mientras termixCustom esté basado en Termix 2.9, la referencia principal será:

```text
upstream/dev-2.9.0
```

---

# 9. Revisar cambios antes de fusionar

Ver commits nuevos:

```bash
git log --oneline HEAD..upstream/dev-2.9.0
```

Ver cambios estadísticos:

```bash
git diff --stat HEAD..upstream/dev-2.9.0
```

Revisar diferencias completas cuando sea necesario:

```bash
git diff HEAD..upstream/dev-2.9.0
```

---

# 10. Fusionar upstream

Desde nuestra rama temporal:

```bash
git merge upstream/dev-2.9.0
```

No usar automáticamente:

```text
--strategy-option theirs
```

ni:

```text
--strategy-option ours
```

porque podrían eliminar cambios importantes.

Resolver los conflictos de forma individual.

---

# 11. Prioridad al resolver conflictos

Cuando exista un conflicto:

Primero entender:

```text
qué cambió upstream
```

Después entender:

```text
qué personalizamos nosotros
```

Finalmente conservar ambos comportamientos cuando sea posible.

Prioridades:

```text
1. Seguridad
2. Compatibilidad con Termix
3. Funcionalidad personalizada
4. Mantenibilidad
```

---

# 12. Archivos personalizados

Los siguientes archivos/directorios pertenecen principalmente a termixCustom:

```text
AGENTS.md

docs/CUSTOM_ARCHITECTURE.md
docs/UPSTREAM_SYNC.md

plugins/advanced-browser/
plugins/parsec-remote/
```

No deben eliminarse durante una actualización upstream.

---

# 13. Cambios al core

Los cambios propios en:

```text
electron/
src/
packages/plugin-sdk/
```

requieren atención especial durante los merges.

Siempre comprobar si upstream implementó ya una función equivalente.

Si upstream agregó una solución oficial mejor, considerar migrar nuestra implementación.

No mantener código duplicado innecesariamente.

---

# 14. Browser

Al sincronizar upstream revisar especialmente:

```text
web-endpoint
Electron windows
WebContentsView
BrowserWindow
TLS handling
Plugin SDK desktop APIs
```

Si upstream mejora Web Endpoint:

- no eliminar Advanced Browser;
- evaluar reutilizar la nueva API;
- reducir modificaciones propias del core cuando sea posible.

---

# 15. Parsec Remote

Revisar especialmente cambios en:

```text
remote-desktop
plugin-sdk
desktop APIs
process spawning
host settings
credential handling
```

Si Termix agrega una API oficial para lanzar aplicaciones locales:

migrar Parsec Remote a esa API en lugar de mantener un bridge personalizado.

---

# 16. Actualizar dependencias

Después de un merge upstream:

```bash
npm install
```

No ejecutar actualizaciones indiscriminadas como:

```bash
npm update
```

salvo que sea una tarea específica.

Queremos conservar las versiones que upstream define.

---

# 17. Validación posterior

Ejecutar:

```bash
npm run build:sdk
```

Luego:

```bash
npm run type-check
```

Después:

```bash
npm run lint
```

Después:

```bash
npm run test
```

Y:

```bash
npm run test:plugins
```

Finalmente:

```bash
npm run build
```

Cuando sea relevante también:

```bash
npm run build:plugins
```

---

# 18. Prueba manual

Después de cada actualización upstream verificar al menos:

```text
Login
Hosts
SSH
File Manager
RDP/VNC
Web Endpoint
Tunnels
Advanced Browser
Parsec Remote
Credentials
Settings
```

Para Browser:

```text
HTTP page
HTTPS page
self-signed certificate
multiple tabs
host endpoint
```

Para Parsec:

```text
Parsec detection
missing Parsec behavior
peer ID
native launch
fallback
```

---

# 19. Si la actualización funciona

Commit:

```bash
git add .
```

```bash
git commit -m "chore(upstream): sync Termix upstream YYYY-MM-DD"
```

Ejemplo:

```bash
git commit -m "chore(upstream): sync Termix upstream 2026-10-01"
```

Subir rama:

```bash
git push -u origin chore/sync-upstream-2026-10-01
```

Después fusionar en `develop`.

---

# 20. Integrar en develop

```bash
git switch develop
```

```bash
git pull origin develop
```

```bash
git merge --no-ff chore/sync-upstream-2026-10-01
```

```bash
git push origin develop
```

---

# 21. Actualizar main

`main` solamente recibirá la actualización cuando `develop` haya sido probado.

```bash
git switch main
```

```bash
git pull origin main
```

```bash
git merge --no-ff develop
```

```bash
git push origin main
```

---

# 22. Volver atrás

Si la actualización falla gravemente:

Consultar tag:

```bash
git tag
```

Ejemplo:

```text
backup-before-upstream-2026-10-01
```

No ejecutar reset destructivo sin comprobar primero cambios pendientes.

Para crear una rama desde el backup:

```bash
git switch -c recovery/upstream-failed backup-before-upstream-2026-10-01
```

Así no se destruye historial.

---

# 23. Nunca hacer

Evitar:

```bash
git reset --hard upstream/dev-2.9.0
```

sobre `develop`.

Eso podría borrar nuestras modificaciones.

Evitar:

```bash
git push --force
```

sobre:

```text
main
develop
```

salvo recuperación excepcional y controlada.

Evitar copiar manualmente todo el repositorio oficial sobre termixCustom.

Evitar borrar:

```text
plugins/advanced-browser
plugins/parsec-remote
```

durante una actualización.

---

# 24. Si upstream cambia de versión

Cuando Termix 2.9 deje de ser rama de desarrollo y aparezca una nueva base estable, no cambiar automáticamente.

Primero crear una rama:

```text
migration/termix-X.Y
```

Ejemplo:

```bash
git switch -c migration/termix-3.0
```

Probar allí la migración.

Solo después cambiar la referencia upstream usada por termixCustom.

---

# 25. Reglas para Codex durante sync

Cuando Codex participe en una sincronización upstream deberá:

1. Leer `AGENTS.md`.
2. Leer `docs/CUSTOM_ARCHITECTURE.md`.
3. Leer `docs/UPSTREAM_SYNC.md`.
4. Revisar conflictos uno por uno.
5. No elegir automáticamente ours/theirs para todo.
6. Conservar plugins personalizados.
7. Buscar APIs upstream nuevas que puedan reemplazar código personalizado.
8. Ejecutar pruebas.
9. Informar claramente cualquier incompatibilidad.
10. No borrar funcionalidades para resolver conflictos rápidamente.

---

# 26. Comando resumido de actualización

Flujo habitual:

```bash
git switch develop
git pull origin develop

git status

git tag backup-before-upstream-YYYY-MM-DD
git push origin backup-before-upstream-YYYY-MM-DD

git fetch upstream

git switch -c chore/sync-upstream-YYYY-MM-DD

git log --oneline HEAD..upstream/dev-2.9.0

git merge upstream/dev-2.9.0

npm install

npm run build:sdk
npm run type-check
npm run lint
npm run test
npm run test:plugins
npm run build
```

Después de validar:

```bash
git add .
git commit -m "chore(upstream): sync Termix upstream YYYY-MM-DD"
git push -u origin chore/sync-upstream-YYYY-MM-DD
```

Y posteriormente integrar en `develop`.

---

# 27. Principio final

Nunca debemos tratar termixCustom como una copia desconectada de Termix.

La relación debe mantenerse:

```text
Termix upstream
      │
      ▼
termixCustom core
      │
      ├── Advanced Browser
      ├── Parsec Remote
      └── futuras extensiones
```

Nuestro objetivo es mantener las personalizaciones separadas, pequeñas y fáciles de actualizar.
