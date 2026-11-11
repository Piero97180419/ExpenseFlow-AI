# ADR-005 — Gobierno de cambios: aprobaciones, ramas, migraciones, datos demo, licencias y dependencias

| Campo | Valor |
|---|---|
| Estado | **Aprobado** |
| Versión | v1.0 |
| Fecha | 2026-11-10 |
| Aprobado por | Piero Carbajal |
| Fecha de aprobación | 2026-11-10 |
| PR de aprobación | PR que fusiona el Spec Pack v1.0 en `main` (ADR-005 §1) |
| Principios afectados | PRINCIPIO-006, PRINCIPIO-008, PRINCIPIO-010, PRINCIPIO-011 |
| Relacionado con | RN-138, RN-158 · `/docs/checklist-revision.md` · `/docs/README.md` |

## Contexto

Un Spec Pack escrito con agentes necesita reglas verificables sobre cuándo una propuesta pasa a ser vigente, quién puede cambiar un estado y qué cambios quedan reservados a un humano. También hay que decidir si las contraseñas de las cuentas demo son secretos, qué convención de ramas y qué licencias se aceptan, cómo se trata la revisión en un equipo de una persona y qué dependencias puede proponer un agente.

El registro acepta correos de cualquier dominio (DEC-52), así que el seed no puede protegerse por dominio: debe limitarse a las cuentas demo.

## Decisión

### 1. Registro de aprobación

- Todo ADR y todo documento de `/docs` tiene un campo **Estado**: `Propuesto`, `Aprobado`, `Rechazado` o `Reemplazado`.
- Un ADR aprobado indica **Aprobado por**, **Fecha de aprobación** y **PR de aprobación**.
- Una aprobación es **vigente cuando el PR que cambia el estado a `Aprobado` se fusiona en `main`**. Por tanto, **un documento está vigente si su versión aprobada está en la rama `main`**. Un agente comprueba la vigencia leyendo el documento desde `main`, no desde otra rama.
- El campo «PR de aprobación» es un dato de **trazabilidad**: se completa con el número del PR en ese mismo PR, antes de fusionarlo. Si en `main` todavía dice «Pendiente», la aprobación sigue siendo vigente, y el revisor corrige el dato en el siguiente PR de documentación (checklist §7).
- Los documentos que no tienen el campo «PR de aprobación» (Brief, alcance, glosario, checklist, índice) siguen la misma regla de vigencia: rigen las versiones que están en `main`.
- Un agente **no implementa** nada que dependa de un documento o regla en estado `Propuesto`.
- Solo un humano cambia un estado a `Aprobado` (PRINCIPIO-010).

### 2. Convención de ramas

- Formato: `<tipo>/<descripción-corta>`, donde `<tipo>` es uno de los tipos de Conventional Commits: `feat`, `fix`, `docs`, `test`, `refactor`, `perf`, `build`, `ci`, `chore`, `style`.
- Ejemplos: `feat/budget-edit`, `docs/adr-008-charts`, `test/rn-120-rounding`.
- Ningún cambio llega a `main` sin PR (PRINCIPIO-010).

### 3. Migraciones destructivas

- **Ningún agente de IA genera** migraciones que eliminen tablas, columnas, índices con datos o filas (`DropTable`, `DropColumn`, `DeleteData`, `Sql("DELETE …")`, etc.), ni las propone en un PR.
- Si un cambio la necesita, el agente se detiene y lo solicita a un humano (PRINCIPIO-010).
- Una migración destructiva la **escribe y la aplica un humano**, con un ADR aprobado que explique qué datos se pierden y por qué.

### 4. Credenciales de las cuentas demo y alcance del seed

- Las contraseñas de las cuentas demo del seed (`demo1@expenseflow.test`, …) **no se consideran secretos** en el sentido de PRINCIPIO-008: protegen solo datos ficticios y deben ser conocidas por los participantes de las sesiones de usabilidad.
- Se versionan en el script del seed y en el README raíz.
- El seed solo crea, elimina o recrea datos de las cuentas `demo1@expenseflow.test` a `demo6@expenseflow.test`, identificadas por su correo exacto normalizado, y **nunca lee, modifica ni elimina** datos de otros usuarios (RN-158). Así puede ejecutarse aunque la base contenga cuentas creadas por participantes o por pruebas, sin alterarlas.(DEC-53).
- Esta excepción no se extiende a ninguna otra credencial: la cadena de conexión, los certificados y cualquier otra clave siguen siendo secretos.

### 5. Licencias

- El repositorio **no tiene licencia de código abierto** (sin archivo `LICENSE`): es un trabajo académico, y nadie más tiene permiso para reutilizar el código.
- Un paquete, modelo o herramienta se acepta si su licencia es gratuita y permite el uso académico del proyecto. Se aceptan sin análisis adicional MIT, Apache 2.0 y BSD. Cualquier otra licencia (por ejemplo, términos de uso de un modelo) se revisa en el ADR que introduce el componente.

### 6. Revisión humana en un equipo de una persona

- El único integrante es a la vez autor (o supervisor del agente) y revisor del PR. La checklist de revisión es válida para PRINCIPIO-011, pero **no es una revisión independiente**.
- Esta limitación se declara en el Brief (`01` §7) y se acepta para el MVP académico.

### 7. Dependencias que un agente puede proponer (PRINCIPIO-010, DEC-50)

- Un agente puede abrir un PR que **agregue o actualice** un paquete, imagen, acción o herramienta que ya figure en ADR-001 o en otro ADR aprobado. La revisión humana y el checklist (§5) siguen siendo obligatorios.
- Un agente **no puede** agregar nada que no figure en esas listas, ni modificar las listas. Si lo necesita, se detiene y lo solicita a un humano, que redacta el ADR.

## Alternativas descartadas

- **Aprobación por mensaje o reunión, fuera del repositorio:** no es verificable por un agente.
- **Prohibir a los agentes cualquier cambio de dependencias:** obligaría a un humano a hacer incluso las actualizaciones de versión de paquetes ya aprobados.
- **Seed protegido por dominio de correo:** se bloquearía en cuanto un participante se registrara con su correo.

## Consecuencias

- **Positivas:** cualquier agente puede verificar si una regla está vigente; las migraciones destructivas quedan siempre en manos humanas; la excepción de las credenciales demo es explícita y acotada; el seed puede ejecutarse antes de cada sesión aunque existan otras cuentas.
- **Negativas:** cada aprobación requiere un PR propio; las migraciones destructivas son más lentas de introducir. El seed ya no detecta por sí mismo una base con datos reales: esa protección depende de PRINCIPIO-006 y de que solo se ejecute en los entornos del proyecto.
