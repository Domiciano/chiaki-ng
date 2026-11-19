# FS-02 · Descubrimiento de consolas

| ID | Versión | Estado | TS relacionadas |
|---|---|---|---|
| FS-02 | v0.1.0 | Borrador | [TS-02](../technical/TS-02-puente-cpp.md), [TS-03](../technical/TS-03-webrtc-y-senalizacion.md), [TS-04](../technical/TS-04-frontend-react.md) |

## Objetivo

Ver las PS5 de la red local, su estado, y poder despertarlas.

## Actores

Jugador.

## Historias de usuario

- Como jugador, quiero ver mis consolas de la LAN con su estado (encendida, en reposo), para saber si puedo conectarme.
- Como jugador, quiero despertar una consola en reposo desde la web, para no tener que levantarme.
- Como jugador, quiero olvidar una consola registrada, para dejar de verla.

## Flujo principal

1. El jugador abre la vista Consolas.
2. La lista muestra cada consola encontrada:
   - Nombre.
   - IP.
   - Estado: "Lista", "En reposo" o "Desconocido".
   - Si está registrada.
3. La lista se actualiza sola cuando cambia un estado.
4. Según el estado de la consola, el jugador pulsa "Conectar" (FS-04), "Despertar" o "Registrar" (FS-03).

## Flujos alternos

- **A1:** si no aparece ninguna consola, se muestra el mensaje "No se encontraron consolas en la red". Incluye una ayuda: misma LAN, Remote Play activado en el PS5, red `host` del puente.
- **A2:** una consola registrada que no aparece en el descubrimiento se lista igual, en estado "Desconocido", con su última IP.

## Reglas

- **R1:** el descubrimiento lo hace el puente. El navegador nunca envía paquetes al PS5.
- **R2:** "Despertar" solo está disponible para consolas registradas, porque usa la clave de registro.
- **R3:** en v0.1.0 solo se listan PS5.

## Criterios de aceptación

- **CA-02.1** Dado un PS5 encendido en la misma LAN que el puente, cuando el jugador abre Consolas, entonces la consola aparece con estado "Lista" en menos de 5 s.
- **CA-02.2** Dado un PS5 que pasa de encendido a reposo, entonces su estado en la UI cambia a "En reposo" en menos de 10 s, sin recargar la página.
- **CA-02.3** Dado un PS5 registrado en reposo, cuando el jugador pulsa "Despertar", entonces la consola enciende y su estado pasa a "Lista" en menos de 60 s.
- **CA-02.4** Dado un PS5 registrado, cuando el jugador pulsa "Olvidar" y confirma, entonces desaparece de las registradas y su clave se borra del puente.
- **CA-02.5** Dada una red sin consolas, entonces se muestra el mensaje de A1.

## Fuera de alcance

- PS4.
- Agregar una consola por IP manual: queda para una versión futura.
