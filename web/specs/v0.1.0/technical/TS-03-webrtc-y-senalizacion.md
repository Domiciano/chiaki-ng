# TS-03 · WebRTC y señalización

| ID | Versión | Estado | FS que cubre |
|---|---|---|---|
| TS-03 | v0.1.0 | Borrador | [FS-02](../functional/FS-02-descubrimiento-consolas.md), [FS-03](../functional/FS-03-registro-consola.md), [FS-04](../functional/FS-04-streaming.md), [FS-05](../functional/FS-05-control-bluetooth.md) |

## Contexto

Este documento define cómo viajan el control, el media y el input entre el navegador y el puente.

## Decisiones (ADR)

| # | Decisión | Alternativa descartada | Razón |
|---|---|---|---|
| D1 | Media por **WebRTC** | WebSocket + WebCodecs | WebSocket corre sobre TCP: un paquete perdido congela el video hasta que se retransmite (bloqueo de cabeza de línea), y habría que implementar a mano el jitter y la sincronía A/V. |
| D2 | Media por WebRTC | WebTransport + WebCodecs | nginx no proxea WebTransport, y el puente tendría que terminar HTTP/3. Más complejo y sin ganancia en LAN. |
| D3 | Video y audio **sin transcodificar** | Transcodificar en el puente | Es lo que más pesa en latencia y CPU. El puente solo reempaqueta en RTP. |
| D4 | **H.264** por defecto | HEVC | Es el único códec que todos los navegadores decodifican en WebRTC. Límite: 1080p60 y sin HDR. HEVC queda como opción en [FS-11](../../v0.3.0/functional/FS-11-calidad-hevc.md). |
| D5 | **Opus** tal cual | Otro códec | Es lo que emite el PS5 y lo que WebRTC trae de forma nativa. |
| D6 | Controles por **DataChannel no fiable** con el **estado completo** | Deltas, o un canal fiable | Comparte la conexión UDP del media. Como cada mensaje lleva el estado completo, una pérdida se corrige con el siguiente y nunca deja un botón pegado. |
| D7 | Señalización por **WebSocket** | WHEP + REST | Además del SDP, el puente empuja eventos (consolas encontradas, PIN pedido, fin de sesión). Con REST habría que sondear. |
| D8 | El **puente crea la oferta** SDP | El navegador ofrece | El puente conoce de antemano los tracks y su códec (H.264 con un `profile-level-id` fijo), así que la negociación es determinista. |

**Limitación conocida:** el PS5 fija el bitrate al iniciar la sesión, y la adaptación de bitrate de WebRTC (REMB/TWCC) no lo puede cambiar. El bitrate se elige en Ajustes ([FS-06](../functional/FS-06-ajustes.md)).

## Diseño de la sesión WebRTC

- Una `rtc::PeerConnection` por sesión de stream. Los candidatos ICE salen del rango `BRIDGE_RTC_PORT_MIN..MAX`, con la IP de `BRIDGE_ANNOUNCE_IP`. En LAN no hace falta STUN ni TURN; TURN llega en [TS-09](../../v0.2.0/technical/TS-09-holepunch-y-turn.md).
- **Track de video** (`sendonly`):
  - H.264 con `packetization-mode=1` y `H264RtpPacketizer` en modo separador `StartSequence` (Annex-B).
  - Reloj RTP de 90 kHz. El timestamp sale del reloj monotónico al recibir el frame.
  - Se agregan `RtcpNackResponder` y `RtcpSrReporter`.
  - Tamaño máximo de paquete RTP de unos 1200 bytes, para caber en la MTU de 1280 del perfil Tailscale ([TS-06](TS-06-despliegue-docker-compose.md#perfil-tailscale-recomendado)).
  - Un PLI o FIR del navegador llama a `chiaki_session_request_idr`. Para no saturar, se limita a uno cada 500 ms.
- **Track de audio** (`sendonly`): Opus a 48 kHz, 2 canales, con `OpusRtpPacketizer` y reloj de 48 kHz.
- **Transceiver de audio N→P**: se negocia `inactive` desde v0.1.0 y queda reservado para el micrófono ([TS-11](../../v0.3.0/technical/TS-11-microfono-webrtc.md)), para poder activarlo sin renegociar el SDP.
- **DataChannel `input`**: `ordered=false`, `maxRetransmits=0`, binario.
- **DataChannel `control`**: fiable y ordenado, JSON.
- En el navegador, el receptor de video usa `jitterBufferTarget = 0` (con `playoutDelayHint = 0` como respaldo) para minimizar la latencia.

## Contratos

### Señalización (`wss://<host>/signal`, JSON)

Formato común: `{ "type": "...", "id": "<opcional, correlación>", ...payload }`. Las respuestas repiten el `id`. Los errores tienen la forma `{ "type": "error", "id", "code", "message" }`.

| Dirección | `type` | Payload |
|---|---|---|
| N→P | `hosts.list` | — |
| P→N | `hosts.updated` | `hosts: [{ id, nickname, address, state: "ready"\|"standby"\|"unknown", registered, target }]` (también se empuja ante cada cambio) |
| N→P | `hosts.wake` | `hostId` |
| N→P | `regist.start` | `address`, `pin` (8 dígitos), `psnAccountId` (base64) |
| P→N | `regist.result` | `ok`, `hostId?`, `error?` |
| N→P | `hosts.forget` | `hostId` |
| N→P | `settings.get` / `settings.set` | `settings` (ver [FS-06](../functional/FS-06-ajustes.md)) |
| N→P | `session.start` | `hostId` |
| P→N | `sdp.offer` | `sdp` |
| N→P | `sdp.answer` | `sdp` |
| N↔P | `ice.candidate` | `candidate`, `mid` |
| P→N | `session.state` | `state: "connecting"\|"streaming"\|"ended"` |
| P→N | `session.pin_request` | `incorrect: bool` |
| N→P | `session.pin` | `pin` |
| N→P | `session.stop` | `sleep: bool` (con `true` llama a `chiaki_session_goto_bed` antes de detener) |
| P→N | `session.ended` | `reason`, `reasonText` |

N = navegador, P = puente.

### DataChannel `input` (binario, little-endian, 56 bytes, versión 1)

| Offset | Tipo | Campo (`ChiakiControllerState`) |
|---|---|---|
| 0 | u8 | versión = 1 |
| 1 | u8 | reservado |
| 2 | u16 | secuencia; el puente descarta los mensajes más viejos que el último recibido |
| 4 | u32 | `buttons` (bitmask `ChiakiControllerButton`) |
| 8 | u8 | `l2_state` |
| 9 | u8 | `r2_state` |
| 10 | i16 ×4 | `left_x`, `left_y`, `right_x`, `right_y` |
| 18 | u8 ×2 | reservado |
| 20 | f32 ×6 | `gyro_x/y/z`, `accel_x/y/z` (en 0 hasta v0.3.0) |
| 44 | u8 ×12 | `touches` (2 × {id i8, x u16, y u16, reservado u8}) (en 0 hasta v0.3.0) |

El puente decodifica el mensaje, rellena un `ChiakiControllerState` (partiendo de `chiaki_controller_state_set_idle`) y llama a `chiaki_session_set_controller_state`. Si un mensaje trae una versión desconocida, se descarta y queda en el log.

### DataChannel `control` (JSON)

| Dirección | `type` | Payload | chiaki-lib |
|---|---|---|---|
| N→P | `home` | — | `chiaki_session_go_home` |
| N→P | `keyboard.set_text` / `.accept` / `.reject` | `text?` | `chiaki_session_keyboard_*` |
| P→N | `keyboard.open` / `.text` / `.close` | `text?` | `CHIAKI_EVENT_KEYBOARD_*` |
| P→N | `rumble` | `left`, `right` (0..255) | `CHIAKI_EVENT_RUMBLE` (`ChiakiRumbleEvent`) |
| P→N | `stats` | `bitrateKbps`, `framesLost`, `fps` (cada 1 s) | contadores del puente |

## Código de chiaki-ng que se reutiliza

- `lib/include/chiaki/session.h`: `chiaki_session_request_idr`, `chiaki_session_set_controller_state`, `chiaki_session_go_home`, `chiaki_session_goto_bed`, `chiaki_session_keyboard_*` y los tipos de evento.
- `lib/include/chiaki/controller.h`: `ChiakiControllerState`, `ChiakiControllerButton`.

## Riesgos

- Algunos navegadores esperan SPS/PPS antes de cada IDR. Si el PS5 no los repite, el puente los cachea del primer IDR y los antepone a los siguientes.
- La negociación de `profile-level-id` puede fallar si el PS5 emite un perfil High que el decodificador del navegador no acepta en WebRTC. Se valida en el spike.

## Pruebas

Ver [TS-07](TS-07-pruebas.md): codificación y decodificación del mensaje `input` con vectores compartidos entre TS y C++, y validación del esquema de señalización.
