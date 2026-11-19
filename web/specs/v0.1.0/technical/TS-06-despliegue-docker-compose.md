# TS-06 · Despliegue con docker-compose

| ID | Versión | Estado | FS que cubre |
|---|---|---|---|
| TS-06 | v0.1.0 | Borrador | [FS-01](../functional/FS-01-acceso-y-seguridad.md), [FS-02](../functional/FS-02-descubrimiento-consolas.md), [FS-04](../functional/FS-04-streaming.md) |

## Contexto

Es para uso personal: una instancia de cada servicio y una sesión de stream a la vez, sin escalado ni orquestador. El único comando de despliegue es `docker compose up -d --build`, ejecutado en `web/`.

## Diseño

**Host recomendado:** Linux en la **misma LAN que el PS5** (mini PC, NAS, Raspberry Pi 4/5 en arm64). Con Docker Desktop en macOS o Windows, `network_mode: host` no pasa el broadcast LAN. Para ese caso se documenta la alternativa `macvlan` (ver más abajo).

**Archivos esperados:**

```
web/
├── docker-compose.yml
├── .env.example
├── frontend/Dockerfile
├── frontend/nginx.conf
└── bridge/Dockerfile          # contexto de build = raíz del repo (necesita lib/ y third-party/)
```

### `docker-compose.yml` (referencia)

```yaml
services:
  bridge:
    build:
      context: ..
      dockerfile: web/bridge/Dockerfile
    network_mode: host            # broadcast UDP, TCP 9295, UDP 9296/9297 y rango WebRTC
    env_file: .env
    volumes:
      - bridge-data:/data
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-fsS", "http://127.0.0.1:${BRIDGE_SIGNAL_PORT:-8090}/health"]
      interval: 30s
      timeout: 3s
      retries: 3

  web:
    build:
      context: ./frontend
    ports:
      - "80:80"
      - "443:443"
    extra_hosts:
      - "host.docker.internal:host-gateway"   # para alcanzar el puente, que está en la red del host
    volumes:
      - ./certs:/etc/nginx/certs:ro
      - ./htpasswd:/etc/nginx/htpasswd:ro
    depends_on:
      - bridge
    restart: unless-stopped

volumes:
  bridge-data:
```

### `frontend/Dockerfile` (referencia)

```dockerfile
FROM node:22-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:1.27-alpine
COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=build /app/dist /usr/share/nginx/html
```

### `frontend/nginx.conf` (referencia)

```nginx
server {
    listen 80;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    http2 on;
    ssl_certificate     /etc/nginx/certs/fullchain.pem;
    ssl_certificate_key /etc/nginx/certs/privkey.pem;

    auth_basic           "chiaki-ng Web";
    auth_basic_user_file /etc/nginx/htpasswd;

    root /usr/share/nginx/html;

    location / {
        try_files $uri /index.html;
    }

    location /signal {
        proxy_pass http://host.docker.internal:8090;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_read_timeout 1h;
    }

    location /health {
        auth_basic off;
        proxy_pass http://host.docker.internal:8090/health;
    }
}
```

### `bridge/Dockerfile` (referencia)

```dockerfile
FROM debian:bookworm AS build
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential cmake ninja-build git pkg-config python3 python3-protobuf protobuf-compiler \
    libssl-dev libjson-c-dev libminiupnpc-dev libopus-dev ca-certificates
WORKDIR /src
COPY . .
RUN cmake -S web/bridge -B build -G Ninja -DCMAKE_BUILD_TYPE=Release \
 && cmake --build build --target chiaki-web-bridge

FROM debian:bookworm-slim
RUN apt-get update && apt-get install -y --no-install-recommends \
    libssl3 libjson-c5 libminiupnpc17 libopus0 curl ca-certificates \
 && rm -rf /var/lib/apt/lists/*
COPY --from=build /src/build/chiaki-web-bridge /usr/local/bin/
VOLUME /data
ENTRYPOINT ["chiaki-web-bridge"]
```

Los submódulos (`third-party/nanopb`, `third-party/jerasure`, `third-party/gf-complete`, `third-party/curl`) deben estar inicializados antes del build (`git submodule update --init --recursive`). Se agrega un `.dockerignore` en la raíz que excluya `build*/`, `android/`, `switch/`, `docs/` y `gui/`, para que el contexto sea pequeño.

### `.env.example`

```
BRIDGE_SIGNAL_PORT=8090
BRIDGE_RTC_PORT_MIN=50000
BRIDGE_RTC_PORT_MAX=50100
BRIDGE_ANNOUNCE_IP=192.168.1.10
BRIDGE_LOG_LEVEL=info
```

### TLS y acceso

- HTTPS es obligatorio: la Gamepad API, WebHID y `getUserMedia` exigen un contexto seguro.
- Opciones de certificado:
  - `mkcert`, instalando la CA local en los equipos cliente.
  - Let's Encrypt, si hay dominio.
  - Autofirmado, aceptando la advertencia del navegador.
- `htpasswd` se genera con `htpasswd -B -c web/htpasswd <usuario>`. Ni `certs/` ni `htpasswd` se versionan (van en `.gitignore`).

### Red y firewall

- Abrir en el host: **443/TCP** (y 80/TCP) y el rango **UDP 50000–50100**.
- El puerto 8090 no se abre: solo nginx lo usa.
- El puente habla con el PS5 en TCP 9295 y UDP 9296/9297, más el broadcast de descubrimiento.

### Alternativa `macvlan` (Docker Desktop, o si el host no puede usar la red host)

El servicio `bridge` usa una red `macvlan` con IP propia en la LAN, en lugar de `network_mode: host`. En ese caso nginx apunta a esa IP, y no a `host.docker.internal`. Se documenta como un perfil opcional (`docker-compose.macvlan.yml`).

### Perfil Tailscale (recomendado)

Es el perfil de referencia para uso personal: el host está en casa, en la LAN del PS5, y los clientes acceden desde cualquier lugar por la tailnet.

- **Host:** Tailscale instalado y en modo desatendido, para que siga funcionando sin sesión iniciada (en Windows, "Run unattended"). En la consola de administración de Tailscale se desactiva la expiración de la clave de este equipo. **No hace falta subnet router** para la web, porque el navegador habla con el host y no con el PS5. Solo es necesario si además se usa chiaki-ng nativo desde fuera.
- **HTTPS:** se usa el nombre MagicDNS del host, `<host>.<tailnet>.ts.net`, con certificado válido emitido por Tailscale. Hay dos formas:
  - `tailscale cert <host>.<tailnet>.ts.net`, que genera `fullchain.pem` y `privkey.pem` para montarlos en `./certs`.
  - `tailscale serve --https=443 http://127.0.0.1:8080`: Tailscale termina el TLS y el servicio `web` publica solo HTTP en `127.0.0.1:8080`.
- **WebRTC:** `BRIDGE_ANNOUNCE_IP` es la IP de Tailscale del host (`100.x.y.z`, se obtiene con `tailscale ip -4`). El rango UDP `50000–50100` solo tiene que estar permitido en el firewall del host para la interfaz de Tailscale. No se abre nada en el router y no se necesita TURN.
- **Acceso:** solo los dispositivos de la tailnet alcanzan la web. `auth_basic` pasa a ser **opcional** (FS-01 R4), y se recomienda una ACL de Tailscale que limite el acceso a los dispositivos propios.
- **Disponibilidad:** si el host tiene encendido programado, la web solo está disponible dentro de ese horario.

`.env` para este perfil:

```
BRIDGE_ANNOUNCE_IP=100.x.y.z
BRIDGE_RTC_PORT_MIN=50000
BRIDGE_RTC_PORT_MAX=50100
WEB_TLS_MODE=tailscale-serve   # o: certs
```

**Host Windows:** este perfil es independiente del sistema operativo. Las opciones para correr el puente en Windows (WSL2 `mirrored`, nativo o arranque dual) están en [TS-00](TS-00-viabilidad.md#decisión-de-host).

## Riesgos

- `extra_hosts: host-gateway` requiere Docker 20.10 o superior.
- Perfil Tailscale: si la conexión directa entre el cliente y el host no se logra, Tailscale usa sus relés DERP. Funciona igual, pero con más latencia, y se comprueba con `tailscale ping <host>`. La MTU del túnel es de 1280, así que el empaquetador RTP debe respetar un tamaño máximo de paquete de unos 1200 bytes.
- `BRIDGE_ANNOUNCE_IP` mal configurada hace que los candidatos ICE apunten a una IP que no existe. El síntoma es que el video nunca empieza aunque la señalización funcione. Se muestra un error claro en la UI si ICE falla.

## Pruebas

Criterio de aceptación del perfil Tailscale: desde un celular en datos móviles con Tailscale activo, `https://<host>.<tailnet>.ts.net` carga la UI con un certificado válido, sin advertencias, y el stream arranca. Además, `tailscale ping <host>` responde.

Criterio de aceptación de despliegue: en un Linux limpio de la LAN, después de inicializar los submódulos, crear `.env`, `certs/` y `htpasswd`, `docker compose up -d --build` deja la UI en `https://<host>`, el `healthcheck` del puente en `healthy` y el PS5 visible en la lista de consolas.
