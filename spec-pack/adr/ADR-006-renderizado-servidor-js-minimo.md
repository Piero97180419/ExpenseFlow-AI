# ADR-006 — Renderizado en servidor, PRG, validación solo en el servidor y JavaScript mínimo

| Campo | Valor |
|---|---|
| Estado | **Aprobado** |
| Versión | v1.0 |
| Fecha | 2026-11-10 |
| Aprobado por | Piero Carbajal |
| Fecha de aprobación | 2026-11-10 |
| PR de aprobación | PR que fusiona el Spec Pack v1.0 en `main` (ADR-005 §1) |
| Principios afectados | PRINCIPIO-002, PRINCIPIO-004, PRINCIPIO-008, PRINCIPIO-012 |
| Relacionado con | DEC-57, DEC-59, DEC-61, DEC-66, DEC-67, DEC-74, DEC-80, DEC-90 · RN-114, RN-131, RN-168, RN-172 · `09-ui-spec.md` §3.0 |

## Contexto

La persona principal usa un Android de gama media, y el diseño debe soportar también gama baja (CPU lenta, 2–3 GB de RAM) con conectividad intermitente (`01` §2.2). Los requisitos que condicionan la interfaz son:

- U2: registrar un gasto en 15 segundos o menos.
- T4: los flujos F-02, F-05 y F-06 deben funcionar desde Android.
- RN-027: el navegador no calcula cifras oficiales.
- W11: sin PWA ni modo offline.
- RN-116, HU-001 y HU-003: los mensajes de error aparecen en español junto a cada campo, con los textos de `03` §10, nunca con los avisos por defecto del navegador.

Con la validación del navegador (`required`, `maxlength`, `pattern`), el envío se bloquea antes de llegar al servidor y el usuario ve un aviso del navegador, a veces en inglés; además, `maxlength` impide llegar a MSG-15. La compresión de respuestas dinámicas sobre HTTPS, con tokens antiforgery en la página, expone a ataques BREACH.

## Decisión

1. Todas las pantallas son vistas Razor renderizadas en el servidor; los formularios usan Post/Redirect/Get (`303`).
2. **Validación solo en el servidor (DEC-61):** los formularios llevan `novalidate` y no usan `required`, `maxlength`, `min`/`max` ni `pattern`. Se mantienen `inputmode="decimal"` y `type="date"` por usabilidad. jQuery Validation, aunque está aprobada en ADR-001, no se referencia en las vistas.
3. JavaScript solo como **mejora progresiva**, en un único archivo local:
   - deshabilitar el botón de envío mientras se envía el formulario (RN-131, RN-168), mediante el atributo `data-disable-on-submit`;
   - mostrar el campo «Descripción» cuando el origen es «Otros» (sin JavaScript el campo está siempre visible; si llega con otro origen, el servidor lo descarta, DEC-66, nota de RN-110);
   - el diálogo de confirmación accesible de la eliminación (RN-114), como mejora sobre el `<details>` del punto 4.
   Sin JavaScript, todos los flujos funcionan. El mismo atributo `data-disable-on-submit` se usa en los formularios de registro, inicio de sesión y presupuesto (DEC-74).
4. **Confirmación de eliminación (RN-114, DEC-67):** la vista tiene un solo marcado.
   - **Sin JavaScript:** en cada fila de la lista, «Eliminar» es el `<summary>` de un `<details>`. Al abrirlo muestra MSG-18 y el botón «Eliminar definitivamente», que envía `POST /transactions/{id}/delete` con el token antiforgery y el campo oculto `page`. Para cancelar, se vuelve a pulsar «Eliminar». No hay ruta `GET` de confirmación.
   - **Con JavaScript:** el script intercepta el clic en el `<summary>`, abre un `<dialog>` modal nativo (`showModal()`) con MSG-18, pone el foco en «Cancelar» y, al cerrarlo, lo devuelve al «Eliminar» de esa fila (HU-006). «Eliminar» envía el mismo `POST`.
5. Bootstrap 5 se sirve desde `wwwroot/lib`, con `asp-append-version` y caché larga; sin CDN ni fuentes externas.
6. **Sin compresión de respuestas dinámicas** (riesgo BREACH sobre HTTPS con tokens antiforgery).
7. **Mensajes de confirmación y de error que no son de un campo:** las confirmaciones MSG-49 a MSG-53 viajan por TempData tras el `303` y se muestran una sola vez con `role="status"` (RN-172); MSG-02, MSG-21 y MSG-22 se muestran en un bloque con `role="alert"` (DEC-80).
8. Peso orientativo por página: ≤ 350 KB transferidos en la primera carga y ≤ 30 KB en las siguientes, con los estáticos en caché. Se registra en las pruebas E2E; no bloquea el CI.

## Alternativas descartadas

- **SPA o librería JavaScript adicional:** prohibida por PRINCIPIO-002 sin ADR; más peso y CPU en el teléfono; duplicaría cálculos en el cliente.
- **PWA con modo offline:** es W11 y exigiría resolver conflictos de sincronización.
- **jQuery Validation con mensajes en es-PE:** está aprobada, pero obliga a replicar cada MSG en el cliente y mantener ambos sincronizados.

## Consecuencias

- **Positivas:** funciona con JavaScript deshabilitado; la eliminación pide confirmación con o sin JavaScript, solo con elementos nativos de HTML (`<details>`, `<dialog>`), sin dependencias nuevas; las cifras salen siempre del servidor; los mensajes siempre están en español; menor consumo de batería y datos; PRG evita reenvíos al recargar.
- **Negativas:** cada error de validación cuesta una recarga de página. Sin JavaScript, o si se pierde la respuesta y el usuario reenvía, el doble envío sigue siendo posible (limitación aceptada, DEC-57; en el resumen de IA, DEC-90). Si la sesión expira durante un formulario, lo escrito se pierde (limitación aceptada, DEC-59).
