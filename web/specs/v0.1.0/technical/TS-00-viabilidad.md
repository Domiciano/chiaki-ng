# TS-00 · Viabilidad de una versión web

| ID | Versión | Estado | FS que cubre |
|---|---|---|---|
| TS-00 | v0.1.0 | Borrador | Todas (decisión de arquitectura) |

## Contexto

La pregunta inicial era si chiaki-ng podía ser **solo una página web** en TypeScript/React.

## Hallazgos

**El protocolo necesita sockets que el navegador no expone:**

| Función | Transporte | Código |
|---|---|---|
| Descubrimiento de consolas | UDP broadcast | `lib/src/discovery.c`, `lib/src/discoveryservice.c` |
| Registro con PIN | UDP + TCP 9295 | `lib/src/regist.c` |
| Control de sesión | TCP 9295 | `lib/src/ctrl.c`, `lib/src/session.c` |
| Streaming (Takion) | UDP 9296 | `lib/src/takion.c`, `lib/src/streamconnection.c` |
| Prueba de red (Senkusha) | UDP 9297 | `lib/src/senkusha.c` |
| Remoto vía PSN (hole-punch, RUDP) | UDP + STUN | `lib/src/remote/holepunch.c`, `lib/src/remote/rudp.c` |

Un navegador solo tiene WebSocket, WebRTC y WebTransport, y el PS5 no habla ninguno de los tres. Direct Sockets API existe solo para Isolated Web Apps en Chrome, no para una página servida por nginx.

**El canje del token de Sony** (`https://auth.api.sonyentertainmentnetwork.com/2.0/oauth/token`, `gui/include/psnaccountid.h`, `gui/src/psntoken.cpp`) usa credenciales de cliente fijas y no permite CORS desde un origen propio. Desde el navegador queda bloqueado.

**Lo que sí es viable en el navegador:**

| Necesidad | API del navegador |
|---|---|
| Decodificar H.264 (y HEVC donde exista) | WebRTC (hardware) |
| Reproducir Opus | WebRTC |
| Control Bluetooth: botones, sticks, gatillos | Gamepad API |
| Rumble básico | `GamepadHapticActuator` (`vibrationActuator`) |
| Giroscopio, touchpad, gatillos adaptativos, luz | WebHID (Chrome/Edge) |
| Micrófono | `getUserMedia` |
| Login Sony | Flujo de "pegar la URL de redirección", que ya existe en `gui/src/qml/PSNLoginDialog.qml` |

## Decisión

Se descarta la página web sola. Se adopta **frontend React/TS + puente C++ que reutiliza `chiaki-lib`**, con WebRTC entre el navegador y el puente. El detalle está en [TS-01](TS-01-arquitectura.md) y las razones de cada decisión de transporte en [TS-03](TS-03-webrtc-y-senalizacion.md).

Descartado también: reescribir el protocolo en Node/TypeScript. Habría que reimplementar Takion, el cifrado (`rpcrypt.c`, `gkcrypt.c`), FEC (`fec.c` + jerasure) y el hole-punch, con un alto riesgo de incompatibilidad con el firmware del PS5.

## Decisión de host

El host de referencia es un **PC con Windows** en la LAN del PS5 (ASUS TUF Gaming F15), conectado por cable al router donde está la consola. Funciona con encendido programado (hiberna de noche y despierta con una tarea programada), y los clientes acceden por Tailscale ([TS-01](TS-01-arquitectura.md), [TS-06](TS-06-despliegue-docker-compose.md#perfil-tailscale-recomendado)). Se descartó usar un servidor fuera de la LAN del PS5, porque el puente necesita estar en la misma red que la consola.

Las specs de despliegue asumen `network_mode: host`, que en Linux da acceso directo al broadcast y a los puertos UDP. En Windows hay tres opciones, que se evalúan en el spike (T-01) y en la tarea T-26:

| Opción | Ventaja | Riesgo |
|---|---|---|
| **A · WSL2 con red `mirrored`** (`.wslconfig`: `networkingMode=mirrored`) y Docker dentro de WSL2 | Se mantiene el `docker-compose` de TS-06 casi sin cambios. | El broadcast de descubrimiento y la recepción de UDP entrante pueden no comportarse igual que en Linux nativo. Hay que verificarlo, junto con las reglas del firewall de Windows y Hyper-V. |
| **B · Puente nativo en Windows** (MSYS2 o MSVC, como el build de escritorio de `scripts/deploy-windows-msys2.sh`), con nginx fuera o dentro de Docker | Acceso directo a la red del host, sin capas de virtualización. | Se pierde el despliegue de un solo `docker compose up`. Hay que empaquetar el puente como servicio de Windows. |
| **C · Arranque dual con Linux** | Es el entorno de referencia de las specs. | Fricción de uso: no se puede usar Windows al mismo tiempo. |

Si el broadcast no cruza (opción A), el puente admite agregar la consola por **IP manual**. Eso hay que incluirlo en FS-02 si se elige A.

## Matriz de navegadores

| Capacidad | Chrome/Edge | Firefox | Safari |
|---|---|---|---|
| WebRTC H.264 + Opus | Sí | Sí | Sí |
| WebRTC H.265 | Sí, con hardware compatible | No | Sí, en versiones recientes |
| Gamepad API con DualSense/DS4 por Bluetooth | Sí, mapeo `standard` | Sí, el mapeo puede variar | Sí |
| `vibrationActuator` | Sí | No | Parcial |
| WebHID | Sí | No | No |

Gamepad, WebHID y `getUserMedia` exigen un **contexto seguro**, así que el despliegue debe servir por HTTPS ([TS-06](TS-06-despliegue-docker-compose.md)).

La matriz se debe confirmar en el spike y en las pruebas manuales ([TS-07](TS-07-pruebas.md)), porque el soporte de los navegadores cambia con cada versión.

## Riesgos

- El salto extra navegador ⇄ puente agrega latencia. En LAN debería ser menor de 2 ms. Se mide en el spike.
- El PS5 fija el bitrate al iniciar la sesión, y WebRTC no lo puede renegociar en caliente.

## Validación

Tarea **T-01 (spike)** de [devprogress.md](../devprogress.md): el puente inicia una sesión con un PS5 real y un navegador muestra el video por WebRTC.
