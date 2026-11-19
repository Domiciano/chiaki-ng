# TS-02 · Puente C++ (`web/bridge/`)

| ID | Versión | Estado | FS que cubre |
|---|---|---|---|
| TS-02 | v0.1.0 | Borrador | [FS-02](../functional/FS-02-descubrimiento-consolas.md), [FS-03](../functional/FS-03-registro-consola.md), [FS-04](../functional/FS-04-streaming.md), [FS-06](../functional/FS-06-ajustes.md) |

## Contexto

Es el único proceso que habla con el PS5. Reutiliza `chiaki-lib` tal cual, sin modificar `lib/`. Es un servicio sin interfaz y sin Qt.

## Diseño

**Construcción:**

- C++17 con CMake. `web/bridge/CMakeLists.txt` incluye la raíz del repo con `add_subdirectory(../.. chiaki)` y estas opciones:
  - `CHIAKI_ENABLE_GUI=OFF`, `CHIAKI_ENABLE_CLI=OFF`, `CHIAKI_ENABLE_TESTS=OFF`.
  - `CHIAKI_ENABLE_FFMPEG_DECODER=OFF`: el puente no decodifica.
  - `CHIAKI_LIB_ENABLE_OPUS=OFF` en v0.1.0. En v0.3.0 vuelve a `ON` para el micrófono ([TS-11](../../v0.3.0/technical/TS-11-microfono-webrtc.md)).
  - `CHIAKI_ENABLE_STEAMDECK_NATIVE=OFF`, `CHIAKI_ENABLE_SETSU=OFF`.
  - `CHIAKI_ENABLE_RUDP=ON`, para tenerlo listo para v0.2.0.
- Dependencias, por `FetchContent` con versión fijada:
  - `libdatachannel`: WebRTC, empaquetadores RTP y servidor WebSocket.
  - `nlohmann/json`.
- Ejecutable: `chiaki-web-bridge`.

**Módulos:**

| Módulo | Responsabilidad | Reutiliza |
|---|---|---|
| `SignalServer` | WebSocket en `:8090`, despacho de mensajes JSON, `GET /health` | (libdatachannel `WebSocketServer`) |
| `DiscoveryService` | Descubrir consolas cada 2 s, cachear su estado y empujar `hosts.updated` | `chiaki_discovery_service_init`, con el patrón de `gui/src/discoverymanager.cpp` |
| `WakeService` | Despertar una consola en reposo | `chiaki_discovery_wakeup` (`lib/include/chiaki/discovery.h`), como en `gui/src/discoverymanager.cpp` |
| `RegistService` | Registrar con el PIN y guardar el `ChiakiRegisteredHost` | `chiaki_regist_start` (`lib/include/chiaki/regist.h`), con el flujo de `QmlRegist` en `gui/src/qmlbackend.cpp` |
| `HostStore` | Persistir las consolas registradas en `/data/hosts.json` | Campos equivalentes a `gui/src/settings.cpp` y `gui/src/host.cpp` |
| `SettingsStore` | Persistir los ajustes de stream en `/data/settings.json` | Valores por defecto de `gui/src/settings.cpp` |
| `StreamSession` | Ciclo de vida de `ChiakiSession` y puente con `WebRtcPeer` | `chiaki_session_init/start/stop/join/fini`, con el patrón de `gui/src/streamsession.cpp` |
| `WebRtcPeer` | `rtc::PeerConnection`, tracks de video y audio, DataChannels | Ver [TS-03](TS-03-webrtc-y-senalizacion.md) |

**Construcción del `ChiakiConnectInfo`**, como hace `StreamSession` en el escritorio:

- `ps5 = true`, `host` = IP de la consola, y `regist_key` y `morning` desde `HostStore`.
- `video_profile` sale de `chiaki_connect_video_profile_preset(resolución, fps)` con los Ajustes, más:
  - `codec = CHIAKI_CODEC_H264`, forzado en v0.1.0.
  - `bitrate` desde los Ajustes.
- `video_profile_auto_downgrade = true`.
- `enable_dualsense = true` y `enable_keyboard = true`.
- `enable_idr_on_fec_failure = true`.
- `psn_account_id` desde `HostStore`.

**Callbacks de sesión** hacia `WebRtcPeer`:

| Callback de chiaki-lib | Acción en el puente |
|---|---|
| `chiaki_session_set_video_sample_cb` | Envía el frame H.264 (Annex-B) al track de video. Si `frames_lost > 0` y no hay `frame_recovered`, marca que se necesita un IDR. |
| `chiaki_session_set_audio_sink`: `header_cb` | Valida canales y frecuencia (esperado 2 canales a 48 kHz) y configura el track Opus. |
| `chiaki_session_set_audio_sink`: `frame_cb` | Envía el paquete Opus al track de audio. |
| `event_cb`: `CHIAKI_EVENT_CONNECTED` | `session.state = streaming` |
| `event_cb`: `CHIAKI_EVENT_LOGIN_PIN_REQUEST` | Pide el PIN al navegador (`session.pin_request`) |
| `event_cb`: `CHIAKI_EVENT_RUMBLE` | Mensaje `rumble` por el DataChannel `control` |
| `event_cb`: `CHIAKI_EVENT_KEYBOARD_OPEN` / `_TEXT_CHANGE` / `_REMOTE_CLOSE` | Mensajes `keyboard.*` |
| `event_cb`: `CHIAKI_EVENT_QUIT` | `session.ended` con `chiaki_quit_reason_string`, y cierra el peer |

**Hilos:** `chiaki-lib` invoca los callbacks desde sus propios hilos. `WebRtcPeer` es seguro para hilos (libdatachannel lo es al enviar). Los envíos al WebSocket pasan por una cola protegida con mutex.

**Configuración por entorno:**

| Variable | Por defecto | Uso |
|---|---|---|
| `BRIDGE_SIGNAL_PORT` | `8090` | WebSocket y `/health` |
| `BRIDGE_DATA_DIR` | `/data` | `hosts.json`, `settings.json` |
| `BRIDGE_RTC_PORT_MIN` / `_MAX` | `50000` / `50100` | Rango UDP de WebRTC |
| `BRIDGE_ANNOUNCE_IP` | (auto) | IP que se anuncia en los candidatos ICE |
| `BRIDGE_LOG_LEVEL` | `info` | Pasa a la máscara de `ChiakiLog` |

## Contratos

Mensajes de señalización: [TS-03](TS-03-webrtc-y-senalizacion.md).

`hosts.json`:

```json
{ "hosts": [ {
  "id": "mac-hex", "nickname": "PS5-123", "target": "ps5",
  "registKey": "hex", "morning": "hex", "serverMac": "hex",
  "psnAccountId": "base64", "lastAddress": "192.168.1.50"
} ] }
```

## Código de chiaki-ng que se reutiliza

- `lib/` completo, sin modificar.
- `gui/src/discoverymanager.cpp`, `gui/src/qmlbackend.cpp` (`QmlRegist`), `gui/src/streamsession.cpp` y `gui/src/settings.cpp`: solo como referencia de uso. No se enlazan porque dependen de Qt.

## Riesgos

- `streamsession.cpp` mezcla lógica de sesión con Qt y SDL. Hay que extraer con cuidado el orden exacto de inicialización (sinks antes de `chiaki_session_start`).
- `hosts.json` contiene la `regist_key`, que da acceso a la consola. El archivo va con permisos `0600` en un volumen que no se expone.

## Pruebas

Ver [TS-07](TS-07-pruebas.md): serialización de `hosts.json`, parseo de mensajes y una prueba manual con un PS5.
