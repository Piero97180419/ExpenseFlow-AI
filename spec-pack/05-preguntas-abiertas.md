# 05 — Preguntas abiertas — ExpenseFlow AI

| Campo | Valor |
|---|---|
| Documento | 05-preguntas-abiertas.md |
| Fase | Discovery (BMAD — Agente QA) |
| Versión | v1.0 |
| Fecha | 2026-11-10 |
| Estado | **Aprobado** |
| Aprobado por | Piero Carbajal |
| Fecha de aprobación | 2026-11-10 |
| PR de aprobación | PR que fusiona el Spec Pack v1.0 en `main` (ADR-005 §1) |
| Insumos | `00-constitucion.md` a `04-glosario.md` v1.0, `06-user-stories.md` v1.0, `07-architecture.md` v1.0 |

> **Propósito:** reunir en un solo lugar lo que el Spec Pack todavía no puede afirmar. Cada pregunta tiene un ID estable (PQ-NN), un responsable, un momento de cierre y el efecto que tendría cada respuesta posible.
>
> **Resultado de la revisión QA:** ninguna pregunta bloquea la implementación del MVP (M1 a M6). Las preguntas PQ-01 a PQ-06 dependen de la validación con usuarios; PQ-07 y PQ-08 solo afectan a funcionalidades Should; PQ-09 es técnica y se cierra con una prueba de concepto dentro del backlog.
>
> **Cómo se cierra una pregunta:** se registra la respuesta en la columna «Estado» de §1, se crea la decisión DEC-NN en `03` §9 si cambia una regla, se actualizan los documentos de la columna «Afecta a» y se sube la versión menor de cada uno (ADR-005 §1).

---

## 1. Resumen

| ID | Pregunta | Tipo | Bloquea el MVP | Responsable | Momento de cierre | Afecta a | Estado |
|---|---|---|---|---|---|---|---|
| PQ-01 | ¿Qué requisitos de consentimiento informado exige el diplomado para entrevistar y hacer pruebas con estudiantes? | Proceso | No | Equipo | Antes de la semana 1 de validación | `01` §6.3, `02` §6.2 | Abierta |
| PQ-02 | ¿Quiénes son los al menos 5 participantes (estudiantes universitarios mayores de 18 años)? | Proceso | No | Equipo | Semanas 1–2 de validación | `01` §6.3, `02` §6.2 | Abierta |
| PQ-03 | ¿Las entrevistas confirman los supuestos SU-1 a SU-4, SU-7 y SU-9? | Producto | No | Equipo | Fin de la semana 2 de validación | `01` §2 y §6.2, `02` §4.1 | Abierta |
| PQ-04 | ¿Qué metas numéricas tendrán las métricas futuras F1 a F5? | Producto | No | Equipo (fase futura) | Implementación con usuarios reales | `01` §5 | Abierta |
| PQ-05 | ¿La mayoría del segmento recibe su dinero cada semana, de modo que S2 debe pasar a Must (A4, SP4)? | Alcance | No | PO (Piero Carbajal) | Tras PQ-03 | `02` §1 y Anexo (A4), `06` HU-012, `08` | Abierta |
| PQ-06 | ¿«Efectivo» es el medio de pago más usado por el segmento, como supone RN-108? | Regla | No | PO | Tras PQ-03 | `03` RN-108, `09` P-004 | Abierta |
| PQ-07 | ¿Qué textos usan los rechazos de RN-143 y RN-161 y el rótulo del saldo del periodo (S2)? | Especificación | No (Should) | PO | T-055, al planificar S2 | `03` §10, `06` HU-012, `09` | Abierta |
| PQ-08 | ¿Qué reglas, mensajes y pantallas completan S1, S3, S4 y S5? | Especificación | No (Should) | PO | T-056, al planificar cada Should | `03`, `06` HU-015 a HU-018, `09` | Abierta |
| PQ-09 | ¿ASP.NET Core Identity usa el `TimeProvider` registrado en todas las rutas del bloqueo de cuenta (RA-05)? | Técnica | No | Desarrollador | Inicio de T-013 | `07` §6.3, `03` RN-048, `08` | Abierta |

---

## 2. Detalle

### PQ-01 — Consentimiento informado

- **Contexto:** el Brief plantea entrevistas exploratorias y pruebas de usabilidad (U1 a U4) con estudiantes. No se ha confirmado qué documento de consentimiento pide el diplomado.
- **Opciones:** (a) formato propio del diplomado; (b) formato breve del equipo aprobado por el docente.
- **Si no se cierra:** no se hacen entrevistas ni pruebas con personas; el MVP se valida solo con las pruebas automáticas y las cuentas demo.
- **Origen:** `01` Anexo, `02` §6.2.

### PQ-02 — Participantes

- **Contexto:** se necesitan al menos 5 estudiantes universitarios mayores de 18 años (PRINCIPIO-009) para las entrevistas y las métricas U1 a U4.
- **Si no se cierra:** las métricas U1 a U4 se miden solo con el equipo y se declaran como tales en la presentación.
- **Origen:** `01` Anexo, `02` §6.2.

### PQ-03 — Validación de supuestos

- **Contexto:** el problema se sustenta en situaciones plausibles, no en evidencia (`01` §2). Los supuestos SU-1 a SU-4, SU-7 y SU-9 se validan con las entrevistas.
- **Efecto de cada respuesta:** si se confirman, se cambia **[Hipótesis]** por la evidencia en `01`. Si se refuta SU-1 o SU-2, se revisa la propuesta de valor (SP1, SP2). Si se refuta SU-9, se revisa el alcance completo (SP7).
- **Origen:** `01` Anexo, `01` §6.3.

### PQ-04 — Metas de las métricas futuras

- **Contexto:** F1 a F5 miden uso real (retención, registros por semana…) y no aplican a la demo académica.
- **Si no se cierra:** no afecta al MVP; F1 a F5 siguen sin meta.
- **Origen:** `01` Anexo, `01` §5.

### PQ-05 — Prioridad de S2

- **Contexto:** SP4 supone que un presupuesto mensual basta. A4 (Anexo de `02`) quedó abierta.
- **Efecto de cada respuesta:** si predomina la entrega semanal, S2 pasa a Must, se reprioriza el backlog y alguna funcionalidad Must o Should baja (decisión del PO). Si no, S2 se mantiene como Should.
- **Dependencia:** PQ-03 y, para S2, PQ-07.

### PQ-06 — Medio de pago por defecto

- **Contexto:** RN-108 preselecciona «Efectivo» para el primer gasto. Es una **[Hipótesis]** de la Decisión del equipo.
- **Efecto de cada respuesta:** si se confirma, RN-108 pierde la etiqueta de hipótesis. Si el medio más usado es otro (por ejemplo, «Yape/Plin»), se cambia el valor de RN-108 y su prueba con una nueva DEC-NN.
- **Dependencia:** PQ-03.

### PQ-07 — Mensajes de S2

- **Contexto:** RN-143 y RN-161 definen rechazos del periodo según la entrega, pero `03` §10 no tiene sus textos MSG-NN ni el rótulo del saldo del periodo. Por eso HU-012 no cumple todavía la Definition of Ready.
- **Se cierra:** en la tarea T-055, agregando los mensajes a `03` §10, sus criterios a HU-012 y su pantalla a `09`.

### PQ-08 — Especificación de S1, S3, S4 y S5

- **Contexto:** HU-015 a HU-018 están en estado «Propuesta» porque S1, S3, S4 y S5 no tienen reglas suficientes (RN-044 a RN-047 cubren solo una parte; RN-046 lo declara para S4; para S5 falta fijar el payload, las instrucciones y las evaluaciones de ADR-003).
- **Se cierra:** con las reglas, los mensajes, las HU listas y las pantallas de cada Should, en la tarea T-056.

### PQ-09 — `TimeProvider` en Identity

- **Contexto:** las pruebas del bloqueo de cuenta (RN-048) avanzan el reloj con `FakeTimeProvider`. Si alguna ruta de Identity usa el reloj del sistema, esas pruebas no serían deterministas (riesgo RA-05 de `07`).
- **Se cierra:** con la prueba de concepto al inicio de T-013. Si falla, se documenta la limitación en `07` §6.3 y la prueba de RN-048 se ajusta a la ruta que sí usa el `TimeProvider`.

---

## 3. Preguntas cerradas durante la elaboración del Spec Pack

Las preguntas que el equipo respondió mientras se escribía el Spec Pack no quedan abiertas: su respuesta está registrada como decisión en `03` §9 (DEC-01 a DEC-90) y aplicada en el documento correspondiente.
