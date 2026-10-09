# Contexto — Simple Stock Flow

## Propósito derivado

El sistema mantiene catálogo e inventario, registra ventas de operadores internos y permite consultar ventas por rango de fechas. El propósito se deriva de las entidades, reglas y patrones de acceso de `spec/data-model.md` (DM §§1–2, 6); el repositorio no contiene descripción directa del problema de negocio.

## Alcance

- Productos con nombre, precio, existencias, categoría e imagen opcional (DM §§1, 3).
- Cinco categorías semilla, de solo lectura (DM §§2.1, 9).
- Usuarios internos con roles `admin` / `seller` y hash de contraseña (DM §§1, 2.5).
- Ventas inmutables con una o más líneas y valores históricos de producto (DM §§1, 2.3–2.4, 7.1).
- Consulta por rango de fechas y agregados calculados al leer (DM §§1, 6).
- Binarios de imagen externos; PostgreSQL guarda clave opaca (DM §§1, 3, 7.1).

## Fuera del alcance

No hay clientes/compradores, pagos, moneda múltiple, CRUD de categorías, borrado físico de productos, edición/borrado de ventas, reportes por vendedor, tabla de reporte, historial `created_at`/`updated_at` ni campos de producto no declarados (DM §§1, 2.1, 7.1, 8, 12).

## Actores y dependencias

| Actor/sistema | Relación | Límite |
|---|---|---|
| Administrador | Rol `admin`; alta inicial desde configuración de arranque | §13 declara cerrado A-1 (401/403); §9.2 conserva descripción anterior sin actualizar (DM §§2.5, 9.2, 13) |
| Vendedor | Rol `seller`, operador que realiza ventas | Privilegios endpoint por endpoint no especificados (DM §§1, 2.5) |
| PostgreSQL | Base `simple_stock_flow`, esquema `sales`, se reporta versión 16.14 | Mediciones SQL del 19-09-2026; cambios posteriores registrados en DM §13 |
| Almacenamiento de archivos | Guarda imágenes por clave opaca | Tecnología, protocolo y limpieza no especificados (DM §§3, 7.1, 11 H-2) |
| API | Referida por `api-contract.md` externo | Contrato no viene en el repositorio del reto (DM §12) |

## Fronteras de datos y restricciones

PostgreSQL guarda catálogo, usuarios, ventas y líneas; imágenes quedan externas; totales e informes se calculan y no se guardan. Algunas invariantes se validan solo en dominio, no en el motor (DM §§0, 3–4, 7.1). No se afirma que la arquitectura sea de microservicios.

## Supuestos

- **CX-01:** `admin` y `seller` pertenecen al personal de un comercio; la fuente solo dice operadores internos (DM §1).
- **CX-02:** existe una API, inferida por la referencia a `api-contract.md`; no conocemos transporte ni endpoints (DM §12).
- **CX-03:** el producto busca mantener consistencia entre ventas e inventario; inferencia de las reglas, no hallazgo de entrevistas (DM §§1–2).

## Incertidumbres

El modelo incluye consultas del 19-09-2026 y un registro de cambios del 20-09. Se toma el registro posterior como estado documental para T-09, T-20 y A-1; las consultas no se actualizaron. T-11 (`category_name`) permanece contradictoria en la única versión. Ver `auditoria.md`.
