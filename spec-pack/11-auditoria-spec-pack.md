# 11 — Auditoría integral del Spec Pack — ExpenseFlow AI

| Campo | Valor |
|---|---|
| Documento | 11-auditoria-spec-pack.md |
| Fase | Cierre (BMAD — Agente Spec Finalizer) |
| Versión | v1.0 |
| Fecha | 2026-11-10 |
| Estado | **Aprobado** |
| Aprobado por | Piero Carbajal |
| Fecha de aprobación | 2026-11-10 |
| Objeto | Spec Pack v1.0: `00` a `10`, ADR-001 a ADR-007, `checklist-revision.md` y `README.md` (todos v1.0, Aprobados) |
| Pregunta que responde | ¿Puede un agente de IA usar este paquete como contexto para implementar el MVP sin inventar ni decidir por su cuenta? |
| Método | Lectura completa del pack. Trazabilidad, conteos, existencia de cada ID citado, restos de numeración antigua, cobertura del glosario y grafo del backlog verificados con scripts sobre el texto, no con estimaciones |

> **Severidad:** 🔴 **Crítico** = un agente no puede implementar sin inventar · 🟠 **Alto** = implementaría algo incorrecto · 🟡 **Medio** = ambigüedad que produce implementaciones distintas · 🟢 **Bajo** = observación sin efecto en la implementación del MVP.
> Los hallazgos se numeran **AF-NN**.

---

## Resumen ejecutivo

**Veredicto: el Spec Pack v1.0 está listo para que un agente de IA implemente el MVP.**

- **0 hallazgos críticos, altos o medios.** Quedan 4 hallazgos bajos, todos con su tratamiento ya planificado (§10).
- Todos los documentos están en estado **Aprobado** (v1.0, Piero Carbajal, 2026-11-10) y el índice `README.md` los lista con su versión.
- **Trazabilidad completa:** 178 de 178 reglas tienen HU; las 20 HU con interfaz tienen pantalla; las 16 pantallas y las 20 HU tienen tareas; las 165 reglas MUST tienen tarea.
- **IDs consistentes:** de los 16 prefijos verificados, ningún ID citado apunta a un elemento inexistente y no quedan restos de numeraciones anteriores.

| Criterio | Puntaje |
|---|:-:|
| Operatividad | **4.7 / 5** |
| Utilidad | **4.6 / 5** |
| Orden | **4.8 / 5** |
| **General** | **4.7 / 5** |

---

## 0. Inventario del pack

| # | Archivo | Versión | Estado | ¿Presente y correcto? |
|---|---|---|---|---|
| 00 | Constitución | v1.0 | Aprobado | ✔ 12 principios con ejemplo de cumplimiento y de violación |
| 01 | Project Brief | v1.0 | Aprobado | ✔ |
| 02 | Alcance | v1.0 | Aprobado | ✔ M1–M6, S1–S5, C1–C4, W1–W20, flujos F-01 a F-07 |
| 03 | Reglas de negocio | v1.0 | Aprobado | ✔ 178 reglas EARS (165 MUST, 13 SHOULD), DEC-01 a DEC-90, MSG-01 a MSG-53 |
| 04 | Glosario | v1.0 | Aprobado | ✔ 130 términos y 15 a evitar; convención de IDs |
| 05 | Preguntas abiertas | v1.0 | Aprobado | ✔ PQ-01 a PQ-09, ninguna bloqueante |
| 06 | Historias de usuario | v1.0 | Aprobado | ✔ 18 HU (+ 2 contenedoras divididas en A/B), 193 criterios |
| 07 | Arquitectura | v1.0 | Aprobado | ✔ 7 secciones, ARQ-01 a ARQ-09 |
| 08 | Backlog | v1.0 | Aprobado | ✔ 56 tareas con DAG verificado |
| 09 | UI Spec | v1.0 | Aprobado | ✔ 16 pantallas, 12 componentes con ficha |
| 10 | Validation log | v1.0 | Aprobado | ✔ 34 observaciones con decisión |
| — | ADR-001 a ADR-007 | v1.0 | Aprobado | ✔ Contexto, decisión, alternativas y consecuencias en los 7 |
| — | `checklist-revision.md`, `README.md` | v1.0 | Aprobado | ✔ |
| — | `10-anexo-reporte-reviewer.md` | — | Fuente | ✔ Reporte original del Agente Prototype Reviewer; conserva la numeración de la revisión y se excluye de las verificaciones de IDs |

---

## 1. Trazabilidad completa

| Cadena | Resultado | Detalle |
|---|---|---|
| Regla → HU | ✔ **178 / 178** | Verificado sobre `06` completo; RN-009 se verifica con el checklist (`06` HU-011) |
| HU → pantalla | ✔ **20 / 20** con interfaz | HU-014 sin pantalla por diseño (`09` §1.3); HU-002 y HU-010 son contenedoras |
| Pantalla → tareas | ✔ **16 / 16** | Matriz de `08` §6; P-013 a P-015 con las tareas Should T-053, T-055 y T-056 |
| HU → tareas | ✔ **20 / 20** | Must 15 / 15; Should 5 / 5 (`08` §5) |
| Regla MUST → tarea | ✔ **165 / 165** | Matriz de `08` §7 |
| Reglas MUST de Cálculo → `Trait` | ✔ | Criterio en T-004, T-019, T-031, T-037; verificación en el CI con T-041 (ADR-001 §3) |
| Observaciones de `10` → documento o tarea | ✔ **34 / 34** | 30 aplicadas en v1.0 con su ubicación; 5 en el backlog (T-051 a T-054) |
| Preguntas de `05` → tarea o responsable | ✔ **9 / 9** | PQ-07 → T-055, PQ-08 → T-056, PQ-09 → T-013; PQ-01 a PQ-06 con responsable y momento |

---

## 2. Consistencia de IDs

Verificado por script en los 21 documentos normativos: todo ID citado existe en su documento de definición.

| Familia | Definidos | Referencias rotas |
|---|---|:-:|
| PRINCIPIO-0NN | 12 | 0 |
| RN-NNN · MSG-NN · DEC-NN | 178 · 53 · 90 | 0 |
| PQ-NN | 9 | 0 |
| HU-NNN | 22 (incluye A/B) | 0 |
| ARQ-NN · DP-N · NS-N · RC-N · RA-NN | 9 · 9 · 8 · 8 · 5 | 0 |
| ADR-NNN | 7 (+ ADR-008 reservado) | 0 |
| T-NNN | 56 | 0 |
| P-NNN · CMP-NN | 16 · 12 | 0 |
| V-NN | 34 | 0 |

- **Sin colisiones de prefijos:** cada prefijo tiene un único significado (`04` §4, DEC-79). Los principios usan PRINCIPIO-0NN y P-NNN es solo para pantallas.
- **Sin restos de numeraciones anteriores:** no aparecen versiones v1.x, PA-, CF-, PR-, D-NN, D-UI-, QA-, AU- ni CP-.
- **Nombres de código alineados** entre `04` y `07`: `ExpenseCategory`, `DemoSeeder`, `AdultDeclaration` (campo del formulario, no se persiste).

---

## 3. Cobertura de la constitución

| Principio | ¿Se respeta? | Evidencia |
|---|:-:|---|
| PRINCIPIO-001 Idioma | ✔ | Código en inglés (`07` §3); UI y mensajes en es-PE; páginas `400`, `404` y `500` en español |
| PRINCIPIO-002 Stack | ✔ | Paquetes cerrados en ADR-001; `09` sin Sass, iconos, fuentes ni gráficos |
| PRINCIPIO-003 Montos | ✔ | `decimal(18,2)`, CHECK, `MoneyRounding.Mode`, sin columna de moneda |
| PRINCIPIO-004 Cálculos | ✔ | `DashboardCalculator` puro; la vista y el JS no calculan; `FigureVerifier` en ejecución |
| PRINCIPIO-005 IA local | ✔ | Hosts de loopback (ADR-002); payload de 9 campos sin identificadores |
| PRINCIPIO-006 Datos ficticios | ✔ | Seed `demo1`–`demo6`; pruebas con `e2e-<GUID>` |
| PRINCIPIO-007 Aislamiento | ✔ | `ICurrentUser`, filtro global, `404` idéntico para ajeno e inexistente, precedencia de ADR-004 §3.1 |
| PRINCIPIO-008 Seguridad | ✔ | HTTPS, TLS ≥ 1.2, HSTS, lockout, antiforgery global con página `400`, registro de Identity correcto (`07` ARQ-02), `401` con `ProblemDetails` |
| PRINCIPIO-009 Alcance | ✔ | Casilla obligatoria; W1–W20 sin HU |
| PRINCIPIO-010 Agentes | ✔ | Migraciones aditivas; seed solo por humano o CI; dependencias según ADR-005 §7 |
| PRINCIPIO-011 Calidad | ✔ | `Trait` en las reglas MUST de Cálculo verificado en el CI (ADR-001 §3, T-041); E2E bloqueantes (T-040 a T-047) |
| PRINCIPIO-012 Accesibilidad y docs | ✔ | WCAG 2.2 AA en `09`; README por módulo (T-048); índice `README.md`; ADR por decisión |

Cada principio se cita en entre 6 y 12 documentos además de la constitución. Ningún documento la contradice.

---

## 4. Glosario

- `04` v1.0 tiene 130 términos en tres grupos: dominio (§1), interfaz (§2) y técnicos y de proceso (§3). Incluye además 15 términos a evitar y la convención de identificadores (§4).
- **Cobertura verificada por script:** los 68 términos técnicos y de interfaz que se usan en el pack están definidos. Entre ellos: PRG, TempData, `ProblemDetails`, `returnUrl`, cookie de marca, Kestrel, HSTS, `mkcert`, Playwright, axe, `Trait`, BMAD, EARS, INVEST, Definition of Ready, ruta crítica, loopback y punycode.

---

## 5. Preguntas abiertas

| Pregunta | ¿Bloquea la implementación? | Tratamiento |
|---|:-:|---|
| PQ-01 a PQ-04 (consentimiento, participantes, entrevistas, metas F1–F5) | No | Validación con usuarios; T-054 |
| PQ-05 (S2 a Must) y PQ-06 (RN-108 «Efectivo») | No | Dependen de PQ-03; procedimiento de cierre en `05` |
| PQ-07 y PQ-08 (especificación de los Should) | No | T-055 y T-056, después del MVP |
| PQ-09 (`TimeProvider` en Identity) | No | Prueba de concepto T-013, al inicio de la fase 1 |

Las decisiones que el pack necesitaba ya están tomadas y registradas (DEC-01 a DEC-90), incluidas las cuatro del PO del 2026-11-10 (V-08, V-12, V-14 y V-17).

---

## 6. Calidad del Documento de Arquitectura (`07` v1.0)

| # | Sección | Completa | Comentario |
|---|---|:-:|---|
| 1 | Overview del sistema | ✔ | Descripción, diagramas textual y Mermaid, secuencia de la IA, DP-1 a DP-9 |
| 2 | Componentes principales | ✔ | Estructura, ARQ-01 a ARQ-09 con HU; registro de Identity con cookies y `401` con `ProblemDetails` |
| 3 | Modelo de datos | ✔ | Entidades, CHECK, índices, ER, NS-1 a NS-8 |
| 4 | API Contracts | ✔ | Convenciones (incluido `405`), endpoints, DTO, rutas MVC con pantalla y redirecciones |
| 5 | ADR | ✔ | Resumen de los 7 ADR con contexto, decisión, alternativas y consecuencias |
| 6 | Restricciones técnicas | ✔ | Alineación por principio, RC-1 a RC-8, entornos |
| (7) | Riesgos y verificación | ✔ | RA-01 a RA-05 con mitigación y tarea |

**Coherencia de los ADR:** los siete tienen la misma cabecera de aprobación y las cuatro secciones obligatorias, con referencias cruzadas que resuelven. ADR-001 §3 y ADR-005 §7 incorporan la verificación de `Trait` y las dependencias que puede proponer un agente, con las que la constitución es coherente.

---

## 7. Calidad del backlog (`08` v1.0)

| Criterio | Resultado |
|---|---|
| Atomicidad | ✔ 56 tareas, cada una con un entregable y un criterio de completitud verificable; ninguna supera 2 h (S: 14, M: 31, L: 11) |
| Estimación | ✔ Tamaño S/M/L por tarea; 54.5 h como máximo para el MVP |
| Prioridad | ✔ 50 Must y 6 Should; las Should no bloquean a ninguna Must |
| Dependencias | ✔ Grafo acíclico verificado por script; 11 niveles de paralelismo |
| Ruta crítica | ✔ T-001 → T-007 → T-008 → T-028 → T-032 → T-033 → T-038 → T-047 → T-050 (11 h) |
| Trazabilidad | ✔ HU, reglas, pantalla/CMP y componente ARQ en cada tarea; matrices HU, pantalla y regla → tarea |

---

## 8. Calidad de la UI Spec (`09` v1.0)

| Aspecto | Resultado |
|---|---|
| Inventario | ✔ 16 pantallas (incluida P-016 «Formulario caducado») y 12 componentes, todos con ficha |
| Flujos | ✔ F-01 a F-07 y transversales, sin callejones sin salida: `returnUrl` hacia rutas solo `POST`, cierre de sesión con la sesión expirada y página `400` resueltos |
| Wireframes | ✔ Las pantallas Must tienen wireframe a 360 px con atributos ARIA, tamaños y estados; el dashboard tiene variantes por escenario del seed |
| Literales frente al DOM | ✔ Dos puntos en los `<dt>` y « · » oculto en CMP-07 (DEC-83) |
| Mensajes | ✔ Errores de campo (CMP-03, CMP-04), errores que no son de un campo (`role="alert"`), confirmaciones MSG-49 a MSG-53 (`role="status"`) y aviso MSG-48 |
| Tokens y navegación | ✔ Contraste calculado, tamaños mínimos de 44 px, mapa de navegación con redirecciones |

---

## 9. Score general

| Criterio | Qué mide | Puntaje | Justificación |
|---|---|:-:|---|
| **Operatividad** | ¿Un agente puede ejecutar con esto sin inventar? | **4.7 / 5** | Cada tarea indica qué construir, con qué reglas, en qué pantalla y componente, y cómo se verifica. Los contratos, las rutas, el registro de Identity y el seed con cifras esperadas son directamente implementables. Todo está aprobado. **Resta:** la ruta exacta de Identity con `TimeProvider` depende de una prueba de concepto (T-013) |
| **Utilidad** | ¿El contenido resuelve las preguntas reales de implementación y prueba? | **4.6 / 5** | 193 criterios con tipo y nivel de prueba; ejemplos y contraejemplos alineados con el seed; casos borde, precedencia HTTP, zona horaria, redondeo, doble envío y sesión expirada resueltos. **Resta:** las métricas U1–U4 y las Should dependen de la validación con usuarios (PQ-01 a PQ-08) |
| **Orden** | ¿Estructura, versionado y nomenclatura consistentes? | **4.8 / 5** | Versión única v1.0 con índice; cabeceras uniformes; un significado por prefijo; 0 referencias rotas; registros de decisiones (DEC) y de preguntas (PQ) centralizados. **Resta:** el anexo del reviewer conserva la numeración de la revisión, señalado como fuente en el índice |
| **General** | Promedio | **4.7 / 5** | Listo para implementar |

---

## 10. Hallazgos

| ID | Hallazgo | Sev. | Tratamiento | Dónde |
|---|---|:-:|---|---|
| AF-01 | La ruta exacta por la que Identity usa el `TimeProvider` en el bloqueo no está confirmada (RA-05) | 🟢 | Prueba de concepto al inicio de la fase 1; si falla, se documenta en `07` §6.3 y se ajusta la prueba de RN-048 | T-013, `05` PQ-09 |
| AF-02 | Las Should (HU-012, HU-015 a HU-018) no cumplen la Definition of Ready | 🟢 | Se especifican después del MVP | T-055, T-056, `05` PQ-07, PQ-08 |
| AF-03 | Los supuestos del Brief y las métricas U1–U4 dependen de entrevistas que aún no ocurren | 🟢 | Validación con usuarios; no afecta al código del MVP | T-054, `05` PQ-01 a PQ-06 |
| AF-04 | `10-anexo-reporte-reviewer.md` conserva la numeración de la revisión (anterior a v1.0) | 🟢 | Es la fuente original de `10`; el índice la marca como «Fuente» y `10` usa la numeración v1.0 | `README.md` §1 |

**Bloqueantes: ninguno.** El agente puede tomar T-001.
