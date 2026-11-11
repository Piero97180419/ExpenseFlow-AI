# 04 — Glosario (lenguaje ubicuo) — ExpenseFlow AI

| Campo | Valor |
|---|---|
| Documento | 04-glosario.md |
| Fase | Discovery (BMAD — Agente Business Analyst) |
| Versión | v1.0 |
| Fecha | 2026-11-10 |
| Estado | **Aprobado** |
| Aprobado por | Piero Carbajal |
| Fecha de aprobación | 2026-11-10 |
| PR de aprobación | PR que fusiona el Spec Pack v1.0 en `main` (ADR-005 §1) |
| Insumos | `00-constitucion.md` a `03-reglas-de-negocio.md` v1.0, `07-architecture.md` v1.0, `09-ui-spec.md` v1.0, ADR-001 a ADR-007 v1.0 |

> **Uso:** la columna «Término» es el nombre en la UI y la documentación (español, es-PE). La columna «En código» es el nombre en el código (inglés, PRINCIPIO-001), tal como lo fija `07-architecture.md`. Las claves en `camelCase` (`month`, `totalIncome`…) son campos del JSON enviado a la IA (RN-034) o de la API (ADR-004 §3); los nombres en `PascalCase` son tipos o propiedades de C#.
>
> Secciones: §1 términos del dominio · §2 términos de interfaz · §3 términos técnicos y de proceso · §4 convención de identificadores · §5 términos a evitar.

---

## 1. Términos del dominio

| # | Término | Definición | En código | Fuente |
|---|---|---|---|---|
| 1 | **Usuario** | Persona que declaró ser mayor de 18 años y tiene una cuenta en ExpenseFlow AI. Solo accede a sus propios datos financieros. | `ApplicationUser` | PRINCIPIO-007, PRINCIPIO-009 |
| 2 | **Usuario autenticado** | Usuario con sesión iniciada, identificado por el `UserId` (GUID de Identity) tomado del contexto de autenticación, nunca de un dato enviado por el cliente. | `UserId` | PRINCIPIO-007, RN-007, RN-103 |
| 3 | **Declaración de mayoría de edad** | Casilla obligatoria del registro con el texto «Declaro ser mayor de 18 años». Sin ella no se crea la cuenta. | `AdultDeclaration` (campo del formulario de registro; no se persiste) | PRINCIPIO-009, RN-001 |
| 4 | **Movimiento** | Registro manual de dinero que entra o sale. Es un gasto o un ingreso. No proviene de bancos ni billeteras. | `Transaction` | PRINCIPIO-003, PRINCIPIO-009 |
| 5 | **Gasto** | Movimiento de salida de dinero, con monto, categoría, medio de pago y fecha, todos obligatorios. | `TransactionType.Expense` | M2, RN-105 |
| 6 | **Ingreso** | Movimiento de entrada de dinero, con monto, origen y fecha obligatorios. No tiene medio de pago. | `TransactionType.Income` | M3, DEC-13, RN-106 |
| 7 | **Monto** | Importe positivo de un movimiento en soles, entre S/ 0.01 y S/ 100,000.00, con máximo 2 decimales. Se representa como `decimal` / `DECIMAL(18,2)`. El tipo, no el signo, indica si es ingreso o gasto. | `Amount` | PRINCIPIO-003, DEC-01, DEC-02 |
| 8 | **Sol peruano (PEN)** | Única moneda del sistema. Se muestra con el patrón «S/ #,##0.00» y un espacio normal entre «S/» y la cifra (p. ej., «S/ 1,250.50»). | `MoneyFormat` | PRINCIPIO-003, RN-013, RN-081 |
| 9 | **Categoría** | Clasificación de un gasto. En el MVP es una lista fija: Alimentación, Transporte, Estudios, Entretenimiento, Salud, Otros. | `ExpenseCategory` (`Food`, `Transport`, `Studies`, `Entertainment`, `Health`, `Other`) | M2, RN-014 |
| 10 | **Medio de pago** | Etiqueta descriptiva de cómo se pagó un gasto: Efectivo, Yape, Plin, Tarjeta, Otro. No implica ninguna conexión con el proveedor. | `PaymentMethod` (`Cash`, `Yape`, `Plin`, `Card`, `Other`) | M2, PRINCIPIO-009, ADR-004 |
| 11 | **Presupuesto mensual** | Monto límite de gasto (S/ 0.01 a S/ 100,000.00) que el usuario define para un mes calendario. Hay uno por usuario y mes; solo se crea o edita el del mes en curso, y no se elimina. | `Budget` | M4, DEC-07, DEC-08, RN-118, RN-119 |
| 12 | **Mes** | Mes calendario según la zona horaria `America/Lima`, expresado como `YYYY-MM`. Es el periodo de cálculo del MVP. | `month` (JSON) | M4, M6 |
| 13 | **Total de gastos del mes** | Suma de los gastos del usuario cuya fecha pertenece al mes. | `totalExpenses` (JSON) | M6, RN-023 |
| 14 | **Total de ingresos del mes** | Suma de los ingresos del usuario cuya fecha pertenece al mes. | `totalIncome` (JSON) | M6 |
| 15 | **Porcentaje consumido** | (Total de gastos del mes ÷ presupuesto del mes) × 100, redondeado a 2 decimales. Solo existe si hay presupuesto. Puede superar 100.00. Se muestra como «82.50 %». | `consumedPercentage` (JSON) | M4, RN-024, RN-124 |
| 16 | **Restante del presupuesto** | Presupuesto del mes − total de gastos del mes. Internamente conserva su signo; en la UI se rotula «Restante del presupuesto» y se muestra con S/ 0.00 como mínimo. No se muestra si no hay presupuesto. | `RemainingBudget`, `remainingBudget` (JSON) | M4, RN-026, RN-071, RN-122 |
| 17 | **Saldo calculado** | Término genérico para el saldo del mes y el saldo acumulado: dinero según los registros del usuario, calculado por el backend. **No es un saldo bancario.** | — | M5, PRINCIPIO-009 |
| 18 | **Umbral** | Porcentaje consumido redondeado a partir del cual se muestra un aviso: 80 % y 100 %. | `BudgetThreshold` | M5, RN-120 |
| 19 | **Aviso de umbral** | Mensaje textual del dashboard que indica que se alcanzó el 80 % o el 100 % del presupuesto. Lo decide el backend, no la IA. No aparece sin presupuesto. | `BudgetAlert` (`Threshold80`, `Threshold100`); `alert` en la API | M5, RN-028 a RN-031, RN-121, RN-127, ADR-004 |
| 20 | **Dashboard** | Pantalla principal con saldo del mes, saldo acumulado, presupuesto, restante del presupuesto, porcentaje consumido, estado del presupuesto, gasto por categoría, últimos movimientos, avisos de umbral y el botón del resumen de IA. | `DashboardController` | M5 |
| 21 | **Resumen explicativo** | Texto en español sencillo que redacta la IA local, a pedido del usuario, a partir de cifras ya calculadas. No es asesoría financiera. Se muestra como texto plano y una sola vez en el dashboard; no se guarda (RN-165). Se descarta si contiene una cifra no válida (RN-166). | `MonthlySummary` | M6, RN-128, RN-165, RN-166 |
| 22 | **Datos agregados** | Totales del mes en curso y por categoría, más el restante y el exceso del presupuesto, sin movimientos individuales ni datos personales. Es lo único que recibe la IA como datos del usuario. | `totalsByCategory`, `remainingBudget`, `overspentAmount` (JSON) | PRINCIPIO-005, M6, RN-034 |
| 23 | **IA local** | Inferencia ejecutada con Ollama en un host de la lista cerrada de ADR-002 (`localhost`, `127.0.0.1`, `::1`), con el modelo `qwen3:4b` (ADR-003). Ningún dato sale a proveedores externos. | `OllamaClient` | PRINCIPIO-005, RN-087 |
| 24 | **Degradación elegante** | Comportamiento por el cual, si la IA no está disponible (Ollama no responde o devuelve un texto no válido) o está deshabilitada (configuración inválida), M1 a M5 siguen funcionando y solo el resumen muestra un mensaje. | — | M6, RN-038, RN-039, RN-088, RN-132 |
| 25 | **Cálculo determinista** | Cálculo financiero hecho por reglas fijas en la capa de lógica de negocio del backend, que siempre produce el mismo resultado con los mismos datos. | `BudgetService`, `BalanceCalculator` | PRINCIPIO-004 |
| 26 | **Modo de redondeo** | Constante única y compartida que define el `MidpointRounding` de todo redondeo a 2 decimales de resultados de cálculo: `AwayFromZero` (redondeo comercial). Nunca se aplica a montos ingresados. | `MoneyRounding.Mode` | PRINCIPIO-003, DEC-03 |
| 27 | **Datos ficticios** | Usuarios y movimientos inventados, generados por un script del repositorio. Son los únicos datos permitidos en el MVP. | — | PRINCIPIO-006 |
| 28 | **Seed** | Script reproducible que crea los usuarios demo (`demo1@expenseflow.test`, …) con sus movimientos ficticios. También restablece las cuentas demo (no hay recuperación de contraseña, W17). Solo crea, elimina o recrea datos de `demo1` a `demo6`; nunca lee ni modifica otros usuarios (RN-158). | `DemoSeeder` | PRINCIPIO-006, RN-138, RN-157 a RN-159 |
| 29 | **Bloqueo de cuenta (lockout)** | Bloqueo de inicio de sesión durante 5 minutos tras 5 intentos fallidos consecutivos, mediante ASP.NET Core Identity. Cada intento fallido incrementa el contador (RN-144); el contador se reinicia con un inicio de sesión exitoso. | `LockoutEnabled`, `MaxFailedAccessAttempts`, `DefaultLockoutTimeSpan`, `lockoutOnFailure` | PRINCIPIO-008, DEC-09, RN-144 |
| 30 | **Entrega de dinero** | Momento en que el estudiante recibe su dinero. Es el momento detonante del producto; como periodo de cálculo es Should (S2). | `PayDay` (Should) | Brief 3.3, S2 |
| 31 | **Meta de ahorro** *(Should)* | Objetivo con nombre y monto (p. ej., «Laptop — S/ 2,500.00») cuyo avance se calcula con montos marcados como ahorro. | `SavingsGoal` (Should) | S4 |
| 32 | **Origen del ingreso** | De dónde proviene un ingreso. Lista cerrada: Apoyo familiar, Trabajo, Prácticas preprofesionales, Trabajo independiente, Otros. | `IncomeSource` (`FamilySupport`, `Job`, `Internship`, `Freelance`, `Other`) | DEC-13, RN-059 |
| 33 | **Saldo del mes** | Total de ingresos del mes − total de gastos del mes. Indicador que se presenta primero. | `MonthlyBalance`, `monthlyBalance` (JSON) | DEC-05, RN-074, RN-126 |
| 34 | **Saldo acumulado** | Total de ingresos registrados − total de gastos registrados, de todo el historial. | `CumulativeBalance`, `cumulativeBalance` (API) | DEC-05, RN-075 |
| 35 | **Presupuesto excedido** | Estado que se da cuando el total de gastos del mes **supera** el presupuesto del mes (se comparan importes, no porcentajes). | `BudgetStatus.Exceeded` | DEC-06, RN-072, RN-120 |
| 36 | **Exceso** | Valor absoluto de (presupuesto del mes − total de gastos del mes) cuando hay presupuesto excedido. | `OverspentAmount`, `overspentAmount` (JSON) | DEC-06, RN-073 |
| 37 | **Presupuesto no configurado** | Estado de un mes sin presupuesto. No se calcula el porcentaje consumido; no se muestran restante, estado, exceso ni avisos. | `BudgetStatus.NotConfigured` | DEC-07, RN-066 a RN-070, RN-121, RN-122 |
| 38 | **Últimos movimientos** | Los 5 movimientos más recientes del usuario en el dashboard, ordenados por fecha del movimiento, fecha de creación e `Id`, todos descendentes. | `DashboardRecentTransactionsLimit` | DEC-15, RN-078, RN-079 |
| 39 | **Tiempo de espera de IA** | Tiempo máximo de una solicitud a Ollama antes de cancelarla, sin reintentos. Por defecto 30 s; rango válido de 5 a 120 s. | `Ai:TimeoutSeconds` | DEC-10, RN-082, RN-134 |
| 40 | **IA deshabilitada** | Estado en que el sistema no envía solicitudes de inferencia porque `Ai:BaseUrl` falta, es inválida o su host no está en la lista de ADR-002, o porque `Ai:Model` está vacío. Se decide al arrancar. Un modelo no descargado no deshabilita la IA: hace fallar la solicitud (RN-038). | `Ai:BaseUrl`, `AiOptions.AllowedHosts` | DEC-12, RN-086 a RN-088, RN-132 |
| 41 | **Frecuencia de entrega** *(Should)* | Cada cuánto recibe dinero el usuario: cada 7 días, cada 15 días o mensual. En la frecuencia mensual, si el día elegido no existe en el mes, se usa el último día de ese mes. | `PayFrequency` | DEC-14, RN-089 a RN-093 |
| 42 | **Tipo de movimiento** | Indica si un movimiento es gasto o ingreso. Lo determina el formulario usado y no se puede cambiar después. En la UI se muestra en texto («Gasto», «Ingreso»). | `TransactionType` | RN-056, RN-080, RN-112 |
| 43 | **Fecha del movimiento** | Día en que ocurrió el movimiento, en hora de Lima. No puede ser futura ni anterior a la ventana de registro. Se muestra como `dd/MM/yyyy`. | `TransactionDate` (`DateOnly`, columna `date`) | RN-017, RN-057, RN-109, RN-125 |
| 44 | **Fecha de creación** | Instante en que se guardó el movimiento, en UTC. Desempata el orden de los movimientos y define el «último gasto creado». | `CreatedAt` (`DateTime` UTC, columna `datetime2`) | RN-018, RN-079, RN-159 |
| 45 | **Descripción del origen** | Texto opcional de hasta 100 caracteres que solo existe en ingresos con origen «Otros». Se descarta si cambia el origen. | `SourceDescription` | RN-060, RN-061, RN-110 |
| 46 | **Estado del presupuesto** | Situación del presupuesto del mes en el dashboard: «No configurado», normal o «Presupuesto excedido». El estado normal no muestra texto de estado. | `BudgetStatus` (`NotConfigured`, `Normal`, `Exceeded`); `budgetStatus` en la API | RN-068, RN-072, ADR-004 |
| 47 | **Editar presupuesto** | Acción que modifica el monto del presupuesto del mes en curso sin crear otro registro. | — | RN-065, RN-118 |
| 48 | **Lista de movimientos** | Pantalla con todos los movimientos del usuario, en páginas de 20, desde la que se corrigen y eliminan. No tiene filtros (los filtros son S3). | `TransactionListPageSize` | RN-117, DEC-19 |
| 49 | **Eliminación de movimiento** | Borrado físico y definitivo de un movimiento, con confirmación previa. No existe la anulación. | — | RN-114, RN-115, DEC-20 |
| 50 | **Formato de entrada de montos** | Texto que el usuario puede escribir como monto en un formulario: dígitos del 0 al 9 y, opcionalmente, un único separador decimal (punto o coma) seguido de 1 o 2 dígitos (p. ej., `12.50` o `12,50`). Sin separador de miles. | — | RN-104 |
| 51 | **Capa de lógica de negocio** | Código de los namespaces `ExpenseFlow.Modules.*.Domain` y `ExpenseFlow.Modules.*.Application`. Es donde se hacen los cálculos oficiales y donde se mide la cobertura del 80 %. | — | PRINCIPIO-004, PRINCIPIO-011, la constitución |
| 52 | **Entorno** | Configuración de ejecución: `Development` (desarrollo local, servicios nativos), `Testing` (CI, SQL Server en Docker e IA simulada) y `Demo` (usabilidad y prueba en Android, servicios nativos). Todos usan HTTPS; HSTS en todos salvo `Development`. | `ASPNETCORE_ENVIRONMENT` | `02-alcance.md` 3.4, ADR-004, RN-005, RN-100 |
| 53 | **Instrucciones del modelo** | Texto fijo y versionado que acompaña a los datos agregados en cada solicitud a Ollama. Define idioma, longitud y límites del resumen. | `src/ExpenseFlow/Modules/Insights/Prompts/monthly-summary.v1.md` | RN-129, RN-130, ADR-002 |
| 54 | **Saldo del periodo** *(Should)* | Ingresos del periodo de entrega − gastos del periodo de entrega. Se muestra además de los saldos mensual y acumulado. | `PeriodBalance` | RN-136 |
| 55 | **Mensaje de la UI** | Texto exacto que el sistema muestra al usuario, identificado como MSG-NN en la sección 10 de `03-reglas-de-negocio.md`. | — | RN-116 |
| 56 | **Crear presupuesto** | Acción que registra el presupuesto del mes en curso cuando aún no existe. Si ya existe, se usa «Editar presupuesto». | — | RN-022, RN-064, RN-070, RN-139, RN-167 |
| 57 | **Reloj del sistema** | Única fuente de la fecha y hora actual: `TimeProvider`, convertido a `America/Lima`. En las pruebas se fija con `FakeTimeProvider`. | `TimeProvider`, `LimaClock` | RN-137, ADR-004 |
| 58 | **Cuentas demo** | Usuarios ficticios `demo1@expenseflow.test` a `demo6@expenseflow.test` que crea el seed, cada uno con un escenario fijo (en curso, excedido, sin presupuesto, vacía, límite de redondeo, paginación). Las pruebas E2E que modifican datos no las usan (`03` 11.4). Sus contraseñas están versionadas y no se consideran secretos. | `DemoSeeder` | RN-138, `03` sección 11, ADR-005 |
| 59 | **Módulo** | Parte del monolito con una responsabilidad propia: `Accounts`, `Transactions`, `Budgets`, `Dashboard`, `Insights` y `Shared`. Cada uno tiene `Domain`, `Application`, `Infrastructure` y `Web`. | `ExpenseFlow.Modules.<Módulo>` | ADR-004, PRINCIPIO-011 |
| 60 | **Evaluaciones de IA** | 10 escenarios fijos que se ejecutan localmente con Ollama real antes de un PR que cambie la integración con IA. Se aprueban con 10 de 10. | `ExpenseFlow.AiEvals`, `ai-eval-report.md` | ADR-003, RN-130 |
| 61 | **Aprobación** | Paso de un documento o ADR al estado `Aprobado` por un humano. Es vigente cuando el PR que lo registra se fusiona en `main`. | — | ADR-005 |
| 62 | **Contrato de la API** | Entradas, salidas, códigos HTTP y objetos de respuesta de cada endpoint REST. | `TransactionDto`, `BudgetDto`, `DashboardDto` | ADR-004 sección 3 |
| 63 | **Cifra** (en el resumen de IA) | Número del resumen que va precedido de «S/», seguido de «%» o con parte decimal. Su valor debe coincidir con el valor absoluto de un número del JSON o con un valor permitido. Se verifica en las evaluaciones y también en ejecución (RN-166). | `FigureVerifier` | ADR-003 sección 3, RN-130, RN-166 |
| 64 | **Índice de documentación** | Archivo que lista los documentos del Spec Pack con su versión, estado y propósito, y los relaciona con lo que exige PRINCIPIO-012. | `/docs/README.md` | PRINCIPIO-012 |
| 65 | **Mes en curso** | Mes calendario que contiene la fecha actual de `TimeProvider` en `America/Lima`. | `LimaClock` | RN-022, RN-137 |
| 66 | **Ventana de registro** | Los 12 meses calendario en los que se puede registrar o modificar un movimiento: el mes en curso y los 11 anteriores. | — | RN-109 |
| 67 | **Sesión** | Periodo durante el cual la cookie de autenticación es válida. Expira entre 30 y 60 minutos sin actividad y al cerrar el navegador, porque la cookie no es persistente. | — | RN-097, RN-098, RN-147 |
| 68 | **Token antiforgery** | Valor que acompaña a toda solicitud `POST`, `PUT` o `DELETE` para evitar envíos desde otros sitios. Está ligado a la sesión. | `X-CSRF-TOKEN` | RN-101 |
| 69 | **Flujos principales** | Los flujos F-01 a F-07 de `02-alcance.md` §3.3, cada uno con prueba E2E y revisión axe. | — | PRINCIPIO-011, PRINCIPIO-012 |
| 70 | **Gasto por categoría** | Total de gastos del mes en curso en cada categoría con importe mayor que S/ 0.00, ordenado de mayor a menor. | `totalsByCategory` | RN-033, RN-163 |
| 71 | **Incidente de IA** | Fallo de una solicitud a Ollama, registrado en el log técnico con los datos de RN-133. | — | RN-084, RN-133 |
| 72 | **Formato de correo** | `texto@texto.texto`: antes de la @, letras sin tildes ni ñ, dígitos, punto, guion y guion bajo; en el dominio, letras, dígitos, guion y punto. Máximo 256 caracteres. | — | RN-150, RN-151 |
| 73 | **Periodo de entrega** *(Should)* | Intervalo desde una entrega de dinero (incluida) hasta la siguiente (excluida). | `PayPeriod` | RN-090, RN-136 |
| 74 | **Día de entrega** *(Should)* | Día del mes elegido para la frecuencia mensual; si no existe en un mes, se usa el último día de ese mes. | `PayDayOfMonth` | RN-091 a RN-093 |
| 75 | **Fecha de última entrega** *(Should)* | Fecha que indica el usuario al configurar la frecuencia; es el inicio del primer periodo. | `LastPayDate` | RN-135, RN-143, RN-161 |

---

---

## 2. Términos de interfaz

| # | Término | Definición | En código / ubicación | Fuente |
|---|---|---|---|---|
| 76 | **Pantalla** | Vista de la aplicación identificada como P-NNN en el inventario de `09-ui-spec.md` §1.1. | `Modules/<Módulo>/Web/Views/` | `09` §1.1 |
| 77 | **Componente de UI** | Elemento de interfaz reutilizable identificado como CMP-NN (por ejemplo, la fila de movimiento). | `09` §1.2 | `09` §1.2 |
| 78 | **Crear cuenta** | Pantalla P-001 y acción de registrarse con correo, contraseña, confirmación y declaración de mayoría de edad. | `GET/POST /account/register` | RN-148 a RN-153 |
| 79 | **Iniciar sesión** | Pantalla P-002 y acción de acceder con correo y contraseña. | `GET/POST /account/login` | RN-050, RN-154, RN-155 |
| 80 | **Cerrar sesión** | Acción del layout autenticado (CMP-01) que elimina la cookie de autenticación. | `POST /account/logout` | RN-097, RN-177 |
| 81 | **Confirmar contraseña** | Segundo campo de contraseña del registro; debe ser idéntico al primero. | `ConfirmPassword` | RN-152 |
| 82 | **Dirección de retorno** | Ruta local a la que vuelve el usuario después de iniciar sesión. Viaja en un campo oculto. | `returnUrl` | RN-155, RN-174 |
| 83 | **Aviso de sesión expirada** | Texto MSG-48 que muestra el inicio de sesión cuando se llega a él porque la sesión expiró. | cookie de marca `ef_session`, `reason=expired` | RN-171 |
| 84 | **Mensaje de confirmación** | Texto breve con `role="status"` que confirma un guardado o una eliminación, una sola vez (MSG-49 a MSG-53). | TempData | RN-172 |
| 85 | **Resumen de errores** | Bloque al inicio del formulario con el título MSG-44 y un enlace a cada campo con error. | CMP-04 | RN-116 |
| 86 | **Error que no es de un campo** | Error que no se asocia a un campo (MSG-02, MSG-21, MSG-22): se muestra en un bloque propio con `role="alert"`. | `09` §3.0 | RN-050, RN-064, RN-167 |
| 87 | **Página de error inesperado** | Pantalla P-011 que muestra MSG-40 y el código `500`, sin detalles técnicos. | `/Error` | RN-164 |
| 88 | **Página no encontrada** | Pantalla P-012 que muestra MSG-45 y el código `404`, igual para rutas inexistentes y recursos ajenos. | — | RN-169 |
| 89 | **Página de formulario caducado** | Pantalla P-016 que muestra MSG-46 y el código `400` cuando el token antiforgery de un formulario no es válido. | — | RN-170 |
| 90 | **Doble envío** | Envío repetido de un mismo formulario. Se evita deshabilitando el botón mientras se envía. | `data-disable-on-submit` | RN-131, RN-168 |
| 91 | **Mosaico de opciones** | Grupo de botones de opción (`radio`) presentados como bloques táctiles de 56 px. | CMP-06 | `09` CMP-06 |
| 92 | **Insignia de tipo** | Texto «Gasto» o «Ingreso» con fondo neutro dentro de la fila de movimiento. | CMP-07 | RN-080 |
| 93 | **Barra de consumo** | Barra de progreso del porcentaje consumido del presupuesto, con valores ARIA. | CMP-08 | RN-141 |
| 94 | **Paginación** | Navegación «Anterior» / «Siguiente» de la lista de movimientos. | CMP-12, `page` | RN-117, RN-178 |
| 95 | **Design token** | Variable CSS con un valor de diseño (color, tipografía, espaciado). | `--ef-*` en `wwwroot/css/site.css` | `09` §4 |
| 96 | **Wireframe textual** | Esquema en texto de una pantalla a 360 px, con sus componentes, atributos y estados. | `09` §3 | `09` §3 |

---

## 3. Términos técnicos y de proceso

| # | Término | Definición | Fuente |
|---|---|---|---|
| 97 | **Monolito modular** | Una sola aplicación ASP.NET Core dividida en módulos con fronteras verificadas por pruebas de arquitectura. | ADR-004 §2 |
| 98 | **Post/Redirect/Get (PRG)** | Patrón por el que todo `POST` válido responde `303` hacia una página `GET`, para que recargar no reenvíe el formulario. | ADR-006 |
| 99 | **TempData** | Almacén de un solo uso entre dos solicitudes, guardado en una cookie cifrada con Data Protection. Lleva el resumen de IA y los mensajes de confirmación. | RN-165, RN-172 |
| 100 | **Data Protection** | Servicio de ASP.NET Core que cifra y firma cookies como la de TempData. | ADR-004 §1 |
| 101 | **Mejora progresiva** | La página funciona sin JavaScript; el JavaScript solo agrega comodidad (botón deshabilitado, campo «Descripción», diálogo de eliminación). | ADR-006 |
| 102 | **Limitación aceptada** | Comportamiento conocido que no se corrige en el MVP, con su decisión registrada (DEC-48, DEC-57, DEC-59, DEC-90). | `03` §9 |
| 103 | **`ProblemDetails`** | Formato JSON estándar de errores HTTP de la API. `ValidationProblemDetails` agrega el diccionario `errors` por campo. | ADR-004 §3.1 |
| 104 | **OpenAPI / Swagger** | Descripción de la API REST (OpenAPI) y su interfaz web de prueba (Swagger UI, solo en `Development`). | PRINCIPIO-012, ADR-001 |
| 105 | **Kestrel** | Servidor web de ASP.NET Core; acepta solo TLS 1.2 y TLS 1.3. | RN-156, ADR-004 §4 |
| 106 | **HTTPS / TLS / HSTS** | Tráfico cifrado (TLS ≥ 1.2) en todos los entornos; la cabecera HSTS obliga al navegador a usar HTTPS (salvo en `Development`). | RN-005, RN-100, RN-156 |
| 107 | **`mkcert`** | Herramienta local que genera el certificado HTTPS de confianza del entorno `Demo`. | ADR-001 |
| 108 | **Playwright** | Herramienta de pruebas E2E de los flujos F-01 a F-07. | ADR-001 |
| 109 | **axe / WCAG 2.2 AA** | Motor de revisión automática de accesibilidad y nivel de conformidad exigido. | PRINCIPIO-012, ADR-001 |
| 110 | **`Trait` de regla** | Atributo `[Trait("Rule", "RN-NNN")]` que identifica la regla que verifica una prueba. El CI exige uno por cada regla MUST de Cálculo. | PRINCIPIO-011, ADR-001 §3 |
| 111 | **Cuenta de prueba** | Cuenta `e2e-<GUID>@expenseflow.test` creada por una prueba que modifica datos; nunca se usa una cuenta demo para eso. | `03` §11.4 |
| 112 | **Migración** | Cambio versionado del esquema de base de datos con EF Core. Solo aditiva; nunca la aplica la aplicación al arrancar. | PRINCIPIO-010, ADR-004 §4.2 |
| 113 | **Checklist de revisión** | Lista de verificación que el revisor humano completa en los PR de cambios sensibles. | `/docs/checklist-revision.md` |
| 114 | **Spec Pack** | Conjunto de documentos `00` a `11`, los ADR, el checklist y el índice, versionados juntos. | `/docs/README.md` |
| 115 | **BMAD** | Marco de trabajo con agentes especializados por etapa (Discovery, Planning, Architecture, Implementation, Verification). | `/docs/README.md` |
| 116 | **EARS** | Notación de las reglas de negocio con cinco patrones. | `03` §0.2 |
| 117 | **Criterio GWT** | Criterio de aceptación con la forma DADO QUE / CUANDO / ENTONCES. | `06` |
| 118 | **INVEST** | Criterios de calidad de una HU: independiente, negociable, valiosa, estimable, pequeña y verificable. | `06` |
| 119 | **[Decisión] / [Hipótesis] / [Pendiente] / [Propuesta]** | Etiquetas de certeza: decidido; supuesto sin evidencia; dato que falta confirmar; propuesta de diseño de `09` que se puede cambiar sin tocar otros documentos. | `01`, `02`, `09` |
| 120 | **Tamaño de tarea** | Estimación de una tarea del backlog: S (< 30 min), M (30–60 min), L (1–2 h). | `08` |
| 121 | **Conventional Commits** | Convención de mensajes de commit y de nombres de rama (`feat`, `fix`, `docs`…), en inglés. | PRINCIPIO-001, ADR-005 §2 |
| 122 | **Ruta crítica** | Secuencia de tareas dependientes más larga del backlog; su duración es la mínima del MVP aunque se trabaje en paralelo. | `08` §4 |
| 123 | **Definition of Ready** | Condición para planificar una HU: cumple INVEST y sus reglas, mensajes y pantallas están especificados. | `06`, `05` PQ-07, PQ-08 |
| 124 | **MoSCoW (Must / Should / Could / Won't)** | Priorización del alcance: imprescindible para el MVP, después del MVP, deseable y fuera de esta versión. | `02` §0 |
| 125 | **xUnit / `WebApplicationFactory` / coverlet** | Marco de pruebas, servidor de pruebas de integración y E2E, y medidor de cobertura de líneas. | ADR-001 |
| 126 | **Razor / Bootstrap 5** | Motor de vistas de ASP.NET Core MVC y librería CSS servida localmente. | PRINCIPIO-002, ADR-006 |
| 127 | **ViewModel / DTO** | Objeto con los datos ya calculados y formateados que recibe una vista (ViewModel) o que viaja por la API (DTO). | `07` §4, ADR-004 §3.3 |
| 128 | **Loopback** | Interfaz de red local del propio equipo (`localhost`, `127.0.0.1`, `::1`); la única donde puede escuchar Ollama. | ADR-002 |
| 129 | **Punycode** | Codificación ASCII de nombres de dominio con caracteres como «ñ». El campo «Correo» usa `type="text"` para que el navegador no la aplique. | DEC-77 |
| 130 | **`DashboardCalculator` / `PayloadBuilder` / `OllamaClient` / `FigureVerifier`** | Servicios de código: cálculos del dashboard sin E/S; JSON de agregados para la IA; cliente HTTP de Ollama; verificador de cifras del resumen. | `07` ARQ-05, ARQ-06 |

---

## 4. Convención de identificadores

Cada prefijo tiene un único significado en todo el Spec Pack (DEC-79).

| Prefijo | Significado | Dónde se define |
|---|---|---|
| PRINCIPIO-0NN | Principio de la constitución | `00` |
| M1–M6, S1–S5, C1–C4, W1–W20 | Funcionalidad Must, Should, Could y Won't | `02` §1–2 |
| F-01 a F-07 | Flujo principal (con guion) | `02` §3.3, `09` §2 |
| U1–U4, T1–T4, F1–F5 | KPI de usabilidad, técnico y futuro (sin guion) | `01` §5 |
| R1–R6 · SU-1 a SU-9 | Riesgo y supuesto del Brief | `01` §6 |
| SP1–SP7 · ST1–ST5 · A1–A5 | Supuesto de producto, supuesto técnico y decisión del alcance | `02` |
| RN-NNN | Regla de negocio | `03` |
| MSG-NN | Mensaje de la UI | `03` §10 |
| DEC-NN | Decisión de producto | `03` §9 |
| PQ-NN | Pregunta abierta | `05` |
| HU-NNN | Historia de usuario | `06` |
| ARQ-NN · DP-N · NS-N · RC-N · RA-NN | Componente, principio de diseño, nota de seguridad, restricción y riesgo de arquitectura | `07` |
| ADR-NNN | Decisión de arquitectura | `/docs/adr/` |
| T-NNN | Tarea del backlog (con guion) | `08` |
| P-NNN | Pantalla | `09` §1.1 |
| CMP-NN | Componente de UI | `09` §1.2 |
| V-NN | Observación de validación del prototipo | `10` |
| AF-NN | Hallazgo de la auditoría final | `11` |

---

## 5. Términos a evitar

| Evitar | Usar en su lugar | Motivo |
|---|---|---|
| «Saldo disponible», «saldo de tu cuenta» | **Saldo del mes**, **saldo acumulado** o **saldo calculado** | El sistema no conoce saldos bancarios (PRINCIPIO-009). |
| «Disponible» como rótulo del presupuesto | **Restante del presupuesto** | Se confunde con un saldo bancario (RN-071). |
| «Cuenta» (para dinero) | **Movimientos** / **saldo calculado** | «Cuenta» se reserva para la cuenta de usuario; evita confusión con cuenta bancaria. |
| «Transacción» o «pago» en la UI | **Movimiento**, **gasto**, **ingreso** | «Pago» sugiere que el sistema procesa pagos (W2). `Transaction` solo se usa en el código. «Medio de pago» sí se usa, porque es la etiqueta descriptiva definida en el término 10. |
| «Recomendación», «asesoría» | **Resumen explicativo** | La IA solo explica; no asesora (W16). |
| «Notificación», «Notificaciones» (también como categoría de reglas) | **Aviso de umbral**, categoría **Avisos** | No hay notificaciones push, correo ni SMS (W14). |
| «Saldo», «total» o «porcentaje» calculado por la IA | — | Toda cifra oficial la calcula el backend (PRINCIPIO-004). |
| «Restante negativo», «S/ -50.00» en el presupuesto | **Restante del presupuesto: S/ 0.00** + **Exceso** | El restante nunca se muestra negativo (DEC-06). |
| «S/ -45.00», «(S/ 45.00)» en un saldo | **-S/ 45.00** + texto explicativo | Formato único de negativos (RN-123). |
| «Movimiento anulado», «anular» | **Eliminar movimiento** | No existe la anulación; la eliminación es física (DEC-20, RN-115). |
| «Quincenal» | **Cada 15 días** | En Perú «quincenal» suele significar los días 15 y fin de mes, que es otro modelo (RN-089). |
| «Movimientos recientes» | **Últimos movimientos** | Término unificado (término 38). |
| «Recordarme», «mantener la sesión iniciada» | — | No existe la sesión persistente (W20, RN-146). |
| «Guardar» a secas en formularios de movimientos | **Guardar gasto**, **Guardar ingreso**, **Guardar cambios** | El botón dice qué se guarda (`09` CMP-10). |
| «Error 400», «Bad Request» en la UI | MSG-46 | La UI no muestra códigos ni textos en inglés (RN-170). |
