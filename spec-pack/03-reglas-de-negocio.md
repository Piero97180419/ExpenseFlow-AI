# 03 — Reglas de negocio — ExpenseFlow AI

| Campo | Valor |
|---|---|
| Documento | 03-reglas-de-negocio.md |
| Fase | Planificación (BMAD — Agente Business Analyst) |
| Versión | v1.0 |
| Fecha | 2026-11-10 |
| Estado | **Aprobado** |
| Aprobado por | Piero Carbajal |
| Fecha de aprobación | 2026-11-10 |
| PR de aprobación | PR que fusiona el Spec Pack v1.0 en `main` (ADR-005 §1) |
| Insumos | `00-constitucion.md` v1.0, `01-project-brief.md` v1.0, `02-alcance.md` v1.0, ADR-001 a ADR-007 v1.0 |
| Rol en `/docs` | Especificación SDD con criterios de aceptación (PRINCIPIO-012): cada regla EARS, con su ejemplo y su contraejemplo, es un criterio de aceptación |

---

## 0. Convenciones

### 0.1 Palabras normativas

- **DEBE**: obligatorio.
- **NO DEBE**: prohibido.
- **PUEDE**: opcional.

### 0.2 Patrones EARS (redacción en español)

| Patrón | Plantilla |
|---|---|
| Ubiquitous | «El sistema DEBE / NO DEBE …» |
| Event-driven | «CUANDO <evento>, el sistema DEBE …» |
| State-driven | «MIENTRAS <estado>, el sistema DEBE …» |
| Unwanted | «SI <condición no deseada>, ENTONCES el sistema DEBE …» |
| Optional | «DONDE <funcionalidad incluida>, el sistema DEBE …» |

### 0.3 Fuentes

- **PRINCIPIO-0NN**: principio de la constitución (`00-constitucion.md`).
- **M1–M6, S1–S5, C1–C4, W1–W20**: funcionalidad del alcance (`02-alcance.md`).
- **Brief**: `01-project-brief.md`.
- **DEC-NN**: decisión de Piero Carbajal, registrada en la sección 9.
- **ADR-NNN**: decisión arquitectónica en `/docs/adr/`.
- **Decisión del equipo**: regla definida al especificar el producto, sin una decisión de producto independiente.

### 0.4 Categorías usadas

Registro, Seguridad, Datos, Cálculo, Avisos, IA, UX.

> Las categorías **Crédito** y **Pagos** no se usan: ExpenseFlow AI no otorga crédito ni procesa pagos (PRINCIPIO-009, W2). Se usa **Avisos** y no «Notificaciones», porque el glosario reserva «notificación» para push, correo o SMS (W14).

### 0.5 Notación

- Importes con el formato de la UI (RN-081): punto decimal, coma de miles y dos decimales (`S/ 1,250.50`).
- Porcentajes en los ejemplos de cálculo: valor numérico (`82.50`). En la UI: `82.50 %` (RN-124).
- `UserId`: cadena GUID por defecto de ASP.NET Core Identity. En los ejemplos se abrevia: `demo1` = `7c9e6679-…`, `demo2` = `b3f1a2c4-…`.
- Los identificadores numéricos de los ejemplos (movimiento 42, presupuesto 7) son ilustrativos: las pruebas obtienen los `Id` consultando la base.

### 0.6 Vigencia, notas y alcance de las validaciones

- **Vigencia:** las reglas rigen desde que el Spec Pack v1.0 está en `main` (ADR-005 §1).
- **Notas:** los bloques de nota («>») que acompañan a una regla son **normativos**: forman parte de ella y se implementan y prueban. Solo no son normativos los párrafos que empiezan con «**Motivo:**» (explican por qué) o «**Sugerencia:**» (una forma posible de implementarlo).
- **Identificadores:** son estables. Una regla nueva se agrega en su sección temática con el siguiente número libre; los números no se reutilizan.
- **Validaciones:** las de movimientos y presupuestos (RN-051 a RN-063, RN-104 a RN-106, RN-109, RN-139) se aplican **al crear y al editar** (DEC-34).
- **Fecha y hora:** «hoy», «mes en curso» y cualquier instante actual se obtienen de `TimeProvider` en la zona horaria `America/Lima` (RN-137).
- **Mensajes:** las reglas citan los mensajes por su ID; el texto exacto está en la sección 10.

### 0.7 Resumen del catálogo

| Prioridad | Reglas | Cantidad |
|---|---|---|
| MUST | RN-001 a RN-043, RN-048 a RN-088, RN-094 a RN-134, RN-137 a RN-142, RN-144 a RN-160, RN-162 a RN-178 | 165 |
| SHOULD | RN-044 a RN-047, RN-089 a RN-093, RN-135, RN-136, RN-143, RN-161 | 13 |
| **Total** | RN-001 a RN-178 | **178** |

---

## 1. Cuenta y acceso (M1)

**RN-001** | Categoría: Registro | Prioridad: MUST | Fuente: PRINCIPIO-009 · Alcance M1

Patrón EARS: Event-driven

CUANDO un visitante envía el formulario de registro sin marcar la casilla «Declaro ser mayor de 18 años», el sistema NO DEBE crear la cuenta.

✅ **Ejemplo:** El 2026-11-08 un visitante ingresa `nuevo1@expenseflow.test`, marca la casilla y envía el formulario: la cuenta se crea.

❌ **Contraejemplo:** El 2026-11-08 un visitante envía el formulario con la casilla desmarcada y se crea la cuenta `nuevo2@expenseflow.test`.

> Los correos `demo1` a `demo6` los crea el seed (sección 11), así que no sirven para probar el registro. Las pruebas E2E de registro usan un correo único por ejecución (`e2e-<GUID>@expenseflow.test`), para no chocar con RN-095.

---

**RN-002** | Categoría: UX | Prioridad: MUST | Fuente: PRINCIPIO-009, PRINCIPIO-012 · Alcance M1

Patrón EARS: Unwanted

SI un visitante envía el formulario de registro sin marcar la casilla de mayoría de edad, ENTONCES el sistema DEBE mostrar un mensaje de error en texto asociado a esa casilla.

✅ **Ejemplo:** Debajo de la casilla aparece «Debes declarar que eres mayor de 18 años para crear tu cuenta» (MSG-01), vinculado mediante `aria-describedby`.

❌ **Contraejemplo:** La casilla solo cambia su borde a rojo, sin ningún texto.

---

**RN-003** | Categoría: Seguridad | Prioridad: MUST | Fuente: PRINCIPIO-008

Patrón EARS: Ubiquitous

El sistema DEBE generar el hash de cada contraseña exclusivamente con el `PasswordHasher` de ASP.NET Core Identity con su configuración por defecto.

✅ **Ejemplo:** El usuario `demo1@expenseflow.test` se registra con la contraseña `Demo#2026x`; la columna `PasswordHash` guarda el valor producido por `UserManager.CreateAsync` (PBKDF2-HMAC-SHA512).

❌ **Contraejemplo:** Un método propio `HashPassword` guarda `SHA256.HashData(Encoding.UTF8.GetBytes("Demo#2026x"))`.

---

**RN-004** | Categoría: Seguridad | Prioridad: MUST | Fuente: PRINCIPIO-008

Patrón EARS: Event-driven

CUANDO el sistema crea un usuario nuevo, el sistema DEBE asignarle `LockoutEnabled = true`.

✅ **Ejemplo:** El usuario `demo2@expenseflow.test`, creado el 2026-11-08, tiene `LockoutEnabled = 1` en `AspNetUsers`.

❌ **Contraejemplo:** El usuario `demo3@expenseflow.test` se crea con `LockoutEnabled = 0`.

> Parámetros del bloqueo: RN-048 y RN-049. Contador de intentos: RN-144.

---

**RN-005** | Categoría: Seguridad | Prioridad: MUST | Fuente: PRINCIPIO-008

Patrón EARS: Ubiquitous

El sistema DEBE redirigir toda petición HTTP a HTTPS en todos los entornos.

✅ **Ejemplo:** En el entorno `Demo`, una petición a `http://192.168.1.20:5000/dashboard` recibe una redirección a `https://192.168.1.20:5001/dashboard`.

❌ **Contraejemplo:** En el entorno `Development`, `http://localhost:5000/dashboard` responde `200 OK` con el dashboard sin cifrar.

> Entornos definidos en `02-alcance.md`, sección 3.4: `Development`, `Testing` y `Demo`. HSTS: RN-100. Versión mínima de TLS: RN-156.

---

**RN-006** | Categoría: Seguridad | Prioridad: MUST | Fuente: PRINCIPIO-008

Patrón EARS: Ubiquitous

El sistema NO DEBE escribir contraseñas en ningún log.

✅ **Ejemplo:** Tras un inicio de sesión fallido, el log registra `Failed login attempt for user 7c9e6679-…`.

❌ **Contraejemplo:** El log registra `Login: demo1@expenseflow.test Demo#2026x`.

---

**RN-048** | Categoría: Seguridad | Prioridad: MUST | Fuente: DEC-09 · PRINCIPIO-008

Patrón EARS: Unwanted

SI un usuario acumula 5 intentos fallidos consecutivos de inicio de sesión, ENTONCES el sistema DEBE bloquear su cuenta durante 5 minutos.

✅ **Ejemplo:** `demo1` falla 5 veces y el quinto fallo ocurre el 2026-11-08 a las 10:01:00. Con la contraseña correcta, el intento de las 10:03:00 se rechaza y el de las 10:06:01 se acepta.

❌ **Contraejemplo:** Tras el quinto fallo se permite un sexto intento a las 10:01:30, o el bloqueo dura 15 minutos.

> Configuración: `MaxFailedAccessAttempts = 5`, `DefaultLockoutTimeSpan = TimeSpan.FromMinutes(5)`, `AllowedForNewUsers = true`. Incremento del contador: RN-144.

---

**RN-049** | Categoría: Seguridad | Prioridad: MUST | Fuente: DEC-09

Patrón EARS: Event-driven

CUANDO un usuario inicia sesión con éxito, el sistema DEBE reiniciar a 0 su contador de intentos fallidos.

✅ **Ejemplo:** `demo1` falla 3 veces, inicia sesión con éxito, cierra sesión y falla 2 veces más: la cuenta no se bloquea (contador = 2).

❌ **Contraejemplo:** En la misma secuencia la cuenta se bloquea porque el contador llegó a 5.

---

**RN-050** | Categoría: Seguridad | Prioridad: MUST | Fuente: DEC-09 · DEC-29

Patrón EARS: Unwanted

SI un inicio de sesión falla por cualquier causa, incluida una cuenta bloqueada, ENTONCES el sistema DEBE mostrar el mensaje MSG-02.

✅ **Ejemplo:** Se muestra «No se pudo iniciar sesión. Verifica tu correo y contraseña, o inténtalo de nuevo en unos minutos.» en tres casos: `noexiste@expenseflow.test`, `demo1@expenseflow.test` con contraseña incorrecta y `demo1@expenseflow.test` bloqueado.

❌ **Contraejemplo:** Para el correo no registrado se muestra «No existe una cuenta con ese correo», o la cuenta bloqueada redirige a la página `Lockout` de la plantilla de Identity.

> Un envío con el correo o la contraseña vacíos no es un intento de inicio de sesión: se aplica RN-154.
>
> **Motivo:** el texto menciona «unos minutos» para orientar al usuario bloqueado sin revelar el estado de la cuenta.

---

**RN-094** | Categoría: Seguridad | Prioridad: MUST | Fuente: Decisión del equipo

Patrón EARS: Unwanted

SI la contraseña elegida en el registro no tiene al menos 8 caracteres, una mayúscula, una minúscula, un dígito y un carácter no alfanumérico, ENTONCES el sistema NO DEBE crear la cuenta.

✅ **Ejemplo:** `Demo#2026x` (10 caracteres, con mayúscula, minúscula, dígitos y «#») se acepta.

❌ **Contraejemplo:** Se acepta `Ab#1x` (5 caracteres) o `demo20261` (sin mayúscula ni símbolo).

> Configuración de Identity: `Password.RequiredLength = 8`, `RequireUppercase = true`, `RequireLowercase = true`, `RequireDigit = true`, `RequireNonAlphanumeric = true`. Mensaje: MSG-03.

> Ayuda visible bajo el campo «Contraseña»: MSG-47 (DEC-87).

---

**RN-095** | Categoría: Registro | Prioridad: MUST | Fuente: Decisión del equipo

Patrón EARS: Unwanted

SI un visitante intenta registrarse con un correo que ya pertenece a otra cuenta, ENTONCES el sistema NO DEBE crear la cuenta.

✅ **Ejemplo:** Existe `demo1@expenseflow.test`; el registro de `DEMO1@expenseflow.test` se rechaza (la comparación no distingue mayúsculas).

❌ **Contraejemplo:** Se crean dos cuentas con `demo1@expenseflow.test`.

> Configuración: `User.RequireUniqueEmail = true`. Mensaje: MSG-04.

---

**RN-096** | Categoría: Registro | Prioridad: MUST | Fuente: Decisión del equipo · Alcance W14, W18

Patrón EARS: Ubiquitous

El sistema NO DEBE exigir confirmación de correo electrónico para iniciar sesión.

✅ **Ejemplo:** `nuevo1@expenseflow.test` se registra y puede iniciar sesión de inmediato.

❌ **Contraejemplo:** Tras registrarse, la cuenta queda sin poder iniciar sesión hasta confirmar un correo que nunca se envía.

> Configuración: `SignIn.RequireConfirmedAccount = false`. La plantilla de Identity trae `true` por defecto. La recuperación de contraseña queda fuera del alcance (W17).

---

**RN-097** | Categoría: Seguridad | Prioridad: MUST | Fuente: Alcance M1 · DEC-48

Patrón EARS: Event-driven

CUANDO el usuario pulsa «Cerrar sesión», el sistema DEBE eliminar la cookie de autenticación del navegador.

✅ **Ejemplo:** Tras cerrar sesión, el navegador ya no tiene la cookie: `GET /dashboard` redirige a la página de inicio de sesión y `GET /api/dashboard` responde `401`.

❌ **Contraejemplo:** Tras cerrar sesión, el navegador conserva la cookie y el usuario puede seguir registrando gastos.

> Después de cerrar sesión se muestra la página de inicio de sesión. Que el botón «Atrás» no muestre datos se garantiza con RN-140.
>
> **Limitación aceptada (DEC-48):** la cookie es autocontenida. Una copia capturada antes del cierre sigue siendo válida hasta su expiración (RN-098). Se acepta en el MVP porque solo hay datos ficticios (PRINCIPIO-006).

> Si la sesión ya no es vigente al pulsar «Cerrar sesión», se aplica RN-177.

---

**RN-098** | Categoría: Seguridad | Prioridad: MUST | Fuente: Decisión del equipo · DEC-49

Patrón EARS: Unwanted

SI una solicitud autenticada llega después del instante de expiración de su cookie de autenticación, ENTONCES el sistema DEBE rechazarla como no autenticada.

✅ **Ejemplo:** `demo1` inicia sesión a las 10:00 (la cookie expira a las 11:00). La solicitud de las 10:20 no renueva la cookie, porque no pasó la mitad de la ventana. La de las 10:40 sí la renueva y la expiración pasa a las 11:40. La solicitud de las 11:41 se rechaza: en MVC redirige al inicio de sesión y en la API responde `401`.

❌ **Contraejemplo:** La solicitud de las 11:41 se acepta, o la sesión sigue válida al día siguiente.

> La cookie expira 60 minutos después de emitirse o de su última renovación. Se renueva con la primera solicitud autenticada que llega pasada la mitad de la ventana (30 minutos). Por eso la sesión expira **entre 30 y 60 minutos sin actividad** (DEC-49). «Actividad» = cualquier solicitud autenticada. Configuración: `ExpireTimeSpan = TimeSpan.FromMinutes(60)`, `SlidingExpiration = true`.
>
> **Limitación aceptada (DEC-59):** si la sesión expira mientras el usuario llena un formulario, al enviarlo se le redirige al inicio de sesión, no se guarda nada y lo escrito se pierde. Tras iniciar sesión vuelve al formulario vacío (RN-155).

> Al volver a la página de inicio de sesión por expiración se muestra MSG-48 (RN-171).

---

**RN-099** | Categoría: UX | Prioridad: MUST | Fuente: Decisión del equipo · PRINCIPIO-006 · Alcance W7

Patrón EARS: Ubiquitous

El formulario de registro DEBE mostrar el texto MSG-05.

✅ **Ejemplo:** Sobre el botón «Crear cuenta» aparece «Versión académica de demostración. No ingreses datos personales ni financieros reales.»

❌ **Contraejemplo:** El formulario de registro no advierte que la aplicación solo admite datos ficticios.

---

**RN-100** | Categoría: Seguridad | Prioridad: MUST | Fuente: PRINCIPIO-008

Patrón EARS: State-driven

MIENTRAS el entorno de ejecución sea distinto de `Development`, el sistema DEBE enviar la cabecera `Strict-Transport-Security`.

✅ **Ejemplo:** En el entorno `Demo`, la respuesta de `/dashboard` incluye `Strict-Transport-Security`.

❌ **Contraejemplo:** En el entorno `Testing`, la cabecera no aparece.

> En `Demo` la aplicación se usa por IP en la red local, y los navegadores ignoran HSTS en direcciones IP. La cabecera se envía igual para mantener la regla uniforme en todos los entornos.

---

**RN-101** | Categoría: Seguridad | Prioridad: MUST | Fuente: Decisión del equipo

Patrón EARS: Unwanted

SI una solicitud `POST`, `PUT` o `DELETE` llega sin un token antiforgery válido, ENTONCES el sistema DEBE rechazarla con `400 Bad Request` sin modificar datos.

✅ **Ejemplo:** Un formulario de otro sitio envía `POST /transactions/create` con la cookie de `demo1` pero sin token: la respuesta es `400` y no se crea el gasto.

❌ **Contraejemplo:** El gasto se crea porque la cookie de sesión es válida.

> Aplica a los formularios MVC y a la API REST, que se autentica con la misma cookie de Identity (RN-102). En la API, el token se envía en la cabecera `X-CSRF-TOKEN` y se obtiene con `GET /api/antiforgery/token` (ADR-004). Swagger UI solo se publica en `Development` (ADR-004), donde se usa el mismo endpoint para obtener el token.
>
> Si una solicitud no tiene sesión ni token, prevalece el `401` de RN-102: la autenticación se evalúa antes que el token. El token está ligado a la sesión: el cliente lo pide **después** de iniciar sesión, y uno obtenido antes deja de ser válido. `GET /api/antiforgery/token` no exige sesión (DEC-60), para que ese caso sea verificable. En la API, el orden completo de evaluación está en ADR-004, sección 3.1 (DEC-58).

> En las vistas MVC, el rechazo se presenta con la página de RN-170. Excepción: `POST /account/logout` sin sesión vigente (RN-177).

---

**RN-102** | Categoría: Seguridad | Prioridad: MUST | Fuente: Decisión del equipo

Patrón EARS: Unwanted

SI una solicitud a un endpoint bajo `/api/` llega sin sesión autenticada, ENTONCES el sistema DEBE responder `401 Unauthorized`.

✅ **Ejemplo:** `GET /api/transactions` sin cookie responde `401`.

❌ **Contraejemplo:** La misma solicitud responde `302` hacia la página de inicio de sesión, o `200` con una lista vacía.

> La API REST usa la cookie de autenticación de Identity (mismo origen). No se usan tokens JWT, para no añadir paquetes (PRINCIPIO-002). Las vistas MVC sí redirigen a la página de inicio de sesión.

> La respuesta `401` lleva el `ProblemDetails` estándar de ASP.NET Core con cuerpo, escrito en el evento `OnRedirectToLogin` (DEC-88).

---

**RN-140** | Categoría: Seguridad | Prioridad: MUST | Fuente: Decisión del equipo

Patrón EARS: Ubiquitous

El sistema DEBE enviar la cabecera `Cache-Control: no-store` en toda respuesta que requiera sesión autenticada.

✅ **Ejemplo:** La respuesta de `GET /dashboard` incluye `Cache-Control: no-store`; tras cerrar sesión, el botón «Atrás» del navegador vuelve a solicitar la página y se redirige al inicio de sesión.

❌ **Contraejemplo:** Tras cerrar sesión, «Atrás» muestra desde la caché del navegador el dashboard con los saldos.

---

**RN-144** | Categoría: Seguridad | Prioridad: MUST | Fuente: PRINCIPIO-008 · Decisión del equipo

Patrón EARS: Event-driven

CUANDO un intento de inicio de sesión con contraseña falla para una cuenta existente, el sistema DEBE incrementar en 1 su contador de intentos fallidos.

✅ **Ejemplo:** `demo1` falla 2 veces: `AccessFailedCount = 2` en `AspNetUsers`.

❌ **Contraejemplo:** Tras 10 fallos, `AccessFailedCount` sigue en 0 y la cuenta nunca se bloquea.

> Implementación: `PasswordSignInAsync(..., lockoutOnFailure: true)`. La plantilla de Identity usa `false` por defecto. Bloqueo: RN-048.

---

**RN-145** | Categoría: Seguridad | Prioridad: MUST | Fuente: Alcance M1, W13, W17, W18, W19 · Decisión del equipo

Patrón EARS: Ubiquitous

El sistema NO DEBE exponer páginas ni endpoints de cuenta distintos del registro, el inicio de sesión y el cierre de sesión.

✅ **Ejemplo:** `GET /Identity/Account/ForgotPassword` y `GET /Identity/Account/Manage` responden `404`.

❌ **Contraejemplo:** La aplicación publica las páginas de la plantilla de Identity para recuperar la contraseña, cambiar el correo, activar la autenticación en dos pasos o descargar y borrar datos personales.

> No se usa `Microsoft.AspNetCore.Identity.UI` (ADR-001). Las vistas del módulo `Accounts` son vistas MVC propias (ADR-004).
>
> Rutas (DEC-63): `GET/POST /account/register`, `GET/POST /account/login` y `POST /account/logout` (`07` §4.9). Cualquier otra ruta de cuenta responde `404` (RN-169).

---

**RN-146** | Categoría: Seguridad | Prioridad: MUST | Fuente: Alcance W20 · Decisión del equipo

Patrón EARS: Ubiquitous

El formulario de inicio de sesión NO DEBE ofrecer la opción «Recordarme».

✅ **Ejemplo:** El formulario tiene solo los campos Correo y Contraseña y el botón «Iniciar sesión».

❌ **Contraejemplo:** El formulario muestra la casilla «Recordarme».

---

**RN-147** | Categoría: Seguridad | Prioridad: MUST | Fuente: Alcance W20 · Decisión del equipo

Patrón EARS: Ubiquitous

El sistema DEBE emitir la cookie de autenticación como cookie no persistente.

✅ **Ejemplo:** La cabecera `Set-Cookie` de la cookie de autenticación no tiene `Expires` ni `Max-Age`; al cerrar todas las ventanas del navegador y volver a abrirlo, se pide iniciar sesión.

❌ **Contraejemplo:** La cookie tiene `Expires` y sobrevive al cierre del navegador.

> Implementación: `isPersistent: false`. La expiración de RN-098 se aplica igual, porque va dentro del ticket de autenticación.

---

**RN-148** | Categoría: UX | Prioridad: MUST | Fuente: Alcance M1 · DEC-51

Patrón EARS: Ubiquitous

El formulario de registro DEBE tener exactamente estos 4 campos de entrada: Correo, Contraseña, Confirmar contraseña y la casilla «Declaro ser mayor de 18 años».

✅ **Ejemplo:** El formulario muestra los 4 campos, el texto MSG-05 (RN-099) y el botón «Crear cuenta».

❌ **Contraejemplo:** El formulario pide además nombre, apellidos o fecha de nacimiento, o no tiene «Confirmar contraseña».

> Tras un error, la casilla conserva el valor enviado; las contraseñas se vacían (DEC-75).

---

**RN-149** | Categoría: Registro | Prioridad: MUST | Fuente: Alcance M1 · Decisión del equipo

Patrón EARS: Unwanted

SI el visitante envía el formulario de registro sin correo, sin contraseña o sin confirmación de contraseña, ENTONCES el sistema NO DEBE crear la cuenta.

✅ **Ejemplo:** Con el correo vacío, bajo el campo Correo aparece «Ingresa tu correo.» (MSG-32) y no se crea la cuenta.

❌ **Contraejemplo:** Se muestra un mensaje en inglés de la validación por defecto, o se crea una cuenta sin contraseña.

> Mensajes: MSG-32, MSG-34 y MSG-36, uno por cada campo vacío. Orden de validación del correo: vacío (RN-149), longitud (RN-151), formato (RN-150), duplicado (RN-095).

---

**RN-150** | Categoría: Registro | Prioridad: MUST | Fuente: DEC-52

Patrón EARS: Unwanted

SI el correo ingresado en el registro, sin espacios exteriores, no cumple el patrón `^[A-Za-z0-9._-]+@[A-Za-z0-9-]+(\.[A-Za-z0-9-]+)+$`, ENTONCES el sistema NO DEBE crear la cuenta.

✅ **Ejemplo:** Se aceptan `ana.perez@gmail.com`, `ANA_P@uni.edu.pe` y `e2e-0f8c…@expenseflow.test`.

❌ **Contraejemplo:** Se aceptan `josé@correo.pe` (tilde), `ñandu@correo.pe` (ñ), `ana+1@gmail.com` (+), `ana@gmail` (dominio sin punto), `ana @gmail.com` (espacio interior) o `ana@mi_dominio.com` (guion bajo en el dominio).

> Mensaje: MSG-33. Solo se valida en el registro. En el inicio de sesión no se valida el formato: cualquier fallo muestra MSG-02 (RN-050). El patrón usa `[A-Za-z0-9]` explícito, no `\w`, para excluir tildes y ñ.

> En el formulario, el campo «Correo» usa `type="text"` con `inputmode="email"` y `autocomplete="email"`, para que el navegador no recorte ni transforme el valor antes de enviarlo (DEC-77).

---

**RN-151** | Categoría: Registro | Prioridad: MUST | Fuente: Decisión del equipo

Patrón EARS: Unwanted

SI el correo ingresado en el registro, sin espacios exteriores, supera 256 caracteres, ENTONCES el sistema NO DEBE crear la cuenta.

✅ **Ejemplo:** Se acepta un correo de exactamente 256 caracteres que cumple RN-150.

❌ **Contraejemplo:** Un correo de 257 caracteres provoca un error de base de datos o se guarda truncado.

> Mensaje: MSG-37.
>
> **Motivo:** la columna `Email` de Identity es `nvarchar(256)`.

---

**RN-152** | Categoría: Registro | Prioridad: MUST | Fuente: DEC-51

Patrón EARS: Unwanted

SI la confirmación de la contraseña no es idéntica a la contraseña, ENTONCES el sistema NO DEBE crear la cuenta.

✅ **Ejemplo:** Contraseña `Demo#2026x` y confirmación `Demo#2026X`: bajo «Confirmar contraseña» aparece «Las contraseñas no coinciden.» (MSG-35).

❌ **Contraejemplo:** La cuenta se crea con la contraseña del primer campo.

> La comparación distingue mayúsculas y minúsculas.

> Orden de los errores (DEC-76): «Contraseña» muestra MSG-34 y, si no está vacía, MSG-03; «Confirmar contraseña» muestra MSG-36 y, si no está vacía y la contraseña tampoco, MSG-35. Los errores de ambos campos pueden aparecer a la vez.

---

**RN-153** | Categoría: Registro | Prioridad: MUST | Fuente: DEC-51

Patrón EARS: Event-driven

CUANDO el sistema crea una cuenta desde el formulario de registro, el sistema DEBE iniciar la sesión del usuario nuevo y redirigirlo al dashboard.

✅ **Ejemplo:** `nuevo1@expenseflow.test` se registra y llega a `/dashboard`, que muestra «Presupuesto: No configurado» y MSG-41.

❌ **Contraejemplo:** Tras registrarse, el usuario llega a la página de inicio de sesión y debe escribir de nuevo sus datos.

---

**RN-154** | Categoría: UX | Prioridad: MUST | Fuente: Decisión del equipo

Patrón EARS: Unwanted

SI el formulario de inicio de sesión se envía con el correo o la contraseña vacíos, ENTONCES el sistema DEBE mostrar el mensaje del campo vacío sin intentar el inicio de sesión.

✅ **Ejemplo:** Con la contraseña vacía, bajo el campo aparece «Ingresa tu contraseña.» (MSG-34) y `AccessFailedCount` no cambia.

❌ **Contraejemplo:** Se muestra MSG-02, o el envío vacío cuenta como intento fallido para el bloqueo.

> Mensajes: MSG-32 y MSG-34.

> El correo se compara sin espacios exteriores, igual que en el registro (DEC-82).

---

**RN-155** | Categoría: Seguridad | Prioridad: MUST | Fuente: Decisión del equipo

Patrón EARS: Event-driven

CUANDO el usuario inicia sesión con éxito, el sistema DEBE redirigirlo a la dirección de retorno solo si es local a la aplicación, y en otro caso al dashboard.

✅ **Ejemplo:** Con `returnUrl=/transactions`, el usuario llega a la lista de movimientos. Con `returnUrl=https://sitio-externo.example`, llega a `/dashboard`.

❌ **Contraejemplo:** El sistema redirige a `https://sitio-externo.example`.

> Implementación: `Url.IsLocalUrl(returnUrl)` y `LocalRedirect`.

> El `returnUrl` viaja en un campo oculto del formulario y se conserva en cada nueva presentación tras un error (DEC-72). Si apunta a una ruta que solo admite `POST`, se aplica RN-174.

---

**RN-156** | Categoría: Seguridad | Prioridad: MUST | Fuente: PRINCIPIO-008 · Decisión del equipo

Patrón EARS: Ubiquitous

El sistema NO DEBE aceptar conexiones con versiones de TLS anteriores a 1.2.

✅ **Ejemplo:** Kestrel se configura con `SslProtocols = SslProtocols.Tls12 | SslProtocols.Tls13`; un cliente que solo ofrece TLS 1.1 no completa el handshake.

❌ **Contraejemplo:** Kestrel usa la configuración por defecto del sistema operativo y acepta TLS 1.0.

---

**RN-171** | Categoría: UX | Prioridad: MUST | Fuente: Alcance M1 · DEC-81

Patrón EARS: Event-driven

CUANDO el sistema muestra la página de inicio de sesión porque la sesión del usuario expiró, el sistema DEBE mostrar el aviso MSG-48 sobre el formulario.

✅ **Ejemplo:** `demo1` inicia sesión a las 10:00 y no hace nada hasta las 11:41; al pulsar «Guardar gasto» llega al inicio de sesión y ve «Tu sesión terminó. Inicia sesión para continuar.».

❌ **Contraejemplo:** El aviso aparece al abrir la aplicación por primera vez, o después de pulsar «Cerrar sesión».

> La expiración se detecta con una cookie de marca no persistente (`ef_session`, `HttpOnly`, `Secure`, sin datos del usuario) que se emite al iniciar sesión y se elimina al cerrar sesión. Si una ruta protegida recibe la marca sin una cookie de autenticación válida, la redirección al inicio de sesión agrega `reason=expired` y el sistema elimina la marca.

> El aviso es informativo: se muestra con `role="status"` y no reemplaza a MSG-02.

---

**RN-173** | Categoría: Seguridad | Prioridad: MUST | Fuente: Alcance M1 · DEC-73

Patrón EARS: Event-driven

CUANDO un usuario con sesión iniciada envía una solicitud a `/account/register` o a `/account/login`, el sistema DEBE redirigirlo a `/dashboard` sin procesar la solicitud.

✅ **Ejemplo:** `demo1`, con sesión iniciada, vuelve con «Atrás» a «Crear cuenta»: llega a `/dashboard` y no se crea ninguna cuenta.

❌ **Contraejemplo:** El usuario con sesión envía el formulario de registro y se crea una segunda cuenta que reemplaza su sesión.

---

**RN-174** | Categoría: Seguridad | Prioridad: MUST | Fuente: Alcance M1 · DEC-70

Patrón EARS: Event-driven

CUANDO el usuario inicia sesión con una dirección de retorno local que corresponde a una ruta que solo admite `POST`, el sistema DEBE redirigirlo a la página desde la que se envía ese formulario.

✅ **Ejemplo:** La sesión de `demo1` expiró y pulsó «Generar resumen del mes»: tras iniciar sesión llega a `/dashboard`, no a `GET /insights/monthly-summary`.

❌ **Contraejemplo:** Tras iniciar sesión, el navegador pide `GET /insights/monthly-summary` y recibe `404` o `405`.

> Rutas que solo admiten `POST` y su destino: `/insights/monthly-summary` ⇒ `/dashboard`; `/transactions/{id}/delete` ⇒ `/transactions`; cualquier otra ruta sin `GET` ⇒ `/dashboard`. Las rutas de formularios con `GET` propio (`/transactions/create`, `/transactions/{id}/edit`, `/budgets/create`, `/budgets/edit`) vuelven a su `GET`, como indica RN-098. `/account/logout` no llega a este caso (RN-177).

---

**RN-177** | Categoría: Seguridad | Prioridad: MUST | Fuente: Alcance M1 · DEC-71

Patrón EARS: Event-driven

CUANDO llega `POST /account/logout` sin una sesión autenticada vigente, el sistema DEBE redirigir a la página de inicio de sesión sin validar el token antiforgery y sin mostrar un error.

✅ **Ejemplo:** La sesión de `demo1` expiró y pulsa «Cerrar sesión»: llega al inicio de sesión, sin MSG-46 ni MSG-48.

❌ **Contraejemplo:** La respuesta es `400` por el token antiforgery, o el usuario vuelve al inicio de sesión con `returnUrl=/account/logout`.

> Con sesión vigente se valida el token (RN-101) y se aplica RN-097. La ruta es `[AllowAnonymous]` y valida el token de forma explícita solo si hay sesión.

---

## 2. Aislamiento de datos y servicios transversales (M1–M6)

**RN-007** | Categoría: Datos | Prioridad: MUST | Fuente: PRINCIPIO-007

Patrón EARS: Ubiquitous

El sistema DEBE filtrar toda consulta sobre datos financieros por el `UserId` obtenido del contexto de autenticación.

✅ **Ejemplo:** `TransactionService` filtra con `User.FindFirstValue(ClaimTypes.NameIdentifier)`; la sesión de `demo1` solo recibe movimientos con `UserId = 7c9e6679-…`.

❌ **Contraejemplo:** `GET /api/transactions?userId=b3f1a2c4-…` devuelve los movimientos de `demo2` a la sesión de `demo1`.

---

**RN-008** | Categoría: Seguridad | Prioridad: MUST | Fuente: PRINCIPIO-007 · Brief T2

Patrón EARS: Unwanted

SI un usuario autenticado envía una solicitud sobre un movimiento (`GET`, `PUT` o `DELETE`) o sobre un presupuesto (`PUT`) que pertenece a otro usuario, ENTONCES el sistema DEBE responder `404 Not Found`.

✅ **Ejemplo:** `demo1` envía `DELETE /api/transactions/42`, donde el movimiento 42 pertenece a `demo2`: la respuesta es `404` y el movimiento 42 sigue existiendo. Lo mismo ocurre con `PUT /api/budgets/7` si el presupuesto 7 es de `demo2`.

❌ **Contraejemplo:** La misma solicitud responde `403 Forbidden` (revela que el recurso existe) o `204 No Content` (lo elimina).

> Los presupuestos no tienen `DELETE` (RN-119) ni `GET` por identificador: solo `GET /api/budgets/current`, que devuelve el del propio usuario (ADR-004, sección 3).

---

**RN-009** | Categoría: Datos | Prioridad: MUST | Fuente: PRINCIPIO-007 · Brief 4.2

Patrón EARS: Ubiquitous

El sistema NO DEBE usar los datos financieros de los usuarios para entrenar o ajustar modelos de IA.

✅ **Ejemplo:** El repositorio no contiene ningún script que exporte la tabla `Transactions` hacia un dataset de entrenamiento.

❌ **Contraejemplo:** Un job exporta los movimientos de los usuarios demo a `finetune.jsonl` para ajustar el modelo de Ollama.

---

**RN-103** | Categoría: Seguridad | Prioridad: MUST | Fuente: PRINCIPIO-007 · Decisión del equipo

Patrón EARS: Ubiquitous

El sistema NO DEBE tomar el `UserId` de un recurso desde datos enviados por el cliente (ruta, query string o cuerpo) al crear o modificar movimientos y presupuestos.

✅ **Ejemplo:** `demo1` envía `POST /api/transactions` con `"userId": "b3f1a2c4-…"` en el cuerpo; el gasto se crea con `UserId = 7c9e6679-…` (`demo1`). Los DTO de entrada no tienen la propiedad `UserId`.

❌ **Contraejemplo:** El gasto se crea a nombre de `demo2`.

---

**RN-137** | Categoría: Datos | Prioridad: MUST | Fuente: Decisión del equipo · ADR-004

Patrón EARS: Ubiquitous

El sistema DEBE obtener la fecha y hora actual únicamente mediante el `TimeProvider` registrado en la aplicación.

✅ **Ejemplo:** `LimaClock` calcula «hoy» con `timeProvider.GetUtcNow()` convertido a `America/Lima`. Una prueba fija `FakeTimeProvider` en 2026-11-30 23:59 (Lima) y verifica que un gasto creado en ese instante cuenta en 2026-11.

❌ **Contraejemplo:** Un servicio usa `DateTime.Now`, y la misma prueba falla según la hora y la zona horaria de la máquina que la ejecuta.

> Se prohíben `DateTime.Now`, `DateTime.UtcNow`, `DateTime.Today`, `DateTimeOffset.Now` y `DateTimeOffset.UtcNow` fuera del registro de `TimeProvider`, en código de producción y de pruebas. Las columnas de fecha no tienen valores por defecto calculados en SQL Server (`GETDATE()`, `GETUTCDATE()`, `SYSDATETIME()`): la aplicación asigna el valor. Identity usa el mismo `TimeProvider` registrado (necesario para probar RN-048 con `FakeTimeProvider`). Un analizador o una prueba de arquitectura lo verifica en el CI (ADR-004, sección 5).

---

**RN-138** | Categoría: Datos | Prioridad: MUST | Fuente: PRINCIPIO-006 · Decisión del equipo · ADR-005

Patrón EARS: Ubiquitous

El seed DEBE crear las cuentas demo con los escenarios de la sección 11, fechando cada movimiento relativo a la fecha actual de `TimeProvider`.

✅ **Ejemplo:** Si el seed se ejecuta el 2026-11-08, los movimientos del «mes en curso» llevan fecha 2026-11-01 y los del «mes anterior», 2026-10-15. Si se ejecuta el 2027-03-01, llevan 2027-03-01 y 2027-02-15.

❌ **Contraejemplo:** El seed crea movimientos fijos en 2025-06, que quedan fuera de la ventana de RN-109 y no se pueden editar; o genera montos aleatorios, y las pruebas E2E no pueden verificar cifras.

> Las fechas elegidas (día 1 del mes en curso, día 15 del mes anterior) nunca son futuras (RN-057) y siempre están dentro de la ventana de RN-109. Idempotencia: RN-157. Usuarios ajenos: RN-158. Fecha de creación: RN-159.
>
> Lo ejecuta un humano o el CI; **un agente no lo ejecuta** contra una base existente, porque elimina datos de las cuentas demo (PRINCIPIO-010). Las contraseñas de las cuentas demo están versionadas y no son secretos (ADR-005, sección 4).
>
> Cuándo ejecutarlo y cómo aislar las pruebas que modifican datos: sección 11.4.

---

**RN-157** | Categoría: Datos | Prioridad: MUST | Fuente: PRINCIPIO-006 · Decisión del equipo

Patrón EARS: Event-driven

CUANDO el seed se ejecuta y una cuenta demo ya existe, el seed DEBE eliminar los movimientos y el presupuesto de esa cuenta y volver a crearlos según la sección 11.

✅ **Ejemplo:** El seed se ejecuta dos veces seguidas: `demo1` tiene 12 movimientos.

❌ **Contraejemplo:** Tras la segunda ejecución, `demo1` tiene 24 movimientos, o el seed falla porque la cuenta ya existe.

---

**RN-158** | Categoría: Datos | Prioridad: MUST | Fuente: PRINCIPIO-006 · DEC-53

Patrón EARS: Ubiquitous

El seed NO DEBE leer, modificar ni eliminar datos de usuarios distintos de `demo1@expenseflow.test` a `demo6@expenseflow.test`.

✅ **Ejemplo:** `ana.perez@gmail.com` tiene 3 movimientos. Se ejecuta el seed: termina con normalidad y los 3 movimientos de `ana.perez@gmail.com` siguen intactos.

❌ **Contraejemplo:** El seed se detiene porque existe un usuario fuera de `@expenseflow.test`, o elimina los movimientos de `ana.perez@gmail.com`.

> El seed identifica las cuentas demo por su correo exacto normalizado y filtra cada consulta y cada borrado por sus `UserId`. (ADR-005, sección 4).

---

**RN-159** | Categoría: Datos | Prioridad: MUST | Fuente: Decisión del equipo

Patrón EARS: Ubiquitous

El seed DEBE asignar a los movimientos de cada cuenta demo fechas de creación crecientes, separadas por 1 segundo, en el orden de la tabla 11.2 y a partir del instante actual de `TimeProvider`.

✅ **Ejemplo:** El primer movimiento de `demo1` tiene `CreatedAt = t`, el segundo `t + 1 s`, y el último (Estudios, Yape) `t + 11 s`. Por eso RN-018 preselecciona «Yape».

❌ **Contraejemplo:** Los 12 movimientos tienen el mismo `CreatedAt`, y el medio preseleccionado depende del motor de la base de datos.

---

**RN-164** | Categoría: UX | Prioridad: MUST | Fuente: PRINCIPIO-008, PRINCIPIO-012 · Decisión del equipo

Patrón EARS: Unwanted

SI ocurre un error no controlado al procesar una solicitud, ENTONCES el sistema DEBE responder con el mensaje MSG-40 sin mostrar detalles técnicos.

✅ **Ejemplo:** Con SQL Server detenido, `GET /dashboard` muestra una página de error con «Ocurrió un error inesperado. Inténtalo nuevamente en unos minutos.» y código `500`; `GET /api/dashboard` responde `500` con un `ProblemDetails` cuyo `title` es ese texto y cuyo `code` es `MSG-40`.

❌ **Contraejemplo:** La página muestra la traza de la excepción o un mensaje en inglés.

> En `Development` se permite la página de excepciones para desarrolladores. El detalle técnico va solo al log, respetando RN-006 y RN-133 (ADR-004, sección 7).

---

**RN-169** | Categoría: UX | Prioridad: MUST | Fuente: PRINCIPIO-001, PRINCIPIO-007 · DEC-64

Patrón EARS: Unwanted

SI una solicitud a las vistas MVC pide una ruta inexistente o un recurso de otro usuario, ENTONCES el sistema DEBE responder `404` con una página en español que muestre MSG-45, sin indicar si el recurso existe.

✅ **Ejemplo:** `demo1` abre `/transactions/42/edit`, donde el movimiento 42 pertenece a `demo2`, y luego `/transactions/99999/edit`, que no existe: ambas respuestas son `404` y muestran la misma página con «La página que buscas no existe.».

❌ **Contraejemplo:** El movimiento ajeno muestra «No tienes permiso para ver este movimiento» y el inexistente muestra la página `404` por defecto de ASP.NET Core en inglés.

> En la API, el `404` usa el `ProblemDetails` estándar, sin `code` (ADR-004 §3.1). Esta página no consulta la base de datos ni requiere sesión. Pantalla: `09-ui-spec.md` P-012.

---

**RN-170** | Categoría: UX | Prioridad: MUST | Fuente: PRINCIPIO-001 · DEC-71

Patrón EARS: Unwanted

SI una solicitud `POST` a las vistas MVC llega con un token antiforgery no válido, ENTONCES el sistema DEBE responder `400` con una página en español que muestre MSG-46 y el enlace «Volver al inicio», sin modificar datos.

✅ **Ejemplo:** Un visitante abre «Crear cuenta», inicia sesión en otra pestaña y luego envía el registro: ve «Tu formulario caducó. Vuelve a la página e inténtalo de nuevo.» y no se crea la cuenta.

❌ **Contraejemplo:** La respuesta `400` llega sin contenido, con un texto en inglés, o con la traza del error.

> La página usa el layout público, no consulta la base de datos y no requiere sesión. «Volver al inicio» lleva a `/dashboard` (o al inicio de sesión si no hay sesión). En la API, el `400` de antiforgery sigue usando el `ProblemDetails` estándar (ADR-004 §3.1).

---

## 3. Registro de movimientos (M2, M3)

**RN-010** | Categoría: Datos | Prioridad: MUST | Fuente: PRINCIPIO-003

Patrón EARS: Ubiquitous

El sistema DEBE almacenar el monto de todo movimiento como `decimal` en C# mapeado a `DECIMAL(18,2)` en SQL Server.

✅ **Ejemplo:** Un gasto de S/ 12.50 se guarda en `Transactions.Amount` (tipo `decimal(18,2)`) con el valor `12.50`.

❌ **Contraejemplo:** La columna `Amount` es `FLOAT` y el gasto de S/ 12.50 se lee como `12.4999999`.

---

**RN-011** | Categoría: Cálculo | Prioridad: MUST | Fuente: PRINCIPIO-003

Patrón EARS: Ubiquitous

El sistema NO DEBE usar los tipos `float` o `double` para calcular importes.

✅ **Ejemplo:** `decimal total = expenses.Sum(e => e.Amount);` devuelve `450.00` para gastos de S/ 200.00 y S/ 250.00.

❌ **Contraejemplo:** `double total = expenses.Sum(e => (double)e.Amount);`

---

**RN-012** | Categoría: Registro | Prioridad: MUST | Fuente: PRINCIPIO-003 · Alcance W9

Patrón EARS: Ubiquitous

El sistema NO DEBE asociar a un movimiento una moneda distinta del sol peruano (PEN).

✅ **Ejemplo:** El gasto de S/ 45.00 del 2026-11-08 se guarda sin campo de moneda; todo importe se interpreta como PEN.

❌ **Contraejemplo:** El gasto se guarda con `Currency = "USD"` y `Amount = 12.00`.

---

**RN-013** | Categoría: UX | Prioridad: MUST | Fuente: PRINCIPIO-001, PRINCIPIO-003 · Decisión del equipo

Patrón EARS: Ubiquitous

El sistema DEBE formatear todo importe mediante el formato monetario compartido `MoneyFormat`, que aplica el patrón de RN-081.

✅ **Ejemplo:** Las vistas y la API usan `MoneyFormat.Format(45.00m)`, que devuelve «S/ 45.00».

❌ **Contraejemplo:** Una vista usa `amount.ToString("C")` con la cultura del servidor y muestra «$ 45.00» o «PEN 45.00».

> `MoneyFormat` parte de la cultura `es-PE` (PRINCIPIO-001) y fija explícitamente el símbolo «S/», el punto decimal, la coma de miles y el espacio normal, para que el resultado no dependa de la versión de ICU ni del sistema operativo.

---

**RN-014** | Categoría: Registro | Prioridad: MUST | Fuente: Alcance M2

Patrón EARS: Ubiquitous

El sistema DEBE restringir la categoría de un gasto a uno de estos valores: Alimentación, Transporte, Estudios, Entretenimiento, Salud, Otros.

✅ **Ejemplo:** Un gasto de S/ 3.50 del 2026-11-08 se registra con la categoría «Transporte».

❌ **Contraejemplo:** La API acepta `POST /api/transactions` con la categoría «Mascotas».

---

**RN-015** | Categoría: Registro | Prioridad: MUST | Fuente: Alcance M2

Patrón EARS: Ubiquitous

El sistema DEBE restringir el medio de pago de un gasto a uno de estos valores: Efectivo, Yape, Plin, Tarjeta, Otro.

✅ **Ejemplo:** Un gasto de S/ 15.00 del 2026-11-08 se registra con el medio de pago «Plin».

❌ **Contraejemplo:** La API acepta un gasto con el medio de pago «Transferencia BCP».

---

**RN-016** | Categoría: UX | Prioridad: MUST | Fuente: Alcance M2 · Brief U2 · Decisión del equipo

Patrón EARS: Ubiquitous

El formulario «Nuevo gasto» DEBE tener exactamente estos 4 campos de entrada: Monto, Categoría, Medio de pago, Fecha.

✅ **Ejemplo:** El formulario muestra los 4 campos y el botón «Guardar gasto». El tipo de movimiento no es un campo: lo determina el formulario («Nuevo gasto» o «Nuevo ingreso»).

❌ **Contraejemplo:** El formulario agrega un campo «Descripción», un selector de tipo, o un campo oculto que el usuario puede desplegar.

> Los botones y los campos ocultos no editables (por ejemplo, el token antiforgery) no son campos de entrada.

---

**RN-017** | Categoría: UX | Prioridad: MUST | Fuente: Alcance M2 · Brief 2.1

Patrón EARS: Event-driven

CUANDO el usuario abre el formulario «Nuevo gasto», el sistema DEBE proponer como fecha la fecha actual en la zona horaria `America/Lima`.

✅ **Ejemplo:** El usuario abre el formulario el 2026-11-08 a las 23:30 (hora de Lima): el campo Fecha muestra `2026-11-08`.

❌ **Contraejemplo:** En el mismo instante el campo muestra `2026-11-09`, porque el servidor usa la fecha UTC (2026-11-09 04:30Z).

> La fecha por defecto la calcula el servidor. La fecha del movimiento se almacena como `DateOnly` (columna `date`), ya en hora de Lima; la fecha de creación (`CreatedAt`) se almacena como `DateTime` en UTC (columna `datetime2`).

---

**RN-018** | Categoría: UX | Prioridad: MUST | Fuente: Alcance M2 · Decisión del equipo

Patrón EARS: Event-driven

CUANDO un usuario con al menos un gasto registrado abre el formulario «Nuevo gasto», el sistema DEBE preseleccionar el medio de pago del gasto existente con la fecha de creación más reciente.

✅ **Ejemplo:** El último gasto creado por `demo1` (`CreatedAt` 2026-11-08 10:15) usó «Yape»; al abrir el formulario, el medio de pago aparece como «Yape». Si ese gasto se elimina, se toma el siguiente más reciente.

❌ **Contraejemplo:** Se toma el gasto con la **fecha del movimiento** más reciente, o se sigue preseleccionando el medio de pago de un gasto ya eliminado.

> Si dos gastos tienen la misma fecha de creación, se toma el de mayor `Id`. Sin gastos previos: RN-108.

---

**RN-019** | Categoría: Registro | Prioridad: MUST | Fuente: PRINCIPIO-009 · Alcance W1, W2

Patrón EARS: Ubiquitous

El sistema NO DEBE establecer comunicación con bancos o billeteras digitales al registrar un movimiento.

✅ **Ejemplo:** El usuario registra un gasto de S/ 8.00 con «Yape»; el sistema solo guarda `PaymentMethod = Yape` en la base de datos local.

❌ **Contraejemplo:** Al guardar el gasto, el sistema llama a un servicio externo de Yape para validar la operación.

---

**RN-020** | Categoría: Cálculo | Prioridad: MUST | Fuente: Alcance M2, M3 · Brief T1

Patrón EARS: Event-driven

CUANDO el usuario elimina un movimiento propio, el sistema DEBE excluir su monto de todos los cálculos.

✅ **Ejemplo:** Los gastos de 2026-11 suman S/ 450.00; el usuario elimina un gasto de S/ 20.00 y el total pasa a S/ 430.00.

❌ **Contraejemplo:** Tras la eliminación, el total de gastos de 2026-11 sigue en S/ 450.00.

---

**RN-021** | Categoría: Cálculo | Prioridad: MUST | Fuente: Alcance M2, M3 · Brief T1

Patrón EARS: Event-driven

CUANDO el usuario corrige el monto de un movimiento propio, el sistema DEBE usar el monto corregido en todos los cálculos.

✅ **Ejemplo:** El usuario cambia un gasto del 2026-11-05 de S/ 30.00 a S/ 25.00; el total de gastos de 2026-11 pasa de S/ 450.00 a S/ 445.00.

❌ **Contraejemplo:** El total sigue en S/ 450.00 porque se conserva el monto original.

---

**RN-051** | Categoría: Registro | Prioridad: MUST | Fuente: DEC-01 · PRINCIPIO-003

Patrón EARS: Unwanted

SI el monto de un movimiento es menor que S/ 0.01, ENTONCES el sistema DEBE rechazar el movimiento.

✅ **Ejemplo:** Se aceptan los gastos de S/ 0.01 y de S/ 150.50.

❌ **Contraejemplo:** Se acepta un gasto de S/ 0.00 o un ingreso de S/ -50.00.

> Mensaje: MSG-07.

---

**RN-052** | Categoría: Registro | Prioridad: MUST | Fuente: DEC-01

Patrón EARS: Unwanted

SI el monto de un movimiento es mayor que S/ 100,000.00, ENTONCES el sistema DEBE rechazar el movimiento.

✅ **Ejemplo:** Se acepta un ingreso de S/ 100,000.00.

❌ **Contraejemplo:** Se acepta un ingreso de S/ 100,000.01.

> Mensaje: MSG-08.

---

**RN-053** | Categoría: Registro | Prioridad: MUST | Fuente: DEC-02 · DEC-30

Patrón EARS: Unwanted

SI el monto ingresado de un movimiento o de un presupuesto tiene más de 2 decimales significativos, ENTONCES el sistema DEBE rechazarlo.

✅ **Ejemplo:** En un formulario se aceptan `12.3` y `12.34`. En la API, que recibe un número JSON, también se acepta `12.340`, porque el cero final no es significativo, y se guarda como 12.34.

❌ **Contraejemplo:** Se acepta `12.345` o `12.999`, o un presupuesto de `600.555`.

> En los formularios, el texto se valida antes con RN-104, que solo admite hasta 2 dígitos tras el separador: `12.340` escrito en un formulario se rechaza con MSG-09.
>
> **Sugerencia:** `amount * 100 == decimal.Truncate(amount * 100)`.

---

**RN-054** | Categoría: UX | Prioridad: MUST | Fuente: DEC-02 · PRINCIPIO-012

Patrón EARS: Unwanted

SI el sistema rechaza un monto por RN-053, ENTONCES el sistema DEBE mostrar el mensaje en texto MSG-09.

✅ **Ejemplo:** El usuario ingresa `12.345`; bajo el campo Monto aparece «El monto debe tener como máximo dos decimales», vinculado con `aria-describedby`.

❌ **Contraejemplo:** El campo solo se pinta de rojo, o aparece el texto genérico «Monto inválido».

---

**RN-055** | Categoría: Registro | Prioridad: MUST | Fuente: DEC-02 · DEC-03 · DEC-30

Patrón EARS: Ubiquitous

El sistema NO DEBE redondear un monto ingresado por el usuario, sea de un movimiento o de un presupuesto.

✅ **Ejemplo:** El usuario ingresa `12.345`; el sistema rechaza el movimiento (RN-053) y no guarda nada.

❌ **Contraejemplo:** El sistema guarda `12.35` sin avisar al usuario.

---

**RN-056** | Categoría: Registro | Prioridad: MUST | Fuente: DEC-01

Patrón EARS: Ubiquitous

El sistema DEBE determinar si un movimiento es ingreso o gasto únicamente por su tipo.

✅ **Ejemplo:** Un gasto de S/ 45.00 se guarda como `Type = Expense`, `Amount = 45.00`.

❌ **Contraejemplo:** Un gasto se guarda como `Amount = -45.00` para distinguirlo de un ingreso.

---

**RN-057** | Categoría: Registro | Prioridad: MUST | Fuente: DEC-04

Patrón EARS: Unwanted

SI la fecha de un movimiento es posterior a la fecha actual en la zona horaria `America/Lima`, ENTONCES el sistema DEBE rechazar el movimiento.

✅ **Ejemplo:** El 2026-11-08 a las 23:30 (hora de Lima) se acepta un gasto con fecha `2026-11-08`.

❌ **Contraejemplo:** En el mismo instante se acepta un gasto con fecha `2026-11-09`, porque en UTC ya es 2026-11-09.

> Mensaje: MSG-10. Límite hacia el pasado: RN-109.

---

**RN-058** | Categoría: Registro | Prioridad: MUST | Fuente: DEC-13

Patrón EARS: Unwanted

SI el usuario guarda un ingreso sin origen, ENTONCES el sistema DEBE rechazar el ingreso.

✅ **Ejemplo:** Un ingreso de S/ 600.00 con origen «Apoyo familiar» se acepta.

❌ **Contraejemplo:** Un ingreso de S/ 600.00 sin origen se guarda con `Source = null`.

> Mensaje: MSG-14.

---

**RN-059** | Categoría: Registro | Prioridad: MUST | Fuente: DEC-13

Patrón EARS: Ubiquitous

El sistema DEBE restringir el origen de un ingreso a uno de estos valores: Apoyo familiar, Trabajo, Prácticas preprofesionales, Trabajo independiente, Otros.

✅ **Ejemplo:** Un ingreso de S/ 350.00 del 2026-11-05 se registra con el origen «Prácticas preprofesionales».

❌ **Contraejemplo:** La API acepta un ingreso con el origen «Herencia».

> Identificadores internos: `FamilySupport`, `Job`, `Internship`, `Freelance`, `Other`.

---

**RN-060** | Categoría: UX | Prioridad: MUST | Fuente: DEC-13

Patrón EARS: State-driven

MIENTRAS el origen seleccionado de un ingreso sea «Otros», el sistema DEBE mostrar el campo opcional «Descripción».

✅ **Ejemplo:** El usuario elige «Otros»; aparece «Descripción», y el ingreso se guarda aunque ese campo quede vacío.

❌ **Contraejemplo:** Con «Otros», el campo no aparece, o el ingreso no se puede guardar con la descripción vacía.

---

**RN-061** | Categoría: Registro | Prioridad: MUST | Fuente: DEC-13 · DEC-32

Patrón EARS: Unwanted

SI la descripción del origen de un ingreso supera 100 caracteres, ENTONCES el sistema DEBE rechazar el ingreso.

✅ **Ejemplo:** Se acepta una descripción de exactamente 100 caracteres.

❌ **Contraejemplo:** Se acepta una de 101 caracteres, o se guarda truncada a 100 sin avisar.

> Mensaje: MSG-15.
>
> **Motivo:** rechazar en lugar de truncar sigue el criterio de DEC-02: no alterar en silencio lo que ingresa el usuario.

---

**RN-104** | Categoría: Registro | Prioridad: MUST | Fuente: Decisión del equipo

Patrón EARS: Unwanted

SI el texto ingresado en un formulario como monto de un movimiento o de un presupuesto no consiste en dígitos seguidos, opcionalmente, de un único separador decimal (punto o coma) y más dígitos, ENTONCES el sistema DEBE rechazarlo.

✅ **Ejemplo:** Se aceptan `3.50` y `3,50` (ambos S/ 3.50), `3`, `1250.5` y ` 12,50 ` (los espacios al inicio y al final se ignoran).

❌ **Contraejemplo:** Se aceptan `1,250.50`, `1.250,50`, `S/ 12`, `-5`, `.50`, `12.`, `12.5.0` o `١٢` (dígitos arábigo-índicos); o `3,50` se interpreta como S/ 350.00.

> **Orden de validación** (cada paso se detiene en el primer error):
> 0. Se quitan los espacios exteriores. Si el texto queda vacío → MSG-11 (o MSG-25 si es un presupuesto).
> 1. Patrón `^[0-9]+([.,][0-9]+)?$`. Si no cumple → MSG-06. Se usa `[0-9]`, no `\d`, porque en .NET `\d` también acepta dígitos de otros alfabetos.
> 2. Parte entera de más de 6 dígitos, contados **después de quitar los ceros a la izquierda** → MSG-08 (o MSG-20 si es un presupuesto), **sin convertir** el texto, para evitar desbordar `decimal`. Así, `0000012.50` se acepta como S/ 12.50.
> 3. Más de 2 dígitos tras el separador → MSG-09. Así, `1,250` se rechaza (no se interpreta como 1.25 ni como 1250).
> 4. Se reemplaza la coma por punto y se convierte con `CultureInfo.InvariantCulture` y `NumberStyles.AllowDecimalPoint`.
> 5. Rango: RN-051, RN-052, RN-062, RN-063.
>
> El campo usa `<input type="text" inputmode="decimal">`. La API REST no usa esta regla: recibe el monto como número JSON (RN-053).
>
> **Motivo:** muchos teclados numéricos de Android en español solo ofrecen la coma como separador decimal (decisión del 2026-11-08). Aceptarla sin separador de miles y con 2 decimales como máximo evita que `3,50` se guarde como S/ 350.00.

---

**RN-105** | Categoría: Registro | Prioridad: MUST | Fuente: Alcance M2 · Decisión del equipo

Patrón EARS: Unwanted

SI el usuario guarda un gasto sin monto, sin categoría, sin medio de pago o sin fecha, ENTONCES el sistema DEBE rechazar el gasto.

✅ **Ejemplo:** Un gasto con los 4 campos completos se guarda.

❌ **Contraejemplo:** Se guarda un gasto sin categoría, que luego no aparece en ningún total por categoría.

> Mensajes: MSG-11, MSG-12, MSG-13 y MSG-16, uno por cada campo vacío.

---

**RN-106** | Categoría: Registro | Prioridad: MUST | Fuente: Alcance M3 · Decisión del equipo

Patrón EARS: Unwanted

SI el usuario guarda un ingreso sin monto o sin fecha, ENTONCES el sistema DEBE rechazar el ingreso.

✅ **Ejemplo:** Un ingreso con monto, origen y fecha se guarda.

❌ **Contraejemplo:** Se guarda un ingreso sin fecha.

> El formulario «Nuevo ingreso» tiene estos campos: Monto, Origen, Fecha y, solo con «Otros», Descripción (RN-060). El ingreso **no** tiene medio de pago. El origen obligatorio está en RN-058. Mensajes: MSG-11 y MSG-16.

---

**RN-107** | Categoría: UX | Prioridad: MUST | Fuente: Decisión del equipo

Patrón EARS: Event-driven

CUANDO el usuario abre el formulario «Nuevo ingreso», el sistema DEBE proponer como fecha la fecha actual en la zona horaria `America/Lima`.

✅ **Ejemplo:** El 2026-11-08 a las 23:30 (hora de Lima), el campo Fecha muestra `2026-11-08`.

❌ **Contraejemplo:** El campo aparece vacío o muestra `2026-11-09`.

---

**RN-108** | Categoría: UX | Prioridad: MUST | Fuente: Alcance M2 · Decisión del equipo · **[Hipótesis]** a revisar con las entrevistas (`05` PQ-06)

Patrón EARS: Event-driven

CUANDO un usuario sin gastos registrados abre el formulario «Nuevo gasto», el sistema DEBE preseleccionar el medio de pago «Efectivo».

✅ **Ejemplo:** `demo4`, que no tiene movimientos (sección 11), abre el formulario: el medio de pago aparece como «Efectivo». Lo mismo ocurre con una cuenta recién registrada.

❌ **Contraejemplo:** El medio de pago aparece vacío y el usuario debe elegirlo antes de guardar.

> **Motivo:** «Efectivo» se elige porque el Brief lo señala como el medio de pago que ninguna app bancaria consolida (1.2). Si las entrevistas muestran otro medio predominante, se cambia el valor con un PR que actualice esta regla.

---

**RN-109** | Categoría: Registro | Prioridad: MUST | Fuente: DEC-04 · DEC-24

Patrón EARS: Unwanted

SI la fecha de un movimiento es anterior al primer día del mes que está 11 meses antes del mes en curso (hora de Lima), ENTONCES el sistema DEBE rechazar el movimiento.

✅ **Ejemplo:** El 2026-11-08, la fecha más antigua permitida es `2025-12-01`: un gasto con esa fecha se acepta. Así, el usuario puede registrar en 12 meses calendario (2025-12 a 2026-11).

❌ **Contraejemplo:** El 2026-11-08 se acepta un gasto con fecha `2025-11-30`.

> Mensaje: MSG-17. Al editar un movimiento ya fuera de la ventana, se aplica la misma regla (DEC-34): el movimiento puede eliminarse, pero no modificarse.

---

**RN-110** | Categoría: Registro | Prioridad: MUST | Fuente: DEC-13 · DEC-32

Patrón EARS: Event-driven

CUANDO el usuario cambia el origen de un ingreso de «Otros» a otro valor, el sistema DEBE descartar la descripción.

✅ **Ejemplo:** Un ingreso con origen «Otros» y descripción «Venta de libros» pasa a «Trabajo»; se guarda con `SourceDescription = null`.

❌ **Contraejemplo:** El ingreso de origen «Trabajo» conserva la descripción «Venta de libros».

> Aplica a los formularios MVC. En la API, un `sourceDescription` con un origen distinto de `Other` responde `400` con MSG-26 (ADR-004, sección 3.2).
>
> **DEC-66:** en los formularios MVC, al crear o al editar, si llega una descripción con un origen distinto de «Otros» (posible sin JavaScript, porque entonces el campo está siempre visible), el sistema la descarta y guarda `SourceDescription = null`, sin mostrar error. Ejemplo: sin JavaScript, el usuario elige «Trabajo», escribe «Venta de libros» y guarda S/ 100.00; el ingreso se crea con origen «Trabajo» y sin descripción. Contraejemplo: el formulario se rechaza con MSG-26, o el ingreso de «Trabajo» se guarda con la descripción.

---

**RN-111** | Categoría: Registro | Prioridad: MUST | Fuente: Alcance M2 · Decisión del equipo

Patrón EARS: Ubiquitous

El sistema DEBE permitir corregir el monto, la fecha, la categoría y el medio de pago de un gasto propio.

✅ **Ejemplo:** `demo1` cambia la categoría de un gasto de «Otros» a «Estudios»; el total de Estudios de su mes aumenta en ese monto.

❌ **Contraejemplo:** Solo se puede corregir el monto, o el gasto recibe un origen.

> Las validaciones de creación también se aplican al editar (0.6). El tipo no se puede cambiar (RN-112). Ingresos: RN-160.

---

**RN-160** | Categoría: Registro | Prioridad: MUST | Fuente: Alcance M3 · Decisión del equipo

Patrón EARS: Ubiquitous

El sistema DEBE permitir corregir el monto, la fecha, el origen y la descripción del origen de un ingreso propio.

✅ **Ejemplo:** `demo1` cambia el monto de un ingreso de S/ 600.00 a S/ 650.00 y su fecha del 2026-11-01 al 2026-11-02.

❌ **Contraejemplo:** El ingreso no permite corregir su fecha, o recibe una categoría o un medio de pago.

> Las validaciones de creación también se aplican al editar (0.6). Cambio de origen: RN-110. El tipo no se puede cambiar (RN-112).

---

**RN-112** | Categoría: Registro | Prioridad: MUST | Fuente: Decisión del equipo

Patrón EARS: Ubiquitous

El sistema NO DEBE permitir cambiar el tipo de un movimiento existente.

✅ **Ejemplo:** Para convertir un gasto en ingreso, el usuario elimina el gasto y registra un ingreso nuevo.

❌ **Contraejemplo:** La API acepta `PUT /api/transactions/42` con `"type": "Income"` sobre un gasto y conserva su categoría y medio de pago.

> En la API, un `type` distinto del actual responde `400` con MSG-30 (ADR-004, sección 3.1).

---

**RN-113** | Categoría: Cálculo | Prioridad: MUST | Fuente: Alcance M2, M3 · Brief T1 · Decisión del equipo

Patrón EARS: Event-driven

CUANDO el usuario cambia la fecha de un movimiento propio a otro mes, el sistema DEBE contar el movimiento solo en el mes de la nueva fecha.

✅ **Ejemplo:** Un gasto de S/ 30.00 pasa del 2026-10-31 al 2026-11-01: los gastos de 2026-10 bajan S/ 30.00 y los de 2026-11 suben S/ 30.00.

❌ **Contraejemplo:** El gasto sigue sumando en 2026-10, o suma en ambos meses.

---

**RN-114** | Categoría: UX | Prioridad: MUST | Fuente: Alcance M2, M3 · Decisión del equipo

Patrón EARS: Event-driven

CUANDO el usuario pulsa «Eliminar» sobre un movimiento, el sistema DEBE pedir confirmación con MSG-18 antes de eliminarlo.

✅ **Ejemplo:** Aparece «¿Eliminar este movimiento? Esta acción no se puede deshacer.» con los botones «Eliminar» y «Cancelar»; con «Cancelar» el movimiento sigue existiendo.

❌ **Contraejemplo:** El movimiento se elimina con un solo clic.

> El diálogo de confirmación es accesible por teclado y devuelve el foco al elemento que lo abrió (PRINCIPIO-012).
>
> **DEC-67:** sin JavaScript, «Eliminar» abre en la propia fila un `<details>` con MSG-18 y el botón «Eliminar definitivamente»; con JavaScript se usa un `<dialog>` modal con «Cancelar» y «Eliminar». Ambos envían el mismo `POST` y no hay ruta de confirmación (ADR-006).

---

**RN-115** | Categoría: Datos | Prioridad: MUST | Fuente: DEC-20

Patrón EARS: Event-driven

CUANDO el usuario confirma la eliminación de un movimiento propio, el sistema DEBE borrar el registro de la base de datos.

✅ **Ejemplo:** Tras eliminar el movimiento 42, `SELECT COUNT(*) FROM Transactions WHERE Id = 42` devuelve 0.

❌ **Contraejemplo:** El movimiento queda con `Status = Cancelled` y sigue apareciendo en la lista.

> No existe el concepto de «movimiento anulado» (DEC-20).

---

**RN-116** | Categoría: UX | Prioridad: MUST | Fuente: PRINCIPIO-012 · Decisión del equipo

Patrón EARS: Unwanted

SI una validación de las secciones 1, 3 o 4 rechaza un dato, ENTONCES el sistema DEBE mostrar el mensaje de la sección 10 que corresponde a esa regla, junto al campo afectado.

✅ **Ejemplo:** El usuario ingresa la fecha `2026-11-09` el 2026-11-08; bajo el campo Fecha aparece «La fecha no puede ser posterior a hoy.» (MSG-10), vinculado con `aria-describedby`.

❌ **Contraejemplo:** Aparece «Datos inválidos» en la parte superior del formulario, o un mensaje en inglés generado por la validación por defecto.

> La API REST devuelve `400` con un `ValidationProblemDetails` cuyo diccionario `errors` usa los mismos textos, con el nombre del campo en `camelCase` como clave (ADR-004, sección 3.1).

> Tras un error, el foco va al primer campo con error; el resumen de errores (MSG-44) no recibe foco (DEC-78). Los errores que no son de un campo (MSG-02, MSG-21, MSG-22) se muestran en un bloque propio con `role="alert"` sobre el formulario (DEC-80).

---

**RN-117** | Categoría: UX | Prioridad: MUST | Fuente: Alcance M2, M3 · DEC-19

Patrón EARS: Ubiquitous

La lista de movimientos DEBE mostrar los movimientos del usuario autenticado en páginas de 20, ordenados según RN-079.

✅ **Ejemplo:** `demo6` tiene 25 movimientos: la página 1 muestra 20 y la página 2 muestra 5. Cada fila tiene las acciones «Editar» y «Eliminar».

❌ **Contraejemplo:** La lista muestra todos los movimientos sin paginar, incluye filtros (son S3) o muestra movimientos de `demo2`.

> Desde el dashboard, el enlace «Ver todos los movimientos» lleva a esta lista. Configuración: `TransactionListPageSize = 20`.
>
> En la vista MVC, un número de página menor que 1 o no entero muestra la página 1; una página posterior a la última muestra la lista vacía con MSG-31 y un enlace a la página 1. Si el usuario no tiene ningún movimiento, la página 1 muestra MSG-41 (RN-162), no MSG-31. En la API, ver ADR-004, sección 3.2.

> Tras eliminar el último movimiento de una página posterior a la primera: RN-178. Tras guardar o eliminar: confirmación de RN-172.

---

**RN-168** | Categoría: UX | Prioridad: MUST | Fuente: Alcance M2, M3 · DEC-57

Patrón EARS: State-driven

MIENTRAS se envía el formulario «Nuevo gasto», «Nuevo ingreso» o de edición de un movimiento, el sistema DEBE mostrar su botón de envío deshabilitado.

✅ **Ejemplo:** Con la respuesta del servidor retrasada, el usuario pulsa «Guardar gasto» dos veces seguidas: el botón queda deshabilitado tras la primera pulsación y se crea 1 solo gasto.

❌ **Contraejemplo:** Cada pulsación envía el formulario y se crean dos gastos iguales.

> Se implementa como mejora progresiva con JavaScript (ADR-006). Si la respuesta vuelve con errores de validación, la página se recarga con el botón habilitado. Sin JavaScript, o si se pierde la respuesta y el usuario vuelve a enviar, el doble envío sigue siendo posible: limitación aceptada (DEC-57). En el resumen de IA se aplica RN-131.

> Los formularios de registro, inicio de sesión y presupuesto también deshabilitan su botón mientras se envían, con el mismo mecanismo (DEC-74).

---

**RN-172** | Categoría: UX | Prioridad: MUST | Fuente: Alcance M2, M3, M4 · DEC-69

Patrón EARS: Event-driven

CUANDO el sistema guarda o elimina un movimiento, o guarda un presupuesto, desde un formulario MVC, el sistema DEBE mostrar una sola vez en la página de destino el mensaje de confirmación correspondiente en un bloque con `role="status"`.

✅ **Ejemplo:** Un usuario de prueba guarda un gasto de S/ 3.50: `/dashboard` muestra «Gasto guardado.»; al recargar la página, el mensaje ya no aparece.

❌ **Contraejemplo:** No aparece ninguna confirmación, o la confirmación reaparece en cada recarga.

> Mensajes: gasto nuevo MSG-49, ingreso nuevo MSG-50, edición de un movimiento MSG-51, eliminación MSG-52, presupuesto creado o editado MSG-53. Viajan en TempData (Post/Redirect/Get). La API no muestra confirmaciones: responde con su código HTTP.

---

**RN-178** | Categoría: UX | Prioridad: MUST | Fuente: Alcance M2, M3 · DEC-86

Patrón EARS: Event-driven

CUANDO el usuario elimina el único movimiento de una página de la lista posterior a la primera, el sistema DEBE redirigirlo a la página anterior.

✅ **Ejemplo:** `demo6` tiene 21 movimientos y elimina el único de la página 2: llega a la página 1 con MSG-52.

❌ **Contraejemplo:** Llega a la página 2 vacía con MSG-31 («No hay más movimientos»), aunque la página 1 tiene 20 movimientos.

---

## 4. Presupuesto mensual (M4)

**RN-022** | Categoría: Registro | Prioridad: MUST | Fuente: Alcance M4

Patrón EARS: Event-driven

CUANDO el usuario guarda un presupuesto mensual, el sistema DEBE asociarlo al mes calendario en curso según la zona horaria `America/Lima`.

✅ **Ejemplo:** El 2026-11-08 el usuario guarda un presupuesto de S/ 600.00; queda asociado al mes `2026-11`.

❌ **Contraejemplo:** El presupuesto guardado el 2026-11-08 queda asociado al mes `2026-12`.

> En el formulario «Crear presupuesto», si el mes cambia entre que se abre y se envía el formulario, se aplica RN-167.

---

**RN-023** | Categoría: Cálculo | Prioridad: MUST | Fuente: PRINCIPIO-004 · Alcance M4

Patrón EARS: Ubiquitous

El sistema DEBE calcular el total de gastos del mes como la suma de los gastos del usuario cuya fecha, en la zona horaria `America/Lima`, pertenece a ese mes.

✅ **Ejemplo:** Gastos de una cuenta de prueba: 2026-10-31 S/ 100.00, 2026-11-01 S/ 200.00, 2026-11-15 S/ 250.00. El total de 2026-11 es S/ 450.00.

❌ **Contraejemplo:** El total de 2026-11 es S/ 550.00 porque incluye el gasto del 2026-10-31.

---

**RN-024** | Categoría: Cálculo | Prioridad: MUST | Fuente: PRINCIPIO-003, PRINCIPIO-004 · Alcance M4

Patrón EARS: State-driven

MIENTRAS el usuario tenga presupuesto para el mes, el sistema DEBE calcular el porcentaje consumido como (total de gastos del mes ÷ presupuesto del mes) × 100.

✅ **Ejemplo:** Presupuesto de 2026-11: S/ 600.00. Gastos de 2026-11: S/ 495.00. Porcentaje consumido: `82.50`.

❌ **Contraejemplo:** El porcentaje se toma del texto del modelo de IA («aproximadamente 80 %»), o se muestra `82.5000001` por un cálculo con `double`.

> El resultado se redondea según RN-025. Sin presupuesto: RN-066 y RN-068 a RN-070. El porcentaje puede superar 100.00 (RN-067).

---

**RN-025** | Categoría: Cálculo | Prioridad: MUST | Fuente: PRINCIPIO-003

Patrón EARS: Ubiquitous

El sistema DEBE redondear a 2 decimales todo valor monetario o porcentual que resulte de un cálculo, con la constante compartida `MoneyRounding.Mode`.

✅ **Ejemplo:** `Math.Round(value, 2, MoneyRounding.Mode)` en `BudgetService`.

❌ **Contraejemplo:** `Math.Round(value, 2)` sin modo explícito, que aplica `ToEven` por defecto, mientras otro servicio usa `AwayFromZero`.

> Decisión DEC-03: `MoneyRounding.Mode = MidpointRounding.AwayFromZero`. Como resultado de cálculo, 12.345 se redondea a 12.35, 12.355 a 12.36 y 12.344 a 12.34. El redondeo se aplica solo a resultados de cálculo, nunca a montos ingresados (RN-055).

---

**RN-026** | Categoría: Cálculo | Prioridad: MUST | Fuente: PRINCIPIO-004 · Alcance M4

Patrón EARS: Ubiquitous

El sistema DEBE calcular el restante del presupuesto como presupuesto del mes − total de gastos del mes.

✅ **Ejemplo:** Presupuesto de 2026-11: S/ 600.00. Gastos: S/ 450.00. Restante del presupuesto: S/ 150.00.

❌ **Contraejemplo:** El restante del presupuesto se calcula restando también los ingresos, o lo calcula el modelo de IA.

> El valor se conserva internamente con su signo. Cuando es negativo, se presenta según RN-071 a RN-073.

---

**RN-027** | Categoría: Cálculo | Prioridad: MUST | Fuente: PRINCIPIO-004

Patrón EARS: Ubiquitous

El sistema NO DEBE calcular cifras financieras oficiales fuera de la capa de lógica de negocio del backend.

✅ **Ejemplo:** El saldo del mes de S/ 105.00 lo calcula `BalanceCalculator` en el backend; la vista Razor solo lo muestra.

❌ **Contraejemplo:** Un script JavaScript de la vista suma los gastos de la tabla para mostrar el total.

> «Capa de lógica de negocio»: namespaces `ExpenseFlow.Modules.*.Domain` y `ExpenseFlow.Modules.*.Application` (PRINCIPIO-011, la constitución).

---

**RN-062** | Categoría: Registro | Prioridad: MUST | Fuente: DEC-07

Patrón EARS: Unwanted

SI el usuario guarda un presupuesto con monto menor que S/ 0.01, ENTONCES el sistema DEBE rechazar el presupuesto.

✅ **Ejemplo:** Se aceptan presupuestos de S/ 0.01 y de S/ 600.00.

❌ **Contraejemplo:** Se guarda un presupuesto de S/ 0.00 para 2026-11.

> Mensaje: MSG-19.

---

**RN-063** | Categoría: Registro | Prioridad: MUST | Fuente: DEC-07

Patrón EARS: Unwanted

SI el usuario guarda un presupuesto con monto mayor que S/ 100,000.00, ENTONCES el sistema DEBE rechazar el presupuesto.

✅ **Ejemplo:** Se acepta un presupuesto de S/ 100,000.00.

❌ **Contraejemplo:** Se acepta un presupuesto de S/ 100,000.01.

> Mensaje: MSG-20. El límite de 2 decimales (RN-053) y el formato de entrada (RN-104) también aplican al presupuesto.

---

**RN-139** | Categoría: Registro | Prioridad: MUST | Fuente: Decisión del equipo

Patrón EARS: Unwanted

SI el usuario guarda un presupuesto sin monto, ENTONCES el sistema DEBE rechazar el presupuesto.

✅ **Ejemplo:** El usuario envía «Crear presupuesto» con el campo Monto vacío: se muestra MSG-25 y no se guarda nada.

❌ **Contraejemplo:** Se guarda un presupuesto con `Amount = 0.00` o `NULL`, o la página muestra un error genérico.

---

**RN-064** | Categoría: Registro | Prioridad: MUST | Fuente: DEC-08 · DEC-17

Patrón EARS: Unwanted

SI el usuario intenta crear un presupuesto para un mes en el que ya tiene uno, ENTONCES el sistema NO DEBE crear un segundo presupuesto.

✅ **Ejemplo:** `demo1` tiene S/ 600.00 para 2026-11 e intenta crear otro de S/ 700.00 para 2026-11: el sistema lo rechaza, conserva el de S/ 600.00 y muestra «Ya tienes un presupuesto para este mes. Puedes cambiarlo desde "Editar presupuesto".»

❌ **Contraejemplo:** Quedan dos presupuestos para 2026-11, o el de S/ 700.00 reemplaza en silencio al de S/ 600.00.

> Mensaje: MSG-21 (RN-116). En la API: `409 Conflict` con MSG-21.
>
> Restricción en base de datos: `UNIQUE (UserId, Year, Month)` en la tabla `Budgets`. La tabla no tiene `CategoryId` en el MVP; entra con S1 mediante una tabla aparte. Dos envíos simultáneos se resuelven por la restricción: el segundo recibe el mismo mensaje.

---

**RN-065** | Categoría: Registro | Prioridad: MUST | Fuente: DEC-08

Patrón EARS: Event-driven

CUANDO el usuario guarda cambios desde «Editar presupuesto», el sistema DEBE actualizar el presupuesto existente de ese mes.

✅ **Ejemplo:** El presupuesto de 2026-11 pasa de S/ 600.00 a S/ 650.00, conserva su mismo `Id` y sigue habiendo 1 registro para ese mes.

❌ **Contraejemplo:** La edición crea un registro nuevo y quedan 2 presupuestos para 2026-11.

> Solo se puede editar el presupuesto del mes en curso (RN-118).

---

**RN-066** | Categoría: Cálculo | Prioridad: MUST | Fuente: DEC-07

Patrón EARS: State-driven

MIENTRAS el usuario no tenga presupuesto para el mes en curso, el sistema NO DEBE ejecutar el cálculo del porcentaje consumido.

✅ **Ejemplo:** Sin presupuesto para 2026-11 y con gastos por S/ 450.00, `ConsumedPercentage` vale `null`.

❌ **Contraejemplo:** El sistema lanza `DivideByZeroException`, o devuelve `0.00` o `100.00`.

---

**RN-067** | Categoría: Cálculo | Prioridad: MUST | Fuente: DEC-06 · DEC-07

Patrón EARS: Ubiquitous

El sistema NO DEBE limitar el porcentaje consumido a un máximo de 100.00.

✅ **Ejemplo:** Presupuesto S/ 500.00, gastos S/ 550.00: porcentaje consumido `110.00`, que se muestra como «110.00 %» (RN-124).

❌ **Contraejemplo:** En la misma situación el porcentaje se muestra como `100.00`.

---

**RN-118** | Categoría: Registro | Prioridad: MUST | Fuente: Alcance M4 · Decisión del equipo

Patrón EARS: Ubiquitous

El sistema DEBE permitir crear y editar solo el presupuesto del mes en curso.

✅ **Ejemplo:** El 2026-11-08, `demo1` edita el presupuesto de 2026-11. No existe ninguna pantalla ni endpoint para consultar o editar presupuestos de meses anteriores.

❌ **Contraejemplo:** `PUT /api/budgets/5` cambia el presupuesto de 2026-10, o se puede crear el de 2026-12.

> El presupuesto del mes siguiente no se puede fijar por adelantado; el usuario lo crea el día 1. Mensaje: MSG-22. En la API, `POST /api/budgets` siempre crea el presupuesto del mes en curso (no acepta mes); `PUT /api/budgets/{id}` sobre un presupuesto propio de otro mes responde `400 Bad Request` con MSG-22, y sobre uno de otro usuario, `404` (RN-008, ADR-004). Formulario «Crear presupuesto» enviado después del cambio de mes: RN-167.
>
> **Motivo:** limita el momento detonante (Brief 3.3) a cambio de simplicidad; se revisa al especificar S2.

---

**RN-119** | Categoría: Registro | Prioridad: MUST | Fuente: Decisión del equipo

Patrón EARS: Ubiquitous

El sistema NO DEBE permitir eliminar un presupuesto.

✅ **Ejemplo:** La pantalla «Editar presupuesto» solo tiene el botón «Guardar»; la API no expone `DELETE /api/budgets/{id}`.

❌ **Contraejemplo:** El usuario elimina el presupuesto de 2026-11 y el dashboard queda en «No configurado».

> Para cambiar el límite, el usuario lo edita (RN-065).

---

**RN-167** | Categoría: Registro | Prioridad: MUST | Fuente: DEC-56

Patrón EARS: Unwanted

SI el formulario «Crear presupuesto» se envía cuando el mes que mostraba ya no es el mes en curso, ENTONCES el sistema DEBE rechazarlo con MSG-22 sin crear ningún presupuesto.

✅ **Ejemplo:** El formulario se abre el 2026-11-30 a las 23:59 (mes 2026-11) y se envía el 2026-12-01 a las 00:00:30 (hora de Lima): se muestra MSG-22 y no existe presupuesto para 2026-12.

❌ **Contraejemplo:** Se crea en silencio el presupuesto de 2026-12.

> El formulario incluye un campo oculto no editable `month` (`yyyy-MM`) con el mes que muestra; no cuenta como campo de entrada. `POST /api/budgets` no cambia: no acepta mes y siempre crea el del mes en curso (RN-118).

> Tras mostrar MSG-22, el formulario se presenta con el mes en curso en el campo oculto `month` y en el rótulo, y conserva el monto escrito (DEC-84).

---

**RN-175** | Categoría: UX | Prioridad: MUST | Fuente: Alcance M4 · DEC-85

Patrón EARS: Event-driven

CUANDO un usuario que ya tiene presupuesto para el mes en curso abre «Crear presupuesto» (`GET /budgets/create`), el sistema DEBE redirigirlo a «Editar presupuesto».

✅ **Ejemplo:** `demo1` tiene S/ 600.00 para 2026-11 y abre `/budgets/create` desde un enlace guardado: llega a `/budgets/edit` con S/ 600.00 precargado.

❌ **Contraejemplo:** Ve el formulario vacío de «Crear presupuesto» y recién al enviarlo recibe MSG-21.

---

**RN-176** | Categoría: UX | Prioridad: MUST | Fuente: Alcance M4 · DEC-85

Patrón EARS: Event-driven

CUANDO un usuario sin presupuesto para el mes en curso abre «Editar presupuesto» (`GET /budgets/edit`), el sistema DEBE redirigirlo a «Crear presupuesto».

✅ **Ejemplo:** `demo3`, sin presupuesto, abre `/budgets/edit`: llega a `/budgets/create`.

❌ **Contraejemplo:** La respuesta es `404` o un formulario de edición vacío.

> El 1 de cada mes, «Editar presupuesto» abierto el mes anterior lleva a «Crear presupuesto» del mes nuevo (RN-118).

---

## 5. Dashboard y avisos (M5)

**RN-028** | Categoría: Avisos | Prioridad: MUST | Fuente: Alcance M5

Patrón EARS: State-driven

MIENTRAS el porcentaje consumido del mes en curso esté en el rango [80.00, 100.00), el sistema DEBE mostrar el aviso de umbral del 80 %.

✅ **Ejemplo:** Presupuesto S/ 600.00, gastos S/ 495.00, porcentaje `82.50`: el dashboard muestra «Alcanzaste el 80 % de tu presupuesto mensual».

❌ **Contraejemplo:** Con porcentaje `79.99` aparece el aviso del 80 %, o con porcentaje `85.00` no aparece ningún aviso.

> El porcentaje que se compara es el redondeado (RN-120).

---

**RN-029** | Categoría: Avisos | Prioridad: MUST | Fuente: Alcance M5

Patrón EARS: State-driven

MIENTRAS el porcentaje consumido del mes en curso sea mayor o igual que 100.00, el sistema DEBE mostrar el aviso de umbral del 100 %.

✅ **Ejemplo:** Presupuesto S/ 600.00, gastos S/ 650.00, porcentaje `108.33`: el dashboard muestra el aviso del 100 % (RN-127).

❌ **Contraejemplo:** Con porcentaje `108.33` el dashboard muestra el aviso del 80 %.

---

**RN-030** | Categoría: UX | Prioridad: MUST | Fuente: PRINCIPIO-012 · Alcance M5

Patrón EARS: Ubiquitous

El sistema DEBE comunicar cada aviso de umbral mediante un texto visible.

✅ **Ejemplo:** Junto a la barra de progreso aparece el texto «Alcanzaste el 80 % de tu presupuesto mensual».

❌ **Contraejemplo:** La barra de progreso cambia a color amarillo sin ningún texto.

> El aviso se presenta con `role="status"` para que lo anuncien los lectores de pantalla.

---

**RN-031** | Categoría: Avisos | Prioridad: MUST | Fuente: PRINCIPIO-004 · Alcance M5

Patrón EARS: Ubiquitous

El sistema NO DEBE condicionar la aparición de un aviso de umbral a una respuesta del modelo de IA.

✅ **Ejemplo:** Con el servicio de Ollama detenido y un porcentaje de `82.50`, el aviso del 80 % se muestra.

❌ **Contraejemplo:** El aviso solo aparece después de que Ollama devuelve el resumen del mes.

---

**RN-032** | Categoría: UX | Prioridad: MUST | Fuente: PRINCIPIO-009 · Alcance M5

Patrón EARS: Ubiquitous

El dashboard DEBE mostrar la leyenda «Saldo calculado según tus registros» junto a cada saldo que presenta.

✅ **Ejemplo:** La leyenda aparece bajo el saldo del mes (S/ 105.00) y bajo el saldo acumulado (S/ 155.00).

❌ **Contraejemplo:** La leyenda aparece solo bajo el saldo del mes, o el saldo acumulado se rotula «Saldo disponible en tu cuenta».

---

**RN-033** | Categoría: UX | Prioridad: MUST | Fuente: Alcance M5

Patrón EARS: Ubiquitous

El dashboard DEBE mostrar el total de gastos de cada categoría del mes en curso.

✅ **Ejemplo:** Para 2026-11 el dashboard muestra Alimentación S/ 270.00, Transporte S/ 120.00, Estudios S/ 105.00 (escenario `demo1`, §11.3).

❌ **Contraejemplo:** El dashboard muestra solo el total general de S/ 495.00, sin desglose por categoría.

> Se muestran solo las categorías con gastos mayores que S/ 0.00, ordenadas de mayor a menor importe. Si dos categorías tienen el mismo importe, van en el orden de la lista de RN-014 (Alimentación, Transporte, Estudios, Entretenimiento, Salud, Otros). Sin gastos en el mes: RN-163.

---

**RN-068** | Categoría: UX | Prioridad: MUST | Fuente: DEC-07

Patrón EARS: State-driven

MIENTRAS el usuario no tenga presupuesto para el mes en curso, el dashboard DEBE mostrar «No configurado» como valor del presupuesto.

✅ **Ejemplo:** `demo3` no tiene presupuesto para 2026-11 y el dashboard muestra «Presupuesto: No configurado».

❌ **Contraejemplo:** El dashboard muestra «Presupuesto: S/ 0.00».

---

**RN-069** | Categoría: UX | Prioridad: MUST | Fuente: DEC-07

Patrón EARS: State-driven

MIENTRAS el usuario no tenga presupuesto para el mes en curso, el dashboard DEBE mostrar «No disponible» como valor del porcentaje consumido.

✅ **Ejemplo:** El dashboard de `demo3` muestra «Porcentaje consumido: No disponible».

❌ **Contraejemplo:** El dashboard muestra «0.00 %» o una barra de progreso vacía sin texto.

---

**RN-070** | Categoría: UX | Prioridad: MUST | Fuente: DEC-07

Patrón EARS: State-driven

MIENTRAS el usuario no tenga presupuesto para el mes en curso, el dashboard DEBE mostrar el mensaje «Configura un presupuesto para controlar tus gastos».

✅ **Ejemplo:** El dashboard de `demo3` muestra ese mensaje con un enlace a «Crear presupuesto».

❌ **Contraejemplo:** No aparece ningún mensaje que oriente a configurar el presupuesto.

---

**RN-071** | Categoría: UX | Prioridad: MUST | Fuente: DEC-06 · Decisión del equipo

Patrón EARS: State-driven

MIENTRAS el total de gastos del mes supere el presupuesto del mes, el dashboard DEBE mostrar el restante del presupuesto como S/ 0.00.

✅ **Ejemplo:** Presupuesto S/ 500.00, gastos S/ 550.00: «Restante del presupuesto: S/ 0.00».

❌ **Contraejemplo:** En la misma situación se muestra «Restante del presupuesto: -S/ 50.00».

> Si los gastos son iguales al presupuesto (S/ 600.00 y S/ 600.00), el restante ya es S/ 0.00 por RN-026, y no hay exceso.
>
> **Motivo:** el glosario evita «disponible» para no confundirlo con un saldo bancario.

---

**RN-072** | Categoría: UX | Prioridad: MUST | Fuente: DEC-06

Patrón EARS: State-driven

MIENTRAS el total de gastos del mes supere el presupuesto del mes, el dashboard DEBE mostrar el estado «Presupuesto excedido».

✅ **Ejemplo:** Presupuesto S/ 500.00, gastos S/ 550.00: se muestra «Estado: Presupuesto excedido».

❌ **Contraejemplo:** Con S/ 550.00 de gastos no se muestra el estado, o se muestra con gastos de S/ 500.00 (igual al presupuesto, no superior).

---

**RN-073** | Categoría: UX | Prioridad: MUST | Fuente: DEC-06

Patrón EARS: State-driven

MIENTRAS el total de gastos del mes supere el presupuesto del mes, el dashboard DEBE mostrar como exceso el valor absoluto de (presupuesto del mes − total de gastos del mes).

✅ **Ejemplo:** Presupuesto S/ 500.00, gastos S/ 550.00: «Exceso: S/ 50.00».

❌ **Contraejemplo:** Se muestra «Exceso: -S/ 50.00», o no se muestra el exceso.

---

**RN-074** | Categoría: Cálculo | Prioridad: MUST | Fuente: DEC-05 · PRINCIPIO-004

Patrón EARS: Ubiquitous

El sistema DEBE calcular el saldo del mes como total de ingresos del mes − total de gastos del mes.

✅ **Ejemplo:** 2026-11: ingresos S/ 600.00, gastos S/ 495.00, saldo del mes S/ 105.00.

❌ **Contraejemplo:** El saldo del mes se muestra como S/ 155.00 porque incluye los movimientos de 2026-10.

---

**RN-075** | Categoría: Cálculo | Prioridad: MUST | Fuente: DEC-05 · PRINCIPIO-004

Patrón EARS: Ubiquitous

El sistema DEBE calcular el saldo acumulado como total de ingresos registrados − total de gastos registrados.

✅ **Ejemplo:** 2026-10: ingresos S/ 600.00, gastos S/ 550.00. 2026-11: ingresos S/ 600.00, gastos S/ 495.00. Saldo acumulado: S/ 1,200.00 − S/ 1,045.00 = S/ 155.00.

❌ **Contraejemplo:** El saldo acumulado muestra S/ 105.00 porque solo considera 2026-11.

> Los movimientos eliminados no existen en la base de datos (RN-115) y, por tanto, no participan.

---

**RN-076** | Categoría: UX | Prioridad: MUST | Fuente: DEC-05

Patrón EARS: Ubiquitous

El dashboard DEBE mostrar el saldo del mes.

✅ **Ejemplo:** El dashboard de `demo1` muestra «Saldo del mes: S/ 105.00».

❌ **Contraejemplo:** El dashboard muestra solo el saldo acumulado.

> Orden: RN-126. Saldo negativo: RN-123.

---

**RN-077** | Categoría: UX | Prioridad: MUST | Fuente: DEC-05

Patrón EARS: Ubiquitous

El dashboard DEBE mostrar el saldo acumulado.

✅ **Ejemplo:** El dashboard de `demo1` muestra «Saldo acumulado: S/ 155.00».

❌ **Contraejemplo:** El dashboard muestra solo el saldo del mes.

---

**RN-078** | Categoría: UX | Prioridad: MUST | Fuente: DEC-15

Patrón EARS: Ubiquitous

El dashboard DEBE mostrar los últimos movimientos del usuario autenticado, hasta un máximo de 5 (`DashboardRecentTransactionsLimit = 5`).

✅ **Ejemplo:** `demo1` tiene 12 movimientos y ve 5, más el enlace «Ver todos los movimientos» (RN-117). `demo3` tiene 3 y ve 3.

❌ **Contraejemplo:** `demo1` ve 10 movimientos, o ve un movimiento de `demo2`.

> Sin movimientos: RN-162.

---

**RN-079** | Categoría: Datos | Prioridad: MUST | Fuente: DEC-15

Patrón EARS: Ubiquitous

El sistema DEBE ordenar los movimientos por la clave (fecha del movimiento DESC, fecha de creación DESC, `Id` DESC).

✅ **Ejemplo:** Hay dos movimientos con fecha `2026-11-08`: el `Id` 41, creado a las 10:15, y el `Id` 40, creado a las 09:00. El 41 aparece primero.

❌ **Contraejemplo:** Los movimientos se ordenan por `Id` ascendente.

> Aplica a los últimos movimientos (RN-078) y a la lista de movimientos (RN-117).

---

**RN-080** | Categoría: UX | Prioridad: MUST | Fuente: DEC-15 · PRINCIPIO-012 · DEC-18

Patrón EARS: Ubiquitous

Cada movimiento de los últimos movimientos y de la lista de movimientos DEBE mostrar estos datos: fecha, categoría u origen, tipo de movimiento en texto, monto.

✅ **Ejemplo:** «08/11/2026 · Transporte · Gasto · S/ 3.50». Para un ingreso con origen «Otros» y descripción: «05/11/2026 · Otros: Venta de libros · Ingreso · S/ 80.00».

❌ **Contraejemplo:** El tipo se indica solo con color (verde o rojo), sin la palabra «Ingreso» o «Gasto»; o se agrega una columna «Descripción» vacía en todas las filas.

> La descripción solo existe para ingresos con origen «Otros» (RN-060) y se muestra unida al origen. El gasto no tiene descripción (RN-016). Formato de fecha: RN-125.

> La vista contiene como texto los separadores « · » de la fila y los dos puntos de los rótulos («Saldo del mes:»), para que las pruebas comparen el literal visible (DEC-83).

---

**RN-081** | Categoría: UX | Prioridad: MUST | Fuente: DEC-11 · PRINCIPIO-003

Patrón EARS: Ubiquitous

El sistema DEBE mostrar todo importe con el patrón «S/ #,##0.00», con un espacio normal (U+0020) entre «S/» y la cifra.

✅ **Ejemplo:** `1250.5m` se muestra como «S/ 1,250.50» y `2500m` como «S/ 2,500.00».

❌ **Contraejemplo:** «S/ 1.250,50», «S/ 2 500,00», «S/ 1250.5», o «S/ 1,250.50» con un espacio de no separación (U+00A0), que ICU inserta en algunos sistemas.

> Implementación: `MoneyFormat` (RN-013). Las pruebas comparan la cadena exacta. Importes negativos: RN-123.

---

**RN-120** | Categoría: Avisos | Prioridad: MUST | Fuente: Decisión del equipo

Patrón EARS: Ubiquitous

El sistema DEBE evaluar los umbrales de 80 % y 100 % con el porcentaje consumido ya redondeado según RN-024.

✅ **Ejemplo:** Presupuesto S/ 600.00, gastos S/ 599.97. El cociente exacto es 99.995 %, que se redondea a `100.00`. El dashboard muestra «100.00 %», el aviso del 100 %, «Restante del presupuesto: S/ 0.03» y **no** muestra «Presupuesto excedido», porque los gastos no superan el presupuesto.

❌ **Contraejemplo:** El dashboard muestra «100.00 %» junto al aviso del 80 %, porque la comparación usó el valor sin redondear.

> «Presupuesto excedido» se decide comparando importes (RN-072), no porcentajes.
>
> **Motivo:** así, el aviso siempre coincide con el porcentaje que ve el usuario.

---

**RN-121** | Categoría: Avisos | Prioridad: MUST | Fuente: DEC-07 · Decisión del equipo

Patrón EARS: State-driven

MIENTRAS el usuario no tenga presupuesto para el mes en curso, el sistema NO DEBE mostrar avisos de umbral.

✅ **Ejemplo:** `demo3`, sin presupuesto y con gastos por S/ 450.00, no ve ningún aviso de umbral; ve el mensaje de RN-070.

❌ **Contraejemplo:** Aparece el aviso del 100 % porque el sistema trató el presupuesto ausente como S/ 0.00.

---

**RN-122** | Categoría: UX | Prioridad: MUST | Fuente: DEC-07 · Decisión del equipo

Patrón EARS: State-driven

MIENTRAS el usuario no tenga presupuesto para el mes en curso, el dashboard NO DEBE mostrar el restante del presupuesto, el estado «Presupuesto excedido» ni el exceso.

✅ **Ejemplo:** El dashboard de `demo3` muestra presupuesto «No configurado», porcentaje «No disponible», el mensaje de RN-070, los saldos, el gasto por categoría y los últimos movimientos.

❌ **Contraejemplo:** Muestra «Restante del presupuesto: S/ 0.00» o «Exceso: S/ 450.00».

---

**RN-123** | Categoría: UX | Prioridad: MUST | Fuente: DEC-26

Patrón EARS: State-driven

MIENTRAS un saldo sea negativo, el dashboard DEBE mostrarlo con el signo «-» delante del símbolo «S/».

✅ **Ejemplo:** Ingresos de 2026-11: S/ 0.00; gastos: S/ 45.00. Se muestra «Saldo del mes: -S/ 45.00».

❌ **Contraejemplo:** Se muestra «S/ -45.00», «(S/ 45.00)», o «S/ 45.00» en rojo sin signo.

> Texto explicativo: RN-142.
>
> **Motivo:** es el caso habitual del primer uso: el usuario registra gastos antes de su primer ingreso.

---

**RN-142** | Categoría: UX | Prioridad: MUST | Fuente: DEC-26

Patrón EARS: State-driven

MIENTRAS un saldo sea negativo, el dashboard DEBE mostrar junto a él el texto «Tus gastos superan tus ingresos registrados».

✅ **Ejemplo:** «Saldo del mes: -S/ 45.00 · Tus gastos superan tus ingresos registrados».

❌ **Contraejemplo:** El saldo negativo solo se distingue por el color rojo.

> Si el saldo del mes y el acumulado son ambos negativos, el texto aparece junto a cada uno.

---

**RN-124** | Categoría: UX | Prioridad: MUST | Fuente: DEC-11 · DEC-25

Patrón EARS: Ubiquitous

El sistema DEBE mostrar todo porcentaje con 2 decimales, punto decimal y un espacio antes del símbolo «%».

✅ **Ejemplo:** `82.5` se muestra como «82.50 %» y `110` como «110.00 %».

❌ **Contraejemplo:** «82.5 %», «110 %», «82,50 %» o «82.50%».

> Excepción: los textos fijos de los avisos de umbral («Alcanzaste el 80 % …») usan el número entero del umbral.

---

**RN-125** | Categoría: UX | Prioridad: MUST | Fuente: PRINCIPIO-001 · DEC-35

Patrón EARS: Ubiquitous

El sistema DEBE mostrar las fechas en la UI con el formato `dd/MM/yyyy`.

✅ **Ejemplo:** La fecha `2026-11-08` se muestra como «08/11/2026».

❌ **Contraejemplo:** Se muestra «11/08/2026», «8/11/26» o «2026-11-08».

> Los campos de entrada usan `<input type="date">`, que el navegador presenta según su configuración y envía en formato ISO (`yyyy-MM-dd`). La API REST usa ISO 8601.

---

**RN-126** | Categoría: UX | Prioridad: MUST | Fuente: DEC-05 · DEC-27

Patrón EARS: Ubiquitous

El dashboard DEBE presentar el saldo del mes antes que el saldo acumulado en el orden de lectura.

✅ **Ejemplo:** En el DOM y en pantalla (móvil y escritorio), «Saldo del mes» aparece antes que «Saldo acumulado», con un encabezado de mayor nivel.

❌ **Contraejemplo:** En móvil, el saldo acumulado aparece primero.

> Encabezados: «Saldo del mes» usa `<h2>` y «Saldo acumulado» `<h3>`.
>
> **Motivo:** esto concreta «indicador principal» (DEC-05) de forma verificable con Playwright.

---

**RN-127** | Categoría: Avisos | Prioridad: MUST | Fuente: Alcance M5 · Decisión del equipo

Patrón EARS: State-driven

MIENTRAS se muestre el aviso de umbral del 100 %, su texto DEBE ser «Alcanzaste el 100 % de tu presupuesto mensual».

✅ **Ejemplo:** Con gastos de S/ 650.00 y presupuesto de S/ 600.00 se ven el aviso «Alcanzaste el 100 % de tu presupuesto mensual» y, por separado, «Estado: Presupuesto excedido» y «Exceso: S/ 50.00».

❌ **Contraejemplo:** El aviso dice «Te quedaste sin dinero», o desaparece cuando se muestra «Presupuesto excedido».

---

**RN-141** | Categoría: UX | Prioridad: MUST | Fuente: PRINCIPIO-012 · Decisión del equipo

Patrón EARS: State-driven

MIENTRAS el porcentaje consumido supere 100.00, la barra de progreso del presupuesto DEBE mostrarse llena con `aria-valuenow="100"`, `aria-valuemax="100"` y `aria-valuetext` igual al porcentaje real.

✅ **Ejemplo:** Con 110.00 %, la barra ocupa el 100 % del ancho y el lector de pantalla anuncia «110.00 %».

❌ **Contraejemplo:** La barra tiene `aria-valuenow="110"` con `aria-valuemax="100"` (atributo inválido), o se desborda del contenedor.

> Hasta 100.00, `aria-valuenow` es el porcentaje real redondeado a 2 decimales y `aria-valuetext` repite el formato de RN-124.

---

**RN-162** | Categoría: UX | Prioridad: MUST | Fuente: Alcance M2, M5 · Decisión del equipo

Patrón EARS: State-driven

MIENTRAS el usuario no tenga movimientos, el sistema DEBE mostrar MSG-41 en lugar de los últimos movimientos del dashboard y en lugar de la lista de movimientos.

✅ **Ejemplo:** `demo4` ve «Aún no tienes movimientos registrados.» en el dashboard y en la página 1 de la lista.

❌ **Contraejemplo:** La sección aparece vacía sin texto, o la lista muestra MSG-31 («No hay más movimientos»).

---

**RN-163** | Categoría: UX | Prioridad: MUST | Fuente: Alcance M5 · Decisión del equipo

Patrón EARS: State-driven

MIENTRAS el usuario no tenga gastos en el mes en curso, el dashboard DEBE mostrar MSG-42 en lugar del gasto por categoría.

✅ **Ejemplo:** `demo4`, y también un usuario que solo registró ingresos este mes, ven «Aún no tienes gastos registrados este mes.».

❌ **Contraejemplo:** La sección aparece vacía o lista las 6 categorías con S/ 0.00.

---

## 6. Resumen explicativo con IA (M6)

**RN-034** | Categoría: IA | Prioridad: MUST | Fuente: PRINCIPIO-005 · Alcance M6 · Decisión del equipo

Patrón EARS: Event-driven

CUANDO el usuario solicita el resumen del mes en curso, el sistema DEBE enviar al modelo, como datos del usuario, únicamente los campos `month`, `totalIncome`, `totalExpenses`, `monthlyBalance`, `budget`, `consumedPercentage`, `remainingBudget`, `overspentAmount`, `totalsByCategory`.

✅ **Ejemplo:** `{ "month": "2026-11", "totalIncome": 600.00, "totalExpenses": 495.00, "monthlyBalance": 105.00, "budget": 600.00, "consumedPercentage": 82.50, "remainingBudget": 105.00, "overspentAmount": null, "totalsByCategory": { "Alimentación": 270.00, "Transporte": 120.00, "Estudios": 105.00 } }`. Sin presupuesto: `"budget": null, "consumedPercentage": null, "remainingBudget": null, "overspentAmount": null`.

❌ **Contraejemplo:** El prompt incluye además la lista de los 34 movimientos individuales del mes con sus fechas.

> Siempre es el mes en curso: no se puede pedir el resumen de un mes anterior. `remainingBudget` es el valor que muestra el dashboard (mínimo 0.00, RN-071) y `overspentAmount` es el exceso de RN-073, o `null` si no hay exceso.
>
> El prompt completo = instrucciones fijas (RN-129) + este JSON. Las claves están en inglés (código, PRINCIPIO-001) y los nombres de categoría en español, para que el modelo los reproduzca tal como los ve el usuario. `totalsByCategory` solo incluye categorías con gastos mayores que S/ 0.00.

---

**RN-035** | Categoría: IA | Prioridad: MUST | Fuente: PRINCIPIO-005

Patrón EARS: Ubiquitous

El sistema NO DEBE incluir datos de identificación del usuario en ningún prompt.

✅ **Ejemplo:** El prompt del resumen de `demo1@expenseflow.test` no contiene su correo, nombre ni `UserId`.

❌ **Contraejemplo:** El prompt comienza con `"Usuario: Ana Pérez (ana@correo.com)"`.

---

**RN-036** | Categoría: IA | Prioridad: MUST | Fuente: PRINCIPIO-005 · Alcance W6

Patrón EARS: Ubiquitous

El sistema DEBE enviar toda solicitud de inferencia a una instancia de Ollama en un host de la lista de RN-087.

✅ **Ejemplo:** La URL base configurada es `http://localhost:11434`.

❌ **Contraejemplo:** La URL base configurada es `https://api.openai.com/v1` como respaldo.

---

**RN-037** | Categoría: IA | Prioridad: MUST | Fuente: Alcance M6

Patrón EARS: Ubiquitous

El sistema NO DEBE generar un resumen del mes sin una solicitud explícita del usuario.

✅ **Ejemplo:** El resumen se genera solo cuando el usuario pulsa el botón «Generar resumen del mes».

❌ **Contraejemplo:** Cada carga del dashboard dispara automáticamente una llamada a Ollama.

---

**RN-038** | Categoría: IA | Prioridad: MUST | Fuente: Alcance M6 · DEC-10

Patrón EARS: Unwanted

SI una solicitud a Ollama falla, ENTONCES el sistema DEBE mostrar el mensaje MSG-23.

✅ **Ejemplo:** Con el servicio de Ollama detenido, tras el tiempo de espera de RN-082, o con una respuesta sin texto, el usuario ve «El análisis con inteligencia artificial no está disponible en este momento. Inténtalo nuevamente» en lugar del resumen.

❌ **Contraejemplo:** El usuario ve una página `500 Internal Server Error`, un recuadro de resumen vacío, o el texto de v1.0 «El resumen no está disponible en este momento».

> «Falla» comprende: error de conexión, tiempo de espera agotado (RN-082), respuesta de error HTTP de Ollama (incluido el modelo no descargado), respuesta `200` cuyo texto generado está vacío o solo contiene espacios después de eliminar cualquier bloque `<think>…</think>` (ADR-003), y texto que contiene alguna cifra no válida (RN-166). En la API: `503` con `code` = `MSG-23` (ADR-004, sección 3).

---

**RN-039** | Categoría: IA | Prioridad: MUST | Fuente: Alcance M6

Patrón EARS: State-driven

MIENTRAS Ollama no esté disponible, el sistema DEBE mostrar el dashboard con todas sus cifras calculadas.

✅ **Ejemplo:** Con Ollama detenido, el dashboard muestra saldos, gasto por categoría, porcentaje «82.50 %» y últimos movimientos.

❌ **Contraejemplo:** Con Ollama detenido, el dashboard no carga o muestra sus cifras vacías.

---

**RN-040** | Categoría: UX | Prioridad: MUST | Fuente: Alcance M6

Patrón EARS: Ubiquitous

El sistema DEBE mostrar inmediatamente encima de cada resumen el texto «Explicación generada automáticamente».

✅ **Ejemplo:** El texto aparece como primera línea del recuadro del resumen del mes 2026-11.

❌ **Contraejemplo:** El texto aparece al pie, en otra sección de la página, o no aparece.

---

**RN-041** | Categoría: UX | Prioridad: MUST | Fuente: PRINCIPIO-009 · Alcance M6, W16

Patrón EARS: Ubiquitous

El sistema DEBE mostrar inmediatamente debajo de cada resumen el texto «No constituye asesoría financiera».

✅ **Ejemplo:** El texto aparece como última línea del recuadro del resumen del mes 2026-11.

❌ **Contraejemplo:** El resumen termina con «Te recomendamos invertir tus S/ 105.00 restantes en un fondo mutuo» sin el aviso.

---

**RN-042** | Categoría: IA | Prioridad: MUST | Fuente: PRINCIPIO-004 · Alcance W4

Patrón EARS: Ubiquitous

El sistema NO DEBE exponer al modelo de IA operaciones que creen, modifiquen o eliminen movimientos o presupuestos.

✅ **Ejemplo:** La integración con Ollama solo invoca el endpoint de generación de texto; el modelo no recibe herramientas.

❌ **Contraejemplo:** El modelo recibe una herramienta `delete_transaction(id)` que ejecuta `DELETE` sobre `Transactions`.

> La única llamada a Ollama es `POST /api/generate` con `stream: false` y `think: false` (ADR-003, sección 1).

---

**RN-043** | Categoría: IA | Prioridad: MUST | Fuente: PRINCIPIO-004 · Alcance W5

Patrón EARS: Ubiquitous

El sistema NO DEBE mostrar como cifra oficial un número producido por el modelo de IA.

✅ **Ejemplo:** El resumen dice «Este mes tu saldo es de S/ 105.00», cifra que copió del JSON (RN-130); la tarjeta «Saldo del mes» muestra S/ 105.00 calculado por `BalanceCalculator`.

❌ **Contraejemplo:** La tarjeta de saldo se llena extrayendo el número del texto generado por el modelo.

---

**RN-082** | Categoría: IA | Prioridad: MUST | Fuente: DEC-10

Patrón EARS: Unwanted

SI Ollama no responde dentro del tiempo configurado en `Ai:TimeoutSeconds`, ENTONCES el sistema DEBE cancelar la solicitud.

✅ **Ejemplo:** Con `Ai:TimeoutSeconds = 30` (valor por defecto), el modelo tarda 45 s: la solicitud se cancela a los 30 s y se muestra MSG-23.

❌ **Contraejemplo:** El usuario espera los 45 s, o la solicitud no tiene límite de tiempo.

> Rango válido de la configuración: RN-134.

---

**RN-083** | Categoría: IA | Prioridad: MUST | Fuente: DEC-10

Patrón EARS: Ubiquitous

El sistema NO DEBE reintentar automáticamente una solicitud a Ollama que haya fallado.

✅ **Ejemplo:** Una solicitud agota el tiempo de espera; el log registra 1 sola llamada HTTP y se muestra MSG-23.

❌ **Contraejemplo:** El cliente reintenta 3 veces y el usuario espera 90 s.

---

**RN-084** | Categoría: IA | Prioridad: MUST | Fuente: DEC-10

Patrón EARS: Unwanted

SI una solicitud a Ollama falla, ENTONCES el sistema DEBE registrar el incidente en el log técnico con el contenido de RN-133.

✅ **Ejemplo:** El log registra `Ollama request failed: type=Timeout duration=30012ms model=<modelo>`.

❌ **Contraejemplo:** La solicitud falla y no queda ningún registro.

---

**RN-085** | Categoría: IA | Prioridad: MUST | Fuente: DEC-12

Patrón EARS: Event-driven

CUANDO la aplicación arranca con Ollama no disponible, el sistema DEBE completar el arranque.

✅ **Ejemplo:** Con el servicio de Ollama detenido, se ejecuta `dotnet run` y la aplicación responde en `/`.

❌ **Contraejemplo:** El arranque se aborta con una excepción en una verificación de conexión con Ollama.

---

**RN-086** | Categoría: IA | Prioridad: MUST | Fuente: DEC-12 · ADR-002

Patrón EARS: Unwanted

SI `Ai:BaseUrl` está ausente o no es una URL absoluta con esquema `http` o `https`, ENTONCES el sistema NO DEBE enviar solicitudes de inferencia.

✅ **Ejemplo:** Con `Ai:BaseUrl = ""` o `Ai:BaseUrl = "localhost:11434"` (sin esquema), el sistema no realiza ninguna llamada HTTP, deshabilita la IA y registra una advertencia al arrancar.

❌ **Contraejemplo:** El sistema intenta llamar a `http:///api/generate` y lanza una excepción no controlada.

> También queda deshabilitada si `Ai:Model` está vacío o ausente (ADR-002). Que el modelo no esté descargado no se detecta al arrancar: es un fallo de la solicitud (RN-038). Presentación al usuario: RN-132.

---

**RN-087** | Categoría: IA | Prioridad: MUST | Fuente: PRINCIPIO-005 · ADR-002 · DEC-16

Patrón EARS: Unwanted

SI el host de `Ai:BaseUrl` no es `localhost`, `127.0.0.1` ni `::1`, ENTONCES el sistema NO DEBE enviar solicitudes de inferencia.

✅ **Ejemplo:** `http://localhost:11434` y `http://[::1]:11434` se aceptan; `http://ollama:11434`, `http://192.168.1.50:11434` y `http://203.0.113.10:11434` se rechazan y la IA queda deshabilitada.

❌ **Contraejemplo:** Se acepta `https://ollama.ejemplo.com` con una clave de API, una IP de la red local o un nombre de contenedor.

> La lista es una constante en código (`AiOptions.AllowedHosts`), no una configuración. La validación ocurre al arrancar (ADR-002).
>
> La comparación usa el tipo de host que obtiene `Uri` tras analizar la URL: si es un nombre (`UriHostNameType.Dns`), debe ser `localhost` (sin distinguir mayúsculas); si es una IP, su valor (`IPAddress`) debe ser igual a `127.0.0.1` o a `::1`. Así, `http://[::1]:11434` se acepta aunque `Uri.Host` devuelva `[::1]`, y `127.0.0.2` se rechaza (ADR-002).

---

**RN-088** | Categoría: IA | Prioridad: MUST | Fuente: DEC-10 · DEC-12

Patrón EARS: State-driven

MIENTRAS las funciones de IA estén deshabilitadas, el sistema DEBE ofrecer las funcionalidades M1 a M5 sin restricciones.

✅ **Ejemplo:** Con `Ai:BaseUrl` vacío, el usuario registra un gasto de S/ 3.50, edita su presupuesto y consulta el dashboard.

❌ **Contraejemplo:** El formulario «Nuevo gasto» devuelve un error porque intenta consultar a la IA.

---

**RN-128** | Categoría: Seguridad | Prioridad: MUST | Fuente: Decisión del equipo

Patrón EARS: Ubiquitous

El sistema NO DEBE interpretar el texto generado por el modelo como HTML ni como Markdown.

✅ **Ejemplo:** El modelo devuelve `<script>alert(1)</script>`; la vista lo muestra literalmente como texto, codificado por Razor, y no ejecuta nada.

❌ **Contraejemplo:** La vista usa `@Html.Raw(summary)`, o convierte el Markdown del modelo a HTML.

> Los saltos de línea del texto se conservan con CSS (`white-space: pre-line`).

---

**RN-129** | Categoría: IA | Prioridad: MUST | Fuente: ADR-002 · Decisión del equipo

Patrón EARS: Ubiquitous

El sistema DEBE tomar las instrucciones del modelo de un archivo versionado en el repositorio.

✅ **Ejemplo:** El cliente lee `src/ExpenseFlow/Modules/Insights/Prompts/monthly-summary.v1.md` y le añade el JSON de RN-034.

❌ **Contraejemplo:** Las instrucciones están concatenadas en un método C#, o se leen de `appsettings.json`.

> Contenido mínimo de las instrucciones: responder en español sencillo (es-PE); un máximo de 150 palabras; usar solo las cifras del JSON (RN-130); no recomendar productos financieros, inversiones ni créditos (W16); si `budget` es `null`, indicar que no hay presupuesto configurado y no mencionar restante ni exceso; si `overspentAmount` no es `null`, explicar que el presupuesto se superó en ese monto. Un cambio en el archivo crea una versión nueva (`v2`) y pasa por la checklist de revisión (PRINCIPIO-011).

---

**RN-130** | Categoría: IA | Prioridad: MUST | Fuente: PRINCIPIO-004 · Decisión del equipo

Patrón EARS: Ubiquitous

Las instrucciones del modelo DEBEN indicar que el resumen solo puede citar cifras presentes en el JSON de datos, sin calcular, redondear ni estimar cifras nuevas.

✅ **Ejemplo:** El resumen dice «gastaste S/ 495.00 de tus S/ 600.00 de presupuesto (82.50 %)».

❌ **Contraejemplo:** El resumen dice «te quedan unos S/ 100» o «a este ritmo gastarás S/ 900 este mes».

> Se verifica de dos formas: una prueba unitaria comprueba que el archivo de instrucciones contiene la indicación, y las evaluaciones con Ollama real de ADR-003 comprueban, en 10 de 10 escenarios, que cada **cifra** del resumen (definición en ADR-003, sección 3) coincide con el valor absoluto de algún número del JSON. La IA puede incumplir la indicación; por eso RN-043 impide que su texto se use como cifra oficial, y el mismo verificador se aplica en ejecución a cada resumen (RN-166).
>
> **Limitación:** las cifras escritas con palabras («quinientos soles») o los enteros sin «S/», sin «%» y sin decimales («495 soles») no se detectan (ADR-003, sección 3).

---

**RN-131** | Categoría: UX | Prioridad: MUST | Fuente: Decisión del equipo

Patrón EARS: State-driven

MIENTRAS una solicitud de resumen esté en curso, el sistema DEBE mostrar el botón «Generar resumen del mes» deshabilitado con el texto «Generando resumen…».

✅ **Ejemplo:** El usuario pulsa el botón dos veces seguidas; se envía 1 sola solicitud a Ollama.

❌ **Contraejemplo:** Cada pulsación envía una solicitud nueva y el modelo procesa 2 en paralelo.

> El texto «Generando resumen…» se anuncia con `aria-live="polite"`.

> Sin JavaScript el botón no se deshabilita y un segundo envío es posible (limitación aceptada, DEC-90).

---

**RN-132** | Categoría: UX | Prioridad: MUST | Fuente: DEC-12 · DEC-33

Patrón EARS: State-driven

MIENTRAS las funciones de IA estén deshabilitadas, el dashboard DEBE mostrar el botón «Generar resumen del mes» deshabilitado junto al texto MSG-24.

✅ **Ejemplo:** Con `Ai:BaseUrl` vacío, el botón aparece deshabilitado y debajo se lee «El análisis con inteligencia artificial no está configurado en esta instalación.»

❌ **Contraejemplo:** El botón desaparece sin explicación, o está habilitado y siempre devuelve MSG-23.

> «Deshabilitada» (configuración inválida, RN-086, RN-087) es distinto de «no disponible» (Ollama no responde, RN-038).

---

**RN-133** | Categoría: Seguridad | Prioridad: MUST | Fuente: PRINCIPIO-005, PRINCIPIO-008 · DEC-28

Patrón EARS: Ubiquitous

El registro de un incidente de IA DEBE contener solo estos datos: fecha y hora UTC, tipo de error, código HTTP (si lo hubo), duración en milisegundos, nombre del modelo.

✅ **Ejemplo:** `2026-11-08T23:30:12Z Ollama request failed: type=HttpError status=500 duration=812ms model=<modelo>`.

❌ **Contraejemplo:** El log incluye el prompt, el JSON de datos, cualquier importe, el `UserId` o el correo del usuario.

> Tipos de error: `Connection`, `Timeout`, `HttpError`, `EmptyResponse`, `InvalidFigure` (RN-166). Ejemplo: `2026-11-08T23:30:12Z Ollama request failed: type=InvalidFigure duration=8120ms model=qwen3:4b`. El log no incluye la cifra rechazada.

---

**RN-134** | Categoría: IA | Prioridad: MUST | Fuente: DEC-10 · ADR-002 · Decisión del equipo

Patrón EARS: Unwanted

SI `Ai:TimeoutSeconds` no es un entero entre 5 y 120, ENTONCES el sistema DEBE usar 30 segundos y registrar una advertencia al arrancar.

✅ **Ejemplo:** Con `Ai:TimeoutSeconds = 0`, la aplicación arranca, usa 30 s y el log muestra `Ai:TimeoutSeconds out of range (0); using 30`.

❌ **Contraejemplo:** Con `Ai:TimeoutSeconds = 0`, toda solicitud falla de inmediato, o con `-1` la espera es infinita.

---

**RN-165** | Categoría: UX | Prioridad: MUST | Fuente: Alcance M6 · DEC-54

Patrón EARS: Event-driven

CUANDO el sistema termina de procesar una solicitud de resumen enviada desde el dashboard, el sistema DEBE redirigir al dashboard y mostrar el resultado una sola vez.

✅ **Ejemplo:** El usuario pulsa «Generar resumen del mes»: `POST /insights/monthly-summary` responde `303` hacia `/dashboard`, que muestra el resumen. Al recargar la página no se llama a Ollama y el resumen ya no aparece. Si la solicitud falló, se muestra MSG-23 de la misma forma.

❌ **Contraejemplo:** Recargar la página genera otro resumen, o el resumen se guarda en la base de datos y se muestra en cada visita.

> El resultado viaja en TempData (proveedor de cookie, cifrado con Data Protection) y no se guarda en la base de datos. Aplica a las vistas MVC; la API devuelve el resumen directamente (`200`, ADR-004).

---

**RN-166** | Categoría: IA | Prioridad: MUST | Fuente: PRINCIPIO-004 · DEC-55

Patrón EARS: Unwanted

SI el texto generado por el modelo contiene alguna cifra no válida según ADR-003, sección 3, ENTONCES el sistema DEBE tratar la solicitud como fallida.

✅ **Ejemplo:** El JSON tiene `remainingBudget: 105.00` y el modelo escribe «Te quedan unos S/ 100»: el usuario ve MSG-23 y el log registra `type=InvalidFigure` (RN-133).

❌ **Contraejemplo:** El texto con «S/ 100» se muestra al usuario.

> Se usa el mismo verificador que las evaluaciones de ADR-003 (`FigureVerifier`, en `Insights.Domain`). No se reintenta (RN-083).

---

## 7. Should have (S1, S2, S3, S4, S5)

**RN-044** | Categoría: Cálculo | Prioridad: SHOULD | Fuente: Alcance S1

Patrón EARS: Optional

DONDE esté implementado el presupuesto por categoría, el sistema DEBE calcular el porcentaje consumido de una categoría como (gastos del mes en esa categoría ÷ presupuesto del mes de esa categoría) × 100, redondeado a 2 decimales.

✅ **Ejemplo:** Alimentación 2026-11: presupuesto S/ 300.00, gastos S/ 270.00, porcentaje `90.00`.

❌ **Contraejemplo:** El porcentaje de Alimentación se calcula sobre el presupuesto mensual total: 270.00 ÷ 600.00 = `45.00`.

---

**RN-045** | Categoría: Datos | Prioridad: SHOULD | Fuente: PRINCIPIO-007 · Alcance S3

Patrón EARS: Optional

DONDE esté implementado el historial con filtros, el sistema DEBE devolver únicamente movimientos del usuario autenticado.

✅ **Ejemplo:** `demo1` filtra por la categoría «Transporte» entre 2026-11-01 y 2026-11-08; solo recibe sus propios gastos de Transporte.

❌ **Contraejemplo:** El mismo filtro devuelve también los gastos de Transporte de `demo2`.

---

**RN-046** | Categoría: Cálculo | Prioridad: SHOULD | Fuente: Alcance S4

Patrón EARS: Optional

DONDE esté implementada la meta de ahorro, el sistema DEBE calcular el avance como (suma de montos marcados como ahorro ÷ monto de la meta) × 100, redondeado a 2 decimales.

✅ **Ejemplo:** Meta «Laptop» de S/ 2,500.00; montos marcados como ahorro: S/ 300.00 y S/ 200.00. Avance: `20.00`.

❌ **Contraejemplo:** El avance se calcula con el saldo acumulado (S/ 155.00 ÷ S/ 2,500.00 = `6.20`).

> Cómo se marca un monto como ahorro se define al especificar S4.

---

**RN-047** | Categoría: IA | Prioridad: SHOULD | Fuente: Alcance S5

Patrón EARS: Optional

DONDE esté implementada la explicación al alcanzar el umbral, el sistema DEBE mostrar el botón «Explicar mi consumo» junto al aviso de umbral del 80 %.

✅ **Ejemplo:** Con porcentaje `82.50`, el aviso del 80 % incluye el botón «Explicar mi consumo».

❌ **Contraejemplo:** Con porcentaje `82.50`, el sistema genera la explicación automáticamente sin que el usuario pulse ningún botón (contradice RN-037).

---

**RN-089** | Categoría: Registro | Prioridad: SHOULD | Fuente: Alcance S2 · DEC-14

Patrón EARS: Optional

DONDE esté implementado el periodo según la entrega de dinero, el sistema DEBE restringir la frecuencia de entrega a uno de estos valores: cada 7 días, cada 15 días, mensual.

✅ **Ejemplo:** El usuario elige «Cada 15 días».

❌ **Contraejemplo:** La API acepta la frecuencia «cada 10 días», o la UI la rotula «Quincenal» (término ambiguo: en Perú suele significar los días 15 y fin de mes).

---

**RN-090** | Categoría: Cálculo | Prioridad: SHOULD | Fuente: DEC-14

Patrón EARS: Optional

DONDE esté implementado el periodo según la entrega de dinero, MIENTRAS la frecuencia sea de N = 7 o N = 15 días, el sistema DEBE iniciar el siguiente periodo N días calendario después del inicio del periodo actual.

✅ **Ejemplo:** Periodo iniciado el 2026-11-01: con N = 7, el siguiente empieza el 2026-11-08; con N = 15, el 2026-11-16.

❌ **Contraejemplo:** Con N = 15, el siguiente periodo empieza el 2026-11-15, porque se toma como día del mes.

> Inicio del primer periodo: RN-135.

---

**RN-091** | Categoría: Cálculo | Prioridad: SHOULD | Fuente: DEC-14

Patrón EARS: Optional

DONDE esté implementado el periodo según la entrega de dinero, MIENTRAS la frecuencia sea mensual, el sistema DEBE fijar la entrega de cada mes en el día de entrega seleccionado por el usuario.

✅ **Ejemplo:** Con el día 15, las entregas caen el 2026-11-15, el 2026-12-15 y el 2027-01-15.

❌ **Contraejemplo:** La siguiente entrega se calcula sumando 30 días (2026-12-15 → 2027-01-14).

---

**RN-092** | Categoría: Cálculo | Prioridad: SHOULD | Fuente: DEC-14

Patrón EARS: Unwanted

DONDE esté implementado el periodo según la entrega de dinero, SI el día de entrega seleccionado no existe en un mes, ENTONCES el sistema DEBE usar el último día de ese mes.

✅ **Ejemplo:** Con el día 31, la entrega cae el 2027-02-28, el 2028-02-29 y el 2027-04-30. Con el día 30, cae el 2027-02-28.

❌ **Contraejemplo:** Con el día 31, la entrega de febrero de 2027 se desborda al 2027-03-03.

---

**RN-093** | Categoría: Cálculo | Prioridad: SHOULD | Fuente: DEC-14

Patrón EARS: Event-driven

DONDE esté implementado el periodo según la entrega de dinero, CUANDO un mes contiene el día de entrega seleccionado, el sistema DEBE usar ese día aunque el mes anterior haya usado su último día válido.

✅ **Ejemplo:** Con el día 31, después de la entrega del 2027-02-28 la siguiente cae el 2027-03-31.

❌ **Contraejemplo:** La siguiente entrega cae el 2027-03-28, porque se arrastra el día ajustado de febrero.

> Las fechas se evalúan en `America/Lima`.

---

**RN-135** | Categoría: Registro | Prioridad: SHOULD | Fuente: Alcance S2 · DEC-31

Patrón EARS: Optional

DONDE esté implementado el periodo según la entrega de dinero, el sistema DEBE tomar como inicio del primer periodo la fecha de la última entrega que indica el usuario al configurar la frecuencia.

✅ **Ejemplo:** El 2026-11-08 el usuario elige «Cada 7 días» e indica que su última entrega fue el 2026-11-06: el periodo actual va del 2026-11-06 al 2026-11-12 y el siguiente empieza el 2026-11-13.

❌ **Contraejemplo:** El primer periodo empieza en la fecha de configuración (2026-11-08) o en la fecha del primer ingreso registrado.

> Validación de la fecha indicada: RN-143 y RN-161.

---

**RN-143** | Categoría: Registro | Prioridad: SHOULD | Fuente: Alcance S2 · DEC-31

Patrón EARS: Unwanted

DONDE esté implementado el periodo según la entrega de dinero, SI la fecha de última entrega indicada es posterior a hoy, ENTONCES el sistema DEBE rechazarla.

✅ **Ejemplo:** El 2026-11-08 se acepta el 2026-11-08.

❌ **Contraejemplo:** El 2026-11-08 se acepta el 2026-11-09.

---

**RN-161** | Categoría: Registro | Prioridad: SHOULD | Fuente: Alcance S2 · DEC-31

Patrón EARS: Unwanted

DONDE esté implementado el periodo según la entrega de dinero, SI la fecha de última entrega indicada es anterior al inicio del periodo que la contendría (hoy − N días para N = 7 o 15; el mismo día del mes anterior para la frecuencia mensual, con el ajuste de RN-092), ENTONCES el sistema DEBE rechazarla.

✅ **Ejemplo:** El 2026-11-08, con «Cada 7 días», se aceptan las fechas del 2026-11-01 al 2026-11-08.

❌ **Contraejemplo:** Con «Cada 7 días» se acepta el 2026-10-31 (ya hubo una entrega posterior).

---

**RN-136** | Categoría: Cálculo | Prioridad: SHOULD | Fuente: Alcance S2 · DEC-21

Patrón EARS: Optional

DONDE esté implementado el periodo según la entrega de dinero, el sistema DEBE calcular el saldo del periodo como total de ingresos del periodo − total de gastos del periodo, donde el periodo va desde su fecha de inicio (incluida) hasta el inicio del siguiente (excluido).

✅ **Ejemplo:** Periodo 2026-11-06 a 2026-11-12: ingresos S/ 150.00 y gastos S/ 92.50. Saldo del periodo: S/ 57.50.

❌ **Contraejemplo:** El saldo del periodo incluye un gasto del 2026-11-13.

> Los presupuestos, el porcentaje consumido y los avisos siguen calculándose por mes calendario. El saldo del periodo se muestra **además** de los saldos de RN-074 y RN-075.

---

## 8. Trazabilidad funcionalidad → reglas

| Funcionalidad | Reglas |
|---|---|
| M1 — Cuenta y acceso | RN-001 a RN-006, RN-048 a RN-050, RN-094 a RN-102, RN-140, RN-144 a RN-156, RN-171, RN-173, RN-174, RN-177 |
| M2 — Registro de gastos | RN-010 a RN-021, RN-051 a RN-057, RN-104, RN-105, RN-108, RN-109, RN-111 a RN-117, RN-162, RN-168, RN-172, RN-178 |
| M3 — Registro de ingresos | RN-010 a RN-013, RN-019 a RN-021, RN-051 a RN-061, RN-104, RN-106, RN-107, RN-109 a RN-117, RN-160, RN-168, RN-172, RN-178 |
| M4 — Presupuesto mensual | RN-022 a RN-027, RN-053, RN-055, RN-062 a RN-067, RN-104, RN-118, RN-119, RN-139, RN-167, RN-172, RN-175, RN-176 |
| M5 — Dashboard | RN-028 a RN-033, RN-068 a RN-081, RN-120 a RN-127, RN-141, RN-142, RN-162, RN-163 |
| M6 — Resumen con IA | RN-034 a RN-043, RN-082 a RN-088, RN-128 a RN-134, RN-165, RN-166 |
| Transversal | RN-007 a RN-009, RN-103, RN-137, RN-138, RN-157 a RN-159, RN-164, RN-169, RN-170 |
| S1 / S3 / S4 / S5 | RN-044 / RN-045 / RN-046 / RN-047 |
| S2 — Periodo según entrega | RN-089 a RN-093, RN-135, RN-136, RN-143, RN-161 |

---

## 9. Registro de decisiones

Decisiones de Piero Carbajal que fundamentan las reglas. Todas están vigentes. Las reglas las citan en su campo «Fuente» o en sus notas.

| ID | Decisión | Dónde se aplica |
|---|---|---|
| DEC-01 | Monto de un movimiento entre S/ 0.01 y S/ 100,000.00; el tipo, no el signo, distingue gasto e ingreso | RN-051, RN-052, RN-056 |
| DEC-02 | Máximo 2 decimales; el monto ingresado nunca se redondea; el error se explica con MSG-09 | RN-053 a RN-055 |
| DEC-03 | Redondeo `AwayFromZero` en una constante compartida, solo para resultados de cálculo | RN-025, RN-120 |
| DEC-04 | La fecha de un movimiento no puede ser futura y debe estar en la ventana de registro | RN-057, RN-109 |
| DEC-05 | El dashboard muestra saldo del mes (indicador principal) y saldo acumulado, con leyenda de saldo calculado | RN-032, RN-074 a RN-077, RN-123, RN-126 |
| DEC-06 | El porcentaje consumido no tiene tope; el restante se muestra con mínimo S/ 0.00 y el exceso aparte | RN-067, RN-071 a RN-073, RN-124 |
| DEC-07 | Presupuesto entre S/ 0.01 y S/ 100,000.00; sin presupuesto: «No configurado», sin porcentaje ni avisos | RN-062, RN-063, RN-066, RN-068 a RN-070, RN-121, RN-122 |
| DEC-08 | Un solo presupuesto por usuario y mes; se edita, no se duplica | RN-064, RN-065 |
| DEC-09 | Bloqueo de 5 minutos tras 5 intentos fallidos; un único mensaje para todo fallo de inicio de sesión | RN-048 a RN-050 |
| DEC-10 | Solicitud a Ollama con 30 s de espera, sin reintentos y con registro de incidentes | RN-038, RN-082 a RN-084, RN-133, RN-134 |
| DEC-11 | Formato de importes «S/ 1,250.50» y de porcentajes «82.50 %» | RN-081, RN-124 |
| DEC-12 | La IA es opcional: la aplicación arranca sin Ollama y, con configuración inválida, la IA queda deshabilitada (MSG-24) | RN-085 a RN-088, RN-132 |
| DEC-13 | Orígenes de ingreso cerrados; descripción opcional solo con «Otros» | RN-058 a RN-061, RN-110 |
| DEC-14 | Frecuencias de entrega: cada 7 días, cada 15 días o mensual (Should S2) | RN-089 a RN-093, RN-135, RN-136 |
| DEC-15 | Últimos 5 movimientos en el dashboard, ordenados por fecha, fecha de creación e `Id` | RN-078 a RN-080, RN-117 |
| DEC-16 | Lista cerrada de hosts de Ollama: `localhost`, `127.0.0.1`, `::1` (ADR-002) | RN-087 |
| DEC-17 | Unicidad del presupuesto por `UNIQUE (UserId, Year, Month)`; el presupuesto por categoría (S1) irá en otra tabla | RN-064 |
| DEC-18 | La descripción solo se muestra unida al origen «Otros»; no hay columna «Descripción» | RN-080 |
| DEC-19 | Lista de movimientos paginada de 20, sin filtros (los filtros son S3) | RN-117 |
| DEC-20 | No existe «movimiento anulado»: la eliminación es física y pide confirmación | RN-075, RN-114, RN-115 |
| DEC-21 | El periodo de entrega solo define el «saldo del periodo»; presupuestos y avisos siguen siendo mensuales | RN-136 |
| DEC-22 | El ejemplo de meta de ahorro usa «S/ 2,500.00» en todos los documentos | Brief 4.3, `02` S4 |
| DEC-23 | Los orígenes de ingreso de M3 son los de DEC-13 | `02` M3 |
| DEC-24 | Ventana de registro de 12 meses calendario, mes en curso incluido | RN-109 |
| DEC-25 | Porcentajes siempre con 2 decimales y espacio antes de «%» | RN-124 |
| DEC-26 | Saldo negativo como «-S/ 45.00» más el texto «Tus gastos superan tus ingresos registrados» | RN-123, RN-142 |
| DEC-27 | El saldo del mes va antes que el acumulado, con un encabezado de mayor nivel | RN-126 |
| DEC-28 | El log de IA solo registra fecha UTC, tipo, código HTTP, duración y modelo | RN-133 |
| DEC-29 | Un único mensaje (MSG-02) también para la cuenta bloqueada | RN-050 |
| DEC-30 | 2 decimales sin redondeo y el mismo formato de entrada también para el presupuesto | RN-053, RN-055, RN-104 |
| DEC-31 | El primer periodo de entrega empieza en la fecha de la última entrega indicada | RN-135 |
| DEC-32 | La descripción del origen se rechaza si supera 100 caracteres y se descarta al cambiar el origen | RN-061, RN-110 |
| DEC-33 | Con la IA deshabilitada: botón deshabilitado + MSG-24 | RN-132 |
| DEC-34 | Las validaciones de movimientos y presupuestos aplican al crear y al editar | §0.6 |
| DEC-35 | Fechas `dd/MM/yyyy` en la UI; ISO en los campos `date` y en la API | RN-125 |
| DEC-36 | Se aprueban las herramientas de desarrollo, prueba y CI de ADR-001 | ADR-001 |
| DEC-37 | Se acepta coma o punto como separador decimal, sin separador de miles | RN-104, MSG-06 |
| DEC-38 | MVC y API sobre los mismos servicios de aplicación | ADR-004 |
| DEC-39 | Equipo de desarrollo: PC x64 con 8 GB de RAM, sin GPU | ADR-003, ADR-004 |
| DEC-40 | SQL Server Express y Ollama nativos en el equipo; Docker solo en el CI | ADR-004, RN-087 |
| DEC-41 | Modelo `qwen3:4b` sin modo de razonamiento | ADR-003 |
| DEC-42 | Las contraseñas de las cuentas demo están versionadas y no son secretos | ADR-005 §4, RN-138 |
| DEC-43 | El payload de IA incluye restante y exceso del presupuesto | RN-034, RN-129 |
| DEC-44 | Evaluaciones de IA locales antes del PR: 10 escenarios, 10/10 | ADR-003, RN-130 |
| DEC-45 | Solo los logs de IA tienen restricción de datos financieros (RN-133); el resto sigue RN-006 | RN-006, RN-133 |
| DEC-46 | Repositorio sin licencia (uso académico); paquetes con licencia gratuita compatible | ADR-005 §5 |
| DEC-47 | Ningún agente genera ni propone migraciones destructivas | ADR-005 §3 |
| DEC-48 | Cerrar sesión elimina la cookie del navegador; una copia capturada antes sigue válida hasta expirar (limitación aceptada) | RN-097 |
| DEC-49 | La sesión expira entre 30 y 60 minutos sin actividad | RN-098 |
| DEC-50 | Un agente puede proponer en un PR una dependencia que ya figura en ADR-001 | ADR-005 §7, PRINCIPIO-010 |
| DEC-51 | Registro con «Confirmar contraseña»; al crear la cuenta, el usuario entra al dashboard | RN-148, RN-152, RN-153 |
| DEC-52 | Correo de cualquier dominio con formato `texto@texto.texto`, sin tildes, ñ ni símbolos distintos de punto, guion y guion bajo | RN-150 |
| DEC-53 | El seed solo toca `demo1` a `demo6` y no se bloquea por otros usuarios | RN-158, ADR-005 §4 |
| DEC-54 | Resumen de IA con Post/Redirect/Get y TempData: se muestra una sola vez | RN-165 |
| DEC-55 | El verificador de cifras se aplica en ejecución; si falla, MSG-23 | RN-166, ADR-003 §3 |
| DEC-56 | «Crear presupuesto» enviado después del cambio de mes se rechaza con MSG-22 | RN-167 |
| DEC-57 | Contra el doble envío basta deshabilitar el botón; sin JavaScript queda como limitación aceptada | RN-168, RN-131 |
| DEC-58 | En la API, la pertenencia del recurso (`404`) se evalúa antes que la validación del cuerpo (`400`) | ADR-004 §3.1 |
| DEC-59 | Si la sesión expira mientras se llena un formulario, lo escrito se pierde (limitación aceptada) | RN-098 |
| DEC-60 | `GET /api/antiforgery/token` no exige sesión; el cliente pide el token después de iniciar sesión | RN-101 |
| DEC-61 | Validación solo en el servidor: formularios con `novalidate` | RN-116, ADR-006 |
| DEC-62 | Estructura del código: un proyecto con módulos `Domain`, `Application`, `Infrastructure`, `Web` y el módulo `Shared` | ADR-004 §2 |
| DEC-63 | Rutas de cuenta: `GET/POST /account/register`, `GET/POST /account/login`, `POST /account/logout` | RN-145 |
| DEC-64 | Página `404` con MSG-45, igual para rutas inexistentes y recursos ajenos | RN-169 |
| DEC-65 | Tras editar o eliminar, `303` a `/transactions?page=n`; «Cancelar» de la edición vuelve a la misma página | RN-117, RN-178 |
| DEC-66 | En los formularios MVC, una descripción con un origen distinto de «Otros» se descarta sin error | RN-110 |
| DEC-67 | Confirmación de eliminación con `<details>` sin JavaScript y `<dialog>` con JavaScript | RN-114 |
| DEC-68 | El aviso de datos ficticios del registro es MSG-05; no se agrega otro | RN-099 |
| DEC-69 | Tras guardar o eliminar se muestra una confirmación breve, una sola vez (MSG-49 a MSG-53) | RN-172 |
| DEC-70 | Tras iniciar sesión, un `returnUrl` hacia una ruta que solo admite `POST` lleva a la página del formulario | RN-174 |
| DEC-71 | El rechazo antiforgery en MVC muestra una página `400` con MSG-46; cerrar sesión sin sesión vigente lleva a P-002 sin error | RN-170, RN-177 |
| DEC-72 | La página de inicio de sesión conserva el `returnUrl` en cada nueva presentación del formulario | RN-155 |
| DEC-73 | Un usuario con sesión que abre el registro o el inicio de sesión va al dashboard | RN-173 |
| DEC-74 | Los formularios de registro, inicio de sesión y presupuesto también deshabilitan el botón al enviar | RN-168 |
| DEC-75 | Tras un error, la casilla de mayoría de edad conserva el valor enviado | RN-148 |
| DEC-76 | Orden de los errores: Contraseña MSG-34 → MSG-03; Confirmación MSG-36 → MSG-35, y MSG-35 solo si la contraseña no está vacía | RN-152 |
| DEC-77 | El campo «Correo» usa `type="text"` con `inputmode="email"`, para que el navegador no transforme el valor | RN-150 |
| DEC-78 | Tras un error, el foco va al primer campo con error; el resumen de errores no recibe foco | RN-116 |
| DEC-79 | Esquema único de identificadores del Spec Pack (`04` §3) | Todos |
| DEC-80 | Los errores que no son de un campo (MSG-02, MSG-21, MSG-22) van en un bloque propio con `role="alert"` | RN-050, RN-064, RN-167 |
| DEC-81 | Al llegar al inicio de sesión por expiración se muestra MSG-48 | RN-171 |
| DEC-82 | En el inicio de sesión, el correo se compara sin espacios exteriores | RN-154 |
| DEC-83 | Las vistas contienen como texto los literales que verifican las HU (dos puntos en los rótulos, « · » en las filas) | RN-080, RN-068, RN-069, RN-071 |
| DEC-84 | Tras MSG-22 por cambio de mes, «Crear presupuesto» se vuelve a mostrar con el mes nuevo; «Editar presupuesto» enlaza a «Crear presupuesto» | RN-167 |
| DEC-85 | Redirecciones cruzadas entre «Crear presupuesto» y «Editar presupuesto» según exista el presupuesto del mes | RN-175, RN-176 |
| DEC-86 | Si tras eliminar la página quedó vacía y no es la primera, se muestra la página anterior | RN-178 |
| DEC-87 | La ayuda de la contraseña es el mensaje de catálogo MSG-47 | RN-094 |
| DEC-88 | La respuesta `401` de la API lleva el `ProblemDetails` estándar con cuerpo | RN-102 |
| DEC-89 | La verificación de `Trait` del CI cubre las reglas de categoría Cálculo y prioridad MUST | PRINCIPIO-011, ADR-001 §3 |
| DEC-90 | Sin JavaScript, el botón del resumen de IA puede enviarse dos veces (misma limitación que DEC-57) | RN-131 |

---

## 10. Catálogo de mensajes de la UI

Textos exactos (es-PE). Toda validación de un campo muestra su mensaje junto al campo afectado, vinculado con `aria-describedby` (RN-116, PRINCIPIO-012).

| ID | Texto | Reglas |
|---|---|---|
| MSG-01 | Debes declarar que eres mayor de 18 años para crear tu cuenta. | RN-002 |
| MSG-02 | No se pudo iniciar sesión. Verifica tu correo y contraseña, o inténtalo de nuevo en unos minutos. | RN-050 |
| MSG-03 | La contraseña debe tener al menos 8 caracteres, con una mayúscula, una minúscula, un número y un símbolo. | RN-094 |
| MSG-04 | Ya existe una cuenta con este correo. | RN-095 |
| MSG-05 | Versión académica de demostración. No ingreses datos personales ni financieros reales. | RN-099 |
| MSG-06 | Ingresa el monto solo con números y, si tiene decimales, un punto o una coma (por ejemplo, 12.50 o 12,50). | RN-104 |
| MSG-07 | El monto debe ser mayor que cero. | RN-051 |
| MSG-08 | El monto no puede superar S/ 100,000.00. | RN-052, RN-104 |
| MSG-09 | El monto debe tener como máximo dos decimales. | RN-053, RN-054, RN-104 |
| MSG-10 | La fecha no puede ser posterior a hoy. | RN-057 |
| MSG-11 | Ingresa el monto. | RN-104, RN-105, RN-106 |
| MSG-12 | Selecciona una categoría. | RN-105 |
| MSG-13 | Selecciona un medio de pago. | RN-105 |
| MSG-14 | Selecciona el origen del ingreso. | RN-058 |
| MSG-15 | La descripción puede tener como máximo 100 caracteres. | RN-061 |
| MSG-16 | Ingresa la fecha. | RN-105, RN-106 |
| MSG-17 | La fecha no puede ser anterior al {fecha mínima en dd/MM/yyyy}. | RN-109 |
| MSG-18 | ¿Eliminar este movimiento? Esta acción no se puede deshacer. | RN-114 |
| MSG-19 | El presupuesto debe ser mayor que cero. | RN-062 |
| MSG-20 | El presupuesto no puede superar S/ 100,000.00. | RN-063, RN-104 |
| MSG-21 | Ya tienes un presupuesto para este mes. Puedes cambiarlo desde «Editar presupuesto». | RN-064 |
| MSG-22 | Solo puedes crear o editar el presupuesto del mes en curso. | RN-118, RN-167 |
| MSG-23 | El análisis con inteligencia artificial no está disponible en este momento. Inténtalo nuevamente. | RN-038, RN-082, RN-166 |
| MSG-24 | El análisis con inteligencia artificial no está configurado en esta instalación. | RN-132 |
| MSG-25 | Ingresa el monto del presupuesto. | RN-104, RN-139 |
| MSG-26 | Este campo no corresponde a este tipo de movimiento. | ADR-004 §3 (solo API), RN-110 |
| MSG-27 | El valor no está permitido. | ADR-004 §3 (solo API) |
| MSG-28 | La fecha debe tener el formato aaaa-mm-dd. | ADR-004 §3 (solo API) |
| MSG-29 | La página debe ser un número entero mayor o igual que 1. | ADR-004 §3 (solo API) |
| MSG-30 | El tipo de un movimiento no se puede cambiar. | RN-112 (solo API) |
| MSG-31 | No hay más movimientos. | RN-117 |
| MSG-32 | Ingresa tu correo. | RN-149, RN-154 |
| MSG-33 | Ingresa un correo válido, por ejemplo, nombre@dominio.com. Usa solo letras sin tildes ni ñ, números, punto, guion o guion bajo. | RN-150 |
| MSG-34 | Ingresa tu contraseña. | RN-149, RN-154 |
| MSG-35 | Las contraseñas no coinciden. | RN-152 |
| MSG-36 | Confirma tu contraseña. | RN-149 |
| MSG-37 | El correo puede tener como máximo 256 caracteres. | RN-151 |
| MSG-38 | La solicitud no tiene un formato JSON válido. | ADR-004 §3.1 (solo API) |
| MSG-39 | El monto debe ser un número. | ADR-004 §3.1 (solo API) |
| MSG-40 | Ocurrió un error inesperado. Inténtalo nuevamente en unos minutos. | RN-164 |
| MSG-41 | Aún no tienes movimientos registrados. | RN-162 |
| MSG-42 | Aún no tienes gastos registrados este mes. | RN-163 |
| MSG-43 | Selecciona el tipo de movimiento. | ADR-004 §3.1 (solo API) |
| MSG-44 | Revisa los datos ingresados. | RN-116, ADR-004 §3.1 (API: `title` de `ValidationProblemDetails`; MVC: título del resumen de errores, `09` CMP-04) |
| MSG-45 | La página que buscas no existe. | RN-169 |
| MSG-46 | Tu formulario caducó. Vuelve a la página e inténtalo de nuevo. | RN-170 |
| MSG-47 | Mínimo 8 caracteres, con mayúscula, minúscula, número y símbolo. | RN-094 (ayuda) |
| MSG-48 | Tu sesión terminó. Inicia sesión para continuar. | RN-171 |
| MSG-49 | Gasto guardado. | RN-172 |
| MSG-50 | Ingreso guardado. | RN-172 |
| MSG-51 | Cambios guardados. | RN-172 |
| MSG-52 | Movimiento eliminado. | RN-172, RN-178 |
| MSG-53 | Presupuesto guardado. | RN-172 |

> Textos fijos que no son mensajes de error y que ya figuran en sus reglas: «Saldo calculado según tus registros» (RN-032), «Configura un presupuesto para controlar tus gastos» (RN-070), «Presupuesto excedido» (RN-072), avisos de umbral (RN-028, RN-127), «Tus gastos superan tus ingresos registrados» (RN-123), «Explicación generada automáticamente» (RN-040), «No constituye asesoría financiera» (RN-041), «Generando resumen…» (RN-131), «Volver al inicio» (RN-164, RN-169, RN-170).

---

---

## 11. Escenarios de las cuentas demo (seed)

Datos fijos que crea el seed (RN-138). Las pruebas E2E y las sesiones de usabilidad los usan como datos conocidos. **M** = mes en curso (movimientos con fecha del día 1); **M−1** = mes anterior (movimientos con fecha del día 15). Los movimientos se crean en el orden de la tabla, con fechas de creación crecientes (RN-159).

Todas las cuentas (`demo1` a `demo6`) usan la contraseña `Demo#2026x` (ADR-005, sección 4).

### 11.1 Resumen

| Cuenta | Escenario | Presupuesto de M | Porcentaje consumido | Saldo del mes | Saldo acumulado | Movimientos | Reglas que ilustra |
|---|---|---|---|---|---|---|---|
| `demo1@expenseflow.test` | Presupuesto en curso, aviso del 80 % | S/ 600.00 | 82.50 % | S/ 105.00 | S/ 155.00 | 12 | RN-018, RN-028, RN-032, RN-076 a RN-078 |
| `demo2@expenseflow.test` | Presupuesto excedido | S/ 500.00 | 110.00 % | S/ 50.00 | S/ 50.00 | 3 | RN-029, RN-067, RN-071 a RN-073, RN-127 |
| `demo3@expenseflow.test` | Sin presupuesto y saldo negativo | No configurado | No disponible | -S/ 150.00 | -S/ 150.00 | 3 | RN-068 a RN-070, RN-078, RN-121 a RN-123, RN-142 |
| `demo4@expenseflow.test` | Cuenta vacía | No configurado | No disponible | S/ 0.00 | S/ 0.00 | 0 | RN-108, RN-162, RN-163, primer uso |
| `demo5@expenseflow.test` | Límite de redondeo: 99.995 % | S/ 600.00 | 100.00 % | S/ 0.03 | S/ 0.03 | 3 | RN-120, RN-029, sin «Presupuesto excedido» |
| `demo6@expenseflow.test` | Paginación: 25 movimientos | No configurado | No disponible | S/ 880.00 | S/ 880.00 | 25 | RN-117 (3 páginas: 20, 5 y vacía), RN-079 |

### 11.2 Movimientos

| Cuenta | Mes | Tipo | Monto | Categoría u origen | Medio de pago |
|---|---|---|---|---|---|
| `demo1` | M−1 | Ingreso | S/ 600.00 | Apoyo familiar | — |
| `demo1` | M−1 | Gasto | S/ 300.00 | Alimentación | Efectivo |
| `demo1` | M−1 | Gasto | S/ 100.00 | Transporte | Yape |
| `demo1` | M−1 | Gasto | S/ 80.00 | Estudios | Plin |
| `demo1` | M−1 | Gasto | S/ 50.00 | Entretenimiento | Efectivo |
| `demo1` | M−1 | Gasto | S/ 20.00 | Salud | Tarjeta |
| `demo1` | M | Ingreso | S/ 600.00 | Apoyo familiar | — |
| `demo1` | M | Gasto | S/ 150.00 | Alimentación | Efectivo |
| `demo1` | M | Gasto | S/ 120.00 | Alimentación | Plin |
| `demo1` | M | Gasto | S/ 60.00 | Transporte | Efectivo |
| `demo1` | M | Gasto | S/ 60.00 | Transporte | Efectivo |
| `demo1` | M | Gasto | S/ 105.00 | Estudios | Yape |
| `demo2` | M | Ingreso | S/ 600.00 | Trabajo | — |
| `demo2` | M | Gasto | S/ 300.00 | Alimentación | Efectivo |
| `demo2` | M | Gasto | S/ 250.00 | Entretenimiento | Tarjeta |
| `demo3` | M | Ingreso | S/ 300.00 | Prácticas preprofesionales | — |
| `demo3` | M | Gasto | S/ 200.00 | Alimentación | Yape |
| `demo3` | M | Gasto | S/ 250.00 | Estudios | Efectivo |
| `demo5` | M | Ingreso | S/ 600.00 | Apoyo familiar | — |
| `demo5` | M | Gasto | S/ 400.00 | Alimentación | Efectivo |
| `demo5` | M | Gasto | S/ 199.97 | Transporte | Yape |
| `demo6` | M | Ingreso | S/ 1,000.00 | Trabajo independiente | — |
| `demo6` | M | Gasto × 24 | S/ 5.00 c/u (24 gastos iguales, uno tras otro) | Transporte | Efectivo |

### 11.3 Cifras esperadas del mes en curso

| Cuenta | Ingresos | Gastos | Gasto por categoría | Restante del presupuesto | Exceso | Medio de pago preseleccionado (RN-018, RN-108) |
|---|---|---|---|---|---|---|
| `demo1` | S/ 600.00 | S/ 495.00 | Alimentación S/ 270.00 · Transporte S/ 120.00 · Estudios S/ 105.00 | S/ 105.00 | — | Yape |
| `demo2` | S/ 600.00 | S/ 550.00 | Alimentación S/ 300.00 · Entretenimiento S/ 250.00 | S/ 0.00 | S/ 50.00 | Tarjeta |
| `demo3` | S/ 300.00 | S/ 450.00 | Estudios S/ 250.00 · Alimentación S/ 200.00 | — | — | Efectivo |
| `demo4` | S/ 0.00 | S/ 0.00 | MSG-42 (RN-163) | — | — | Efectivo |
| `demo5` | S/ 600.00 | S/ 599.97 | Alimentación S/ 400.00 · Transporte S/ 199.97 | S/ 0.03 | — | Yape |
| `demo6` | S/ 1,000.00 | S/ 120.00 | Transporte S/ 120.00 | — | — | Efectivo |

> Saldo acumulado de `demo1`: (S/ 600.00 − S/ 550.00) + (S/ 600.00 − S/ 495.00) = S/ 155.00. El gasto por categoría se ordena de mayor a menor importe (RN-033). Cualquier cambio en estas tablas es un cambio de datos de prueba y debe actualizar las pruebas E2E que los usan.

### 11.4 Uso de los escenarios

| Situación | Regla |
|---|---|
| Antes de cada sesión de usabilidad | Se ejecuta el seed. Así cada participante empieza con los mismos datos, aunque el anterior haya registrado o borrado movimientos. |
| Antes de cada corrida de pruebas E2E | El CI aplica las migraciones y ejecuta el seed sobre una base vacía en el entorno `Testing` (ADR-004, sección 4.2). |
| Mes de ejecución | El seed se ejecuta **el mismo mes calendario (hora de Lima)** en que se usan los datos. Si una sesión o corrida cruza el cambio de mes, se vuelve a ejecutar. |
| La base contiene usuarios que no son cuentas demo (por ejemplo, cuentas creadas por participantes o por pruebas E2E) | El seed se ejecuta igual y no los lee ni los modifica (RN-158). |
| Pruebas E2E que **solo leen** (dashboard, avisos, lista, paginación) | Usan las cuentas demo de esta sección y verifican las cifras de 11.3. |
| Pruebas E2E que **modifican datos** (registrar, corregir o eliminar movimientos; crear o editar presupuestos) | Usan una cuenta propia creada en la propia prueba (`e2e-<GUID>@expenseflow.test`), nunca una cuenta demo. Así no alteran los escenarios que otras pruebas leen. |
| Pruebas de aislamiento (RN-008) | Usan dos cuentas demo (por ejemplo, `demo1` intenta leer un movimiento de `demo2`) y solo leen o reciben `404`. |

> Esta sección es normativa (§0.6): una prueba E2E que modifique una cuenta demo incumple esta regla aunque pase.