# Arquitectura — Simple Stock Flow

## Alcance y evidencia

Este documento deriva la arquitectura únicamente de `spec/data-model.md` (en adelante, DM). El repositorio de la actividad entrega ese modelo y el README, pero no contiene código, API, migraciones ni ADR. Las decisiones inferidas se etiquetan como supuestos.

## Estilo arquitectónico

El modelo menciona agregados y objetos de valor de C#, puertos para hashing, repositorios y lectura, y adaptadores para PostgreSQL/EF Core e imágenes externas (DM §§1–2, 6–7, 9, 12). Esto es compatible con arquitectura hexagonal. **Supuesto ARQ-01:** organizar el sistema así; el modelo no demuestra que esta arquitectura esté implementada ni que sea la única posible. No hay evidencia de microservicios.

## Componentes inferidos

| Componente | Responsabilidad | Evidencia / límite |
|---|---|---|
| Dominio | `Product`, `Sale`, `SaleItem`, `User`, `Category`; invariantes y valores `Money`, `Quantity` | DM §§1–2 |
| Aplicación | Coordinar casos de uso, puertos, autorización y operación de venta/stock | DM §§2.2–2.3, 7, 9.2. **Supuesto ARQ-02:** esta capa no está descrita por clases |
| Persistencia | Mapear agregados al esquema `sales` con EF Core y migraciones | DM §§0, 3, 5, 9, 12 |
| API | Exponer operaciones y controlar autenticación/autorización | DM §§7, 9.2, 12 menciona contrato externo no entregado. **Supuesto ARQ-03:** transporte API existente |
| Almacenamiento de imágenes | Guardar binarios externos; DB conserva `image_key` opaca | DM §§1, 3, 7.1 |
| Consultas/reportes | Consultar ventas por rango y agregar datos al leer; no se persiste reporte | DM §§1, 6, 11.1 |

## Agregados y flujos

- `Product` es raíz de catálogo; valida precio, stock, categoría e imagen (DM §2.2).
- `Sale` es raíz de ventas; contiene `SaleItem`; no puede confirmarse vacía, no repite productos y es inmutable (DM §§2.3–2.4).
- `User` es raíz de identidad; username normalizado, rol cerrado y hash protegido (DM §§1, 2.5, 7).
- `Category` es referencia sembrada, de solo lectura (DM §§2.1, 9).
- Una línea copia nombre/precio del producto vendido; subtotal y total son cálculos (DM §§1, 2.4).

### Venta (mecanismo inferido)

1. Autenticar al operador y verificar autorización (DM §§7, 9.2; endpoints no entregados).
2. Cargar productos y aplicar invariantes de cantidad y disponibilidad.
3. Agregar línea con valores congelados y descontar stock como una operación de dominio (DM §§2.2–2.4).
4. Persistir venta y existencias coordinadamente. **Supuesto ARQ-04:** una transacción PostgreSQL implementa esa atomicidad; DM exige la operación lógica, pero no describe el código transaccional.

### Imagen

Primero anular/cambiar `image_key` y confirmar la transacción; después borrar binario externo. Si falla el segundo paso queda archivo huérfano, pero no una referencia rota. La limpieza de huérfanos está sin definir (DM §§7.1, 11 H-2).

### Reporte

Consulta de rango, agrupada por producto y valores congelados, sin persistencia ni desglose por vendedor (DM §§1, 6, 7.1, 11.1, 12). Hay una contradicción externa: CA-06.1 dice “una fila por producto”, mientras DM §11.1 decide agrupar también por etiqueta de categoría congelada.

## Bloqueos de consistencia de la fuente

| Tema | Contradicción | Tratamiento |
|---|---|---|
| `deleted_at` | §13 registra T-09 saldada y declara la columna/filtro aplicados el 20-09; la consulta adjunta es del 19-09 (§§3, 10, 13) | Tomar §13 como estado documental posterior; regenerar la consulta para una verificación física actual |
| `sale_item.sale_id` | La tabla y consulta del 19-09 lo marcan nullable; §13 dice que T-20 lo dejó `NOT NULL` el 20-09 (§§3, 10, 13) | Para el ejercicio, usar el estado posterior de §13; documentar la salida de §10 como obsoleta |
| FK/índice T-20 | §13 dice que ya están aplicados; consultas del 19-09 reflejan estado anterior (§§5, 10, 13) | Aplicados según el último estado escrito; volver a medir para certificar el motor actual |
| `category_name` T-11 | La tabla física lo marca pendiente, pero invariantes/consulta de reporte lo tratan como presente (§§2.4, 3, 11.1) | Estado no resuelto en la única versión; no asumir columna aplicada |
| Columnas | §10 lista 21 columnas el 19-09; §13 declara después `product.deleted_at`, que eleva a 22 el conteo documentado (§§3, 10, 13) | El conteo posterior se infiere de §13; falta una consulta actualizada |
| Autorización | §9.2 aún describe alta anónima como rota; §13 registra A-1 cerrado con 401/403 (§§9.2, 13) | Tomar §13 como estado documental posterior y señalar §9.2 como texto sin actualizar |

## Supuestos

ARQ-01 arquitectura hexagonal recomendada; ARQ-02 capa de aplicación para casos de uso; ARQ-03 existe API; ARQ-04 transacción para venta/stock. Ninguno se presenta como implementación probada.

**Fuente:** `spec/data-model.md`, §§0–13. No se tuvieron los documentos externos que allí se enlazan.

## Cierre de coherencia tras completar los documentos

- **Contexto:** el alcance cubre catálogo, existencias, operadores, ventas y consultas; imágenes quedan fuera de PostgreSQL. No se amplía a clientes, pagos ni multimoneda.
- **Dominio:** las raíces `Product`, `Sale` y `User`, junto con `Category` de solo lectura y `SaleItem` interno, son las mismas entidades usadas por requisitos y flujos.
- **Producto:** objetivos y visión se limitan a capacidades que el modelo respalda; impacto comercial y métricas permanecen sin afirmar.
- **Requisitos:** criterios derivan de invariantes y consultas Q1–Q8. Los permisos no definidos se marcan como supuestos; A-1 se toma del estado posterior de §13.
- **Persistencia:** T-09 y T-20 se reflejan según §13; los SQL del 19-09 quedan como snapshots anteriores. T-11 y CA-06.1 siguen abiertos y no se resuelven por inferencia.
- **Duplicidades y lógica:** requisitos no duplican invariantes como funciones independientes; venta/stock se describe como una operación lógica, sin afirmar detalles de implementación; eventos son propuestas conceptuales.

La arquitectura hexagonal y sus capas siguen siendo una recomendación inferida, no una constatación del sistema desplegado. Para verificar la implementación se requieren migraciones, código y una consulta actualizada del motor.
