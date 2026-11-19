# TS-09 · Hole-punch PSN y TURN

| ID | Versión | Estado | FS que cubre |
|---|---|---|---|
| TS-09 | v0.2.0 | Borrador | [FS-08](../functional/FS-08-conexion-remota.md) |

## Contexto

Hay dos tramos que pueden ser remotos (ver los escenarios E1 y E2 en FS-08):

- **Puente ⇄ PS5:** lo resuelve `chiaki-lib` con el hole-punch de PSN y RUDP. El puente lo reutiliza tal cual.
- **Navegador ⇄ puente:** WebRTC necesita STUN y TURN cuando no hay una ruta directa.

## Diseño

### Tramo puente ⇄ PS5 (E2)

**Lista de consolas:** `chiaki_holepunch_list_devices` (`lib/include/chiaki/remote/holepunch.h`) con el token PSN de [TS-08](TS-08-auth-psn.md), como en `gui/src/qmlbackend.cpp` (cerca de `chiaki_holepunch_list_devices`). La lista se libera con `chiaki_holepunch_free_device_list`.

**Sesión remota:** el mismo orden de llamadas que `StreamSession::InitiatePsnConnection` (`gui/src/streamsession.cpp`):

1. `chiaki_holepunch_session_init(psn_token, log)`
2. Opcional: `chiaki_holepunch_session_force_port_guessing`, `_set_port_guessing_ports` y `_set_port_guessing_socks`
3. `chiaki_holepunch_upnp_discover`
4. `chiaki_holepunch_session_create`
5. `chiaki_holepunch_session_start(session, duid, console_type)`, con un DUID de `chiaki_holepunch_generate_client_device_uid` que se guarda en `/data/psn.json` para que sea estable
6. `chiaki_holepunch_session_punch_hole(session, CHIAKI_HOLEPUNCH_PORT_TYPE_CTRL)`
7. `ChiakiConnectInfo.holepunch_session = session` y luego `chiaki_session_init`. El resto de [TS-02](../../v0.1.0/technical/TS-02-puente-cpp.md) no cambia.

Para cancelar: `chiaki_holepunch_main_thread_cancel`. Para cerrar: `chiaki_holepunch_session_fini`.

**Registro remoto:** `ChiakiRegistInfo.holepunch_info` con `chiaki_get_regist_info(session)` y `rudp`, como en el registro remoto del escritorio.

**Despertar remoto:** se replica lo que hace el cliente de escritorio al conectar a una consola en reposo vía PSN.

**Red:** el puente ya usa `network_mode: host`, así que UPnP y los sockets del hole-punch funcionan sin cambios en el compose.

### Tramo navegador ⇄ puente (E1)

- Servicio `coturn` opcional en `web/docker-compose.yml`, con `profiles: [remote]`:

```yaml
  turn:
    image: coturn/coturn:4.6
    profiles: [remote]
    network_mode: host
    command: >
      -n --listening-port=3478 --min-port=49160 --max-port=49200
      --use-auth-secret --static-auth-secret=${TURN_SECRET}
      --realm=${TURN_REALM} --external-ip=${TURN_EXTERNAL_IP}
    restart: unless-stopped
```

- Nuevas variables de `.env`: `TURN_SECRET`, `TURN_REALM`, `TURN_EXTERNAL_IP` y `TURN_URL` (p. ej. `turn:mi-dominio:3478`).
- El puente genera **credenciales TURN temporales** (REST API de coturn: usuario `expira:web` y contraseña `HMAC-SHA1(secret)`) y las envía en `session.start` → `ice.config`. El navegador nunca conoce `TURN_SECRET`.
- Tanto el puente como el navegador usan `iceServers` con STUN (el mismo coturn) y TURN.
- Hay que abrir en el router: **443/TCP** hacia nginx, **3478/UDP+TCP** y **49160–49200/UDP** hacia coturn. El rango WebRTC del puente (50000–50100) solo hace falta si se quiere intentar la conexión directa.

## Contratos

| Dirección | `type` | Payload |
|---|---|---|
| N→P | `psn.devices` | — |
| P→N | `psn.devices` | `devices: [{ duid, name, state, type }]` |
| N→P | `session.start` | `hostId` o `psnDuid`, `remote: bool` |
| P→N | `ice.config` | `iceServers` (con credenciales temporales) |
| P→N | `session.state` | Se agrega `holepunching` |

## Código de chiaki-ng que se reutiliza

- `lib/src/remote/holepunch.c`, `lib/src/remote/rudp.c` (sin cambios).
- `gui/src/streamsession.cpp` (`InitiatePsnConnection`) y `gui/src/qmlbackend.cpp`: referencia del orden de llamadas.

## Riesgos

- El hole-punch depende del NAT de ambos lados. Hay casos sin solución, y solo se puede dar un buen diagnóstico (FS-08 A1).
- La latencia en remoto la dominan las dos rutas por internet. En E1, TURN agrega un salto más.

## Pruebas

- Manual E1: navegador en datos móviles, puente en casa, perfil `remote` activo.
- Manual E2: puente en un VPS, PS5 en casa.
- Para ambos: 30 minutos de sesión y un registro remoto.
