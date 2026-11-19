# chiaki-ng Web

Versión web de chiaki-ng: Remote Play de PS5 desde el navegador, en LAN y en remoto vía PSN, con un control Bluetooth (DualSense/DS4).

Un navegador no puede abrir los sockets TCP/UDP crudos que usa el protocolo de Remote Play. Por eso la versión web tiene dos piezas, que se despliegan juntas con `docker-compose`:

- **`web`**: frontend React + TypeScript servido por nginx (HTTPS). nginx también proxea la señalización (`/signal`, WebSocket).
- **`bridge`**: un puente nativo en C++ que reutiliza `chiaki-lib` (`lib/`). Habla el protocolo con el PS5 por TCP/UDP y retransmite al navegador por WebRTC:
  - El video H.264 y el audio Opus pasan sin transcodificar.
  - Los controles viajan por DataChannel.

```
Navegador ⇄ web (nginx: estáticos + /signal) ⇄ bridge (C++, chiaki-lib) ⇄ PS5
     ⇅_____________ WebRTC (UDP: video, audio, controles) _____________⇅
```

- Versión actual: ver [`VERSION`](VERSION). La versión web usa su propio semver, independiente de la versión de escritorio.
- Especificaciones, hoja de ruta y avance: [`specs/`](specs/README.md).
- Cómo queda desplegada en casa (Chiaki web ⇄ Tailscale ⇄ TUF F15 ⇄ PS5): [`arch_spine.md`](arch_spine.md).
