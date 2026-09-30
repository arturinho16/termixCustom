# termixCustom — Custom Architecture

## 1. Propósito

Este documento define la arquitectura personalizada de `termixCustom`.

Repositorio principal:

```text
https://github.com/arturinho16/termixCustom
```

Proyecto upstream:

```text
https://github.com/Termix-SSH/Termix
```

`termixCustom` extiende Termix sin reemplazar innecesariamente sus funciones existentes.

El objetivo es convertir Termix en una consola unificada para:

- Servidores Linux.
- Servidores Windows.
- Equipos macOS.
- Workstations.
- Proxmox.
- iDRAC.
- iLO.
- ESXi.
- Switches.
- Routers.
- Firewalls.
- Cámaras IP.
- Interfaces web de dispositivos.
- Escritorio remoto de alto rendimiento.
- SSH.
- RDP.
- VNC.
- Gestión de archivos.
- Métricas.
- Túneles.
- Tailscale.
- Futuras integraciones.

---

# 2. Principio arquitectónico

La prioridad será conservar Termix lo más cercano posible al upstream.

El orden de implementación preferido es:

```text
Plugin
  ↓
Plugin SDK
  ↓
Extensión pequeña de Electron
  ↓
Modificación mínima de Core
```

No se modificará el core cuando la funcionalidad pueda resolverse mediante un plugin.

Las personalizaciones deberán ser:

- Modulares.
- Reemplazables.
- Probables.
- Actualizables.
- Multiplataforma.
- Seguras.

---

# 3. Arquitectura general

```text
                        termixCustom
                             │
                ┌────────────┴─────────────┐
                │                          │
             Termix Core             Custom Layer
                │                          │
       ┌────────┼─────────┐       ┌────────┴────────┐
       │        │         │       │                 │
      Hosts    SDK     Electron  Plugins        Providers
                                    │                 │
                          ┌─────────┴──────┐    ┌────┴─────┐
                          │                │    │          │
                    Advanced Browser  Parsec Remote    Future
```

---

# 4. Componentes originales de Termix

termixCustom conserva las funciones existentes de Termix.

Entre ellas:

```text
SSH Terminal
Remote Desktop
RDP
VNC
Telnet
File Manager
Docker / Podman
Tunnels
Host Metrics
Tailscale
Proxmox
Automations
Alerts
Web Endpoints
Credentials
Workspaces
Session Recording
Session Sharing
Serial
```

Las nuevas funciones deberán integrarse con estas capacidades.

No deben duplicarlas innecesariamente.

---

# 5. Plugins personalizados iniciales

Los primeros módulos personalizados serán:

```text
plugins/
├── advanced-browser/
└── parsec-remote/
```

Browser y Parsec serán plugins independientes.

No deberán compartir código directamente salvo mediante APIs o componentes comunes claramente definidos.

---

# 6. Advanced Browser

## 6.1 Objetivo

Advanced Browser proporcionará un navegador completo dentro de Termix.

No reemplazará Web Endpoint.

Web Endpoint continuará siendo útil para accesos web asociados a hosts.

Advanced Browser agregará navegación libre.

---

## 6.2 Funciones iniciales

```text
Advanced Browser
│
├── URL bar
├── Back
├── Forward
├── Reload
├── Tabs
├── HTTPS
├── HTTP
├── Fullscreen
├── Favorites
├── History
├── Per-host shortcuts
└── Self-signed certificate handling
```

Funciones posteriores:

```text
SSH tunnels
SOCKS5
Proxy per host
Tailscale-aware routes
Saved browser profiles
Credential integration
Custom user agents
Compatibility profiles
```

---

# 7. Integración Browser + Hosts

Los hosts seguirán siendo el centro de la experiencia Termix.

Ejemplo:

```text
Dell PowerEdge R730
│
├── SSH
├── Files
├── Metrics
├── Docker
├── Browser
└── iDRAC
```

Ejemplo para una cámara:

```text
CAM-TORRE-01
│
├── Browser
├── Web Endpoint
└── Metrics
```

Cada host podrá tener múltiples endpoints web.

Ejemplo:

```text
HOST: Dell R730

Endpoint 1
Label: iDRAC
Scheme: https
Port: 443
Path: /

Endpoint 2
Label: Zabbix Agent UI
Scheme: http
Port: 8080
Path: /
```

---

# 8. Arquitectura del navegador

La arquitectura recomendada es:

```text
React UI
   │
   ▼
Advanced Browser Plugin
   │
   ▼
Electron Bridge
   │
   ▼
WebContentsView / BrowserWindow
   │
   ▼
Web destination
```

No se dará acceso Node.js a las páginas cargadas.

Configuración base:

```text
sandbox = true
contextIsolation = true
nodeIntegration = false
webSecurity = true
```

---

# 9. Certificados autofirmados

Muchos dispositivos de infraestructura utilizan certificados HTTPS autofirmados.

Ejemplos:

```text
iDRAC
iLO
Proxmox
ESXi
Switches
Firewalls
Cámaras
```

termixCustom deberá permitir aceptar certificados de forma controlada.

Nunca se deshabilitará TLS globalmente.

Una excepción debe corresponder a un origen determinado.

Ejemplo:

```text
https://192.168.10.50:443
```

No deberá equivaler a:

```text
ignore all certificate errors
```

---

# 10. Parsec Remote

## 10.1 Objetivo

Parsec Remote será el proveedor principal de escritorio remoto de alto rendimiento.

Casos principales:

```text
Windows → macOS
macOS   → Windows
Windows → Windows
macOS   → macOS
```

cuando sean compatibles con Parsec.

---

# 11. Parsec como motor externo

termixCustom no reimplementará el protocolo de Parsec.

La arquitectura será:

```text
Termix
  │
  │ selecciona host
  ▼
Parsec Remote Plugin
  │
  │ peer_id
  ▼
Native Parsec Client
  │
  ▼
Remote Host
```

Parsec seguirá siendo responsable de:

```text
Video streaming
Hardware encoding
Hardware decoding
H.264
H.265
Input
Audio
Network transport
Latency optimization
```

Termix será responsable de:

```text
Host management
Connection shortcuts
Peer ID mapping
Provider selection
Fallbacks
Credential references
UI
OS detection
Native app detection
```

---

# 12. Datos Parsec por host

Ejemplo:

```text
Host:
Mac Studio Oficina

Parsec:
enabled: true

peerId:
xxxxxxxxxxxxxxxx

targetOS:
macos

provider:
parsec

fallback:
vnc
```

Para Windows:

```text
Host:
Workstation Diseño

Parsec:
enabled: true

peerId:
yyyyyyyyyyyyyyyy

targetOS:
windows

provider:
parsec

fallback:
rdp
```

---

# 13. Credenciales

Parsec no almacenará la contraseña de la cuenta Parsec en termixCustom.

La sesión de Parsec será administrada por el cliente oficial de Parsec.

Termix almacenará solamente referencias necesarias.

Ejemplo:

```text
parsecPeerId
```

Las credenciales del sistema operativo deberán utilizar el almacenamiento seguro de Termix.

Ejemplo:

```text
Credential:
Windows Administrator

Username:
administrator

Domain:
EMPRESA

Password:
encrypted
```

Ejemplo macOS:

```text
Credential:
Mac Studio Admin

Username:
arturcm

Password:
encrypted
```

Los plugins deben guardar la referencia:

```text
credentialId
```

y no repetir el password.

---

# 14. Remote Provider Architecture

Parsec debe implementarse mediante una abstracción que permita añadir otros proveedores.

Interfaz conceptual:

```ts
interface RemoteProvider {
  id: string;
  name: string;

  isAvailable(): Promise<boolean>;

  connect(options: RemoteConnectionOptions): Promise<void>;
}
```

Providers potenciales:

```text
parsec
rdp
vnc
moonlight
sunshine
nomachine
```

---

# 15. Selección automática de provider

Un host podrá definir:

```text
Primary Provider
Fallback Provider
Emergency Provider
```

Ejemplo macOS:

```text
Primary:
Parsec

Fallback:
VNC

Emergency:
SSH
```

Ejemplo Windows:

```text
Primary:
Parsec

Fallback:
RDP

Emergency:
SSH
```

Modo automático:

```text
Remote Desktop: Auto
```

Flujo:

```text
Is Parsec installed?
        │
      Yes
        │
Does host have peer_id?
        │
      Yes
        │
Launch Parsec
```

Si falla:

```text
Windows → RDP
macOS   → VNC
```

---

# 16. Detección de Parsec

termixCustom deberá detectar la aplicación Parsec en cada plataforma.

Ejemplos conceptuales:

Windows:

```text
C:\Program Files\Parsec\parsecd.exe
```

macOS:

```text
/Applications/Parsec.app/Contents/MacOS/parsecd
```

Linux podrá ofrecer solamente las funciones compatibles con el cliente Parsec disponible.

No se asumirá que Parsec está instalado.

Si no está disponible:

```text
Parsec is not installed on this device.
```

No deberá producir crash.

---

# 17. Ejecución segura

Nunca ejecutar:

```text
shell(userInput)
```

Nunca utilizar comandos construidos directamente con valores provenientes de usuarios.

El plugin deberá:

1. Detectar ejecutable autorizado.
2. Validar `peer_id`.
3. Construir argumentos internamente.
4. Ejecutar el binario mediante APIs seguras.

Ejemplo conceptual:

```text
spawn(
  parsecExecutable,
  [
    "peer_id=XXXXXXXX"
  ]
)
```

Sin:

```text
shell: true
```

cuando no sea necesario.

---

# 18. Electron Bridge

Los plugins no deben tener acceso arbitrario a Electron.

Se implementarán APIs limitadas.

Ejemplo Browser:

```text
browser.create()
browser.navigate()
browser.reload()
browser.goBack()
browser.goForward()
browser.close()
```

Ejemplo Parsec:

```text
parsec.detect()
parsec.connect(peerId)
parsec.open()
```

No deberá exponerse:

```text
child_process
fs
shell
exec
spawn
```

directamente al renderer.

---

# 19. Compatibilidad

El proyecto deberá mantener compatibilidad con:

```text
Linux
Windows
macOS
```

El entorno principal de desarrollo inicial es:

```text
Ubuntu
Node.js 24
npm 12
```

Una función exclusiva de macOS o Windows deberá degradarse correctamente en Linux.

---

# 20. Seguridad

Principios:

```text
Least privilege
Plugin isolation
Credential encryption
No plaintext secrets
Origin-scoped certificate exceptions
Input validation
No arbitrary process execution
No arbitrary Node access from browser content
```

Los permisos del plugin deberán ser solamente los necesarios.

---

# 21. Estructura prevista

```text
termixCustom/
│
├── electron/
│
├── packages/
│   └── plugin-sdk/
│
├── plugins/
│   ├── advanced-browser/
│   ├── parsec-remote/
│   ├── web-endpoint/
│   ├── remote-desktop/
│   └── ...
│
├── src/
│
├── docs/
│   ├── CUSTOM_ARCHITECTURE.md
│   └── UPSTREAM_SYNC.md
│
└── AGENTS.md
```

---

# 22. Estrategia de desarrollo

Orden:

```text
1. Baseline Termix funcionando
2. Advanced Browser skeleton
3. Browser navigation
4. Browser + Hosts
5. Browser TLS handling
6. Parsec Remote skeleton
7. Parsec detection
8. Peer ID per host
9. Native Parsec launch
10. Remote Provider abstraction
11. RDP/VNC fallback
12. Windows ↔ macOS tests
13. Packaging
```

---

# 23. Principio final

termixCustom deberá poder evolucionar sin convertirse en un fork imposible de actualizar.

La meta no es modificar Termix indiscriminadamente.

La meta es construir:

```text
Termix
+
Custom Plugins
+
Minimal Electron Extensions
```

manteniendo compatibilidad con upstream siempre que sea razonablemente posible.
