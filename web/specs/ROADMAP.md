# Hoja de ruta de chiaki-ng Web

## v0.1.0 · MVP LAN

Jugar en un PS5 desde Chrome/Edge/Safari, con un DualSense o DS4 por Bluetooth, desplegado con `docker compose up -d --build` en un host de la LAN del PS5. Desde fuera de casa se accede con el perfil Tailscale: el celular o la tablet abren la web del host por la tailnet.

**Puerta de viabilidad:** la tarea `T-01` (spike) de [v0.1.0/devprogress.md](v0.1.0/devprogress.md) debe demostrar video del PS5 reproducido en un navegador por WebRTC a través del puente. Si falla, no se sigue con la versión y se revisa [TS-00](v0.1.0/technical/TS-00-viabilidad.md).

Criterios de salida:
- Todos los CA de FS-01 a FS-06 se cumplen con un PS5 real.
- `docker compose up -d --build` funciona en el host de la LAN (Windows con la opción elegida en T-26, o Linux).
- Con el perfil Tailscale, un celular en datos móviles abre la web y juega.
- La latencia de extremo a extremo medida en el overlay (FS-04) es comparable a la del cliente de escritorio en la misma red, con una diferencia de ±15 ms.

## v0.2.0 · Remoto vía PSN (opcional)

Iniciar sesión en Sony y conectarse a un PS5 desde fuera de su red.

**Prioridad baja desde el 2026-11-19.** Con el perfil Tailscale de v0.1.0 ([TS-06](v0.1.0/technical/TS-06-despliegue-docker-compose.md#perfil-tailscale-recomendado)), el puente vive en la LAN del PS5 y el acceso remoto ya está resuelto sin PSN ni hole punching de Sony. Esta versión solo tiene sentido si se quiere un despliegue sin un host propio en la red del PS5. La v0.3.0 puede hacerse antes que esta.

Criterios de salida:
- Todos los CA de FS-07 y FS-08 se cumplen.
- Una sesión remota se mantiene estable 30 minutos con el navegador en una red móvil y el puente en la LAN del PS5.

## v0.3.0 · DualSense completo, micrófono y HEVC

Giroscopio, touchpad, gatillos adaptativos, luz y rumble completos por WebHID; micrófono; HEVC cuando el navegador lo soporte.

Criterios de salida:
- Todos los CA de FS-09, FS-10 y FS-11 se cumplen.
- Sin WebHID (Firefox/Safari) el juego sigue funcionando con la Gamepad API, sin errores.

## Más adelante (sin versión asignada)

- PS4 (el protocolo ya lo soporta en `chiaki-lib`).
- Varias consolas o usuarios simultáneos.
- PWA instalable.
