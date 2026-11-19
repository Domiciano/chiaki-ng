# FS-11 · Calidad HEVC

| ID | Versión | Estado | TS relacionadas |
|---|---|---|---|
| FS-11 | v0.3.0 | Borrador | [TS-12](../technical/TS-12-hevc-webrtc.md) |

## Objetivo

Más calidad de imagen con el mismo bitrate, y la puerta a 4K y HDR, cuando el navegador soporta H.265 en WebRTC.

## Actores

Jugador.

## Historias de usuario

- Como jugador con un navegador compatible, quiero usar HEVC para ver mejor la imagen sin subir el bitrate.

## Flujo principal

1. En Ajustes aparece "Códec de video", con las opciones "Automático" (por defecto), "H.264" y "H.265 (HEVC)". La opción H.265 solo aparece si el navegador lo soporta.
2. En "Automático", si el navegador soporta H.265, se usa H.265. Si no, H.264.
3. El overlay indica el códec en uso.

## Flujos alternos

- **A1:** si se eligió H.265 y la negociación falla, la sesión reintenta con H.264 y avisa "Tu navegador no pudo usar HEVC; se usa H.264".

## Reglas

- **R1:** el soporte se detecta en el navegador con `RTCRtpReceiver.getCapabilities('video')`, buscando `video/H265`.
- **R2:** la resolución 4K y el HDR se exponen solo con H.265, en una versión posterior a v0.3.0. Aquí solo se habilita el códec.

## Criterios de aceptación

- **CA-11.1** Dado un navegador con H.265 en WebRTC y el ajuste en "Automático", cuando se inicia el stream, entonces el overlay muestra "H.265".
- **CA-11.2** Dado un navegador sin H.265, entonces la opción H.265 no aparece y el stream usa H.264.
- **CA-11.3** Dado el mismo bitrate, entonces la sesión H.265 muestra al menos 58 FPS sostenidos a 1080p60, igual que H.264.
- **CA-11.4** Dado un fallo de negociación H.265, entonces la sesión se recupera en H.264 en menos de 15 s.

## Fuera de alcance

- 4K y HDR en la interfaz (versión futura).
