# TS-04 · Frontend React (`web/frontend/`)

| ID | Versión | Estado | FS que cubre |
|---|---|---|---|
| TS-04 | v0.1.0 | Borrador | [FS-01](../functional/FS-01-acceso-y-seguridad.md), [FS-02](../functional/FS-02-descubrimiento-consolas.md), [FS-03](../functional/FS-03-registro-consola.md), [FS-04](../functional/FS-04-streaming.md), [FS-06](../functional/FS-06-ajustes.md) |

## Contexto

Es una SPA estática que se compila a `dist/` y sirve nginx. No guarda secretos: todo el estado persistente vive en el puente.

## Diseño

**Stack:**

- Vite, React 18 y TypeScript en modo `strict`.
- `react-router` con estas rutas:

| Ruta | Vista | FS |
|---|---|---|
| `/` | Consolas: lista, estado, despertar, conectar, olvidar | FS-02 |
| `/register` | Registro con PIN | FS-03 |
| `/stream/:hostId` | Stream a pantalla completa | FS-04, FS-05 |
| `/settings` | Ajustes | FS-06 |
| `/about` | Versión y enlace al código fuente (AGPL) | FS-01 |

**Estructura:**

```
web/frontend/src/
├── main.tsx, App.tsx
├── signal/        SignalClient.ts (WebSocket con reconexión exponencial, request/response por id, eventos)
├── rtc/           StreamPeer.ts (RTCPeerConnection, respuesta a la oferta, jitterBufferTarget, getStats)
├── input/         GamepadPoller.ts, mapping.ts, inputCodec.ts, keyboardFallback.ts (ver TS-05)
├── views/         HostsView, RegisterView, StreamView, SettingsView, AboutView
├── components/    HostCard, StatsOverlay, PinDialog, OnScreenKeyboardDialog, ...
└── types/         protocol.ts (tipos de los mensajes de TS-03, fuente única en el front)
```

**Stream (`StreamView`):**

- `<video autoplay playsinline>` con el `MediaStream` remoto. El audio sale por el mismo elemento.
- Pantalla completa con `requestFullscreen()` al pulsar "Jugar". El gesto del usuario sirve también para habilitar el audio.
- `StatsOverlay`, que se alterna con una tecla o con el combo PS + Options:
  - FPS y frames perdidos (`inbound-rtp`).
  - `jitterBufferDelay`.
  - RTT (`candidate-pair`).
  - Bitrate.
  - Latencia estimada.
- Si el PS5 abre su teclado: `OnScreenKeyboardDialog` con el texto, que responde con `keyboard.*`.
- Si el PS5 pide el PIN de login: `PinDialog`.
- Al salir: `session.stop`, con la opción "Apagar la consola" (`sleep: true`).

**Estado:** solo hooks de React y un contexto para `SignalClient`. Sin librería de estado global.

**Textos de la UI:** en español. Los identificadores de código, en inglés.

## Contratos

Los tipos de `types/protocol.ts` reflejan exactamente las tablas de [TS-03](TS-03-webrtc-y-senalizacion.md).

## Riesgos

- Safari exige `playsinline` y un gesto del usuario para el audio.
- La pantalla completa en iOS es limitada, aunque iOS no es un objetivo de v0.1.0.

## Pruebas

`vitest` para `SignalClient` (con un WebSocket simulado) y para `inputCodec`. Más una prueba manual en Chrome, Edge y Safari ([TS-07](TS-07-pruebas.md)).
