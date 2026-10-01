# Prompts

Aquí van **todos los prompts que lanzaste** para hacer el ejercicio, en el orden en que los
lanzaste, con el modelo y la herramienta de cada uno.

Esto no es papeleo. Lo que se revisa es **cómo pediste las cosas**, no solo lo que salió: un
resultado flojo con un prompt bueno y un resultado flojo con un prompt vago necesitan feedback
distinto, y sin este archivo no se distinguen.

## Cómo rellenarlo

- Un apartado `## Prompt N` por cada prompt.
- **Pega el prompt tal cual lo lanzaste**, dentro del bloque de código, aunque ocupe diez líneas
  y aunque tenga faltas. No lo reescribas para que quede bien: el que arreglaste mentalmente
  después no es el que lanzaste.
- Incluye también los que **no funcionaron**. Suelen ser los más útiles de leer.
- `Modelo` y `Herramienta` en todos. Si cambiaste de una a otra a mitad, se nota aquí.

Borra el ejemplo de abajo cuando escribas el primero.

---

## Prompt 1

**Modelo:** Opus 5.5 (según la interfaz)
**Herramienta:** Claude Code

```
Vamos a realizar el ejercicio del módulo 3 descrito en README.md: escribir la spec viva del comportamiento actual de cuentas y acceso.

Lee AGENTS.md, CLAUDE.md y las instrucciones del ejercicio en README.md.

Límites de esta tarea:
- Trabaja en la rama actual spec-viva-jr.
- No ejecutes operaciones Git que escriban: no crees ramas, commits, pushes ni PR. Yo manejaré Git.
- No modifiques código, pruebas, dependencias, configuración ni el harness. No corrijas los problemas que encuentres.
- No inicialices OpenSpec en este proyecto.
- Los únicos archivos que puedes escribir son prompts.md y docs/spec-viva/jr.md.

Primero registra este prompt completo y literal en prompts.md como Prompt 1, sustituyendo el ejemplo y conservando las instrucciones de la plantilla. Herramienta: Claude Code. Modelo: Opus 5.5, según la interfaz.

En esta primera fase explora el código, sin escribir todavía la spec:
1. Revisa cuentas y acceso de punta a punta: registro, inicio y cierre de sesión, recuperación del estado de sesión y perfil. Incluye backend y frontend.
2. Basa tus conclusiones en el código actual. Usa la documentación como orientación, no como prueba de que algo está implementado.
3. Identifica los archivos relevantes y resume qué comportamiento observable encontraste.
4. Separa lo confirmado por lectura de lo incierto o que exigiría ejecución. Si necesitas consultar dependencias, revisa las instaladas.
5. Propón un plan breve para redactar y revisar la spec con el formato exacto del ejercicio.

No escribas aún docs/spec-viva/jr.md. No rellenes mis listas de revisión ni atribuyas a mí verificaciones hechas por ti.

Termina mostrando el mapa de archivos y el plan. Espera mi revisión antes de redactar.
```

## Prompt 2

**Modelo:** Opus 5.5 (según la interfaz)
**Herramienta:** Claude Code

```
Apruebo el plan con los siguientes ajustes. Registra este prompt literal como Prompt 2 en prompts.md, con la herramienta y el modelo de esta sesión.

Redacta docs/spec-viva/jr.md conforme al formato del README:
- ## Purpose.
- ## Requirements.
- ### Requirement: con SHALL.
- Al menos un #### Scenario: por requisito, con viñetas WHEN y THEN. Las precondiciones van dentro del WHEN.
- Texto en castellano, conservando las palabras del formato.

Limita la spec al comportamiento actual de cuentas y acceso, en API y pantalla. Mantén requisitos concretos y comprobables; separa API y pantalla cuando tengan reglas diferentes.

No conviertas una incertidumbre en un requisito normativo. Si falta evidencia, deja ese comportamiento fuera de Requirements y menciónalo en tu respuesta como pendiente de confirmar.

Evita afirmar que los tokens “nunca caducan”. Distingue ausencia de vencimiento automático de revocación por cierre de sesión. Incluye solo lo que puedas sostener con la configuración y las dependencias instaladas.

No incluyas detalles internos como clases, archivos, almacenamiento del token o algoritmos. 

No añadas ADDED, MODIFIED ni REMOVED. No implementes mejoras ni corrijas código.

Al final deja los tres apartados de la Parte B pendientes de mi revisión

Puedes contar los requisitos escritos, pero no inventes cuántos comprobé yo ni rellenes mis conclusiones.

Mantén los límites anteriores: solo modifica docs/spec-viva/jr.md y prompts.md; sin operaciones Git de escritura ni inicialización de OpenSpec.

Al terminar indica cuántos requisitos redactaste y propone tres para que yo los contraste personalmente, indicando en el chat los archivos y fragmentos relevantes. Estas referencias no deben incluirse en Requirements.
```

## Prompt 3

**Modelo:** Opus 5.5 (según la interfaz)
**Herramienta:** Claude Code

```
Se terminaron los 45 minutos. No modifiques la Parte A de docs/spec-viva/jr.md ni sigas investigando.

Registra este prompt completo y literal como Prompt 3 en prompts.md, indicando herramienta y modelo de esta sesión.

Completa únicamente la Parte B de docs/spec-viva/jr.md con este contenido:

### 1. Requisitos escritos por el agente y comprobados por mí

- Requisitos escritos por el agente: 15
- Requisitos comprobados por mí abriendo el código: 1

### 2. Incoherencias que aparecieron al escribirla

- El formato de respuesta no es uniforme: el inicio de sesión usa serialize(...) y el cierre devuelve directamente { message: 'Logged out successfully' }. Se observa en los métodos store y destroy de backend/app/controllers/access_tokens_controller.ts.
- La spec afirma que un token solo deja de aceptarse al cerrar sesión con él, pero mi revisión no alcanzó a demostrar esa exclusividad. Se observa en el requisito «Vigencia de los tokens».

### 3. Lo que no supe decidir si era un bug o el contrato

- Al revisar el cierre de sesión, inicialmente interpreté que cerraba todas las sesiones del usuario; el código selecciona únicamente el token actual. No pude determinar si cerrar solo la sesión actual es la decisión de producto o si se esperaba cerrar todas.

Después de las tres listas, añade esta nota:

Alcance de mi comprobación: revisé con ayuda el requisito «Cierre de sesión por API» en access_tokens_controller.ts y routes.ts. Contrasté la selección del token actual, la respuesta directa y la protección de la ruta. No confirmé los códigos HTTP exactos ni ejecuté una petición reutilizando el token revocado.

Solo puedes modificar la Parte B de docs/spec-viva/jr.md y añadir este registro a prompts.md. No cambies código, configuración ni otros archivos. No ejecutes operaciones Git de escritura.

Al terminar, resume brevemente qué actualizaste.
```
