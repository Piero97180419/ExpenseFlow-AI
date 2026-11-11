# 10 (anexo) — Reporte del Agente Prototype Reviewer, R1 a R3 — ExpenseFlow AI

| Campo | Valor |
|---|---|
| Documento | 10-anexo-reporte-reviewer.md (fuente 1 de `10-validation-log.md` v2.0) |
| Fase | Validación (BMAD — Agente Prototype Reviewer) |
| Fecha | 2026-11-10 |
| Versión | v1.2 — revisión R3 (P-003 a P-015) y estado consolidado agregados al final |
| Objeto revisado | Descripción del prototipo tomada de `09-ui-spec.md` v1.2. **R1:** P-001, CMP-02, CMP-04, CMP-10, reglas comunes y F-01. **R2:** P-002, CMP-02, reglas comunes y F-01. **R3:** P-003 a P-015, CMP-01, CMP-02, CMP-04 a CMP-10, flujos F-02 a F-07 y transversales |
| Insumos | `06-user-stories.md` v1.3, `03-reglas-de-negocio.md` v1.9 (incluye el catálogo §10 y los escenarios §11) |
| Estado | **Propuesto**: requiere la aprobación de Piero Carbajal mediante un PR (ADR-006) |

> **Convenciones**
> - Severidad: **Alta** = impide validar o implementar una HU Must, o deja al usuario sin salida · **Media** = ambigüedad o fricción que llevaría a implementaciones distintas · **Baja** = trazabilidad, redacción o fricción ya aceptada.
> - Tipo: **comprensión** (la especificación se puede leer de dos formas) · **gap** (falta especificación) · **fricción** (la tarea se completa, pero con esfuerzo evitable) · **mejora**.
> - Identificadores de pantalla (P-NNN) y de principio (PRINCIPIO-0NN) como en `09`. Ver OBS-12 sobre la colisión con la notación P-0NN de `03` y `06`.
> - Las HU contenedoras (HU-002, HU-010) no se cuentan: se dan por terminadas cuando terminan sus HU hijas.
> - Las observaciones (OBS-NN) y las decisiones (DEC-NN) se numeran de forma continua entre revisiones.

### Historial de revisiones

| Revisión | Fecha | Pantalla | Observaciones | Veredicto de la pantalla |
|---|---|---|---|---|
| R1 | 2026-11-10 | P-001 — Crear cuenta | OBS-01 a OBS-17 | Lista con ajustes (DEC-04 a DEC-09) |
| R2 | 2026-11-10 | P-002 — Iniciar sesión | OBS-18 a OBS-24 | Lista con ajustes (DEC-13 a DEC-18); depende de DEC-02 a DEC-04 |
| R3 | 2026-11-10 | P-003 a P-012 (Must) y P-013 a P-015 (Should, borradores) | OBS-25 a OBS-35 | 11 de 14 HU Must listas; HU-002B, HU-006 y HU-010A dependen de DEC-02, DEC-03 y DEC-13 |

---

# Revisión R1 — P-001 Crear cuenta

## Resumen ejecutivo

El prototipo especifica con detalle **una sola pantalla (P-001, Crear cuenta)** y tres componentes. P-001 es sólida: refleja todas las reglas de HU-001 que son observables en la UI, con mensajes del catálogo, orden de validación, accesibilidad y mejora progresiva. El resto de las pantallas de las HU Must (P-002, P-003, P-004, P-005, P-007, P-012 y las de lista y presupuesto) **se citan pero no se incluyen**, así que la mayor parte del MVP no se puede validar todavía.

| Indicador | Valor |
|---|---|
| HU Must cubiertas por completo | **1 de 14 (7.1 %)** |
| HU Must representadas (completas + parciales) | 8 de 14 (57.1 %) |
| Cobertura ponderada (completa = 1, parcial = 0.5) | 32.1 % |
| Reglas Must reflejadas explícitamente en la UI | **19 de 156 (12.2 %)** |
| Reglas Must reflejadas (explícitas + parciales) | 28 de 156 (17.9 %) |
| Reglas Must de UX y Avisos reflejadas (explícitas + parciales) | 13 de 52 (25.0 %) |
| Reglas de HU-001 observables en la UI y reflejadas en P-001 | 13 de 13 (100 %) |
| Observaciones | 17 (Alta 5 · Media 8 · Baja 4) |
| Callejones sin salida | 4 (OBS-03, OBS-04, OBS-05, OBS-14) |

**Veredicto:** P-001 está **lista para implementarse** con los ajustes de severidad media (OBS-05 a OBS-08, OBS-10, OBS-11). El prototipo en conjunto **no está listo**: hay que completar las fichas de las demás pantallas y resolver los callejones sin salida antes de pasar a implementación.

---

## 1. Cobertura de HU

Denominador: 15 HU Must hoja menos HU-014, que es un habilitador sin interfaz (el seed lo ejecuta un humano o el CI, RN-138) = **14 HU con interfaz**.

| HU | Título | Cobertura | Dónde aparece | Qué falta |
|---|---|---|---|---|
| HU-001 | Crear mi cuenta declarando mayoría de edad | ✔ **Completa** | P-001, CMP-02, CMP-04, F-01 | Solo el destino (P-003 con «Presupuesto: No configurado» y MSG-41) no está descrito |
| HU-002A | Iniciar sesión con protección ante intentos fallidos | ◐ Parcial | F-01 (MSG-02, MSG-32, MSG-34, `returnUrl`) | P-002: campos, sin «Recordarme» (RN-146), enlace «Crear una cuenta» |
| HU-002B | Cerrar sesión y mantener la sesión protegida | ◐ Parcial | F-01 («Cerrar sesión», sesión expirada → P-004 vacío) | CMP-01: dónde está «Cerrar sesión»; comportamiento con la sesión expirada (OBS-03, OBS-04) |
| HU-003 | Registrar un gasto en pocos segundos | ◐ Parcial | CMP-10 («Guardar gasto» / «Guardando…»), ejemplo de CMP-04 (MSG-11, MSG-12), F-01 | P-004: 4 campos, fecha y medio de pago propuestos, `inputmode="decimal"` |
| HU-004 | Registrar el dinero que recibo | ◐ Parcial | CMP-10 («Guardar ingreso»), reglas comunes (mostrar u ocultar «Descripción») | P-005: campos, orígenes, «Descripción» sin JavaScript |
| HU-005 | Consultar la lista paginada de mis movimientos | ✘ Ausente | — | Ficha de la lista: paginación de 20, MSG-31, MSG-41, «Editar» y «Eliminar» |
| HU-006 | Corregir o eliminar un movimiento | ◐ Parcial | CMP-10 (P-007 «Guardar cambios»), reglas comunes (diálogo de eliminación) | P-007; `<dialog>` y `<details>` con MSG-18 (RN-114, D-UI-05); destino tras guardar (D-UI-03) |
| HU-007 | Crear y editar el presupuesto del mes | ✘ Ausente | — | Fichas «Crear presupuesto» y «Editar presupuesto»: campo oculto `month`, sin botón eliminar, MSG-21, MSG-22 |
| HU-008 | Ver mis saldos y en qué gasto | ✘ Ausente | Solo como destino (P-003) en F-01 | P-003: saldos `<h2>`/`<h3>`, leyendas, gasto por categoría, últimos 5 movimientos, estados vacíos |
| HU-009 | Recibir avisos al 80 % y al 100 % del presupuesto | ✘ Ausente | — | Avisos con `role="status"`, barra con `aria-valuenow` y `aria-valuetext`, «Presupuesto excedido», «Exceso» |
| HU-010A | Solicitar y ver el resumen del mes | ◐ Parcial | CMP-10 («Generar resumen del mes» / «Generando resumen…») | Recuadro del resumen con «Explicación generada automáticamente» y «No constituye asesoría financiera»; PRG (RN-165) |
| HU-010B | Proteger el resumen ante fallos y cifras inventadas | ✘ Ausente | — | Presentación de MSG-23 y del botón deshabilitado con MSG-24 |
| HU-011 | Tener la garantía de que nadie más ve mis datos | ✘ Ausente (en la UI) | — | P-012 (`404` con MSG-45, RN-169). El resto de la HU es backend y no se espera en el prototipo |
| HU-013 | Ver un mensaje comprensible ante un error inesperado | ◐ Parcial | CMP-02 se declara «marco de las páginas de error» | Página `500` con MSG-40, sin trazas y con salida (OBS-14) |
| HU-014 | Restablecer las cuentas demo | — No aplica | — | Sin interfaz por diseño |

**HU Must que faltan por completo:** HU-005, HU-007, HU-008, HU-009, HU-010B y HU-011 (parte de UI).
**HU Should:** ninguna está representada, como corresponde (`02` §0).

---

## 2. Reglas de negocio

Universo: **169 reglas**, de las cuales **156 son MUST** y 13 SHOULD (RN-044 a RN-047, RN-089 a RN-093, RN-135, RN-136, RN-143, RN-161). Las Should quedan fuera del cálculo.

### 2.1 Reglas reflejadas explícitamente en la UI (19)

| Regla | Cómo se refleja | Pantalla o componente |
|---|---|---|
| RN-001 | Casilla «Declaro ser mayor de 18 años»; sin marcar no se crea la cuenta | P-001 |
| RN-002 | MSG-01 bajo la casilla, `id="AdultDeclaration-error"`, `aria-describedby` | P-001 |
| RN-094 | Errores MSG-34 y MSG-03 en Contraseña; texto de ayuda | P-001 |
| RN-095 | MSG-04 como último paso del orden de validación del correo | P-001 |
| RN-099 | MSG-05 sobre el botón; es el único aviso (D-UI-06) | P-001 |
| RN-148 | Exactamente 4 entradas, sin «Recordarme» | P-001 |
| RN-149 | MSG-32, MSG-34 y MSG-36 a la vez; orden del correo MSG-32 → MSG-37 → MSG-33 → MSG-04 | P-001 |
| RN-150 | MSG-33; `novalidate`, valida el servidor | P-001 |
| RN-151 | MSG-37 | P-001 |
| RN-152 | MSG-35 bajo «Confirmar contraseña» | P-001 |
| RN-153 | Válido ⇒ sesión iniciada y `303` a `/dashboard` | P-001, F-01 |
| RN-116 | Error bajo el campo con icono y texto, `aria-invalid`, `aria-describedby`, CMP-04 | Reglas comunes, CMP-04 |
| RN-168 | Botón con `data-disable-on-submit`, `disabled` y `aria-disabled`; un segundo `submit` se cancela | CMP-10 |
| RN-131 | «Generar resumen del mes» → «Generando resumen…» en una región `aria-live="polite"` | CMP-10 |
| RN-050 | Inicio de sesión inválido, incluida la cuenta bloqueada ⇒ MSG-02 | F-01 |
| RN-154 | Correo o contraseña vacíos ⇒ MSG-32 / MSG-34 | F-01 |
| RN-155 | Válido ⇒ `returnUrl` local o P-003 | F-01 |
| RN-097 | «Cerrar sesión» (`POST /account/logout`) → P-002 | F-01 |
| RN-098 | Sesión expirada → P-002 → P-004 vacío (limitación D-24) | F-01 |

### 2.2 Reglas reflejadas de forma parcial (9)

| Regla | Qué se refleja | Qué falta |
|---|---|---|
| RN-146 | P-001 dice «sin Recordarme» | La regla es del inicio de sesión (P-002, no incluida). Ver OBS-13 |
| RN-096 | Implícita: tras registrarse se entra sin confirmar el correo | — (correcto) |
| RN-145 | Ruta `POST /account/logout` | Rutas `/account/register` y `/account/login` en las fichas |
| RN-060 | Mostrar u ocultar «Descripción» citado como mejora progresiva | Ficha P-005 |
| RN-114 | Diálogo de eliminación citado como mejora progresiva | `<details>` / `<dialog>` con MSG-18 y devolución del foco |
| RN-080 | «El tipo de movimiento siempre se dice en texto» | Fecha, categoría u origen y monto en cada fila |
| RN-068 | «Presupuesto: No configurado» como resultado de F-01 | Ficha P-003 |
| RN-162 | MSG-41 como resultado de F-01 | Ficha P-003 y ficha de la lista |
| RN-164 | CMP-02 sirve de marco a las páginas de error | Página con MSG-40 |

> Balance de las 156 reglas Must: 19 explícitas (2.1) + 9 parciales (2.2) + 63 invisibles por diseño (2.3) + 65 visibles pero no representadas (2.4).

### 2.3 Reglas invisibles por diseño (es correcto que no aparezcan)

Son reglas de backend, infraestructura o datos. No se esperan en un prototipo visual; se verifican con pruebas de integración y unitarias.

| Grupo | Reglas |
|---|---|
| Seguridad de cuenta y transporte | RN-003, RN-004, RN-005, RN-006, RN-048, RN-049, RN-100, RN-101, RN-102, RN-140, RN-144, RN-147, RN-156 |
| Aislamiento y datos | RN-007, RN-008, RN-009, RN-103, RN-137 |
| Seed | RN-138, RN-157, RN-158, RN-159 |
| Almacenamiento y cálculo | RN-010, RN-011, RN-012, RN-019, RN-020, RN-021, RN-023 a RN-027, RN-055, RN-056, RN-066, RN-067, RN-074, RN-075, RN-079, RN-113, RN-115 |
| IA (servidor) | RN-034, RN-035, RN-036, RN-037, RN-038, RN-039, RN-042, RN-043, RN-082 a RN-088, RN-128, RN-129, RN-130, RN-133, RN-134, RN-166 |

> RN-101 es invisible cuando funciona, pero su rechazo (`400`) sí llega al usuario y no tiene pantalla: ver OBS-04.

### 2.4 Reglas visibles en la UI que el prototipo no representa (gap)

**De UX y Avisos (39 de 52):** RN-013, RN-016, RN-017, RN-018, RN-028, RN-029, RN-030, RN-031, RN-032, RN-033, RN-040, RN-041, RN-054, RN-069, RN-070, RN-071, RN-072, RN-073, RN-076, RN-077, RN-078, RN-081, RN-107, RN-108, RN-117, RN-120, RN-121, RN-122, RN-123, RN-124, RN-125, RN-126, RN-127, RN-132, RN-141, RN-142, RN-163, RN-165, RN-169.

**De Registro con mensaje o control en pantalla:** RN-014, RN-015, RN-022, RN-051, RN-052, RN-053, RN-057, RN-058, RN-059, RN-061, RN-062, RN-063, RN-064, RN-065, RN-104, RN-105, RN-106, RN-109, RN-110, RN-111, RN-112, RN-118, RN-119, RN-139, RN-160, RN-167.

Todas dependen de las fichas que faltan (OBS-01).

---

## 3. Flujo de usuario

### 3.1 Tareas principales

| Tarea | HU | ¿Se completa en el prototipo? | Fricción o bloqueo |
|---|---|---|---|
| Crear una cuenta | HU-001 | **Sí**, en un solo envío si los datos son válidos; llega al dashboard con sesión iniciada | Casilla tras un error (OBS-07); contraseñas vacías tras cualquier error (OBS-17); doble envío (OBS-06) |
| Iniciar sesión | HU-002A | Solo a nivel de flujo | P-002 sin ficha (OBS-01); el bloqueo no se distingue (OBS-16, aceptado) |
| Cerrar sesión | HU-002B | No verificable | El punto de entrada está en CMP-01, que no se describe (OBS-02) |
| Primer uso tras registrarse | HU-008, HU-003, HU-007 | No verificable | Dashboard vacío sin ficha ni llamadas a la acción (OBS-15) |
| Registrar un gasto o un ingreso | HU-003, HU-004 | No verificable | P-004 y P-005 sin ficha |
| Corregir o eliminar un movimiento | HU-005, HU-006 | No verificable | Lista y P-007 sin ficha |
| Crear o editar el presupuesto | HU-007 | No verificable | Fichas ausentes |
| Ver saldos y avisos | HU-008, HU-009 | No verificable | P-003 sin ficha |
| Pedir el resumen de IA | HU-010A, HU-010B | No verificable | Solo el estado del botón (CMP-10) |

### 3.2 Callejones sin salida detectados

| # | Situación | Resultado actual (según la especificación) | Observación |
|---|---|---|---|
| CS-01 | La sesión expira y el usuario envía un formulario cuya ruta solo admite `POST` (resumen de IA, eliminar movimiento, cerrar sesión) | Redirección al inicio de sesión con `returnUrl` = ruta solo `POST`; al volver se pide con `GET` ⇒ `404` o `405` sin página en español | OBS-03 |
| CS-02 | Rechazo antiforgery (`400`): registro enviado desde una pestaña vieja tras iniciar sesión en otra, o «Cerrar sesión» con la sesión ya expirada | `400` sin página definida (solo existen la `404` y la `500`) | OBS-04 |
| CS-03 | Un usuario con sesión vuelve con «Atrás» a P-001 o abre `/account/login` | Comportamiento no definido; puede crear una segunda cuenta y cambiar de sesión | OBS-05 |
| CS-04 | Página `404` (P-012) o `500` (MSG-40) sin sesión | Sin enlace de salida: la marca de CMP-02 no es enlace sin sesión; solo queda «Atrás» | OBS-14 |

El camino P-001 ↔ P-002 sí es bidireccional según la especificación («¿Ya tienes cuenta? Inicia sesión» y «Crear una cuenta»), pero el segundo enlace solo puede verificarse cuando exista la ficha de P-002.

---

## 4. Observaciones

| # | Observación | Tipo | Severidad | Acción sugerida | HU afectada |
|---|---|---|---|---|---|
| OBS-01 | El prototipo solo describe P-001, CMP-02, CMP-04 y CMP-10. Las pantallas P-002, P-003, P-004, P-005, P-007 y P-012, y las de lista y presupuesto, se citan en flujos y tablas pero no tienen ficha. 65 reglas Must que se ven en pantalla (sección 2.4) no se pueden contrastar. | gap | **Alta** | Agregar a `09` las fichas de todas las pantallas de las HU Must, con el mismo nivel que P-001 (propósito, campos, acciones, estados, wireframe de 360 px) y su trazabilidad a RN-NNN | HU-002A, HU-002B, HU-003 a HU-011, HU-013 |
| OBS-02 | CMP-01 (layout con sesión, `<nav aria-label="Principal">`, «Cerrar sesión», pie) y CMP-03 (campo) se referencian («footer igual que CMP-01», «Cada campo usa CMP-03», F-01), pero no se describen. No se puede verificar desde dónde se cierra la sesión. | gap | **Alta** | Describir CMP-01 y CMP-03 en `09` | HU-002B (RN-097), HU-001 (RN-116) |
| OBS-03 | Si la sesión expira (RN-098) y el usuario envía un formulario cuya ruta solo admite `POST` (`POST /insights/monthly-summary`, eliminación de un movimiento, `POST /account/logout`), la redirección al inicio de sesión lleva un `returnUrl` que después se pide con `GET` y no existe. F-01 solo documenta P-004, cuya ruta sí tiene `GET`. | gap (callejón) | **Alta** | En F-01 y en la nota de RN-155: si el `returnUrl` apunta a una ruta sin `GET`, redirigir a la pantalla que contiene ese formulario o a `/dashboard`. Criterio nuevo en HU-002B con `FakeTimeProvider` | HU-002B, HU-006, HU-010A |
| OBS-04 | El rechazo antiforgery de RN-101 (`400`) llega al usuario en casos reales (registro desde una pestaña abierta antes de iniciar sesión en otra; «Cerrar sesión» con la sesión expirada, porque el token está ligado a la sesión) y no tiene página. Solo hay páginas para `404` (RN-169) y `500` (RN-164); por defecto ASP.NET Core devuelve un `400` sin contenido, contra la regla común «Ningún texto en inglés visible». | gap (callejón) | **Alta** | Definir en `09` una página `400` en español con CMP-02, un mensaje de catálogo nuevo (MSG-46 propuesto: p. ej. «Tu formulario caducó. Vuelve a intentarlo.») y un enlace para volver. Decidir si «Cerrar sesión» con la sesión expirada redirige a P-002 sin error | HU-001, HU-002B, HU-011 |
| OBS-05 | No se define qué pasa si un usuario con sesión abre `GET /account/register` o `GET /account/login`, por ejemplo con «Atrás» tras registrarse. Podría crear una segunda cuenta y quedar con la sesión de la nueva sin haber cerrado la anterior. | gap (callejón) | Media | Usuario autenticado en `/account/register` o `/account/login` ⇒ `303` a `/dashboard`. Nota en RN-153 y RN-145; criterios nuevos en HU-001 y HU-002A | HU-001, HU-002A |
| OBS-06 | CMP-10 no incluye «Crear cuenta» (P-001), «Iniciar sesión» (P-002) ni los formularios de presupuesto. P-001 propone «Creando cuenta…» en su tabla de estados, pero esa fila no está en CMP-10, y `site.js` solo actúa sobre botones con `data-disable-on-submit`. Con un doble envío, el primero crea la cuenta y el segundo puede mostrar MSG-04 aunque la cuenta se acaba de crear; en presupuesto, el segundo muestra MSG-21. | fricción | Media | Agregar a CMP-10 las filas de P-001, P-002 y los formularios de presupuesto, como mejora progresiva sin regla nueva, o declarar la limitación junto a RN-168 | HU-001, HU-007 |
| OBS-07 | Tras un error, «el servidor vuelve a mostrar lo que el usuario escribió (salvo contraseñas)», pero no se dice si la casilla «Declaro ser mayor de 18 años» conserva su estado. Si vuelve desmarcada, cada reintento (por ejemplo, tras MSG-04) dispara MSG-01. | comprensión | Media | En el estado «Errores» de P-001: «la casilla conserva el valor enviado». Criterio E2E en HU-001 | HU-001 |
| OBS-08 | No se define el orden ni la combinación de errores de Contraseña y Confirmar contraseña: RN-149 solo ordena los del correo. Casos abiertos: contraseña `Ab#1x` con una confirmación distinta (¿MSG-03 y MSG-35 a la vez?); contraseña vacía con confirmación escrita (¿MSG-35?). | comprensión | Media | Nota en RN-149 o RN-152, por ejemplo: Contraseña MSG-34 → MSG-03; Confirmación MSG-36 → MSG-35, y MSG-35 solo si la contraseña no está vacía. Criterios en HU-001 | HU-001 |
| OBS-09 | El texto de ayuda de la contraseña está marcado **[Propuesta]** y no figura en el catálogo de `03` §10, que es la fuente de los textos exactos. Además repite casi literalmente MSG-03. | gap | Baja | Agregarlo a §10 (MSG-47 propuesto) o usar el texto de MSG-03 como ayuda y retirar la marca [Propuesta] | HU-001 |
| OBS-10 | `type="email"` puede modificar el valor antes de que llegue al servidor: en algunos navegadores (Chromium), un dominio con caracteres no ASCII se envía en su forma punycode (`xn--…`), que sí cumple el patrón de RN-150. Así, un correo con «ñ» en el dominio podría aceptarse. Los criterios de HU-001 solo prueban la «ñ» en la parte local (`ñandu@correo.pe`). **Riesgo a verificar.** | gap | Media | Agregar a HU-001 un caso E2E en Chromium con «ñ» en el dominio. Si se acepta, usar `type="text"` con `inputmode="email"` (mantiene el teclado móvil) | HU-001 |
| OBS-11 | La gestión del foco es ambigua: CMP-04 lleva `role="alert" tabindex="-1"` (patrón para enfocar el resumen), mientras las reglas comunes ponen `autofocus` en el primer campo con error. | comprensión | Media | Elegir una: foco en el primer campo (funciona sin JavaScript) y quitar `tabindex="-1"`, o foco en el resumen. Verificarlo con axe y un lector de pantalla | HU-001, HU-003, HU-004, HU-007 |
| OBS-12 | Colisión de identificadores: `09` usa P-NNN para pantallas (P-007 = edición, P-012 = página `404`) y PRINCIPIO-0NN para principios, pero `03` y `06` siguen usando P-0NN para principios. Ejemplos: HU-011 («la impone P-007»), RN-116 («Constitución P-012»), y RN-169, que cita a la vez «Constitución P-001, P-007» y «`09-ui-spec.md` P-012». | comprensión | Media | Renombrar las pantallas (p. ej., PAN-NNN) o actualizar `03` y `06` a PRINCIPIO-0NN; registrar la convención en `/docs/README.md` | Todas (trazabilidad) |
| OBS-13 | P-001 y HU-001 citan RN-146 («sin Recordarme»), pero esa regla trata del formulario de inicio de sesión. En el registro, la ausencia de «Recordarme» ya la garantiza RN-148. RN-146 queda sin verificar en su pantalla real (P-002). | comprensión | Baja | Quitar RN-146 de la ficha P-001 y asignarla a P-002 | HU-001, HU-002A |
| OBS-14 | La página `404` (P-012, MSG-45) y la página `500` (MSG-40) usan CMP-02, donde la marca «ExpenseFlow AI» no es enlace sin sesión. No se define ninguna acción de salida; solo queda «Atrás». | gap (callejón) | Media | Agregar a ambas páginas un enlace «Ir al inicio» (`/dashboard` con sesión, P-002 sin sesión) | HU-011, HU-013 |
| OBS-15 | Tras el registro, el usuario llega a un dashboard vacío («Presupuesto: No configurado», MSG-41, MSG-42). El siguiente paso (el enlace a «Crear presupuesto» de RN-070, el acceso a «Nuevo gasto») no está prototipado. No se puede evaluar el primer uso ni preparar las sesiones de los KPI U2 y U3. | gap | **Alta** | Prototipar P-003 en el estado de `demo4` / cuenta nueva, con sus llamadas a la acción, como parte de OBS-01 | HU-008, HU-003, HU-007 |
| OBS-16 | MSG-02 no distingue la cuenta bloqueada: el usuario no sabe que debe esperar 5 minutos. Es una decisión aprobada (PR-06, RN-050), pero puede generar reintentos con frustración. | fricción | Baja | Sin cambios en la especificación. Observarlo en las sesiones de usabilidad | HU-002A |
| OBS-17 | Las contraseñas se vacían tras cualquier error, incluso uno que solo afecta al correo (MSG-33, MSG-04). En Android obliga a reescribir dos contraseñas complejas. Es una práctica de seguridad habitual. | fricción | Baja | Mantenerlo y medirlo en las sesiones de usabilidad. La ayuda de la contraseña ya está siempre visible, lo que reduce el retrabajo | HU-001 |

---

## 5. Decisiones recomendadas antes de implementar

| # | Decisión | Bloquea | Documento a cambiar | Origen |
|---|---|---|---|---|
| DEC-01 | Completar las fichas de P-002, P-003 (incluido el estado de cuenta nueva), P-004, P-005, P-007, P-012, la lista de movimientos y los formularios de presupuesto, y describir CMP-01 y CMP-03 | **Sí** — todas las HU Must salvo HU-001 | `09-ui-spec.md` | OBS-01, OBS-02, OBS-15 |
| DEC-02 | Definir el destino tras iniciar sesión cuando el `returnUrl` apunta a una ruta sin `GET` | **Sí** — HU-002B | `09` F-01, nota de RN-155 en `03`, criterio en HU-002B | OBS-03 |
| DEC-03 | Definir la presentación del `400` antiforgery en MVC (página en español + MSG-46) y el comportamiento de «Cerrar sesión» con la sesión expirada | **Sí** — HU-002B, HU-011 | `09`, `03` §10, nota de RN-101 | OBS-04 |
| DEC-04 | Usuario autenticado en `/account/register` o `/account/login` ⇒ `303` a `/dashboard` | No (P-001 puede empezar, pero la regla debe existir antes de cerrar HU-001) | Notas de RN-153 y RN-145, criterios en HU-001 y HU-002A | OBS-05 |
| DEC-05 | Agregar a CMP-10 las filas de «Crear cuenta», «Iniciar sesión» y presupuesto, o declarar la limitación | No | `09` CMP-10 | OBS-06 |
| DEC-06 | La casilla de mayoría de edad conserva su valor tras un error | No | `09` P-001 | OBS-07 |
| DEC-07 | Fijar el orden de validación de Contraseña y Confirmar contraseña | No | Nota de RN-149 o RN-152, criterios en HU-001 | OBS-08 |
| DEC-08 | Catalogar el texto de ayuda de la contraseña (MSG-47) o reutilizar MSG-03 | No | `03` §10, `09` P-001 | OBS-09 |
| DEC-09 | Verificar `type="email"` con «ñ» en el dominio; si falla, cambiar a `type="text"` + `inputmode="email"` | No | `09` P-001, criterio en HU-001 | OBS-10 |
| DEC-10 | Elegir una única estrategia de foco tras un error | No | `09` CMP-04 y reglas comunes | OBS-11 |
| DEC-11 | Resolver la colisión entre P-NNN (pantallas) y P-0NN (principios) | No, pero conviene antes de que el código y los `Trait` usen los IDs | `09`, o bien `03` y `06`; `/docs/README.md` | OBS-12 |
| DEC-12 | Enlace de salida en las páginas `404` y `500`; corregir la traza de RN-146 | No | `09` P-012, página de error, P-001 | OBS-13, OBS-14 |

> Las decisiones DEC-01 a DEC-03 se registran como bloqueantes: sin ellas, la Definition of Ready no se cumple para HU-002B y las HU de E2 a E5. P-001 (HU-001) puede pasar a implementación en paralelo si se aprueban DEC-04 a DEC-09.

---

## 6. Score de completitud

| Métrica | Cálculo | Resultado |
|---|---|---|
| HU Must cubiertas por completo | 1 / 14 | **7.1 %** |
| HU Must representadas (completas + parciales) | (1 + 7) / 14 | 57.1 % |
| Cobertura ponderada de HU Must | (1 × 1 + 7 × 0.5) / 14 | 32.1 % |
| Reglas Must reflejadas explícitamente | 19 / 156 | **12.2 %** |
| Reglas Must reflejadas (explícitas + parciales) | (19 + 9) / 156 | 17.9 % |
| Reglas Must de UX y Avisos reflejadas | (7 + 6) / 52 | 25.0 % |
| Reglas de HU-001 observables en la UI reflejadas en P-001 | 13 / 13 (RN-003 y RN-004 son de backend) | 100 % |

**Lectura:** la calidad de lo que está especificado es alta (P-001 cubre el 100 % de su HU); el problema es de **alcance** del prototipo, no de calidad. Con DEC-01 resuelto, la siguiente revisión debería medir la cobertura sobre las 14 HU y las 52 reglas de UX y Avisos.

---

## Trazabilidad de esta revisión

| Elemento revisado | HU | Observaciones |
|---|---|---|
| P-001 | HU-001 | OBS-05 a OBS-11, OBS-13, OBS-17 |
| CMP-02 | HU-001, HU-002A, HU-013 | OBS-14 |
| CMP-04 | HU-001, HU-003 | OBS-11 |
| CMP-10 | HU-001, HU-003, HU-004, HU-006, HU-007, HU-010A | OBS-06 |
| Reglas comunes | Todas | OBS-04, OBS-11, OBS-12 |
| F-01 | HU-001, HU-002A, HU-002B | OBS-02 a OBS-05, OBS-16 |
| Pantallas citadas sin ficha | HU-002A a HU-013 | OBS-01, OBS-15 |

---

# Revisión R2 — P-002 Iniciar sesión

| Campo | Valor |
|---|---|
| Fecha | 2026-11-10 |
| Objeto revisado | P-002, CMP-02, «Reglas comunes a todas las pantallas» y F-01 (`09-ui-spec.md` v1.2) |
| Insumos | Los mismos de R1: `06` v1.3 y `03` v1.9, sin cambios |

## R2 · Resumen ejecutivo

P-002 cumple los criterios de HU-002A que se ven en pantalla. Tiene solo dos campos, sin «Recordarme» ni «¿Olvidaste tu contraseña?». MSG-02 es un mensaje único en un bloque sobre el formulario, sin señalar qué campo falló, que es lo que exige RN-050. Los campos vacíos se rechazan sin contar como intento (RN-154), y el `returnUrl` va en un campo oculto que valida el servidor (RN-155).

Los problemas están en **lo que pasa alrededor del inicio de sesión**:
- **Dirección de retorno (`returnUrl`).** No se dice que se conserve tras un intento fallido (OBS-18). Tampoco qué pasa si apunta a una ruta que solo se envía (OBS-03, abierta desde R1).
- **Mensajes de error.** P-002 usa su propio bloque para MSG-02 y no el resumen de errores común (CMP-04), pero no lo declara como excepción (OBS-20).
- **Sesión expirada.** El usuario llega a esta pantalla sin ninguna explicación (OBS-22).

| Indicador | Solo R2 | Acumulado R1 + R2 |
|---|---|---|
| HU Must cubiertas por completo | 1 de 14 (7.1 %) | **2 de 14 (14.3 %)** |
| HU Must representadas (completas + parciales) | 8 de 14 (57.1 %) | 8 de 14 (57.1 %) |
| Cobertura ponderada (completa = 1, parcial = 0.5) | 32.1 % | 35.7 % |
| Reglas Must reflejadas explícitamente | 9 de 156 (5.8 %) | **21 de 156 (13.5 %)** |
| Reglas Must reflejadas (explícitas + parciales) | 18 de 156 (11.5 %) | 28 de 156 (17.9 %) |
| Reglas de HU-002A observables en la UI reflejadas | 5 de 5 (100 %) | — |
| Observaciones nuevas | 7 (Alta 0 · Media 4 · Baja 3) | 24 en total |

## R2 · 1. Cobertura de HU

| HU | Cobertura en R2 | Dónde aparece | Qué falta |
|---|---|---|---|
| HU-002A | ✔ **Completa** | P-002, F-01 | Nada observable en la UI. RN-006, RN-048, RN-049 y RN-144 son de backend |
| HU-002B | ◐ Parcial | P-002 («llegada tras cerrar sesión o tras expirar»), F-01 | CMP-01 con «Cerrar sesión» (OBS-02); conservar el `returnUrl` tras un error (OBS-18) |
| HU-001 | ◐ Parcial | Enlace a P-001, F-01 | Ficha P-001 (cubierta en R1) |
| HU-003 | ◐ Parcial | F-01 (sesión expirada en P-004), reglas comunes (RN-168) | Ficha P-004 |
| HU-004, HU-006 | ◐ Parcial | Reglas comunes («Descripción», diálogo de eliminación) | Fichas P-005 y P-007 |
| HU-010A | ◐ Parcial | Reglas comunes (RN-131) | Ficha del resumen. CMP-10 no se incluyó en este paquete |
| HU-013 | ◐ Parcial | CMP-02 como marco de las páginas de error | Página con MSG-40 |
| HU-005, HU-007, HU-008, HU-009, HU-010B, HU-011 | ✘ Ausentes | — | Igual que en R1 |
| HU-014 | — No aplica | — | Sin interfaz |

**Acumulado:** HU-001 y HU-002A ya tienen una ficha completa. HU-002B sigue parcial hasta que se describa CMP-01.

## R2 · 2. Reglas de negocio

### Reflejadas explícitamente (9)

| Regla | Cómo se refleja en P-002 |
|---|---|
| RN-050 | MSG-02 único en un bloque `role="alert"` sobre el formulario, para credenciales incorrectas, correo inexistente o cuenta bloqueada |
| RN-145 | Sin enlaces a «¿Olvidaste tu contraseña?» ni a otras páginas de cuenta. La parte del `404` es de backend |
| RN-146 | «Solo 2 [campos], sin “Recordarme”» |
| RN-154 | MSG-32 / MSG-34 bajo el campo, sin intentar el inicio de sesión |
| RN-155 | `returnUrl` en un campo oculto que el servidor valida |
| RN-116 | Error bajo el campo con `aria-invalid` y `aria-describedby` (reglas comunes) |
| RN-153 | F-01: registro válido ⇒ P-003 con sesión iniciada |
| RN-097 | F-01: «Cerrar sesión» ⇒ P-002 |
| RN-098 | F-01 y estado «llegada tras expirar la sesión» |

### Reflejadas de forma parcial (9)

RN-148 (F-01 «completa 4 campos»), RN-168 y RN-131 (solo se mencionan en «Mejora progresiva»; CMP-10 no está en este paquete), RN-060, RN-114, RN-080, RN-068, RN-162 y RN-164: igual que en R1.

### Invisibles por diseño (citadas por P-002)

RN-006 (sin contraseñas en el log), RN-048 (bloqueo de 5 minutos: el usuario solo ve MSG-02), RN-049 (reinicio del contador), RN-144 (incremento del contador). Las cubren las pruebas de integración con `FakeTimeProvider` de HU-002A. RN-147 (cookie no persistente) es invisible y P-002 no la cita (OBS-24).

### Visibles pero no representadas

Las mismas 65 reglas de la sección 2.4 de R1. P-002 no agrega ni quita ninguna, porque las reglas de inicio de sesión no figuraban en esa lista.

## R2 · 3. Flujo de usuario

| Tarea | ¿Se completa? | Fricción o bloqueo |
|---|---|---|
| Iniciar sesión con credenciales correctas | **Sí**, en un envío | El teclado de Android puede autocorregir el correo, y MSG-02 no dice qué campo falló (OBS-23) |
| Corregir un error de credenciales | Sí | Se conserva el correo y el foco va a «Contraseña». Bien resuelto |
| Volver a la tarea tras una sesión expirada | **No garantizado** | Si el usuario se equivoca una vez, el `returnUrl` puede perderse (OBS-18); llega sin explicación (OBS-22) |
| Pasar a «Crear cuenta» y volver | Sí | El texto del enlace no coincide con F-01 (OBS-19) |
| Recuperar una contraseña olvidada | **No**, por alcance (W17) | Callejón aceptado pero no declarado (OBS-24) |
| Cuenta bloqueada | Sí, tras 5 minutos | MSG-02 no explica la espera (OBS-16, aceptado) |

**Callejones sin salida:**

| # | Situación | Estado |
|---|---|---|
| CS-01 | `returnUrl` hacia una ruta que solo admite `POST` | **Abierto** (OBS-03). P-002 confirma que el `returnUrl` va en un campo oculto, pero solo se valida que sea local |
| CS-02 | Rechazo antiforgery `400`. Caso nuevo en P-002: el usuario abre «Iniciar sesión» en dos pestañas, inicia sesión en una y envía la otra. El token anónimo ya no es válido | **Abierto** (OBS-04) |
| CS-03 | Usuario con sesión que abre `/account/login` | **Abierto** (OBS-05). P-002 no define el estado «ya autenticado» |
| CS-05 (nuevo) | Contraseña olvidada en una cuenta que no es demo | Sin salida por alcance (W17); se registra como limitación (OBS-24) |

## R2 · 4. Observaciones

| # | Observación | Tipo | Severidad | Acción sugerida | HU afectada |
|---|---|---|---|---|---|
| OBS-18 | El `returnUrl` va en un campo oculto, pero los estados de P-002 no dicen que se conserve al volver a mostrar el formulario tras MSG-02, MSG-32 o MSG-34. La regla común solo conserva «lo que el usuario escribió», y el `returnUrl` no lo escribió el usuario. Si se pierde, basta un error de tipeo para que el usuario termine en `/dashboard` y no en «Nuevo gasto», lo que hace fallar un criterio de HU-002B. | gap | Media | En los estados de P-002: «el campo oculto `returnUrl` se conserva en cada nueva presentación del formulario». Criterio en HU-002B: sesión expirada → un intento fallido → inicio correcto → vuelve a P-004 | HU-002B, HU-002A · RN-155, RN-098 |
| OBS-19 | El texto del enlace a P-001 no coincide: P-002 dice «¿No tienes cuenta? Crea una» y F-01 dice «Crear una cuenta». `09` declara literales los textos entre comillas latinas, así que las pruebas E2E no sabrían cuál buscar. | comprensión | Baja | Unificar el texto en P-002 y F-01 | HU-001, HU-002A |
| OBS-20 | P-002 contradice las reglas comunes sin declararlo como excepción. Las reglas comunes dicen que, ante errores, se muestra CMP-04 arriba («Revisa los datos ingresados» y enlaces a los campos) y el primer campo con error recibe `autofocus`. P-002 usa en cambio un bloque propio con MSG-02 y pone el foco en «Contraseña», que no está marcada con error. Tampoco se dice si CMP-04 aparece cuando hay campos vacíos. | comprensión | Media | Declarar en P-002 la excepción para MSG-02 (bloque propio, sin CMP-04, foco en «Contraseña», sin `aria-invalid` en ningún campo, que es lo que exige RN-050) y decir explícitamente si con campos vacíos se usa CMP-04 | HU-002A · RN-050, RN-154, RN-116 |
| OBS-21 | El estado «campos vacíos (MSG-32 / MSG-34 bajo el campo)» no dice si, con ambos campos vacíos, los dos mensajes aparecen a la vez. P-001 sí lo dice («a la vez»). | comprensión | Baja | Escribir «a la vez, cada uno bajo su campo», como en P-001. Agregar el caso con ambos vacíos al criterio de RN-154 en HU-002A | HU-002A · RN-154 |
| OBS-22 | Al llegar a P-002 porque la sesión expiró, no se muestra ningún mensaje. Se cita D-UI-07, pero esa decisión trata de las confirmaciones tras guardar o eliminar, no de la expiración. El usuario acaba de perder lo que escribió (limitación D-24) y no sabe por qué está en el inicio de sesión. | fricción | Media | Decisión del PO: (a) mostrar un texto informativo nuevo en §10 cuando la llegada trae `returnUrl` (p. ej., «Tu sesión terminó. Inicia sesión para continuar.»), que no revela nada de la cuenta; o (b) aceptar la limitación citando D-24, no D-UI-07 | HU-002B · RN-098 |
| OBS-23 | El campo «Correo» de P-002 no lleva `spellcheck="false"` ni `inputmode="email"`, que sí tiene P-001. En Android, el autocorrector puede cambiar la dirección, y como MSG-02 no dice qué campo falló, el usuario no puede diagnosticar el error. Tampoco se dice si el servidor quita los espacios exteriores del correo al iniciar sesión, como sí lo hace el registro (RN-150, RN-151). | fricción | Media | Igualar los atributos del correo con P-001. Agregar una nota en RN-154 o RN-155: el correo se compara sin espacios exteriores. Criterio en HU-002A con `" demo1@expenseflow.test "` | HU-002A · RN-050, RN-150 |
| OBS-24 | Faltan dos cosas en P-002. Primero, no tiene «¿Olvidaste tu contraseña?», que es correcto (RN-145, W17), pero no se declara como limitación: un participante que olvide la contraseña de una cuenta creada por él no tiene salida. Segundo, la trazabilidad está incompleta: P-002 se presenta como destino de HU-002B pero no cita RN-097 ni RN-098, ni RN-147, que acompaña a la ausencia de «Recordarme». | gap | Baja | Agregar en P-002 la nota «Limitación aceptada (W17): sin recuperación de contraseña». Completar la fila «Reglas» con RN-097, RN-098 y RN-147 | HU-002A, HU-002B · RN-145, RN-147 |

### Estado de las observaciones de R1 relacionadas con P-002

| Observación | Estado tras R2 | Comentario |
|---|---|---|
| OBS-01 | Parcialmente resuelta | P-002 ya tiene ficha. Faltan P-003, P-004, P-005, P-007, P-012, la lista y el presupuesto |
| OBS-02 | Abierta | CMP-01 sigue sin describirse; F-01 lo usa para «Cerrar sesión» |
| OBS-03 | Abierta | P-002 confirma el campo oculto `returnUrl`, pero no trata las rutas que solo admiten `POST` |
| OBS-04 | Abierta, con un caso nuevo | Se agrega el caso de dos pestañas de inicio de sesión (CS-02) |
| OBS-05 | Abierta | P-002 no define qué pasa con un usuario ya autenticado |
| OBS-11 | Parcialmente informada | P-002 fija el foco para MSG-02. La ambigüedad general de CMP-04 sigue abierta (ver OBS-20) |
| OBS-13 | Resuelta en P-002 | RN-146 ya se verifica en su pantalla. Solo queda quitarla de la ficha P-001 |
| OBS-16 | Sin cambios (aceptada) | P-002 aplica MSG-02 también al bloqueo, como pide RN-050 |

## R2 · 5. Decisiones recomendadas

| # | Decisión | Bloquea | Documento a cambiar | Origen |
|---|---|---|---|---|
| DEC-02, DEC-03, DEC-04 | Se reafirman las decisiones de R1 sobre `returnUrl` hacia rutas solo `POST`, la página `400` y el usuario autenticado en `/account/login` | **Sí** — HU-002B | `09` P-002 y F-01; `03` notas de RN-101, RN-145, RN-155 | OBS-03, OBS-04, OBS-05 |
| DEC-13 | Conservar el `returnUrl` oculto en cada nueva presentación del formulario de P-002 | **Sí** — un criterio de HU-002B | `09` P-002 (estados), criterio en HU-002B | OBS-18 |
| DEC-14 | Declarar en P-002 la excepción a CMP-04 para MSG-02 y fijar la presentación de los campos vacíos (a la vez, con o sin CMP-04) | No | `09` P-002 y reglas comunes | OBS-20, OBS-21 |
| DEC-15 | Unificar el texto del enlace a P-001 | No | `09` P-002 o F-01 | OBS-19 |
| DEC-16 | El PO decide si la llegada por expiración muestra un texto informativo o se acepta citando D-24 | No | `09` P-002; `03` §10 si se elige (a) | OBS-22 |
| DEC-17 | Atributos del correo iguales a P-001 y comparación sin espacios exteriores | No | `09` P-002; nota en `03` RN-154 o RN-155; criterio en HU-002A | OBS-23 |
| DEC-18 | Declarar la limitación W17 y completar la trazabilidad de P-002 (RN-097, RN-098, RN-147) | No | `09` P-002 | OBS-24 |

> P-002 puede pasar a implementación junto con P-001 si se aprueban DEC-13 a DEC-17. HU-002B no se puede cerrar hasta resolver DEC-01 (CMP-01) y DEC-02 a DEC-04.

## R2 · 6. Score de completitud

| Métrica | Cálculo | Solo R2 | Acumulado R1 + R2 |
|---|---|---|---|
| HU Must cubiertas por completo | completas / 14 | 1 / 14 = 7.1 % | **2 / 14 = 14.3 %** (HU-001, HU-002A) |
| HU Must representadas | (completas + parciales) / 14 | 8 / 14 = 57.1 % | 8 / 14 = 57.1 % |
| Cobertura ponderada | (completas + 0.5 × parciales) / 14 | 4.5 / 14 = 32.1 % | 5 / 14 = 35.7 % |
| Reglas Must explícitas | explícitas / 156 | 9 / 156 = 5.8 % | **21 / 156 = 13.5 %** (R1 + RN-145 y RN-146) |
| Reglas Must explícitas + parciales | / 156 | 18 / 156 = 11.5 % | 28 / 156 = 17.9 % |
| Reglas Must de UX y Avisos | / 52 | 11 / 52 = 21.2 % (2 explícitas: RN-116, RN-154; 9 parciales) | 13 / 52 = 25.0 % (sin cambios: RN-145 y RN-146 son de Seguridad) |
| Reglas de la HU revisada observables en la UI | / observables | HU-002A: 5 / 5 = 100 % | HU-001 13/13 · HU-002A 5/5 |

**Lectura:** el prototipo avanza pantalla por pantalla con buena calidad (100 % en cada HU revisada). Aun así, la cobertura del MVP sigue baja (14.3 % de HU Must). La siguiente revisión debería incluir CMP-01 y P-003, que cierran HU-002B y desbloquean HU-008 y HU-009, la mitad visible del producto.

---

# Revisión R3 — P-003 a P-015

| Campo | Valor |
|---|---|
| Fecha | 2026-11-10 |
| Objeto revisado | Fichas P-003 a P-015 de `09-ui-spec.md` v1.2. Componentes CMP-01, CMP-02, CMP-04, CMP-05, CMP-06, CMP-07, CMP-08, CMP-09 y CMP-10. Flujos F-02 a F-07 y flujos transversales |
| Insumos | Los mismos de R1 y R2: `06` v1.3 y `03` v1.9, sin cambios |
| Método | Las 13 entregas repiten el mismo contexto y los mismos bloques comunes. Se revisó una sola vez cada bloque distinto (las copias anidadas de P-004, P-005, P-006, P-008 y P-009 son idénticas a sus fichas principales) |

## R3 · Resumen ejecutivo

Con esta entrega, **las 14 HU Must con interfaz quedan representadas por completo**. Cada criterio de aceptación observable en pantalla tiene un elemento, un estado o un mensaje que lo materializa. Los estados por escenario del seed (`demo1` a `demo6`) están definidos en P-003 y P-006, lo que facilita mucho las pruebas E2E. Las 5 HU Should aparecen como borradores bloqueados (QA-10, QA-12), como corresponde.

La entrega **resuelve** cuatro observaciones de R1:
- **OBS-01:** faltaban fichas de pantallas.
- **OBS-02:** faltaba CMP-01, el menú con «Cerrar sesión».
- **OBS-14:** las páginas de error no tenían salida.
- **OBS-15:** el primer uso tras registrarse no estaba prototipado.

**Confirma** los dos callejones sin salida de severidad alta:
- **OBS-03:** el resumen de IA, la eliminación y el cierre de sesión solo se envían (`POST`), así que no hay a dónde volver después de iniciar sesión.
- **OBS-04:** «Cerrar sesión» es un formulario con token antiforgery, así que con la sesión expirada devuelve un `400` que no tiene página.

Los hallazgos nuevos son de severidad media o baja. Los más relevantes:
- **OBS-25:** los textos literales de las HU no coinciden con la estructura de la pantalla, lo que rompería las pruebas E2E.
- **OBS-26:** un posible bucle con MSG-22 al cambiar de mes.
- **OBS-28:** sin mensaje de confirmación, a veces el usuario no puede ver el resultado de lo que guardó.

| Indicador | Acumulado R1 + R2 + R3 |
|---|---|
| HU Must representadas por completo | **14 de 14 (100 %)** |
| HU Must listas para implementar (sin decisión bloqueante abierta) | **11 de 14 (78.6 %)** |
| HU Should representadas (borrador) | 5 de 5 (HU-012, HU-015, HU-016, HU-017, HU-018) |
| Reglas Must reflejadas explícitamente en la UI | **92 de 156 (59.0 %)** |
| Reglas Must observables en la UI que están reflejadas | **92 de 93 (98.9 %)**; la restante (RN-096) es implícita |
| Reglas Must de UX y Avisos reflejadas | 52 de 52 (100 %) |
| Observaciones nuevas | 11 (Alta 0 · Media 6 · Baja 5) |

## R3 · 1. Cobertura de HU

| HU | Cobertura acumulada | Pantallas y componentes | Observaciones abiertas que la afectan |
|---|---|---|---|
| HU-001 | ✔ Completa | P-001 (R1) | OBS-05 a OBS-10, OBS-13 |
| HU-002A | ✔ Completa | P-002 (R2), P-012 (`/Identity/Account/*` → `404`) | OBS-19 a OBS-23 |
| HU-002B | ✔ Completa, **bloqueada** | CMP-01 (formulario «Cerrar sesión»), P-002, P-004 (estado «Sesión expirada») | **OBS-03, OBS-04, OBS-18**, OBS-22 |
| HU-003 | ✔ Completa | P-004, CMP-05, CMP-06, CMP-10, F-02 | OBS-25, OBS-28 |
| HU-004 | ✔ Completa | P-005, CMP-05, CMP-06, CMP-10, F-04 | OBS-25, OBS-28 |
| HU-005 | ✔ Completa | P-006, CMP-07, F-03 | OBS-29, OBS-30, OBS-32 |
| HU-006 | ✔ Completa, **bloqueada** | P-007, P-008 (`<dialog>` y `<details>`), F-03 | **OBS-03** (eliminar con la sesión expirada), OBS-28 a OBS-30, OBS-34 |
| HU-007 | ✔ Completa | P-009, P-010, F-05, enlace «Presupuesto» de CMP-01 | OBS-26, OBS-27, OBS-06 |
| HU-008 | ✔ Completa | P-003 (variantes por escenario), CMP-07, F-06 | OBS-25 |
| HU-009 | ✔ Completa | P-003, CMP-08, CMP-09 | OBS-25 |
| HU-010A | ✔ Completa, **bloqueada** | P-003 (sección IA, estados), CMP-10, F-07 | **OBS-03** (resumen con la sesión expirada), OBS-33 |
| HU-010B | ✔ Completa | P-003 (estados MSG-23 y MSG-24) | — |
| HU-011 | ✔ Completa (UI) | P-012, P-007 (recurso ajeno ⇒ P-012) | OBS-04 (el `400` sin página) |
| HU-013 | ✔ Completa | P-011 | — |
| HU-014 | — No aplica | — | — |

**HU Should (borradores, no forman parte del MVP):** HU-012 (P-013), HU-015 (P-014), HU-016 (filtros plegables en P-006), HU-017 (P-015) y HU-018 (botón «Explicar mi consumo» dentro de CMP-09). Todas están marcadas como bloqueadas por QA-10 o QA-12, en línea con `06`.

## R3 · 2. Reglas de negocio

### Reflejadas en esta entrega

Las **65 reglas** que R1 listó como «visibles pero no representadas» (sección 2.4) quedan reflejadas:

| Bloque | Reglas | Dónde |
|---|---|---|
| Formulario de gasto | RN-014, RN-015, RN-016, RN-017, RN-018, RN-051, RN-052, RN-053, RN-054, RN-057, RN-104, RN-105, RN-108, RN-109 | P-004, CMP-05, CMP-06 |
| Formulario de ingreso | RN-058, RN-059, RN-061 (sin `maxlength`), RN-106, RN-107, RN-110 (D-UI-04) | P-005 |
| Edición y eliminación | RN-111, RN-112 («Tipo: Gasto», no editable), RN-160 | P-007, P-008 |
| Lista | RN-117, RN-125 | P-006, CMP-07 |
| Presupuesto | RN-022, RN-062, RN-063, RN-064 (MSG-21 con enlace a P-010), RN-065, RN-118, RN-119, RN-139, RN-167 | P-009, P-010 |
| Dashboard y avisos | RN-013, RN-028 a RN-033, RN-069 a RN-073, RN-076 a RN-078, RN-081, RN-120 a RN-124, RN-126, RN-127, RN-141, RN-142, RN-163 | P-003, CMP-07, CMP-08, CMP-09 |
| IA | RN-040, RN-041, RN-132, RN-165 | P-003, F-07 |
| Transversal | RN-169 | P-012 |

Además, las 6 reglas que R1 marcó como parciales pasan a explícitas: RN-060, RN-068, RN-080, RN-114, RN-162 y RN-164.

### Reglas de backend que también tienen expresión visible

No cambian de grupo en el cómputo (siguen en el 2.3 de R1), pero ahora tienen un elemento comprobable en pantalla:
- **RN-027:** «la vista no calcula nada», con una prueba sin JavaScript.
- **RN-037 y RN-165:** al recargar no se llama a Ollama.
- **RN-038 y RN-039:** MSG-23 aparece con el resto del dashboard intacto.
- **RN-128:** el texto del modelo se codifica con Razor, nunca con `Html.Raw`.
- **RN-079:** el orden de la lista.
- **RN-008:** un recurso ajeno lleva a P-012.
- **RN-026:** la ayuda «Los ingresos no lo modifican» de P-009.

### Balance acumulado de las 156 reglas Must

| Grupo | Reglas | % |
|---|---|---|
| Reflejadas explícitamente en la UI | 92 | 59.0 % |
| Reflejadas de forma implícita (RN-096) | 1 | 0.6 % |
| Invisibles por diseño (backend, seed, IA en el servidor; sección 2.3 de R1) | 63 | 40.4 % |
| Visibles pero no representadas | **0** | 0 % |

**Reglas Should:** las 13 aparecen en los borradores (RN-044 en P-014; RN-045 en P-006; RN-046 en P-015; RN-047 en P-003; RN-089 a RN-093, RN-135, RN-136, RN-143 y RN-161 en P-013).

## R3 · 3. Flujo de usuario

### Tareas principales

| Tarea | HU | ¿Se completa? | Fricción |
|---|---|---|---|
| Primer uso: registro → dashboard vacío → crear presupuesto → primer gasto | HU-001, HU-007, HU-003 | **Sí**. P-003 en el estado de `demo4` ofrece «Crear presupuesto», «Nuevo gasto» y «Nuevo ingreso» | — |
| Registrar un gasto en ≤ 15 s (U2) | HU-003 | **Sí**. Camino óptimo de 3 acciones: monto, categoría, «Guardar gasto» | Si la fecha es antigua, el gasto no aparece entre los 5 últimos y no hay confirmación (OBS-28) |
| Registrar un ingreso | HU-004 | **Sí** | Igual que el gasto (OBS-28) |
| Encontrar y corregir un movimiento antiguo | HU-005, HU-006 | **Sí**, por paginación | Si se cambia la fecha, la fila puede pasar a otra página sin aviso (OBS-28) |
| Eliminar un movimiento | HU-006 | **Sí**, con confirmación accesible, con y sin JavaScript | Al borrar el único de la última página aparece «No hay más movimientos» (OBS-29) |
| Crear o editar el presupuesto | HU-007 | **Sí**. MSG-21 enlaza a «Editar presupuesto» | Abrir la pantalla que no corresponde (OBS-27); cambio de mes (OBS-26) |
| Ver saldos y avisos | HU-008, HU-009 | **Sí**, en los 6 escenarios del seed | Literales frente a la estructura del DOM (OBS-25) |
| Pedir el resumen de IA | HU-010A, HU-010B | **Sí**, con PRG y estados de fallo y de IA deshabilitada | Con la sesión expirada no hay retorno válido (OBS-03) |
| Cerrar sesión | HU-002B | **Sí** con la sesión vigente | Con la sesión expirada: `400` sin página (OBS-04) |
| Recuperarse de un error 404 o 500 | HU-011, HU-013 | **Sí**, «Volver al inicio» | — |

### Callejones sin salida

| # | Situación | Estado tras R3 |
|---|---|---|
| CS-01 | `returnUrl` hacia una ruta que solo admite `POST` | **Abierto y confirmado.** Hay tres rutas así en el prototipo: `POST /insights/monthly-summary` (P-003), `POST /transactions/{id}/delete` (P-008) y `POST /account/logout` (CMP-01) |
| CS-02 | `400` antiforgery sin página | **Abierto y confirmado.** CMP-01 implementa «Cerrar sesión» como `<form method="post">` con token. Con la sesión expirada, el token pertenece al usuario anterior y la validación falla |
| CS-03 | Usuario con sesión en `/account/login` o `/account/register` | Abierto (OBS-05) |
| CS-04 | Páginas 404 y 500 sin salida | **Resuelto.** P-011 y P-012 tienen «Volver al inicio» |
| CS-05 | Contraseña olvidada | Limitación W17 (OBS-24) |
| CS-06 (nuevo) | Al cambiar de mes, P-009 o P-010 pueden repetir MSG-22 en cada reenvío | Abierto (OBS-26) |

## R3 · 4. Observaciones

| # | Observación | Tipo | Severidad | Acción sugerida | HU afectada |
|---|---|---|---|---|---|
| OBS-25 | Los textos literales de las HU no existen como texto en el DOM descrito. (a) CMP-07 se declara «literal de las HU» («08/11/2026 · Transporte · Gasto · S/ 3.50»), pero pone el monto en otra línea, sin « · », y «Gasto» en una insignia. (b) En P-003, presupuesto, porcentaje y restante van en un `<dl>` sin dos puntos, mientras HU-007, HU-008 y HU-009 verifican «Presupuesto: S/ 600.00», «Porcentaje consumido: No disponible» y «Restante del presupuesto: S/ 105.00». Si los dos puntos se agregan con CSS (`::after`), no forman parte del texto que leen Playwright ni los lectores de pantalla. | comprensión | Media | Elegir una de dos: (1) que el DOM contenga el literal (los dos puntos en el `<dt>`, y « · » como texto visualmente oculto en CMP-07); o (2) reescribir esos criterios de `06` para que verifiquen cada campo por separado. Registrar la decisión en CMP-07 y P-003 | HU-003, HU-004, HU-007, HU-008, HU-009 · RN-068, RN-069, RN-071, RN-080 |
| OBS-26 | Cuando se muestra MSG-22 en P-009 (el mes cambió entre abrir y enviar), no se dice qué mes lleva el campo oculto `month` al volver a mostrar el formulario. Si conserva el mes viejo, cada reenvío vuelve a dar MSG-22: un bucle. En P-010, tras MSG-22 el presupuesto que se edita ya no es del mes en curso y no hay enlace a «Crear presupuesto». | gap (callejón) | Media | Tras MSG-22, volver a mostrar P-009 con `month` y el rótulo del **mes nuevo**, conservando el monto escrito. En P-010, mostrar MSG-22 con un enlace a P-009. Criterio en HU-007 con `FakeTimeProvider` | HU-007 · RN-118, RN-167 |
| OBS-27 | P-009 está pensada para quien no tiene presupuesto y P-010 para quien sí, pero no se define qué responde `GET /budgets/create` si ya existe uno, ni `GET /budgets/edit` si no existe. Puede pasar con «Atrás», con una pestaña vieja o con un enlace guardado. En el primer caso el usuario llena el formulario y recién al enviar recibe MSG-21; en el segundo, el comportamiento queda abierto (¿`404`?). | gap | Media | `GET /budgets/create` con presupuesto ⇒ `303` a `/budgets/edit`; `GET /budgets/edit` sin presupuesto ⇒ `303` a `/budgets/create`. Agregarlo a F-05 y a HU-007 | HU-007 · RN-064, RN-118 |
| OBS-28 | Sin mensajes de confirmación (D-UI-07), a veces el resultado no se ve. (a) Un gasto o ingreso con una fecha anterior a los 5 movimientos más recientes no aparece en P-003 (orden de RN-079), pero F-02 promete «con el gasto entre los últimos movimientos». (b) Al editar la fecha en P-007 se vuelve a la página *n*, donde la fila puede ya no estar. El usuario puede creer que no se guardó y repetir el registro (duplicado). | fricción | Media | Decisión del PO: (a) excepción a D-UI-07, con una confirmación breve `role="status"` tras guardar (nuevo MSG en §10); o (b) mantener D-UI-07 y corregir F-02 («el gasto aparece entre los últimos movimientos si su fecha es de las más recientes»). Observarlo en las sesiones U2 | HU-003, HU-004, HU-006 · RN-078, RN-079 |
| OBS-29 | Si se elimina el único movimiento de la última página (*n* > 1), P-006 aplica el estado «Página posterior a la última» y muestra MSG-31 («No hay más movimientos») aunque existan movimientos en páginas anteriores. | fricción | Media | Tras eliminar, si la página *n* quedó vacía y *n* > 1, redirigir a la página *n* − 1 (`303`). Ajustar D-UI-03 y P-006 | HU-005, HU-006 · RN-117 |
| OBS-30 | La vuelta al origen es irregular. «Nuevo gasto» y «Nuevo ingreso» se ofrecen también en P-006, pero al guardar o cancelar siempre se vuelve a `/dashboard`. «Cancelar» de P-007 dice «P-006», sin precisar si conserva el `page` de origen como «Guardar cambios». | fricción | Baja | «Cancelar» de P-007 ⇒ `/transactions?page=n`. Para P-004 y P-005 desde P-006, decidir si vuelven a la lista (p. ej., con un campo oculto `returnTo` local) o aceptar el dashboard como destino único | HU-005, HU-006 · D-UI-03 |
| OBS-31 | Hay excepciones a CMP-04 dispersas y no declaradas en las reglas comunes. P-002 muestra MSG-02 en un bloque propio (OBS-20). P-009 muestra MSG-21 y MSG-22 en un bloque `role="alert"` que no es CMP-04 ni está bajo un campo. Las reglas comunes dicen «Si hay errores, el formulario muestra arriba CMP-04». | comprensión | Baja | Agregar a las reglas comunes: «Errores que no corresponden a un campo (MSG-02, MSG-21, MSG-22) se muestran en un bloque `role="alert"` propio, sin CMP-04». Así se cierran también OBS-20 y OBS-31 juntas | HU-002A, HU-007 · RN-050, RN-064, RN-116, RN-167 |
| OBS-32 | Siguen sin ficha dos componentes referenciados: CMP-03 (campo con etiqueta, ayuda y error; lo usan todos los formularios) y CMP-12 (paginación de P-006, que se menciona en el wireframe). | gap | Media | Describir CMP-03 y CMP-12 en `09` | HU-001, HU-003, HU-004, HU-005, HU-007 · RN-116, RN-117 |
| OBS-33 | Errores de trazabilidad. (a) P-008 cita RN-113 (cambio de fecha a otro mes), que no aplica a la eliminación; corresponden RN-114, RN-115 y RN-020. (b) P-003 aplica la limitación D-22 (doble envío sin JavaScript) al botón de IA, pero D-22 y RN-168 cubren los formularios de movimientos y RN-131 no declara la limitación. | comprensión | Baja | Corregir la fila «Reglas» de P-008. Agregar a RN-131 la nota «sin JavaScript, el doble envío es posible (limitación aceptada, extensión de D-22)» o citar la fuente correcta en P-003 | HU-006, HU-010A · RN-113, RN-114, RN-115, RN-131 |
| OBS-34 | En P-007, un movimiento ya fuera de la ventana de 12 meses solo puede eliminarse (RN-109), pero el usuario lo descubre recién al pulsar «Guardar cambios» (MSG-17). | mejora | Baja | Al cargar P-007 con un movimiento fuera de la ventana, mostrar MSG-17 desde el inicio, como aviso del formulario | HU-006 · RN-109 |
| OBS-35 | (Should) P-014 y P-015 reutilizan CMP-08 tal como está. Su `aria-label` es fijo («Porcentaje consumido del presupuesto»), así que habría varias barras con el mismo nombre accesible. Además, su color «danger» al llegar al 100 % no tiene sentido para una meta de ahorro cumplida (HU-017 deja abierto si el avance supera 100 %). | mejora | Baja | Al especificar S1 y S4, parametrizar en CMP-08 el `aria-label` y la escala de color, o crear un componente propio para metas | HU-015, HU-017 · RN-044, RN-046, RN-141 |

## R3 · 5. Decisiones recomendadas

| # | Decisión | Bloquea | Documento a cambiar | Origen |
|---|---|---|---|---|
| DEC-01 | ~~Completar las fichas de todas las pantallas Must~~ | **Resuelta en R3** (quedan CMP-03 y CMP-12, en DEC-23) | — | OBS-01, OBS-02, OBS-15 |
| DEC-02 | Destino tras iniciar sesión cuando el `returnUrl` apunta a una ruta solo `POST` (P-003 IA, P-008, CMP-01) | **Sí** — HU-002B, HU-006, HU-010A | `09` F-01; nota de RN-155 en `03` | OBS-03 |
| DEC-03 | Página `400` en español, y comportamiento de «Cerrar sesión» con la sesión expirada (p. ej., `POST /account/logout` sin validación antiforgery para usuarios anónimos ⇒ `303` a P-002) | **Sí** — HU-002B | `09` CMP-01 y página nueva; `03` §10, nota de RN-101 | OBS-04 |
| DEC-13 | Conservar el `returnUrl` en cada nueva presentación de P-002 | **Sí** — HU-002B | `09` P-002 | OBS-18 |
| DEC-19 | Literales de las HU frente al DOM: el DOM contiene el literal, o los criterios verifican por campo | No, pero debe decidirse antes de escribir los E2E de HU-003, HU-004, HU-007, HU-008 y HU-009 | `09` CMP-07 y P-003, o `06` | OBS-25 |
| DEC-20 | Tras MSG-22, volver a mostrar con el mes nuevo (P-009) y enlazar a P-009 (P-010) | No | `09` P-009, P-010, F-05; criterio en HU-007 | OBS-26 |
| DEC-21 | Redirecciones cruzadas entre `GET /budgets/create` y `GET /budgets/edit` | No | `09` F-05; criterio en HU-007 | OBS-27 |
| DEC-22 | Confirmación tras guardar: excepción a D-UI-07, o corregir F-02 y aceptarlo | No (decisión del PO) | `09` D-UI-07, F-02; `03` §10 si se agrega un mensaje | OBS-28 |
| DEC-23 | Describir CMP-03 y CMP-12 | No | `09` | OBS-32 |
| DEC-24 | Tras eliminar, si la página quedó vacía y *n* > 1, ir a la página *n* − 1; «Cancelar» de P-007 conserva `page` | No | `09` P-006, P-007, D-UI-03 | OBS-29, OBS-30 |
| DEC-25 | Regla común para errores que no son de campo (cierra OBS-20 y OBS-31) | No | `09` reglas comunes | OBS-20, OBS-31 |
| DEC-26 | Corregir la trazabilidad de P-008 y la limitación sin JavaScript de RN-131; aviso inicial de MSG-17 en P-007 | No | `09` P-007, P-008, P-003; nota en RN-131 | OBS-33, OBS-34 |
| DEC-27 | (Should) Parametrizar CMP-08 antes de especificar S1 y S4 | No | `09` CMP-08 | OBS-35 |

> **Orden sugerido de implementación:** las 11 HU sin bloqueo pueden empezar ya, en el orden de `06` (HU-014 → HU-001 → HU-002A → HU-011 → HU-003 …). Antes de cerrar HU-002B, HU-006 y HU-010A deben aprobarse DEC-02, DEC-03 y DEC-13. DEC-19 conviene aprobarla antes de escribir los E2E.

## R3 · 6. Score de completitud

| Métrica | Cálculo | R1 | R1 + R2 | **R1 + R2 + R3** |
|---|---|---|---|---|
| HU Must representadas por completo | completas / 14 | 7.1 % | 14.3 % | **100 %** |
| HU Must listas para implementar | sin decisión bloqueante / 14 | — | — | **78.6 %** (11 / 14) |
| Reglas Must reflejadas explícitamente | / 156 | 12.2 % | 13.5 % | **59.0 %** (92) |
| Reglas Must reflejadas (explícitas + implícitas) | / 156 | 17.9 % | 17.9 % | 59.6 % (93) |
| Reglas Must observables en la UI reflejadas | / 93 | — | — | **98.9 %** (100 % con RN-096) |
| Reglas Must de UX y Avisos | / 52 | 25.0 % | 25.0 % | **100 %** |
| Reglas Should en borradores | / 13 | 0 % | 0 % | 100 % |

**Lectura:** el prototipo ya cubre todo el alcance funcional del MVP. El 40 % de las reglas Must que no se ve en pantalla es backend por diseño (seguridad, cálculo, seed, IA en el servidor) y se verifica con pruebas de integración y unitarias. Lo que falta para pasar a implementación son **decisiones de comportamiento**, no pantallas. Tres son bloqueantes (DEC-02, DEC-03, DEC-13); el resto se puede resolver en paralelo.

---

# Estado consolidado de las observaciones

| Observación | Severidad | Estado tras R3 | Comentario |
|---|---|---|---|
| OBS-01 | Alta | ✔ Resuelta | Todas las pantallas Must tienen ficha. CMP-03 y CMP-12 pasan a OBS-32 |
| OBS-02 | Alta | ✔ Resuelta | CMP-01 descrito. CMP-03 pasa a OBS-32 |
| OBS-03 | Alta | ✘ **Abierta, confirmada** | Tres rutas solo `POST` en el prototipo |
| OBS-04 | Alta | ✘ **Abierta, confirmada** | «Cerrar sesión» de CMP-01 es un formulario con token |
| OBS-05 | Media | ✘ Abierta | — |
| OBS-06 | Media | ✘ Abierta | CMP-10 sigue sin P-001, P-002, P-009 ni P-010 |
| OBS-07 a OBS-10 | Media / Baja | ✘ Abiertas | P-001 no se volvió a entregar |
| OBS-11 | Media | ✘ Abierta | CMP-04 sin cambios |
| OBS-12 | Media | ✘ Abierta | La colisión persiste (P-007 = pantalla y principio; P-008 = pantalla) |
| OBS-13 | Baja | ✘ Abierta | Solo falta corregir la ficha P-001 |
| OBS-14 | Media | ✔ Resuelta | P-011 y P-012 con «Volver al inicio» |
| OBS-15 | Alta | ✔ Resuelta | P-003 en el estado de `demo4`, con llamadas a la acción |
| OBS-16, OBS-17 | Baja | Aceptadas | Observar en las sesiones de usabilidad |
| OBS-18 a OBS-24 | Media / Baja | ✘ Abiertas | P-002 no se volvió a entregar. OBS-20 se cierra junto con OBS-31 (DEC-25) |
| OBS-25 a OBS-35 | Media / Baja | ✘ Abiertas (nuevas) | — |

**Totales:** 35 observaciones · 4 resueltas · 2 aceptadas · **29 abiertas**, de las cuales 2 son de severidad alta (OBS-03, OBS-04).
