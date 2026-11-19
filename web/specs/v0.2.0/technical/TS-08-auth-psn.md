# TS-08 · Autenticación PSN en el puente

| ID | Versión | Estado | FS que cubre |
|---|---|---|---|
| TS-08 | v0.2.0 | Borrador | [FS-07](../functional/FS-07-login-psn.md), [FS-08](../functional/FS-08-conexion-remota.md) |

## Contexto

En el escritorio, el login usa un WebEngine embebido que intercepta la redirección (`gui/src/qml/PSNLoginDialog.qml`). Además existe un modo alternativo: el usuario pega la URL (`checkPsnRedirectURL`, `handlePsnLoginRedirect` en `gui/src/qmlbackend.cpp`). En la web solo es posible el modo de pegar, porque una página no puede leer la URL de una pestaña de otro origen.

## Diseño

**Construcción de la URL de login:** la arma el puente, con `LOGIN_URL` y `CLIENT_ID` de `gui/include/psnaccountid.h`, y el frontend la abre con `window.open`. La URL sale del puente para tener un único lugar a mantener si Sony cambia los parámetros.

**Módulo `PsnAuth`** (C++, libcurl, que ya es dependencia de `chiaki-lib` por `third-party/curl`). Es una traducción a libcurl, sin Qt, de:

| Origen (Qt) | Función en el puente |
|---|---|
| `PSNAccountID::GetPsnAccountId` (`gui/src/psnaccountid.cpp`) | `exchangeCode(code)`: pide el token y luego el ID de cuenta |
| `PSNToken::InitPsnToken` (`gui/src/psntoken.cpp`) | Canje `grant_type=authorization_code` |
| `PSNToken::RefreshPsnToken` (`gui/src/psntoken.cpp`) | `refresh()` con `grant_type=refresh_token` |
| `JsonRequester` (`gui/src/jsonrequester.cpp`) | Helper `postForm()` con autenticación básica de cliente |

Se respetan exactamente el `scope`, el `redirect_uri` y las credenciales de cliente del escritorio.

**Persistencia:** `/data/psn.json` (0600):

```json
{ "accessToken": "...", "refreshToken": "...", "expiresAt": "ISO-8601", "accountId": "base64", "onlineId": "..." }
```

**Refresco:** antes de cada operación remota, si faltan menos de 5 minutos para `expiresAt`, se refresca. Un 401 del refresco borra los tokens y emite `psn.expired`.

## Contratos

Mensajes nuevos de señalización (se agregan a [TS-03](../../v0.1.0/technical/TS-03-webrtc-y-senalizacion.md)):

| Dirección | `type` | Payload |
|---|---|---|
| N→P | `psn.login_url` | — |
| P→N | `psn.login_url` | `url` |
| N→P | `psn.link` | `redirectUrl` |
| P→N | `psn.linked` | `accountId`, `onlineId` |
| P→N | `psn.status` | `linked: bool`, `accountId?` |
| N→P | `psn.unlink` | — |
| P→N | `psn.expired` | — |

`redirectUrl` se valida en el puente (host `remoteplay.dl.playstation.net` y un `code` presente) antes de llamar a Sony.

## Código de chiaki-ng que se reutiliza

- `gui/include/psnaccountid.h`, `gui/src/psnaccountid.cpp`, `gui/src/psntoken.cpp`, `gui/src/jsonrequester.cpp`: la lógica y las constantes, traducidas a libcurl.
- `scripts/psn-account-id.py`: referencia alternativa del flujo.

## Riesgos

- Sony puede cambiar los endpoints o las credenciales de cliente. Al estar aislado en `PsnAuth`, el impacto es acotado. Hay que seguir los cambios upstream en `gui/src/psn*.cpp`.
- La URL pegada contiene un código de un solo uso: viaja por el WebSocket bajo TLS y no se registra en los logs.

## Pruebas

- Unitarias: validación de `redirectUrl` y parseo de las respuestas de Sony (fixtures JSON).
- Manual: login real, refresco forzado (bajando `expiresAt` a mano) y cierre de sesión.
