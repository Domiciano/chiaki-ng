# TS-11 · Micrófono por WebRTC

| ID | Versión | Estado | FS que cubre |
|---|---|---|---|
| TS-11 | v0.3.0 | Borrador | [FS-10](../functional/FS-10-microfono.md) |

## Contexto

`chiaki-lib` ya envía el audio del micrófono al PS5 (`lib/src/audiosender.c`, `lib/src/opusencoder.c`). El escritorio captura con SDL o Qt, codifica en Opus con los parámetros que pide la sesión y lo envía. En la web, la captura la hace el navegador.

## Diseño

**Navegador:**

- `getUserMedia({ audio: { echoCancellation: true, noiseSuppression: true, autoGainControl: true } })`.
- El track se agrega a la `RTCPeerConnection` existente como un transceiver de audio `sendonly` hacia el puente. En v0.1.0 ese transceiver ya se negocia como `inactive`, así que activarlo no requiere renegociar.
- Silenciar es `track.enabled = false`. Además se envía `mic.mute` por `control`.

**Puente:**

- Se recibe el RTP Opus del navegador con el `onFrame` del track de libdatachannel.
- **Camino base:** se decodifica con libopus a PCM 48 kHz y se vuelve a codificar con `chiaki_opus_encoder_frame` (`lib/include/chiaki/opusencoder.h`), después de `chiaki_opus_encoder_header`. Así se garantiza el tamaño de trama, los canales y el bitrate que espera el PS5.
- **Optimización opcional:** si en el spike de esta versión se comprueba que el tamaño de trama, los canales y la frecuencia del navegador coinciden con los del encoder de chiaki, el paquete pasa directo a `chiaki_audio_sender_opus_data` sin transcodificar.
- Al primer `mic.unmute`: `chiaki_session_connect_microphone`. Luego, `chiaki_session_toggle_microphone(session, muted)`.
- Esto requiere `CHIAKI_LIB_ENABLE_OPUS=ON` en el build del puente ([TS-02](../../v0.1.0/technical/TS-02-puente-cpp.md)), y agregar `libopus-dev`/`libopus0` al Dockerfile (ya están).

## Contratos

| Canal | Dirección | `type` | Payload |
|---|---|---|---|
| `control` | N→P | `mic.mute` / `mic.unmute` | — |
| `control` | P→N | `mic.state` | `connected`, `muted` |

## Código de chiaki-ng que se reutiliza

`lib/src/opusencoder.c`, `lib/src/audiosender.c`, y `chiaki_session_connect_microphone` y `chiaki_session_toggle_microphone` (`lib/include/chiaki/session.h`). Como referencia de uso, `gui/src/streamsession.cpp`.

## Riesgos

- Transcodificar agrega unos 10–20 ms en el micrófono. Es aceptable para voz.
- Con WebHID, el botón de silencio del DualSense debe sincronizarse con el estado (FS-09).

## Pruebas

Manual: chat de grupo con un segundo jugador, silencio y prueba de eco con altavoces.
