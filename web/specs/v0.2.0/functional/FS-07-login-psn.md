# FS-07 · Inicio de sesión en Sony (PSN)

| ID | Versión | Estado | TS relacionadas |
|---|---|---|---|
| FS-07 | v0.2.0 | Borrador | [TS-08](../technical/TS-08-auth-psn.md) |

## Objetivo

Vincular la cuenta PSN del jugador al puente, para obtener el ID de cuenta automáticamente y poder conectarse en remoto (FS-08).

## Actores

- Jugador.
- Servicio de autenticación de Sony.

## Historias de usuario

- Como jugador, quiero iniciar sesión con mi cuenta de Sony desde la web, para usar Remote Play fuera de casa.
- Como jugador, quiero no tener que escribir mi ID de cuenta PSN al registrar una consola.
- Como jugador, quiero cerrar la sesión de PSN y que el puente olvide mis tokens.

## Flujo principal

1. En Ajustes → Cuenta PSN, el jugador pulsa "Iniciar sesión en PlayStation Network".
2. Se abre en una pestaña nueva la página oficial de login de Sony.
3. El jugador inicia sesión. Sony redirige a una página de `remoteplay.dl.playstation.net` que no carga contenido útil.
4. La UI explica: "Copia la dirección completa de esa pestaña y pégala aquí". El jugador la pega y pulsa "Vincular".
5. El puente canjea el código por los tokens y obtiene el ID de cuenta. La UI muestra "Cuenta vinculada: <ID de cuenta>".
6. Desde entonces:
   - El registro (FS-03) rellena el ID de cuenta solo.
   - La conexión remota (FS-08) queda habilitada.

## Flujos alternos

- **A1:** si la URL pegada no es una redirección válida (no contiene `code=`), se muestra "La dirección no es válida. Copia la URL completa después de iniciar sesión."
- **A2:** si el código expiró o ya se usó, se muestra "El código expiró. Vuelve a iniciar sesión." (los códigos duran pocos minutos).
- **A3:** si el token de acceso vence, el puente lo refresca solo, sin intervención del jugador. Si el refresco falla (401), la UI muestra "La sesión de PSN expiró" y pide repetir el login.
- **A4:** con "Cerrar sesión de PSN", el puente borra los tokens. El ID de cuenta de las consolas ya registradas se conserva.

## Reglas

- **R1:** el canje y el refresco de tokens ocurren **en el puente**. El navegador solo transporta la URL de redirección una vez y nunca recibe los tokens.
- **R2:** los tokens se guardan en el volumen del puente con permisos `0600`.
- **R3:** la URL pegada y los tokens nunca aparecen en los logs.

## Criterios de aceptación

- **CA-07.1** Dado un jugador con una cuenta PSN válida, cuando pega la URL de redirección, entonces en menos de 10 s ve "Cuenta vinculada" con su ID de cuenta.
- **CA-07.2** Dada una URL inválida, entonces se muestra el mensaje de A1 sin llamar a Sony.
- **CA-07.3** Dada una cuenta vinculada, cuando el jugador registra una consola (FS-03), entonces el campo del ID de cuenta ya está relleno.
- **CA-07.4** Dado un token de acceso vencido, cuando el jugador abre la lista remota (FS-08), entonces la lista carga sin pedir un nuevo login.
- **CA-07.5** Dada una cuenta vinculada, cuando se revisan el almacenamiento y el tráfico del navegador después del paso 5, entonces no aparece ningún `access_token` ni `refresh_token`.
- **CA-07.6** Dada una cuenta vinculada, cuando el jugador cierra la sesión de PSN, entonces los tokens se borran del volumen del puente.

## Fuera de alcance

- Login embebido en un iframe o webview: Sony no lo permite y no hay forma de interceptar la redirección desde una página web.
