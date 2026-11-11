# Spec Pack — ExpenseFlow AI

| Campo | Valor |
|---|---|
| Documento | README.md (índice y manifiesto de `/docs`) |
| Versión del Spec Pack | v1.0 |
| Fecha | 2026-11-10 |
| Estado | **Aprobado** |
| Aprobado por | Piero Carbajal |
| Fecha de aprobación | 2026-11-10 |
| PR de aprobación | PR que fusiona el Spec Pack v1.0 en `main` (ADR-005 §1) |

> **Para agentes de IA:** lee primero `00-constitucion.md`. Un documento está vigente si su versión aprobada está en `main` (ADR-005 §1). Ante una diferencia entre documentos, prevalecen la constitución, luego `03-reglas-de-negocio.md` y los ADR, y después el resto.

## 1. Documentos

| # | Archivo | Contenido | Agente BMAD | Versión | Estado |
|---|---|---|---|---|---|
| 00 | `00-constitucion.md` | 12 principios no negociables (PRINCIPIO-001 a PRINCIPIO-012) | Constitucional | v1.0 | Aprobado |
| 01 | `01-project-brief.md` | Problema, usuarios, propuesta de valor, KPIs, riesgos y supuestos | Analyst | v1.0 | Aprobado |
| 02 | `02-alcance.md` | Funcionalidades M/S/C/W, MVP, flujos F-01 a F-07, supuestos y dependencias | PM | v1.0 | Aprobado |
| 03 | `03-reglas-de-negocio.md` | 178 reglas EARS (RN-001 a RN-178), registro de decisiones (DEC-01 a DEC-90), catálogo de mensajes (MSG-01 a MSG-53) y datos de prueba | Business Analyst | v1.0 | Aprobado |
| 04 | `04-glosario.md` | Lenguaje ubicuo del dominio, de la interfaz y técnico; convención de identificadores | Business Analyst | v1.0 | Aprobado |
| 05 | `05-preguntas-abiertas.md` | Preguntas abiertas PQ-01 a PQ-09 (ninguna bloquea el MVP) | QA | v1.0 | Aprobado |
| 06 | `06-user-stories.md` | 18 HU (HU-001 a HU-018) con criterios DADO QUE / CUANDO / ENTONCES | Product Owner + QA Funcional | v1.0 | Aprobado |
| 07 | `07-architecture.md` | Overview, componentes ARQ-01 a ARQ-09, modelo de datos, contratos de API, ADR, restricciones y riesgos | Architect | v1.0 | Aprobado |
| 08 | `08-backlog.md` | 56 tareas T-001 a T-056 con dependencias, ruta crítica y matrices de trazabilidad | Backlog Builder + QA de Backlog | v1.0 | Aprobado |
| 09 | `09-ui-spec.md` | Pantallas P-001 a P-016, componentes CMP-01 a CMP-12, flujos, wireframes, tokens y mapa de navegación | UI Designer | v1.0 | Aprobado |
| 10 | `10-validation-log.md` | 34 observaciones del prototipo (V-01 a V-34) con su decisión | Consolidador de Validación | v1.0 | Aprobado |
| — | `10-anexo-reporte-reviewer.md` | Reporte completo del Agente Prototype Reviewer (OBS-01 a OBS-35), fuente de `10` | Prototype Reviewer | — | Fuente |
| 11 | `11-auditoria-spec-pack.md` | Auditoría final del Spec Pack | Spec Finalizer | v1.0 | Aprobado |
| — | `checklist-revision.md` | Checklist de revisión humana de los PR | — | v1.0 | Aprobado |
| — | `adr/ADR-001` a `adr/ADR-007` | Decisiones de arquitectura (ADR-008 reservado como ejemplo de la constitución) | Architect | v1.0 | Aprobado |

## 2. Decisiones de arquitectura

| ADR | Archivo | Decisión |
|---|---|---|
| ADR-001 | `adr/ADR-001-herramientas-desarrollo-prueba-ci.md` | Paquetes del producto, herramientas de prueba y CI, verificación de reglas de cálculo y E2E bloqueantes |
| ADR-002 | `adr/ADR-002-integracion-ollama.md` | Hosts de Ollama permitidos y configuración |
| ADR-003 | `adr/ADR-003-modelo-ia-y-evaluaciones.md` | Modelo `qwen3:4b`, evaluaciones y verificador de cifras |
| ADR-004 | `adr/ADR-004-arquitectura-y-entornos.md` | Monolito modular, contrato de la API y entornos |
| ADR-005 | `adr/ADR-005-gobierno-de-cambios.md` | Aprobaciones, ramas, migraciones, datos demo, licencias y dependencias |
| ADR-006 | `adr/ADR-006-renderizado-servidor-js-minimo.md` | Renderizado en servidor, PRG y JavaScript mínimo |
| ADR-007 | `adr/ADR-007-unicidad-por-indices.md` | Unicidad por índices en la base de datos |

## 3. Convención de identificadores

Cada prefijo tiene un único significado en todo el Spec Pack (DEC-79). La tabla completa está en `04-glosario.md` §4. Las más usadas:

| Prefijo | Significado |
|---|---|
| PRINCIPIO-0NN | Principio de la constitución |
| RN-NNN · MSG-NN · DEC-NN | Regla, mensaje y decisión (`03`) |
| PQ-NN | Pregunta abierta (`05`) |
| HU-NNN | Historia de usuario (`06`) |
| ARQ-NN · DP-N · ADR-NNN | Componente, principio de diseño y decisión de arquitectura (`07`, `adr/`) |
| T-NNN | Tarea del backlog (`08`) |
| P-NNN · CMP-NN | Pantalla y componente de UI (`09`) |
| V-NN | Observación de validación (`10`) |

## 4. Cómo se usa

1. **Implementar una tarea:** abrir la tarea T-NNN en `08`, leer su HU en `06`, sus reglas en `03`, su pantalla en `09` y su componente en `07`.
2. **Cambiar una especificación:** PR con el checklist (§7), nueva versión del documento y actualización de este índice. Una decisión nueva se registra como DEC-NN en `03` §9.
3. **Resolver una pregunta abierta:** seguir el procedimiento de `05` (encabezado).

## 5. Ejecución BMAD (prompts)

Cada documento se generó con el prompt de su agente según las guías de laboratorio (Clase 1: 00–05; Clase 2: 06–08; Clase 3: 09–11). Los prompts, con los documentos de entrada pegados completos, se conservan junto al Spec Pack en `prompts/NN-<documento>.xml`.
