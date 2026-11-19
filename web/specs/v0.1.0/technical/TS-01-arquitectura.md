# TS-01 · Arquitectura

| ID | Versión | Estado | FS que cubre |
|---|---|---|---|
| TS-01 | v0.1.0 | Borrador | [FS-01](../functional/FS-01-acceso-y-seguridad.md), [FS-02](../functional/FS-02-descubrimiento-consolas.md), [FS-03](../functional/FS-03-registro-consola.md), [FS-04](../functional/FS-04-streaming.md), [FS-05](../functional/FS-05-control-bluetooth.md), [FS-06](../functional/FS-06-ajustes.md) |

## Contexto

[TS-00](TS-00-viabilidad.md) concluye que hace falta un puente nativo entre el navegador y el PS5.

## Diseño

```
┌────────────── Host Linux en la LAN del PS5 (docker-compose) ──────────────┐
│                                                                           │
│  ┌── web (nginx:alpine) ──┐   WebSocket   ┌── bridge (C++) ─────────────┐ │
│  │ HTTPS :443             │──/signal────▶ │ señalización :8090          │ │
│  │ auth_basic             │               │ chiaki-lib (sesión, regist, │ │
│  │ estáticos React (dist) │               │  discovery)                 │ │
│  └────────────────────────┘               │ libdatachannel (WebRTC)     │ │
│                                           │ network_mode: host          │ │
│                                           └──────┬─────────────▲────────┘ │
└──────────────────────────────────────────────────┼─────────────┼──────────┘
                                                   │TCP 9295     │ UDP WebRTC
                                                   │UDP 9296/9297│ 50000-50100
                                                   ▼             │
                                                  PS5        Navegador
```

**Componentes:**

| Componente | Responsabilidad | Spec |
|---|---|---|
| Frontend (`web/frontend/`) | UI, Gamepad API, `RTCPeerConnection`, reproducción | [TS-04](TS-04-frontend-react.md), [TS-05](TS-05-mapeo-de-entrada.md) |
| nginx (contenedor `web`) | TLS, autenticación, estáticos, proxy de `/signal` | [TS-06](TS-06-despliegue-docker-compose.md) |
| Puente (`web/bridge/`) | Protocolo con el PS5, persistencia de consolas, WebRTC | [TS-02](TS-02-puente-cpp.md), [TS-03](TS-03-webrtc-y-senalizacion.md) |

**Flujos:**

1. **Control** (WebSocket `/signal`, por nginx): listar consolas, despertar, registrar, iniciar o detener una sesión, intercambiar SDP e ICE, y recibir los eventos que empuja el puente.
2. **Media e input** (WebRTC, UDP directo entre el navegador y el puente, sin pasar por nginx):
   - Video y audio del PS5 al navegador.
   - Controles del navegador al PS5.
   - Eventos de rumble del PS5 al navegador.

**Perfiles de acceso al navegador:**

| Perfil | Cómo llega el navegador | Uso |
|---|---|---|
| **Tailscale (recomendado, uso personal)** | El host y los clientes (celular, tablet, portátil) están en la misma tailnet. El navegador abre `https://<host>.<tailnet>.ts.net`, y el media WebRTC viaja por el túnel hacia la IP de Tailscale del host (`100.x.y.z`). | Acceso desde la LAN y desde fuera de casa, sin abrir puertos ni usar TURN. |
| LAN | El navegador está en la misma red local que el host. | Pruebas en casa. |

Con el perfil Tailscale, el puente siempre habla con el PS5 por LAN. El acceso remoto no depende de PSN ni del hole punching de Sony, y por eso la v0.2.0 (remoto vía PSN) pasa a ser opcional ([ROADMAP](../../ROADMAP.md)). El detalle de despliegue está en [TS-06](TS-06-despliegue-docker-compose.md#perfil-tailscale-recomendado).

**Host de referencia:** un PC con Windows en la LAN del PS5 (ASUS TUF Gaming F15), por cable al router donde está la consola y con encendido programado. Las opciones para correr el puente en Windows están en [TS-00](TS-00-viabilidad.md#decisión-de-host).

**Restricciones:**

- El puente debe estar en la **misma LAN que el PS5**, porque el descubrimiento es por broadcast UDP. Por eso usa `network_mode: host`.
- Una sola sesión de stream a la vez. Es de uso personal: no hay escalado.
- El estado persistente (consolas registradas y, desde v0.2.0, los tokens PSN) vive **solo en el puente**, en un volumen. El navegador no guarda secretos.

## Contratos

Los define [TS-03](TS-03-webrtc-y-senalizacion.md).

## Riesgos

- Docker Desktop (macOS/Windows) no pasa el broadcast LAN con `network_mode: host`. La alternativa documentada en TS-06 es `macvlan`.
- Licencia AGPL-3.0: el puente es un servicio de red, así que la UI debe ofrecer el enlace al código fuente ([FS-01](../functional/FS-01-acceso-y-seguridad.md)).
