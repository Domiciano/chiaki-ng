# FS-01 · Acceso y seguridad

| ID | Versión | Estado | TS relacionadas |
|---|---|---|---|
| FS-01 | v0.1.0 | Borrador | [TS-01](../technical/TS-01-arquitectura.md), [TS-04](../technical/TS-04-frontend-react.md), [TS-06](../technical/TS-06-despliegue-docker-compose.md) |

## Objetivo

Que solo el dueño pueda usar la aplicación. Quien llega a la UI puede encender, controlar y apagar su PS5.

## Actores

- **Jugador**: el dueño del despliegue.
- **Tercero**: cualquier otra persona que alcance el host por la red.

## Historias de usuario

- Como jugador, quiero que la aplicación me pida usuario y contraseña, para que nadie más en mi red controle mi consola.
- Como jugador, quiero acceder por HTTPS, para que el navegador permita usar el control y el micrófono.

## Flujo principal

1. El jugador abre `https://<host>`.
2. El navegador pide las credenciales (autenticación básica).
3. Con credenciales válidas, se carga la vista Consolas.

## Flujos alternos

- **A1:** con credenciales inválidas o cancelando, nginx responde 401 y no se carga ningún recurso de la aplicación.
- **A2:** si se accede por `http://`, nginx redirige a `https://`.

## Reglas

- **R1:** la señalización (`/signal`) exige las mismas credenciales que la UI.
- **R2:** el navegador no guarda claves de registro ni tokens. Viven solo en el puente.
- **R3:** la vista "Acerca de" muestra la versión web (`web/VERSION`) y un enlace al código fuente, como exige la AGPL-3.0.
- **R4:** en el perfil Tailscale ([TS-06](../technical/TS-06-despliegue-docker-compose.md#perfil-tailscale-recomendado)), el control de acceso principal es la pertenencia a la tailnet, y `auth_basic` es opcional. En ese perfil, CA-01.1 y CA-01.2 se verifican así: un dispositivo fuera de la tailnet no alcanza ni la UI ni `/signal`.

## Criterios de aceptación

- **CA-01.1** Dado un navegador sin credenciales, cuando abre `https://<host>/`, entonces recibe 401 y no ve la UI.
- **CA-01.2** Dado un cliente sin credenciales, cuando intenta abrir un WebSocket a `wss://<host>/signal`, entonces la conexión se rechaza con 401.
- **CA-01.3** Dado un navegador que abre `http://<host>/`, entonces se redirige a `https://<host>/`.
- **CA-01.4** Dado un jugador autenticado, cuando abre "Acerca de", entonces ve la versión web y un enlace al repositorio del código fuente.
- **CA-01.5** Dado un jugador autenticado que revisa el almacenamiento del navegador (localStorage, IndexedDB, cookies), entonces no encuentra ninguna `regist_key` ni ningún token.

## Fuera de alcance

- Cuentas de varios usuarios o roles.
- Inicio de sesión con proveedores externos.
