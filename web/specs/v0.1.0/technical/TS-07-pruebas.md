# TS-07 · Estrategia de pruebas

| ID | Versión | Estado | FS que cubre |
|---|---|---|---|
| TS-07 | v0.1.0 | Borrador | FS-01 a FS-06 |

## Contexto

La cobertura unitaria actual de chiaki-ng es pequeña (`test/`, munit, binario `chiaki-unit`). Buena parte del comportamiento de media solo se valida con un PS5 real. La versión web no modifica `lib/`, así que no agrega pruebas a `chiaki-unit`.

## Diseño

**Automáticas:**

| Capa | Herramienta | Qué cubre |
|---|---|---|
| `chiaki-lib` | `chiaki-unit` existente (`ctest --test-dir build`) | Se sigue ejecutando, sin cambios |
| Puente | munit o Catch2 (`web/bridge/test/`) | Decodificación del mensaje `input`, parseo y validación de la señalización, serialización de `hosts.json` y `settings.json` |
| Frontend | `vitest` | Mapeo del Gamepad, zona muerta, codificación de `input`, `SignalClient` con un WebSocket simulado |
| Contrato | Vectores compartidos en `web/shared/fixtures/input-v1.json` | Los mismos bytes se codifican en TS y se decodifican en C++ con resultados iguales |

**Manuales**, con un PS5 real (lista que se ejecuta antes de cerrar la versión y se registra en `devprogress.md`):

1. Despliegue limpio con `docker compose up -d --build` ([TS-06](TS-06-despliegue-docker-compose.md)).
2. Sin credenciales no se accede a la UI ni a `/signal` (FS-01).
3. Descubrimiento en menos de 5 s, con el estado correcto en reposo y encendido (FS-02).
4. Despertar desde el reposo (FS-02).
5. Registro con PIN: correcto, con un PIN incorrecto y con la consola apagada (FS-03).
6. Stream 720p60 y 1080p60 en Chrome, Edge y Safari, 10 minutos cada uno, anotando FPS, frames perdidos y latencia del overlay (FS-04).
7. Latencia comparada con el cliente de escritorio en la misma red: ±15 ms (ROADMAP).
8. Pérdida simulada con `tc qdisc ... netem loss 1%` en el host: el video se recupera con IDR y no se congela más de 1 s (FS-04).
9. DualSense y DS4 por Bluetooth: todos los botones, los sticks, los gatillos analógicos y el rumble (FS-05).
10. Desconectar el control a mitad de partida: el PS5 recibe el estado en reposo y no queda ningún botón pegado (FS-05).
11. Reconexión: recargar la página durante el stream y volver a conectar (FS-04).
12. Ajustes guardados que se respetan en la sesión siguiente (FS-06).

## Pruebas

En CI: un job opcional `web` en `.github/workflows/` que ejecuta `npm ci && npm test` en `web/frontend` y construye el puente con sus pruebas. Se agrega en la tarea correspondiente de `devprogress.md`.
