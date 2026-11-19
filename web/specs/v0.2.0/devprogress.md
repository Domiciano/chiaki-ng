# devprogress · chiaki-ng Web v0.2.0 (remoto vía PSN)

Alcance: [ROADMAP](../ROADMAP.md#v020--remoto-vía-psn-opcional). Specs: FS-07, FS-08, TS-08, TS-09. Depende de que v0.1.0 esté cerrada. **Opcional desde el 2026-11-19:** el acceso remoto queda cubierto por el perfil Tailscale de v0.1.0 (ver [ROADMAP](../ROADMAP.md)).

## Tareas

| ID tarea | Spec | Descripción | Estado | Fecha | Notas |
|---|---|---|---|---|---|
| T-201 | TS-08 | `PsnAuth` en libcurl: URL de login, canje del código, ID de cuenta y refresco | PENDIENTE | | CA-07.1, CA-07.4 |
| T-202 | TS-08 | Persistencia en `/data/psn.json` (0600), cierre de sesión y logs sin secretos | PENDIENTE | | CA-07.5, CA-07.6 |
| T-203 | TS-08, FS-07 | Mensajes `psn.*` en la señalización | PENDIENTE | | |
| T-204 | FS-07 | UI Ajustes → Cuenta PSN: abrir el login, pegar la URL, validar y ver el estado | PENDIENTE | | CA-07.1, CA-07.2 |
| T-205 | FS-07, FS-03 | El registro rellena el ID de cuenta desde PSN | PENDIENTE | | CA-07.3 |
| T-206 | TS-09, FS-08 | `psn.devices` con `chiaki_holepunch_list_devices`; sección "Consolas de tu cuenta PSN" | PENDIENTE | | CA-08.1 |
| T-207 | TS-09, FS-08 | Sesión remota con hole-punch (orden de `InitiatePsnConnection`) y DUID estable | PENDIENTE | | CA-08.2, CA-08.5 |
| T-208 | TS-09, FS-08 | Registro remoto con `holepunch_info` | PENDIENTE | | CA-08.6 |
| T-209 | TS-09 | Servicio `turn` (coturn) con el perfil `remote`, credenciales temporales e `ice.config` | PENDIENTE | | CA-08.3 |
| T-210 | FS-08 | Bitrate remoto aparte en Ajustes; indicador "Relé TURN" en el overlay | PENDIENTE | | |
| T-211 | TS-09 | Pruebas manuales E1 y E2, 30 minutos cada una | PENDIENTE | | CA-08.4 |
| T-212 | — | Actualizar `web/README.md` (puertos del router, perfil `remote`) | PENDIENTE | | |

## Trazabilidad de criterios de aceptación

| CA | Tarea(s) |
|---|---|
| CA-07.1 | T-201, T-204 |
| CA-07.2 | T-204 |
| CA-07.3 | T-205 |
| CA-07.4 | T-201 |
| CA-07.5, CA-07.6 | T-202 |
| CA-08.1 | T-206 |
| CA-08.2, CA-08.5 | T-207 |
| CA-08.3 | T-209 |
| CA-08.4 | T-211 |
| CA-08.6 | T-208 |

## Criterios de salida de la versión

- [ ] Todos los CA de FS-07 y FS-08 verificados.
- [ ] Sesión remota estable de 30 minutos (E1 y E2).
- [ ] Sin tokens en el navegador ni en los logs.
- [ ] Tag `web-v0.2.0` creado y `web/VERSION` en `0.3.0`.

## Bitácora

| Fecha | Entrada |
|---|---|
| 2026-11-19 | Se crean las specs de v0.2.0 (FS-07, FS-08, TS-08, TS-09). |
| 2026-11-19 | Versión marcada como opcional y de baja prioridad: el perfil Tailscale de v0.1.0 resuelve el acceso remoto. |
