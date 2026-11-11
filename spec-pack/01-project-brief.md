# Project Brief — ExpenseFlow AI

| Campo | Valor |
|---|---|
| Documento | 01-project-brief.md |
| Fase | Discovery (BMAD — Agente Analyst) |
| Contexto | Proyecto académico — Diploma en AI-First Software Architecture |
| País de operación | Perú (foco inicial: Lima Metropolitana) |
| Versión | v1.0 |
| Fecha | 2026-11-10 |
| Estado | **Aprobado** |
| Aprobado por | Piero Carbajal |
| Fecha de aprobación | 2026-11-10 |
| PR de aprobación | PR que fusiona el Spec Pack v1.0 en `main` (ADR-005 §1) |
| Nota | Las **[Hipótesis]** siguen pendientes de validación con usuarios (`05-preguntas-abiertas.md`) |

> **Convención de este documento**
> - **[Decisión]**: definido por el equipo; no requiere validación para avanzar.
> - **[Hipótesis]**: supuesto razonable sin evidencia todavía; debe validarse.
> - **[Pendiente]**: información que falta confirmar.
>
> **Formato de importes:** este documento usa el formato de la UI definido en `03-reglas-de-negocio.md` (RN-081): punto decimal y coma de miles (`S/ 2,500.00`).
>
> **Lenguaje:** este documento describe el problema con palabras del usuario, como «dinero disponible» o «cuánto le queda». En la UI y en las especificaciones se usan los términos del glosario (`04-glosario.md`): «Saldo del mes», «Saldo acumulado» y «Restante del presupuesto». «Disponible» no se usa como rótulo en la UI.

---

## 1. Problema y oportunidad

### 1.1 Problema

Los estudiantes universitarios realizan **gastos pequeños y frecuentes** (alimentación, transporte, materiales de estudio, entretenimiento) pagados en efectivo, Yape o Plin, **sin llevar un registro consolidado**. Al terminar la semana o el mes, no saben cuánto gastaron, en qué, ni cuánto les queda disponible hasta la siguiente entrega de dinero.

El problema de fondo no es la falta de herramientas, sino el **abandono del registro**: los métodos actuales exigen disciplina y tiempo, y la persona deja de usarlos tras pocos días.

### 1.2 Cómo lo resuelven hoy y dónde falla **[Hipótesis]**

| Alternativa actual | Limitación percibida |
|---|---|
| Apps bancarias (BCP, Interbank, BBVA, etc.) | Solo muestran movimientos de su propia cuenta; no consolidan efectivo ni otros medios de pago. |
| Historial de Yape / Plin | Muestra pagos de una sola billetera; no hay presupuesto ni categorías personales. |
| Excel / Google Sheets | Requiere organización y actualización constante; se abandona. |
| Notas del celular | Sin cálculos, sin categorías, sin visión de conjunto. |
| Memoria | Imprecisa; los gastos pequeños se olvidan. |
| Apps de finanzas genéricas (Monefy, Wallet, Spendee) | No están pensadas para el contexto del estudiante peruano. Su uso real por el segmento no está verificado. |

### 1.3 Consecuencias para el usuario **[Hipótesis]**

- Pérdida de control sobre el dinero disponible y dificultad para llegar a la siguiente entrega.
- Gastos no planificados ("gastos hormiga") que reducen la capacidad de ahorro.
- Retraso de metas concretas: laptop, materiales universitarios, pago de estudios.
- Preocupación al no saber si el dinero alcanzará para necesidades importantes.

*Ejemplo ilustrativo (no es un promedio comprobado):* S/ 5.00 diarios en gastos no planificados equivalen a unos S/ 150.00 en 30 días.

### 1.4 Oportunidad

Existe un espacio para una herramienta que **consolide todos los medios de pago** (incluido el efectivo), sea **rápida de usar** y aporte **valor desde el primer día**, en el momento en que el estudiante recibe su dinero y necesita distribuirlo.

### 1.5 Evidencia disponible

**[Pendiente]** No se han realizado encuestas ni entrevistas. El problema se sustenta en situaciones cotidianas plausibles. La validación exploratoria está planificada (sección 6.3; seguimiento en `05-preguntas-abiertas.md`, PQ-01 a PQ-03).

---

## 2. Usuarios y segmentos objetivo

### 2.1 Segmento prioritario (MVP) **[Decisión]**

**Estudiantes universitarios de Lima Metropolitana, mayores de 18 años, con presupuesto personal limitado.**

### 2.2 Persona principal **[Hipótesis]**

| Atributo | Descripción |
|---|---|
| Edad | 18 a 25 años (mayor de edad) |
| Ubicación | Lima Metropolitana |
| Ocupación | Estudiante universitario |
| Fuente de ingresos | Dinero semanal o mensual de familiares; en algunos casos, ingresos adicionales por trabajos de medio tiempo, ocasionales o prácticas preprofesionales |
| Medios de pago | Efectivo, Yape, Plin |
| Dispositivo | Teléfono Android de gama media |
| Gastos frecuentes | Alimentación, transporte, materiales de estudio, entretenimiento |
| Comportamiento | No registra todos sus movimientos; a fin de mes no sabe en qué gastó ni cuánto pudo ahorrar |
| Expectativa | Herramienta sencilla, gratuita y accesible desde el celular |

### 2.3 Segmentos secundarios (no prioritarios en el MVP) **[Decisión]**

| Segmento | Diferencia principal | Tratamiento |
|---|---|---|
| Jóvenes profesionales | Ingreso fijo en planilla; mayor capacidad de ahorro | Pueden usar el sistema; no guían las decisiones del MVP |
| Trabajadores independientes | Ingresos irregulares (recibos por honorarios) | Pueden usar el sistema; no guían las decisiones del MVP |

### 2.4 Necesidades diferenciales del segmento prioritario

- Registrar **ingresos regulares u ocasionales** sin requerir empleo formal ni cuenta bancaria.
- Distribuir un monto limitado entre categorías esenciales (transporte, alimentación, estudios).
- Saber **cuánto queda** hasta la próxima entrega de dinero.

---

## 3. Propuesta de valor

### 3.1 Declaración **[Decisión]**

> **"ExpenseFlow AI te ayuda a saber en qué gastas tu dinero, cuánto te queda disponible y cómo administrarlo mejor, sin complicarte con hojas de cálculo."**

> Nota: esta declaración es un mensaje de producto (ver «Lenguaje» en la cabecera).

### 3.2 Resultados que espera el usuario

| Resultado esperado | Dolor que atiende |
|---|---|
| Saber en qué categorías gasta más | Desconocimiento de en qué se fue el dinero |
| Conocer cuánto dinero le queda | Incertidumbre sobre si alcanzará hasta la próxima entrega |
| Saber a tiempo cuándo se acerca a su límite de gasto | Descubrir el problema cuando ya es tarde |
| Recibir explicaciones comprensibles sobre su situación | Dificultad para interpretar sus propios números |
| Registrar movimientos con mínimo esfuerzo | Abandono del registro manual |

> Estos son **resultados**, no funcionalidades. Su traducción a funcionalidades corresponde a la fase de Product Manager (PRD).

### 3.3 Momento detonante **[Hipótesis]**

- **Principal (prioritario para el MVP):** el día en que el estudiante **recibe su dinero** semanal o mensual y necesita distribuirlo para que le alcance.
- **Secundario:** cuando descubre que gasta más de lo previsto o quiere ahorrar para una compra concreta (por ejemplo, una laptop).

**Implicancia:** el producto debe aportar valor **desde el primer uso**, sin depender de meses de historial.

### 3.4 Rol de la inteligencia artificial **[Decisión]**

- La IA cumple un **rol complementario**: interpreta la situación financiera del usuario y la explica en lenguaje sencillo.
- Los **cálculos de montos, saldos y presupuestos** son responsabilidad del sistema mediante **reglas determinísticas**, no de la IA.
- *Ejemplo:* con un presupuesto de S/ 600.00 y S/ 450.00 gastados a mitad de mes, el sistema calcula el restante y el porcentaje consumido, y la IA explica en lenguaje sencillo el riesgo de exceder el presupuesto.
- Se reconoce que la inclusión de IA también responde a los **objetivos académicos** del diplomado. **No se presenta como una necesidad comprobada del usuario.**

### 3.5 Diferenciación frente a alternativas **[Hipótesis]**

- Enfoque específico en el estudiante universitario peruano y sus medios de pago (incluido el efectivo).
- Consolidación de todos los movimientos en un solo lugar.
- Menor esfuerzo y menor configuración que una hoja de cálculo.
- Explicaciones comprensibles en lugar de solo números.

**Advertencia:** una persona que abandonó Excel también podría abandonar ExpenseFlow AI. No se asume mayor retención que las alternativas sin pruebas con usuarios.

---

## 4. Modelo de negocio preliminar

### 4.1 MVP académico **[Decisión]**

- **Producto gratuito, sin monetización.**
- No incluye suscripciones, publicidad, afiliaciones comerciales ni procesamiento de pagos.
- El objetivo principal del MVP es **demostrar la aplicación de los conocimientos del diplomado**.

### 4.2 Compromiso con los datos **[Decisión]**

- Los datos financieros de los usuarios **no se usarán para publicidad**.
- **No se venderá** información personal a terceros.

### 4.3 Alternativa comercial futura **[Hipótesis]**

| Elemento | Descripción |
|---|---|
| Modelo | Freemium |
| Gratuito | Funciones principales: registro, presupuestos y reportes |
| De pago | Análisis financiero avanzado |
| Precio exploratorio | S/ 5.00 mensuales (no validado) |
| Condición | El acceso gratuito a las herramientas básicas no debe verse comprometido |

**[Pendiente]** Validar si el segmento tiene disposición de pago, considerando su presupuesto limitado.

### 4.4 Alternativas descartadas para el MVP

- Publicidad y afiliación a productos financieros: incompatibles con el compromiso de datos (4.2).
- Alianzas B2B2C (universidades, bancos, fintechs): no evaluadas; podrían explorarse en el futuro.

---

## 5. Métricas de éxito (KPIs medibles)

Las métricas se organizan en tres niveles. **Solo los niveles A y B se reportan como resultados del proyecto.**

### 5.A Métricas de usabilidad del MVP académico (medibles en 6 semanas)

| ID | KPI | Meta | Método de medición |
|---|---|---|---|
| U1 | Participantes en prueba de usabilidad | ≥ 5 estudiantes universitarios, mayores de 18 años | Registro de sesiones (Participante 1…N) |
| U2 | Tiempo para registrar un gasto | ≥ 4 de 5 participantes (≥ 80 %) en ≤ 15 segundos | Cronometraje en el **tercer gasto registrado**, tras una tarea de práctica guiada |
| U3 | Configuración autónoma de presupuesto | ≥ 80 % de participantes configura un presupuesto mensual **sin asistencia** | Observación durante la sesión |
| U4 | Comprensibilidad de los resúmenes explicativos de IA | ≥ 4 de 5 participantes califican con 4 o 5 (escala 1–5) | Cuestionario post-tarea |

> **Limitación de U4:** las pruebas usan **datos ficticios**. Por tanto, U4 mide si los resúmenes son **comprensibles y plausibles**, no si son **útiles para la vida financiera real** del participante. **No debe interpretarse como validación de la propuesta de valor.**

### 5.B Criterios de aceptación técnicos (verificables en 6 semanas)

| ID | Criterio | Meta | Método |
|---|---|---|---|
| T1 | Exactitud de cálculos financieros (saldos, totales, presupuesto restante) | 100 % de casos de prueba definidos pasan | Pruebas automatizadas (xUnit) |
| T2 | Aislamiento de datos entre usuarios | 0 accesos exitosos a datos de otro usuario en los casos de prueba | Pruebas automatizadas (xUnit / Playwright) |
| T3 | Flujos principales completos (lista única en `02-alcance.md`, sección 3.3) | 100 % de flujos definidos completados de extremo a extremo | Pruebas E2E (Playwright) |
| T4 | Uso desde celular | ≥ 1 prueba funcional exitosa de los flujos principales desde un Android físico | Prueba manual en red local, entorno `Demo` |

### 5.C Métricas futuras del producto (no se reportan como resultados del proyecto)

| ID | KPI | Definición |
|---|---|---|
| F1 | Frecuencia de registro | % de usuarios activos que registran movimientos ≥ 3 días por semana |
| F2 | Retención a 4 semanas | % de usuarios que siguen registrando movimientos en la semana 4 tras su alta |
| F3 | Presupuesto actualizado | % de usuarios que mantienen un presupuesto vigente en el periodo actual |
| F4 | Control percibido | % de usuarios que declaran que la app les ayuda a controlar mejor sus gastos (encuesta) |
| F5 | Disposición de pago | % de usuarios dispuestos a pagar por funciones avanzadas (hipótesis freemium) |

**[Pendiente]** Las metas numéricas de F1 a F5 se definirán cuando exista una implementación con usuarios reales.

---

## 6. Riesgos identificados y supuestos

### 6.1 Riesgos

| ID | Riesgo | Probabilidad | Impacto | Mitigación prevista |
|---|---|---|---|---|
| R1 | **Abandono del registro**: el estudiante deja de registrar tras los primeros días, como con Excel | **Alta** (riesgo más probable) | Alto (producto) | Minimizar esfuerzo de registro; valor desde los primeros movimientos; evitar configuración innecesaria |
| R2 | **Alcance excede las 6 semanas** para un equipo de 1 persona (IA, pruebas, 15 temas del diplomado) | Media-Alta | **Muy alto** (riesgo de mayor impacto académico) | Priorizar solo lo esencial; datos ficticios; pruebas desde las primeras semanas |
| R3 | **Calidad de la IA**: el modelo local genera resúmenes incorrectos, genéricos o poco útiles | Media | Medio | Cálculos financieros independientes de la IA; la IA solo interpreta; evaluaciones con Ollama real (PRINCIPIO-011) |
| R4 | **Validación**: no se consiguen 5 participantes a tiempo | Media | Medio | Planificar la validación en las semanas 1–2 |
| R5 | **Confianza**: los usuarios no quieren registrar datos financieros en una app nueva | Media | Alto (producto) | Compromiso de no usar datos para publicidad; en el MVP solo datos ficticios |
| R6 | **Validez limitada de los resultados**: probar con datos ficticios no demuestra utilidad real | Alta | Medio | Declarar explícitamente la limitación (ver 5.A, U4) |

> Probabilidades e impactos son **estimaciones iniciales** y se revisarán durante el proyecto.

### 6.2 Supuestos por validar

| ID | Supuesto | Cómo validarlo |
|---|---|---|
| SU-1 | Los estudiantes no registran sus gastos pequeños y no saben en qué gastan | Entrevistas exploratorias |
| SU-2 | Usan apps bancarias, notas, Excel o memoria, y los abandonan por esfuerzo y tiempo | Entrevistas exploratorias |
| SU-3 | Los gastos no planificados afectan sus metas financieras de forma relevante | Entrevistas exploratorias |
| SU-4 | El segmento tiene interés suficiente en usar una app financiera con frecuencia | Entrevistas + prueba de prototipo |
| SU-5 | Mantendrán el hábito si registrar es rápido y obtienen información útil | Solo validable a mediano plazo (F1, F2) |
| SU-6 | Los resúmenes explicativos les resultan útiles | Parcialmente con U4; plenamente con datos reales en el futuro |
| SU-7 | Están más dispuestos a probar la app al recibir su dinero | Entrevistas exploratorias |
| SU-8 | Existe disposición a pagar S/ 5.00 mensuales por funciones avanzadas | Fuera del alcance del MVP |
| SU-9 | La persona principal (sección 2.2) representa al usuario real | Entrevistas exploratorias |

### 6.3 Plan de validación exploratoria **[Pendiente]**

- **Meta:** 5 entrevistas exploratorias y 1 prueba de usabilidad con prototipo.
- **Cuándo:** semanas 1 y 2 del proyecto.
- **Temas:** cómo administran hoy su dinero, dificultades, uso de apps financieras, utilidad percibida.
- **Estado:** sin entrevistas realizadas ni participantes confirmados.

---

## 7. Restricciones conocidas

### 7.1 Equipo y plazo

| Restricción | Origen |
|---|---|
| Equipo de **1 integrante** | Contexto del proyecto |
| Plazo total de **6 semanas** | Objetivo académico de planificación |
| Demostrar los **15 temas de la malla curricular** (desarrollo asistido por IA, especificaciones, arquitectura, pruebas, documentación, despliegue) | Requisito académico |
| La revisión humana de los PR la hace el único integrante, que también es autor o supervisor del agente: **no es una revisión independiente** | Equipo de 1 persona (ADR-005, sección 6) |
| Equipo de desarrollo: PC x64 con 8 GB de RAM, sin GPU | Recursos del integrante (ADR-004) |

### 7.2 Tecnología

| Componente | Elección | Origen |
|---|---|---|
| Base de datos | **Microsoft SQL Server Express** | **Requisito técnico obligatorio** |
| Frontend | ASP.NET Core MVC, Razor Views, Bootstrap, JavaScript | Decisión del equipo |
| Backend | ASP.NET Core con C# | Decisión del equipo |
| ORM | Entity Framework Core | Decisión del equipo |
| IA | Ollama con modelo local | Decisión del equipo |
| Herramientas | Git, GitHub, GitHub Actions (con Docker solo en CI), xUnit, Playwright y demás herramientas de ADR-001 | Decisión del equipo |
| Entorno local | SQL Server Express y Ollama (`qwen3:4b`) instalados de forma nativa | ADR-003, ADR-004 |

> JavaScript se usa solo para interacción y validación del lado del cliente; nunca para calcular cifras oficiales (RN-027).

### 7.3 Costos

- No se usarán APIs de IA de pago.
- No se usarán servicios de infraestructura con suscripción obligatoria.
- Se priorizan herramientas gratuitas y recursos locales.

### 7.4 Entorno de uso y despliegue **[Decisión]**

- Aplicación **web responsive**, utilizable desde computadora y celular.
- **Obligatorio:** acceso funcional desde un Android de gama media a los flujos principales (registro de gastos, presupuestos, dashboard).
- **No obligatorio:** publicación en Internet o contratación de VPS durante las 6 semanas. El despliegue público queda para una fase posterior.
- Desarrollo y demostración en entorno local y controlado, con prueba desde teléfono físico en red local.

### 7.5 Datos y privacidad

| Aspecto | Decisión / estado |
|---|---|
| Datos del MVP | **Solo datos financieros ficticios** para desarrollo, pruebas y demostraciones **[Decisión]** |
| Pruebas con participantes | Cuentas de demostración y escenarios preparados; no se solicita información financiera real **[Decisión]** |
| Identificación de participantes | Seudónimos (Participante 1, 2, 3…); se evita información personal innecesaria **[Decisión]** |
| Marco legal | Ley N.° 29733, Ley de Protección de Datos Personales, y su reglamento vigente. Si se recopilan respuestas identificables: información al participante, consentimiento cuando proceda, acceso restringido y conservación limitada |
| Consentimiento informado | **[Pendiente]** Confirmar si el diplomado exige un formato específico antes de entrevistas o pruebas |
| Edad de participantes | Solo mayores de 18 años **[Decisión]** |

---

## Anexo — Pendientes abiertos

Los pendientes de este documento (consentimiento informado, reclutamiento, entrevistas y metas de las métricas futuras) se gestionan en `05-preguntas-abiertas.md` (PQ-01 a PQ-04). Ninguno bloquea la implementación.
