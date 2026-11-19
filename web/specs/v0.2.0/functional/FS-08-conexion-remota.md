# FS-08 · Conexión remota vía PSN

| ID | Versión | Estado | TS relacionadas |
|---|---|---|---|
| FS-08 | v0.2.0 | Borrador | [TS-08](../technical/TS-08-auth-psn.md), [TS-09](../technical/TS-09-holepunch-y-turn.md) |

## Objetivo

Jugar en el PS5 de casa desde fuera de la red local.

## Actores

- Jugador.
- Servidores de PSN (señalización del hole-punch).

## Contexto de despliegue

Hay dos escenarios, y v0.2.0 cubre ambos:

| Escenario | Puente | Navegador | Qué tramo es remoto |
|---|---|---|---|
| E1 | En la LAN del PS5 | Fuera de casa (móvil, otra red) | Navegador ⇄ puente (WebRTC con TURN) |
| E2 | Fuera de la LAN del PS5 (p. ej. un VPS o la red de un amigo) | Junto al puente o en cualquier lado | Puente ⇄ PS5 (hole-punch PSN) |

## Historias de usuario

- Como jugador fuera de casa, quiero abrir mi web y jugar en el PS5 de mi casa.
- Como jugador, quiero ver las consolas de mi cuenta PSN aunque no estén en la red del puente.

## Flujo principal

1. Con la cuenta vinculada (FS-07), la vista Consolas muestra una sección "Consolas de tu cuenta PSN", con el nombre y el estado que devuelve PSN.
2. El jugador pulsa "Conectar (remoto)" en una consola.
3. Si la consola no está registrada con el puente, el registro se hace por internet: el jugador ingresa el PIN de vinculación igual que en FS-03, sin necesidad de estar en la LAN.
4. La UI muestra "Conectando por internet…" y, más adelante, el stream como en FS-04.

## Flujos alternos

- **A1:** si el hole-punch falla (NAT estricto en ambos lados), se muestra "No se pudo abrir una conexión directa con la consola. Revisa UPnP en tu router." con el detalle técnico desplegable.
- **A2:** en E1, si la conexión WebRTC directa falla, se usa TURN automáticamente. El overlay indica "Relé TURN".
- **A3:** si la consola está en reposo, se intenta despertarla por PSN antes de conectar, como hace el cliente de escritorio.

## Reglas

- **R1:** el bitrate por defecto en remoto es menor que en LAN (720p/60 a 10.000 kbps), y es configurable aparte en Ajustes.
- **R2:** TURN solo se despliega si se activa el perfil `remote` del compose (TS-09).

## Criterios de aceptación

- **CA-08.1** Dada una cuenta vinculada, cuando el jugador abre Consolas, entonces ve sus PS5 de PSN con su estado en menos de 10 s.
- **CA-08.2** E2: dado el puente fuera de la LAN del PS5, cuando el jugador conecta en remoto, entonces el video aparece en menos de 30 s.
- **CA-08.3** E1: dado el navegador en una red móvil y el puente en casa con el perfil `remote` activo, entonces el stream funciona aunque la conexión directa falle (por TURN).
- **CA-08.4** Dada una sesión remota estable, entonces se mantiene 30 minutos sin cortes de más de 2 s.
- **CA-08.5** Dado un fallo de hole-punch, entonces se muestra el mensaje de A1 y el puente queda libre para otro intento.
- **CA-08.6** Dada una consola no registrada, cuando el jugador la registra en remoto con el PIN, entonces queda registrada sin estar en la misma LAN.

## Fuera de alcance

- Servidores de relé propios para el tramo puente ⇄ PS5 (lo maneja el hole-punch de PSN).
