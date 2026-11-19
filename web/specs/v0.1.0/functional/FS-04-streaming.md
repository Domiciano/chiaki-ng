# FS-04 · Streaming (jugar)

| ID | Versión | Estado | TS relacionadas |
|---|---|---|---|
| FS-04 | v0.1.0 | Borrador | [TS-02](../technical/TS-02-puente-cpp.md), [TS-03](../technical/TS-03-webrtc-y-senalizacion.md), [TS-04](../technical/TS-04-frontend-react.md), [TS-06](../technical/TS-06-despliegue-docker-compose.md) |

## Objetivo

Ver y oír el PS5 en el navegador con baja latencia, y salir de la sesión de forma limpia.

## Actores

Jugador.

## Historias de usuario

- Como jugador, quiero conectarme a mi consola y verla a pantalla completa con sonido, para jugar desde el navegador.
- Como jugador, quiero ver estadísticas (FPS, latencia, pérdidas), para entender si la red va bien.
- Como jugador, quiero escribir en el teclado del PS5 desde mi teclado, para no tener que hacerlo con el control.
- Como jugador, quiero salir del stream y, si quiero, poner la consola en reposo.

## Flujo principal

1. En Consolas, el jugador pulsa "Conectar" en una consola registrada y lista.
2. La UI muestra "Conectando…".
3. Empieza el video y el audio. El jugador pulsa "Jugar" para entrar en pantalla completa y habilitar el sonido.
4. El jugador juega con el control (FS-05).
5. Para salir: el atajo PS + Create, o el botón "Salir". La UI pregunta "¿Salir?", con las opciones "Salir" y "Salir y apagar la consola".
6. La sesión termina y se vuelve a Consolas.

## Flujos alternos

- **A1:** si la consola está en reposo, se ofrece "Despertar y conectar".
- **A2:** si el PS5 pide el PIN de inicio de sesión del usuario, aparece un diálogo para ingresarlo. Con un PIN incorrecto, el diálogo vuelve a aparecer con un aviso.
- **A3:** si la consola ya está en uso por otro cliente de Remote Play, se muestra el motivo que devuelve la consola (`chiaki_quit_reason_string`).
- **A4:** si la sesión WebRTC no logra conectar en 15 s, se muestra "No se pudo establecer el video. Revisa `BRIDGE_ANNOUNCE_IP` y el rango UDP del firewall."
- **A5:** si se recarga o cierra la pestaña, el puente detecta el cierre y detiene la sesión con el PS5 en menos de 10 s.
- **A6:** si el PS5 abre su teclado en pantalla, aparece un campo de texto. Lo que escribe el jugador se envía al PS5, y se puede aceptar o cancelar.

## Reglas

- **R1:** una sola sesión de stream a la vez. Un segundo intento mientras hay una activa muestra "Ya hay una sesión activa".
- **R2:** la resolución, los FPS y el bitrate salen de Ajustes (FS-06).
- **R3:** el overlay de estadísticas se muestra y oculta con PS + Options mantenidos, o con la tecla `F2`.

## Criterios de aceptación

- **CA-04.1** Dado un PS5 registrado y listo en la LAN, cuando el jugador pulsa "Conectar" en Chrome, entonces el video aparece en menos de 10 s.
- **CA-04.2** Dada una sesión a 720p60 por LAN en Chrome, entonces el overlay muestra al menos 58 FPS sostenidos durante 10 minutos.
- **CA-04.3** Dada una sesión a 1080p60 por LAN cableada, entonces el overlay muestra al menos 58 FPS sostenidos durante 10 minutos.
- **CA-04.4** Dada una sesión activa, entonces el audio se escucha sin cortes audibles y sin desfase perceptible con el video (menos de 80 ms).
- **CA-04.5** Dada una pérdida de paquetes del 1 % en la red, entonces el video se recupera y no queda congelado más de 1 s seguido.
- **CA-04.6** Dada una sesión activa, cuando el jugador elige "Salir y apagar la consola", entonces el PS5 pasa a reposo.
- **CA-04.7** Dada una sesión activa, cuando se cierra la pestaña, entonces el puente libera la sesión y otra conexión es posible en menos de 10 s.
- **CA-04.8** Dado que el PS5 abre su teclado, cuando el jugador escribe "hola" y acepta, entonces el texto aparece en el campo del PS5.
- **CA-04.9** Dada una sesión activa, entonces el overlay muestra FPS, frames perdidos, bitrate, RTT y latencia estimada.
- **CA-04.10** Dada una sesión activa en Edge y en Safari, entonces CA-04.1 y CA-04.2 también se cumplen.

## Fuera de alcance

- HDR y 4K: v0.3.0 (HEVC).
- Micrófono: v0.3.0.
- Varias sesiones simultáneas.
