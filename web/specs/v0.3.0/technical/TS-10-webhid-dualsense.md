# TS-10 · WebHID para DualSense y DS4

| ID | Versión | Estado | FS que cubre |
|---|---|---|---|
| TS-10 | v0.3.0 | Borrador | [FS-09](../functional/FS-09-retroalimentacion-dualsense.md) |

## Contexto

La Gamepad API solo expone los botones, los ejes y un rumble básico. El resto requiere leer y escribir los reportes HID del control. WebHID solo existe en Chrome y Edge, y exige un gesto del usuario la primera vez.

## Diseño

**Conexión:**

- `navigator.hid.requestDevice({ filters: [{ vendorId: 0x054C, productId: 0x0CE6 }, { vendorId: 0x054C, productId: 0x0DF2 }, { vendorId: 0x054C, productId: 0x09CC }, { vendorId: 0x054C, productId: 0x05C4 }] })`. Son el DualSense, el DualSense Edge y los DS4 v2 y v1.
- Las visitas siguientes usan `navigator.hid.getDevices()`.

**Entrada por Bluetooth** (módulo `input/hid/dualsense.ts`):

- Al conectar, se pide el reporte de características 0x05 (calibración). Eso habilita el modo de reporte completo 0x31.
- El reporte de entrada 0x31 trae los botones, los sticks, los gatillos, el giroscopio, el acelerómetro y el touchpad (dos puntos), protegidos con CRC32.
- Los datos crudos se convierten a las unidades de `ChiakiControllerState`:
  - `gyro_*` en rad/s y `accel_*` en g, aplicando la calibración, con la misma escala que usa el escritorio vía SDL.
  - `touches[2]` con un `id` incremental por toque (`touch_id_next`) y coordenadas en el rango del touchpad (1920×1080).
- El estado completo va en el mensaje `input` v1, que ya reservaba esos campos ([TS-03](../../v0.1.0/technical/TS-03-webrtc-y-senalizacion.md)).

**Salida por Bluetooth:**

- El reporte de salida 0x31, con CRC32 (semilla 0xA2), lleva:
  - Rumble (motores izquierdo y derecho).
  - Gatillos adaptativos: tipo y 10 bytes de parámetros por gatillo, tal como llegan en `ChiakiTriggerEffectsEvent`.
  - Barra de luz RGB (`CHIAKI_EVENT_LED_COLOR`).
  - LEDs de jugador (`CHIAKI_EVENT_PLAYER_INDEX`).
  - Botón de silencio.
- El DS4 usa su propio reporte de salida 0x11, con rumble y luz.
- Escala de intensidad: `CHIAKI_EVENT_HAPTIC_INTENSITY` y `CHIAKI_EVENT_TRIGGER_INTENSITY`.
- `CHIAKI_EVENT_MOTION_RESET` reinicia la orientación.

**Nuevos mensajes del DataChannel `control`** (P→N), emitidos por `event_cb` en el puente:

| `type` | Payload | Evento chiaki-lib |
|---|---|---|
| `trigger_effects` | `typeLeft`, `typeRight`, `left[10]`, `right[10]` | `CHIAKI_EVENT_TRIGGER_EFFECTS` |
| `led_color` | `r`, `g`, `b` | `CHIAKI_EVENT_LED_COLOR` |
| `player_index` | `index` | `CHIAKI_EVENT_PLAYER_INDEX` |
| `haptic_intensity` / `trigger_intensity` | `value` | `CHIAKI_EVENT_HAPTIC_INTENSITY` / `_TRIGGER_INTENSITY` |
| `motion_reset` | — | `CHIAKI_EVENT_MOTION_RESET` |

**Audio háptico HD:** el `haptics_sink` de `chiaki-lib` entrega PCM pensado para el DualSense por USB (el escritorio lo envía como audio). Por Bluetooth y desde la web no hay vía fiable. Se resume la energía del PCM en una magnitud de rumble cada 10 ms y se envía como `rumble`.

## Código de chiaki-ng que se reutiliza

- `gui/src/controllermanager.cpp` y `gui/include/controllermanager.h`: referencia de las escalas, la calibración y la aplicación de los efectos de gatillo y la luz.
- `lib/include/chiaki/session.h`: estructuras de los eventos.
- `lib/include/chiaki/controller.h`: `ChiakiControllerState`, `ChiakiControllerTouch`.

## Riesgos

- El formato de los reportes BT no está documentado oficialmente. Hay que usar como referencia lo que hacen SDL y el escritorio, y probar con cada revisión de firmware del control.
- Si el CRC del reporte de salida está mal, el control lo ignora en silencio. Se cubre con una prueba unitaria contra un vector conocido.

## Pruebas

- `vitest`: parseo del reporte 0x31 con capturas reales y el CRC32 de salida.
- Manual: un juego con giroscopio, uno con gatillos adaptativos y la luz.
