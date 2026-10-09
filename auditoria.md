# Auditoría — actividad Simple Stock Flow

## Resultado

Se prepararon los cinco documentos pedidos en las carpetas del README: contexto, dominio, producto, requisitos y arquitectura. Las afirmaciones se trazan a `spec/data-model.md`; las inferencias están marcadas como supuestos. La documentación adopta la declaración más reciente del §13 (20-09-2026) por encima de las consultas fechadas el 19-09-2026.

**Estado de entrega: completada como documentación derivada y auditada.** No se certifica el motor actual porque no se entregaron base, migraciones ni código. Las consultas antiguas deben actualizarse antes de usarlas como verificación ejecutable.

## Hallazgos de la fuente y resolución aplicada

| ID | Hallazgo | Evidencia | Resolución en los documentos |
|---|---|---|---|
| AUD-01 | Consultas SQL del 19-09 y registro de cambios del 20-09 tienen fechas distintas | DM §§10, 13 | Se toma §13 como estado documental posterior para T-09, T-20 y A-1; se anotan las consultas como snapshot anterior |
| AUD-02 | El snapshot lista 21 columnas y no contiene `deleted_at`; §13 dice que `product.deleted_at` existe desde T-09 | DM §§3, 10.1, 13 | El estado posterior implica 22 columnas documentadas; queda pendiente regenerar la consulta, no se trata como cambio desconocido |
| AUD-03 | `sale_item.sale_id` sale nullable el 19-09; §13 indica `NOT NULL` después de T-20 el 20-09 | DM §§3, 10.1, 13 | Adoptado `NOT NULL` según el último estado; query antigua marcada como obsoleta |
| AUD-04 | T-11 / `category_name` está pendiente en el modelo físico, pero otras secciones lo tratan como existente | DM §§2.4, 3, 11.1 | No se inventó estado: queda como contradicción no resuelta en la única versión y como supuesto no confirmado para el reporte |
| AUD-05 | FK a producto e índice único de línea no aparecen en el snapshot del 19-09; §13 declara ambos aplicados tras T-20 | DM §§5, 10.2–10.3, 13 | Se adoptan como aplicados según §13; se exige nueva medición para certificación técnica |
| AUD-06 | §9.2 conserva que alta anónima está rota, pero §13 registra A-1 cerrado (sin token → 401; `seller` → 403) | DM §§9.2, 13 | Se usa §13 como estado posterior y se marca §9.2 como texto pendiente de actualización |
| AUD-07 | La aceptación CA-06.1 indica una fila por producto; §11.1 decide agrupar por etiqueta congelada, lo que puede producir varias filas por producto | DM §11.1 | Se conserva la decisión explícita de §11.1 y se deja CA-06.1 pendiente de reconciliación; el usuario no tiene instrucción adicional |
| AUD-08 | El modelo remite a `spec.md`, `plan.md`, ADR, contrato API y código que no se entregan en este repositorio | README y DM §§0, 12 | No se atribuyeron hechos a documentos ausentes; arquitectura se presenta como inferida |

## Auditoría de duplicidad y lógica en los entregables

- Hay un documento por carpeta solicitada; ninguna historia de usuario duplica otra con un ID diferente.
- Las historias expresan objetivos y criterios; las invariantes aparecen en dominio sin convertirse en funcionalidades separadas.
- Eventos de dominio están etiquetados como propuestas conceptuales; no se afirma que exista mensajería o event sourcing.
- La atomicidad lógica de venta/stock se distingue del mecanismo transaccional concreto, que no está descrito.
- No se añadieron microservicios, compradores, pagos, multimoneda, CRUD de categorías, borrado de ventas, campos extra ni métricas de negocio.
- Se respeta el estado más reciente del §13 en lugar de tratar snapshots antiguos como estado presente.
- El caso de `category_name` permanece señalado porque no existe una versión posterior ni una decisión que permita resolverlo.

## Trazabilidad

| Documento | Cobertura | Resultado de auditoría |
|---|---|---|
| `01-context/context.md` | Propósito, alcance, actores, fronteras y supuestos | Sin alcance inventado; referencias a DM |
| `02-domain/domain.md` | Glosario, entidades, relaciones, reglas y eventos propuestos | Distingue persistencia del dominio y estados posteriores |
| `03-product/product.md` | Problema, visión, objetivos y exclusiones | No presenta impactos no medidos como hechos |
| `04-requirements/requirements.md` | Historias, criterios y requisitos no funcionales | Conserva CA-06.1 como conflicto abierto |
| `05-architecture/architecture.md` | Componentes, agregados, flujos y supuestos | Arquitectura hexagonal identificada como inferencia |

## Pendientes documentales explícitos

1. Actualizar las consultas de §10 para reflejar T-09 y T-20 después del 20-09-2026.
2. Resolver T-11 / `category_name` en la única versión: confirmar si la migración se aplicó o sigue pendiente.
3. Resolver CA-06.1 frente al reporte por etiqueta congelada. Hasta entonces, explicar ambas reglas, no presentar aceptación final.
4. Alinear §9.2 con el cierre de A-1 indicado en §13.
5. Si se quiere certificar el estado ejecutado y no solo reconstruir documentación, contrastar el modelo con la base y el código actuales.

## Límites de esta auditoría

Se revisaron el README y `spec/data-model.md` del repositorio. No se accedió al motor PostgreSQL, migraciones, API ni código. La auditoría valida consistencia lógica y trazabilidad documental; no certifica una instalación en ejecución.
