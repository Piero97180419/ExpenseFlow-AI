# 10 — Registro de validación del prototipo — ExpenseFlow AI

| Campo | Valor |
|---|---|
| Documento | 10-validation-log.md |
| Fase | Validación (BMAD — Agente Consolidador de Validación) |
| Versión | v1.0 |
| Fecha | 2026-11-10 |
| Estado | **Aprobado** |
| Aprobado por | Piero Carbajal |
| Fecha de aprobación | 2026-11-10 |
| PR de aprobación | PR que fusiona el Spec Pack v1.0 en `main` (ADR-005 §1) |
| Insumos | Prototipo visual de `09-ui-spec.md` (P-001 a P-015), reporte del Agente Reviewer (`10-anexo-reporte-reviewer.md`, OBS-01 a OBS-35) |

> **Convenciones**
> - **#** = ID consolidado (V-NN). La columna **Fuente** indica el ID original (OBS-NN del anexo).
> - Severidad: **Alta** = deja al usuario sin salida o bloquea una HU Must · **Media** = ambigüedad o fricción que llevaría a implementaciones distintas · **Baja** = trazabilidad, redacción o mejora.
> - Orden: severidad (Alta → Media → Baja); dentro de cada severidad, primero las que bloquean una HU.
> - Las observaciones describen el prototipo **revisado**; las referencias a decisiones y reglas usan la numeración del Spec Pack v1.0.
> - Estados: **✅ APLICADA** (resuelta en los documentos v1.0) · **⏳ PENDIENTE** (requiere una decisión que aún no existe) · **⏳ BACKLOG** (tarea T-NNN de `08-backlog.md`).

---

## 0. Fuentes

| Fuente | Documento | Estado | Observaciones aportadas |
|---|---|---|---|
| **Agente Reviewer** | `10-anexo-reporte-reviewer.md` (revisiones R1, R2 y R3; OBS-01 a OBS-35) | ✔ Incorporada | 35 → 34 filas (OBS-20 y OBS-31 combinadas en V-11) |
| **Compañero** | Revisión humana | ✘ No recibida | 0 |
| **Propia** | Observaciones del Agente Consolidador | Sin observaciones nuevas: la consolidación se limita a las fuentes recibidas | 0 |

> Si llega la revisión de un compañero, sus hallazgos se agregan a la tabla 1 con la fuente «Compañero»; si coinciden con una fila existente, se combinan indicando «Compañero + Agente Reviewer».

---

## 1. Tabla de observaciones

| # | Observación | Fuente | Tipo | Severidad | Acción sugerida | HU afectada |
|---|---|---|---|---|---|---|
| V-01 | Callejón sin salida: si la sesión expira y el usuario envía un formulario cuya ruta solo admite `POST` (`POST /insights/monthly-summary` en P-003, `POST /transactions/{id}/delete` en P-008, `POST /account/logout` en CMP-01), tras iniciar sesión se redirige con `GET` al `returnUrl`, que no existe. | Agente Reviewer (OBS-03) | gap | **Alta** | En F-01 y en la nota de RN-155: si el `returnUrl` apunta a una ruta sin `GET`, redirigir a la pantalla que contiene ese formulario (P-003 para la IA, `/transactions?page=n` para la eliminación) o a `/dashboard`. Criterio nuevo en HU-002B con `FakeTimeProvider` | HU-002B, HU-006, HU-010A · RN-098, RN-155 |
| V-02 | El rechazo antiforgery `400` (RN-101) no tiene página: un registro o inicio de sesión enviado desde una pestaña abierta antes de iniciar sesión en otra recibe un `400` vacío. Además, «Cerrar sesión» (CMP-01) con la sesión expirada lleva al inicio de sesión con `returnUrl=/account/logout`, una ruta que solo admite `POST`, porque la autorización se evalúa antes que el antiforgery. | Agente Reviewer (OBS-04) | gap | **Alta** | (1) Página `400` en español con CMP-02, un mensaje nuevo en `03` §10 y «Volver al inicio». (2) `POST /account/logout` sin sesión ⇒ `303` a P-002 sin error ni `returnUrl`. Criterio en HU-002B | HU-002B, HU-001, HU-011 · RN-097, RN-101 |
| V-03 | P-002 no dice que el campo oculto `returnUrl` se conserve al volver a mostrar el formulario tras MSG-02, MSG-32 o MSG-34. Con un solo error de tipeo, el usuario termina en `/dashboard` en lugar de P-004 y falla un criterio de HU-002B. | Agente Reviewer (OBS-18) | gap | Media | Estado de P-002: «el `returnUrl` se conserva en cada nueva presentación del formulario». Criterio en HU-002B: expiración → un intento fallido → inicio correcto → P-004 | HU-002B, HU-002A · RN-098, RN-155 |
| V-04 | No se define qué pasa si un usuario con sesión abre `GET /account/register` o `GET /account/login` (por ejemplo, con «Atrás»). Podría crear una segunda cuenta y cambiar de sesión. | Agente Reviewer (OBS-05) | gap | Media | Usuario autenticado en `/account/register` o `/account/login` ⇒ `303` a `/dashboard`. Notas en RN-145 y RN-153; criterios en HU-001 y HU-002A | HU-001, HU-002A · RN-145, RN-153 |
| V-05 | CMP-10 no incluye P-001 («Crear cuenta»), P-002 («Iniciar sesión»), P-009 ni P-010. Un doble envío puede mostrar MSG-04 justo después de crear la cuenta, o MSG-21 justo después de crear el presupuesto. | Agente Reviewer (OBS-06) | fricción | Media | Agregar a CMP-10 las filas de P-001 («Creando cuenta…»), P-002, P-009 y P-010, como mejora progresiva y sin regla nueva | HU-001, HU-002A, HU-007 · RN-064, RN-095, RN-168 |
| V-06 | No se dice si la casilla «Declaro ser mayor de 18 años» conserva su estado tras un error; si vuelve desmarcada, cada reintento provoca MSG-01. | Agente Reviewer (OBS-07) | comprensión | Media | Estado «Errores» de P-001: «la casilla conserva el valor enviado». Criterio E2E en HU-001 | HU-001 · RN-001, RN-002 |
| V-07 | No se define el orden ni la combinación de errores de «Contraseña» y «Confirmar contraseña» (RN-149 solo ordena el correo). | Agente Reviewer (OBS-08) | comprensión | Media | Nota en RN-152: Contraseña MSG-34 → MSG-03; Confirmación MSG-36 → MSG-35, y MSG-35 solo si la contraseña no está vacía. Criterios en HU-001 | HU-001 · RN-094, RN-149, RN-152 |
| V-08 | `type="email"` puede enviar en punycode un dominio con «ñ», que sí cumple RN-150. Los criterios de HU-001 solo prueban la «ñ» en la parte local. Riesgo por verificar. | Agente Reviewer (OBS-10) | gap | Media | Prueba E2E en Chromium con «ñ» en el dominio. Si se acepta, cambiar a `type="text"` + `inputmode="email"` en P-001 y P-002 | HU-001 · RN-150 |
| V-09 | La gestión del foco tras un error es ambigua: CMP-04 tiene `role="alert" tabindex="-1"` (patrón para enfocar el resumen) y las reglas comunes ponen `autofocus` en el primer campo con error. | Agente Reviewer (OBS-11) | comprensión | Media | Elegir el foco en el primer campo con error (funciona sin JavaScript) y quitar `tabindex="-1"` de CMP-04. Verificar con axe y un lector de pantalla | HU-001, HU-003, HU-004, HU-007 · RN-116 |
| V-10 | Colisión de identificadores: `09` usa P-NNN para pantallas, mientras `03` y `06` usan P-0NN para principios (P-007, P-008 y P-012 significan cosas distintas según el documento). | Agente Reviewer (OBS-12) | comprensión | Media | Renombrar las pantallas a PAN-NNN en `09` (y en este log), o actualizar `03` y `06` a PRINCIPIO-0NN. Registrar la convención en `/docs/README.md` | Todas · RN-116, RN-169 |
| V-11 | Hay errores que no corresponden a un campo y no usan CMP-04, sin que esté declarado en las reglas comunes: MSG-02 en P-002 (bloque propio, foco en «Contraseña»), y MSG-21 y MSG-22 en P-009 (bloque `role="alert"`). P-002 tampoco dice si CMP-04 aparece con campos vacíos. | Agente Reviewer (OBS-20 + OBS-31) | comprensión | Media | Regla común nueva: «Los errores que no son de campo (MSG-02, MSG-21, MSG-22) se muestran en un bloque `role="alert"` propio, sin CMP-04 y sin `aria-invalid` en los campos». En P-002: con campos vacíos se usa CMP-04 | HU-002A, HU-007 · RN-050, RN-064, RN-116, RN-154, RN-167 |
| V-12 | Al llegar a P-002 por expiración de la sesión no se muestra ningún mensaje. Se cita la decisión de no mostrar confirmaciones tras guardar o eliminar, que no trata de este caso. El usuario acaba de perder lo escrito (DEC-59) y no sabe por qué. | Agente Reviewer (OBS-22) | fricción | Media | Decisión del PO: (a) texto informativo nuevo en §10 cuando la llegada trae `returnUrl` (p. ej., «Tu sesión terminó. Inicia sesión para continuar.»); o (b) aceptarlo citando DEC-59, no DEC-69 | HU-002B · RN-098 |
| V-13 | El correo de P-002 no lleva `spellcheck="false"` ni `inputmode="email"`, a diferencia de P-001. Como MSG-02 no dice qué campo falló, un autocorrector de Android puede producir un fallo imposible de diagnosticar. Tampoco se dice si el inicio de sesión quita los espacios exteriores del correo. | Agente Reviewer (OBS-23) | fricción | Media | Igualar los atributos con P-001. Nota en RN-154: el correo se compara sin espacios exteriores. Criterio en HU-002A con `" demo1@expenseflow.test "` | HU-002A · RN-050, RN-150 |
| V-14 | Los textos literales que verifican las HU no existen como texto en el DOM. CMP-07 pone el monto en otra línea sin « · ». P-003 usa un `<dl>` sin dos puntos, cuando los criterios esperan «Presupuesto: S/ 600.00», «Porcentaje consumido: No disponible» o «Restante del presupuesto: S/ 105.00». | Agente Reviewer (OBS-25) | comprensión | Media | Decidir una opción: (1) que el DOM contenga el literal (dos puntos en el `<dt>`, « · » visualmente oculto en CMP-07); o (2) reescribir esos criterios de `06` para que verifiquen cada campo. Hacerlo antes de escribir los E2E | HU-003, HU-004, HU-007, HU-008, HU-009 · RN-068, RN-069, RN-071, RN-080 |
| V-15 | Tras MSG-22 (cambio de mes) no se define qué mes lleva el campo oculto `month` al volver a mostrar P-009: si es el viejo, cada reenvío repite MSG-22. En P-010, tras MSG-22 no hay salida hacia «Crear presupuesto». | Agente Reviewer (OBS-26) | gap | Media | P-009: volver a mostrar con `month` y el rótulo del mes nuevo, conservando el monto. P-010: MSG-22 con un enlace a P-009. Criterio en HU-007 con `FakeTimeProvider` | HU-007 · RN-118, RN-167 |
| V-16 | No se define `GET /budgets/create` con un presupuesto ya existente, ni `GET /budgets/edit` sin presupuesto (con «Atrás», una pestaña vieja o un enlace guardado). | Agente Reviewer (OBS-27) | gap | Media | `GET /budgets/create` con presupuesto ⇒ `303` a `/budgets/edit`; `GET /budgets/edit` sin presupuesto ⇒ `303` a `/budgets/create`. Agregarlo a F-05 y a HU-007 | HU-007 · RN-064, RN-118 |
| V-17 | Sin confirmaciones tras guardar (decisión del prototipo, reemplazada por DEC-69), el resultado a veces no se ve. Un movimiento con fecha antigua no aparece entre los 5 últimos de P-003, aunque F-02 lo promete. Una edición que cambia la fecha puede mover la fila fuera de la página *n*. Hay riesgo de que el usuario registre el movimiento dos veces. | Agente Reviewer (OBS-28) | fricción | Media | Decisión del PO: (a) excepción a la decisión de no mostrar confirmaciones con una confirmación breve `role="status"` tras guardar (nuevo MSG en §10); o (b) mantener DEC-69 y corregir el texto de F-02. Observarlo en las sesiones U2 | HU-003, HU-004, HU-006 · RN-078, RN-079 |
| V-18 | Si se elimina el único movimiento de la última página (*n* > 1), P-006 muestra MSG-31 («No hay más movimientos») aunque existan movimientos en páginas anteriores. | Agente Reviewer (OBS-29) | fricción | Media | Tras eliminar, si la página *n* quedó vacía y *n* > 1 ⇒ `303` a la página *n* − 1. Ajustar DEC-65 y P-006 | HU-005, HU-006 · RN-117 |
| V-19 | Dos componentes referenciados siguen sin ficha: CMP-03 (campo con etiqueta, ayuda y error; lo usan todos los formularios) y CMP-12 (paginación de P-006). | Agente Reviewer (OBS-32; continúa OBS-01 y OBS-02) | gap | Media | Describir CMP-03 y CMP-12 en `09`, con el mismo nivel que CMP-05 y CMP-07 | HU-001, HU-003, HU-004, HU-005, HU-007 · RN-116, RN-117 |
| V-20 | El texto de ayuda de la contraseña está marcado [Propuesta] y no figura en el catálogo de `03` §10. | Agente Reviewer (OBS-09) | gap | Baja | Agregar MSG-47 a `03` §10 con ese texto y quitar la marca [Propuesta] de P-001 | HU-001 · RN-094, RN-116 |
| V-21 | P-001 y HU-001 citan RN-146 («sin Recordarme»), que es una regla del inicio de sesión; en el registro ya lo cubre RN-148. | Agente Reviewer (OBS-13; parcialmente resuelta en R2) | comprensión | Baja | Quitar RN-146 de la fila «Reglas» de P-001 (y de HU-001). Se mantiene en P-002 | HU-001, HU-002A · RN-146, RN-148 |
| V-22 | El texto del enlace de P-002 a P-001 no coincide: P-002 dice «¿No tienes cuenta? Crea una» y F-01 dice «Crear una cuenta». Ambos se declaran literales. | Agente Reviewer (OBS-19) | comprensión | Baja | Unificar en «¿No tienes cuenta? Crea una» en P-002 y en F-01 | HU-001, HU-002A |
| V-23 | El estado «campos vacíos» de P-002 no dice si MSG-32 y MSG-34 aparecen a la vez. | Agente Reviewer (OBS-21) | comprensión | Baja | Escribir «a la vez, cada uno bajo su campo», como en P-001. Agregar el caso al criterio de RN-154 en HU-002A | HU-002A · RN-154 |
| V-24 | P-002 no declara como limitación la falta de recuperación de contraseña (W17). Además, no cita RN-097, RN-098 ni RN-147, pese a ser el destino de HU-002B. | Agente Reviewer (OBS-24) | gap | Baja | Nota en P-002: «Limitación aceptada (W17): sin recuperación de contraseña». Completar su fila «Reglas» | HU-002A, HU-002B · RN-145, RN-147 |
| V-25 | La vuelta al origen es irregular: «Cancelar» de P-007 no dice si conserva el `page` de origen. Además, P-004 y P-005 abiertas desde P-006 vuelven siempre a `/dashboard`. | Agente Reviewer (OBS-30) | fricción | Baja | Ahora: «Cancelar» de P-007 ⇒ `/transactions?page=n`. El destino de P-004 y P-005 abiertas desde P-006 queda en el backlog | HU-005, HU-006 · DEC-65 |
| V-26 | Errores de trazabilidad. P-008 cita RN-113, que no aplica, en lugar de RN-114, RN-115 y RN-020. P-003 aplica DEC-57 al botón de IA sin JavaScript, pero RN-131 no declara esa limitación. | Agente Reviewer (OBS-33) | comprensión | Baja | Corregir la fila «Reglas» de P-008. Agregar a RN-131 la nota de la limitación sin JavaScript (extensión de DEC-57) | HU-006, HU-010A · RN-113, RN-114, RN-115, RN-131 |
| V-27 | En P-007, un movimiento fuera de la ventana de 12 meses solo puede eliminarse, pero el usuario lo descubre recién al guardar (MSG-17). | Agente Reviewer (OBS-34) | mejora | Baja | Mostrar MSG-17 como aviso desde la carga de P-007 cuando el movimiento ya está fuera de la ventana | HU-006 · RN-109 |
| V-28 | (Should) P-014 y P-015 reutilizan CMP-08 con un `aria-label` fijo y el color «danger» al llegar al 100 %, que no sirve para metas de ahorro. | Agente Reviewer (OBS-35) | mejora | Baja | Al especificar S1 y S4, parametrizar en CMP-08 el `aria-label` y la escala de color, o crear un componente de avance de meta | HU-015, HU-017 · RN-044, RN-046, RN-141 |
| V-29 | MSG-02 no distingue la cuenta bloqueada: el usuario no sabe que debe esperar 5 minutos. Es una decisión aprobada (DEC-29). | Agente Reviewer (OBS-16, aceptada) | fricción | Baja | Sin cambios en la especificación. Medir en las sesiones de usabilidad si genera reintentos o abandono | HU-002A · RN-050 |
| V-30 | Las contraseñas se vacían tras cualquier error de P-001, incluso uno que solo afecta al correo, y en Android hay que reescribir dos contraseñas complejas. Es una práctica de seguridad habitual. | Agente Reviewer (OBS-17, aceptada) | fricción | Baja | Sin cambios en la especificación. Medir el tiempo de registro en las sesiones de usabilidad | HU-001 · RN-094 |
| V-31 | El prototipo solo describía P-001; las demás pantallas Must no tenían ficha. | Agente Reviewer (OBS-01) | gap | Alta | R3 entregó P-003 a P-012. CMP-03 y CMP-12 siguen pendientes en V-19 | HU-002A a HU-013 |
| V-32 | CMP-01 (layout con sesión y «Cerrar sesión») no estaba descrito. | Agente Reviewer (OBS-02) | gap | Alta | R3 describe CMP-01 | HU-002B · RN-097 |
| V-33 | El primer uso tras el registro (dashboard vacío y sus siguientes pasos) no estaba prototipado. | Agente Reviewer (OBS-15) | gap | Alta | P-003 incluye el estado de `demo4` con «Crear presupuesto», «Nuevo gasto» y «Nuevo ingreso» | HU-008, HU-003, HU-007 · RN-070 |
| V-34 | Las páginas 404 y 500 no tenían acción de salida. | Agente Reviewer (OBS-14) | gap | Media | P-011 y P-012 tienen «Volver al inicio» | HU-011, HU-013 · RN-164, RN-169 |

---

## 2. Decisiones tomadas

**Criterio:** se aplican en v1.0 todas las observaciones de severidad Alta y Media y las de severidad Baja que corrigen texto o trazabilidad. Las cuatro que requerían una decisión del PO (V-08, V-12, V-14 y V-17) se resolvieron con la decisión de Piero Carbajal del 2026-11-10. Van al backlog las mejoras Should y lo que se mide en las sesiones de usabilidad.

| # | Estado | Decisión y dónde se aplica | Tarea |
|---|---|---|---|
| V-01 | ✅ APLICADA | RN-174 y DEC-70; `07` §4.9; `09` F-01 y P-002; criterio en HU-002A | — |
| V-02 | ✅ APLICADA | RN-170 (MSG-46, P-016) y RN-177 (`POST /account/logout` `[AllowAnonymous]`), DEC-71; `07` §2.3 ARQ-01 y §4.9; `09` P-016 y CMP-01; criterios en HU-002B y HU-011 | — |
| V-03 | ✅ APLICADA | DEC-72 (nota de RN-155); `09` P-002; criterio en HU-002A | — |
| V-04 | ✅ APLICADA | RN-173 y DEC-73; `07` §4.9; `09` P-001 y P-002; criterios en HU-001 y HU-002A | — |
| V-05 | ✅ APLICADA | DEC-74 (nota de RN-168); `09` CMP-10 con P-001, P-002, P-009 y P-010; criterios en HU-001 y HU-007 | — |
| V-06 | ✅ APLICADA | DEC-75 (nota de RN-148); `09` P-001; criterio en HU-001 | — |
| V-07 | ✅ APLICADA | DEC-76 (nota de RN-152); `09` P-001; criterio en HU-001 | — |
| V-08 | ✅ APLICADA | Decisión del PO (Piero Carbajal, 2026-11-10): correo con `type="text"` + `inputmode="email"`, DEC-77 (nota de RN-150); `09` P-001 y P-002. No requiere la verificación en Chromium | — |
| V-09 | ✅ APLICADA | DEC-78 (nota de RN-116); `09` §3.0 y CMP-04 sin `tabindex` | — |
| V-10 | ✅ APLICADA | Esquema único de identificadores DEC-79: PRINCIPIO-0NN para principios y P-NNN solo para pantallas; convención en `04` §4 y en `/docs/README.md` | — |
| V-11 | ✅ APLICADA | DEC-80 (nota de RN-116); `09` §3.0, P-002, P-009 y P-010 | — |
| V-12 | ✅ APLICADA | Decisión del PO (Piero Carbajal, 2026-11-10): aviso MSG-48 al llegar por expiración, RN-171 y DEC-81; `07` §2.3 (cookie de marca `ef_session`); `09` P-002; criterios en HU-002B | — |
| V-13 | ✅ APLICADA | DEC-82 (nota de RN-154); `09` P-002; criterio en HU-002A | — |
| V-14 | ✅ APLICADA | Decisión del PO (Piero Carbajal, 2026-11-10): el DOM contiene el literal, DEC-83 (nota de RN-080); `09` P-003 y CMP-07 | — |
| V-15 | ✅ APLICADA | DEC-84 (nota de RN-167); `09` P-009 y P-010; criterio en HU-007 | — |
| V-16 | ✅ APLICADA | RN-175, RN-176 y DEC-85; `07` §4.9; `09` F-05, P-009 y P-010; criterios en HU-007 | — |
| V-17 | ✅ APLICADA | Decisión del PO (Piero Carbajal, 2026-11-10): confirmación breve tras guardar o eliminar, RN-172, MSG-49 a MSG-53 y DEC-69; `09` §3.0, F-02 a F-05; criterios en HU-003, HU-004, HU-006 y HU-007 | — |
| V-18 | ✅ APLICADA | RN-178 y DEC-86; `07` §4.9; `09` P-006 y P-008; criterio en HU-006 | — |
| V-19 | ✅ APLICADA | Fichas de CMP-03 y CMP-12 en `09` §3 | — |
| V-20 | ✅ APLICADA | MSG-47 en `03` §10, DEC-87; `09` P-001 | — |
| V-21 | ✅ APLICADA | RN-146 fuera de P-001 y HU-001; se mantiene en P-002 y HU-002A | — |
| V-22 | ✅ APLICADA | Texto «¿No tienes cuenta? Crea una» en `09` F-01, P-002 y §5 | — |
| V-23 | ✅ APLICADA | `09` P-002: MSG-32 y MSG-34 a la vez con CMP-04 | — |
| V-24 | ✅ APLICADA | `09` P-002: limitación W17 y reglas RN-097, RN-098, RN-147 | — |
| V-25 | ✅ APLICADA / ⏳ BACKLOG | «Cancelar» de P-007 vuelve a `/transactions?page=n` (DEC-65, `09` P-007, HU-006). Volver a la lista tras guardar desde P-006 queda en el backlog | T-052 |
| V-26 | ✅ APLICADA | Reglas de P-008 corregidas (RN-020, RN-114, RN-115); limitación sin JavaScript del botón de IA en la nota de RN-131 (DEC-90) | — |
| V-27 | ⏳ BACKLOG | Aviso anticipado de MSG-17 al abrir P-007 (Should; requiere registrar la decisión antes de implementarla) | T-051 |
| V-28 | ⏳ BACKLOG | Parametrizar CMP-08 al especificar S1 y S4 (HU-015, HU-017; `05` PQ-08) | T-053 |
| V-29 | ⏳ BACKLOG | Medir el efecto de MSG-02 con cuenta bloqueada en las sesiones de usabilidad (DEC-29 se mantiene) | T-054 |
| V-30 | ⏳ BACKLOG | Medir el tiempo de registro con contraseñas vaciadas en las sesiones de usabilidad (DEC-75 se mantiene) | T-054 |
| V-31 | ✅ APLICADA | Fichas de P-003 a P-012 en `09` §3 | — |
| V-32 | ✅ APLICADA | Ficha de CMP-01 en `09` §3 | — |
| V-33 | ✅ APLICADA | Estado de `demo4` en P-003 con «Crear presupuesto», «Nuevo gasto» y «Nuevo ingreso» | — |
| V-34 | ✅ APLICADA | P-011, P-012 y P-016 con «Volver al inicio» | — |

### Resumen

| Estado | Cantidad | Observaciones |
|---|---|---|
| ✅ APLICADA | 30 | V-01 a V-26, V-31 a V-34 (V-25 aplicada en parte) |
| ⏳ PENDIENTE | 0 | — |
| ⏳ BACKLOG | 5 | V-25 (parte), V-27, V-28, V-29, V-30 → T-052, T-051, T-053, T-054 |

**Antes de implementar:** ninguna observación bloquea una HU Must. Las tareas del backlog son Should y no tienen tareas Must que dependan de ellas (`08` §8).

---

## 3. Archivos actualizados

| Archivo | Versión | Cambios | Observaciones |
|---|---|---|---|
| `03-reglas-de-negocio.md` | v1.0 | RN-170 a RN-178; MSG-46 a MSG-53; DEC-69 a DEC-90 en §9; notas de RN-080, RN-101, RN-116, RN-131, RN-148, RN-150, RN-152, RN-154, RN-155, RN-167, RN-168 | V-01 a V-18, V-20, V-26 |
| `04-glosario.md` | v1.0 | Términos de interfaz y técnicos (aviso de sesión expirada, mensaje de confirmación, página de formulario caducado…); convención de identificadores (§4) | V-10, V-12, V-17 |
| `06-user-stories.md` | v1.0 | Criterios nuevos en HU-001, HU-002A, HU-002B, HU-003, HU-004, HU-006, HU-007 y HU-011; RN-146 fuera de HU-001 | V-01 a V-07, V-12, V-13, V-15 a V-18, V-21, V-23 |
| `07-architecture.md` | v1.0 | ARQ-01 (cookie de marca, página `400`, `404`), ARQ-02 (`returnUrl`, usuario autenticado), ARQ-04 (redirecciones), §4.9 (rutas con pantallas, confirmaciones y `POST /account/logout` `[AllowAnonymous]`) | V-01, V-02, V-04, V-12, V-16, V-17, V-18 |
| `08-backlog.md` | v1.0 | T-051 a T-054 | V-25, V-27 a V-30 |
| `09-ui-spec.md` | v1.0 | F-01 a F-05; §3.0 (errores que no son de un campo, confirmaciones); CMP-01, CMP-03, CMP-04, CMP-07, CMP-10, CMP-12; P-001 a P-010; P-016 nueva | V-01 a V-26, V-31 a V-34 |
| ADR-004, ADR-006 | v1.0 | Antiforgery en MVC, `POST /account/logout` `[AllowAnonymous]`, `401` con `ProblemDetails`; confirmaciones y errores que no son de un campo | V-02, V-11, V-17 |
| `checklist-revision.md` | v1.0 | Puntos de RN-171 a RN-174, RN-177, páginas `404`/`400`, foco y confirmaciones | V-01 a V-04, V-09, V-11, V-17 |
