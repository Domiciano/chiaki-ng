# Especificaciones de chiaki-ng Web

## Convención de versionado

- La versión de chiaki-ng Web es **semver propio** (`MAJOR.MINOR.PATCH`), empezando en `0.1.0` e independiente de la versión de escritorio (`CHIAKI_VERSION_*` en `CMakeLists.txt`). La versión en desarrollo está en [`../VERSION`](../VERSION).
- Cada versión tiene su carpeta `vX.Y.Z/`, con:
  - `functional/`: specs funcionales (`FS-NN-*.md`), qué hace el producto y cómo se acepta.
  - `technical/`: specs técnicas (`TS-NN-*.md`), cómo se construye.
  - `devprogress.md`: tareas, estado, bitácora y criterios de salida de esa versión.
- Cada carpeta es un **delta**: describe lo que esa versión agrega o cambia. La especificación vigente de la versión N es la unión de todas las carpetas hasta N.
- Si una versión posterior cambia una spec existente, crea una spec nueva con un ID nuevo y marca la anterior con `Estado: Reemplazada por FS-xx (vX.Y.Z)`. Nunca se reescribe la historia de una versión publicada.
- Los IDs `FS-NN` y `TS-NN` son estables y no se reutilizan.
- Estados de una spec: `Borrador` → `Aprobada` → `Implementada`, o `Reemplazada`.
- Estados de una tarea en `devprogress.md`: `PENDIENTE`, `EN CURSO`, `HECHO`, `BLOQUEADO`.
- Una versión se cierra cuando todos sus criterios de salida están marcados. Entonces se crea el tag git `web-vX.Y.Z` y `VERSION` pasa a la siguiente.

## Índice

| Versión | Alcance | Specs |
|---|---|---|
| [v0.1.0](v0.1.0/devprogress.md) | MVP: descubrimiento, registro, streaming, control Bluetooth, despliegue docker-compose, acceso remoto por Tailscale | FS-01..FS-06, TS-00..TS-07 |
| [v0.2.0](v0.2.0/devprogress.md) | (Opcional) Remoto vía PSN: login Sony, conexión por hole-punch, TURN | FS-07..FS-08, TS-08..TS-09 |
| [v0.3.0](v0.3.0/devprogress.md) | DualSense completo, micrófono, calidad HEVC | FS-09..FS-11, TS-10..TS-12 |

Hoja de ruta y criterios de salida: [ROADMAP.md](ROADMAP.md).

## Plantilla FS

```markdown
# FS-NN · Título

| ID | Versión | Estado | TS relacionadas |
|---|---|---|---|
| FS-NN | vX.Y.Z | Borrador | [TS-NN](../technical/TS-NN-*.md) |

## Objetivo
## Actores
## Historias de usuario
## Flujo principal
## Flujos alternos
## Reglas
## Criterios de aceptación
- **CA-NN.1** Dado ..., cuando ..., entonces ...
## Fuera de alcance
```

## Plantilla TS

```markdown
# TS-NN · Título

| ID | Versión | Estado | FS que cubre |
|---|---|---|---|

## Contexto
## Diseño
## Contratos
## Código de chiaki-ng que se reutiliza
## Riesgos
## Pruebas
```

Idioma: las specs están en español. Los identificadores de código (clases, mensajes, rutas, variables) van en inglés, y los textos visibles de la UI en español.
