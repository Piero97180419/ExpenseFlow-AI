# 06 — Historias de usuario — ExpenseFlow AI

| Campo | Valor |
|---|---|
| Documento | 06-user-stories.md |
| Fase | Planificación (BMAD — Agente PO) · Validación (BMAD — Agente QA Funcional) |
| Versión | v1.0 |
| Fecha | 2026-11-10 |
| Estado | **Aprobado** |
| Aprobado por | Piero Carbajal |
| Fecha de aprobación | 2026-11-10 |
| PR de aprobación | PR que fusiona el Spec Pack v1.0 en `main` (ADR-005 §1) |
| Insumos | `00-constitucion.md` a `05-preguntas-abiertas.md` v1.0, ADR-001 a ADR-007 v1.0, `checklist-revision.md` v1.0 |

> **Convenciones**
> - Formato: «Como [rol], quiero [acción], para [beneficio]», validación INVEST y criterios DADO QUE / CUANDO / ENTONCES.
> - Cada HU Must se traza a reglas `RN-NNN` de `03-reglas-de-negocio.md`. El ejemplo y el contraejemplo de cada regla son criterios de aceptación adicionales (`03` §0).
> - Los mensajes se citan por su ID del catálogo (`03` §10). El texto exacto está allí.
> - Las cifras usan los escenarios fijos del seed (`03` §11). Las pruebas que **modifican** datos, incluidos los contadores de inicio de sesión, usan una cuenta `e2e-<GUID>@expenseflow.test` creada en la propia prueba, nunca una cuenta demo (`03` §11.4).
> - Los `Id` numéricos de los criterios (movimiento 42, presupuesto 7, X) son ilustrativos: la prueba usa el `Id` real del registro del seed o del que crea la propia prueba (`03` §0.5).
> - «Hoy» = 2026-11-08, hora de Lima, fijada con `FakeTimeProvider` (RN-137), salvo que el criterio indique otro instante.
> - Importes con formato RN-081 (`S/ 1,250.50`); porcentajes con formato RN-124 (`82.50 %`).
> - Tipo de escenario: **(P)** positivo · **(N)** negativo · **(B)** borde.
> - Nivel de prueba sugerido: **U** unitaria · **I** integración (`WebApplicationFactory`) · **E2E** Playwright.
> - No se escriben HU para W1–W20 ni C1–C4 (fuera de alcance o Could).
> - Las pantallas de cada HU están en `09-ui-spec.md` §1.1 y sus tareas en `08-backlog.md`.

---

## Resumen

| Épica | HU | Título | Prioridad | Funcionalidad | Flujo (`02` §3.3) |
|---|---|---|---|---|---|
| E1 — Cuenta y acceso | HU-001 | Crear mi cuenta declarando mayoría de edad | Must | M1 | F-01 |
| | HU-002 | Iniciar y cerrar sesión de forma segura (contenedora) | Must | M1 | F-01 |
| | HU-002A | Iniciar sesión con protección ante intentos fallidos | Must | M1 | F-01 |
| | HU-002B | Cerrar sesión y mantener la sesión protegida | Must | M1 | F-01 |
| E2 — Registro de movimientos | HU-003 | Registrar un gasto en pocos segundos | Must | M2 | F-02 |
| | HU-004 | Registrar el dinero que recibo | Must | M3 | F-04 |
| | HU-005 | Consultar la lista paginada de mis movimientos | Must | M2, M3 | F-03 |
| | HU-006 | Corregir o eliminar un movimiento | Must | M2, M3 | F-03 |
| E3 — Presupuesto | HU-007 | Crear y editar el presupuesto del mes | Must | M4 | F-05 |
| E4 — Dashboard | HU-008 | Ver mis saldos y en qué gasto | Must | M5 | F-06 |
| | HU-009 | Recibir avisos al 80 % y al 100 % del presupuesto | Must | M5 | F-06 |
| E5 — IA explicativa | HU-010 | Pedir un resumen del mes en lenguaje sencillo (contenedora) | Must | M6 | F-07 |
| | HU-010A | Solicitar y ver el resumen del mes | Must | M6 | F-07 |
| | HU-010B | Proteger el resumen ante fallos y cifras inventadas | Must | M6 | F-07 |
| Transversal | HU-011 | Tener la garantía de que nadie más ve mis datos | Must | M1–M6 | Todos |
| | HU-013 | Ver un mensaje comprensible ante un error inesperado | Must | M1–M6 | Todos |
| | HU-014 | Restablecer las cuentas demo antes de una sesión de prueba | Must (habilitador) | — | — |
| E6 — Post-MVP | HU-012 | Ver cuánto me queda hasta la próxima entrega | Should | S2 | — |
| | HU-015 | Repartir el presupuesto entre categorías (propuesta) | Should | S1 | — |
| | HU-016 | Filtrar mi historial de movimientos (propuesta) | Should | S3 | — |
| | HU-017 | Seguir el avance de una meta de ahorro (propuesta) | Should | S4 | — |
| | HU-018 | Pedir una explicación al alcanzar el 80 % (propuesta) | Should | S5 | — |

> **Orden de construcción sugerido:** HU-014 (seed) → HU-001 → HU-002A → HU-002B → HU-011 → HU-003 → HU-004 → HU-005 → HU-006 → HU-007 → HU-008 → HU-009 → HU-013 → HU-010A → HU-010B. HU-012 y HU-015 a HU-018 solo se inician con todos los Must terminados y el CI en verde (`02` §0).

---

## E1 — Cuenta y acceso (M1)

### HU-001: Crear mi cuenta declarando mayoría de edad
**Prioridad:** Must

Como **estudiante universitario mayor de 18 años**, quiero **crear una cuenta con mi correo y una contraseña**, para **empezar a registrar mi dinero sin pasos adicionales**.

**Validación INVEST:** I ✔ · N ✔ · V ✔ · E ✔ · S ◐ · T ✔ — Con 18 criterios queda en el límite de «Small», pero todos ejercitan el mismo formulario y el mismo servicio. No se divide.

**Criterios de aceptación:**
- **(P)** DADO QUE soy un visitante en el formulario de registro, CUANDO lo abro, ENTONCES veo exactamente 4 campos de entrada (Correo, Contraseña, Confirmar contraseña y la casilla «Declaro ser mayor de 18 años»), la ayuda MSG-47 bajo «Contraseña», el texto MSG-05 sobre el botón «Crear cuenta» y el enlace «¿Ya tienes cuenta? Inicia sesión»; el campo Correo es `type="text"` con `inputmode="email"`. · E2E
- **(P)** DADO QUE ingreso un correo nuevo `e2e-<GUID>@expenseflow.test`, la contraseña `Demo#2026x` en ambos campos y marco la casilla, CUANDO pulso «Crear cuenta», ENTONCES se crea la cuenta con `LockoutEnabled = 1`, inicio sesión automáticamente y llego a `/dashboard`, que muestra «Presupuesto: No configurado» y MSG-41. · E2E
- **(N)** DADO QUE completo correctamente todos los campos pero dejo la casilla desmarcada, CUANDO pulso «Crear cuenta», ENTONCES no se crea la cuenta, debajo de la casilla aparece MSG-01 vinculado con `aria-describedby` y el foco va al primer campo con error. · E2E
- **(N)** DADO QUE ingreso la contraseña `Ab#1x` (5 caracteres), CUANDO envío el formulario, ENTONCES no se crea la cuenta y se muestra MSG-03. · I
- **(N)** DADO QUE ingreso la contraseña `demo20261` (sin mayúscula ni símbolo) o `DEMO#2026X` (sin minúscula), CUANDO envío el formulario, ENTONCES no se crea la cuenta y se muestra MSG-03. · I
- **(N)** DADO QUE existe `demo1@expenseflow.test`, CUANDO intento registrar `DEMO1@expenseflow.test`, ENTONCES no se crea la cuenta y se muestra MSG-04. · I
- **(N)** DADO QUE ingreso `Demo#2026x` como contraseña y `Demo#2026X` como confirmación, CUANDO envío el formulario, ENTONCES se muestra MSG-35 bajo «Confirmar contraseña» y no se crea la cuenta. · I
- **(N)** DADO QUE ingreso el correo `josé@correo.pe` o uno de 257 caracteres, CUANDO envío el formulario, ENTONCES se muestra MSG-33 o MSG-37, respectivamente, y no se crea la cuenta. · I
- **(N)** DADO QUE ingreso `ñandu@correo.pe`, `ana+1@gmail.com`, `ana@gmail`, `ana @gmail.com` o `ana@mi_dominio.com`, CUANDO envío el formulario, ENTONCES en cada caso se muestra MSG-33 y no se crea la cuenta. · U (validador) + I (un caso)
- **(N)** DADO QUE envío el formulario con el correo, la contraseña y la confirmación vacíos, CUANDO pulso «Crear cuenta», ENTONCES aparecen a la vez MSG-32, MSG-34 y MSG-36, cada uno bajo su campo, en español (nunca el texto por defecto en inglés), y no se crea la cuenta. · E2E
- **(B)** DADO QUE ingreso la contraseña `Demo#26x` (exactamente 8 caracteres) y un correo válido de exactamente 256 caracteres, CUANDO envío el formulario, ENTONCES la cuenta se crea. · I
- **(B)** DADO QUE ingreso un correo de 257 caracteres que además contiene «ñ», CUANDO envío el formulario, ENTONCES solo se muestra MSG-37, porque la longitud se valida antes que el formato (RN-149). · U
- **(B)** DADO QUE ingreso `  e2e-<GUID>@expenseflow.test  ` (con espacios al inicio y al final), CUANDO envío el formulario, ENTONCES la cuenta se crea, porque los espacios exteriores no cuentan (RN-150, RN-151). · I
- **(N)** DADO QUE se creó una cuenta con `Demo#2026x`, CUANDO se lee su fila en `AspNetUsers`, ENTONCES `PasswordHash` no contiene la contraseña en claro y `PasswordHasher.VerifyHashedPassword` devuelve `Success`. · I
- **(B)** DADO QUE marqué la casilla y escribí una contraseña de 5 caracteres, CUANDO envío el formulario, ENTONCES se muestra MSG-03, la casilla sigue marcada, el correo se conserva y los dos campos de contraseña aparecen vacíos. · E2E
- **(N)** DADO QUE dejo la contraseña vacía y escribo `abc` en «Confirmar contraseña», CUANDO envío el formulario, ENTONCES se muestra MSG-34 bajo «Contraseña» y ningún error bajo «Confirmar contraseña» (MSG-35 solo aparece si la contraseña no está vacía). · U
- **(B)** DADO QUE la respuesta del servidor está retrasada, CUANDO pulso «Crear cuenta» dos veces seguidas, ENTONCES el botón queda deshabilitado tras la primera pulsación y se crea 1 sola cuenta. · E2E
- **(N)** DADO QUE soy `demo1` con sesión iniciada, CUANDO pido `GET /account/register` o envío `POST /account/register` con un token válido, ENTONCES llego a `/dashboard` y no se crea ninguna cuenta. · I

**Reglas de negocio relacionadas:** RN-001, RN-002, RN-003, RN-004, RN-094, RN-095, RN-096, RN-099, RN-116, RN-148, RN-149, RN-150, RN-151, RN-152, RN-153, RN-168, RN-173

**Notas:**
- Orden de validación del correo: vacío (MSG-32) → longitud (MSG-37) → formato (MSG-33) → duplicado (MSG-04) (RN-149). Orden de la contraseña y la confirmación: nota de RN-152 (DEC-76).
- Tras un error, la casilla conserva el valor enviado y las contraseñas se vacían (DEC-75). El botón se deshabilita al enviar (DEC-74).
- No hay confirmación de correo (W18) ni recuperación de contraseña (W17).
- `nuevo1@expenseflow.test` es válido como ejemplo en pruebas manuales; en pruebas automáticas se usa siempre un correo único `e2e-<GUID>` (`03` §11.4).

---

### HU-002: Iniciar y cerrar sesión de forma segura
**Prioridad:** Must (contenedora)

Como **usuario registrado**, quiero **iniciar y cerrar sesión**, para **acceder a mis datos y que nadie más los vea desde mi dispositivo**.

> HU-002 es una contenedora: se divide en HU-002A y HU-002B para cumplir la «S» de INVEST. No tiene criterios propios y se da por terminada cuando HU-002A y HU-002B lo están.

**Reglas de negocio relacionadas:** las de HU-002A y HU-002B.

#### HU-002A: Iniciar sesión con protección ante intentos fallidos
**Prioridad:** Must

Como **usuario registrado**, quiero **iniciar sesión con mi correo y mi contraseña**, para **acceder a mis datos sin que nadie pueda adivinar mi contraseña probando muchas veces**.

**Validación INVEST:** I ✔ · N ✔ · V ✔ · E ✔ · S ✔ · T ✔

**Criterios de aceptación:**
- **(P)** DADO QUE soy `demo1@expenseflow.test` con contraseña `Demo#2026x` y accedí con `returnUrl=/transactions`, CUANDO inicio sesión, ENTONCES llego a la lista de movimientos; con `returnUrl=https://sitio-externo.example` llego a `/dashboard`. · E2E
- **(N)** DADO QUE accedí con `returnUrl=//sitio-externo.example` (URL relativa al protocolo), CUANDO inicio sesión, ENTONCES llego a `/dashboard`. · I
- **(N)** DADO QUE ingreso una contraseña incorrecta para una cuenta existente o el correo `noexiste@expenseflow.test`, CUANDO pulso «Iniciar sesión», ENTONCES en ambos casos se muestra el mismo mensaje MSG-02, en un bloque con `role="alert"` sobre el formulario, y el correo escrito se conserva. · I + E2E (un caso)
- **(N)** DADO QUE una cuenta `e2e-<GUID>@expenseflow.test` creada en la prueba falló 5 veces seguidas y el quinto fallo fue a las 10:01:00, CUANDO intenta con la contraseña correcta a las 10:03:00, ENTONCES se rechaza con MSG-02 (el mismo texto que una contraseña incorrecta); y CUANDO lo intenta a las 10:06:01, ENTONCES inicia sesión. · I con `FakeTimeProvider`
- **(B)** DADO QUE una cuenta de prueba falló 4 veces seguidas, CUANDO intenta con la contraseña correcta, ENTONCES inicia sesión, `AccessFailedCount` vuelve a 0 y otros 4 fallos posteriores no la bloquean. · I
- **(N)** DADO QUE envío el formulario con la contraseña vacía, CUANDO pulso «Iniciar sesión», ENTONCES se muestra MSG-34 y `AccessFailedCount` no cambia; con el correo vacío se muestra MSG-32 y tampoco cambia. · I
- **(N)** DADO QUE abro el formulario de inicio de sesión, CUANDO se carga, ENTONCES tiene solo los campos Correo y Contraseña y el botón «Iniciar sesión», sin la casilla «Recordarme». · E2E
- **(N)** DADO QUE soy un visitante, CUANDO pido `GET /Identity/Account/Manage` o `GET /Identity/Account/ForgotPassword`, ENTONCES la respuesta es `404`. · I
- **(N)** DADO QUE un inicio de sesión falla con la contraseña `Demo#2026x-mal`, CUANDO se revisa el log, ENTONCES ninguna línea contiene esa contraseña. · I
- **(B)** DADO QUE accedí con `returnUrl=/transactions` y la primera vez ingreso una contraseña incorrecta, CUANDO la corrijo y vuelvo a enviar, ENTONCES llego a la lista de movimientos: el `returnUrl` se conserva tras el error. · E2E
- **(B)** DADO QUE la sesión de `demo1` expiró y pulsó «Generar resumen del mes» (`returnUrl=/insights/monthly-summary`), CUANDO inicia sesión, ENTONCES llega a `/dashboard`; con `returnUrl=/transactions/5/delete` llega a `/transactions`. · I
- **(B)** DADO QUE escribo `  demo1@expenseflow.test  ` (con espacios exteriores) y la contraseña correcta, CUANDO pulso «Iniciar sesión», ENTONCES inicio sesión. · I
- **(N)** DADO QUE soy `demo1` con sesión iniciada, CUANDO pido `GET /account/login`, ENTONCES llego a `/dashboard` sin ver el formulario. · I

**Reglas de negocio relacionadas:** RN-006, RN-048, RN-049, RN-050, RN-116, RN-144, RN-145, RN-146, RN-154, RN-155, RN-168, RN-173, RN-174

**Notas:**
- `PasswordSignInAsync(..., isPersistent: false, lockoutOnFailure: true)` (RN-144, RN-147).
- Las pruebas de bloqueo usan `FakeTimeProvider`; Identity debe usar el `TimeProvider` registrado (RN-137).
- Las pruebas que fallan contraseñas nunca usan una cuenta demo (`03` §11.4).
- El `returnUrl` viaja en un campo oculto y se conserva en cada nueva presentación del formulario (DEC-72). El correo se compara sin espacios exteriores (DEC-82).

#### HU-002B: Cerrar sesión y mantener la sesión protegida
**Prioridad:** Must

Como **usuario**, quiero **cerrar sesión y que mi sesión caduque si la dejo abierta**, para **que nadie vea mis datos desde mi dispositivo ni en la red**.

**Validación INVEST:** I ◐ · N ✔ · V ✔ · E ✔ · S ✔ · T ✔ — Necesita HU-002A para tener una sesión iniciada.

**Criterios de aceptación:**
- **(P)** DADO QUE tengo la sesión iniciada, CUANDO pulso «Cerrar sesión», ENTONCES el navegador ya no tiene la cookie: `GET /dashboard` redirige al inicio de sesión, `GET /api/dashboard` responde `401` y el botón «Atrás» no muestra saldos (`Cache-Control: no-store`). · E2E
- **(B)** DADO QUE inicié sesión a las 10:00 y la última solicitud que renovó la cookie fue a las 10:40, CUANDO envío una solicitud a las 11:41, ENTONCES se rechaza como no autenticada (MVC redirige al inicio de sesión; API responde `401`). · I con `FakeTimeProvider`
- **(B)** DADO QUE inicié sesión a las 10:00 y solo hice una solicitud a las 10:20 (que no renueva la cookie), CUANDO envío una solicitud a las 11:01, ENTONCES se rechaza como no autenticada. · I con `FakeTimeProvider`
- **(B)** DADO QUE abrí «Nuevo gasto» y mi sesión expiró antes de enviarlo, CUANDO pulso «Guardar gasto», ENTONCES se me redirige al inicio de sesión con el aviso MSG-48, no se crea ningún gasto y, tras iniciar sesión, vuelvo al formulario «Nuevo gasto» vacío. · I con `FakeTimeProvider` (limitación aceptada, DEC-59)
- **(N)** DADO QUE inicio sesión, CUANDO se inspecciona la respuesta, ENTONCES la cabecera `Set-Cookie` de autenticación no tiene `Expires` ni `Max-Age`. · I
- **(N)** DADO QUE la aplicación corre en `Testing`, CUANDO pido `http://localhost:<puerto>/dashboard`, ENTONCES recibo una redirección a HTTPS, la respuesta HTTPS incluye `Strict-Transport-Security`, y un cliente que solo ofrece TLS 1.1 no completa el handshake. · I
- **(N)** DADO QUE pulsé «Cerrar sesión», CUANDO se muestra la página de inicio de sesión, ENTONCES no aparece MSG-48; y al abrir `/account/login` por primera vez tampoco aparece. · E2E
- **(B)** DADO QUE mi sesión expiró, CUANDO pulso «Cerrar sesión», ENTONCES llego al inicio de sesión sin MSG-46 ni MSG-48 y sin `returnUrl=/account/logout`. · I con `FakeTimeProvider`

**Reglas de negocio relacionadas:** RN-005, RN-097, RN-098, RN-100, RN-102, RN-140, RN-147, RN-155, RN-156, RN-171, RN-177

**Notas:**
- MSG-48 se muestra solo cuando el sistema detecta la expiración (cookie de marca `ef_session` sin cookie de autenticación, `reason=expired`; `07` §4.9).
- Limitación aceptada (DEC-48): una copia de la cookie capturada antes del cierre sigue válida hasta expirar.
- **Limitación aceptada (DEC-59, nota de RN-098):** si la sesión expira mientras el usuario llena un formulario, lo escrito se pierde. Se acepta porque el formulario tiene 4 campos y se rehace en segundos (U2: ≤ 15 s).

---

## E2 — Registro de movimientos (M2, M3)

### HU-003: Registrar un gasto en pocos segundos
**Prioridad:** Must

Como **estudiante que paga en efectivo, Yape o Plin**, quiero **anotar un gasto con monto, categoría, medio de pago y fecha**, para **no olvidar los gastos pequeños y saber en qué se va mi dinero**.

**Validación INVEST:** I ✔ · N ✔ · V ✔ · E ✔ · S ✔ · T ✔ — La mayoría de los casos de monto son pruebas unitarias del mismo validador; no aumentan el tamaño de la HU.

**Criterios de aceptación:**
- **(P)** DADO QUE soy `demo1` y son las 23:30 del 2026-11-08 (hora de Lima), CUANDO abro «Nuevo gasto», ENTONCES veo exactamente 4 campos (Monto, Categoría, Medio de pago, Fecha), la fecha propuesta es `2026-11-08` y el medio de pago preseleccionado es «Yape». · E2E
- **(P)** DADO QUE soy `demo4` (sin gastos), CUANDO abro «Nuevo gasto», ENTONCES el medio de pago preseleccionado es «Efectivo». · E2E
- **(P)** DADO QUE soy un usuario de prueba, CUANDO registro el monto `3,50`, la categoría «Transporte», el medio «Efectivo» y la fecha `2026-11-08`, ENTONCES el gasto se guarda con `Amount = 3.50`, llego a `/dashboard` con el mensaje MSG-49 («Gasto guardado.») en un bloque con `role="status"` y el gasto aparece en los últimos movimientos como «08/11/2026 · Transporte · Gasto · S/ 3.50»; al recargar la página, MSG-49 ya no aparece. · E2E
- **(N)** DADO QUE estoy en «Nuevo gasto», CUANDO ingreso `12.345`, `1,250.50`, `0` o `100000.01` como monto, ENTONCES no se guarda nada y se muestra MSG-09, MSG-06, MSG-07 o MSG-08, respectivamente, junto al campo Monto con `aria-describedby`. · U + E2E (un caso)
- **(N)** DADO QUE estoy en «Nuevo gasto», CUANDO ingreso `.50`, `12.`, `12.5.0`, `-5`, `S/ 12`, `1.250,50` o `١٢`, ENTONCES se muestra MSG-06; y CUANDO dejo el monto vacío o solo con espacios, ENTONCES se muestra MSG-11. · U
- **(B)** DADO QUE estoy en «Nuevo gasto», CUANDO ingreso `0.01`, `100000`, `100000.00`, `0000012.50` o ` 12,50 `, ENTONCES el gasto se guarda con 0.01, 100000.00, 100000.00, 12.50 y 12.50, respectivamente. · U + I
- **(B)** DADO QUE estoy en «Nuevo gasto», CUANDO ingreso `1234567` o `0.001`, ENTONCES se muestra MSG-08 (sin convertir el texto) o MSG-09, respectivamente. · U
- **(N)** DADO QUE hoy es 2026-11-08, CUANDO registro un gasto con fecha `2026-11-09` o `2025-11-30`, ENTONCES se rechaza con MSG-10 o con MSG-17 («La fecha no puede ser anterior al 01/12/2025.»), respectivamente. · I
- **(B)** DADO QUE hoy es 2026-11-08, CUANDO registro un gasto con fecha `2025-12-01` (primer día de la ventana), ENTONCES se guarda. · I
- **(N)** DADO QUE dejo la categoría sin seleccionar, CUANDO pulso «Guardar gasto», ENTONCES se muestra MSG-12 y el gasto no se guarda; con los 4 campos vacíos se muestran a la vez MSG-11, MSG-12, MSG-13 y MSG-16. · E2E
- **(B)** DADO QUE la red es lenta (respuesta del servidor retrasada en la prueba), CUANDO pulso «Guardar gasto» dos veces seguidas, ENTONCES el botón queda deshabilitado tras la primera pulsación y se crea 1 solo gasto; si el servidor responde con un error de validación, el botón vuelve a habilitarse. · E2E
- **(B)** DADO QUE el reloj está en 2026-11-30 23:59 (hora de Lima), CUANDO registro un gasto con la fecha propuesta, ENTONCES se guarda con fecha `2026-11-30` y cuenta en el total de 2026-11; y CUANDO el reloj pasa a 2026-12-01 00:00:30, ENTONCES el formulario propone `2026-12-01`. · I con `FakeTimeProvider` (RN-137)
- **(B)** DADO QUE una cuenta de prueba tiene dos gastos con la misma fecha de creación, CUANDO abro «Nuevo gasto», ENTONCES se preselecciona el medio de pago del gasto de mayor `Id`; y si elimino ese gasto, se preselecciona el del siguiente más reciente. · I
- **(N)** DADO QUE uso la API, CUANDO envío `POST /api/transactions` con `"category": "Pets"`, con `"paymentMethod": "BcpTransfer"`, con `"amount": "12.50"` (texto) o con `"amount": 12.345`, ENTONCES la respuesta es `400` con MSG-27, MSG-27, MSG-39 o MSG-09, respectivamente; con `"amount": 12.340` se crea con 12.34. · I
- **(N)** DADO QUE uso la API, CUANDO envío `POST /api/transactions` sin la propiedad `type`, ENTONCES la respuesta es `400` con MSG-43 bajo la clave `type` y no se crea nada. · I
- **(N)** DADO QUE uso la API, CUANDO envío un gasto válido con `"currency": "USD"`, ENTONCES se crea igual, la propiedad se ignora y el movimiento no tiene moneda. · I
- **(P)** DADO QUE uso solo el teclado, CUANDO completo y envío el formulario, ENTONCES puedo terminar el registro y la revisión axe no reporta infracciones WCAG 2.2 AA. · E2E
- **(N)** DADO QUE el código de producción o de pruebas usa `DateTime.Now`, `DateTime.UtcNow`, `DateTime.Today`, `DateTimeOffset.Now` o `DateTimeOffset.UtcNow` fuera del registro de `TimeProvider`, CUANDO corre el CI, ENTONCES la prueba de arquitectura falla. · U (arquitectura, RN-137)

**Reglas de negocio relacionadas:** RN-010, RN-011, RN-012, RN-014, RN-015, RN-016, RN-017, RN-018, RN-019, RN-051, RN-052, RN-053, RN-054, RN-055, RN-056, RN-057, RN-104, RN-105, RN-108, RN-109, RN-116, RN-137, RN-168, RN-172

**Notas:**
- KPI U2: ≥ 4 de 5 participantes registran el tercer gasto en ≤ 15 s.
- Campo Monto: `<input type="text" inputmode="decimal">`; se acepta coma o punto (DEC-37).
- RN-108 es una **[Hipótesis]**: puede cambiar tras las entrevistas (`05` PQ-06).
- El botón se deshabilita mientras se envía (RN-168), como RN-131 en el resumen de IA. Sin JavaScript, el doble envío queda como limitación aceptada (DEC-57).

---

### HU-004: Registrar el dinero que recibo
**Prioridad:** Must

Como **estudiante que recibe dinero de su familia, de prácticas o de trabajos ocasionales**, quiero **registrar cada ingreso con su origen**, para **saber cuánto dinero entró y calcular mi saldo**.

**Validación INVEST:** I ✔ · N ✔ · V ✔ · E ✔ · S ✔ · T ✔ — Reutiliza el validador de monto de HU-003; si HU-003 no está lista, el validador se hace aquí.

**Criterios de aceptación:**
- **(P)** DADO QUE abro «Nuevo ingreso» el 2026-11-08, CUANDO se carga el formulario, ENTONCES veo los campos Monto, Origen y Fecha (con `2026-11-08` propuesto), sin campo de medio de pago. · E2E
- **(P)** DADO QUE soy un usuario de prueba sin movimientos, CUANDO registro S/ 600.00 con origen «Apoyo familiar» y fecha `2026-11-08`, ENTONCES llego a `/dashboard` con MSG-50 («Ingreso guardado.») y el dashboard muestra «Saldo del mes: S/ 600.00». · E2E
- **(P)** DADO QUE elijo el origen «Otros», CUANDO se actualiza el formulario, ENTONCES aparece el campo opcional «Descripción»; si escribo «Venta de libros» y guardo S/ 80.00, la fila se muestra como «Otros: Venta de libros · Ingreso · S/ 80.00». · E2E
- **(P)** DADO QUE una cuenta de prueba tiene presupuesto de S/ 600.00 y gastos de S/ 495.00 en 2026-11, CUANDO registro un ingreso de S/ 100.00, ENTONCES el saldo del mes sube S/ 100.00, pero el porcentaje sigue en «82.50 %» y el restante del presupuesto en S/ 105.00 (los ingresos no cambian el presupuesto, RN-026). · I
- **(B)** DADO QUE escribo una descripción de 101 caracteres, CUANDO guardo, ENTONCES se muestra MSG-15 y el ingreso no se guarda (una de 100 caracteres sí se guarda). · I
- **(B)** DADO QUE elijo «Otros» y dejo la descripción vacía, CUANDO guardo, ENTONCES el ingreso se guarda con `SourceDescription = null`. · I
- **(B)** DADO QUE ingreso los montos `0.01` y `100000.00`, CUANDO guardo, ENTONCES ambos ingresos se guardan; con `100000.01` se muestra MSG-08. · U
- **(N)** DADO QUE no selecciono origen, CUANDO guardo, ENTONCES se muestra MSG-14 y no se guarda nada. · E2E
- **(B)** DADO QUE la red es lenta, CUANDO pulso «Guardar ingreso» dos veces seguidas, ENTONCES el botón queda deshabilitado tras la primera pulsación y se crea 1 solo ingreso. · E2E
- **(N)** DADO QUE dejo el monto o la fecha vacíos, o ingreso la fecha `2026-11-09`, CUANDO guardo, ENTONCES se muestra MSG-11, MSG-16 o MSG-10, respectivamente, y no se guarda nada. · I
- **(N)** DADO QUE uso la API, CUANDO envío un ingreso con `category`, con `"source": "Inheritance"` o con `"source": "Job"` y `sourceDescription`, ENTONCES la respuesta es `400` con MSG-26, MSG-27 o MSG-26, respectivamente. · I
- **(B)** DADO QUE son las 23:30 del 2026-11-08 (hora de Lima), CUANDO abro «Nuevo ingreso», ENTONCES la fecha propuesta es `2026-11-08`, no `2026-11-09`. · I con `FakeTimeProvider`

**Reglas de negocio relacionadas:** RN-026, RN-051, RN-052, RN-053, RN-056, RN-057, RN-058, RN-059, RN-060, RN-061, RN-074, RN-104, RN-106, RN-107, RN-109, RN-116, RN-168, RN-172

**Notas:**
- Orígenes cerrados: `FamilySupport`, `Job`, `Internship`, `Freelance`, `Other` (RN-059).
- La descripción se rechaza si es larga; no se trunca (DEC-32).

---

### HU-005: Consultar la lista paginada de mis movimientos
**Prioridad:** Must

Como **usuario con varios movimientos**, quiero **ver todos mis movimientos en una lista ordenada y paginada**, para **encontrar los que no aparecen entre los 5 últimos del dashboard**.

**Validación INVEST:** I ✔ · N ✔ · V ✔ · E ✔ · S ✔ · T ✔

**Criterios de aceptación:**
- **(P)** DADO QUE soy `demo6` (25 movimientos), CUANDO abro la lista, ENTONCES la página 1 muestra 20 movimientos y la página 2 muestra 5, cada uno con las acciones «Editar» y «Eliminar». · E2E
- **(B)** DADO QUE soy `demo6`, CUANDO pido la página 3, ENTONCES veo MSG-31 y un enlace a la página 1; CUANDO pido la página `0`, `-1` o `abc`, ENTONCES veo la página 1. · E2E
- **(B)** DADO QUE una cuenta de prueba tiene exactamente 20 movimientos, CUANDO pido la página 2, ENTONCES veo MSG-31; con 21 movimientos, la página 2 muestra 1 movimiento. · I
- **(B)** DADO QUE una cuenta de prueba tiene dos movimientos con fecha `2026-11-08`, uno creado a las 09:00 y otro a las 10:15 (con `FakeTimeProvider`), CUANDO abro la lista, ENTONCES el creado a las 10:15 aparece primero; y si dos tienen la misma fecha y la misma fecha de creación, aparece primero el de mayor `Id`. · I
- **(B)** DADO QUE soy `demo4` (sin movimientos), CUANDO abro la lista, ENTONCES la página 1 muestra MSG-41, no MSG-31. · E2E
- **(P)** DADO QUE estoy en el dashboard de `demo1`, CUANDO pulso «Ver todos los movimientos», ENTONCES llego a esta lista. · E2E
- **(N)** DADO QUE uso la API, CUANDO pido `GET /api/transactions?page=0` o `?page=abc`, ENTONCES la respuesta es `400` con MSG-29; CUANDO `demo6` pide `?page=3`, ENTONCES la respuesta es `200` con `items: []`, `pageSize: 20` y `totalCount: 25`. · I
- **(N)** DADO QUE soy `demo1`, CUANDO recorro todas las páginas de mi lista, ENTONCES no aparece ningún movimiento de `demo2`. · I

**Reglas de negocio relacionadas:** RN-007, RN-079, RN-080, RN-117, RN-125, RN-162

**Notas:**
- Sin filtros: los filtros son S3 (HU-016).
- Configuración `TransactionListPageSize = 20`.

---

### HU-006: Corregir o eliminar un movimiento
**Prioridad:** Must

Como **usuario que se equivocó al registrar**, quiero **corregir o eliminar un movimiento desde la lista**, para **que mis saldos y totales reflejen la realidad**.

**Validación INVEST:** I ◐ · N ✔ · V ✔ · E ✔ · S ✔ · T ✔ — Depende de HU-005 para llegar a la fila desde la UI; la API se puede construir y probar antes.

**Criterios de aceptación:**
- **(P)** DADO QUE un usuario de prueba tiene gastos de noviembre de 2026 por S/ 450.00, incluido uno de S/ 30.00 del 2026-11-05, CUANDO cambio ese monto a S/ 25.00, ENTONCES el total de gastos de 2026-11 pasa a S/ 445.00. · I
- **(P)** DADO QUE tengo un gasto de S/ 40.00 con categoría «Otros», CUANDO la cambio a «Estudios», ENTONCES el total de Estudios del mes sube S/ 40.00 y el de Otros baja S/ 40.00. · I
- **(P)** DADO QUE tengo un ingreso de S/ 600.00 del 2026-11-01, CUANDO cambio su monto a S/ 650.00 y su fecha a `2026-11-02`, ENTONCES se guardan ambos cambios y el saldo del mes sube S/ 50.00. · I
- **(B)** DADO QUE tengo un gasto de S/ 30.00 con fecha `2026-10-31`, CUANDO cambio su fecha a `2026-11-01`, ENTONCES los gastos de 2026-10 bajan S/ 30.00 y los de 2026-11 suben S/ 30.00. · I
- **(N)** DADO QUE pulso «Eliminar» sobre un movimiento, CUANDO aparece el diálogo con MSG-18 y pulso «Cancelar», ENTONCES el movimiento sigue existiendo y el foco vuelve al botón que abrió el diálogo. · E2E
- **(P)** DADO QUE confirmo la eliminación de un gasto de S/ 20.00 sobre un total de S/ 450.00, CUANDO termina la operación, ENTONCES `SELECT COUNT(*) FROM Transactions WHERE Id = <id>` devuelve 0, el total pasa a S/ 430.00 y vuelvo a la misma página de la lista con MSG-52 («Movimiento eliminado.»). · I + E2E
- **(P)** DADO QUE tengo un ingreso con origen «Otros» y descripción «Venta de libros», CUANDO cambio el origen a «Trabajo» y guardo, ENTONCES se guarda con `SourceDescription = null`. · I
- **(N)** DADO QUE edito un gasto de S/ 30.00, CUANDO cambio el monto a `12.345`, ENTONCES se muestra MSG-09 y el gasto conserva S/ 30.00. · I
- **(B)** DADO QUE una cuenta de prueba tiene un gasto con fecha `2025-11-15` (creado con el reloj en 2026-10) y hoy es 2026-11-08, CUANDO intento guardar cualquier cambio, ENTONCES se rechaza con MSG-17 y el gasto no cambia; CUANDO lo elimino, ENTONCES se elimina. · I con `FakeTimeProvider`
- **(N)** DADO QUE envío `PUT /api/transactions/{id}` con `"type": "Income"` sobre un gasto propio, CUANDO se procesa, ENTONCES responde `400` con MSG-30 y el gasto no cambia. · I
- **(N)** DADO QUE envío `PUT /api/transactions/{id}` sobre un gasto propio sin `paymentMethod`, CUANDO se procesa, ENTONCES responde `400` con MSG-13, porque `PUT` reemplaza el movimiento completo. · I
- **(N)** DADO QUE eliminé un movimiento en otra pestaña, CUANDO envío otra vez `DELETE /api/transactions/{id}` o intento editarlo, ENTONCES la respuesta es `404`. · I
- **(P)** DADO QUE edito desde la página 2 de la lista un gasto de `demo6`, CUANDO pulso «Guardar cambios», ENTONCES vuelvo a `/transactions?page=2` con MSG-51 («Cambios guardados.»); y CUANDO pulso «Cancelar», ENTONCES vuelvo a `/transactions?page=2` sin mensaje. · E2E
- **(B)** DADO QUE una cuenta de prueba tiene 21 movimientos, CUANDO elimino el único movimiento de la página 2, ENTONCES llego a la página 1 con MSG-52, no a una página 2 vacía con MSG-31. · I
- **(B)** DADO QUE la respuesta del servidor está retrasada, CUANDO pulso «Guardar cambios» dos veces seguidas, ENTONCES el botón queda deshabilitado tras la primera pulsación. · E2E

**Reglas de negocio relacionadas:** RN-008, RN-020, RN-021, RN-053, RN-109, RN-110, RN-111, RN-112, RN-113, RN-114, RN-115, RN-160, RN-168, RN-172, RN-178

**Notas:**
- Al editar se aplican todas las validaciones de creación (DEC-34).
- No existe «movimiento anulado» (DEC-20): el borrado es físico.
- Tras editar o eliminar se vuelve a la página de la lista de origen (DEC-65); la confirmación de eliminación usa `<details>` sin JavaScript y `<dialog>` con JavaScript (DEC-67).

---

## E3 — Presupuesto (M4)

### HU-007: Crear y editar el presupuesto del mes
**Prioridad:** Must

Como **estudiante que acaba de recibir su dinero**, quiero **fijar cuánto puedo gastar como máximo este mes**, para **saber cuánto me queda y cuándo me acerco al límite**.

**Validación INVEST:** I ✔ · N ✔ · V ✔ · E ✔ · S ✔ · T ✔

**Criterios de aceptación:**
- **(P)** DADO QUE soy un usuario de prueba sin presupuesto y hoy es 2026-11-08, CUANDO creo un presupuesto de S/ 600.00, ENTONCES queda asociado al mes `2026-11`, llego a `/dashboard` con MSG-53 («Presupuesto guardado.») y el dashboard muestra «Presupuesto: S/ 600.00». · E2E
- **(P)** DADO QUE tengo S/ 600.00 para 2026-11, CUANDO lo cambio a S/ 650.00 desde «Editar presupuesto», ENTONCES conserva el mismo `Id` y sigue habiendo 1 solo registro para ese mes; la pantalla no tiene botón para eliminarlo. · I + E2E
- **(N)** DADO QUE ya tengo S/ 600.00 para 2026-11, CUANDO intento crear otro de S/ 700.00, ENTONCES se muestra MSG-21 en un bloque con `role="alert"`, se conserva el de S/ 600.00 y, en la API, `POST /api/budgets` responde `409`. · I
- **(N)** DADO QUE estoy en «Crear presupuesto», CUANDO envío el monto vacío, `0`, `100000.01` o `600,555`, ENTONCES se muestra MSG-25, MSG-19, MSG-20 o MSG-09, respectivamente, y no se guarda nada. · U + I
- **(N)** DADO QUE tengo S/ 650.00 para 2026-11, CUANDO lo edito a `0`, ENTONCES se muestra MSG-19 y se conserva S/ 650.00. · I
- **(B)** DADO QUE estoy en «Crear presupuesto», CUANDO ingreso `0.01`, `100000.00` o `600,5`, ENTONCES se guarda con 0.01, 100000.00 o 600.50, respectivamente. · U + I
- **(B)** DADO QUE abrí «Crear presupuesto» el 2026-11-30 a las 23:59, CUANDO lo envío el 2026-12-01 a las 00:00:30 (hora de Lima), ENTONCES se muestra MSG-22, no existe presupuesto para 2026-12 y el formulario se vuelve a mostrar con el mes 2026-12 y el monto escrito. · I con `FakeTimeProvider`
- **(B)** DADO QUE tenía S/ 600.00 para 2026-11, CUANDO el reloj pasa a 2026-12-01, ENTONCES el dashboard muestra «Presupuesto: No configurado»: el presupuesto no se arrastra al mes siguiente. · I con `FakeTimeProvider`
- **(B)** DADO QUE envío dos `POST /api/budgets` simultáneos sin presupuesto previo, CUANDO terminan, ENTONCES uno responde `201`, el otro `409` con MSG-21, y existe 1 solo registro. · I
- **(N)** DADO QUE una cuenta de prueba creó un presupuesto con el reloj en 2026-10-20 (`Id` = X) y el reloj pasó a 2026-11-08, CUANDO envío `PUT /api/budgets/X`, ENTONCES responde `400` con MSG-22 y el presupuesto no cambia. · I con `FakeTimeProvider`
- **(N)** DADO QUE uso la API, CUANDO envío `POST /api/budgets` con `{ "amount": 600, "month": "2026-12" }`, ENTONCES se crea el presupuesto de 2026-11 (la propiedad `month` se ignora). · I
- **(N)** DADO QUE tengo un presupuesto, CUANDO envío `DELETE /api/budgets/{id}`, ENTONCES la respuesta es `405` y el presupuesto sigue existiendo. · I
- **(B)** DADO QUE no tengo presupuesto para el mes en curso, CUANDO pido `GET /api/budgets/current`, ENTONCES la respuesta es `204`, nunca `404`. · I
- **(B)** DADO QUE `demo1` tiene S/ 600.00 para 2026-11, CUANDO abre `/budgets/create`, ENTONCES llega a `/budgets/edit` con S/ 600.00 precargado. · I
- **(B)** DADO QUE `demo3` no tiene presupuesto, CUANDO abre `/budgets/edit`, ENTONCES llega a `/budgets/create`. · I
- **(B)** DADO QUE la respuesta del servidor está retrasada, CUANDO pulso «Guardar presupuesto» dos veces seguidas, ENTONCES el botón queda deshabilitado tras la primera pulsación y existe 1 solo presupuesto. · E2E

**Reglas de negocio relacionadas:** RN-022, RN-053, RN-055, RN-062, RN-063, RN-064, RN-065, RN-104, RN-118, RN-119, RN-139, RN-167, RN-168, RN-172, RN-175, RN-176

**Notas:**
- Restricción `UNIQUE (UserId, Year, Month)` en `Budgets` (RN-064).
- KPI U3: ≥ 80 % de participantes configura el presupuesto sin asistencia.
- El formulario lleva el campo oculto `month` (`yyyy-MM`), que no cuenta como campo de entrada (RN-167). Tras MSG-22 el formulario se presenta con el mes nuevo y conserva el monto (DEC-84).
- `DELETE /api/budgets/{id}` no existe: la respuesta `405` la da el enrutamiento (RN-119, ADR-004 §3.2).

---

## E4 — Dashboard (M5)

### HU-008: Ver mis saldos y en qué gasto
**Prioridad:** Must

Como **estudiante**, quiero **ver en una sola pantalla mi saldo, mi presupuesto y mis gastos por categoría**, para **saber cuánto me queda hasta la próxima entrega de dinero**.

**Validación INVEST:** I ✔ · N ✔ · V ✔ · E ✔ · S ✔ · T ✔ — Los casos de solo lectura usan el seed y no dependen de los formularios.

**Criterios de aceptación:**
- **(P)** DADO QUE soy `demo1`, CUANDO abro el dashboard, ENTONCES veo «Saldo del mes: S/ 105.00» (`<h2>`) antes que «Saldo acumulado: S/ 155.00» (`<h3>`), con la leyenda «Saldo calculado según tus registros» junto a cada uno, en móvil (360 px) y en escritorio. · E2E
- **(P)** DADO QUE soy `demo1`, CUANDO abro el dashboard, ENTONCES veo «Restante del presupuesto: S/ 105.00», «82.50 %» de consumo y el gasto por categoría en este orden: Alimentación S/ 270.00, Transporte S/ 120.00, Estudios S/ 105.00, sin categorías con S/ 0.00. · E2E
- **(P)** DADO QUE soy `demo1` (12 movimientos), CUANDO abro el dashboard, ENTONCES veo exactamente 5 movimientos recientes con fecha `dd/MM/yyyy`, categoría u origen, la palabra «Gasto» o «Ingreso» y el monto, más el enlace «Ver todos los movimientos». · E2E
- **(N)** DADO QUE soy `demo3` (sin presupuesto, gastos S/ 450.00, ingresos S/ 300.00), CUANDO abro el dashboard, ENTONCES veo «Presupuesto: No configurado», «Porcentaje consumido: No disponible», el mensaje «Configura un presupuesto para controlar tus gastos» con enlace a «Crear presupuesto», «Saldo del mes: -S/ 150.00» y «Saldo acumulado: -S/ 150.00», cada uno con el texto «Tus gastos superan tus ingresos registrados», y no veo restante, exceso ni «Presupuesto excedido». · E2E
- **(B)** DADO QUE soy `demo4`, CUANDO abro el dashboard, ENTONCES veo saldos de S/ 0.00 sin el texto de saldo negativo, «Presupuesto: No configurado», MSG-41 en lugar de los últimos movimientos y MSG-42 en lugar del gasto por categoría. · E2E
- **(B)** DADO QUE una cuenta de prueba solo registró un ingreso de S/ 200.00 en el mes, CUANDO abro el dashboard, ENTONCES veo «Saldo del mes: S/ 200.00» y MSG-42 en el gasto por categoría. · I
- **(B)** DADO QUE una cuenta de prueba tiene en el mes Transporte S/ 50.00 y Alimentación S/ 50.00, CUANDO abro el dashboard, ENTONCES Alimentación aparece antes que Transporte (orden de RN-014 en el empate). · I
- **(B)** DADO QUE una cuenta de prueba tiene un gasto de S/ 100.00 con fecha `2026-10-31` y otro de S/ 200.00 con fecha `2026-11-01`, CUANDO abro el dashboard el 2026-11-08, ENTONCES el total de gastos del mes es S/ 200.00. · I
- **(B)** DADO QUE soy `demo6`, CUANDO abro el dashboard, ENTONCES veo «Saldo del mes: S/ 880.00»; y `MoneyFormat.Format(1250.5m)` devuelve exactamente «S/ 1,250.50» con espacio U+0020, nunca U+00A0. · E2E + U (RN-013, RN-081)
- **(N)** DADO QUE soy `demo1` y abro el dashboard con JavaScript deshabilitado, CUANDO se carga la página, ENTONCES veo las mismas cifras: ninguna la calcula el navegador (RN-027). · E2E
- **(N)** DADO QUE abro el dashboard en vista móvil, CUANDO corre la revisión axe, ENTONCES no hay infracciones WCAG 2.2 AA y el saldo negativo de `demo3` no se comunica solo con color. · E2E

**Reglas de negocio relacionadas:** RN-013, RN-023, RN-025, RN-026, RN-027, RN-032, RN-033, RN-066, RN-068, RN-069, RN-070, RN-074, RN-075, RN-076, RN-077, RN-078, RN-079, RN-080, RN-081, RN-122, RN-123, RN-124, RN-125, RN-126, RN-142, RN-162, RN-163

**Notas:**
- Todas las cifras las calcula el backend (`ExpenseFlow.Modules.*.Application`); la vista solo las muestra (RN-027).
- Sin librería de gráficos: barras de progreso de Bootstrap y tablas (C4 queda como Could).
- Los rótulos y las filas contienen como texto los literales que verifican los criterios («Saldo del mes:», « · ») (DEC-83).
- KPI T1 (100 % de casos de cálculo) y T4 (prueba desde Android en entorno `Demo`).

---

### HU-009: Recibir avisos al 80 % y al 100 % del presupuesto
**Prioridad:** Must

Como **estudiante con presupuesto limitado**, quiero **ver un aviso claro cuando me acerco o supero mi límite**, para **corregir mis gastos antes de quedarme sin dinero**.

**Validación INVEST:** I ✔ · N ✔ · V ✔ · E ✔ · S ✔ · T ✔

**Criterios de aceptación:**
- **(P)** DADO QUE soy `demo1` (82.50 %) y el cliente de IA está caído (en el CI, el cliente simulado devuelve error de conexión), CUANDO abro el dashboard, ENTONCES veo el texto «Alcanzaste el 80 % de tu presupuesto mensual» con `role="status"`. · E2E
- **(P)** DADO QUE soy `demo2` (presupuesto S/ 500.00, gastos S/ 550.00), CUANDO abro el dashboard, ENTONCES veo «110.00 %», el aviso «Alcanzaste el 100 % de tu presupuesto mensual», «Estado: Presupuesto excedido», «Restante del presupuesto: S/ 0.00» y «Exceso: S/ 50.00». · E2E
- **(N)** DADO QUE tengo un presupuesto de S/ 600.00 y gastos de S/ 479.94 (79.99 %), CUANDO abro el dashboard, ENTONCES no veo ningún aviso de umbral. · U
- **(N)** DADO QUE soy `demo3` (sin presupuesto), CUANDO abro el dashboard, ENTONCES no veo ningún aviso de umbral. · E2E
- **(B)** DADO QUE tengo un presupuesto de S/ 600.00 y gastos de S/ 480.00 (80.00 %), CUANDO abro el dashboard, ENTONCES veo el aviso del 80 %. · U
- **(B)** DADO QUE tengo un presupuesto de S/ 600.00 y gastos de S/ 599.94 (99.99 %), CUANDO abro el dashboard, ENTONCES veo el aviso del 80 % y no el del 100 %. · U
- **(B)** DADO QUE soy `demo5` (presupuesto S/ 600.00, gastos S/ 599.97), CUANDO abro el dashboard, ENTONCES veo «100.00 %», el aviso del 100 % y «Restante del presupuesto: S/ 0.03», y no veo «Presupuesto excedido». · E2E
- **(B)** DADO QUE tengo un presupuesto de S/ 600.00 y gastos de S/ 600.00, CUANDO abro el dashboard, ENTONCES veo «100.00 %», el aviso del 100 % y «Restante del presupuesto: S/ 0.00», y no veo «Presupuesto excedido» ni «Exceso». · U + I
- **(B)** DADO QUE soy `demo2`, CUANDO un lector de pantalla lee la barra de progreso, ENTONCES la barra está llena con `aria-valuenow="100"`, `aria-valuemax="100"` y `aria-valuetext="110.00 %"`. · E2E
- **(B)** DADO QUE soy `demo1`, CUANDO se inspecciona la barra de progreso, ENTONCES tiene `aria-valuenow="82.50"` y `aria-valuetext="82.50 %"` y no se desborda del contenedor. · E2E

**Reglas de negocio relacionadas:** RN-024, RN-028, RN-029, RN-030, RN-031, RN-067, RN-071, RN-072, RN-073, RN-120, RN-121, RN-127, RN-141

**Notas:**
- Los umbrales se comparan con el porcentaje ya redondeado (`AwayFromZero`); «Presupuesto excedido» se decide comparando importes (RN-120).
- Los casos 79.99, 80.00, 99.99, 100.00 y 99.995 son pruebas unitarias con `[Trait("Rule", "RN-028")]`, `[Trait("Rule", "RN-029")]` y `[Trait("Rule", "RN-120")]` (checklist §1).

---

## E5 — IA explicativa (M6)

### HU-010: Pedir un resumen del mes en lenguaje sencillo
**Prioridad:** Must (contenedora)

Como **estudiante al que le cuesta interpretar sus números**, quiero **pedir una explicación en español sencillo de cómo va mi mes**, para **entender mi situación sin analizar tablas**.

> HU-010 es una contenedora: se divide en HU-010A y HU-010B para cumplir la «S» de INVEST. No tiene criterios propios y se da por terminada cuando HU-010A y HU-010B lo están.

**Reglas de negocio relacionadas:** las de HU-010A y HU-010B.

#### HU-010A: Solicitar y ver el resumen del mes
**Prioridad:** Must

Como **estudiante**, quiero **pedir el resumen de mi mes con un botón y leerlo en el dashboard**, para **entender mi situación en lenguaje sencillo**.

**Validación INVEST:** I ✔ · N ✔ · V ✔ · E ✔ · S ✔ · T ✔

**Criterios de aceptación:**
- **(P)** DADO QUE soy `demo1` y el cliente de IA (simulado) devuelve un texto que solo cita S/ 495.00, S/ 600.00 y 82.50 %, CUANDO pulso «Generar resumen del mes», ENTONCES `POST /insights/monthly-summary` responde `303` hacia `/dashboard`, que muestra «Explicación generada automáticamente» encima del texto y «No constituye asesoría financiera» debajo. · E2E
- **(P)** DADO QUE soy `demo1`, CUANDO se envía la solicitud a Ollama, ENTONCES el JSON de datos contiene solo `month`, `totalIncome` (600.00), `totalExpenses` (495.00), `monthlyBalance` (105.00), `budget` (600.00), `consumedPercentage` (82.50), `remainingBudget` (105.00), `overspentAmount` (`null`) y `totalsByCategory`, sin correo, nombre ni `UserId`; y la solicitud es `POST /api/generate` con `stream: false` y `think: false`, sin herramientas. · I
- **(P)** DADO QUE soy `demo3` (sin presupuesto), CUANDO pido el resumen, ENTONCES el JSON lleva `budget`, `consumedPercentage`, `remainingBudget` y `overspentAmount` en `null`, y un texto como «Tu saldo del mes es de -S/ 150.00, un déficit de S/ 150.00» se muestra (cifra válida por valor absoluto). · I
- **(B)** DADO QUE soy `demo4` (sin movimientos), CUANDO pido el resumen, ENTONCES el JSON lleva `totalsByCategory: {}` y totales en 0.00, y un texto que cita «S/ 0.00» se muestra. · I
- **(B)** DADO QUE ya vi el resumen, CUANDO recargo el dashboard, ENTONCES no se llama a Ollama y el resumen no vuelve a aparecer; y al cargar el dashboard por primera vez tampoco se llama a Ollama (RN-037). · E2E
- **(B)** DADO QUE pulso el botón dos veces seguidas, CUANDO la primera solicitud está en curso, ENTONCES el botón aparece deshabilitado con «Generando resumen…» (`aria-live="polite"`) y se envía 1 sola solicitud. · E2E
- **(N)** DADO QUE el modelo devuelve `<script>alert(1)</script>`, CUANDO se muestra el resumen, ENTONCES aparece como texto literal y no se ejecuta nada. · E2E

**Reglas de negocio relacionadas:** RN-034, RN-035, RN-037, RN-040, RN-041, RN-042, RN-128, RN-129, RN-131, RN-165

**Notas:**
- Modelo `qwen3:4b`; instrucciones en `src/ExpenseFlow/Modules/Insights/Prompts/monthly-summary.v1.md` (ADR-003, RN-129).
- KPI U4 (comprensibilidad) se mide con datos ficticios: no valida la utilidad real (`01` §5).
- Sin JavaScript el botón no se deshabilita y un segundo envío es posible (limitación aceptada, DEC-90).

#### HU-010B: Proteger el resumen ante fallos y cifras inventadas
**Prioridad:** Must

Como **estudiante**, quiero **que el dashboard siga funcionando cuando la IA falla y no ver nunca cifras inventadas**, para **confiar en lo que leo**.

**Validación INVEST:** I ◐ · N ✔ · V ✔ · E ✔ · S ✔ · T ✔ — Necesita el flujo de HU-010A para mostrar MSG-23 en el dashboard; el cliente, el verificador y la configuración se construyen y prueban antes.

**Criterios de aceptación:**
- **(P)** DADO QUE `Ai:BaseUrl` es `http://localhost:11434` y `Ai:Model` es `qwen3:4b`, CUANDO arranca la aplicación, ENTONCES la IA queda habilitada y el botón «Generar resumen del mes» aparece habilitado, aunque Ollama no esté disponible (RN-085). · I
- **(N)** DADO QUE el cliente de IA (simulado) señala tiempo de espera agotado, CUANDO pido el resumen, ENTONCES se hace 1 sola llamada, se muestra MSG-23, el dashboard conserva todas sus cifras y el log registra `type=Timeout` sin importes ni identificadores. · I
- **(B)** DADO QUE `Ai:TimeoutSeconds` no está configurado, CUANDO se registra el cliente de Ollama, ENTONCES su tiempo de espera es exactamente 30 s y no tiene política de reintentos. · U
- **(N)** DADO QUE Ollama está detenido (conexión rechazada), CUANDO pido el resumen, ENTONCES se muestra MSG-23, el log registra `type=Connection` y el aviso del 80 % de `demo1` sigue visible. · I
- **(N)** DADO QUE Ollama responde `404` porque el modelo no está descargado, CUANDO pido el resumen, ENTONCES se muestra MSG-23 (no MSG-24), el log registra `type=HttpError status=404` y el botón sigue habilitado. · I
- **(N)** DADO QUE Ollama responde `200` con un texto que solo contiene `<think>…</think>` o espacios, CUANDO termina la solicitud, ENTONCES se muestra MSG-23 y el log registra `type=EmptyResponse`. · U + I
- **(N)** DADO QUE el modelo escribe «Te quedan unos S/ 100» (cifra que no está en el JSON), CUANDO termina la solicitud, ENTONCES se muestra MSG-23 y el log registra `type=InvalidFigure`, sin la cifra rechazada. · U + I
- **(B)** DADO QUE el modelo escribe «Superaste el 80 % de tu presupuesto en noviembre de 2026», CUANDO termina la solicitud, ENTONCES el texto se muestra («80 %» es valor permitido y «2026» no es cifra). · U
- **(N)** DADO QUE `Ai:BaseUrl` está vacío o es `http://192.168.1.50:11434`, CUANDO arranca la aplicación, ENTONCES arranca con normalidad, el botón aparece deshabilitado junto a MSG-24 y M1–M5 funcionan sin restricciones; lo mismo con `localhost:11434` (sin esquema), `http://127.0.0.2:11434` o `Ai:Model` vacío. · I
- **(B)** DADO QUE `Ai:BaseUrl` es `http://[::1]:11434` o `http://LOCALHOST:11434`, CUANDO arranca la aplicación, ENTONCES la IA queda habilitada. · U
- **(B)** DADO QUE `Ai:TimeoutSeconds` vale `0`, `4`, `121` o `-1`, CUANDO arranca la aplicación, ENTONCES usa 30 s y registra la advertencia; con `5` o `120` usa ese valor. · U
- **(N)** DADO QUE uso la API, CUANDO envío `POST /api/insights/monthly-summary`, ENTONCES responde `200` con `{ month, text }`; con la IA deshabilitada, `503` con `code` = `MSG-24`; con un fallo de Ollama o una cifra no válida, `503` con `code` = `MSG-23`. · I

**Reglas de negocio relacionadas:** RN-036, RN-038, RN-039, RN-043, RN-082, RN-083, RN-084, RN-085, RN-086, RN-087, RN-088, RN-130, RN-132, RN-133, RN-134, RN-166

**Notas:**
- En el CI la IA es simulada; las 10 evaluaciones con Ollama real se ejecutan localmente antes del PR (10/10, ADR-003).
- El tiempo de espera se prueba en dos partes, sin esperar 30 s reales: una prueba unitaria verifica que el cliente se configura con 30 s y sin reintentos, y una de integración usa el cliente simulado que señala el tiempo agotado.
- Limitación aceptada (nota de RN-130): no se detectan cifras escritas con palabras («quinientos soles»).

---

## Transversal — Aislamiento, errores y datos de prueba

### HU-011: Tener la garantía de que nadie más ve mis datos
**Prioridad:** Must

Como **usuario**, quiero **que mis movimientos y presupuestos solo sean accesibles desde mi sesión**, para **confiar en registrar mi información financiera en la app**.

**Validación INVEST:** I ✔ · N ◐ · V ✔ · E ✔ · S ✔ · T ✔ — Poco negociable: la impone PRINCIPIO-007.

**Criterios de aceptación:**
- **(P)** DADO QUE soy `demo1`, CUANDO pido `GET /api/budgets/current` y `GET /api/dashboard`, ENTONCES recibo mi presupuesto de S/ 600.00 y ningún movimiento de `demo2` en `recentTransactions`. · I
- **(N)** DADO QUE soy `demo1` y el movimiento 42 pertenece a `demo2`, CUANDO envío `GET`, `PUT` o `DELETE /api/transactions/42`, ENTONCES la respuesta es `404` y el movimiento 42 sigue existiendo sin cambios. · I
- **(N)** DADO QUE soy `demo1` y el movimiento 42 pertenece a `demo2`, CUANDO abro su pantalla de edición o envío su eliminación desde las vistas MVC, ENTONCES la respuesta es `404` con la página en español que muestra MSG-45, idéntica a la de `/transactions/99999/edit` (inexistente), y el movimiento no cambia. · I + E2E (un caso)
- **(N)** DADO QUE soy `demo1` y el presupuesto 7 pertenece a `demo2`, CUANDO envío `PUT /api/budgets/7`, ENTONCES la respuesta es `404`. · I
- **(N)** DADO QUE soy `demo1`, CUANDO envío `POST /api/transactions` con `"userId": "<UserId de demo2>"` en el cuerpo, ENTONCES el gasto se crea con el `UserId` de `demo1`. · I (cuenta de prueba en lugar de `demo1` si la prueba persiste el gasto, `03` §11.4)
- **(N)** DADO QUE soy `demo1`, CUANDO envío `GET /api/transactions?userId=<UserId de demo2>`, ENTONCES solo recibo mis propios movimientos. · I
- **(N)** DADO QUE no tengo sesión, CUANDO envío `GET /api/transactions`, ENTONCES la respuesta es `401` con un `ProblemDetails` en el cuerpo (no `302` ni `200`). · I
- **(N)** DADO QUE tengo sesión pero no envío la cabecera `X-CSRF-TOKEN`, CUANDO envío `POST /api/transactions`, ENTONCES la respuesta es `400` y no se crea el gasto. · I
- **(N)** DADO QUE un formulario de otro sitio envía `POST /transactions/create` con mi cookie pero sin token, CUANDO se procesa, ENTONCES la respuesta es `400` con la página en español que muestra MSG-46 y el enlace «Volver al inicio», sin traza ni texto en inglés, y no se crea el gasto. · I
- **(B)** DADO QUE no tengo sesión ni token, CUANDO envío `POST /api/transactions`, ENTONCES la respuesta es `401`, no `400` (la autenticación se evalúa primero). · I
- **(B)** DADO QUE obtuve un token con `GET /api/antiforgery/token` antes de iniciar sesión, CUANDO lo uso después de iniciar sesión en `POST /api/transactions`, ENTONCES la respuesta es `400`. · I
- **(B)** DADO QUE soy `demo1` y el movimiento 42 es un gasto de `demo2`, CUANDO envío `PUT /api/transactions/42` con `"type": "Income"`, ENTONCES la respuesta es `404`, no `400` con MSG-30: la pertenencia se verifica antes que el cuerpo. · I
- **(N)** DADO QUE abro una ruta que no existe, como `/pagina-inexistente`, CUANDO se carga, ENTONCES veo la página `404` en español con MSG-45 y el enlace «Volver al inicio». · E2E

**Reglas de negocio relacionadas:** RN-007, RN-008, RN-009, RN-101, RN-102, RN-103, RN-112, RN-169, RN-170

**Notas:**
- KPI T2: 0 accesos exitosos a datos de otro usuario en los casos de prueba.
- RN-009 (no entrenar modelos con los datos) no es verificable por prueba: se controla con la checklist de revisión.
- En un `PUT` sobre un recurso ajeno, la pertenencia se verifica antes que el cuerpo, para no revelar que el recurso existe (DEC-58, ADR-004 §3.1 «Precedencia»).
- La página `404` es la misma para recursos ajenos e inexistentes (DEC-64); el rechazo antiforgery en MVC muestra la página `400` con MSG-46 (DEC-71). La respuesta `401` de la API lleva `ProblemDetails` (DEC-88).

---

### HU-013: Ver un mensaje comprensible ante un error inesperado
**Prioridad:** Must

Como **usuario**, quiero **ver un mensaje en español cuando algo falla en el sistema**, para **saber que debo reintentar y no ver información técnica que no entiendo**.

**Validación INVEST:** I ✔ · N ✔ · V ✔ · E ✔ · S ✔ · T ✔

**Criterios de aceptación:**
- **(P)** DADO QUE la aplicación funciona con normalidad, CUANDO abro el dashboard, ENTONCES no veo MSG-40. · E2E
- **(N)** DADO QUE SQL Server está detenido en el entorno `Testing`, CUANDO abro `/dashboard`, ENTONCES veo una página en español con MSG-40, código `500` y ninguna traza, nombre de clase ni texto en inglés. · I
- **(N)** DADO QUE SQL Server está detenido, CUANDO pido `GET /api/dashboard`, ENTONCES la respuesta es `500` con un `ProblemDetails` cuyo `title` es el texto de MSG-40 y cuyo `code` es `MSG-40`. · I
- **(N)** DADO QUE SQL Server se detiene mientras guardo un gasto, CUANDO pulso «Guardar gasto», ENTONCES veo MSG-40 y, al restablecer la base, el gasto no existe. · I
- **(B)** DADO QUE el entorno es `Development`, CUANDO ocurre el mismo error, ENTONCES se permite la página de excepciones para desarrolladores. · I
- **(B)** DADO QUE ocurrió un error no controlado durante un inicio de sesión, CUANDO se revisa el log, ENTONCES contiene el detalle técnico pero no la contraseña. · I

**Reglas de negocio relacionadas:** RN-164, RN-006, RN-133

**Notas:** ADR-004 §7. Cubre el escenario «la red o el servidor fallan» que las demás HU no cubren. Las páginas `404` (RN-169) y `400` (RN-170) se verifican en HU-011.

---

### HU-014: Restablecer las cuentas demo antes de una sesión de prueba
**Prioridad:** Must (habilitador)

Como **facilitador de las sesiones de usabilidad y de las pruebas E2E**, quiero **restablecer las cuentas demo a sus escenarios fijos**, para **que cada participante y cada corrida empiecen con las mismas cifras conocidas**.

**Validación INVEST:** I ✔ · N ◐ · V ✔ · E ✔ · S ✔ · T ✔ — Poco negociable: las HU-005, HU-008, HU-009 y HU-011 dependen de sus datos.

**Criterios de aceptación:**
- **(P)** DADO QUE el seed se ejecuta el 2026-11-08 sobre una base vacía, CUANDO termina, ENTONCES existen `demo1` a `demo6` con la contraseña `Demo#2026x`, los movimientos de la sección 11.2 (mes en curso con fecha 2026-11-01 y mes anterior con fecha 2026-10-15) y las cifras de la sección 11.3. · I
- **(P)** DADO QUE el seed termina, CUANDO abro «Nuevo gasto» como `demo1`, ENTONCES se preselecciona «Yape», porque los `CreatedAt` crecen de 1 en 1 segundo en el orden de la tabla 11.2. · I
- **(B)** DADO QUE el seed se ejecuta dos veces seguidas, CUANDO termina la segunda, ENTONCES `demo1` tiene 12 movimientos, no 24, y ninguna ejecución falla. · I
- **(B)** DADO QUE el seed se ejecuta el 2027-03-01, CUANDO termina, ENTONCES los movimientos llevan fecha 2027-03-01 y 2027-02-15. · I con `FakeTimeProvider`
- **(N)** DADO QUE `ana.perez@gmail.com` tiene 3 movimientos y una cuenta `e2e-<GUID>@expenseflow.test` tiene 2, CUANDO se ejecuta el seed, ENTONCES termina con normalidad y esos 5 movimientos siguen intactos. · I
- **(N)** DADO QUE `demo1` registró un gasto extra durante una sesión, CUANDO se ejecuta el seed, ENTONCES el gasto extra desaparece y `demo1` vuelve a sus 12 movimientos. · I

**Reglas de negocio relacionadas:** RN-138, RN-157, RN-158, RN-159

**Notas:** Lo ejecuta un humano o el CI; un agente no lo ejecuta contra una base existente (RN-138, PRINCIPIO-010). Las credenciales demo no son secretos (ADR-005 §4).

---

## E6 — Post-MVP (Should)

> HU-012 y HU-015 a HU-018 solo se planifican con todos los Must terminados y el CI en verde (`02` §0). HU-015 a HU-018 están en estado **Propuesta** hasta que se especifique su Should (`05` PQ-08, tarea T-056).

### HU-012: Ver cuánto me queda hasta la próxima entrega
**Prioridad:** Should

Como **estudiante que recibe dinero cada semana o cada 15 días**, quiero **indicar mi frecuencia y la fecha de mi última entrega**, para **saber cuánto me queda hasta la próxima entrega y no solo hasta fin de mes**.

**Validación INVEST:** I ✔ · N ✔ · V ✔ · E ✘ · S ✔ · T ◐ — No es estimable ni completamente verificable mientras no existan los mensajes de rechazo de RN-143 y RN-161 ni el texto del rótulo del saldo del periodo. **Pendiente:** los mensajes se definen al especificar S2 (`05` PQ-07, tarea T-055); hasta entonces la HU no cumple la Definition of Ready.

**Criterios de aceptación:**
- **(P)** DADO QUE hoy es 2026-11-08, CUANDO elijo «Cada 7 días» e indico que mi última entrega fue el 2026-11-06, ENTONCES el periodo actual va del 2026-11-06 al 2026-11-12 y el siguiente empieza el 2026-11-13. · U
- **(P)** DADO QUE en el periodo 2026-11-06 a 2026-11-12 tengo ingresos por S/ 150.00 y gastos por S/ 92.50, CUANDO abro el dashboard, ENTONCES veo «saldo del periodo» de S/ 57.50 **además** del saldo del mes y del saldo acumulado; un gasto del 2026-11-13 no cuenta en este periodo. · I
- **(P)** DADO QUE elijo «Cada 15 días» con última entrega el 2026-11-01, CUANDO el sistema calcula el periodo, ENTONCES el siguiente empieza el 2026-11-16 (no el 2026-11-15). · U
- **(B)** DADO QUE elijo «Mensual» con el día 31, CUANDO el sistema calcula las entregas, ENTONCES caen el 2027-02-28 y luego el 2027-03-31; y en 2028 cae el 2028-02-29. · U
- **(B)** DADO QUE hoy es 2026-11-08 y elijo «Cada 7 días», CUANDO indico como última entrega el 2026-11-09 o el 2026-10-31, ENTONCES se rechaza la fecha; las fechas del 2026-11-01 al 2026-11-08 se aceptan. · U
- **(B)** DADO QUE hoy es 2026-11-08, CUANDO elijo «Cada 15 días», ENTONCES se acepta el 2026-10-24 y se rechaza el 2026-10-23; con «Mensual», se acepta el 2026-10-08 y se rechaza el 2026-10-07. · U
- **(N)** DADO QUE configuro la frecuencia, CUANDO veo las opciones, ENTONCES solo existen «Cada 7 días», «Cada 15 días» y «Mensual» (nunca «Quincenal»), y la API rechaza «cada 10 días». · I
- **(B)** DADO QUE configuré «Cada 7 días» y tengo un presupuesto mensual de S/ 600.00, CUANDO abro el dashboard, ENTONCES el porcentaje consumido y los avisos se siguen calculando por mes calendario (DEC-21). · I

**Reglas de negocio relacionadas:** RN-089, RN-090, RN-091, RN-092, RN-093, RN-135, RN-136, RN-143, RN-161

**Notas:**
- Solo se inicia si los Must están terminados con CI en verde (`02` §0).
- **[Pendiente]** Mensajes de rechazo de RN-143 y RN-161 y rótulo del saldo del periodo (`05` PQ-07).
- Si las entrevistas muestran que predomina la entrega semanal, esta HU sube a Must (`02` A4, supuesto SP4; `05` PQ-05).

---

### HU-015: Repartir el presupuesto entre categorías
**Estado:** Propuesta · **Prioridad:** Should (S1)

Como **estudiante**, quiero **asignar un límite a cada categoría**, para **saber qué parte de mi gasto se está pasando**.

**Validación INVEST:** I ✔ · N ✔ · V ✔ · E ✘ · S ✔ · T ◐ — Falta especificar el modelo de datos (tabla aparte, RN-064), las validaciones de monto, si la suma puede superar el presupuesto mensual y los mensajes. **No está lista** para planificar (`05` PQ-08).

**Criterios de aceptación:**
- **(P)** DADO QUE Alimentación tiene presupuesto de S/ 300.00 y gastos de S/ 270.00 en 2026-11, CUANDO abro el dashboard, ENTONCES veo «90.00 %» para Alimentación. · U
- **(N)** DADO QUE el presupuesto mensual es S/ 600.00, CUANDO se calcula el porcentaje de Alimentación, ENTONCES no se usa el presupuesto mensual (no da «45.00 %»). · U
- **(B)** DADO QUE una categoría no tiene presupuesto asignado, CUANDO abro el dashboard, ENTONCES no se calcula su porcentaje (comportamiento análogo a RN-066; **[Pendiente]** confirmar). · U

**Reglas de negocio relacionadas:** RN-044

---

### HU-016: Filtrar mi historial de movimientos
**Estado:** Propuesta · **Prioridad:** Should (S3)

Como **usuario con muchos movimientos**, quiero **filtrar por fecha, categoría, tipo y medio de pago**, para **encontrar un movimiento sin recorrer todas las páginas**.

**Validación INVEST:** I ◐ · N ✔ · V ✔ · E ✘ · S ✔ · T ◐ — Depende de HU-005. Faltan reglas para rangos inválidos (inicio posterior al fin), combinación de filtros con la paginación y mensajes. **No está lista** (`05` PQ-08).

**Criterios de aceptación:**
- **(P)** DADO QUE soy `demo1`, CUANDO filtro por «Transporte» entre el 2026-11-01 y el 2026-11-08, ENTONCES veo mis 2 gastos de Transporte de S/ 60.00. · I
- **(N)** DADO QUE soy `demo1`, CUANDO aplico el mismo filtro, ENTONCES no aparece ningún gasto de `demo2`. · I
- **(B)** DADO QUE soy `demo4`, CUANDO aplico cualquier filtro, ENTONCES veo una lista vacía con un mensaje (**[Pendiente]** definir el texto). · I

**Reglas de negocio relacionadas:** RN-045

---

### HU-017: Seguir el avance de una meta de ahorro
**Estado:** Propuesta · **Prioridad:** Should (S4)

Como **estudiante que ahorra para una laptop**, quiero **definir una meta y ver su avance**, para **saber cuánto me falta**.

**Validación INVEST:** I ✔ · N ✔ · V ✔ · E ✘ · S ◐ · T ✘ — RN-046 indica que «cómo se marca un monto como ahorro se define al especificar S4». Sin esa definición no se puede estimar ni probar. **No está lista** (`05` PQ-08).

**Criterios de aceptación:**
- **(P)** DADO QUE tengo la meta «Laptop» de S/ 2,500.00 y marqué como ahorro S/ 300.00 y S/ 200.00, CUANDO veo la meta, ENTONCES el avance es «20.00 %». · U
- **(N)** DADO QUE tengo un saldo acumulado de S/ 155.00 sin montos marcados como ahorro, CUANDO veo la meta, ENTONCES el avance es «0.00 %» (no se calcula con el saldo acumulado). · U
- **(B)** DADO QUE marqué como ahorro S/ 2,600.00 para una meta de S/ 2,500.00, CUANDO veo la meta, ENTONCES el avance es «104.00 %» (**[Pendiente]** confirmar si se limita a 100 %). · U

**Reglas de negocio relacionadas:** RN-046

---

### HU-018: Pedir una explicación al alcanzar el 80 %
**Estado:** Propuesta · **Prioridad:** Should (S5)

Como **estudiante que acaba de recibir el aviso del 80 %**, quiero **pedir una explicación breve de qué categorías pesan más**, para **decidir dónde recortar**.

**Validación INVEST:** I ◐ · N ✔ · V ✔ · E ◐ · S ✔ · T ✔ — Depende de HU-009 y HU-010. Falta fijar si usa el mismo payload, instrucciones (`v1` o un archivo nuevo) y evaluaciones de ADR-003.

**Criterios de aceptación:**
- **(P)** DADO QUE soy `demo1` (82.50 %), CUANDO veo el aviso del 80 %, ENTONCES junto a él está el botón «Explicar mi consumo». · E2E
- **(N)** DADO QUE soy `demo1`, CUANDO abro el dashboard, ENTONCES no se genera ninguna explicación hasta que pulso el botón (RN-037). · I
- **(B)** DADO QUE soy `demo2` (110.00 %) o `demo3` (sin presupuesto), CUANDO abro el dashboard, ENTONCES no aparece el botón, porque no se muestra el aviso del 80 %. · E2E

**Reglas de negocio relacionadas:** RN-047, RN-037

---

## Verificación contra la constitución

No se encontró ningún criterio que contradiga la constitución. Puntos revisados:

| Principio | Resultado |
|---|---|
| PRINCIPIO-003, PRINCIPIO-004 | Todas las cifras de los criterios las calcula el backend; HU-008 agrega una prueba con JavaScript deshabilitado. |
| PRINCIPIO-005 | HU-010 solo usa hosts de loopback y un payload sin identificadores. |
| PRINCIPIO-006 | Todos los criterios usan cuentas demo o `e2e-<GUID>`; ninguno usa datos reales. |
| PRINCIPIO-007 | HU-011 cubre API y vistas MVC. |
| PRINCIPIO-010 | HU-014 deja la ejecución del seed a un humano o al CI. |
| PRINCIPIO-011 | La IA se prueba simulada en el CI; las reglas de cálculo llevan `Trait`. |
| PRINCIPIO-001 | Todos los textos visibles se citan por su MSG en español; las páginas `400`, `404` y `500` están en español (HU-011, HU-013). |
| PRINCIPIO-012 | Criterios de teclado, axe, `aria-describedby`, `role="status"`, `role="alert"` y color en HU-001, HU-002A, HU-003, HU-006, HU-008 y HU-009. |


---

## Verificación INVEST

| HU | I | N | V | E | S | T | Tamaño | Criterios (P/N/B) | Observación |
|---|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|---|
| HU-001 | ✔ | ✔ | ✔ | ✔ | ◐ | ✔ | M | 2 / 11 / 5 | Base técnica de las demás HU |
| HU-002A | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | M | 1 / 8 / 4 | `returnUrl`, bloqueo y sesión iniciada |
| HU-002B | ◐ | ✔ | ✔ | ✔ | ✔ | ✔ | S | 1 / 3 / 4 | Expiración, MSG-48 y cierre sin sesión |
| HU-003 | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | M | 4 / 8 / 6 | Mide U2; doble envío con RN-168 |
| HU-004 | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | S | 4 / 3 / 5 | Comparte validador con HU-003 |
| HU-005 | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | S | 2 / 2 / 4 | Solo lectura con `demo6` |
| HU-006 | ◐ | ✔ | ✔ | ✔ | ✔ | ✔ | M | 6 / 5 / 4 | Depende de HU-005 en la UI |
| HU-007 | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | M | 2 / 6 / 8 | Mide U3 |
| HU-008 | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | M | 3 / 3 / 5 | Incluye RN-013 |
| HU-009 | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | S | 2 / 2 / 6 | Límites como pruebas unitarias |
| HU-010A | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | M | 3 / 1 / 3 | — |
| HU-010B | ◐ | ✔ | ✔ | ✔ | ✔ | ✔ | M | 1 / 7 / 4 | Necesita el flujo de HU-010A para MSG-23 |
| HU-011 | ✔ | ◐ | ✔ | ✔ | ✔ | ✔ | S | 1 / 9 / 3 | Incluye las páginas `404` y `400` |
| HU-012 | ✔ | ✔ | ✔ | ✘ | ✔ | ◐ | M | 3 / 1 / 4 | No lista: `05` PQ-07 |
| HU-013 | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | S | 1 / 3 / 2 | — |
| HU-014 | ✔ | ◐ | ✔ | ✔ | ✔ | ✔ | S | 2 / 2 / 2 | Habilitador |
| HU-015 a HU-018 | — | — | ✔ | ✘/◐ | ✔ | ◐/✘ | — | 1 / 1 / 1 c/u | Propuestas; no listas: `05` PQ-08 |

✔ cumple · ◐ cumple con observación · ✘ no cumple (motivo en la HU)

---

## Trazabilidad

### Funcionalidad → HU → KPI

| Funcionalidad | HU | KPI del Brief |
|---|---|---|
| M1 — Cuenta y acceso | HU-001, HU-002A, HU-002B, HU-011 | T2, T3 |
| M2 — Registro de gastos | HU-003, HU-005, HU-006 | U2, T1, T3, T4 |
| M3 — Registro de ingresos | HU-004, HU-005, HU-006 | T1, T3 |
| M4 — Presupuesto mensual | HU-007 | U3, T1, T3, T4 |
| M5 — Dashboard | HU-008, HU-009 | T1, T3, T4 |
| M6 — Resumen con IA | HU-010A, HU-010B | U4 |
| Transversal | HU-011, HU-013, HU-014 | T2, T3 |
| S1 / S2 / S3 / S4 / S5 | HU-015 / HU-012 / HU-016 / HU-017 / HU-018 | — |

### Cobertura de reglas

**Las 178 reglas de `03-reglas-de-negocio.md` tienen al menos una HU** (verificado por script al cerrar el Spec Pack; `11` §2). RN-009 se verifica con el checklist de revisión, no con una prueba.

### Fuera de alcance confirmado

No hay HU para W1–W20 ni C1–C4.
