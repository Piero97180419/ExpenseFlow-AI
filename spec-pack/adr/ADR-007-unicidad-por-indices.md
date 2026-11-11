# ADR-007 — Unicidad del presupuesto mensual y del correo garantizada por índices únicos

| Campo | Valor |
|---|---|
| Estado | **Aprobado** |
| Versión | v1.0 |
| Fecha | 2026-11-10 |
| Aprobado por | Piero Carbajal |
| Fecha de aprobación | 2026-11-10 |
| PR de aprobación | PR que fusiona el Spec Pack v1.0 en `main` (ADR-005 §1) |
| Principios afectados | PRINCIPIO-003, PRINCIPIO-007, PRINCIPIO-008 |
| Relacionado con | RN-064, RN-095 · `07-architecture.md` ARQ-02, ARQ-04 · `08-backlog.md` T-008, T-018, T-029 |

## Contexto

- RN-064 exige un solo presupuesto por usuario y mes. HU-007 exige que dos `POST /api/budgets` simultáneos produzcan un `201` y un `409` con MSG-21, y que exista un solo registro.
- RN-095 exige que no haya dos cuentas con el mismo correo, sin distinguir mayúsculas. `User.RequireUniqueEmail` de Identity solo lo comprueba en la aplicación, y el índice `EmailIndex` que crea Identity **no es único**, así que dos registros simultáneos podrían crear dos cuentas.
- Una comprobación previa en el código no basta: dos solicitudes concurrentes pueden pasarla a la vez.

## Decisión

1. **Presupuesto:** índice `UNIQUE (UserId, Year, Month)` en `Budgets` (`UX_Budgets_UserId_Year_Month`). `BudgetService.CreateAsync` consulta primero (caso normal) y, si `SaveChanges` falla por la violación del índice (`SqlException.Number` 2601 o 2627), lo traduce a `BudgetAlreadyExists` ⇒ `409` + MSG-21 en la API y MSG-21 en la vista.
2. **Correo:** en el registro, `UserName` = correo sin espacios exteriores. El índice único `UserNameIndex` de Identity, sobre `NormalizedUserName`, garantiza la unicidad en la base. La colisión se traduce a MSG-04.
3. La traducción de los códigos de error de SQL Server vive en `ExpenseFlow.Modules.Shared.Infrastructure`; la capa `Application` no conoce `SqlException`.
4. Pruebas de integración con solicitudes simultáneas contra SQL Server real (contenedor Docker en el CI, ADR-001), para ambos casos (T-018 y T-029 de `08-backlog.md`).

## Alternativas descartadas

- **Solo comprobación previa en código:** condición de carrera; ambas solicitudes pasan.
- **Transacción `SERIALIZABLE` o bloqueo explícito:** resuelve la carrera, pero añade riesgo de interbloqueos y complejidad sin ventaja sobre el índice.
- **`MERGE` / upsert:** convertiría el segundo `POST` en una edición silenciosa, contra HU-007, que exige `409`.
- **Índice único adicional sobre `NormalizedEmail`:** posible, pero redundante si `UserName` = correo.

## Consecuencias

- **Positivas:** la garantía está en la base de datos aunque cambie el código de aplicación; un único patrón para el presupuesto y el correo.
- **Negativas:** el servicio depende de los códigos de error de SQL Server; la dependencia queda encapsulada en `Shared.Infrastructure`.
