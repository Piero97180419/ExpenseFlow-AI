# 09 — UI Specification — ExpenseFlow AI

| Campo | Valor |
|---|---|
| Documento | 09-ui-spec.md |
| Fase | Solución (BMAD — Agente UI Designer) |
| Versión | v1.0 |
| Fecha | 2026-11-10 |
| Estado | **Aprobado** |
| Aprobado por | Piero Carbajal |
| Fecha de aprobación | 2026-11-10 |
| PR de aprobación | PR que fusiona el Spec Pack v1.0 en `main` (ADR-005 §1) |
| Insumos | `00-constitucion.md`, `02-alcance.md`, `03-reglas-de-negocio.md`, `06-user-stories.md`, `07-architecture.md`, `10-validation-log.md` v1.0, ADR-006 v1.0 |

> **Convenciones**
> - Los textos entre «comillas latinas» son **literales** tomados de las HU o de la arquitectura; se copian tal cual a la vista.
> - Los mensajes se citan por su ID (`MSG-NN`). El texto exacto vive en `03-reglas-de-negocio.md` §10; la vista lo obtiene del catálogo `Messages` (`Shared.Application`) y nunca lo escribe a mano.
> - **[Propuesta]**: decisión de diseño de este documento (texto de apoyo, layout, orden visual). Se puede cambiar en un PR sin tocar otros documentos.
> - Identificadores: pantallas P-NNN y componentes CMP-NN (§1); principios de la constitución PRINCIPIO-0NN; decisiones DEC-NN (`03` §9); preguntas abiertas PQ-NN (`05`); principios de diseño DP-N y restricciones RC-N (`07`).
> - Las pantallas Should (P-013 a P-015) dependen de las preguntas abiertas PQ-07 y PQ-08. Esta especificación no tiene pendientes propios.
> - Importes con formato RN-081 (`S/ 1,250.50`, espacio U+0020); porcentajes con RN-124 (`82.50 %`). La vista solo muestra lo que calcula el backend (RN-027, DP-1).
> - Si este documento difiere de `03`, `06`, `07` o un ADR, prevalecen esos documentos.

### Cumplimiento de la constitución sin ADR nuevo

Esta especificación **no requiere ningún ADR**. Para lograrlo se fijan estas restricciones de diseño:

| Decisión | Motivo |
|---|---|
| Solo Bootstrap 5 servido desde `wwwroot/lib` (sin CDN) | PRINCIPIO-002, ADR-001, RC-2 (sin Internet en `Demo`) |
| Personalización con variables CSS (`--bs-*` y `--ef-*`) en `wwwroot/css/site.css`; **sin compilar Sass** | Un compilador Sass sería una herramienta nueva (PRINCIPIO-002) |
| Tipografía del sistema (sin fuentes web) | Sin dependencias ni Internet (RC-2) |
| **Sin librería de iconos** (Bootstrap Icons es un paquete aparte). Los pocos iconos son SVG en línea, decorativos y con `aria-hidden="true"` | PRINCIPIO-002 |
| Sin librería de gráficos: barras de progreso de Bootstrap y listas | PRINCIPIO-002, C4 (ADR-008 reservado) |
| JavaScript solo como mejora progresiva, en un único archivo propio `wwwroot/js/site.js` | ADR-006, DP-4 |
| Validación solo en el servidor: todos los formularios llevan `novalidate` | ADR-006, DEC-61 |

> Si en el futuro se quiere una librería de iconos, una fuente web, Sass o gráficos (C4), cada uno requiere un ADR antes de agregarlo.

---

## 1. Inventario de pantallas

### 1.1 Pantallas

| ID | Pantalla | Ruta MVC | Actor principal | HU que implementa | Prioridad |
|---|---|---|---|---|---|
| P-001 | Crear cuenta | `GET/POST /account/register` | Visitante | HU-001 | Must |
| P-002 | Iniciar sesión | `GET/POST /account/login` | Usuario registrado | HU-002A, HU-002B | Must |
| P-003 | Dashboard | `GET /dashboard` | Usuario autenticado | HU-008, HU-009, HU-010A, HU-010B, HU-001 (destino), HU-002B (cerrar sesión), HU-011 | Must |
| P-004 | Nuevo gasto | `GET/POST /transactions/create?type=Expense` | Usuario autenticado | HU-003, HU-002B (sesión expirada), HU-011 | Must |
| P-005 | Nuevo ingreso | `GET/POST /transactions/create?type=Income` | Usuario autenticado | HU-004, HU-011 | Must |
| P-006 | Lista de movimientos | `GET /transactions?page=n` | Usuario autenticado | HU-005, HU-006, HU-011 | Must |
| P-007 | Editar movimiento | `GET/POST /transactions/{id}/edit` | Usuario autenticado | HU-006, HU-011 | Must |
| P-008 | Confirmar eliminación (diálogo) | `POST /transactions/{id}/delete` | Usuario autenticado | HU-006, HU-011 | Must |
| P-009 | Crear presupuesto | `GET/POST /budgets/create` | Usuario autenticado | HU-007 | Must |
| P-010 | Editar presupuesto | `GET/POST /budgets/edit` | Usuario autenticado | HU-007 | Must |
| P-011 | Error inesperado (500) | `/Error` | Cualquiera | HU-013 | Must |
| P-012 | Página no encontrada (404) | cualquier ruta inexistente o ajena | Cualquiera | HU-011, HU-002A | Must |
| P-013 | Frecuencia de entrega | Se define al especificar S2 (PQ-07) | Usuario autenticado | HU-012 | Should |
| P-014 | Presupuesto por categoría | Se define al especificar S1 (PQ-08) | Usuario autenticado | HU-015 | Should (no lista) |
| P-015 | Metas de ahorro | Se define al especificar S4 (PQ-08) | Usuario autenticado | HU-017 | Should (no lista) |
| P-016 | Formulario caducado (400) | respuesta a un `POST` MVC con token antiforgery no válido | Cualquiera | HU-011 | Must |

### 1.2 Componentes compartidos

| ID | Componente | Usado en | HU |
|---|---|---|---|
| CMP-01 | Layout autenticado (cabecera, navegación, «Cerrar sesión», pie) | P-003 a P-010, P-013 a P-015 | HU-002B |
| CMP-02 | Layout público (cabecera simple, aviso de datos ficticios) | P-001, P-002, P-011, P-012, P-016 | HU-001, HU-002A |
| CMP-03 | Campo con etiqueta, ayuda y error (`aria-describedby`, `aria-invalid`) | Todos los formularios | HU-001, HU-003 |
| CMP-04 | Resumen de errores del formulario | Todos los formularios | HU-001, HU-003 |
| CMP-05 | Campo Monto (`S/`, coma o punto) | P-004, P-005, P-007, P-009, P-010 | HU-003, HU-004, HU-007 |
| CMP-06 | Grupo de opciones (radios en mosaico) | P-004, P-005, P-007 | HU-003, HU-004 |
| CMP-07 | Fila de movimiento | P-003, P-006 | HU-005, HU-008 |
| CMP-08 | Barra de consumo del presupuesto | P-003 | HU-009 |
| CMP-09 | Aviso de umbral | P-003 | HU-009 |
| CMP-10 | Botón de envío con estado «enviando» | P-001, P-002, P-003 (IA), P-004, P-005, P-007, P-009, P-010 | HU-001, HU-002A, HU-003, HU-004, HU-007, HU-010A |
| CMP-11 | Diálogo de confirmación de eliminación | P-008 | HU-006 |
| CMP-12 | Paginación | P-006 | HU-005 |

### 1.3 HU sin pantalla

| HU | Motivo |
|---|---|
| HU-014 | El seed lo ejecuta un humano o el CI desde la línea de comandos (RN-138, PRINCIPIO-010). No tiene UI. |
| HU-016 | Es una variante de P-006 (filtros sobre la lista); se especifica en P-006 §«Should». No está lista (PQ-08). |
| HU-018 | Es un botón dentro del aviso del 80 % de P-003; se especifica allí como Should. |

---

## 2. User flows

Los flujos siguen la lista única de `02` §3.3. Notación: `[Pantalla] --acción--> [Pantalla]`. Después de cada `POST` válido, el servidor responde `303` (Post/Redirect/Get).

### F-01 — Registro de cuenta, inicio y cierre de sesión (HU-001, HU-002A, HU-002B)

```
Registro
[P-002 Iniciar sesión] --enlace «¿No tienes cuenta? Crea una»--> [P-001 Crear cuenta]
[P-001] --completa 4 campos y pulsa «Crear cuenta»--> válido:   [P-003 Dashboard] (sesión iniciada, «Presupuesto: No configurado», MSG-41)
                                                     inválido: [P-001] con errores bajo cada campo, CMP-04 arriba y la casilla conservada
[Usuario con sesión] --abre /account/register o /account/login (p. ej., con «Atrás»)--> [P-003] (RN-173)

Inicio de sesión
[Cualquier ruta protegida sin sesión] --302--> [P-002] (con returnUrl)
[P-002] --correo + contraseña, «Iniciar sesión»--> válido:   returnUrl local o [P-003]
                                                  inválido: [P-002] con MSG-02 (mismo texto si la cuenta está bloqueada); returnUrl conservado
                                                  vacío:    [P-002] con MSG-32 y MSG-34 a la vez, CMP-04 arriba; returnUrl conservado
returnUrl hacia una ruta que solo admite POST (RN-174):
   /insights/monthly-summary --> [P-003]      /transactions/{id}/delete --> [P-006]

Cierre de sesión
[CMP-01 en cualquier pantalla] --«Cerrar sesión» (POST /account/logout)--> [P-002] sin mensaje
[CMP-01 con la sesión ya expirada] --«Cerrar sesión»--> [P-002] sin mensaje ni error (RN-177)

Sesión expirada (HU-002B)
[P-004 con datos escritos] --«Guardar gasto» tras la expiración--> [P-002] con MSG-48 (RN-171)
   --inicia sesión--> [P-004] vacío (limitación aceptada DEC-59)

Formulario caducado
[P-001 o P-002 abierta antes de iniciar sesión en otra pestaña] --envío--> [P-016] con MSG-46 (RN-170)
```

### F-02 — Registrar un gasto (HU-003)

```
[P-003 Dashboard] --botón «Nuevo gasto»--> [P-004]
[P-004] (fecha = hoy de Lima; medio preseleccionado = último gasto o «Efectivo")
   --Monto → Categoría → Medio de pago (ya marcado) → Fecha (ya puesta) → «Guardar gasto»-->
       botón deshabilitado y con «Guardando…» (JS)
       válido:   [P-003] con MSG-49 («Gasto guardado.»); el gasto aparece entre los 5 últimos si es de los más recientes por fecha
       inválido: [P-004] con errores; botón habilitado otra vez; los valores escritos se conservan
```

Camino óptimo para U2 (≤ 15 s): escribir el monto, tocar una categoría, tocar «Guardar gasto». Medio de pago y fecha ya vienen resueltos.

### F-03 — Corregir y eliminar un movimiento desde la lista (HU-005, HU-006)

```
[P-003] --enlace «Ver todos los movimientos»--> [P-006 página 1]
[P-006] --«Siguiente» / «Anterior»--> [P-006 página n]

Corregir
[P-006] --«Editar» en una fila--> [P-007]
[P-007] --cambia campos, «Guardar cambios»--> válido:   [P-006] en la página de origen con MSG-51 (DEC-65)
                                             inválido: [P-007] con errores (incl. MSG-17 si la fecha quedó fuera de la ventana)
[P-007] --«Cancelar»--> [P-006] en la página de origen

Eliminar
[P-006] --«Eliminar» en una fila--> [P-008 diálogo con MSG-18]
[P-008] --«Cancelar»--> [P-006], foco de vuelta en el botón «Eliminar» de esa fila
[P-008] --«Eliminar»--> POST /transactions/{id}/delete --303--> [P-006] en la página de origen con MSG-52 (DEC-65);
           si la página quedó vacía y no es la primera, la página anterior (RN-178)
Sin JS: «Eliminar» abre un <details> en la propia fila con MSG-18 y «Eliminar definitivamente» (DEC-67)
```

### F-04 — Registrar un ingreso (HU-004)

```
[P-003] --botón «Nuevo ingreso»--> [P-005]
[P-005] --Monto → Origen → (si «Otros»: Descripción opcional) → Fecha → «Guardar ingreso»-->
       válido:   [P-003] con MSG-50 («Ingreso guardado.») y «Saldo del mes» actualizado
       inválido: [P-005] con errores
```

### F-05 — Crear y editar el presupuesto del mes (HU-007)

```
Crear
[P-003 sin presupuesto] --enlace «Crear presupuesto» (junto a «Configura un presupuesto…»)--> [P-009]
[CMP-01] --«Presupuesto» (sin presupuesto del mes)--> [P-009]
[P-009] --monto, «Guardar presupuesto»--> válido:   [P-003] con MSG-53 y «Presupuesto: S/ 600.00»
                                         inválido: [P-009] con MSG-25 / MSG-19 / MSG-20 / MSG-09
                                         ya existe: [P-009] con MSG-21 y enlace a «Editar presupuesto»
                                         cambió el mes: [P-009] con MSG-22, el mes nuevo y el monto escrito (DEC-84)
[GET /budgets/create con presupuesto del mes] --302--> [P-010] (RN-175)

Editar
[P-003 con presupuesto] --enlace «Editar presupuesto»--> [P-010]
[CMP-01] --«Presupuesto» (con presupuesto del mes)--> [P-010]
[P-010] --nuevo monto, «Guardar cambios»--> [P-003] con MSG-53
                                          cambió el mes: [P-010] con MSG-22 y enlace a «Crear presupuesto» (DEC-84)
[GET /budgets/edit sin presupuesto del mes] --302--> [P-009] (RN-176)
```

### F-06 — Consultar el dashboard (HU-008, HU-009)

```
[P-002] --inicio de sesión--> [P-003]
[P-003] muestra, según el escenario del seed:
   demo1 → Normal + aviso 80 %      demo2 → Excedido + aviso 100 %
   demo3 → Sin presupuesto, saldo negativo      demo4 → Sin movimientos
   demo5 → 100.00 % sin exceso      demo6 → 25 movimientos
[P-003] --«Ver todos los movimientos»--> [P-006]
[P-003] --«Nuevo gasto» / «Nuevo ingreso»--> [P-004] / [P-005]
[P-003] --«Crear presupuesto» / «Editar presupuesto»--> [P-009] / [P-010]
```

### F-07 — Solicitar el resumen de IA (HU-010A, HU-010B)

```
[P-003 sección «Resumen del mes con IA»] --«Generar resumen del mes» (POST /insights/monthly-summary)-->
     botón deshabilitado + «Generando resumen…» (aria-live="polite")
     éxito:  303 → [P-003] con el resumen (una sola vez)
     fallo:  303 → [P-003] con MSG-23; resto del dashboard intacto
[P-003] --recargar--> [P-003] sin resumen y sin llamar a Ollama
IA deshabilitada por configuración → el botón aparece deshabilitado junto a MSG-24 desde la carga
```

### Flujos transversales (HU-011, HU-013)

```
[Cualquier pantalla] --recurso ajeno o inexistente (p. ej. /transactions/42/edit de otro usuario)--> [P-012] (404)
[Cualquier pantalla] --error no controlado (≠ Development)--> [P-011] (500, MSG-40)
[Cualquier formulario] --POST con token antiforgery no válido--> [P-016] (400, MSG-46)
```

---

## 3. Screen specs

### 3.0 Reglas comunes a todas las pantallas

Estas reglas aplican a todas las pantallas y no se repiten en cada ficha.

**Documento y semántica (PRINCIPIO-001, PRINCIPIO-012)**

- `<html lang="es-PE">`, `<meta name="viewport" content="width=device-width, initial-scale=1">` (sin `maximum-scale` ni `user-scalable=no`).
- `<title>` único: «`<Nombre de la pantalla>` — ExpenseFlow AI».
- Primer elemento enfocable: enlace «Saltar al contenido principal» (visible al recibir foco) que lleva a `<main id="contenido">`.
- Landmarks: `<header>`, `<nav aria-label="Principal">` (solo con sesión), `<main id="contenido">`, `<footer>`.
- Un único `<h1>` por pantalla. Sin saltos de nivel de encabezado.
- Ningún texto en inglés visible ni en atributos accesibles (`aria-label`, `title`, `alt`, `placeholder`).

**Formularios (PRINCIPIO-012, ADR-006)**

- `<form method="post" novalidate>` con el token antiforgery de Razor.
- Cada campo usa CMP-03: `<label for>` visible siempre (nunca solo `placeholder`).
- Error de un campo: texto bajo el campo, con icono SVG decorativo (`aria-hidden="true"`) **y** el texto del mensaje; el campo recibe `aria-invalid="true"` y `aria-describedby` apunta al `id` del error (y al de la ayuda, si hay). El borde rojo nunca es la única señal.
- Si hay errores de campo, el formulario muestra arriba CMP-04 y el **primer campo con error** lleva el atributo `autofocus` (lo pone Razor, funciona sin JS). CMP-04 no recibe el foco (DEC-78).
- **Errores que no son de un campo** (MSG-02, MSG-21, MSG-22): se muestran en un bloque propio sobre el formulario, con `role="alert"` y estilo `--ef-danger-bg`, sin CMP-04 y sin `aria-invalid` en los campos (DEC-80). Si además hay errores de campo, se muestran ambos.
- Todo botón de envío de formulario usa CMP-10 (DEC-74).
- Tras un error, el servidor vuelve a mostrar lo que el usuario escribió (salvo contraseñas).
- Los campos de radio se agrupan en `<fieldset>` con `<legend>`; el error del grupo se asocia al `<fieldset>` con `aria-describedby`.

**Mensajes de confirmación (RN-172)**

- Tras guardar o eliminar, la página de destino muestra **una sola vez** la confirmación (MSG-49 a MSG-53), al inicio de `<main>`, en un bloque con `role="status"`, estilo `--ef-success-bg` / `--ef-success-text` y un icono decorativo. Llega por TempData: al recargar, ya no aparece.
- El bloque no recibe el foco ni se cierra solo; desaparece en la siguiente navegación.

**Mejora progresiva (DP-4)**

- Toda pantalla funciona igual sin JavaScript, salvo: deshabilitar el botón al enviar (RN-131, RN-168), mostrar u ocultar «Descripción» en ingresos y el diálogo de eliminación. Cada uno tiene su comportamiento sin JS documentado en su ficha.
- `site.js` se carga con `defer` al final del `<body>`.

**Tamaños mínimos (WCAG 2.2, 2.5.8)**

- Toda área interactiva mide al menos 44 × 44 px (el mínimo AA es 24 × 24; se usa 44 por el contexto Android).
- El foco es siempre visible (token `--ef-focus-ring`) y no queda tapado por la cabecera (2.4.11): la cabecera **no es fija** (`position: static`).

**Montos**

- Columna o bloque de montos con `font-variant-numeric: tabular-nums` y alineado a la derecha en listas.
- El tipo de movimiento siempre se dice en texto («Gasto» / «Ingreso»), no solo con color.

---

### CMP-01 — Layout autenticado

**Propósito:** marco común de las pantallas con sesión; da acceso a las secciones y a «Cerrar sesión».
**HU:** HU-002B.

**Datos:** ninguno financiero. Puede mostrar el correo del usuario `[Propuesta]`; nunca su contraseña.

**Acciones:**

| Acción | Elemento | Destino |
|---|---|---|
| Ir al dashboard | Enlace «Inicio» y la marca «ExpenseFlow AI» | `/dashboard` |
| Ir a la lista | Enlace «Movimientos» | `/transactions` |
| Ir al presupuesto | Enlace «Presupuesto» | `/budgets/create` si no hay presupuesto del mes; `/budgets/edit` si lo hay (lo decide el servidor) |
| Cerrar sesión | `<form method="post" action="/account/logout">` con botón «Cerrar sesión» y token antiforgery | P-002 (DEC-63). Con la sesión ya expirada, también P-002, sin error (RN-177) |

**Reglas:** RN-097, RN-140 (`Cache-Control: no-store`, lo aplica el servidor), RN-177.

**Wireframe (360 px):**

```
┌────────────────────────────────────────┐
│ [Saltar al contenido principal]  ← solo visible con foco
├────────────────────────────────────────┤ <header> fondo --ef-color-primary
│ ExpenseFlow AI            [Cerrar sesión]│ marca: enlace a /dashboard (texto blanco, 1.125rem, 700)
│                                         │ botón: btn-outline-light, 44 px de alto
├────────────────────────────────────────┤ <nav aria-label="Principal"> fondo --ef-color-surface
│  Inicio  │ Movimientos │ Presupuesto   │ 3 enlaces en fila, cada uno 1/3 del ancho, 48 px de alto
│  ▔▔▔▔▔▔                                │ el activo: aria-current="page", texto 700 y borde inferior 3 px primario
├────────────────────────────────────────┤
│ <main id="contenido">                  │ container, padding lateral 16 px
│   …contenido de la pantalla…           │
├────────────────────────────────────────┤ <footer> texto --ef-color-text-muted, 0.875rem
│ Aplicación académica con datos         │
│ ficticios. Los saldos son calculados   │
│ según tus registros.                   │ [Propuesta] (PRINCIPIO-006, PRINCIPIO-009)
└────────────────────────────────────────┘
```

**Escritorio (≥ 768 px):** cabecera y navegación en una sola franja: marca a la izquierda, los 3 enlaces en el centro, «Cerrar sesión» a la derecha. Contenido en `container` (máx. 1140 px).

**Sin JavaScript:** igual. **No** se usa el componente `collapse` («hamburguesa») de Bootstrap, porque depende de JS; con 3 enlaces cabe en 360 px.

---

### CMP-02 — Layout público

**Propósito:** marco de registro, inicio de sesión y páginas de error.
**HU:** HU-001, HU-002A.

```
┌────────────────────────────────────────┐
│ [Saltar al contenido principal]        │
├────────────────────────────────────────┤ <header> fondo --ef-color-primary
│ ExpenseFlow AI                          │ texto, no enlace si no hay sesión
├────────────────────────────────────────┤
│ <main>  tarjeta centrada, máx. 440 px   │
├────────────────────────────────────────┤
│ <footer> igual que CMP-01              │
└────────────────────────────────────────┘
```

Sin `<nav>`. El fondo de la página es `--ef-color-bg` y el formulario va en una tarjeta `--ef-color-surface`.

---

### CMP-03 — Campo con etiqueta, ayuda y error

**Usado en:** todos los formularios. **HU:** HU-001, HU-003, HU-004, HU-007. **Reglas:** RN-116.

```html
<div class="ef-field">
  <label for="Amount" class="form-label">Monto</label>
  <input id="Amount" name="Amount" class="form-control"
         aria-describedby="Amount-help Amount-error" aria-invalid="true">
  <p id="Amount-help" class="ef-help">…</p>          <!-- solo si hay ayuda -->
  <p id="Amount-error" class="ef-error">              <!-- solo si hay error -->
    <svg aria-hidden="true" …></svg> El monto debe tener como máximo dos decimales.
  </p>
</div>
```

| Parte | Regla |
|---|---|
| Etiqueta | `<label for>` visible siempre, 1rem / 600. Nunca se reemplaza por `placeholder` |
| Ayuda | Opcional, bajo el campo, 0.875rem `--ef-color-text-muted`, con `id="<Campo>-help"` |
| Error | Bajo la ayuda, `--ef-color-danger`, icono decorativo + texto del MSG, con `id="<Campo>-error"`. Solo se muestra el primer error del campo según el orden de su regla |
| `aria-describedby` | Lista la ayuda y el error presentes, en ese orden |
| `aria-invalid` | `true` solo si el campo tiene error |
| Borde | 1 px `--ef-color-border`; con error, 2 px `--ef-color-danger` (nunca es la única señal) |
| Grupo de radios | El mismo patrón sobre el `<fieldset>` (CMP-06) |

### CMP-12 — Paginación

**Usado en:** P-006. **HU:** HU-005. **Reglas:** RN-117, RN-178.

```html
<nav aria-label="Paginación">
  <a class="btn btn-outline-primary" href="/transactions?page=1" rel="prev">Anterior</a>
  <p class="ef-page" aria-current="page">Página 2 de 2</p>
  <span class="btn btn-outline-secondary disabled" aria-disabled="true">Siguiente</span>
</nav>
```

- Solo se renderiza si hay más de 1 página.
- «Anterior» y «Siguiente» son enlaces `GET` de 48 px de alto; en la primera o la última página, el que no aplica se renderiza como `<span aria-disabled="true">` (no es enfocable).
- El texto «Página *n* de *m*» es el único indicador de posición; no hay números de página intermedios.
- Funciona igual sin JavaScript.

---

### P-001 — Crear cuenta

| Campo | Valor |
|---|---|
| Propósito | Crear una cuenta con correo y contraseña declarando la mayoría de edad, y entrar directo al dashboard |
| Actor | Visitante (sin sesión) |
| HU | HU-001 |
| Reglas | RN-001, RN-002, RN-003, RN-004, RN-094, RN-095, RN-096, RN-099, RN-116, RN-148, RN-149, RN-150, RN-151, RN-152, RN-153, RN-168, RN-170, RN-173 |
| Layout | CMP-02 |

**Datos que muestra:** MSG-05 sobre el botón «Crear cuenta». MSG-05 **es** el aviso de RN-099 de no ingresar datos reales (DEC-68); no hay otro aviso en la pantalla.

**Campos (exactamente 4 entradas, sin «Recordarme»):**

| # | Etiqueta | Control | Atributos | Errores posibles (orden) |
|---|---|---|---|---|
| 1 | «Correo» | `<input type="text">` | `inputmode="email"`, `autocomplete="email"`, `autocapitalize="none"`, `spellcheck="false"` | MSG-32 → MSG-37 → MSG-33 → MSG-04 (RN-149: solo se muestra el primero que aplique) |
| 2 | «Contraseña» | `<input type="password">` | `autocomplete="new-password"`; ayuda MSG-47 | MSG-34 → MSG-03 |
| 3 | «Confirmar contraseña» | `<input type="password">` | `autocomplete="new-password"` | MSG-36 → MSG-35 (MSG-35 solo si la contraseña no está vacía) |
| 4 | «Declaro ser mayor de 18 años» | `<input type="checkbox">` | etiqueta clicable | MSG-01 |

- El correo usa `type="text"` con `inputmode="email"`: muestra el teclado de correo en el móvil y el navegador no transforma el valor (por ejemplo, a punycode) antes de enviarlo; RN-150 lo valida el servidor (DEC-77).
- Ayuda de la contraseña bajo el campo, ligada con `aria-describedby`: MSG-47 («Mínimo 8 caracteres, con mayúscula, minúscula, número y símbolo.», DEC-87).
- Los errores de campos vacíos aparecen **a la vez** (MSG-32, MSG-34, MSG-36), cada uno bajo su campo. Los errores de «Contraseña» y «Confirmar contraseña» pueden aparecer a la vez (DEC-76).
- Un usuario con sesión que abre esta pantalla va a `/dashboard` (RN-173).

**Acciones:**

| Acción | Resultado |
|---|---|
| «Crear cuenta» (botón primario, ancho completo) | Válido ⇒ cuenta creada, sesión iniciada, `303` a `/dashboard`. Inválido ⇒ misma pantalla con errores |
| Enlace «¿Ya tienes cuenta? Inicia sesión» | P-002 |

**Wireframe (360 px):**

```
┌────────────────────────────────────────┐
│ ExpenseFlow AI                          │ CMP-02
├────────────────────────────────────────┤
│ ┌────────────────────────────────────┐ │ tarjeta, padding 24 px
│ │ <h1> Crear cuenta                  │ │ 1.5rem / 700
│ │                                    │ │
│ │ [CMP-04 resumen de errores]        │ │ solo si hay errores
│ │                                    │ │
│ │ Correo                             │ │ <label for="Email">
│ │ ┌────────────────────────────────┐ │ │ 48 px de alto
│ │ └────────────────────────────────┘ │ │
│ │ ⚠ MSG-33                           │ │ id="Email-error", color --ef-color-danger
│ │                                    │ │
│ │ Contraseña                         │ │
│ │ ┌────────────────────────────────┐ │ │
│ │ └────────────────────────────────┘ │ │
│ │ Mínimo 8 caracteres, con…          │ │ MSG-47, id="Password-help", texto muted 0.875rem
│ │                                    │ │
│ │ Confirmar contraseña               │ │
│ │ ┌────────────────────────────────┐ │ │
│ │ └────────────────────────────────┘ │ │
│ │                                    │ │
│ │ ┌──┐                               │ │ casilla 24×24 dentro de un área de 44 px
│ │ │  │ Declaro ser mayor de 18 años  │ │ toda la fila es la <label>
│ │ └──┘                               │ │
│ │ ⚠ MSG-01                           │ │ id="AdultDeclaration-error"
│ │                                    │ │
│ │ MSG-05                             │ │ aviso RN-099, 0.875rem, sobre el botón
│ │ ┌────────────────────────────────┐ │ │
│ │ │          Crear cuenta          │ │ │ btn-primary, w-100, 48 px
│ │ └────────────────────────────────┘ │ │
│ │                                    │ │
│ │ ¿Ya tienes cuenta? Inicia sesión   │ │ enlace, centrado
│ └────────────────────────────────────┘ │
└────────────────────────────────────────┘
```

**Estados:**

| Estado | Qué cambia |
|---|---|
| Inicial | Campos vacíos, casilla desmarcada, sin errores |
| Errores | CMP-04 arriba; error bajo cada campo; `autofocus` en el primero con error; el correo conserva lo escrito; la casilla conserva el valor enviado (DEC-75); contraseñas vacías |
| Enviando | CMP-10: botón deshabilitado con «Creando cuenta…» (DEC-74) |

---

### P-002 — Iniciar sesión

| Campo | Valor |
|---|---|
| Propósito | Acceder con correo y contraseña |
| Actor | Usuario registrado |
| HU | HU-002A (y destino de HU-002B al cerrar sesión o expirar) |
| Reglas | RN-006, RN-048, RN-049, RN-050, RN-097, RN-098, RN-116, RN-144, RN-145, RN-146, RN-147, RN-154, RN-155, RN-168, RN-170, RN-171, RN-173, RN-174, RN-177 |
| Layout | CMP-02 |

**Campos (solo 2, sin «Recordarme», sin «¿Olvidaste tu contraseña?»):**

| Etiqueta | Control | Atributos | Errores |
|---|---|---|---|
| «Correo» | `<input type="text">` | `inputmode="email"`, `autocomplete="username"`, `autocapitalize="none"`, `spellcheck="false"` | MSG-32 |
| «Contraseña» | `<input type="password">` | `autocomplete="current-password"` | MSG-34 |

- Credenciales incorrectas, correo inexistente o cuenta bloqueada ⇒ **un único** mensaje MSG-02 en un bloque de error **sobre** el formulario (no bajo un campo, para no indicar cuál falló). El bloque tiene `role="alert"`.
- `returnUrl` va en un campo oculto y **se conserva en cada nueva presentación** del formulario tras MSG-02, MSG-32 o MSG-34 (DEC-72). El servidor lo valida (RN-155) y, si apunta a una ruta que solo admite `POST`, redirige a la página del formulario (RN-174).
- El correo se compara sin espacios exteriores (DEC-82).
- **Aviso de sesión expirada:** si la llegada es por expiración (`reason=expired`), sobre el formulario se muestra MSG-48 en un bloque informativo con `role="status"` y estilo `--ef-info-bg` / `--ef-info-text` (RN-171). No aparece tras «Cerrar sesión» ni en el primer acceso.
- Un usuario con sesión que abre esta pantalla va a `/dashboard` (RN-173).
- **Limitación aceptada (W17):** no hay recuperación de contraseña.

**Acciones:** «Iniciar sesión» (CMP-10, primario, ancho completo, «Iniciando sesión…» al enviar); enlace «¿No tienes cuenta? Crea una» → P-001.

**Wireframe (360 px):**

```
┌────────────────────────────────────────┐
│ ExpenseFlow AI                          │
├────────────────────────────────────────┤
│ ┌────────────────────────────────────┐ │
│ │ <h1> Iniciar sesión                │ │
│ │                                    │ │
│ │ ┌ role="status" (info) ──────────┐ │ │ solo si la sesión expiró
│ │ │ ℹ MSG-48                        │ │ │
│ │ └────────────────────────────────┘ │ │
│ │ ┌ alert-danger role="alert" ─────┐ │ │ solo tras un intento fallido
│ │ │ ⚠ MSG-02                        │ │ │
│ │ └────────────────────────────────┘ │ │
│ │                                    │ │
│ │ Correo                             │ │
│ │ ┌────────────────────────────────┐ │ │
│ │ └────────────────────────────────┘ │ │
│ │ Contraseña                         │ │
│ │ ┌────────────────────────────────┐ │ │
│ │ └────────────────────────────────┘ │ │
│ │                                    │ │
│ │ ┌────────────────────────────────┐ │ │
│ │ │         Iniciar sesión         │ │ │ btn-primary, w-100, 48 px
│ │ └────────────────────────────────┘ │ │
│ │ ¿No tienes cuenta? Crea una        │ │
│ └────────────────────────────────────┘ │
└────────────────────────────────────────┘
```

**Estados:** inicial; MSG-02 (bloque `role="alert"`, correo conservado, contraseña vacía, foco en «Contraseña», `returnUrl` conservado); campos vacíos (MSG-32 y MSG-34 a la vez, cada uno bajo su campo, con CMP-04 arriba); llegada tras cerrar sesión (sin mensaje); llegada por expiración (MSG-48); enviando (CMP-10).

---

### P-003 — Dashboard

| Campo | Valor |
|---|---|
| Propósito | Ver en una pantalla saldos, presupuesto, avisos, gasto por categoría, últimos movimientos y el resumen de IA |
| Actor | Usuario autenticado |
| HU | HU-008, HU-009, HU-010A, HU-010B, HU-011 (solo datos propios) |
| Reglas | RN-013, RN-023 a RN-033, RN-037, RN-066 a RN-081, RN-120 a RN-128, RN-131, RN-132, RN-141, RN-142, RN-162, RN-163, RN-165, RN-166, RN-172 |
| Layout | CMP-01, «Inicio» activo |

**Datos que muestra** (todos llegan calculados y formateados en el `ViewModel`; la vista no calcula nada, RN-027):

| Bloque | Dato | Ejemplo `demo1` |
|---|---|---|
| Saldos | «Saldo del mes» (`<h2>`) | «Saldo del mes: S/ 105.00» |
| | «Saldo acumulado» (`<h3>`) | «Saldo acumulado: S/ 155.00» |
| | Leyenda junto a cada saldo | «Saldo calculado según tus registros» |
| | Texto si el saldo es negativo | «Tus gastos superan tus ingresos registrados» |
| Presupuesto | Monto | «Presupuesto: S/ 600.00» / «Presupuesto: No configurado» |
| | Porcentaje | «Porcentaje consumido: 82.50 %» / «Porcentaje consumido: No disponible» |
| | Restante | «Restante del presupuesto: S/ 105.00» (mínimo S/ 0.00) |
| | Estado y exceso (solo si excedido) | «Estado: Presupuesto excedido», «Exceso: S/ 50.00» |
| | Barra | CMP-08 |
| Avisos | Umbral | CMP-09: «Alcanzaste el 80 % de tu presupuesto mensual» o «Alcanzaste el 100 % de tu presupuesto mensual» |
| Categorías | Total por categoría > 0, de mayor a menor | Alimentación S/ 270.00, Transporte S/ 120.00, Estudios S/ 105.00 |
| Movimientos | Hasta 5, con CMP-07 | 5 filas + «Ver todos los movimientos» |
| IA | Resumen (una sola vez), MSG-23 o MSG-24 | — |
| Confirmación | MSG-49, MSG-50 o MSG-53 tras guardar (una sola vez, RN-172) | «Gasto guardado.» |

**Acciones:**

| Acción | Elemento | Destino |
|---|---|---|
| Registrar gasto | Botón-enlace primario «Nuevo gasto» | P-004 |
| Registrar ingreso | Botón-enlace secundario «Nuevo ingreso» | P-005 |
| Crear presupuesto | Enlace «Crear presupuesto» (solo sin presupuesto) | P-009 |
| Editar presupuesto | Enlace «Editar presupuesto» (solo con presupuesto) | P-010 |
| Ver lista | Enlace «Ver todos los movimientos» | P-006 |
| Pedir resumen | Botón «Generar resumen del mes» (`POST /insights/monthly-summary`) | `303` → P-003 |

**Orden de lectura** (es el orden del DOM; no se reordena con CSS para no romper el orden de foco):

0. Confirmación (si llega)
1. `<h1>` + acciones rápidas
2. Saldos (`<h2>` saldo del mes antes de `<h3>` saldo acumulado, HU-008)
3. Presupuesto del mes (aviso, cifras, barra)
4. Gasto por categoría
5. Últimos movimientos
6. Resumen del mes con IA

**Wireframe (360 px), escenario `demo1`:**

```
┌────────────────────────────────────────┐
│ CMP-01 (Inicio activo)                  │
├────────────────────────────────────────┤
│ <h1> Mi mes: noviembre de 2026         │ [Propuesta]; mes en es-PE desde el ViewModel
│ ┌──────────────────┐┌─────────────────┐│ fila de 2 botones, 50 % cada uno, 48 px
│ │ + Nuevo gasto    ││ + Nuevo ingreso ││ primario / outline-primario
│ └──────────────────┘└─────────────────┘│ el «+» es texto, no icono
│                                        │
│ ┌ section aria-labelledby="saldos-h" ─┐│ tarjeta
│ │ <h2 id="saldos-h">                   ││ «Saldo del mes: » 1.125rem / 600
│ │   Saldo del mes: S/ 105.00           ││ importe en <span> 1.75rem / 700, tabular-nums
│ │ Saldo calculado según tus registros  ││ 0.875rem muted
│ │ ───────────────────────────────────  ││ separador --ef-color-divider
│ │ <h3> Saldo acumulado: S/ 155.00      ││ importe 1.25rem / 700
│ │ Saldo calculado según tus registros  ││
│ └──────────────────────────────────────┘│
│                                        │
│ ┌ section aria-labelledby="pres-h" ───┐│ tarjeta
│ │ <h2 id="pres-h"> Presupuesto del mes ││ 1.125rem / 600 [Propuesta]
│ │                                      ││
│ │ ┌ CMP-09 role="status" ───────────┐  ││ fondo --ef-warning-bg, borde izq. 4 px
│ │ │ ⚠ Alcanzaste el 80 % de tu      │  ││ texto --ef-warning-text, 600
│ │ │   presupuesto mensual            │  ││
│ │ └─────────────────────────────────┘  ││
│ │                                      ││
│ │ <dl> (dos columnas: término | valor) ││
│ │  Presupuesto:           S/ 600.00    ││ el <dt> incluye los dos puntos (DEC-83)
│ │  Porcentaje consumido:  82.50 %      ││
│ │  Restante del presupuesto: S/ 105.00 ││
│ │                                      ││
│ │ [CMP-08 ████████████████░░░░ 82.50 %]││ barra 16 px de alto, relleno ámbar (≥ 80 %)
│ │                                      ││
│ │ Editar presupuesto                   ││ enlace, alineado a la izquierda
│ └──────────────────────────────────────┘│
│                                        │
│ ┌ section aria-labelledby="cat-h" ────┐│
│ │ <h2 id="cat-h"> Gasto por categoría  ││
│ │ <ul>                                 ││ lista sin viñetas, una fila por categoría
│ │  Alimentación          S/ 270.00     ││ nombre a la izquierda, monto a la derecha
│ │  Transporte            S/ 120.00     ││ 48 px por fila, separador entre filas
│ │  Estudios              S/ 105.00     ││ (sin barras: no son datos del backend)
│ │ </ul>                                ││
│ └──────────────────────────────────────┘│
│                                        │
│ ┌ section aria-labelledby="mov-h" ────┐│
│ │ <h2 id="mov-h"> Últimos movimientos  ││
│ │ <ol> 5 × CMP-07                      ││ ver CMP-07 (sin botones Editar/Eliminar aquí)
│ │  01/11/2026 · Estudios · Gasto       ││
│ │                         S/ 105.00    ││
│ │  …                                   ││
│ │ </ol>                                ││
│ │ Ver todos los movimientos            ││ enlace, 48 px de alto
│ └──────────────────────────────────────┘│
│                                        │
│ ┌ section aria-labelledby="ia-h" ─────┐│
│ │ <h2 id="ia-h"> Resumen del mes con IA││ [Propuesta]
│ │ ┌──────────────────────────────────┐ ││
│ │ │      Generar resumen del mes     │ ││ btn-outline-primary, w-100, 48 px
│ │ └──────────────────────────────────┘ ││
│ │ <div id="ia-estado" aria-live="polite">  vacío; el JS escribe «Generando resumen…»
│ └──────────────────────────────────────┘│
└────────────────────────────────────────┘
```

**Escritorio (≥ 992 px):** cuadrícula de 12 columnas. Fila 1: `<h1>` (8 col.) y los dos botones (4 col., a la derecha). Fila 2: saldos (5 col.) y presupuesto (7 col.). Fila 3: gasto por categoría (5 col.) y últimos movimientos (7 col.). Fila 4: resumen de IA (12 col., máx. 72 caracteres por línea en el texto). El orden del DOM es el mismo que en móvil.

**Variantes por escenario (estados):**

| Escenario | Saldos | Bloque Presupuesto | Aviso | Categorías | Movimientos |
|---|---|---|---|---|---|
| Normal < 80 % | Positivos | Monto, %, restante, barra azul | Ninguno | Lista | Lista |
| `demo1` 80 %–<100 % | Positivos | Igual, barra ámbar | «Alcanzaste el 80 %…» | Lista | Lista |
| `demo5` 100.00 % sin exceso | — | «100.00 %», «Restante del presupuesto: S/ 0.03», barra roja llena; **sin** «Presupuesto excedido» | «Alcanzaste el 100 %…» | Lista | Lista |
| `demo2` excedido | — | «110.00 %», «Restante del presupuesto: S/ 0.00», «Estado: Presupuesto excedido», «Exceso: S/ 50.00», barra roja llena | «Alcanzaste el 100 %…» | Lista | Lista |
| `demo3` sin presupuesto, saldo negativo | «Saldo del mes: -S/ 150.00» y «Saldo acumulado: -S/ 150.00», cada uno con «Tus gastos superan tus ingresos registrados» | «Presupuesto: No configurado», «Porcentaje consumido: No disponible», «Configura un presupuesto para controlar tus gastos» + enlace «Crear presupuesto». **Sin** barra, restante, exceso ni estado | Ninguno | Lista | Lista |
| `demo4` sin movimientos | S/ 0.00, sin texto de saldo negativo | Como `demo3` | Ninguno | MSG-42 | MSG-41 (sin enlace «Ver todos…» `[Propuesta]`) |
| Solo ingresos | Positivos | Según presupuesto | — | MSG-42 | Lista |

**Saldo negativo:** el importe se escribe con signo «-S/ …» (RN-081), en color `--ef-color-danger` **y** con el texto «Tus gastos superan tus ingresos registrados» debajo, en el mismo bloque. El color nunca va solo (HU-008).

**Resumen de IA (estados de la sección 6):**

| Estado | Qué se ve |
|---|---|
| IA habilitada, sin resumen | Botón «Generar resumen del mes» habilitado |
| Generando (con JS) | Botón con `disabled`, texto «Generando resumen…» en el botón **y** en `#ia-estado` (`aria-live="polite"`); spinner decorativo `aria-hidden="true"`. Se envía 1 sola solicitud |
| Generando (sin JS) | El navegador muestra su indicador de carga; un segundo toque puede reenviar (limitación aceptada DEC-90) |
| Resultado | Sobre el botón: «Explicación generada automáticamente» (etiqueta 0.875rem, 600) → texto del modelo en `<p class="ef-ai-text">` (`white-space: pre-line`, codificado por Razor, nunca `Html.Raw`) → «No constituye asesoría financiera» (0.875rem, muted). Bloque con borde izquierdo 4 px `--ef-color-accent`. Se muestra **una sola vez** (TempData) |
| Fallo (MSG-23) | Bloque `alert-warning` con `role="status"` y MSG-23; botón habilitado; el resto del dashboard intacto, incluido el aviso del 80 % |
| IA deshabilitada (MSG-24) | Botón con `disabled` y `aria-describedby` hacia un texto con MSG-24 bajo el botón, desde la carga |

**Wireframe del resultado de IA:**

```
│ ┌ section ─────────────────────────────┐│
│ │ <h2> Resumen del mes con IA          ││
│ │ ┃ Explicación generada               ││ borde izquierdo acento
│ │ ┃ automáticamente                    ││
│ │ ┃                                    ││
│ │ ┃ Este mes gastaste S/ 495.00 de     ││ texto del modelo, 1rem, interlineado 1.6
│ │ ┃ tu presupuesto de S/ 600.00…       ││
│ │ ┃                                    ││
│ │ ┃ No constituye asesoría financiera  ││ 0.875rem muted
│ │ ┌──────────────────────────────────┐ ││
│ │ │      Generar resumen del mes     │ ││
│ │ └──────────────────────────────────┘ ││
│ └──────────────────────────────────────┘│
```

**Literales en el DOM (DEC-83):** cada `<dt>` termina en dos puntos («Presupuesto:», «Porcentaje consumido:», «Restante del presupuesto:», «Estado:», «Exceso:») para que el texto visible coincida con los criterios de HU-007 a HU-009.

**Sin JavaScript:** todas las cifras son idénticas (HU-008). Lo único que cambia es que el botón de IA no se deshabilita al pulsarlo.

**Should (HU-018, no se implementa en el MVP):** dentro de CMP-09 del 80 %, un botón «Explicar mi consumo». No aparece con el aviso del 100 % ni sin presupuesto.

---

### CMP-07 — Fila de movimiento

**Usado en:** P-003 (sin acciones) y P-006 (con «Editar» y «Eliminar»).
**HU:** HU-003, HU-004, HU-005, HU-008. **Reglas:** RN-079, RN-080, RN-081.

**Formato del texto (literal de las HU):**

- Gasto: «08/11/2026 · Transporte · Gasto · S/ 3.50»
- Ingreso: «08/11/2026 · Apoyo familiar · Ingreso · S/ 600.00»
- Ingreso «Otros» con descripción: «08/11/2026 · Otros: Venta de libros · Ingreso · S/ 80.00»

**Wireframe (360 px):**

```
<li class="ef-tx">
┌────────────────────────────────────────┐
│ 08/11/2026 · Transporte · Gasto        │ 1ª línea: fecha (tabular), categoría u origen, tipo
│                            S/ 3.50     │ 2ª línea (o misma en ≥ 576 px): monto, 600, derecha
│ [ Editar ]  [ Eliminar ]               │ solo en P-006: botones pequeños, 44 px de alto
└────────────────────────────────────────┘
```

- El DOM contiene el literal completo de la fila: antes del monto hay un separador « · » visualmente oculto (`visually-hidden`), así que el texto es «08/11/2026 · Transporte · Gasto · S/ 3.50» aunque el monto se vea en otra línea (DEC-83).
- La palabra «Gasto» / «Ingreso» va en una insignia de texto (`badge`) con fondo neutro; el color del monto es el texto normal (no se colorea por tipo, para no depender del color).
- La descripción «Otros» larga (hasta 100 caracteres) se ajusta en varias líneas (`overflow-wrap: anywhere`); nunca se trunca.
- En P-006, los botones llevan un nombre accesible completo: `aria-label="Editar gasto del 08/11/2026 de S/ 3.50"` y `aria-label="Eliminar gasto del 08/11/2026 de S/ 3.50"`. El texto visible sigue siendo «Editar» / «Eliminar».
- Separador inferior 1 px `--ef-color-divider`; altura mínima 56 px.

---

### CMP-08 — Barra de consumo del presupuesto

**HU:** HU-009. **Reglas:** RN-024, RN-028, RN-029, RN-120, RN-141.

```html
<div class="progress ef-progress"
     role="progressbar"
     aria-label="Porcentaje consumido del presupuesto"
     aria-valuemin="0" aria-valuemax="100"
     aria-valuenow="82.50" aria-valuetext="82.50 %">
  <div class="progress-bar ef-progress--warning" style="width: 82.50%"></div>
</div>
```

| Porcentaje (ya redondeado por el backend) | Ancho visual | `aria-valuenow` | `aria-valuetext` | Color de relleno |
|---|---|---|---|---|
| < 80.00 | el porcentaje | el porcentaje | «NN.NN %» | `--ef-progress-normal` |
| 80.00 – 99.99 | el porcentaje | el porcentaje | «NN.NN %» | `--ef-progress-warning` |
| ≥ 100.00 (`demo2`: 110.00) | 100 % (no se desborda) | `100` | el porcentaje real («110.00 %») | `--ef-progress-danger` |

- El ancho y los valores ARIA los entrega el `ViewModel` ya limitados; la vista no compara umbrales.
- La barra mide 16 px de alto, con borde 1 px `--ef-color-border` para que el contenedor sea perceptible (contraste ≥ 3:1, WCAG 1.4.11).
- El porcentaje también se muestra en texto en la lista `<dl>` (el color de la barra no es la única señal).

---

### CMP-09 — Aviso de umbral

**HU:** HU-009. **Reglas:** RN-028, RN-029, RN-120, RN-121.

| `alert` del backend | Texto | Estilo |
|---|---|---|
| `Threshold80` | «Alcanzaste el 80 % de tu presupuesto mensual» | `--ef-warning-bg` / `--ef-warning-text`, borde izq. 4 px |
| `Threshold100` | «Alcanzaste el 100 % de tu presupuesto mensual» | `--ef-danger-bg` / `--ef-danger-text`, borde izq. 4 px |
| `null` | — (no se renderiza el contenedor) | — |

- Contenedor con `role="status"`, texto 600, icono SVG triangular `aria-hidden="true"`.
- Es independiente de la IA: aparece aunque Ollama esté caído (HU-009, HU-010B).

---

### P-004 — Nuevo gasto

| Campo | Valor |
|---|---|
| Propósito | Registrar un gasto en ≤ 15 s (U2) |
| Actor | Usuario autenticado |
| HU | HU-003, HU-002B (sesión expirada), HU-011 |
| Reglas | RN-010 a RN-012, RN-014 a RN-019, RN-051 a RN-057, RN-104, RN-105, RN-108, RN-109, RN-116, RN-137, RN-168, RN-171, RN-172 |
| Layout | CMP-01 (ningún enlace activo) |

**Campos (exactamente 4, en este orden):**

| # | Etiqueta | Control | Valor inicial | Errores |
|---|---|---|---|---|
| 1 | «Monto» | CMP-05: `<input type="text" inputmode="decimal" autocomplete="off">` | vacío | MSG-11, MSG-06, MSG-07, MSG-08, MSG-09 |
| 2 | «Categoría» | CMP-06: `<fieldset>` con 6 radios, en el orden de RN-014: Alimentación, Transporte, Estudios, Entretenimiento, Salud, Otros | ninguna | MSG-12 |
| 3 | «Medio de pago» | CMP-06: `<fieldset>` con 5 radios: Efectivo, Yape, Plin, Tarjeta, Otro | el del último gasto creado; «Efectivo» si no hay gastos (RN-018, RN-108) | MSG-13 |
| 4 | «Fecha» | `<input type="date">`, **sin** `min` ni `max` (ADR-006, punto 2) | hoy de Lima (RN-137) | MSG-16, MSG-10, MSG-17 |

- Se usan radios en mosaico en lugar de `<select>`: un toque por campo en Android (U2) y todas las opciones visibles.
- El campo de fecha no lleva `min`, `max` ni `required` (ADR-006): los límites los valida solo el servidor (MSG-10, MSG-17). Esto aplica a todos los campos de fecha del documento.

**Acciones:**

| Acción | Resultado |
|---|---|
| «Guardar gasto» (primario, ancho completo) | Con JS: CMP-10 deshabilita el botón y muestra «Guardando…» `[Propuesta]`. Válido ⇒ `303` a `/dashboard` con MSG-49. Inválido ⇒ misma pantalla con errores y el botón habilitado |
| «Cancelar» (enlace) | `/dashboard` |

**Wireframe (360 px):**

```
┌────────────────────────────────────────┐
│ CMP-01                                  │
├────────────────────────────────────────┤
│ <h1> Nuevo gasto                       │
│                                        │
│ [CMP-04 resumen de errores]            │ solo con errores
│                                        │
│ Monto                                  │ <label for="Amount">
│ ┌────┬───────────────────────────────┐ │ input-group, 56 px de alto
│ │ S/ │ 3,50                          │ │ «S/» aria-hidden; texto 1.25rem, tabular-nums
│ └────┴───────────────────────────────┘ │
│ Puedes usar coma o punto. Ej.: 3,50    │ id="Amount-help" [Propuesta], 0.875rem muted
│ ⚠ MSG-06                               │ id="Amount-error"
│                                        │
│ <fieldset> <legend> Categoría          │
│ ┌────────────┐┌────────────┐           │ cuadrícula 2 columnas (3 en ≥ 576 px)
│ │○Alimentación││○Transporte │           │ cada mosaico: <label> que contiene el radio,
│ └────────────┘└────────────┘           │ 56 px de alto, borde 1 px --ef-color-border
│ ┌────────────┐┌────────────┐           │ seleccionado: borde 2 px primario, fondo
│ │○ Estudios  ││○Entretenim.│           │ --ef-primary-subtle y radio marcado visible
│ └────────────┘└────────────┘           │ (el texto completo «Entretenimiento» se
│ ┌────────────┐┌────────────┐           │  ajusta en dos líneas; no se abrevia)
│ │○ Salud     ││○ Otros     │           │
│ └────────────┘└────────────┘           │
│ ⚠ MSG-12                               │
│                                        │
│ <fieldset> <legend> Medio de pago      │
│ ┌──────┐┌──────┐┌──────┐               │ cuadrícula 3 columnas
│ │○Efect││●Yape ││○Plin │               │ «Efectivo» completo, ajusta en 2 líneas
│ └──────┘└──────┘└──────┘               │
│ ┌──────┐┌──────┐                       │
│ │○Tarj.││○Otro │                       │ «Tarjeta» completo
│ └──────┘└──────┘                       │
│                                        │
│ Fecha                                  │
│ ┌────────────────────────────────────┐ │ selector nativo
│ │ 08/11/2026                         │ │
│ └────────────────────────────────────┘ │
│                                        │
│ ┌────────────────────────────────────┐ │
│ │            Guardar gasto           │ │ btn-primary, w-100, 56 px
│ └────────────────────────────────────┘ │
│              Cancelar                  │ enlace centrado
└────────────────────────────────────────┘
```

**Escritorio:** formulario en una columna de máx. 560 px, centrada. Categoría en 3 columnas, medio de pago en 5.

**Estados:**

| Estado | Qué se ve |
|---|---|
| Inicial | Fecha de hoy, medio preseleccionado, sin categoría |
| Errores | Valores conservados; errores bajo cada campo; con los 4 vacíos, MSG-11, MSG-12, MSG-13 y MSG-16 a la vez |
| Enviando | Botón `disabled` + «Guardando…»; se vuelve a habilitar si el servidor responde con errores (es una página nueva) |
| Sesión expirada | El `POST` redirige a P-002 con MSG-48 (RN-171); tras iniciar sesión, P-004 vacío (DEC-59) |

---

### CMP-05 — Campo Monto

- `<input type="text" inputmode="decimal" autocomplete="off" spellcheck="false">` (no `type="number"`: no admite la coma en todos los navegadores y oculta el texto original, que el servidor necesita para RN-104).
- Prefijo «S/» como `input-group-text` con `aria-hidden="true"`; la etiqueta es «Monto» y la ayuda menciona los soles.
- Texto a 1.25rem para lectura cómoda; alineado a la izquierda.
- Sin formateo automático mientras se escribe (no hay JS de máscara).

### CMP-06 — Grupo de opciones en mosaico

- `<fieldset>` + `<legend>` visible (1rem, 600).
- Cada opción: `<label class="ef-choice">` que envuelve un `<input type="radio" class="ef-choice__input">` **visible** (no `display:none`, para conservar el foco y el anuncio nativo) y el texto.
- Navegación con flechas dentro del grupo (comportamiento nativo de radios); `Tab` entra y sale del grupo.
- Estado seleccionado: borde 2 px `--ef-color-primary`, fondo `--ef-primary-subtle`, texto 600 y el radio marcado. Así el estado no depende solo del color.
- Foco: `--ef-focus-ring` sobre el mosaico completo (`:has(:focus-visible)`; si el navegador no soporta `:has`, el anillo queda en el radio).

### CMP-10 — Botón de envío con estado «enviando»

- Botón `<button type="submit" data-disable-on-submit data-submitting-text="…">` con texto de reposo (ADR-006, punto 3). `site.js` solo actúa sobre botones con `data-disable-on-submit`.
- `site.js`: en el evento `submit` del formulario, si no hay otro envío en curso, pone `disabled` y `aria-disabled="true"`, cambia el texto al de envío y escribe el mismo texto en una región `aria-live="polite"` del formulario. Un segundo `submit` se cancela con `preventDefault()`.
- `pageshow` con `event.persisted` (volver con «Atrás» desde la caché del navegador) restaura el botón.
- Lo exigen RN-131 y RN-168; en P-001, P-002, P-009 y P-010 lo aplica DEC-74.

| Formulario | Texto en reposo | Texto al enviar |
|---|---|---|
| P-001 | «Crear cuenta» | «Creando cuenta…» `[Propuesta]` |
| P-002 | «Iniciar sesión» | «Iniciando sesión…» `[Propuesta]` |
| P-004 | «Guardar gasto» | «Guardando…» `[Propuesta]` |
| P-005 | «Guardar ingreso» | «Guardando…» `[Propuesta]` |
| P-007 | «Guardar cambios» | «Guardando…» `[Propuesta]` |
| P-009 | «Guardar presupuesto» | «Guardando…» `[Propuesta]` |
| P-010 | «Guardar cambios» | «Guardando…» `[Propuesta]` |
| P-003 IA | «Generar resumen del mes» | «Generando resumen…» (HU-010A) |

### CMP-04 — Resumen de errores

```
┌ div role="alert" class="ef-error-summary" ──────────────┐  fondo --ef-danger-bg, borde izq. 4 px
│ ⚠ Revisa los datos ingresados.                          │  MSG-44, 600
│   • MSG-11  ← enlace a #Amount                          │  lista de enlaces a cada campo con error
│   • MSG-12  ← enlace a #Category-legend                 │
└─────────────────────────────────────────────────────────┘
```

- Sin `tabindex`: el foco va al primer campo con error, no al resumen (DEC-78). `role="alert"` hace que el lector de pantalla lo anuncie al cargar.
- Solo agrupa errores de campo; los que no son de un campo van en su propio bloque (§3.0, DEC-80).

---

### P-005 — Nuevo ingreso

| Campo | Valor |
|---|---|
| Propósito | Registrar el dinero recibido con su origen |
| Actor | Usuario autenticado |
| HU | HU-004, HU-011 |
| Reglas | RN-026, RN-051 a RN-053, RN-056 a RN-061, RN-074, RN-104, RN-106, RN-107, RN-109, RN-110, RN-116, RN-168, RN-172 |
| Layout | CMP-01 |

**Campos (sin medio de pago):**

| # | Etiqueta | Control | Valor inicial | Errores |
|---|---|---|---|---|
| 1 | «Monto» | CMP-05 | vacío | MSG-11, MSG-06 a MSG-09 |
| 2 | «Origen» | CMP-06, 5 radios: Apoyo familiar, Trabajo, Prácticas preprofesionales, Trabajo independiente, Otros | ninguno | MSG-14 |
| 2b | «Descripción (opcional)» | `<input type="text">` **sin** `maxlength` (el servidor rechaza 101 caracteres con MSG-15; no se trunca, DEC-32) | vacío | MSG-15 |
| 3 | «Fecha» | `<input type="date">` | hoy de Lima | MSG-16, MSG-10, MSG-17 |

**Campo «Descripción»:**

- **Con JS:** el contenedor de «Descripción» está oculto (`hidden`) y su `<input>` con `disabled` mientras el origen no sea «Otros». Al elegir «Otros», `site.js` quita `hidden` y `disabled`; al elegir otro origen, los vuelve a poner (un campo deshabilitado no se envía, así nunca llega una descripción con otro origen). El cambio no mueve el foco.
- **Sin JS:** el campo es visible siempre, con la ayuda «Solo si elegiste «Otros».» `[Propuesta]`. Si llega una descripción con un origen distinto de «Otros», el servidor MVC **la descarta** y guarda `SourceDescription = null`, sin error (DEC-66). La API no cambia: sigue respondiendo `400` con MSG-26.

**Acciones:** «Guardar ingreso» (CMP-10) → `/dashboard` con MSG-50; «Cancelar» → `/dashboard`.

**Wireframe (360 px), con «Otros» elegido:**

```
┌────────────────────────────────────────┐
│ <h1> Nuevo ingreso                     │
│ Monto                                  │
│ ┌────┬───────────────────────────────┐ │
│ │ S/ │ 80.00                         │ │
│ └────┴───────────────────────────────┘ │
│ <fieldset><legend> Origen              │ 1 columna (textos largos), 56 px por opción
│ ┌────────────────────────────────────┐ │
│ │ ○ Apoyo familiar                   │ │
│ │ ○ Trabajo                          │ │
│ │ ○ Prácticas preprofesionales       │ │
│ │ ○ Trabajo independiente            │ │
│ │ ● Otros                            │ │
│ └────────────────────────────────────┘ │
│ Descripción (opcional)                 │ aparece solo con «Otros» (con JS)
│ ┌────────────────────────────────────┐ │
│ │ Venta de libros                    │ │
│ └────────────────────────────────────┘ │
│ Hasta 100 caracteres.                  │ ayuda [Propuesta]
│ Fecha                                  │
│ ┌────────────────────────────────────┐ │
│ │ 08/11/2026                         │ │
│ └────────────────────────────────────┘ │
│ ┌────────────────────────────────────┐ │
│ │          Guardar ingreso           │ │
│ └────────────────────────────────────┘ │
│              Cancelar                  │
└────────────────────────────────────────┘
```

**Estados:** inicial; errores (MSG-14 sin origen; MSG-11 / MSG-16 / MSG-10 en monto y fecha); enviando; «Otros» elegido tras un error (el servidor renderiza el campo visible y habilitado).

---

### P-006 — Lista de movimientos

| Campo | Valor |
|---|---|
| Propósito | Ver todos los movimientos, ordenados y paginados, y corregir o eliminar cualquiera |
| Actor | Usuario autenticado |
| HU | HU-005, HU-006, HU-011 |
| Reglas | RN-007, RN-079, RN-080, RN-117, RN-125, RN-162, RN-172, RN-178 |
| Layout | CMP-01, «Movimientos» activo |

**Datos:** 20 movimientos por página (RN-117), orden `Date DESC, CreatedAt DESC, Id DESC`, con CMP-07 y botones «Editar» / «Eliminar». Total y página actual en texto.

**Acciones:**

| Acción | Resultado |
|---|---|
| «Editar» en una fila | P-007 (`/transactions/{id}/edit?page=n`, para volver a la misma página, DEC-65) |
| «Eliminar» en una fila | P-008 |
| «Anterior» / «Siguiente» | `?page=n-1` / `?page=n+1` |
| «Nuevo gasto» / «Nuevo ingreso» | P-004 / P-005 |

**Wireframe (360 px):**

```
┌────────────────────────────────────────┐
│ CMP-01 (Movimientos activo)             │
├────────────────────────────────────────┤
│ <h1> Movimientos                       │
│ 25 movimientos · Página 1 de 2         │ [Propuesta] texto 0.875rem muted
│ ┌──────────────────┐┌─────────────────┐│
│ │ + Nuevo gasto    ││ + Nuevo ingreso ││
│ └──────────────────┘└─────────────────┘│
│                                        │
│ ┌ ol class="ef-tx-list" ─────────────┐ │
│ │ 01/11/2026 · Transporte · Gasto     │ │ CMP-07 con acciones
│ │                          S/ 5.00    │ │
│ │ [ Editar ]  [ Eliminar ]            │ │
│ │─────────────────────────────────────│ │
│ │ …19 filas más…                      │ │
│ └─────────────────────────────────────┘ │
│                                        │
│ <nav aria-label="Paginación">          │ CMP-12
│ ┌──────────┐  Página 1 de 2 ┌─────────┐│
│ │ Anterior │                │Siguiente││ «Anterior» deshabilitado en la página 1
│ └──────────┘                └─────────┘│ (renderizado como <span aria-disabled="true">)
└────────────────────────────────────────┘
```

**Escritorio:** misma lista; cada fila en una sola línea: texto a la izquierda, monto y acciones a la derecha.

**Estados:**

| Estado | Qué se ve |
|---|---|
| Con movimientos | Lista + paginación (solo si hay más de 1 página) |
| Sin movimientos (`demo4`) | MSG-41 (no MSG-31) en un bloque vacío centrado + los dos botones de alta |
| Página posterior a la última | MSG-31 + enlace «Ir a la página 1» `[Propuesta: texto del enlace]` |
| Página `0`, `-1`, `abc` | El servidor muestra la página 1 (sin mensaje) |
| Tras editar | La misma página con MSG-51 («Cambios guardados.») (DEC-65, RN-172) |
| Tras eliminar | La misma página con MSG-52 («Movimiento eliminado.»). Si la página quedó vacía y no es la primera, la página anterior con MSG-52 (RN-178) |

**Should (HU-016, no lista — PQ-08):** sobre la lista, un `<form method="get">` plegable con `<details><summary>Filtrar</summary>`: rango de fechas (dos `type="date"`), categoría, tipo y medio de pago (`<select>`), botones «Aplicar filtros» y «Quitar filtros». Los filtros se conservan en la paginación (`?page=2&category=…`). Bloqueado por las reglas y mensajes que faltan en HU-016.

---

### P-007 — Editar movimiento

| Campo | Valor |
|---|---|
| Propósito | Corregir un gasto o un ingreso |
| Actor | Usuario autenticado (dueño del movimiento) |
| HU | HU-006, HU-011 |
| Reglas | RN-008, RN-020, RN-021, RN-053, RN-109 a RN-112, RN-160, RN-168, RN-172 |
| Layout | CMP-01, «Movimientos» activo |

**Campo oculto:** `page`, con la página de P-006 de origen (1 si no llega o no es válida).

**Datos y campos:** los mismos de P-004 (gasto) o P-005 (ingreso), precargados con los valores actuales. Arriba, el tipo como **texto no editable**: «Tipo: Gasto» o «Tipo: Ingreso» (el tipo no se puede cambiar, RN-112/MSG-30). Se aplican todas las validaciones de creación (DEC-34), incluida la ventana de fechas (MSG-17, también para movimientos antiguos que ya están fuera).

**Acciones:**

| Acción | Resultado |
|---|---|
| «Guardar cambios» (CMP-10) | Válido ⇒ `303` a `/transactions?page=n`, la página de P-006 desde la que se abrió, con MSG-51 (DEC-65). Inválido ⇒ misma pantalla |
| «Cancelar» (enlace) | `/transactions?page=n`, la misma página de origen (DEC-65) |

**Wireframe (360 px), gasto:**

```
┌────────────────────────────────────────┐
│ <h1> Editar gasto                      │ «Editar ingreso» para ingresos
│ Tipo: Gasto                            │ texto, 0.875rem muted
│ [CMP-04]                               │
│ Monto        [ S/ | 30.00          ]   │
│ Categoría    (mosaicos, «Otros» marcado)│
│ Medio de pago (mosaicos)               │
│ Fecha        [ 05/11/2026          ]   │
│ ┌────────────────────────────────────┐ │
│ │          Guardar cambios           │ │
│ └────────────────────────────────────┘ │
│              Cancelar                  │
└────────────────────────────────────────┘
```

**Estados:** precargado; errores (p. ej. `12.345` ⇒ MSG-09 y el valor guardado sigue siendo S/ 30.00); fecha fuera de ventana (MSG-17 bajo «Fecha»); movimiento ajeno o eliminado ⇒ P-012.

**Nota:** la pantalla no tiene botón «Eliminar» `[Propuesta]`: la eliminación se hace desde P-006, como dice HU-006.

---

### P-008 — Confirmar eliminación

| Campo | Valor |
|---|---|
| Propósito | Evitar borrados accidentales; el borrado es físico y definitivo |
| Actor | Usuario autenticado |
| HU | HU-006, HU-011 |
| Reglas | RN-020, RN-114, RN-115, RN-172, RN-178 |

**Datos:** MSG-18 y la descripción de la fila («01/11/2026 · Transporte · Gasto · S/ 5.00»).

**Con JS (diálogo):**

- `<dialog id="ef-confirm-delete" aria-labelledby="dlg-title" aria-describedby="dlg-desc">` abierto con `showModal()` (modal nativo: atrapa el foco y cierra con `Esc`).
- Al abrirse, el foco va a «Cancelar» (la opción segura).
- «Cancelar» o `Esc` cierran y **devuelven el foco al botón «Eliminar» que lo abrió** (HU-006).
- «Eliminar» es un botón de un `<form method="post" action="/transactions/{id}/delete">` con token antiforgery.

```
┌────────── fondo oscurecido ───────────┐
│  ┌──────────────────────────────────┐ │ ancho: 100 % − 32 px (máx. 420 px)
│  │ <h2 id="dlg-title">              │ │
│  │   Eliminar movimiento            │ │ [Propuesta]
│  │ <p id="dlg-desc"> MSG-18         │ │
│  │ 01/11/2026 · Transporte · Gasto  │ │
│  │ · S/ 5.00                        │ │
│  │ ┌──────────────┐┌──────────────┐ │ │ 48 px
│  │ │   Cancelar   ││   Eliminar   │ │ │ secundario / btn-danger
│  │ └──────────────┘└──────────────┘ │ │
│  └──────────────────────────────────┘ │
└───────────────────────────────────────┘
```

**Sin JS (DEC-67):** sin ruta nueva. En cada fila de P-006, «Eliminar» es un `<details class="ef-delete">`:

```html
<details class="ef-delete">
  <summary class="btn btn-outline-danger btn-sm" aria-label="Eliminar gasto del 01/11/2026 de S/ 5.00">Eliminar</summary>
  <div class="ef-delete__panel">
    <p>MSG-18</p>
    <form method="post" action="/transactions/{id}/delete">
      <!-- token antiforgery; campo oculto page -->
      <button type="submit" class="btn btn-danger">Eliminar definitivamente</button>
    </form>
  </div>
</details>
```

- El panel se abre bajo la fila, dentro del flujo normal (no superpuesto). Para cancelar, se vuelve a pulsar «Eliminar» (`<summary>`), que conserva el foco.
- Con JS, `site.js` intercepta el clic en el `<summary>` (`preventDefault()`), abre el `<dialog>` descrito arriba y deja el `<details>` cerrado. Así existe un solo marcado y el comportamiento sin JS sigue disponible.
- El formulario del diálogo y el del `<details>` envían el mismo `POST` con el campo oculto `page` (DEC-65). Tras eliminar, P-006 muestra MSG-52 (RN-172) y, si la página quedó vacía y no es la primera, la anterior (RN-178).

---

### CMP-11 — Diálogo de confirmación de eliminación

**Usado en:** P-006 / P-008. **HU:** HU-006. **Reglas:** RN-114 (DEC-67).

| Parte | Regla |
|---|---|
| Sin JavaScript | `<details class="ef-delete">` en la fila: `<summary>` «Eliminar» con `aria-label` completo; panel con MSG-18 y el botón «Eliminar definitivamente» dentro de un `<form method="post">` con token y `page` |
| Con JavaScript | `<dialog>` modal (`showModal()`), `aria-labelledby` y `aria-describedby`; foco inicial en «Cancelar»; `Esc` o «Cancelar» cierran y devuelven el foco al «Eliminar» de la fila |
| Botones | «Cancelar» secundario y «Eliminar» `btn-danger`, 48 px de alto; el texto nunca depende solo del color |
| Ancho | 100 % − 32 px, máximo 420 px |

El marcado completo y el wireframe están en P-008.

---

### P-009 — Crear presupuesto

| Campo | Valor |
|---|---|
| Propósito | Fijar el límite de gasto del mes en curso |
| Actor | Usuario autenticado sin presupuesto del mes |
| HU | HU-007 |
| Reglas | RN-022, RN-053, RN-055, RN-062 a RN-064, RN-104, RN-118, RN-139, RN-167, RN-168, RN-172, RN-175 |
| Layout | CMP-01, «Presupuesto» activo |

**Campos:**

| Etiqueta | Control | Errores |
|---|---|---|
| «Monto máximo de gasto» `[Propuesta]` | CMP-05 | MSG-25, MSG-19, MSG-20, MSG-09 |
| (oculto) `month` = `yyyy-MM` del mes en curso | `<input type="hidden">` (no cuenta como campo, RN-167) | MSG-22 (bloque arriba) |

**Datos:** el mes en texto: «Presupuesto para noviembre de 2026» `[Propuesta]`, y una ayuda: «Solo cuenta para este mes. Los ingresos no lo modifican.» `[Propuesta]` (RN-026).

**Acciones:** «Guardar presupuesto» `[Propuesta]` (CMP-10) → `/dashboard` con MSG-53; «Cancelar» → `/dashboard`.

**Redirección:** si ya existe el presupuesto del mes, `GET /budgets/create` lleva a P-010 (RN-175).

**Wireframe (360 px):**

```
┌────────────────────────────────────────┐
│ <h1> Crear presupuesto                 │
│ Presupuesto para noviembre de 2026     │
│                                        │
│ ┌ alert role="alert" ────────────────┐ │ solo con MSG-21 o MSG-22
│ │ ⚠ MSG-21  Editar presupuesto →     │ │ con MSG-21, enlace a P-010
│ └────────────────────────────────────┘ │
│                                        │
│ Monto máximo de gasto                  │
│ ┌────┬───────────────────────────────┐ │
│ │ S/ │ 600                           │ │
│ └────┴───────────────────────────────┘ │
│ Solo cuenta para este mes. Los         │
│ ingresos no lo modifican.              │
│ ┌────────────────────────────────────┐ │
│ │         Guardar presupuesto        │ │
│ └────────────────────────────────────┘ │
│              Cancelar                  │
└────────────────────────────────────────┘
```

**Estados:** inicial; error de monto; MSG-21 (ya existe, bloque `role="alert"` con enlace a P-010); MSG-22 (cambió el mes entre abrir y enviar): el formulario se vuelve a mostrar con el mes nuevo en el rótulo y en `month`, y conserva el monto escrito (DEC-84).

**Sin botón para eliminar** (RN-119).

---

### P-010 — Editar presupuesto

| Campo | Valor |
|---|---|
| Propósito | Cambiar el monto del presupuesto del mes en curso |
| Actor | Usuario autenticado con presupuesto del mes |
| HU | HU-007 |
| Reglas | RN-062, RN-063, RN-065, RN-118, RN-119, RN-167, RN-168, RN-172, RN-176 |
| Layout | CMP-01, «Presupuesto» activo |

Igual que P-009 con estas diferencias: `<h1>` «Editar presupuesto»; el monto viene precargado (`600.00`); el botón es «Guardar cambios»; **no hay** botón ni enlace para eliminar; errores MSG-19 (con `0` se conserva el monto anterior), MSG-20, MSG-09, MSG-25, MSG-22. Con MSG-22 (el presupuesto ya no es del mes en curso), el bloque `role="alert"` incluye el enlace «Crear presupuesto» a P-009 (DEC-84). Guardar ⇒ `/dashboard` con MSG-53. Sin presupuesto del mes, `GET /budgets/edit` lleva a P-009 (RN-176).

**Wireframe (360 px), tras MSG-22:**

```
┌────────────────────────────────────────┐
│ CMP-01 (Presupuesto activo)             │
├────────────────────────────────────────┤
│ <h1> Editar presupuesto                │
│ Presupuesto para noviembre de 2026     │
│ ┌ alert role="alert" ────────────────┐ │ solo con MSG-22
│ │ ⚠ MSG-22  Crear presupuesto →      │ │ enlace a P-009 (DEC-84)
│ └────────────────────────────────────┘ │
│ Monto máximo de gasto                  │
│ ┌────┬───────────────────────────────┐ │
│ │ S/ │ 600.00                        │ │ precargado
│ └────┴───────────────────────────────┘ │
│ ┌────────────────────────────────────┐ │
│ │          Guardar cambios           │ │ CMP-10
│ └────────────────────────────────────┘ │
│              Cancelar                  │
└────────────────────────────────────────┘
```

---

### P-011 — Error inesperado

| Campo | Valor |
|---|---|
| Propósito | Explicar en español que algo falló, sin detalles técnicos |
| Actor | Cualquiera |
| HU | HU-013 |
| Reglas | RN-164, RN-006, RN-133 |
| Layout | CMP-02 (no consulta la base de datos ni el usuario: debe funcionar con SQL Server detenido) |

```
┌────────────────────────────────────────┐
│ ExpenseFlow AI                          │
├────────────────────────────────────────┤
│ <h1> Algo salió mal                    │ [Propuesta]
│ MSG-40                                 │
│ Código de error: 500                   │ [Propuesta]; HU-013 pide el código
│ ┌────────────────────────────────────┐ │
│ │         Volver al inicio           │ │ enlace a /dashboard
│ └────────────────────────────────────┘ │
└────────────────────────────────────────┘
```

Sin traza, nombre de clase ni texto en inglés. En `Development` se muestra la página de excepciones del desarrollador (HU-013).

---

### P-012 — Página no encontrada

| Campo | Valor |
|---|---|
| Propósito | Responder `404` en español a rutas inexistentes o a recursos ajenos, sin revelar si existen |
| Actor | Cualquiera |
| HU | HU-011, HU-002A (`/Identity/Account/Manage`, `/Identity/Account/ForgotPassword`) |
| Reglas | RN-169, RN-008, RN-145 |
| Layout | CMP-02 |

```
┌────────────────────────────────────────┐
│ <h1> Página no encontrada              │ [Propuesta]
│ MSG-45                                 │ «La página que buscas no existe.» (DEC-64)
│ Código de error: 404                   │
│ [ Volver al inicio ]                   │
└────────────────────────────────────────┘
```

El mismo texto para «no existe» y «pertenece a otro usuario».

---

### P-016 — Formulario caducado

| Campo | Valor |
|---|---|
| Propósito | Responder en español a un formulario MVC enviado con un token antiforgery no válido, sin modificar datos |
| Actor | Cualquiera |
| HU | HU-011 (y HU-001, HU-002A cuando el formulario se abrió antes de iniciar sesión en otra pestaña) |
| Reglas | RN-101, RN-170 |
| Layout | CMP-02 |

```
┌────────────────────────────────────────┐
│ <h1> Formulario caducado               │ [Propuesta]
│ MSG-46                                 │ «Tu formulario caducó. Vuelve a la página e inténtalo de nuevo.»
│ Código de error: 400                   │
│ [ Volver al inicio ]                   │ enlace a /dashboard (sin sesión, lleva a P-002)
└────────────────────────────────────────┘
```

Sin traza ni texto en inglés. `POST /account/logout` sin sesión no llega a esta página (RN-177).

---

### P-013 — Frecuencia de entrega (Should, HU-012)

> **Borrador:** no se implementa hasta que los Must estén terminados con CI en verde y se resuelva PQ-07 (mensajes de RN-143 y RN-161 y rótulo del saldo del periodo).

| Campo | Valor |
|---|---|
| Propósito | Indicar cada cuánto se recibe dinero y la fecha de la última entrega |
| HU | HU-012 |
| Reglas | RN-089 a RN-093, RN-135, RN-136, RN-143, RN-161 |

**Campos:** `<fieldset>` «¿Cada cuánto recibes dinero?» con 3 radios exactamente: «Cada 7 días», «Cada 15 días», «Mensual» (nunca «Quincenal»); «Fecha de tu última entrega» (`type="date"`). Botón «Guardar». Errores: `[Pendiente PQ-07]`.

**Efecto en P-003:** en el bloque de saldos, después de «Saldo acumulado», un `<h3>` con el saldo del periodo y el rango de fechas (`[Pendiente PQ-07: rótulo]`). El presupuesto y los avisos siguen siendo mensuales (DEC-21).

### P-014 — Presupuesto por categoría (Should, HU-015, no lista)

Formulario con un campo Monto (CMP-05) por cada categoría de RN-014 y, en P-003, una barra CMP-08 por categoría con presupuesto. Bloqueado por PQ-08 (modelo de datos, validaciones y mensajes).

### P-015 — Metas de ahorro (Should, HU-017, no lista)

Lista de metas con nombre, monto objetivo, avance en texto y CMP-08. Bloqueado por PQ-08 (RN-046: cómo se marca un monto como ahorro).

---

## 4. Design tokens

Los tokens se definen como variables CSS en `wwwroot/css/site.css`, cargado **después** de `bootstrap.min.css`. Los que tienen equivalente en Bootstrap se asignan también a `--bs-*`.

### 4.1 Colores

Todos los contrastes están calculados con la fórmula de WCAG 2.2.

| Token | Valor | Uso | Contraste verificado |
|---|---|---|---|
| `--ef-color-primary` | `#0B5C8A` | Botones primarios, enlaces, cabecera, barra normal | Blanco sobre él 7.21:1; sobre `--ef-color-bg` 6.72:1 |
| `--ef-color-primary-hover` | `#084A6F` | Hover y activo del primario | Blanco sobre él 9.46:1 |
| `--ef-primary-subtle` | `#E8F2F9` | Fondo de opción seleccionada, aviso informativo | `#0B4F75` sobre él 7.74:1 |
| `--ef-color-secondary` | `#475467` | Botones secundarios, insignias «Gasto»/«Ingreso» | Blanco sobre él 7.69:1 |
| `--ef-color-accent` | `#0F6E5C` | Borde del bloque de IA, detalles de marca | Sobre blanco 6.17:1 |
| `--ef-color-bg` | `#F5F7FA` | Fondo de página | — |
| `--ef-color-surface` | `#FFFFFF` | Tarjetas, formularios, diálogos | — |
| `--ef-color-text` | `#1A1F29` | Texto principal | Sobre blanco 16.51:1; sobre fondo 15.38:1 |
| `--ef-color-text-muted` | `#4B5565` | Ayudas, leyendas, pie | Sobre blanco 7.54:1; sobre fondo 7.02:1 |
| `--ef-color-border` | `#6B7280` | Bordes de campos, mosaicos y barra (componentes) | Sobre blanco 4.83:1 (≥ 3:1, WCAG 1.4.11) |
| `--ef-color-divider` | `#E4E8EE` | Separadores decorativos y pista de la barra | Decorativo; no transmite información |
| `--ef-color-danger` | `#B42318` | Texto de error, saldo negativo | Sobre blanco 6.57:1 |
| `--ef-danger-bg` / `--ef-danger-text` | `#FEF0EE` / `#912018` | Aviso del 100 %, resumen de errores | 7.80:1 |
| `--ef-warning-bg` / `--ef-warning-text` | `#FFF4E0` / `#7A3E00` | Aviso del 80 %, MSG-23 | 7.66:1 |
| `--ef-success-bg` / `--ef-success-text` | `#ECF7EF` / `#1F6E37` | Confirmaciones MSG-49 a MSG-53 (RN-172) | 5.72:1 |
| `--ef-info-bg` / `--ef-info-text` | `#E8F2F9` / `#0B4F75` | Aviso de datos ficticios, MSG-48 | 7.74:1 |
| `--ef-progress-normal` | `#0B5C8A` | Barra < 80 % | Sobre la pista 5.86:1 |
| `--ef-progress-warning` | `#B45309` | Barra 80 %–<100 % | Sobre la pista 4.08:1 (≥ 3:1) |
| `--ef-progress-danger` | `#B42318` | Barra ≥ 100 % | Sobre la pista 5.35:1 |
| `--ef-focus-ring` | `0 0 0 3px #FFFFFF, 0 0 0 6px #0B5C8A` | Indicador de foco (doble anillo: visible sobre fondos claros y sobre la cabecera azul) | Azul sobre fondo 6.72:1 |

**Reglas de uso del color:**

- Sin modo oscuro en el MVP: `color-scheme: light` en `:root`.
- El rojo, el ámbar y el verde nunca son la única señal: siempre van con texto (y un icono decorativo).
- Los importes se escriben en `--ef-color-text`, salvo el saldo negativo (`--ef-color-danger` + texto).

### 4.2 Tipografía

| Token | Valor |
|---|---|
| `--ef-font-family` | `system-ui, -apple-system, "Segoe UI", Roboto, "Noto Sans", "Helvetica Neue", Arial, sans-serif` (Roboto en Android) |
| `--ef-font-numeric` | `font-variant-numeric: tabular-nums` en importes, fechas y porcentajes |
| Tamaño base | `16px` (`1rem`); los campos nunca bajan de 16 px (evita el zoom automático en móviles) |
| Interlineado | `1.5` en general, `1.6` en el texto de la IA |

| Estilo | Móvil | ≥ 768 px | Peso | Uso |
|---|---|---|---|---|
| `h1` | 1.5rem | 1.75rem | 700 | Título de pantalla |
| `h2` | 1.125rem | 1.25rem | 600 | Título de sección |
| Importe destacado | 1.75rem | 2rem | 700 | Saldo del mes |
| Importe secundario | 1.25rem | 1.25rem | 700 | Saldo acumulado |
| Cuerpo | 1rem | 1rem | 400 | Texto, valores |
| Etiqueta | 1rem | 1rem | 600 | `<label>`, `<legend>` |
| Pequeño | 0.875rem | 0.875rem | 400 | Ayudas, leyendas, pie |

El texto debe seguir siendo legible con zoom al 200 % y con espaciado de texto aumentado (WCAG 1.4.4, 1.4.12): sin alturas fijas en contenedores de texto.

### 4.3 Espaciado, formas y tamaños

| Token | Valor | Uso |
|---|---|---|
| `--ef-space-1` … `--ef-space-6` | 4, 8, 12, 16, 24, 32 px | Escala de márgenes y padding |
| Gutter lateral | 16 px | Contenedor en móvil |
| `--ef-radius` | 0.5rem (8 px) | Tarjetas, campos, botones, mosaicos |
| `--ef-radius-sm` | 0.25rem | Insignias |
| Sombra de tarjeta | `0 1px 2px rgba(16, 24, 40, .08)` | Solo tarjetas |
| Ancho máx. de formularios | 560 px | P-004, P-005, P-007, P-009, P-010 |
| Ancho máx. de autenticación | 440 px | P-001, P-002 |
| Contenedor del dashboard | 1140 px | P-003, P-006 |

| Componente | Alto mínimo | Notas |
|---|---|---|
| Botón primario de formulario | 56 px | Ancho completo en móvil |
| Botón normal / enlace de navegación | 48 px | |
| Botón de fila («Editar», «Eliminar») | 44 px | `min-width: 44px` |
| Campo de texto / fecha | 48 px (Monto: 56 px) | Padding horizontal 12 px |
| Mosaico de opción | 56 px | |
| Casilla de verificación | 24 × 24 px dentro de un área clicable de 44 px | |
| Barra de progreso | 16 px | |
| Fila de movimiento | 56 px | |

### 4.4 Puntos de quiebre

Los de Bootstrap 5, diseñando primero para 360 px (RC-1, T4):

| Nombre | Desde | Cambios principales |
|---|---|---|
| base | 0 (diseño a 360 px) | Una columna; botones a ancho completo |
| `sm` | 576 px | Categorías en 3 columnas; filas de movimiento en una línea |
| `md` | 768 px | Cabecera y navegación en una franja |
| `lg` | 992 px | Dashboard en dos columnas |

### 4.5 Ejemplo de `site.css` (cabecera de tokens)

```css
:root {
  color-scheme: light;
  --ef-color-primary: #0B5C8A;
  --ef-color-primary-hover: #084A6F;
  --ef-primary-subtle: #E8F2F9;
  --ef-color-secondary: #475467;
  --ef-color-accent: #0F6E5C;
  --ef-color-bg: #F5F7FA;
  --ef-color-surface: #FFFFFF;
  --ef-color-text: #1A1F29;
  --ef-color-text-muted: #4B5565;
  --ef-color-border: #6B7280;
  --ef-color-divider: #E4E8EE;
  --ef-color-danger: #B42318;
  --ef-danger-bg: #FEF0EE;   --ef-danger-text: #912018;
  --ef-warning-bg: #FFF4E0;  --ef-warning-text: #7A3E00;
  --ef-success-bg: #ECF7EF;  --ef-success-text: #1F6E37;
  --ef-info-bg: #E8F2F9;     --ef-info-text: #0B4F75;
  --ef-progress-normal: #0B5C8A;
  --ef-progress-warning: #B45309;
  --ef-progress-danger: #B42318;
  --ef-focus-ring: 0 0 0 3px #FFFFFF, 0 0 0 6px #0B5C8A;
  --ef-radius: .5rem;

  /* Puente con Bootstrap 5 */
  --bs-primary: var(--ef-color-primary);
  --bs-primary-rgb: 11, 92, 138;
  --bs-body-bg: var(--ef-color-bg);
  --bs-body-color: var(--ef-color-text);
  --bs-border-color: var(--ef-color-border);
  --bs-border-radius: var(--ef-radius);
  --bs-link-color: var(--ef-color-primary);
  --bs-link-hover-color: var(--ef-color-primary-hover);
  --bs-body-font-family: system-ui, -apple-system, "Segoe UI", Roboto, "Noto Sans", "Helvetica Neue", Arial, sans-serif;
}
.btn-primary {
  --bs-btn-bg: var(--ef-color-primary);
  --bs-btn-border-color: var(--ef-color-primary);
  --bs-btn-hover-bg: var(--ef-color-primary-hover);
  --bs-btn-hover-border-color: var(--ef-color-primary-hover);
  --bs-btn-active-bg: var(--ef-color-primary-hover);
}
:focus-visible { outline: none; box-shadow: var(--ef-focus-ring); }
.ef-amount { font-variant-numeric: tabular-nums; }
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after { transition: none !important; animation: none !important; }
}
```

---

## 5. Navigation map

```
ExpenseFlow AI
│
├── Público (CMP-02, sin sesión)
│   ├── P-002 Iniciar sesión ........................ /account/login    ← destino de toda ruta protegida sin sesión (302); con sesión ──► P-003
│   │   ├── «¿No tienes cuenta? Crea una» ──► P-001
│   │   └── llegada por expiración ──► MSG-48 sobre el formulario
│   └── P-001 Crear cuenta .......................... /account/register (con sesión ──► P-003)
│       ├── «Crear cuenta» (válido) ──► P-003
│       └── «¿Ya tienes cuenta? Inicia sesión» ──► P-002
│
├── Autenticado (CMP-01: Inicio · Movimientos · Presupuesto · Cerrar sesión)
│   │
│   ├── P-003 Dashboard ............................. /dashboard          [nav: Inicio]
│   │   ├── «Nuevo gasto» ──► P-004 ................. /transactions/create?type=Expense
│   │   │                      └── «Guardar gasto» ──► P-003 (MSG-49)
│   │   ├── «Nuevo ingreso» ──► P-005 ............... /transactions/create?type=Income
│   │   │                      └── «Guardar ingreso» ──► P-003 (MSG-50)
│   │   ├── «Crear presupuesto» ──► P-009 (sin presupuesto)
│   │   ├── «Editar presupuesto» ──► P-010 (con presupuesto)
│   │   ├── «Ver todos los movimientos» ──► P-006
│   │   └── «Generar resumen del mes» ──► POST /insights/monthly-summary ──303──► P-003
│   │
│   ├── P-006 Movimientos ........................... /transactions?page=n  [nav: Movimientos]
│   │   ├── «Anterior» / «Siguiente» ──► P-006 (otra página)
│   │   ├── «Nuevo gasto» / «Nuevo ingreso» ──► P-004 / P-005
│   │   ├── «Editar» ──► P-007 ....................... /transactions/{id}/edit
│   │   │                └── «Guardar cambios» ──► P-006 (misma página, MSG-51)  · «Cancelar» ──► P-006 (misma página)
│   │   └── «Eliminar» ──► P-008 (diálogo)
│   │                      ├── «Cancelar» ──► P-006 (foco en el botón de origen; sin JS: <details> en la fila)
│   │                      └── «Eliminar» ──► POST /transactions/{id}/delete ──303──► P-006 (misma página o la anterior, MSG-52)
│   │
│   ├── Presupuesto (el servidor elige el destino)                        [nav: Presupuesto]
│   │   ├── P-009 Crear presupuesto ................. /budgets/create   (con presupuesto ──302──► P-010)
│   │   │   ├── «Guardar presupuesto» ──► P-003 (MSG-53)
│   │   │   └── MSG-21 ──► enlace a P-010
│   │   └── P-010 Editar presupuesto ................ /budgets/edit     (sin presupuesto ──302──► P-009)
│   │       ├── «Guardar cambios» ──► P-003 (MSG-53)
│   │       └── MSG-22 ──► enlace a P-009
│   │
│   ├── [Should] P-013 Frecuencia de entrega ........ ruta al especificar S2
│   ├── [Should] P-014 Presupuesto por categoría .... ruta al especificar S1
│   ├── [Should] P-015 Metas de ahorro .............. ruta al especificar S4
│   │
│   └── «Cerrar sesión» (POST /account/logout) ──► P-002 (también con la sesión expirada)
│
└── Sistema (CMP-02)
    ├── P-011 Error inesperado ...................... /Error  (500, ≠ Development)
    ├── P-012 No encontrada ......................... cualquier ruta inexistente o ajena (404)
    │   └── incluye /Identity/Account/Manage y /Identity/Account/ForgotPassword
    └── P-016 Formulario caducado ................... POST con token antiforgery no válido (400)

No existen (W17, W19, W20): recuperar contraseña, gestionar cuenta, «Recordarme».
```

**Profundidad máxima:** 2 toques desde el dashboard para cualquier acción Must (p. ej. Dashboard → Movimientos → Editar).

---

## 6. Decisiones aplicadas en esta especificación

Las decisiones están registradas en `03` §9. Esta tabla indica dónde se aplican en la UI.

| Decisión | Tema | Pantallas / componentes |
|---|---|---|
| DEC-63 | Rutas de cuenta | P-001, P-002, CMP-01 |
| DEC-64 | Página `404` con MSG-45, igual para inexistente y ajeno | P-012 |
| DEC-65 | Volver a la página de origen de la lista tras editar, eliminar o cancelar | P-006, P-007, P-008 |
| DEC-66 | Descripción descartada con un origen distinto de «Otros» (MVC sin JS) | P-005, P-007 |
| DEC-67 | Confirmación de eliminación con `<details>` / `<dialog>` | P-006, P-008, CMP-11 |
| DEC-68 | MSG-05 es el aviso de datos ficticios del registro | P-001 |
| DEC-69 | Confirmación breve tras guardar o eliminar (MSG-49 a MSG-53) | §3.0, P-003, P-006 |
| DEC-70 | `returnUrl` hacia rutas que solo admiten `POST` | F-01, P-002 |
| DEC-71 | Página `400` con MSG-46; cierre de sesión sin sesión sin error | P-016, CMP-01 |
| DEC-72 | `returnUrl` conservado tras errores | P-002 |
| DEC-73 | Usuario con sesión en registro o inicio de sesión ⇒ dashboard | P-001, P-002 |
| DEC-74 | CMP-10 en registro, inicio de sesión y presupuesto | CMP-10 |
| DEC-75 | Casilla conservada tras un error | P-001 |
| DEC-76 | Orden de los errores de contraseña | P-001 |
| DEC-77 | Correo con `type="text"` + `inputmode="email"` | P-001, P-002 |
| DEC-78 | Foco en el primer campo con error; CMP-04 sin foco | §3.0, CMP-04 |
| DEC-80 | Errores que no son de un campo en bloque `role="alert"` | §3.0, P-002, P-009, P-010 |
| DEC-81 | MSG-48 al llegar por expiración | P-002 |
| DEC-82 | Correo sin espacios exteriores en el inicio de sesión | P-002 |
| DEC-83 | Literales de las HU en el DOM | P-003, CMP-07 |
| DEC-84 | Mes nuevo tras MSG-22; enlace de P-010 a P-009 | P-009, P-010 |
| DEC-85 | Redirecciones entre «Crear» y «Editar presupuesto» | P-009, P-010 |
| DEC-86 | Página anterior tras eliminar el único movimiento de la página | P-006, P-008 |
| DEC-87 | Ayuda de la contraseña MSG-47 | P-001 |
| DEC-90 | Doble envío del resumen de IA sin JavaScript (limitación) | P-003 |

**ADR:** esta especificación no necesita ningún ADR nuevo; se apoya en ADR-006. Haría falta un ADR si se decide agregar una librería de iconos, una fuente web, un compilador Sass o una librería de gráficos.
