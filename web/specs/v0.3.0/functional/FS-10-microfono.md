# FS-10 · Micrófono

| ID | Versión | Estado | TS relacionadas |
|---|---|---|---|
| FS-10 | v0.3.0 | Borrador | [TS-11](../technical/TS-11-microfono-webrtc.md) |

## Objetivo

Hablar en el chat de voz del PS5 usando el micrófono del equipo del navegador.

## Actores

Jugador.

## Historias de usuario

- Como jugador, quiero activar mi micrófono durante el stream para hablar con mi grupo.
- Como jugador, quiero silenciarlo rápido.

## Flujo principal

1. En el stream, el jugador pulsa el botón de micrófono del overlay o el botón de silencio del DualSense, si WebHID está activo.
2. La primera vez, el navegador pide permiso para usar el micrófono.
3. El indicador pasa a "Micrófono activo". Los demás jugadores lo oyen.
4. Al pulsar de nuevo, el micrófono se silencia y el indicador pasa a "Silenciado".

## Flujos alternos

- **A1:** si el jugador niega el permiso, se muestra "Sin permiso de micrófono" con instrucciones para habilitarlo en el navegador.
- **A2:** si no hay micrófono, el botón aparece deshabilitado.

## Reglas

- **R1:** el micrófono empieza silenciado en cada sesión.
- **R2:** se usa la cancelación de eco del navegador (`echoCancellation: true`), para que el audio del juego no vuelva al chat.

## Criterios de aceptación

- **CA-10.1** Dado el permiso concedido, cuando el jugador activa el micrófono, entonces otro jugador en el chat de grupo del PS5 lo oye de forma inteligible.
- **CA-10.2** Dado el micrófono activo, cuando el jugador lo silencia, entonces deja de oírse en menos de 500 ms.
- **CA-10.3** Dados unos altavoces con el audio del juego, entonces los demás no oyen el eco del juego.
- **CA-10.4** Dado un permiso negado, entonces se muestra el mensaje de A1 y el stream sigue sin cambios.

## Fuera de alcance

- Selección del dispositivo de entrada desde la UI (se usa el predeterminado del navegador).
