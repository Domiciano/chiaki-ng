# TS-05 · Mapeo de entrada (control Bluetooth)

| ID | Versión | Estado | FS que cubre |
|---|---|---|---|
| TS-05 | v0.1.0 | Borrador | [FS-05](../functional/FS-05-control-bluetooth.md) |

## Contexto

El control se empareja por Bluetooth **con el equipo donde corre el navegador**, no con el PS5 ni con el puente. El navegador lo lee con la Gamepad API y envía el estado por el DataChannel `input` ([TS-03](TS-03-webrtc-y-senalizacion.md)).

## Diseño

**Sondeo (`GamepadPoller`):**

- Se activa con `gamepadconnected` y se detiene con `gamepaddisconnected`.
- Se usa el primer gamepad con `mapping === "standard"`. Si no hay ninguno, el primero conectado, con un aviso en la UI.
- El estado se lee en cada `requestAnimationFrame`.
- El mensaje se envía **cuando cambia el estado** y, como keep-alive, cada 50 ms aunque no cambie.
- Si la pestaña pierde visibilidad, se envía el estado en reposo una vez y se pausa el envío.

**Tabla de botones** (mapeo `standard` → `ChiakiControllerButton`, `lib/include/chiaki/controller.h`):

| Índice Gamepad | Botón DualSense | Chiaki |
|---|---|---|
| 0 | Cruz | `CROSS` |
| 1 | Círculo | `MOON` |
| 2 | Cuadrado | `BOX` |
| 3 | Triángulo | `PYRAMID` |
| 4 / 5 | L1 / R1 | `L1` / `R1` |
| 6 / 7 | L2 / R2 (analógicos) | `l2_state` / `r2_state` (0..255) |
| 8 | Create | `SHARE` |
| 9 | Options | `OPTIONS` |
| 10 / 11 | L3 / R3 | `L3` / `R3` |
| 12..15 | D-pad arriba/abajo/izquierda/derecha | `DPAD_UP/DOWN/LEFT/RIGHT` |
| 16 | PS | `PS` |
| 17 | Touchpad (clic) | `TOUCHPAD` |

**Ejes:**

- Del rango `[-1, 1]` a `int16` con `Math.round(v * 32767)`.
- Los ejes 1 y 3 (Y) no se invierten, porque Chiaki usa la misma convención que SDL (abajo es positivo). Se confirma en el spike.
- La zona muerta radial es configurable (por defecto 0,08). Con el stick dentro de ella, el valor es 0.

**Gatillos:** `buttons[6|7].value` en `[0, 1]` pasa a `uint8` (0..255).

**Respaldo con teclado** (`keyboardFallback`, solo si no hay gamepad):
- Flechas: D-pad.
- `Enter`: Cruz. `Backspace`: Círculo.
- `Esc`: PS. `Tab`: Options.
- Es configurable en una versión futura.

**Rumble:** al recibir `rumble {left, right}` por `control`, se llama a:

```ts
gamepad.vibrationActuator?.playEffect("dual-rumble",
  { duration: 100, strongMagnitude: left / 255, weakMagnitude: right / 255 })
```

y se renueva mientras sigan llegando eventos. Si la API no existe, no se hace nada.

**Atajos reservados:**
- PS + Options mantenidos 1 s: alternar el overlay de estadísticas.
- PS + Create mantenidos 1 s: salir del stream.

Mientras se mantienen, estos combos no se envían al PS5.

## Código de chiaki-ng que se reutiliza

- `lib/include/chiaki/controller.h`: los valores de los bits.
- `gui/src/controllermanager.cpp`: referencia del mapeo SDL a Chiaki, para conservar las mismas convenciones de ejes y gatillos.

## Riesgos

- Firefox puede reportar un mapeo distinto de `standard` para el DualSense. Se mitiga con una tabla alternativa por `gamepad.id`, si la prueba manual lo confirma.
- Los botones PS y touchpad (16/17) no siempre se exponen en todos los navegadores.

## Pruebas

`vitest`: tabla de mapeo, zona muerta, conversión de ejes y gatillos, y codificación binaria con vectores compartidos con el puente. Prueba manual con un DualSense por Bluetooth ([TS-07](TS-07-pruebas.md)).
