# FS-09 · DualSense completo (movimiento, touchpad y retroalimentación)

| ID | Versión | Estado | TS relacionadas |
|---|---|---|---|
| FS-09 | v0.3.0 | Borrador | [TS-10](../technical/TS-10-webhid-dualsense.md) |

## Objetivo

Aprovechar las funciones propias del DualSense por Bluetooth que la Gamepad API no expone: el giroscopio, el acelerómetro, el touchpad, los gatillos adaptativos, la luz y el indicador de jugador.

## Actores

Jugador.

## Historias de usuario

- Como jugador, quiero apuntar con el giroscopio en los juegos que lo usan.
- Como jugador, quiero usar el touchpad como en la consola.
- Como jugador, quiero sentir los gatillos adaptativos y ver la luz del control como manda el juego.

## Flujo principal

1. En la vista de stream, con un DualSense detectado y un navegador con WebHID (Chrome/Edge), la UI ofrece "Activar funciones avanzadas del DualSense".
2. El jugador pulsa el botón y el navegador muestra su selector de dispositivos HID. El jugador elige el DualSense.
3. A partir de ahí:
   - El giroscopio, el acelerómetro y el touchpad (posición y multitáctil) llegan al PS5.
   - Los gatillos adaptativos, la luz y el indicador de jugador responden al juego.
4. La elección se recuerda: en las siguientes visitas, el navegador da acceso al dispositivo sin volver a preguntar.

## Flujos alternos

- **A1:** si el navegador no tiene WebHID (Firefox o Safari), el botón no se muestra y se juega con la Gamepad API como en v0.1.0, sin errores.
- **A2:** si el jugador cancela el selector, se sigue con la Gamepad API.
- **A3:** si el control es un DS4, se habilitan el giroscopio y el touchpad. No hay gatillos adaptativos.

## Reglas

- **R1:** con WebHID activo, los botones y ejes se leen de WebHID y la Gamepad API se ignora para ese control, para no duplicar la entrada.
- **R2:** el rumble se sigue enviando. Con WebHID se envía por el reporte de salida, que es más fiel que `vibrationActuator`.
- **R3:** la vibración háptica HD de voz (el audio háptico del DualSense) no se reproduce por Bluetooth en la web. Se aproxima con rumble.

## Criterios de aceptación

- **CA-09.1** Dado un DualSense con WebHID activo en un juego con apuntado por giroscopio, cuando el jugador mueve el control, entonces la mira se mueve de forma proporcional.
- **CA-09.2** Dado WebHID activo, cuando el jugador desliza el dedo por el touchpad, entonces el PS5 recibe la posición, con dos dedos como máximo.
- **CA-09.3** Dado un juego con gatillos adaptativos, entonces se siente la resistencia en L2/R2.
- **CA-09.4** Dado un juego que cambia la luz del control, entonces la barra de luz cambia al color indicado.
- **CA-09.5** Dado Firefox o Safari, entonces el juego funciona con la Gamepad API y la consola del navegador no muestra errores de WebHID.
- **CA-09.6** Dado un permiso concedido, cuando se recarga la página, entonces WebHID se activa sin volver a mostrar el selector.

## Fuera de alcance

- Audio háptico HD por Bluetooth.
- El altavoz y la toma de audífonos del control.
