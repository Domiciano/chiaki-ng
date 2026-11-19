# FS-05 · Control Bluetooth

| ID | Versión | Estado | TS relacionadas |
|---|---|---|---|
| FS-05 | v0.1.0 | Borrador | [TS-03](../technical/TS-03-webrtc-y-senalizacion.md), [TS-05](../technical/TS-05-mapeo-de-entrada.md) |

## Objetivo

Jugar con un DualSense o un DS4 emparejado por Bluetooth **al equipo del navegador**, sin configuración.

## Actores

Jugador.

## Historias de usuario

- Como jugador, quiero emparejar mi DualSense por Bluetooth con mi computador y que la web lo reconozca, para jugar sin cables.
- Como jugador, quiero sentir la vibración del juego en el control.
- Como jugador sin control a mano, quiero navegar los menús del PS5 con el teclado.

## Flujo principal

1. El jugador empareja el control con su sistema operativo (en el DualSense, PS + Create hasta que parpadee).
2. Abre el stream (FS-04) y pulsa cualquier botón del control. Los navegadores solo exponen el gamepad después de una interacción.
3. La UI muestra "Control conectado: <nombre>".
4. Todo lo que hace el jugador con el control llega al PS5: botones, sticks y gatillos analógicos.
5. Cuando el juego vibra, el control vibra, si el navegador lo soporta.

## Flujos alternos

- **A1:** si no hay control, se muestra "Pulsa un botón del control para activarlo, o usa el teclado". El respaldo de teclado queda activo.
- **A2:** si el control se desconecta a mitad de partida, el PS5 recibe el estado en reposo (nada pulsado) y la UI avisa "Control desconectado".
- **A3:** si el navegador no reporta un mapeo `standard`, la UI avisa "Control no reconocido por completo; algunos botones pueden no funcionar".

## Reglas

- **R1:** el control se usa a través del navegador. No se empareja con el PS5 ni con el puente.
- **R2:** PS + Options y PS + Create mantenidos 1 s son atajos locales y no se envían al PS5 (TS-05).
- **R3:** en v0.1.0 no hay giroscopio, touchpad (salvo el clic), gatillos adaptativos ni luz: llegan en v0.3.0.

## Criterios de aceptación

- **CA-05.1** Dado un DualSense emparejado por Bluetooth a un computador con Chrome, cuando el jugador pulsa un botón en la vista de stream, entonces la UI lo reconoce y puede navegar el menú del PS5 sin configurar nada.
- **CA-05.2** Dado un control reconocido, entonces los 18 botones de la tabla de TS-05 producen la acción esperada en el PS5.
- **CA-05.3** Dado un juego que usa los gatillos analógicos, entonces la presión parcial de L2/R2 se refleja de forma proporcional.
- **CA-05.4** Dado un control en Chrome con `vibrationActuator`, cuando el juego envía rumble, entonces el control vibra.
- **CA-05.5** Dado un control desconectado a mitad de partida, entonces el PS5 no queda con ningún botón o stick pegado.
- **CA-05.6** Dado un DS4 por Bluetooth, entonces CA-05.1 y CA-05.2 también se cumplen.
- **CA-05.7** Sin control, las flechas, `Enter`, `Backspace`, `Esc` y `Tab` permiten navegar el menú del PS5.
- **CA-05.8** Dado un control reconocido, la latencia de entrada agregada por el navegador y el puente es menor de 10 ms en LAN (medida en el spike con marcas de tiempo en el puente).

## Fuera de alcance

- Varios controles o jugadores locales.
- Remapeo de botones.
- Controles de Xbox u otros (podrían funcionar por el mapeo `standard`, pero no se garantizan).
