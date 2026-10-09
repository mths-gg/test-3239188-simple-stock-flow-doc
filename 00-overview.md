# 00 — Guía de lectura y síntesis del sistema

## Propósito

Este documento ofrece una entrada breve al trabajo de Simple Stock Flow y explica cómo se relacionan sus entregables. Se redactó como complemento a petición del usuario; no sustituye las instrucciones del `README.md` ni modifica el modelo fuente. La única fuente técnica es `spec/data-model.md` (DM); las demás páginas reconstruyen la documentación a partir de ese modelo.

## Síntesis del sistema

Simple Stock Flow documenta un sistema para mantener un catálogo de productos y sus existencias, registrar ventas hechas por operadores internos y consultar ventas por periodo. Una venta contiene una o más líneas; cada línea conserva el nombre y el precio del producto al momento de la operación. La venta y el descuento de existencias deben coordinarse como una operación lógica. Los totales e informes se calculan al consultar y no se almacenan como entidades o tablas de reporte (DM §§1–2, 6–7).

El modelo contempla los roles `admin` y `seller`, cinco categorías sembradas de solo lectura y almacenamiento externo para los archivos de imagen. PostgreSQL conserva los datos del sistema y una referencia opaca a la imagen; no almacena sus bytes (DM §§1–3, 9).

## Alcance y límites

El alcance documentado comprende catálogo, existencias, operadores, ventas, líneas de venta, imágenes externas y consultas/reportes derivados. No incluye clientes, pagos, multimoneda, mantenimiento de categorías, borrado físico de productos, edición o eliminación de ventas, informes por vendedor ni atributos de producto que el modelo no declare (DM §§1, 2, 7–8, 12).

Este trabajo reconstruye documentación; no demuestra que exista una aplicación implementada ni que la arquitectura propuesta esté desplegada. No se proporcionaron código, migraciones, contrato API completo ni acceso a la base de datos.

## Cómo se conectan los documentos

| Documento | Pregunta que responde | Resultado principal |
|---|---|---|
| `01-context/context.md` | ¿Qué sistema se construye y dónde están sus límites? | Propósito, alcance, actores, dependencias y supuestos de contexto |
| `02-domain/domain.md` | ¿Qué conceptos y reglas rigen el negocio? | Glosario, entidades, relaciones, invariantes y eventos conceptuales propuestos |
| `03-product/product.md` | ¿Qué problema aborda y qué valor busca? | Visión provisional, objetivos respaldados y exclusiones |
| `04-requirements/requirements.md` | ¿Qué comportamientos se desprenden del modelo? | Historias, criterios y requisitos no funcionales trazables |
| `05-architecture/architecture.md` | ¿Qué componentes y flujos podrían sostener esos comportamientos? | Arquitectura inferida, agregados, flujos y revisión de coherencia |
| `auditoria.md` | ¿Qué duplicidades, contradicciones y límites se encontraron? | Hallazgos, tratamiento aplicado y pendientes explícitos |
| `spec/data-model.md` | ¿Cuál es la fuente técnica de la reconstrucción? | Modelo de datos y reglas de referencia |

El README define el orden de elaboración como arquitectura → requisitos → producto → dominio → contexto y, al final, una revisión de coherencia de arquitectura. La tabla anterior ofrece un orden práctico para leer el resultado desde el contexto hacia la solución.

## Reglas de interpretación y trazabilidad

1. Las afirmaciones técnicas deben apuntar a una sección del modelo de datos, por ejemplo `DM §2.3`.
2. Las conclusiones que no se desprenden directamente de la fuente se identifican como supuestos o propuestas; no se presentan como decisiones aprobadas ni como funciones implementadas.
3. Las reglas de dominio no se confunden con restricciones garantizadas por la base de datos. El modelo distingue validaciones del motor, del dominio y pendientes (DM §§0, 4).
4. Las consultas SQL fechadas el 19-09-2026 son anteriores a los cambios que el §13 registra el 20-09-2026. Para la descripción documental se usa el estado posterior de §13, pero hace falta una consulta nueva para verificar la base actual.
5. La arquitectura hexagonal se recomienda como interpretación compatible con los puertos y adaptadores mencionados; no se afirma que sea la arquitectura ya implementada.

## Pendientes que condicionan el cierre

- **T-11 (`category_name`):** el modelo deja la columna pendiente en una sección, aunque otras secciones la usan. No se puede confirmar su existencia con la fuente disponible.
- **T-12 (`sold_by_user_id`):** la relación de la venta con el usuario sigue pendiente en el estado documentado.
- **CA-06.1:** exige una fila por producto, mientras DM §11.1 describe agrupación también por etiqueta de categoría congelada. La diferencia puede producir varias filas por producto y requiere una decisión externa.
- **Evidencia desactualizada:** deben regenerarse las consultas SQL para verificar T-09, T-20 y el número actual de columnas; §13 describe cambios posteriores al snapshot.
- **Texto sin alinear:** DM §9.2 conserva una descripción anterior de alta anónima, mientras §13 declara A-1 cerrada con respuestas 401/403.

Los pendientes anteriores no se resuelven por inferencia en este documento; su detalle y las demás observaciones están en `auditoria.md`.

## Estado de esta guía

Es una síntesis auxiliar derivada de los documentos existentes. No añade requisitos, entidades ni decisiones de arquitectura al alcance. Si la fuente técnica cambia, esta guía y los documentos que dependan de ella deben revisarse juntos.
