# Espina de arquitectura · chiaki web en casa

```
Chiaki web (navegador) <> Tailscale <> TUF F15 (host en casa) <> PS5
```

Este es el resumen de cómo queda desplegada chiaki web para uso personal. El detalle está en las specs ([`specs/`](specs/README.md)), en especial TS-01, TS-03, TS-06 y TS-00 de v0.1.0. Si este documento y una spec no coinciden, manda la spec, y este archivo se corrige.

## 1. Vista general

```
FUERA DE CASA (trabajo / datos móviles)
  DualSense ──Bluetooth──▶ Celular / tablet / portátil
                             ├─ Navegador: UI chiaki web + Gamepad API + RTCPeerConnection
                             └─ App Tailscale
                                   ║
                                   ║  Tailscale: túnel WireGuard
                                   ║  (directo, o relé DERP si no hay ruta directa)
                                   ║
CASA                               ▼
  TUF F15 (Windows, Ethernet al router Movistar, 100.x.y.z = <tuf>.<tailnet>.ts.net)
    ├─ :443 HTTPS ───────▶ web (nginx): estáticos de la UI + proxy /signal
    │                                        │ WebSocket (localhost:8090)
    │                                        ▼
    └─ :50000-50100 UDP ─▶ bridge (C++ + chiaki-lib), red del host
                                             │ TCP 9295 · UDP 9296/9297 · broadcast
                                             ▼
                            Router Movistar ─── PS5
```

## 2. Invariantes (lo que no cambia)

1. **El puente vive en la LAN del PS5.** El TUF F15 está por cable en el mismo router que la consola (Movistar). El puente nunca habla con el PS5 a través de internet, así que no usa PSN ni el hole punching de Sony.
2. **El navegador nunca habla con el PS5.** Solo habla con el TUF: la UI y la señalización por HTTPS/WSS, y el media por WebRTC.
3. **Todo el acceso desde fuera de casa pasa por Tailscale.** No se abren puertos en el router, no hay TURN y no hay nada expuesto a internet. Solo los dispositivos de la tailnet alcanzan el host.
4. **Video y audio pasan sin transcodificar.** H.264 y Opus del PS5 se reempaquetan en RTP; el puente no decodifica.
5. **Los secretos viven solo en el host.** La `regist_key`, `morning` y, si alguna vez se usan, los tokens PSN, están en el volumen del puente. El navegador no guarda ningún secreto.
6. **Una sola sesión de stream a la vez.** Uso personal, sin escalado.
7. **HTTPS siempre**, con el certificado `*.ts.net` que emite Tailscale. Es obligatorio para la Gamepad API, WebHID y el micrófono.

## 3. Capas y responsabilidades

| Capa | Dónde corre | Responsabilidad | Spec |
|---|---|---|---|
| UI chiaki web | Navegador del cliente | Vistas (Consolas, Registro, Stream, Ajustes), lectura del control, reproducción WebRTC | TS-04, TS-05 |
| Control Bluetooth | Emparejado al **dispositivo cliente** | DualSense/DS4 leído con la Gamepad API (WebHID en v0.3.0, solo en escritorio) | FS-05, TS-05 |
| Tailscale | Cliente y TUF | Túnel WireGuard cifrado, NAT traversal y relé DERP de respaldo, nombre `*.ts.net` y certificado | TS-06 (perfil Tailscale) |
| `web` (nginx) | TUF, contenedor | Estáticos de la UI y proxy de `/signal` hacia el puente. TLS propio o detrás de `tailscale serve` | TS-06 |
| `bridge` (C++) | TUF, red del host | Descubrimiento, despertar, registro, sesión con el PS5 (`chiaki-lib`), peer WebRTC, persistencia | TS-02, TS-03 |
| PS5 | LAN de casa | Servidor de Remote Play | — |

## 4. Flujos

**Arranque de una sesión:**

1. El cliente abre `https://<tuf>.<tailnet>.ts.net`. La petición viaja por Tailscale y llega a nginx, que entrega la UI.
2. La UI abre `wss://…/signal`, que nginx pasa al puente. Con `hosts.list`, el puente devuelve el PS5, que descubrió por broadcast en la LAN.
3. Al pulsar "Conectar", `session.start`:
   - El puente abre la sesión con el PS5 por LAN (TCP 9295 para el control, UDP 9296 para Takion).
   - El puente crea el peer WebRTC y envía `sdp.offer`, con candidatos ICE en `BRIDGE_ANNOUNCE_IP` = IP de Tailscale del TUF.
4. El navegador responde con `sdp.answer`. ICE se conecta por el túnel y empieza el media.

**Durante el juego:**

| Dirección | Contenido | Transporte entre el cliente y el TUF | Transporte entre el TUF y el PS5 |
|---|---|---|---|
| PS5 → cliente | Video H.264, audio Opus | WebRTC RTP, paquetes de unos 1200 bytes (la MTU de Tailscale es 1280) | Takion UDP 9296 |
| Cliente → PS5 | Estado del control (56 bytes, estado completo) | DataChannel `input`, no fiable | Takion (`chiaki_session_set_controller_state`) |
| PS5 → cliente | Vibración, teclado, fin de sesión | DataChannel `control`, fiable | Eventos de `chiaki-lib` |
| Cliente → PS5 | Home, texto, apagar | DataChannel `control` | `chiaki_session_*` |

## 5. Puertos

| Equipo | Puerto | Para qué | Expuesto a |
|---|---|---|---|
| TUF | 443/TCP | UI y `/signal` (HTTPS) | Solo la tailnet |
| TUF | 8090/TCP | Señalización del puente | Solo localhost (nginx) |
| TUF | 50000–50100/UDP | Media WebRTC | Solo la interfaz de Tailscale (regla de firewall de Windows) |
| PS5 | 9295/TCP+UDP, 9296/UDP, 9297/UDP, 9302/UDP | Remote Play | Solo la LAN |
| Router Movistar | — | Nada abierto | — |

## 6. Operación

- **Horario:** el TUF hiberna de noche y despierta en la mañana con una tarea programada de Windows (`WakeToRun`). Fuera de ese horario no hay servicio.
- **Energía:** reposo de unos 15–30 W. En estrato 4 de Cali, unos 4.000–8.000 COP al mes con horario.
- **Arranque automático:** Tailscale en modo desatendido. Los contenedores con `restart: unless-stopped`.
- **Diagnóstico rápido:**
  - `tailscale ping <tuf>` debe decir *direct*. Si dice *DERP*, funciona pero con más latencia.
  - `/health` del puente.
  - El overlay de estadísticas de la UI.

## 7. Puntos abiertos

| Punto | Opciones | Dónde se decide |
|---|---|---|
| Cómo correr el puente en Windows | A · WSL2 con red `mirrored` y Docker · B · puente nativo (MSYS2) · C · arranque dual con Linux | T-01 + T-26, [TS-00](specs/v0.1.0/technical/TS-00-viabilidad.md#decisión-de-host) |
| Descubrimiento si el broadcast no cruza (opción A) | Agregar el PS5 por IP manual (IP fija reservada en el Movistar) | T-26 → FS-02 |
| TLS | `tailscale serve` o `tailscale cert` + nginx | T-25 |

## 8. Fuera de esta espina

- **v0.2.0, remoto vía PSN:** es opcional, porque Tailscale ya resuelve el acceso remoto.
- **App oficial PS Remote Play:** no funciona de forma fiable por este túnel, porque no deja escribir una IP y busca la consola por broadcast.
- **chiaki-ng nativo desde fuera:** solo se usa si se activa el subnet router de Tailscale en el TUF (`--advertise-routes=<subred-movistar>`) y se agrega el PS5 como host manual.
