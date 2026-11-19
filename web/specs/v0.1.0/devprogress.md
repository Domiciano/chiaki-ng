# devprogress · chiaki-ng Web v0.1.0 (MVP LAN)

Alcance: [ROADMAP](../ROADMAP.md#v010--mvp-lan). Specs: FS-01..FS-06, TS-00..TS-07.

Estados: `PENDIENTE`, `EN CURSO`, `HECHO`, `BLOQUEADO`.

## Tareas

| ID tarea | Spec | Descripción | Estado | Fecha | Notas |
|---|---|---|---|---|---|
| T-01 | TS-00, TS-03 | **Spike de viabilidad** (en el host Windows de casa, ver T-26): un binario mínimo con `chiaki-lib` + libdatachannel inicia una sesión con un PS5 (con la `regist_key` del cliente de escritorio) y una página HTML mínima reproduce el video H.264 por WebRTC. Medir el salto de latencia y validar el `profile-level-id`, SPS/PPS y el signo de los ejes. **Puerta: si falla, se detiene la versión.** | PENDIENTE | | |
| T-02 | TS-02 | Esqueleto de `web/bridge/`: CMake con `add_subdirectory` de la raíz, opciones sin GUI/CLI/FFmpeg, `FetchContent` de libdatachannel y nlohmann/json, binario `chiaki-web-bridge` | PENDIENTE | | |
| T-03 | TS-02, TS-03 | `SignalServer`: WebSocket en `:8090`, despacho JSON, errores y `GET /health` | PENDIENTE | | |
| T-04 | TS-02 | `HostStore` y `SettingsStore`: `/data/hosts.json` (0600) y `/data/settings.json` | PENDIENTE | | |
| T-05 | TS-02, FS-02 | `DiscoveryService` y `WakeService`, con empuje de `hosts.updated` | PENDIENTE | | CA-02.1, CA-02.2, CA-02.3, CA-02.4 |
| T-06 | TS-02, FS-03 | `RegistService` (`regist.start` → `regist.result`), sin registrar el PIN en los logs | PENDIENTE | | CA-03.1, CA-03.2, CA-03.4, CA-03.5 |
| T-07 | TS-02, TS-03, FS-04 | `StreamSession` + `WebRtcPeer`: oferta, ICE, track H.264 (NACK, SR, PLI → IDR) y track Opus | PENDIENTE | | CA-04.1..CA-04.5 |
| T-08 | TS-03, FS-04 | DataChannel `control`: home, teclado, rumble, stats, PIN de login, `goto_bed` y fin de sesión | PENDIENTE | | CA-04.6, CA-04.8 |
| T-09 | TS-03, FS-05 | DataChannel `input`: decodificador binario v1 → `chiaki_session_set_controller_state`, descartando las secuencias viejas | PENDIENTE | | |
| T-10 | FS-04 | Cierre de sesión al caerse el WebSocket o el peer; una sola sesión a la vez | PENDIENTE | | CA-04.7 |
| T-11 | TS-04 | Esqueleto de `web/frontend/` (Vite + React + TS), router, `types/protocol.ts` y `SignalClient` | PENDIENTE | | |
| T-12 | TS-04, FS-02, FS-03 | Vistas Consolas y Registro | PENDIENTE | | CA-02.5, CA-03.3 |
| T-13 | TS-04, FS-04 | `StreamView`: `StreamPeer`, pantalla completa, `jitterBufferTarget`, `StatsOverlay`, `PinDialog`, `OnScreenKeyboardDialog`, diálogo de salida | PENDIENTE | | CA-04.9 |
| T-14 | TS-05, FS-05 | `GamepadPoller`, mapeo, zona muerta, `inputCodec`, atajos y respaldo de teclado | PENDIENTE | | CA-05.1, CA-05.2, CA-05.3, CA-05.5, CA-05.7 |
| T-15 | TS-05, FS-05 | Rumble con `vibrationActuator` | PENDIENTE | | CA-05.4 |
| T-16 | TS-04, FS-06 | Vista Ajustes (`settings.get/set` + `localStorage`) | PENDIENTE | | CA-06.1..CA-06.4 |
| T-17 | TS-04, FS-01 | Vista "Acerca de" (versión y enlace AGPL); verificar que no hay secretos en el navegador | PENDIENTE | | CA-01.4, CA-01.5 |
| T-18 | TS-06, FS-01 | `docker-compose.yml`, Dockerfiles, `nginx.conf`, `.env.example`, `.dockerignore`, `.gitignore` (certs, htpasswd) | PENDIENTE | | CA-01.1, CA-01.2, CA-01.3 |
| T-19 | TS-06 | Documentar la alternativa `macvlan` (`docker-compose.macvlan.yml`) | PENDIENTE | | |
| T-20 | TS-07 | Pruebas del puente (input, señalización, stores) y vectores compartidos en `web/shared/fixtures/input-v1.json` | PENDIENTE | | |
| T-21 | TS-07 | Pruebas `vitest` del frontend (mapeo, zona muerta, codec, `SignalClient`) | PENDIENTE | | |
| T-22 | TS-07 | Job de CI opcional `web` (frontend + puente) | PENDIENTE | | |
| T-23 | TS-07 | Ejecutar la lista de prueba manual con un PS5 real (12 puntos) y registrar los resultados aquí | PENDIENTE | | CA-04.10, CA-05.6, CA-05.8 |
| T-24 | — | Guía de instalación en `web/README.md` (submódulos, `.env`, certificados, htpasswd, firewall) | PENDIENTE | | |
| T-25 | TS-06, FS-01 | Perfil Tailscale: `tailscale serve` o `tailscale cert`, `BRIDGE_ANNOUNCE_IP` con la IP de Tailscale, `auth_basic` opcional, ACL de la tailnet y prueba desde un celular en datos móviles | PENDIENTE | | CA de TS-06 (perfil Tailscale), FS-01 R4 |
| T-26 | TS-00 | Decidir cómo correr el puente en el host Windows (A: WSL2 `mirrored`, B: nativo, C: arranque dual). Validar broadcast de descubrimiento, UDP entrante y firewall; si el broadcast no cruza, agregar host por IP manual a FS-02. Se hace junto con T-01. | PENDIENTE | | |

## Trazabilidad de criterios de aceptación

| CA | Tarea(s) |
|---|---|
| CA-01.1, CA-01.2, CA-01.3 | T-18, T-23 |
| CA-01.4, CA-01.5 | T-17 |
| CA-02.1..CA-02.4 | T-05, T-23 |
| CA-02.5 | T-12 |
| CA-03.1, CA-03.2, CA-03.4, CA-03.5 | T-06, T-23 |
| CA-03.3 | T-12 |
| CA-04.1..CA-04.5 | T-07, T-23 |
| CA-04.6, CA-04.8 | T-08 |
| CA-04.7 | T-10 |
| CA-04.9 | T-13 |
| CA-04.10 | T-23 |
| CA-05.1, CA-05.2, CA-05.3, CA-05.5, CA-05.7 | T-09, T-14, T-23 |
| CA-05.4 | T-15 |
| CA-05.6, CA-05.8 | T-01, T-23 |
| CA-06.1..CA-06.4 | T-16 |

## Criterios de salida de la versión

- [ ] T-01 (spike) en HECHO, con las mediciones registradas en la bitácora.
- [ ] Todos los CA de FS-01 a FS-06 verificados con un PS5 real (T-23).
- [ ] `docker compose up -d --build` funciona en el host de la LAN (TS-06), con la opción elegida en T-26.
- [ ] Perfil Tailscale: un celular en datos móviles abre la web con un certificado válido y juega (T-25).
- [ ] Latencia comparable a la del cliente de escritorio en la misma red, ±15 ms.
- [ ] `chiaki-unit`, las pruebas del puente y `vitest` en verde.
- [ ] Tag `web-v0.1.0` creado y `web/VERSION` en `0.2.0`.

## Bitácora

| Fecha | Entrada |
|---|---|
| 2026-11-19 | Se crean las specs de v0.1.0 (FS-01..FS-06, TS-00..TS-07). Decisión de arquitectura: React/TS + puente C++ con `chiaki-lib` + WebRTC, desplegado con docker-compose. |
| 2026-11-19 | Se adopta el **perfil Tailscale** como acceso principal: la web se despliega en un PC Windows de casa (ASUS TUF F15) y los clientes entran por la tailnet. Se agregan T-25 y T-26. La v0.2.0 (PSN) pasa a ser opcional. Se descarta usar un servidor fuera de la LAN del PS5. |
