# Spec viva: cuentas y acceso

## Purpose

Permitir que una persona cree una cuenta, inicie y cierre sesión y consulte su perfil, y proteger las pantallas y peticiones que exigen una sesión válida. Describe el comportamiento actual, visto desde la API y desde la pantalla.

## Requirements

### Requirement: Registro de cuenta por API

La API SHALL crear una cuenta con `POST /api/v1/auth/signup` cuando los datos son válidos y SHALL devolver en la misma respuesta el usuario creado y un token de acceso, de modo que la cuenta queda con la sesión iniciada.

#### Scenario: Registro con datos válidos

- **WHEN** se envía `fullName` "Ada Lovelace", un `email` que no está registrado, un `password` de entre 8 y 32 caracteres y un `passwordConfirmation` igual a él
- **THEN** la respuesta es 200 con cuerpo `{ "data": { "user": { ... }, "token": "..." } }`, y el token sirve de inmediato para consultar el perfil

#### Scenario: Registro sin nombre

- **WHEN** se envía `fullName` con valor `null` o como texto vacío, junto con email, contraseña y confirmación válidos
- **THEN** la cuenta se crea y el usuario devuelto tiene `fullName` igual a `null`

### Requirement: Validación del registro en la API

La API SHALL rechazar el registro con un 422 y un cuerpo `{ "errors": [ ... ] }`, en el que cada error indica al menos `field` y `rule`, cuando los datos no cumplen estas reglas: email presente, con formato válido y de 254 caracteres como máximo; email no registrado previamente; contraseña de entre 8 y 32 caracteres; confirmación idéntica a la contraseña.

#### Scenario: Email ya registrado

- **WHEN** existe una cuenta con el email "ana@flowsync.dev" y se intenta registrar otra con ese mismo email
- **THEN** la respuesta es 422 y contiene un error con `field` "email" y `rule` "database.unique", y no se crea ninguna cuenta

#### Scenario: Contraseña demasiado corta

- **WHEN** se intenta registrar una cuenta con una contraseña de 7 caracteres
- **THEN** la respuesta es 422 y contiene un error con `field` "password" y `rule` "minLength"

#### Scenario: Contraseña demasiado larga

- **WHEN** se intenta registrar una cuenta con una contraseña de 33 caracteres
- **THEN** la respuesta es 422 y contiene un error con `field` "password" y `rule` "maxLength"

#### Scenario: Confirmación distinta

- **WHEN** se intenta registrar una cuenta con un `passwordConfirmation` distinto de `password`
- **THEN** la respuesta es 422 y contiene un error con `field` "passwordConfirmation" y `rule` "sameAs"

#### Scenario: Email con formato inválido

- **WHEN** se intenta registrar una cuenta con el email "no-es-un-email"
- **THEN** la respuesta es 422 y contiene un error con `field` "email"

### Requirement: Inicio de sesión por API

La API SHALL iniciar sesión con `POST /api/v1/auth/login` cuando el email y la contraseña corresponden a una cuenta existente, y SHALL devolver el usuario y un token de acceso nuevo en cada inicio de sesión.

#### Scenario: Credenciales correctas

- **WHEN** existe una cuenta con email "ana@flowsync.dev" y contraseña "secreto123", y se envían exactamente esos datos
- **THEN** la respuesta es 200 con cuerpo `{ "data": { "user": { ... }, "token": "..." } }`

#### Scenario: Varias sesiones simultáneas

- **WHEN** la misma cuenta inicia sesión dos veces y obtiene dos tokens
- **THEN** los dos tokens sirven para consultar el perfil

### Requirement: Rechazo de credenciales en la API

La API SHALL rechazar el inicio de sesión con un 400 y el cuerpo `{ "errors": [ { "message": "Invalid user credentials" } ] }` cuando el email no pertenece a ninguna cuenta o la contraseña no es la de esa cuenta, sin indicar cuál de las dos cosas falló y sin emitir token. Antes de eso, SHALL responder 422 si falta el email, si su formato no es válido o si falta la contraseña.

#### Scenario: Contraseña incorrecta

- **WHEN** existe la cuenta "ana@flowsync.dev" y se inicia sesión con ese email y una contraseña distinta
- **THEN** la respuesta es 400 con `{ "errors": [ { "message": "Invalid user credentials" } ] }` y no incluye token

#### Scenario: Email inexistente

- **WHEN** se inicia sesión con un email que no pertenece a ninguna cuenta
- **THEN** la respuesta es idéntica a la de la contraseña incorrecta: 400 con el mismo mensaje

#### Scenario: Campos vacíos

- **WHEN** se inicia sesión con el email o la contraseña vacíos
- **THEN** la respuesta es 422 con un error asociado al campo vacío

### Requirement: Consulta del perfil por API

La API SHALL devolver con `GET /api/v1/account/profile`, a quien presente un token válido en la cabecera `Authorization: Bearer <token>`, los datos de su propia cuenta envueltos en `data`, con exactamente los campos `id`, `fullName`, `email`, `createdAt`, `updatedAt` e `initials`. La contraseña nunca forma parte de la respuesta.

#### Scenario: Perfil con token válido

- **WHEN** una persona registrada con email "ana@flowsync.dev" pide su perfil con su token
- **THEN** la respuesta es 200 con `{ "data": { "id": ..., "fullName": ..., "email": "ana@flowsync.dev", "createdAt": ..., "updatedAt": ..., "initials": ... } }`, sin campo de contraseña

### Requirement: Iniciales del usuario

La API SHALL calcular las iniciales en mayúsculas así: con un nombre de dos o más palabras, la primera letra de las dos primeras palabras; con un nombre de una sola palabra, sus dos primeras letras; sin nombre, la primera letra de lo que va antes de la arroba y la primera letra de lo que va después.

#### Scenario: Nombre de dos palabras

- **WHEN** el nombre de la cuenta es "Ada Lovelace"
- **THEN** `initials` vale "AL"

#### Scenario: Nombre de una palabra

- **WHEN** el nombre de la cuenta es "Ada"
- **THEN** `initials` vale "AD"

#### Scenario: Sin nombre

- **WHEN** la cuenta no tiene nombre y su email es "ana@flowsync.dev"
- **THEN** `initials` vale "AF"

### Requirement: Protección de las peticiones de cuenta

La API SHALL responder 401 con `{ "errors": [ { "message": "Unauthorized access" } ] }` a las peticiones de perfil (`GET /api/v1/account/profile`) y de cierre de sesión (`POST /api/v1/account/logout`) que no traen token, traen un token que la API no reconoce o traen un token revocado.

#### Scenario: Sin token

- **WHEN** se pide el perfil sin cabecera `Authorization`
- **THEN** la respuesta es 401 con `{ "errors": [ { "message": "Unauthorized access" } ] }`

#### Scenario: Token inventado

- **WHEN** se pide el perfil con `Authorization: Bearer abc`
- **THEN** la respuesta es 401

### Requirement: Cierre de sesión por API

La API SHALL revocar con `POST /api/v1/account/logout` únicamente el token con el que se hace la petición, y SHALL responder 200 con `{ "message": "Logged out successfully" }`, sin envoltorio `data`. Los demás tokens de la misma cuenta SHALL seguir siendo válidos.

#### Scenario: Token revocado tras cerrar sesión

- **WHEN** una persona cierra sesión con su token y después pide el perfil con ese mismo token
- **THEN** el cierre responde 200 con `{ "message": "Logged out successfully" }` y la petición de perfil posterior responde 401

#### Scenario: Otras sesiones se mantienen

- **WHEN** una cuenta tiene dos tokens y cierra sesión con el primero
- **THEN** el segundo token sigue sirviendo para consultar el perfil

### Requirement: Vigencia de los tokens

La API SHALL emitir los tokens de registro e inicio de sesión sin fecha de vencimiento, de modo que un token deja de ser aceptado solo cuando se revoca al cerrar sesión con él. No existe ningún vencimiento automático por tiempo configurado.

#### Scenario: Token sin revocar

- **WHEN** se usa un token que nunca se ha revocado con un cierre de sesión
- **THEN** la petición de perfil responde 200

### Requirement: Navegación según el estado de sesión

La pantalla SHALL dejar ver la página de perfil (`/profile`) solo con sesión iniciada, y las páginas de inicio de sesión (`/login`) y registro (`/register`) solo sin ella. Cualquier otra dirección SHALL llevar a `/profile`. Mientras se comprueba la sesión al cargar, SHALL mostrarse un indicador de carga en lugar de redirigir.

#### Scenario: Perfil sin sesión

- **WHEN** una persona sin sesión abre `/profile`
- **THEN** acaba en la página de inicio de sesión

#### Scenario: Acceso con sesión iniciada

- **WHEN** una persona con sesión iniciada abre `/login` o `/register`
- **THEN** acaba en la página de perfil

#### Scenario: Dirección desconocida

- **WHEN** una persona abre una dirección que no existe, por ejemplo `/cualquier-cosa`
- **THEN** se la manda a `/profile`, que a su vez la manda a `/login` si no tiene sesión

### Requirement: Formulario de registro

La pantalla de registro SHALL mostrar los campos «Nombre completo (opcional)», «Email», «Contraseña», con la pista «Entre 8 y 32 caracteres.», y «Repite la contraseña». Si la contraseña y su repetición no coinciden, SHALL avisar con «Las contraseñas no coinciden.» sin enviar nada al servidor. Si el registro tiene éxito, SHALL llevar al perfil con la sesión iniciada.

#### Scenario: Contraseñas distintas

- **WHEN** en el registro se escribe "secreto123" como contraseña y "secreto124" en la repetición, y se pulsa «Crear cuenta»
- **THEN** bajo el campo de repetición aparece «Las contraseñas no coinciden.» y no se hace ninguna petición de registro

#### Scenario: Email ya registrado

- **WHEN** en el registro se usa un email que ya tiene cuenta
- **THEN** bajo el campo de email aparece «Ese email ya está registrado. Inicia sesión en su lugar.»

#### Scenario: Registro correcto

- **WHEN** se rellena el registro con datos válidos y se pulsa «Crear cuenta»
- **THEN** el botón muestra «Creando cuenta…» mientras espera y después se ve la página de perfil de la nueva cuenta

### Requirement: Formulario de inicio de sesión

La pantalla de inicio de sesión SHALL mostrar los campos «Email» y «Contraseña» y el botón «Entrar». Ante unas credenciales rechazadas, SHALL mostrar en un aviso «El email o la contraseña no son correctos.». Si el inicio de sesión tiene éxito, SHALL llevar al perfil.

#### Scenario: Credenciales incorrectas

- **WHEN** se inicia sesión con un email registrado y una contraseña equivocada
- **THEN** aparece el aviso «El email o la contraseña no son correctos.» y la persona sigue en la página de inicio de sesión

#### Scenario: Email con formato inválido

- **WHEN** se inicia sesión con el email "no-es-un-email"
- **THEN** bajo el campo de email aparece «Introduce una dirección de email válida.»

#### Scenario: Inicio correcto

- **WHEN** se inicia sesión con credenciales correctas
- **THEN** el botón muestra «Entrando…» mientras espera y después se ve la página de perfil

### Requirement: Recuperación de la sesión al recargar

La pantalla SHALL conservar la sesión iniciada entre recargas, comprobándola contra el servidor al cargar. Si el servidor rechaza la sesión, SHALL darla por terminada y explicarlo en la página de inicio de sesión. Si no se puede contactar con el servidor, SHALL mostrar la página de inicio de sesión con el motivo, sin descartar la sesión, de modo que se recupere al recargar cuando el servidor vuelva a responder.

#### Scenario: Recarga con sesión válida

- **WHEN** una persona con sesión iniciada recarga `/profile`
- **THEN** tras el indicador de carga sigue viendo su perfil, sin volver a iniciar sesión

#### Scenario: Sesión rechazada por el servidor

- **WHEN** la sesión guardada se revocó desde otro sitio y la persona recarga la página
- **THEN** acaba en la página de inicio de sesión con el aviso «Tu sesión ha caducado. Vuelve a iniciar sesión.»

#### Scenario: Servidor inalcanzable

- **WHEN** hay una sesión iniciada, el servidor está caído y se recarga la página
- **THEN** se ve la página de inicio de sesión con el aviso «No se pudo conectar con el servidor. Comprueba que el backend está arrancado.»; y si se recarga cuando el servidor vuelve a responder, se ve de nuevo el perfil sin iniciar sesión

### Requirement: Página de perfil

La página de perfil SHALL mostrar las iniciales, el nombre (o «Sin nombre» si la cuenta no tiene), el email y «Miembro desde» con la fecha de alta en formato largo en castellano.

#### Scenario: Cuenta sin nombre

- **WHEN** una cuenta registrada sin nombre abre su perfil
- **THEN** se ve «Sin nombre» como título, su email debajo y la fecha de alta junto a «Miembro desde», en un formato como «1 de octubre de 2026»

### Requirement: Cierre de sesión en pantalla

La página de perfil SHALL ofrecer el botón «Cerrar sesión». Al pulsarlo, SHALL dar por terminada la sesión en la pantalla y llevar a la página de inicio de sesión, aunque el servidor no confirme la revocación.

#### Scenario: Cierre de sesión

- **WHEN** una persona con sesión iniciada pulsa «Cerrar sesión» en su perfil
- **THEN** acaba en la página de inicio de sesión sin ningún aviso de error, y volver a abrir `/profile` la manda de nuevo a inicio de sesión

#### Scenario: Cierre con el servidor caído

- **WHEN** el servidor no responde y la persona pulsa «Cerrar sesión»
- **THEN** igualmente acaba en la página de inicio de sesión

---

## Parte B

### 1. Requisitos escritos por el agente y comprobados por mí

- Requisitos escritos por el agente: 15
- Requisitos comprobados por mí abriendo el código: 1

### 2. Incoherencias que aparecieron al escribirla

- El formato de respuesta no es uniforme: el inicio de sesión usa serialize(...) y el cierre devuelve directamente { message: 'Logged out successfully' }. Se observa en los métodos store y destroy de backend/app/controllers/access_tokens_controller.ts.
- La spec afirma que un token solo deja de aceptarse al cerrar sesión con él, pero mi revisión no alcanzó a demostrar esa exclusividad. Se observa en el requisito «Vigencia de los tokens».

### 3. Lo que no supe decidir si era un bug o el contrato

- Al revisar el cierre de sesión, inicialmente interpreté que cerraba todas las sesiones del usuario; el código selecciona únicamente el token actual. No pude determinar si cerrar solo la sesión actual es la decisión de producto o si se esperaba cerrar todas.

Alcance de mi comprobación: revisé con ayuda el requisito «Cierre de sesión por API» en access_tokens_controller.ts y routes.ts. Contrasté la selección del token actual, la respuesta directa y la protección de la ruta. No confirmé los códigos HTTP exactos ni ejecuté una petición reutilizando el token revocado.
