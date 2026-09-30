# termixCustom — Project Rules

## Objetivo del proyecto

termixCustom es una personalización de Termix orientada a convertirlo en una consola unificada para administrar servidores, estaciones Windows/macOS, dispositivos de red y equipos con interfaces web.
ø
El proyecto parte de Termix 2.9 y debe conservar compatibilidad razonable con el proyecto upstream.

Las dos extensiones principales iniciales son:

- Advanced Browser.
- Parsec Remote.

En el futuro deben poder agregarse nuevos proveedores y módulos sin rediseñar la aplicación.

## 1. Regla principal: preservar Termix

No eliminar, reemplazar ni romper funciones existentes de Termix salvo que exista una razón técnica documentada.

Antes de modificar código existente, comprobar si la funcionalidad puede implementarse mediante el sistema de plugins.

Se debe preferir siempre:

plugin → SDK → extensión mínima de Electron → modificación de core

y nunca al revés.

No realizar refactors masivos del código original únicamente para adaptar una función personalizada.

Los cambios al core deben ser pequeños, aislados, reutilizables y fáciles de mantener cuando se incorporen actualizaciones de upstream.

## 2. Estrategia Git

`upstream` representa exclusivamente el proyecto oficial:

Termix-SSH/Termix

`origin` representa nuestro proyecto:

arturinho16/termixCustom

Nunca realizar push hacia `upstream`.

`main` contiene una versión integrada y comprobada de termixCustom.

`develop` es la rama de integración del desarrollo.

Las funcionalidades nuevas deben desarrollarse en ramas independientes creadas desde `develop`.

Ejemplos:

feature/advanced-browser

feature/parsec-remote

feature/remote-provider-core

fix/browser-certificates

fix/parsec-detection

No desarrollar directamente sobre `main`.

No mezclar una funcionalidad grande de Browser con una funcionalidad grande de Parsec en el mismo commit.

## 3. Actualizaciones desde Termix oficial

termixCustom debe poder incorporar cambios futuros de Termix.

Las modificaciones propias no deben dificultar innecesariamente los merges desde `upstream`.

Antes de integrar una actualización de upstream:

guardar el estado actual;

crear un punto de restauración/tag;

actualizar una rama de prueba;

resolver conflictos;

ejecutar pruebas;

integrar después en `develop`.

Nunca sobrescribir nuestros módulos personalizados al actualizar Termix.

## 4. Arquitectura basada en plugins

Toda funcionalidad personalizada debe implementarse como plugin siempre que el SDK de Termix lo permita.

Los primeros plugins propios serán:

plugins/advanced-browser

plugins/parsec-remote

Browser y Parsec Remote deben mantenerse separados.

No crear un único plugin gigante para todas las funciones personalizadas.

Las dependencias entre plugins deben ser explícitas.

No acceder directamente a componentes internos del core cuando exista una API del SDK equivalente.

## 5. Advanced Browser

Advanced Browser será un navegador completo integrado con Termix.

No debe ser únicamente una lista de Web Endpoints.

Debe permitir navegación libre mediante URL.

Funciones previstas:

barra de URL;

atrás;

adelante;

recargar;

pestañas;

favoritos;

historial;

fullscreen;

HTTP;

HTTPS;

integración con Hosts;

acceso rápido a interfaces como iDRAC, iLO, Proxmox, ESXi, routers, switches, firewalls y cámaras;

soporte controlado para certificados autofirmados;

futuro soporte para túneles SSH/SOCKS.

Web Endpoint seguirá existiendo y no debe romperse.

Advanced Browser debe poder reutilizar información de Web Endpoints cuando sea conveniente.

## 6. Seguridad del navegador

El contenido web remoto debe estar aislado del proceso principal de Termix.

Cuando se utilicen ventanas o vistas Electron se conservarán principios equivalentes a:

sandbox habilitado;

contextIsolation habilitado;

nodeIntegration deshabilitado;

webSecurity habilitado.

Nunca desactivar globalmente la validación TLS de Electron.

Los certificados autofirmados solo podrán autorizarse explícitamente para el host/origen correspondiente.

Una excepción TLS para un dispositivo nunca debe convertirse en una excepción global.

Una página cargada dentro del navegador no debe obtener acceso a APIs internas de Termix, Node.js, credenciales o filesystem.

## 7. Integración Browser + Hosts

Un Host de Termix podrá tener cero, uno o varios endpoints web.

Ejemplo:

Dell R730

SSH

Files

Metrics

iDRAC

Browser

Los endpoints guardarán referencias como:

nombre;

protocolo;

host/IP;

puerto;

path;

modo de apertura;

preferencias TLS.

No guardar contraseñas web en texto plano dentro de la configuración del navegador.

Si en el futuro se implementa autenticación automática, deberá usar el sistema seguro de credenciales de Termix.

## 8. Parsec Remote

Parsec será tratado como proveedor de escritorio remoto de alto rendimiento.

Parsec Remote será un plugin independiente.

No se intentará reimplementar el protocolo de streaming de Parsec.

No utilizar Parsec Web como método principal.

La integración debe lanzar el cliente nativo de Parsec para conservar aceleración gráfica, codecs, latencia y rendimiento.

Debe contemplar al menos:

Windows → macOS;

macOS → Windows;

Windows → Windows;

macOS → macOS cuando Parsec lo permita.

## 9. Datos Parsec por host

Cada Host podrá tener configuración específica de Parsec.

La configuración podrá contener:

Parsec habilitado;

Peer ID;

sistema operativo destino;

proveedor preferido;

fallback;

opciones de conexión admitidas.

Nunca guardar la contraseña de la cuenta Parsec dentro de termixCustom.

Parsec conservará su propia sesión autenticada.

Termix únicamente almacenará la información necesaria para identificar y abrir el host correspondiente.

## 10. Credenciales

No guardar secretos en:

JSON sin cifrar;

localStorage;

archivos de configuración;

logs;

variables hardcodeadas;

código fuente;

manifest de plugins.

Las credenciales Windows, macOS, Linux, SSH, RDP o VNC deben reutilizar el sistema seguro de credenciales de Termix siempre que sea posible.

Los plugins deben guardar referencias a credenciales, no duplicar contraseñas.

Parsec Remote no debe tener su propio almacén independiente de contraseñas del sistema operativo.

Nunca imprimir passwords, tokens, claves privadas o secretos en logs de desarrollo.

## 11. Remote Providers

Parsec no debe quedar hardcodeado como el único método de escritorio remoto futuro.

Diseñar una abstracción de proveedor que permita posteriormente incorporar:

Parsec;

RDP;

VNC;

Moonlight/Sunshine;

NoMachine;

otros proveedores.

Un host podrá definir:

Primary Remote Provider

Fallback Provider

Emergency Provider

Ejemplo para macOS:

Primary: Parsec

Fallback: VNC

Emergency: SSH

Ejemplo para Windows:

Primary: Parsec

Fallback: RDP

Emergency: SSH/PowerShell

## 12. Lanzamiento de aplicaciones nativas

Parsec debe lanzarse mediante una capa segura controlada por Termix.

Nunca permitir que una cadena proveniente del usuario sea ejecutada directamente como comando de shell.

No utilizar:

exec(userInput)

o equivalentes.

Los ejecutables admitidos y sus argumentos deben validarse.

El plugin únicamente podrá lanzar proveedores expresamente autorizados.

Detectar automáticamente las rutas conocidas de Parsec según el sistema operativo cuando sea posible.

Si Parsec no está instalado, mostrar un error claro y no romper Termix.

## 13. Compatibilidad multiplataforma

termixCustom debe seguir compilando en:

Linux;

Windows;

macOS.

Una función exclusiva de Windows/macOS no debe romper la compilación de Linux.

El entorno principal de desarrollo puede ser Ubuntu, pero cualquier integración nativa debe abstraerse por plataforma.

Usar detección de plataforma y degradación elegante cuando una función no esté disponible.

## 14. UI y experiencia de usuario

Las nuevas funciones deben respetar visualmente Termix.

Reutilizar componentes, estilos, iconografía y patrones existentes.

No crear una segunda interfaz visual diferente dentro de los plugins.

Advanced Browser y Parsec Remote deben sentirse como funcionalidades nativas de Termix.

Los hosts deben continuar siendo el centro de navegación.

Ejemplo:

HOST

SSH

Files

Docker

Metrics

Browser

Parsec Remote

RDP/VNC

## 15. Cambios mínimos en Electron

Modificar Electron solamente cuando una función no pueda implementarse mediante el Plugin SDK.

Ejemplos aceptables:

crear una WebContentsView segura;

crear una BrowserWindow aislada;

detectar aplicaciones locales;

lanzar Parsec de forma segura.

Estas funciones deben exponerse mediante APIs reducidas y controladas.

Nunca entregar al renderer acceso general a:

child_process;

filesystem completo;

shell arbitrario;

Node.js.

## 16. Base de datos y migraciones

No modificar tablas existentes innecesariamente.

Los datos específicos de cada plugin deben pertenecer al plugin.

Cuando una modificación requiera migración:

crear migración;

mantener compatibilidad con instalaciones existentes;

no eliminar datos automáticamente;

incluir pruebas de migración.

No almacenar datos sensibles sin utilizar los mecanismos seguros correspondientes.

## 17. Calidad de código

Mantener TypeScript estricto.

No introducir `any` sin justificación.

No duplicar lógica existente.

No dejar código muerto.

No agregar dependencias npm cuando la funcionalidad ya exista en Termix o en Node/Electron.

Respetar ESLint, Prettier y convenciones actuales del proyecto.

Los nombres de código, variables, commits y archivos permanecerán preferentemente en inglés.

La interfaz podrá utilizar el sistema i18n de Termix para español y demás idiomas.

## 18. Pruebas obligatorias

Cada nueva función importante debe incluir pruebas.

Antes de considerar una tarea terminada ejecutar, según corresponda:

npm run type-check

npm run lint

npm run test

npm run test:plugins

npm run build:sdk

npm run build:plugins

No declarar una tarea terminada si el proyecto no compila.

Si existe un fallo previo de upstream que no está relacionado con nuestros cambios, documentarlo explícitamente.

## 19. Regla para Codex

Antes de modificar código, Codex debe:

leer AGENTS.md;

revisar la implementación existente relacionada;

identificar si existe un plugin o API que ya resuelva parte del problema;

proponer cambios mínimos;

preservar comportamiento existente.

Codex no debe borrar ni reescribir una funcionalidad completa para implementar algo pequeño.

Cuando el usuario agregue un nuevo requisito, se deben conservar todas las reglas y decisiones anteriores salvo que el usuario explícitamente las cambie.

## 20. Alcance de las tareas

Cada iteración debe tener un objetivo claro.

No aprovechar una tarea para modificar componentes no relacionados.

Ejemplo:

si la tarea es implementar pestañas en Advanced Browser, no modificar simultáneamente el sistema de RDP.

Si la tarea es detectar Parsec en macOS, no refactorizar Browser.

Los cambios deben mantenerse pequeños y revisables.

## 21. Commits

Utilizar commits claros y atómicos.

Ejemplos:

feat(browser): add advanced browser plugin skeleton

feat(browser): add tab navigation

feat(browser): add per-host web endpoints

feat(parsec): add native client detection

feat(parsec): launch host by peer id

feat(parsec): add remote provider fallback

fix(browser): scope certificate exception to origin

Los commits no deben mezclar funciones independientes.

## 22. Prioridad de desarrollo

El orden inicial del proyecto será:

Primero conseguir una instalación limpia de termixCustom funcionando.

Después implementar Advanced Browser.

Después integrar Advanced Browser con Hosts.

Después implementar Parsec Remote.

Después integrar el sistema de Remote Providers y fallbacks.

Después realizar pruebas Windows ↔ macOS.

Después preparar builds Windows, macOS y Linux.

No comenzar todas las funciones simultáneamente.

## 23. Definición de terminado

Una función se considera terminada solamente cuando:

compila;

tiene pruebas razonables;

no rompe funciones existentes;

funciona en el entorno objetivo;

respeta las reglas de seguridad;

tiene configuración persistente cuando corresponde;

está integrada visualmente con Termix;

está documentada;

y puede mantenerse al incorporar actualizaciones de upstream.

## 24. Regla de conservación

Las decisiones anteriores de arquitectura son acumulativas.

Un nuevo requerimiento complementa los anteriores y no los reemplaza automáticamente.

Si una nueva petición entra en conflicto con una regla anterior, se debe señalar el conflicto antes de eliminar comportamiento existente.
