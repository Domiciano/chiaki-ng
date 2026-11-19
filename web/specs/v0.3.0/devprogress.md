# devprogress · chiaki-ng Web v0.3.0 (DualSense completo, micrófono y HEVC)

Alcance: [ROADMAP](../ROADMAP.md#v030--dualsense-completo-micrófono-y-hevc). Specs: FS-09, FS-10, FS-11, TS-10, TS-11, TS-12. Depende de que v0.2.0 esté cerrada.

## Tareas

| ID tarea | Spec | Descripción | Estado | Fecha | Notas |
|---|---|---|---|---|---|
| T-301 | TS-10 | Conexión WebHID (`requestDevice` / `getDevices`) con los filtros de los controles Sony | PENDIENTE | | CA-09.6 |
| T-302 | TS-10 | Parseo del reporte de entrada BT 0x31 con calibración: giroscopio, acelerómetro y touchpad hacia `input` v1 | PENDIENTE | | CA-09.1, CA-09.2 |
| T-303 | TS-10 | Reporte de salida BT 0x31 con CRC32: rumble, gatillos, luz, LEDs de jugador y silencio | PENDIENTE | | CA-09.3, CA-09.4 |
| T-304 | TS-10 | Eventos `trigger_effects`, `led_color`, `player_index`, `*_intensity` y `motion_reset` en el puente | PENDIENTE | | |
| T-305 | TS-10 | Aproximación del audio háptico a rumble | PENDIENTE | | |
| T-306 | TS-10, FS-09 | Degradación a la Gamepad API sin WebHID, y soporte DS4 | PENDIENTE | | CA-09.5 |
| T-307 | TS-11 | Spike: comparar los parámetros Opus del navegador con los del encoder de chiaki | PENDIENTE | | |
| T-308 | TS-11, FS-10 | Micrófono: transceiver de audio, decodificación y recodificación en el puente, `connect_microphone`/`toggle_microphone` | PENDIENTE | | CA-10.1, CA-10.2, CA-10.3 |
| T-309 | FS-10 | UI del micrófono (botón, indicador, permisos) | PENDIENTE | | CA-10.4 |
| T-310 | TS-12 | Detección de H.265, lista de códecs en `session.start` y ajuste `codec` | PENDIENTE | | CA-11.1, CA-11.2 |
| T-311 | TS-12 | `H265RtpPacketizer`, caché VPS/SPS/PPS y recuperación a H.264 | PENDIENTE | | CA-11.3, CA-11.4 |
| T-312 | — | Pruebas manuales: juego con giroscopio, juego con gatillos, chat de voz, HEVC en 2 navegadores | PENDIENTE | | |

## Trazabilidad de criterios de aceptación

| CA | Tarea(s) |
|---|---|
| CA-09.1, CA-09.2 | T-302 |
| CA-09.3, CA-09.4 | T-303, T-304 |
| CA-09.5 | T-306 |
| CA-09.6 | T-301 |
| CA-10.1..CA-10.3 | T-308 |
| CA-10.4 | T-309 |
| CA-11.1, CA-11.2 | T-310 |
| CA-11.3, CA-11.4 | T-311 |

## Criterios de salida de la versión

- [ ] Todos los CA de FS-09, FS-10 y FS-11 verificados.
- [ ] Sin WebHID, el juego funciona igual que en v0.1.0.
- [ ] Tag `web-v0.3.0` creado y `web/VERSION` actualizado.

## Bitácora

| Fecha | Entrada |
|---|---|
| 2026-11-19 | Se crean las specs de v0.3.0 (FS-09..FS-11, TS-10..TS-12). |
