# TS-12 · HEVC por WebRTC

| ID | Versión | Estado | FS que cubre |
|---|---|---|---|
| TS-12 | v0.3.0 | Borrador | [FS-11](../functional/FS-11-calidad-hevc.md) |

## Contexto

El PS5 puede emitir H.265 (`CHIAKI_CODEC_H265`, `CHIAKI_CODEC_H265_HDR` en `lib/include/chiaki/common.h`). En v0.1.0 se fuerza H.264 por compatibilidad ([TS-03](../../v0.1.0/technical/TS-03-webrtc-y-senalizacion.md), decisión D4). Esta spec habilita H.265 sin transcodificar, en los navegadores que lo decodifican en WebRTC.

## Diseño

**Detección (navegador):**
- `RTCRtpReceiver.getCapabilities('video').codecs` contiene `mimeType === 'video/H265'`.
- Se envía en `session.start` como `codecs: ["H265","H264"]`, en orden de preferencia.

**Puente:**
- Si `H265` está en la lista y el ajuste lo permite, se usa `video_profile.codec = CHIAKI_CODEC_H265`.
- Se crea el track con `H265RtpPacketizer` (libdatachannel) y la oferta SDP `video/H265`.
- Si el navegador no acepta el códec en la respuesta SDP (no aparece en `sdp.answer`), se cierra el peer y se reinicia la sesión con H.264 (FS-11 A1).
- PLI → `chiaki_session_request_idr` igual que con H.264. Se cachean VPS/SPS/PPS para anteponerlos a cada IDR si hace falta.

**Ajustes:** nuevo ajuste `codec: "auto" | "h264" | "h265"` en `settings.json` (FS-06).

**HDR:** `CHIAKI_CODEC_H265_HDR` queda fuera de esta versión. Exige que el navegador reproduzca HDR desde WebRTC y una pantalla compatible, y se evaluará aparte.

## Código de chiaki-ng que se reutiliza

- `lib/include/chiaki/common.h`: `ChiakiCodec`, `chiaki_codec_is_h265`.
- `gui/src/streamsession.cpp`: referencia de cómo el escritorio elige el códec.

## Riesgos

- El soporte de H.265 en WebRTC depende del hardware y del sistema operativo, aunque el navegador lo anuncie. La recuperación a H.264 (CA-11.4) es obligatoria.

## Pruebas

- Manual: Chrome con decodificación HEVC por hardware, Safari reciente y un navegador sin soporte (recuperación a H.264).
