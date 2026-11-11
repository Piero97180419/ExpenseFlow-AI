# 02 — Alcance del producto — ExpenseFlow AI

| Campo | Valor |
|---|---|
| Documento | 02-alcance.md |
| Fase | Discovery (BMAD — Agente PM) |
| Insumos | `00-constitucion.md` v1.0, `01-project-brief.md` v1.0, ADR-001 a ADR-007 v1.0 |
| Rol en `/docs` | **PRD** del MVP (PRINCIPIO-012): funcionalidades, prioridades, fuera de alcance, supuestos y flujos principales. Índice en `/docs/README.md` |
| Versión | v1.0 |
| Fecha | 2026-11-10 |
| Estado | **Aprobado** |
| Aprobado por | Piero Carbajal |
| Fecha de aprobación | 2026-11-10 |
| PR de aprobación | PR que fusiona el Spec Pack v1.0 en `main` (ADR-005 §1) |
| Nota | Las **[Hipótesis]** siguen pendientes de validación con usuarios (`05-preguntas-abiertas.md`) |

> **Convención** (igual que el Brief)
> - **[Decisión]**: definido; no requiere validación para avanzar.
> - **[Hipótesis]**: supuesto sin evidencia; debe validarse.
> - **[Pendiente]**: información o decisión que falta confirmar.
>
> **Regla de este documento:** si una funcionalidad contradice la constitución, queda fuera del alcance, aunque aporte valor. Cualquier cambio a este alcance que toque un principio requiere ADR aprobado por un humano.
>
> **Formato de importes:** `S/ 1,250.50` (RN-081).
>

---

## 0. Criterio de priorización

El riesgo de mayor impacto es **R2: el alcance excede 6 semanas para 1 persona**. Por eso:

- Un **Must** es algo sin lo cual el producto no cumple su propuesta de valor **o** no se pueden medir los KPI de los niveles A y B del Brief (U1–U4, T1–T4).
- Todo lo que solo "mejora" la experiencia es **Should** o inferior.
- Presupuesto de capacidad orientativo: **Must en semanas 1–4**, semanas 5–6 para estabilización, pruebas E2E, accesibilidad, documentación y pruebas de usabilidad. Los **Should solo se inician si los Must están terminados con CI en verde**.

---

## 1. IN-SCOPE

### 1.1 Must have (MVP) — 6 funcionalidades

| ID | Funcionalidad | Descripción (una frase) | Principios / KPI relacionados |
|---|---|---|---|
| M1 | **Cuenta y acceso seguro** | El usuario se registra con correo, contraseña y confirmación de la contraseña (ASP.NET Core Identity), declara ser mayor de 18 años, entra directamente a su dashboard, inicia y cierra sesión, y solo ve sus propios datos. | PRINCIPIO-007, PRINCIPIO-008, PRINCIPIO-009 · T2 |
| M2 | **Registro rápido de gastos** | El usuario registra un gasto con monto, categoría predefinida, medio de pago (efectivo, Yape, Plin, tarjeta, otro) y fecha (hoy por defecto), y puede corregirlo o eliminarlo. | PRINCIPIO-003, PRINCIPIO-009 · U2, T1, T3 |
| M3 | **Registro de ingresos** | El usuario registra el dinero que recibe con monto, origen (apoyo familiar, trabajo, prácticas preprofesionales, trabajo independiente, otros) y fecha, y puede corregirlo o eliminarlo. | PRINCIPIO-003 · T1, T3 |
| M4 | **Presupuesto mensual** | El usuario define un monto límite de gasto para el mes en curso, que el sistema usa para calcular cuánto ha consumido y cuánto le queda. | PRINCIPIO-003, PRINCIPIO-004 · U3, T1 |
| M5 | **Dashboard de situación** | Muestra el saldo del mes, el saldo acumulado, el restante del presupuesto, el gasto por categoría, el porcentaje de presupuesto consumido, los últimos movimientos y un aviso textual y visual al llegar al 80 % y al 100 % del presupuesto. | PRINCIPIO-004, PRINCIPIO-009, PRINCIPIO-012 · T1, T3, T4 |
| M6 | **Resumen explicativo con IA** | A pedido del usuario, Ollama local redacta en español sencillo una explicación del mes en curso a partir de cifras ya calculadas por el backend y agregadas por categoría. | PRINCIPIO-004, PRINCIPIO-005 · U4 |

**Notas sobre los Must**

- **M1 — cuenta [Decisión, DEC-51, DEC-52]:** el formulario de registro tiene Correo, Contraseña, Confirmar contraseña y la casilla de mayoría de edad. El correo puede ser de cualquier dominio, con el formato de RN-150 (sin tildes, ñ ni símbolos distintos de punto, guion y guion bajo). Al crear la cuenta, el usuario entra directamente a su dashboard. Solo existen las pantallas de registro, inicio y cierre de sesión (RN-145), y no hay opción «Recordarme» (W20).
- **M2 — velocidad:** el formulario de gasto tiene exactamente 4 campos (Monto, Categoría, Medio de pago, Fecha) y valores por defecto (fecha de hoy, medio de pago del último gasto creado) para cumplir U2 (≤ 15 s). Las categorías son **fijas** en el MVP: Alimentación, Transporte, Estudios, Entretenimiento, Salud, Otros **[Decisión]**.
- **M2 — medio de pago:** es una **etiqueta descriptiva** ingresada manualmente; no implica conexión con Yape ni Plin (PRINCIPIO-009).
- **M2, M3 — corrección y eliminación [Decisión]:** se hacen desde una **lista de movimientos paginada y sin filtros**, ordenada como los últimos movimientos del dashboard. No es S3: S3 añade filtros por fecha, categoría, tipo y medio de pago. La eliminación es definitiva y pide confirmación; no existe el concepto de «movimiento anulado».
- **M3 — origen:** lista cerrada; con «Otros» se admite una descripción opcional de hasta 100 caracteres **[Decisión, DEC-13]**.
- **M5 — avisos:** los umbrales de 80 % y 100 % los calcula el backend con reglas deterministas; el aviso no depende de la IA. El dashboard muestra la leyenda «Saldo calculado según tus registros» junto a cada saldo (PRINCIPIO-009).
- **M5 — gráficos:** se usan barras de progreso de Bootstrap y tablas, **sin librería de gráficos**, para no requerir ADR (PRINCIPIO-002). Una librería de gráficos queda como Could (C4).
- **M6 — por qué es Must:** desde el punto de vista del usuario, el producto funcionaría sin IA (el Brief lo reconoce en 3.4). Se clasifica como Must porque **sin ella no se puede medir U4 ni demostrar los temas de IA del diplomado**, que son criterio de éxito del proyecto. Para contener el riesgo:
  - es **una sola** interacción: un resumen mensual bajo demanda (botón), no un chat;
  - recibe solo datos agregados del mes en curso (`month`, `totalIncome`, `totalExpenses`, `monthlyBalance`, `budget`, `consumedPercentage`, `remainingBudget`, `overspentAmount`, `totalsByCategory`), nunca datos personales (PRINCIPIO-005);
  - si Ollama no responde, el dashboard sigue funcionando y se muestra un mensaje de indisponibilidad (**degradación elegante**);
  - el texto indica que es una explicación generada automáticamente y no asesoría financiera;
  - el resumen se muestra **una sola vez** en el dashboard y no se guarda (RN-165) **[Decisión, DEC-54]**;
  - si el texto contiene una cifra que no está en los datos agregados, se descarta y se muestra el mensaje de indisponibilidad (RN-166) **[Decisión, DEC-55]**.

### 1.2 Should have — 5 funcionalidades (post-MVP, dentro de las 6 semanas solo si sobra capacidad)

| ID | Funcionalidad | Descripción (una frase) |
|---|---|---|
| S1 | **Presupuesto por categoría** | El usuario reparte su presupuesto mensual entre categorías y ve el consumo de cada una por separado. |
| S2 | **Periodo según la entrega de dinero** | El usuario elige si recibe su dinero cada 7 días, cada 15 días o una vez al mes, y la fecha de su última entrega; el sistema muestra el saldo del periodo «hasta la próxima entrega». Los presupuestos siguen siendo mensuales. |
| S3 | **Historial con filtros** | El usuario filtra la lista de movimientos por rango de fechas, categoría, tipo y medio de pago. |
| S4 | **Meta de ahorro simple** | El usuario define una meta (por ejemplo, «Laptop — S/ 2,500.00») y el sistema calcula el avance a partir de montos que él marca como ahorro. |
| S5 | **Explicación de IA al alcanzar el umbral** | Cuando el backend detecta el 80 % de consumo, se ofrece una explicación breve de la IA sobre las categorías que más pesan, con las cifras ya calculadas. |

> S2 responde mejor al momento detonante del Brief (3.3) que el presupuesto mensual fijo. Se deja como Should porque cambia el modelo de cálculo de periodos y multiplica los casos de prueba financieros (PRINCIPIO-011). Es el **primer candidato** a construir tras el MVP.

### 1.3 Could have — 4 funcionalidades

| ID | Funcionalidad | Descripción (una frase) |
|---|---|---|
| C1 | **Gastos frecuentes en un toque** | El usuario guarda plantillas («Pasaje — S/ 3.50») para registrarlas con un solo clic. |
| C2 | **Categorías personalizadas** | El usuario crea, renombra o archiva sus propias categorías además de las predefinidas. |
| C3 | **Exportación CSV** | El usuario descarga sus propios movimientos en un archivo CSV. |
| C4 | **Gráficos visuales** | Gráficos de torta o de líneas sobre gasto por categoría y evolución mensual (requiere ADR por la librería, PRINCIPIO-002). |

---

## 2. OUT-OF-SCOPE (Won't have en esta versión)

| ID | Funcionalidad | Razón por la que está fuera |
|---|---|---|
| W1 | Conexión con bancos o billeteras (BCP, Interbank, Yape, Plin, etc.) | Prohibido por PRINCIPIO-009. Además implicaría datos reales (PRINCIPIO-006) y dependencias externas. |
| W2 | Pagos, transferencias, depósitos, retiros o inversiones | Prohibido por PRINCIPIO-009: el producto solo registra y analiza. |
| W3 | Importación de estados de cuenta, PDF, Excel o lectura de vouchers (OCR) | PRINCIPIO-009 prohíbe leer estados de cuenta; PRINCIPIO-006 prohíbe datos reales; requeriría dependencias nuevas (PRINCIPIO-002). |
| W4 | Registro o edición de movimientos mediante IA (chat «gasté 10 en menú», agente que crea gastos) | PRINCIPIO-004: la IA nunca crea, modifica ni elimina movimientos o presupuestos. |
| W5 | Cálculos, proyecciones o pronósticos de gasto hechos por la IA | PRINCIPIO-004: toda cifra oficial la calcula el backend de forma determinista. |
| W6 | Uso de proveedores de IA en la nube (OpenAI, Anthropic, Google, etc.), incluso como respaldo, o de un Ollama en otro equipo | PRINCIPIO-005 y ADR-002: solo Ollama en los hosts permitidos. También contradice la restricción de costo cero. |
| W7 | Operación con datos reales o registro abierto al público | PRINCIPIO-006: requiere un ADR previo con evaluación de riesgos y obligaciones de la Ley N.° 29733. El formulario de registro existe solo en la aplicación local y advierte que no deben ingresarse datos reales (RN-099). |
| W8 | Despliegue público en Internet (VPS, nube, dominio) | Brief 7.4: no obligatorio en 6 semanas; PRINCIPIO-010 prohíbe a los agentes desplegar; PRINCIPIO-006 lo hace innecesario. |
| W9 | Multimoneda o conversión de divisas | PRINCIPIO-003: la única moneda es el sol (PEN). |
| W10 | Interfaz multilingüe | PRINCIPIO-001: solo español (es-PE) en el MVP. |
| W11 | App móvil nativa o PWA con modo offline | PRINCIPIO-002: el stack es web con Razor y Bootstrap; Brief 7.4 exige web responsive. |
| W12 | Cuentas compartidas, familiares o de grupo | PRINCIPIO-007: cada usuario solo accede a sus propios datos; el aislamiento compartido multiplica el riesgo de T2. |
| W13 | Inicio de sesión con Google, Facebook u otros proveedores externos | Dependencia externa y configuración de secretos de terceros; no aporta a los KPI del MVP. |
| W14 | Notificaciones push, por correo o SMS | Requieren servicios externos (posible costo y secretos); el aviso en el dashboard (M5) cubre la necesidad. |
| W15 | Monetización (freemium, suscripciones, publicidad, afiliaciones) | Brief 4.1 y 4.2: MVP gratuito; los datos no se usan para publicidad (PRINCIPIO-007). |
| W16 | Asesoría financiera o de inversión personalizada | PRINCIPIO-009 y riesgo regulatorio; la IA solo explica la situación, no recomienda productos financieros. |
| W17 | Recuperación de contraseña por correo **[Decisión]** | Requiere un servicio de correo (W14). Las cuentas demo se restablecen ejecutando el seed. |
| W18 | Confirmación de correo electrónico **[Decisión]** | Requiere un servicio de correo (W14). Con datos ficticios no aporta seguridad. |
| W19 | Gestión de cuenta: cambio de contraseña o de correo, eliminación de la cuenta, autenticación en dos pasos, descarga de datos personales | No aportan a los KPI del MVP; las cuentas son ficticias y las demo se restablecen con el seed. Sin correo (W17) no se podría recuperar el acceso tras un cambio de contraseña. |
| W20 | Sesión persistente («Recordarme») | M1 no la incluye; mantener la sesión tras cerrar el navegador alarga la exposición de la cookie (RN-097, DEC-48). |

---

## 3. MVP — Qué puede hacer un usuario solo con los Must

Un estudiante mayor de 18 años crea su cuenta declarando su mayoría de edad, entra directamente a su dashboard y vuelve a ingresar cuando lo necesita desde la computadora o desde su celular Android. El día en que recibe su dinero lo registra como ingreso y fija cuánto quiere gastar como máximo en el mes. Cada vez que paga algo, en efectivo, con Yape o con Plin, lo anota en pocos segundos indicando monto, categoría y medio de pago, y desde la lista de movimientos puede corregir o borrar lo que registró mal. En su dashboard ve en todo momento su saldo del mes y su saldo acumulado según sus registros, cuánto le queda de presupuesto, en qué categorías se le va más dinero y qué porcentaje de su presupuesto ya consumió, con un aviso claro cuando llega al 80 % y al 100 %. Cuando quiere entender su situación, pide un resumen y la IA local le explica en lenguaje sencillo, con las cifras ya calculadas por el sistema, cómo va su mes. Ninguno de sus datos es visible para otro usuario, y en esta versión todo funciona en local con cuentas y datos ficticios.

### 3.1 Trazabilidad Must → KPI del Brief

| KPI | M1 | M2 | M3 | M4 | M5 | M6 |
|---|:-:|:-:|:-:|:-:|:-:|:-:|
| U2 — Gasto en ≤ 15 s | | ✔ | | | | |
| U3 — Presupuesto sin asistencia | | | | ✔ | | |
| U4 — Comprensibilidad de la IA | | | | | | ✔ |
| T1 — Exactitud de cálculos | | ✔ | ✔ | ✔ | ✔ | |
| T2 — Aislamiento entre usuarios | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ |
| T3 — Flujos completos E2E | ✔ | ✔ | ✔ | ✔ | ✔ | |
| T4 — Uso desde Android | | ✔ | | ✔ | ✔ | |

> Si una funcionalidad no aparece en esta tabla, no es Must. U1 (participantes) no depende de ninguna funcionalidad, sino del reclutamiento.

### 3.2 Habilitadores técnicos (no son funcionalidades, pero son obligatorios)

- Seed reproducible con 6 cuentas demo y escenarios fijos (`03-reglas-de-negocio.md`, sección 11) para pruebas y sesiones de usabilidad (PRINCIPIO-006).
- API REST documentada en OpenAPI/Swagger (PRINCIPIO-012).
- Pipeline de CI bloqueante con las condiciones de PRINCIPIO-011.
- Pruebas E2E de la sección 3.3 como condición bloqueante del CI (PRINCIPIO-011, ADR-001).
- Pruebas de accesibilidad WCAG 2.2 AA en todos los flujos principales de la sección 3.3 (PRINCIPIO-012).

### 3.3 Flujos principales (lista única) **[Decisión]**

Esta es la lista a la que remiten PRINCIPIO-011, PRINCIPIO-012 y los KPI T3 y T4. Cada flujo tiene una prueba E2E con Playwright y una revisión axe.

| # | Flujo | Funcionalidad | T3 | T4 (Android) |
|---|---|---|:-:|:-:|
| F-01 | Registro de cuenta, inicio y cierre de sesión | M1 | ✔ | |
| F-02 | Registrar un gasto | M2 | ✔ | ✔ |
| F-03 | Corregir y eliminar un movimiento desde la lista | M2, M3 | ✔ | |
| F-04 | Registrar un ingreso | M3 | ✔ | |
| F-05 | Crear y editar el presupuesto del mes | M4 | ✔ | ✔ |
| F-06 | Consultar el dashboard (con y sin presupuesto, con exceso) | M5 | ✔ | ✔ |
| F-07 | Solicitar el resumen de IA (con Ollama simulado) | M6 | ✔ | |

### 3.4 Entornos de ejecución **[Decisión]**

| Entorno | Uso | Servicios | HTTPS | HSTS |
|---|---|---|---|---|
| `Development` | Desarrollo en el equipo del integrante | SQL Server Express y Ollama nativos | Sí, certificado de desarrollo de .NET | No |
| `Testing` | Pruebas de integración y E2E en CI | SQL Server en Docker; IA simulada | Sí | Sí |
| `Demo` | Sesiones de usabilidad y prueba T4 desde Android en red local | SQL Server Express y Ollama nativos, solo en `localhost` | Sí, certificado de `mkcert` instalado en el teléfono (ADR-001) | Sí |

> Detalle completo en ADR-004, sección 4.

---

## 4. Supuestos

### 4.1 De producto y usuario **[Hipótesis]** (heredados del Brief, sección 6.2: SU-1 a SU-9)

| ID | Supuesto | Impacto en el alcance si resulta falso |
|---|---|---|
| SP1 | Los estudiantes no registran sus gastos pequeños y no saben en qué gastan (SU-1). | El problema central se debilita; habría que revisar la propuesta de valor. |
| SP2 | Abandonan los métodos actuales por esfuerzo y tiempo (SU-2). | M2 dejaría de ser el diferenciador principal. |
| SP3 | El momento de recibir el dinero es el mejor para empezar a usar la app (SU-7). | S2 (periodo según entrega) perdería prioridad. |
| SP4 | Un presupuesto mensual único es suficiente para la primera versión. | Si las entrevistas muestran que la mayoría recibe dinero semanal, **S2 debe subir a Must** y algo debe bajar. |
| SP5 | Seis categorías fijas cubren la mayoría de gastos del segmento. | C2 (categorías personalizadas) subiría de prioridad. |
| SP6 | Una explicación en lenguaje natural resulta comprensible y aporta algo que los números solos no dan (SU-6). | M6 se reduciría a un requisito académico sin valor de producto. |
| SP7 | La persona principal representa al usuario real (SU-9). | Revisión del alcance completo. |

### 4.2 Técnicos y de ejecución **[Hipótesis]**

| ID | Supuesto |
|---|---|
| ST1 | `qwen3:4b`, ejecutado en CPU con 8 GB de RAM, genera texto en español aceptable dentro del tiempo de espera de 30 s (RN-082, ADR-003). |
| ST2 | Una sola persona puede completar M1–M6 con pruebas y CI en verde en unas 4 semanas, dejando 2 semanas para estabilización y validación. |
| ST3 | Un navegador de Android de gama media puede acceder a la aplicación en red local por HTTPS en el entorno `Demo`, con un certificado de `mkcert` cuya CA se instala en el teléfono, sin incumplir PRINCIPIO-008. |
| ST4 | Los minutos gratuitos de GitHub Actions alcanzan para el CI del proyecto, incluidas las pruebas E2E. |
| ST5 | SQL Server Express funciona de forma nativa en el equipo de desarrollo y en Docker en el CI sin limitaciones relevantes para el volumen de datos ficticios. |

---

## 5. Restricciones

### 5.1 De recursos

| Restricción | Origen |
|---|---|
| Equipo de 1 persona | Brief 7.1 |
| Plazo de 6 semanas | Brief 7.1 |
| Costo cero: sin APIs de IA de pago ni infraestructura con suscripción | Brief 7.3 |
| Evidenciar los 15 temas de la malla curricular | Brief 7.1 |

### 5.2 Técnicas

| Restricción | Origen |
|---|---|
| Stack de producto cerrado: ASP.NET Core MVC + Razor + Bootstrap 5, C# con API REST, EF Core, SQL Server Express, Identity, Ollama; monolito modular | PRINCIPIO-002 |
| Paquetes del stack de producto y herramientas de desarrollo, prueba y CI: solo las de ADR-001 | PRINCIPIO-002, ADR-001 |
| Toda tecnología o dependencia nueva requiere ADR aprobado por un humano | PRINCIPIO-002, PRINCIPIO-010 |
| Montos en `decimal` / `DECIMAL(18,2)`, solo PEN, redondeo con modo explícito centralizado (`AwayFromZero`) | PRINCIPIO-003 |
| Cálculos financieros solo en el backend y deterministas; la IA solo redacta | PRINCIPIO-004 |
| IA exclusivamente local (hosts de ADR-002) con datos agregados y sin datos personales | PRINCIPIO-005 |
| Código en inglés; documentación y UI en español (es-PE) | PRINCIPIO-001 |
| HTTPS en todos los entornos, hasher de Identity, lockout habilitado, secretos fuera del repositorio | PRINCIPIO-008 |
| CI bloqueante: formato, analizadores, pruebas unitarias, de integración y E2E, cobertura ≥ 80 % en la lógica de negocio, 100 % de reglas de cálculo, sin vulnerabilidades altas | PRINCIPIO-011 |
| WCAG 2.2 AA, mobile-first, operable por teclado | PRINCIPIO-012 |
| Ejecución en local; prueba desde Android físico en red local | Brief 7.4 |

### 5.3 Legales y éticas

| Restricción | Origen |
|---|---|
| Solo datos ficticios en desarrollo, pruebas y demostraciones | PRINCIPIO-006 |
| Ley N.° 29733 y su reglamento: si se recogen respuestas identificables en entrevistas, informar, obtener consentimiento cuando proceda, restringir acceso y limitar conservación | Brief 7.5 |
| Solo personas mayores de 18 años, con declaración obligatoria en el registro | PRINCIPIO-009, Brief 7.5 |
| Participantes identificados con seudónimos | Brief 7.5 |
| Los datos financieros no se usan para publicidad, venta ni entrenamiento de modelos | PRINCIPIO-007 |
| La UI aclara que los saldos son calculados y no bancarios, y que el resumen de IA no es asesoría financiera | PRINCIPIO-009 |

---

## 6. Dependencias

### 6.1 Técnicas

| Dependencia | Uso | Riesgo / nota |
|---|---|---|
| **Ollama** + `qwen3:4b` | Generación del resumen de M6 | Modelo, licencia (Apache 2.0), solicitud y evaluaciones en ADR-003. Instalación nativa. El cliente se implementa con `HttpClient`, sin paquete NuGet. Hosts y configuración en ADR-002. |
| **SQL Server Express** | Persistencia | Requisito obligatorio. Nativo en el equipo de desarrollo; en contenedor en el CI (ADR-004). |
| **.NET 10 (LTS)** (ASP.NET Core, EF Core, Identity) | Aplicación y migraciones | Versión fijada en `global.json` (ADR-001). Los paquetes que materializan el stack (Identity con EF Core, proveedor de SQL Server, OpenAPI) y la herramienta `dotnet-ef` están listados en ADR-001. No se usa `Microsoft.AspNetCore.Identity.UI`. |
| **Docker** | SQL Server para las pruebas de integración y E2E, **solo en el CI** | En el equipo de desarrollo no se usa, para dejar RAM al modelo (ADR-004). |
| **GitHub + GitHub Actions** | Repositorio, PR y CI | Depende del plan gratuito (ST4). |
| **NuGet / npm** | Obtención de paquetes | Solo paquetes de ADR-001 o de un ADR aprobado; versiones en `Directory.Packages.props`. |
| **Bootstrap 5** | UI responsive | Servir localmente (no por CDN) para que funcione en red local sin Internet. |
| **xUnit, Mvc.Testing, TimeProvider.Testing, coverlet, Playwright, axe** | Pruebas unitarias, integración, fecha fija, cobertura, E2E y accesibilidad | Aprobadas en ADR-001. |
| **mkcert** | Certificado HTTPS para el entorno `Demo` | Herramienta local; el certificado no se versiona (ADR-001). |
| **Teléfono Android de gama media** y red local | Verificación de T4 | Equipo físico disponible. |

> Las herramientas de prueba y CI que exigen PRINCIPIO-011 y PRINCIPIO-012 están listadas en **ADR-001**.

### 6.2 No técnicas

| Dependencia | Uso | Estado |
|---|---|---|
| Al menos 5 estudiantes universitarios mayores de 18 años | Entrevistas exploratorias y pruebas de usabilidad (U1–U4) | **[Pendiente]** Sin participantes confirmados (`05` PQ-02). |
| Formato de consentimiento informado del diplomado | Habilitar entrevistas y pruebas | **[Pendiente]** (`05` PQ-01). |
| Revisor humano | Aprobación de ADR, PR y checklist de revisión (PRINCIPIO-010, PRINCIPIO-011) | Rol asumido por el integrante del equipo. |

---

## Anexo — Decisiones de este documento

| # | Decisión | Estado |
|---|---|---|
| A1 | Herramientas de prueba y CI | **Cerrada:** ADR-001 |
| A2 | Modelo de Ollama | **Cerrada:** `qwen3:4b` (ADR-003) |
| A3 | HTTPS en red local para la prueba en Android (ST3) | **Cerrada:** entorno `Demo` + `mkcert` (ADR-001, ADR-004) |
| A4 | Reclasificar S2 a Must si las entrevistas muestran que predomina la entrega semanal (SP4) | Abierta, no bloquea el MVP: `05-preguntas-abiertas.md`, PQ-05 |
| A5 | Arquitectura, entornos y gobierno de cambios | **Cerrada:** ADR-004 y ADR-005 |
