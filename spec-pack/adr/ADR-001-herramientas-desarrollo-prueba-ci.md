# ADR-001 — Paquetes del stack de producto, herramientas de desarrollo, prueba y CI, y verificación de la calidad

| Campo | Valor |
|---|---|
| Estado | **Aprobado** |
| Versión | v1.0 |
| Fecha | 2026-11-10 |
| Aprobado por | Piero Carbajal |
| Fecha de aprobación | 2026-11-10 |
| PR de aprobación | PR que fusiona el Spec Pack v1.0 en `main` (ADR-005 §1) |
| Principios afectados | PRINCIPIO-002, PRINCIPIO-005, PRINCIPIO-010, PRINCIPIO-011, PRINCIPIO-012 |
| Relacionado con | `02-alcance.md` A1 y A3 · `07-architecture.md` §2.1 · `08-backlog.md` T-001, T-002, T-041 |

## Contexto

PRINCIPIO-002 define un stack cerrado y exige un ADR para cualquier tecnología o paquete que no figure en la lista. Sin embargo, PRINCIPIO-011 y PRINCIPIO-012 **exigen** herramientas que no aparecen en esa lista: xUnit, `WebApplicationFactory`, Playwright, axe, medición de cobertura y OpenAPI/Swagger. Como la constitución obliga al agente a detenerse ante un conflicto, sin esta decisión el proyecto no podría escribir su primera prueba.

Además, el stack necesita paquetes NuGet concretos (Identity con EF Core, proveedor de SQL Server, herramienta de migraciones) y librerías que se ejecutan en la aplicación (jQuery, OpenAPI/Swagger), que deben distinguirse de las herramientas que solo se usan para desarrollar y probar.

Por último, PRINCIPIO-011 exige el 100 % de las reglas de cálculo financiero y pruebas E2E de los flujos principales: hay que definir qué cuenta como «regla de cálculo» y cómo lo verifica el CI.

## Decisión

Se distinguen tres listas y se fija cómo el CI verifica la calidad:

1. **Stack de producto:** la lista de PRINCIPIO-002, sin cambios.
2. **Paquetes y librerías del stack de producto** (se ejecutan en la aplicación): sección 1.
3. **Herramientas de desarrollo, prueba y CI** (no forman parte del producto en ejecución): sección 2.
4. **Verificación de la calidad en el CI**: sección 3.

### 1. Paquetes y librerías del stack de producto

| Paquete / librería | Uso | Exigido por |
|---|---|---|
| `Microsoft.AspNetCore.Identity.EntityFrameworkCore` | ASP.NET Core Identity sobre EF Core | PRINCIPIO-002, PRINCIPIO-008 |
| `Microsoft.EntityFrameworkCore.SqlServer` | Proveedor de SQL Server para EF Core | PRINCIPIO-002 |
| `Microsoft.EntityFrameworkCore.Design` | Generación de migraciones. Solo en el proyecto web, con `PrivateAssets="all"` | PRINCIPIO-002 |
| `Microsoft.AspNetCore.OpenApi` | Documento OpenAPI | PRINCIPIO-012 |
| `Swashbuckle.AspNetCore.SwaggerUI` | Interfaz Swagger, publicada solo en `Development` (ADR-004) | PRINCIPIO-012 |
| Bootstrap 5, jQuery, jQuery Validation, jQuery Validation Unobtrusive | UI responsive y validación del lado del cliente incluidas en la plantilla MVC. Archivos estáticos servidos localmente desde `wwwroot/lib`, sin CDN | PRINCIPIO-002, PRINCIPIO-012 |

> **No se usa** `Microsoft.AspNetCore.Identity.UI`: sus páginas son Razor Pages (el stack es MVC con Razor Views) y publicarían pantallas fuera del alcance, como recuperar la contraseña o gestionar la cuenta (RN-145). Las vistas del módulo `Accounts` son vistas MVC propias (ADR-004).

### 2. Herramientas de desarrollo, prueba y CI

| Herramienta / paquete | Uso | Exigido por |
|---|---|---|
| .NET SDK **10 (LTS)**, fijado en `global.json` | Compilación y ejecución | PRINCIPIO-002 |
| `dotnet-ef` como herramienta local (`.config/dotnet-tools.json`), en la misma versión que EF Core | Crear y aplicar migraciones (ADR-004, sección 4.2) | PRINCIPIO-002, PRINCIPIO-010 |
| `xunit`, `xunit.runner.visualstudio`, `Microsoft.NET.Test.Sdk` | Pruebas unitarias y de integración | PRINCIPIO-011 |
| `Microsoft.AspNetCore.Mvc.Testing` | `WebApplicationFactory` para pruebas de integración y E2E | PRINCIPIO-011 |
| `Microsoft.Extensions.TimeProvider.Testing` | `FakeTimeProvider` para fijar la fecha y hora en las pruebas (RN-137) | PRINCIPIO-011 |
| `coverlet.msbuild` | Cobertura con umbral bloqueante (configuración más abajo) | PRINCIPIO-011 |
| `Microsoft.Playwright` y PowerShell (`pwsh`) | Pruebas E2E. `pwsh` ejecuta `playwright.ps1 install` para descargar los navegadores y el script de verificación de reglas de la sección 3 | PRINCIPIO-011 |
| `Deque.AxeCore.Playwright` | Pruebas de accesibilidad WCAG 2.2 AA dentro de Playwright | PRINCIPIO-012 |
| Docker | **Solo en CI**: contenedor de SQL Server Express para las pruebas de integración y E2E | PRINCIPIO-011 |
| GitHub Actions | CI bloqueante | PRINCIPIO-011 |
| Ollama **0.9.0 o superior** (instalación nativa; versión exacta registrada según ADR-003) | Ejecutar el modelo en `Development`, `Demo` y las evaluaciones de IA | PRINCIPIO-005, ADR-003 |
| `mkcert` (herramienta local, fuera del repositorio) | Certificado HTTPS de confianza para la red local en el entorno `Demo` (prueba T4 en Android) | PRINCIPIO-008, Alcance A3 |

> En el equipo de desarrollo, SQL Server Express y Ollama se instalan de forma nativa, sin Docker (ADR-004).

### 3. Verificación de la calidad en el CI

**Reglas de cálculo (PRINCIPIO-011, DEC-89):**

- «Regla de cálculo financiero» = toda regla de `03-reglas-de-negocio.md` con `Categoría: Cálculo` y prioridad MUST. Las reglas SHOULD de Cálculo entran en la verificación cuando su Should se implementa.
- Cada prueba que verifica una regla lleva `[Trait("Rule", "RN-NNN")]`, una vez por regla verificada.
- Un paso del CI (script `pwsh`) extrae esas reglas de `03-reglas-de-negocio.md` y falla si alguna no tiene al menos una prueba con su `Trait` (tarea T-041).
- Que la prueba cubra el ejemplo y el contraejemplo de la regla lo verifica el revisor con el checklist (§1).

**Pruebas E2E bloqueantes:** las pruebas E2E con Playwright de los flujos F-01 a F-07 (`02-alcance.md` §3.3) son condición bloqueante del CI. Su infraestructura está en ADR-004 §4.1.

### Configuración de la cobertura

PRINCIPIO-011 exige el 80 % de **líneas**, medido **solo** sobre la capa de lógica de negocio. Por defecto, coverlet exige el umbral también para ramas y métodos y lo mide sobre todos los ensamblados, así que la configuración debe ser explícita:

```
/p:CollectCoverage=true
/p:Threshold=80
/p:ThresholdType=line
/p:ThresholdStat=total
/p:Include="[ExpenseFlow*]ExpenseFlow.Modules.*.Domain.*%2c[ExpenseFlow*]ExpenseFlow.Modules.*.Application.*"
```

### Reglas de uso

- Las versiones de los paquetes NuGet se fijan en `Directory.Packages.props` (Central Package Management). No se usan rangos ni comodines.
- La imagen de SQL Server del CI se fija por **etiqueta exacta de actualización acumulativa** (por ejemplo, `mcr.microsoft.com/mssql/server:2022-CU<n>-ubuntu-22.04`, con `MSSQL_PID=Express`), nunca `latest`.
- Las acciones de GitHub Actions (`actions/checkout`, `actions/setup-dotnet`, etc.) se fijan por **SHA de commit**, con la versión en un comentario.
- Los paquetes de prueba solo se referencian desde proyectos de prueba (`tests/*`).
- Las pruebas de integración usan SQL Server Express. **No** se usa el proveedor InMemory de EF Core para probar reglas financieras, porque no reproduce `DECIMAL(18,2)` ni las restricciones `UNIQUE`.
- Agregar o actualizar un paquete, imagen o acción de estas listas requiere un PR con revisión humana (PRINCIPIO-010, PRINCIPIO-011). Un agente puede abrir ese PR (ADR-005 §7). **Agregar uno que no esté en las listas requiere un ADR nuevo, que solo redacta y aprueba un humano.**
- Todo paquete debe tener una licencia gratuita que permita el uso académico del proyecto: MIT, Apache 2.0, BSD o similar (ADR-005).
- El certificado y la clave de `mkcert` son secretos locales: no se versionan (PRINCIPIO-008).

## Alternativas descartadas

- Testcontainers (añade un paquete y una dependencia de la API de Docker).
- NUnit o MSTest (PRINCIPIO-011 nombra xUnit).
- `Microsoft.AspNetCore.Identity.UI` (Razor Pages y páginas fuera del alcance).

## Consecuencias

- **Positivas:** desaparece el conflicto entre PRINCIPIO-002, PRINCIPIO-011 y PRINCIPIO-012; el 100 % de las reglas de cálculo es verificable de forma automática; el CI bloquea los flujos principales rotos; las listas cerradas se mantienen explícitas y auditables; el CI mide la cobertura exactamente como la define PRINCIPIO-011; un agente puede preparar el proyecto con los paquetes del stack sin detenerse.
- **Negativas:** el CI necesita Docker y tarda más, porque ejecuta las E2E en cada PR; cada prueba de cálculo necesita su `Trait`. Las vistas de registro e inicio de sesión se escriben a mano, porque no se usa `Identity.UI`.
- **Supuesto:** el equipo de desarrollo es un PC x64 (ADR-004). El CI usa `ubuntu-latest` x64, compatible con la imagen de SQL Server.
