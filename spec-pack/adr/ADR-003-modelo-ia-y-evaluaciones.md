# ADR-003 — Modelo de IA y evaluaciones con Ollama real

| Campo | Valor |
|---|---|
| Estado | **Aprobado** |
| Versión | v1.0 |
| Fecha | 2026-11-10 |
| Aprobado por | Piero Carbajal |
| Fecha de aprobación | 2026-11-10 |
| PR de aprobación | PR que fusiona el Spec Pack v1.0 en `main` (ADR-005 §1) |
| Principios afectados | PRINCIPIO-004, PRINCIPIO-005, PRINCIPIO-011 |
| Relacionado con | `02-alcance.md` A2 · RN-034, RN-129, RN-130, RN-166 · `07-architecture.md` ARQ-06 · `08-backlog.md` T-035 a T-039 |

## Contexto

M6 (resumen explicativo) es Must y es la única forma de medir U4. Sin un modelo elegido, `Ai:Model` no tenía valor y la IA quedaba deshabilitada. PRINCIPIO-011 exige además «un conjunto de evaluaciones con Ollama real», pero no definía los casos, el criterio de aprobación ni dónde se ejecutan.

Restricciones del equipo de desarrollo (ADR-004): PC x64 con 8 GB de RAM, sin GPU, con Ollama nativo. El CI (GitHub Actions gratuito) no puede ejecutar el modelo.

Además, hay que fijar el formato de la solicitud a Ollama y evitar que, en uso real, un resumen con cifras que no están en los datos llegue al usuario.

## Decisión

### 1. Modelo y solicitud

| Campo | Valor |
|---|---|
| Modelo | `qwen3:4b` (Ollama) |
| Tamaño aproximado | ~2.6 GB en disco (cuantización por defecto de Ollama) |
| Licencia | Apache 2.0 (compatible con ADR-005) |
| Modo de razonamiento | Desactivado: cada solicitud envía `"think": false`, para que la respuesta no incluya bloques de razonamiento y se mantenga dentro del tiempo de espera |
| Solicitud | `POST {Ai:BaseUrl}/api/generate` con el cuerpo `{ "model": <Ai:Model>, "prompt": <instrucciones de RN-129 + JSON de RN-034>, "stream": false, "think": false }`. El texto generado se lee de la propiedad `response`. No se fija un límite de tokens: la longitud la indican las instrucciones y la acota el tiempo de espera |
| Tiempo de espera | 30 s por defecto (RN-082, ADR-002) |
| Versión de Ollama | **0.9.0 o superior**: es la primera versión que admite el parámetro `think`. La versión exacta instalada (`ollama --version`) se registra en el README del módulo `Insights` |
| Fijación | Tras `ollama pull qwen3:4b`, el digest que muestra `ollama list` se registra en el README del módulo `Insights`. Si cambian el digest o la versión de Ollama, se repiten las evaluaciones |
| Defensa ante versiones antiguas | Si la respuesta incluye un bloque `<think>…</think>` (por ejemplo, con una versión de Ollama que ignora `think`), `OllamaClient` lo elimina antes de mostrar el texto. Si después de eliminarlo el texto queda vacío, es un fallo de la solicitud (RN-038) |


**Riesgo:** con 8 GB de RAM, el modelo comparte memoria con SQL Server Express, el navegador y el IDE. Si la latencia supera habitualmente los 30 s, se ajusta `Ai:TimeoutSeconds` (máximo 120 s) o se evalúa un modelo más pequeño mediante un ADR nuevo.

### 2. Evaluaciones con Ollama real

| Aspecto | Decisión |
|---|---|
| Dónde se ejecutan | **Localmente**, en el equipo de desarrollo, antes de abrir el PR. No se ejecutan en el CI. |
| Cuándo son obligatorias | En todo PR que cambie la integración con IA: instrucciones del prompt (RN-129), payload (RN-034), cliente de Ollama, verificador de cifras, modelo o su digest. |
| Conjunto de casos | 10 escenarios fijos en `tests/ExpenseFlow.AiEvals/scenarios.json`, cada uno con un JSON de RN-034. Deben incluir al menos: sin presupuesto (`budget: null`), consumo < 80 %, consumo entre 80 % y 100 %, exactamente 100.00 %, presupuesto excedido, saldo del mes negativo, una sola categoría con gastos, seis categorías con gastos, mes sin ingresos y mes sin gastos. |
| Criterio de aprobación | **10 de 10** escenarios cumplen: toda **cifra** del resumen tiene el mismo valor que algún número del JSON de entrada (RN-130). La definición de «cifra» está más abajo. |
| Verificaciones informativas | Se registran pero no bloquean: idioma español, longitud ≤ 150 palabras y ausencia de recomendaciones de productos financieros (RN-129). |
| Ejecución | `dotnet run --project tests/ExpenseFlow.AiEvals` |
| Evidencia | El script genera `ai-eval-report.md` con el modelo, su digest, la fecha y el resultado de cada escenario. El reporte se adjunta al PR y la checklist lo verifica (sección 4). |

> El proyecto `ExpenseFlow.AiEvals` es una aplicación de consola en C# que usa el mismo cliente `OllamaClient` y el mismo verificador `FigureVerifier` del producto. No necesita paquetes nuevos.

### 3. Qué es una «cifra» y cómo se verifica

El verificador (`FigureVerifier`) sigue estos pasos sobre el texto del resumen, en orden:

1. **Detección.** Es cifra toda secuencia de dígitos que cumpla al menos una condición:
   - está precedida por «S/» (con o sin espacio): `S/ 495.00`, `S/495`;
   - está seguida por «%» (con o sin espacio): `82.50 %`, `82,5%`;
   - tiene parte decimal: `495.00`, `82,50`.

   Un número entero sin «S/», sin «%» y sin decimales **no es cifra** y se ignora: «3 categorías», «el día 15», «2026».
2. **Normalización.** En una cifra, la coma seguida de exactamente 3 dígitos y de más dígitos o del final de la cifra se trata como separador de miles (`1,250.50` → 1250.50); cualquier otra coma es separador decimal (`82,50` → 82.50).
3. **Comparación.** La cifra es válida si su valor, con 2 decimales, es igual al **valor absoluto** de cualquier número del JSON de entrada. El signo se ignora porque la detección no lo captura y el modelo puede expresarlo con palabras («un déficit de S/ 150.00»). También son válidos estos **valores permitidos**, aunque no estén en el JSON:
   - `80` y `100`, solo cuando van seguidos de «%» (los umbrales de RN-028 y RN-029);
   - `0`, con «S/» o con «%».
4. **Resultado.** El texto es válido si todas sus cifras son válidas. En las evaluaciones, el reporte lista cada cifra no válida con su posición en el texto.

**Ejemplos con el JSON de RN-034** (`totalExpenses` 495.00, `budget` 600.00, `consumedPercentage` 82.50, `remainingBudget` 105.00):

| Texto del resumen | Resultado |
|---|---|
| «Gastaste S/ 495.00 de tus S/ 600.00 (82,50 %).» | Válido: 495.00, 600.00 y 82.50 están en el JSON. |
| «Superaste el 80 % de tu presupuesto en noviembre de 2026.» | Válido: «80 %» es un valor permitido y «2026» no es cifra. |
| «Te quedan unos S/ 100.» | No válido: «S/ 100» es cifra, no está en el JSON y el 100 solo se permite seguido de «%». |
| «Te quedan S/ 105.00 de presupuesto.» | Válido: 105.00 es `remainingBudget`. |
| Con `monthlyBalance: -150.00`: «Tu saldo del mes es de -S/ 150.00, un déficit de S/ 150.00.» | Válido: 150.00 es el valor absoluto de -150.00. |

**Uso en ejecución (DEC-55).** El verificador vive en `ExpenseFlow.Modules.Insights.Domain` y lo usan el producto y `ExpenseFlow.AiEvals`. En el producto, un resumen con alguna cifra no válida se trata como fallo de la solicitud (RN-166): el usuario ve MSG-23 y el log registra `type=InvalidFigure` (RN-133), sin incluir la cifra rechazada. No se reintenta (RN-083).

**Limitación aceptada.** No se detectan las cifras escritas con palabras («quinientos soles», «ochenta por ciento») ni los enteros sin «S/», sin «%» y sin decimales («495 soles»). Las instrucciones (RN-129) piden usar las cifras tal como aparecen en el JSON, y RN-043 impide que el texto se use como cifra oficial.

## Alternativas descartadas

- `gemma3:4b` (licencia Gemma Terms of Use, con restricciones de uso).
- `llama3.2:3b` (más ligero, pero con licencia comunitaria propia y menor calidad en español en tareas de redacción).
- `/api/chat` (no aporta nada para una única interacción sin historial).
- Ejecutar las evaluaciones en el CI: GitHub Actions gratuito no puede ejecutar el modelo.

## Consecuencias

- **Positivas:** M6 es implementable y U4 medible; la verificación de RN-130 deja de depender del criterio del revisor; en uso real, un resumen con cifras inventadas no llega al usuario.
- **Negativas:** las evaluaciones no son automáticas en el CI. Su cumplimiento depende de que el reporte se adjunte al PR, lo que se controla con la checklist. Algunos resúmenes se descartan en ejecución, lo que puede reducir la cantidad de resúmenes mostrados durante las sesiones de U4.
