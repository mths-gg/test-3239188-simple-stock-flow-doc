# Requisitos — Simple Stock Flow

Las referencias `DM §n` apuntan a `spec/data-model.md`, único insumo técnico recibido. Lo que no surge del modelo se marca como supuesto o pendiente.

## Actores

- **Administrador:** rol `admin`; el administrador inicial se provisiona al arrancar la aplicación (DM §§1, 2.5, 9.2). Privilegios exactos por operación no especificados.
- **Vendedor:** rol `seller`, operador interno que registra ventas (DM §§1, 2.5).
- **Almacenamiento de imágenes:** sistema externo que guarda binarios con claves opacas (DM §§3, 7.1). Actor técnico inferido.

## Historias de usuario

### HU-01 Consultar catálogo

Como usuario de la aplicación quiero buscar productos activos por texto o categoría para encontrar artículos del catálogo.

- Búsqueda parcial, filtro por categoría, orden por nombre y paginación (DM §6 Q1).
- Consultar por ID y cargar productos activos por lote de IDs (DM §6 Q2–Q3).
- **Supuesto:** el modelo no define qué rol puede consultar ni respuestas para ID inexistente.

### HU-02 Mantener productos

Como usuario autorizado quiero mantener productos para que el catálogo represente los artículos vendibles.

- Producto: nombre, precio, stock, categoría obligatoria e imagen opcional; no se agregan descripción, SKU ni otros campos (DM §§1, 3, 12).
- Nombre recortado y no vacío; precio estrictamente positivo; stock nunca negativo; no permitir retirar más que el stock disponible (DM §2.2).
- Categoría debe existir. Imagen ausente es `NULL`, no cadena vacía (DM §2.2).
- Categorías: cinco semillas de solo lectura; no hay CRUD de categorías (DM §§2.1, 9).
- Producto dado de baja no se elimina físicamente; T-09 figura aplicada en el estado del 20-09 (DM §§2.2, 7.1, 13). El resultado SQL adjunto es del día anterior.
- **Supuesto:** rol autorizado para administrar catálogo no se define en la fuente.

### HU-03 Registrar venta

Como vendedor autenticado quiero registrar una venta para guardar qué se vendió y actualizar existencias.

- La venta guarda operador y fecha. Cada venta confirmada contiene al menos una línea (DM §§1, 2.3).
- Línea: producto, cantidad positiva y nombre/precio congelados al vender; subtotal calculado (DM §§1, 2.4).
- Un producto no puede repetirse dentro de una venta; no se confirma venta vacía (DM §2.3).
- Agregar línea y descontar stock forman una sola operación; rechazo si falta stock (DM §§2.2–2.3).
- Total se calcula, no se guarda. Sistema monomoneda (DM §§1, 3).
- Venta una vez registrada es inmutable y no se borra (DM §§1, 2.3, 7.1).
- **Supuesto:** el mecanismo transaccional concreto para guardar venta y stock debe verificarse en código.

### HU-04 Consultar ventas e informe

Como usuario autorizado quiero consultar ventas por fechas para revisar actividad comercial.

- Búsqueda por rango, orden fecha descendente y variantes paginada/no paginada (DM §6 Q7–Q8).
- La fecha final no puede ser anterior a la inicial (DM §1).
- Informe calculado al consultar, no persistido, sin desglose por vendedor (DM §§1, 7.1, 12).
- Usar valores históricos congelados; la decisión de negocio es agrupar por etiqueta congelada (DM §11.1), pero el almacenamiento de `category_name` depende de T-11, cuyo estado es contradictorio en la única versión (§§2.4, 3).
- **Conflicto pendiente:** CA-06.1 externo exige “una fila por producto”, incompatible con agrupar por etiqueta congelada cuando la categoría cambió. Se mantiene la decisión de §11.1 y se documenta CA-06.1 sin resolver.

### HU-05 Gestionar usuarios y acceso

Como administrador quiero administrar usuarios internos para controlar quién opera el sistema.

- Username único, recortado y minúsculo; rol dentro de `admin` / `seller` (DM §§1, 2.5).
- Dominio nunca recibe la clave en claro; conservar hash generado por puerto; no exponer hash en logs, respuestas ni errores (DM §§1, 7, 9.2).
- Administrador inicial provisionado en arranque desde credenciales del entorno; no insertar hash fijo por SQL (DM §9.2).
- Nadie asigna `admin` en uso normal; lo provisiona despliegue (DM §11 H-3).
- El estado más reciente del modelo declara A-1 cerrado: sin token → 401; usuario `seller` → 403 al intentar la operación administrativa (DM §13). §9.2 conserva una descripción anterior y debe actualizarse.

### HU-06 Gestionar imágenes

Como usuario autorizado quiero asociar imagen a producto para mostrarla en catálogo.

- DB conserva clave opaca `image_key`; bytes y ruta se guardan fuera (DM §§1, 3).
- Al reemplazar o borrar: anular referencia y confirmar DB antes de borrar binario (DM §7.1).
- **Pendiente operativo:** limpieza de binarios huérfanos (DM §11 H-2). Formatos/tamaños/permisos son **supuestos no especificados**.

## Requisitos no funcionales

| ID | Requisito derivado | Evidencia y límite |
|---|---|---|
| RNF-01 | El motor debe impedir stock negativo incluso ante SQL directo | `CHECK` reportado, DM §§2.2, 4; la consulta asociada es un snapshot del 19-09 |
| RNF-02 | Validar reglas que solo viven en dominio en todos los casos de uso | DM §§0, 2, 4; SQL directo puede saltárselas |
| RNF-03 | Concurrencia de stock sigue la estrategia optimista mencionada en ADR-002 | DM §§0, 2.2, 3; mecanismo detallado no está incluido |
| RNF-04 | No filtrar credenciales ni hash de contraseñas | DM §§2.5, 7 |
| RNF-05 | Mantener imágenes externas; privilegiar una referencia consistente aunque falle la limpieza | DM §§3, 7.1, 11 H-2 |
| RNF-06 | Retener ventas indefinidamente; usar baja lógica de producto | DM §§2, 7.1, 13; T-09 figura aplicada en la actualización del 20-09 |
| RNF-07 | Marcas temporales con zona horaria; servidor se reporta UTC | DM §3; evidencia fechada 19-09-2026 |
| RNF-08 | Restringir datos personales de username/operador y rol; hash nunca sale | DM §7 |
| RNF-09 | Índices justificados por patrones Q1–Q8 | DM §§6, 13; estado T-20 del 20-09 prevalece sobre la lista física del 19-09 |

## Exclusiones para evitar ampliar alcance

No cliente final, pagos/tarjetas, multimoneda, edición/borrado de ventas, CRUD de categorías, borrado físico de productos, campos adicionales de producto, reportes por vendedor ni `created_at`/`updated_at` (DM §§1, 2.1, 7.1, 8, 12).

## Pendientes de trazabilidad

Antes de cerrar criterios: actualizar snapshots SQL posteriores a los cambios de T-09 y T-20; resolver T-11 `category_name`; T-12 `sold_by_user_id`; y CA-06.1 frente a la agrupación por etiqueta congelada. Detalle en `auditoria.md`.
