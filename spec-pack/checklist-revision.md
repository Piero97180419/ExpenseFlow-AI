# Checklist de revisión humana — ExpenseFlow AI

| Campo | Valor |
|---|---|
| Documento | checklist-revision.md |
| Versión | v1.0 |
| Fecha | 2026-11-10 |
| Estado | **Aprobado** |
| Aprobado por | Piero Carbajal |
| Fecha de aprobación | 2026-11-10 |
| PR de aprobación | PR que fusiona el Spec Pack v1.0 en `main` (ADR-005 §1) |
| Fuentes | `00-constitucion.md` (PRINCIPIO-010, PRINCIPIO-011), `03-reglas-de-negocio.md` v1.0, ADR-001 a ADR-007 v1.0 |

> Obligatoria en todo PR que modifique cálculos financieros, seguridad, autenticación, esquema de base de datos, integración con IA, dependencias, o la constitución, los ADR o las especificaciones de `/docs` (PRINCIPIO-011, ADR-005). El revisor marca cada punto aplicable y escribe «N/A» en los que no apliquen. El PR no se fusiona con casillas sin marcar.
>
> Esta checklist se copia también en `.github/pull_request_template.md`, para que GitHub la cargue en cada PR nuevo.
>
> **Limitación conocida:** en un equipo de una persona, el revisor es también el autor o el supervisor del agente. La revisión es válida para PRINCIPIO-011, pero no es independiente (ADR-005 §6).

## 0. General (todo PR de este tipo)

- [ ] El CI está en verde y ninguna prueba, analizador o check se desactivó, omitió o debilitó (PRINCIPIO-010).
- [ ] Las pruebas E2E de los flujos principales están en verde en el CI (PRINCIPIO-011, ADR-001 §3).
- [ ] La rama sigue el formato `<tipo>/<descripción>`, con un tipo de Conventional Commits (ADR-005 §2), y llega a `main` por PR.
- [ ] Los mensajes de commit siguen Conventional Commits en inglés (PRINCIPIO-001).
- [ ] Las reglas de negocio afectadas (RN-xxx) están citadas en la descripción del PR.
- [ ] Ninguna regla implementada depende de un documento en estado `Propuesto` (ADR-005 §1).
- [ ] No se usaron datos reales en pruebas, seeds ni capturas (PRINCIPIO-006).

## 1. Cálculos financieros

- [ ] Todos los importes usan `decimal` / `DECIMAL(18,2)`; no aparece `float` ni `double` (PRINCIPIO-003, RN-010, RN-011).
- [ ] Todo redondeo usa `MoneyRounding.Mode` y solo se aplica a resultados de cálculo (RN-025, RN-055).
- [ ] La fecha y hora se obtienen solo de `TimeProvider`; las pruebas usan `FakeTimeProvider` (RN-137).
- [ ] Ninguna columna de fecha tiene un valor por defecto calculado en SQL Server, e Identity usa el `TimeProvider` registrado (RN-137, ADR-004 §5).
- [ ] Cada prueba de una regla de categoría Cálculo y prioridad MUST lleva `[Trait("Rule", "RN-NNN")]` y cubre su ejemplo y su contraejemplo; el paso de verificación del CI está en verde (ADR-001 §3).
- [ ] Hay pruebas para cada caso que afecte el cambio:
  - [ ] presupuesto ausente (RN-066, RN-121, RN-122);
  - [ ] montos en el límite: 0.01 y 100,000.00 (RN-051, RN-052, RN-062, RN-063);
  - [ ] entrada de monto: vacío, solo espacios, `3.50`, `3,50`, `12.345`, `1,250.50`, `.50`, `1234567`, `١٢` (RN-053, RN-104);
  - [ ] umbrales: 79.99, 80.00, 99.99, 100.00 y el redondeo de 99.995 a 100.00 (RN-028, RN-029, RN-120);
  - [ ] gastos iguales y superiores al presupuesto (RN-071 a RN-073);
  - [ ] fechas: hoy, futura, primer día y día anterior de la ventana de 12 meses, y medianoche en Lima (RN-017, RN-057, RN-109);
  - [ ] cambio de fecha de un movimiento a otro mes (RN-113);
  - [ ] «Crear presupuesto» enviado después del cambio de mes (RN-167).
- [ ] Las pruebas E2E que modifican datos usan una cuenta `e2e-<GUID>@expenseflow.test`, nunca una cuenta demo (`03` sección 11.4).
- [ ] Ninguna cifra oficial se calcula en JavaScript ni la produce la IA (RN-027, RN-043).
- [ ] Las fechas se interpretan en `America/Lima` (RN-017, RN-022, RN-023, RN-057, RN-109, RN-118).

## 2. Seguridad y autenticación

- [ ] Toda consulta financiera filtra por el `UserId` del contexto de autenticación (RN-007).
- [ ] Ningún `UserId` se toma de datos enviados por el cliente; los DTO de entrada no lo incluyen (RN-103).
- [ ] Existe una prueba de integración con dos usuarios que verifica `404` sobre movimientos **y** presupuestos ajenos (RN-008).
- [ ] Los endpoints `/api/*` responden `401` sin sesión, aunque también falte el token antiforgery (RN-101, RN-102).
- [ ] Cada endpoint nuevo o modificado cumple el contrato de ADR-004 §3 (entradas, códigos y objetos de respuesta), y la sección está actualizada.
- [ ] Las operaciones que modifican datos validan el token antiforgery, en MVC y en la API (RN-101).
- [ ] Las páginas y respuestas autenticadas envían `Cache-Control: no-store` (RN-140).
- [ ] Se respetan la política de contraseñas, el correo único, el bloqueo y la expiración de sesión (RN-048 a RN-050, RN-094 a RN-098).
- [ ] El inicio de sesión usa `lockoutOnFailure: true` (RN-144), y los envíos con campos vacíos no cuentan como intento fallido (RN-154).
- [ ] Solo existen las páginas de registro, inicio y cierre de sesión; no se usa `Microsoft.AspNetCore.Identity.UI` (RN-145).
- [ ] No hay opción «Recordarme» y la cookie de autenticación no es persistente (RN-146, RN-147).
- [ ] El registro tiene sus 4 campos y valida vacíos, longitud, formato del correo y confirmación de contraseña (RN-148 a RN-152), con pruebas para `josé@correo.pe`, `ñandu@correo.pe`, `ana+1@gmail.com`, `ana@gmail` y `ana@mi_dominio.com`.
- [ ] Tras registrarse, el usuario entra al dashboard (RN-153); tras iniciar sesión, solo se redirige a direcciones locales (RN-155).
- [ ] Kestrel acepta solo TLS 1.2 y 1.3 (RN-156).
- [ ] Los errores no controlados muestran MSG-40 sin detalles técnicos fuera de `Development` (RN-164).
- [ ] HTTPS en todos los entornos y HSTS fuera de `Development` (RN-005, RN-100).
- [ ] Ningún log registra contraseñas ni secretos (RN-006, PRINCIPIO-008), y los logs de IA solo contienen los datos de RN-133.
- [ ] Ningún secreto, cadena de conexión con credenciales ni `.env` entra al repositorio. Las credenciales demo son la única excepción (PRINCIPIO-008, ADR-005).
- [ ] No hay criptografía propia (RN-003).
- [ ] Los datos financieros no se usan para entrenar modelos ni se envían a servicios externos de entrenamiento (RN-009; no es verificable por prueba).
- [ ] Se conoce y acepta la limitación de RN-097: una copia de la cookie capturada antes del cierre de sesión sigue siendo válida hasta expirar (DEC-48).
- [ ] Un usuario autenticado que abre el registro o el inicio de sesión va al dashboard (RN-173), y un `returnUrl` hacia una ruta que solo admite `POST` lleva a la página del formulario (RN-174).
- [ ] `POST /account/logout` es `[AllowAnonymous]` y, sin sesión, redirige al inicio de sesión sin validar el token (RN-177); la página de inicio de sesión muestra MSG-48 solo cuando la sesión expiró (RN-171).
- [ ] La respuesta `401` de la API lleva el `ProblemDetails` estándar (RN-102, DEC-88).
- [ ] Las páginas `404` (MSG-45) y `400` antiforgery (MSG-46) están en español, sin trazas, y la `404` es idéntica para recursos ajenos e inexistentes (RN-169, RN-170).

## 3. Esquema de base de datos

- [ ] El cambio se hace con una migración **nueva**; ninguna migración ya presente en `main` se modificó ni eliminó (PRINCIPIO-010).
- [ ] La migración no elimina tablas, columnas ni datos. Si el cambio lo necesitaba, la migración la escribió y la aplicará un humano, con un ADR aprobado (ADR-005 §3).
- [ ] No se agregó `Database.Migrate()` al código de producción (ADR-004 §4.2).
- [ ] Las restricciones de unicidad necesarias están en la base de datos, no solo en el código (p. ej., RN-064, RN-095).
- [ ] Nombres de tablas y columnas en inglés (PRINCIPIO-001).

## 4. Integración con IA

- [ ] La URL de Ollama solo admite los hosts de ADR-002, comparados por tipo de host y valor de IP (RN-087).
- [ ] La solicitud cumple el contrato de ADR-003 §1 (`POST /api/generate`, `stream: false`, `think: false`).
- [ ] El payload solo contiene los campos de RN-034, sin datos de identificación (RN-035).
- [ ] La salida del modelo se muestra como texto codificado, nunca como HTML ni Markdown (RN-128).
- [ ] Un texto con alguna cifra no válida se trata como fallo, muestra MSG-23 y registra `type=InvalidFigure` (RN-166).
- [ ] El resumen se muestra una sola vez mediante Post/Redirect/Get con TempData y no se guarda en la base de datos (RN-165).
- [ ] Si cambió el prompt, el payload, el cliente, el verificador de cifras o el modelo: se creó una versión nueva de las instrucciones cuando correspondía (RN-129) y se adjunta `ai-eval-report.md` con **10 de 10** escenarios aprobados según la definición de «cifra» de ADR-003, con la versión de Ollama y el digest del modelo (RN-130, ADR-003).
- [ ] Hay pruebas con respuestas simuladas para éxito, timeout, error HTTP, modelo no descargado, respuesta vacía y cifra no válida (RN-038, RN-082, RN-166).
- [ ] El botón de resumen no envía solicitudes duplicadas (RN-131) y muestra MSG-24 con la IA deshabilitada (RN-132).
- [ ] El log de un fallo de IA solo contiene los datos de RN-133.
- [ ] El dashboard funciona con Ollama detenido (RN-039, RN-088).

## 5. Dependencias

- [ ] Todo paquete, herramienta, imagen o acción figura en ADR-001 (lista del stack de producto o de herramientas) o en un ADR aprobado (PRINCIPIO-002). Si lo agregó un agente, ya figuraba en esas listas (ADR-005 §7).
- [ ] La versión de cada paquete está fijada en `Directory.Packages.props`.
- [ ] Las imágenes de contenedor se fijan por etiqueta exacta y las acciones de GitHub Actions por SHA de commit (ADR-001).
- [ ] `dotnet list package --vulnerable` no reporta vulnerabilidades críticas ni altas.
- [ ] La licencia es gratuita y permite el uso académico: MIT, Apache 2.0, BSD o una licencia analizada en el ADR del componente (ADR-005 §5).

## 6. Accesibilidad y UI (si el PR cambia la UI)

- [ ] Las pruebas axe en Playwright pasan sin errores AA (PRINCIPIO-012).
- [ ] Cada campo nuevo tiene `<label>` y sus errores se muestran en texto con `aria-describedby` (RN-116).
- [ ] Ningún dato se comunica solo con color, y el contraste del texto es de al menos 4.5:1 (PRINCIPIO-012, RN-030, RN-080, RN-123).
- [ ] El flujo se completa solo con teclado y se probó en vista móvil.
- [ ] Importes, porcentajes y fechas siguen RN-081, RN-123, RN-124 y RN-125.
- [ ] Los estados vacíos muestran MSG-41 y MSG-42 (RN-162, RN-163).
- [ ] Tras un error, el foco va al primer campo con error; los errores que no son de un campo (MSG-02, MSG-21, MSG-22) van en un bloque con `role="alert"` (DEC-78, DEC-80).
- [ ] Los guardados y eliminaciones muestran una sola vez su confirmación MSG-49 a MSG-53 con `role="status"` (RN-172).
- [ ] Los botones de envío se deshabilitan mientras se envía el formulario (RN-131, RN-168, DEC-74).
- [ ] Las vistas contienen como texto los literales que verifican las HU (DEC-83).
- [ ] Los textos de la UI están en español (es-PE) y coinciden con la sección 10 de `03-reglas-de-negocio.md`.

## 7. Constitución, ADR y especificaciones (si el PR cambia `/docs`)

- [ ] Todo cambio a la constitución tiene un ADR aprobado que lo justifica (PRINCIPIO-010).
- [ ] Cada ADR o documento modificado actualiza su campo Estado y su versión (ADR-005 §1), y el índice `/docs/README.md`.
- [ ] Una decisión nueva se registra como DEC-NN en `03` §9, y una pregunta que queda abierta como PQ-NN en `05`.
- [ ] Al pasar un documento a `Aprobado`, constan el aprobador, la fecha y el número de este PR (ADR-005 §1).
- [ ] Las referencias cruzadas (RN-NNN, MSG-NN, DEC-NN, PQ-NN, ADR-NNN, T-NNN, P-NNN, CMP-NN) apuntan a elementos que existen y siguen la convención de `04` §4.

---

**Revisor:** ____________________ · **Fecha:** __________ · **PR:** #____
