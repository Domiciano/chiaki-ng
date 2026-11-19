# FS-03 · Registro de una consola

| ID | Versión | Estado | TS relacionadas |
|---|---|---|---|
| FS-03 | v0.1.0 | Borrador | [TS-02](../technical/TS-02-puente-cpp.md), [TS-03](../technical/TS-03-webrtc-y-senalizacion.md), [TS-04](../technical/TS-04-frontend-react.md) |

## Objetivo

Emparejar una PS5 con el puente una sola vez, con el PIN de Remote Play, para poder conectarse después.

## Actores

Jugador.

## Historias de usuario

- Como jugador, quiero registrar mi PS5 con el PIN que muestra la consola, para conectarme sin repetir el proceso.

## Flujo principal

1. En el PS5: Ajustes → Sistema → Uso a distancia → Vincular dispositivo. La consola muestra un PIN de 8 dígitos.
2. En la web, el jugador pulsa "Registrar" en la consola (o abre `/register`), con la IP ya rellena.
3. El jugador ingresa el PIN y su **ID de cuenta PSN** (base64).
4. Pulsa "Registrar". La UI muestra "Registrando…".
5. El puente completa el registro y guarda la consola. La UI muestra "Consola registrada" y vuelve a Consolas, donde la consola aparece como registrada.

## Flujos alternos

- **A1:** con un PIN incorrecto o vencido, se muestra "El PIN no es válido o expiró. Genera uno nuevo en la consola." y se puede reintentar.
- **A2:** si la consola no responde, se muestra "La consola no respondió. Verifica que esté encendida y en la misma red."
- **A3:** si el ID de cuenta no es un base64 de 8 bytes, se rechaza en el formulario antes de enviar.

## Reglas

- **R1:** en v0.1.0 el ID de cuenta PSN se ingresa a mano. La UI enlaza la guía de cómo obtenerlo (`docs/setup/configuration.md`, o `scripts/psn-account-id.py` del repo). Desde v0.2.0 se obtiene solo con el login de Sony ([FS-07](../../v0.2.0/functional/FS-07-login-psn.md)).
- **R2:** el PIN no se guarda ni se registra en los logs.
- **R3:** el ID de cuenta se guarda junto con la consola, para usarlo al conectar.

## Criterios de aceptación

- **CA-03.1** Dado un PS5 mostrando un PIN válido, cuando el jugador lo ingresa con un ID de cuenta válido, entonces el registro termina en menos de 15 s y la consola queda marcada como registrada.
- **CA-03.2** Dado un PIN incorrecto, entonces se muestra el mensaje de A1 y no se guarda nada.
- **CA-03.3** Dado un ID de cuenta con un formato inválido, entonces el botón "Registrar" queda deshabilitado con el error junto al campo.
- **CA-03.4** Dada una consola registrada, cuando se reinicia el contenedor del puente, entonces sigue registrada (la persistencia está en el volumen).
- **CA-03.5** Dado un registro, entonces el PIN no aparece en los logs del puente ni de nginx.

## Fuera de alcance

- Registro por internet (remoto): v0.2.0.
