# FS-06 · Ajustes

| ID | Versión | Estado | TS relacionadas |
|---|---|---|---|
| FS-06 | v0.1.0 | Borrador | [TS-02](../technical/TS-02-puente-cpp.md), [TS-04](../technical/TS-04-frontend-react.md) |

## Objetivo

Elegir la calidad del stream y las preferencias de entrada antes de conectar.

## Actores

Jugador.

## Historias de usuario

- Como jugador, quiero elegir la resolución, los FPS y el bitrate, para ajustar la calidad a mi red.
- Como jugador, quiero ajustar la zona muerta de los sticks.

## Flujo principal

1. El jugador abre Ajustes.
2. Cambia los valores y pulsa "Guardar". La UI confirma "Ajustes guardados".
3. La siguiente sesión usa esos valores.

## Ajustes de v0.1.0

| Ajuste | Opciones | Por defecto |
|---|---|---|
| Resolución | 360p, 540p, 720p, 1080p (`ChiakiVideoResolutionPreset`) | 720p |
| FPS | 30, 60 | 60 |
| Bitrate | Automático según la resolución (`chiaki_connect_video_profile_preset`) o manual, 2.000–50.000 kbps | Automático |
| Zona muerta de los sticks | 0,00–0,30 | 0,08 |
| Mostrar estadísticas al iniciar | Sí, No | No |

## Reglas

- **R1:** los ajustes de stream se guardan en el puente (`settings.json`). La zona muerta y la preferencia del overlay son del navegador y van en `localStorage`.
- **R2:** cambiar un ajuste durante una sesión no la afecta: aplica en la siguiente. El PS5 fija el bitrate al iniciar la sesión ([TS-03](../technical/TS-03-webrtc-y-senalizacion.md)).

## Criterios de aceptación

- **CA-06.1** Dado que el jugador guarda 1080p/60, cuando inicia una sesión, entonces el overlay muestra una resolución de 1920×1080.
- **CA-06.2** Dados unos ajustes guardados, cuando se reinicia el contenedor del puente, entonces se conservan.
- **CA-06.3** Dado un bitrate manual fuera de rango, entonces el formulario lo rechaza.
- **CA-06.4** Dada una zona muerta de 0,20, entonces los movimientos del stick menores que ese radio no mueven nada en el PS5.

## Fuera de alcance

- Códec (llega con HEVC en v0.3.0).
- Perfiles de ajustes por consola.
