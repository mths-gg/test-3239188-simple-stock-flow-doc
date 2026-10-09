# Problema y visión — Simple Stock Flow

## Problema documentable

El modelo describe un sistema que debe mantener productos y existencias, registrar ventas con sus líneas y permitir consultas por periodo. Las líneas conservan nombre y precio de producto al momento de venta, de modo que el histórico no dependa del catálogo actual (DM §§1–2, 6).

**Límite:** el insumo no explica el proceso actual del negocio, sector, usuarios organizacionales, impactos ni métricas. No se inventan pérdidas o demoras como si fueran hallazgos.

## Visión provisional

> Para operadores internos que administran un catálogo y registran ventas, Simple Stock Flow conserva productos y existencias, registra ventas con sus valores históricos y permite consultar operaciones por fecha, manteniendo las reglas de consistencia del dominio.

Es una visión derivada para el ejercicio, no una declaración aprobada por propietario (**supuesto PROD-01**). Se sustenta en DM §§1–2, 6.

## Objetivos respaldados

1. Mantener productos con nombre, precio, stock, categoría e imagen opcional (DM §§1, 2.2).
2. Impedir stock negativo y retiro superior a disponibilidad (DM §2.2).
3. Conservar ventas inmutables con líneas y datos del momento de venta (DM §§1, 2.3–2.4, 7.1).
4. Consultar ventas por rango y calcular agregados sin guardarlos (DM §§1, 6).
5. Proteger identidad y secretos de operadores internos (DM §§1, 2.5, 7).

## Fuera de alcance según el modelo

Clientes/compradores, pagos, multimoneda, gestión de categorías, borrado físico de productos, edición/borrado de ventas, reportes por vendedor, auditoría `created_at`/`updated_at` y campos extra como descripción/SKU (DM §§1, 2.1, 7.1, 8, 12).

## Usuarios y valor

Roles identificados: `admin` y `seller`; el modelo los describe como operadores internos (DM §§1, 2.5). **Supuesto PROD-02:** trabajan en un negocio que vende productos. El valor esperado es coherencia de inventario y consulta histórica; no se dispone de métricas de impacto.

## Decisiones por resolver

- Reporte por categoría congelada puede producir filas distintas tras renombrar categoría; CA-06.1 dice una fila por producto (DM §11.1). Resolver con producto/propietario.
- La autorización A-1 se declara cerrada en DM §13 (sin token → 401; vendedor → 403), mientras §9.2 conserva la descripción anterior de alta anónima; se usa §13 como estado posterior.
- T-09 y T-20 se declaran aplicadas el 20-09 en §13; las consultas del 19-09 son evidencia anterior. T-11 (`category_name`) sí queda contradictoria en la única versión; no asumir que está aplicada (DM §§3, 10, 13).
