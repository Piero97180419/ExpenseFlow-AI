# ADR-002 — Integración con Ollama: hosts permitidos y configuración

| Campo | Valor |
|---|---|
| Estado | **Aprobado** |
| Versión | v1.0 |
| Fecha | 2026-11-10 |
| Aprobado por | Piero Carbajal |
| Fecha de aprobación | 2026-11-10 |
| PR de aprobación | PR que fusiona el Spec Pack v1.0 en `main` (ADR-005 §1) |
| Principios afectados | PRINCIPIO-004, PRINCIPIO-005 |
| Relacionado con | DEC-12, DEC-16 · RN-082 a RN-088, RN-132, RN-134 · `07-architecture.md` ARQ-06 · `08-backlog.md` T-034 |

## Contexto

PRINCIPIO-005 solo admite Ollama local, y W6 excluye la IA remota incluso como respaldo. Si la dirección de Ollama fuera libre, una «dirección privada» o una «dirección remota con HTTPS» la contradirían, y un nombre de host arbitrario puede resolverse por DNS a un equipo remoto. La lista de hosts debe ser cerrada.

El equipo de desarrollo ejecuta Ollama de forma nativa (ADR-004) y el CI usa respuestas simuladas, así que no hace falta un host de contenedor.

## Decisión

Se admite solo Ollama en loopback (DEC-16); no se admiten direcciones privadas ni remotas.

### Hosts permitidos (lista cerrada en código, no configurable)

| Host | Caso de uso |
|---|---|
| `localhost` | Ollama instalado de forma nativa en el equipo de desarrollo |
| `127.0.0.1` | Ídem, por IP IPv4 |
| `::1` | Ídem, por IP IPv6 |

- La lista vive en una constante (`AiOptions.AllowedHosts`), no en `appsettings`, para que una configuración no pueda ampliarla.
- El esquema puede ser `http` o `https`. El puerto es libre (por defecto, `11434`).
- Ollama escucha solo en la interfaz de loopback (`OLLAMA_HOST=127.0.0.1`), su valor por defecto: no es accesible desde la red local, ni siquiera en el entorno `Demo`.
- **Comparación:** se usa el tipo de host que obtiene `Uri` al analizar la URL. Si es un nombre (`UriHostNameType.Dns`), debe ser `localhost`, sin distinguir mayúsculas. Si es una IP (`UriHostNameType.IPv4` o `IPv6`), su valor (`IPAddress.Equals`) debe ser igual a `127.0.0.1` o a `::1`. No se compara el texto de `Uri.Host`, que para IPv6 incluye corchetes (`[::1]`). `127.0.0.2` y otras direcciones de loopback se rechazan.

### Configuración

| Clave | Valor por defecto | Rango válido | Si no es válido |
|---|---|---|---|
| `Ai:BaseUrl` | `http://localhost:11434` | URL absoluta `http`/`https` con host de la lista | IA deshabilitada y advertencia en el log |
| `Ai:TimeoutSeconds` | `30` | Entero de 5 a 120 | Se usa `30` y se registra una advertencia (RN-134) |
| `Ai:Model` | `qwen3:4b` (ADR-003) | Cualquier texto no vacío | Vacío o ausente: IA deshabilitada y advertencia en el log |

- La validación ocurre **al arrancar** la aplicación y solo comprueba lo que se puede verificar sin llamar a Ollama: la URL, el host, el rango del tiempo de espera y que el modelo no esté vacío. El resultado (habilitada o deshabilitada) no cambia hasta el siguiente arranque.
- Que el modelo **no esté descargado** en Ollama no se puede saber al arrancar sin contactar con Ollama (RN-085). Se detecta en la primera solicitud, cuando Ollama responde con error, y se trata como un **fallo de la solicitud** (RN-038, MSG-23), no como IA deshabilitada (MSG-24).
- La aplicación arranca siempre, aunque la IA quede deshabilitada o Ollama no responda (RN-085).

### Instrucciones del modelo (prompt)

- El prompt se compone de **instrucciones fijas** versionadas en el repositorio (`src/ExpenseFlow/Modules/Insights/Prompts/monthly-summary.v1.md`; módulo `Insights` de ADR-004) más el **JSON de datos agregados** de RN-034. El formato de la solicitud está en ADR-003, sección 1.
- Cualquier cambio en las instrucciones es un cambio de «integración con IA» y requiere la checklist de revisión y las evaluaciones de ADR-003 (PRINCIPIO-011).

## Alternativas descartadas

- **Dirección privada autorizada (por ejemplo, otro equipo de la red local):** contradice PRINCIPIO-005, que solo admite inferencia local.
- **Dirección remota con HTTPS y autenticación:** es IA remota (W6) y enviaría datos fuera del equipo.
- **Lista de hosts configurable en `appsettings`:** una configuración podría ampliarla sin ADR.
- **Comparar el texto de `Uri.Host`:** para IPv6 incluye corchetes (`[::1]`) y fallaría con formas equivalentes.

## Consecuencias

- **Positivas:** se cumple PRINCIPIO-005 sin excepciones; la validación es determinista y testeable; un modelo ausente no confunde «no configurado» con «no disponible»; la comparación de hosts no depende del formato de texto de `Uri.Host`.
- **Negativas:** no se puede usar un Ollama que corra en otro equipo de la red local ni en un contenedor con otro nombre de host. Para eso haría falta un ADR que modifique PRINCIPIO-005.
