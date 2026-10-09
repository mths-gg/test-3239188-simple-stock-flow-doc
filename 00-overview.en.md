# 00 — System overview and reading guide

## Purpose

This document provides an entry point to the Simple Stock Flow work and explains how its deliverables relate to one another. It was prepared as a supplementary document at the user's request; it does not replace the `README.md` instructions or modify the source data model. The only technical source is `spec/data-model.md` (DM); the other pages reconstruct the documentation from that model.

## System summary

Simple Stock Flow documents a system for maintaining a product catalog and its stock, recording sales made by internal operators, and querying sales by period. A sale contains one or more line items; each line preserves the product name and price at the time of the transaction. The sale and stock deduction must be coordinated as one logical operation. Totals and reports are calculated when queried and are not stored as entities or report tables (DM §§1–2, 6–7).

The model includes the `admin` and `seller` roles, five seeded read-only categories, and external storage for image files. PostgreSQL stores the system data and an opaque reference to each image; it does not store image bytes (DM §§1–3, 9).

## Scope and boundaries

The documented scope includes the catalog, stock, operators, sales, sale items, external images, and derived queries/reports. It excludes customers, payments, multiple currencies, category management, physical product deletion, editing or deleting sales, reports by seller, and product attributes not declared by the model (DM §§1, 2, 7–8, 12).

This work reconstructs documentation; it does not prove that an application has been implemented or that the proposed architecture has been deployed. No code, migrations, complete API contract, or database access was provided.

## How the documents fit together

| Document | Question it answers | Main result |
|---|---|---|
| `01-context/context.md` | What system is being built, and where are its boundaries? | Purpose, scope, actors, dependencies, and context assumptions |
| `02-domain/domain.md` | What concepts and rules govern the business? | Glossary, entities, relationships, invariants, and proposed conceptual events |
| `03-product/product.md` | What problem does it address, and what value does it seek? | Provisional vision, supported objectives, and exclusions |
| `04-requirements/requirements.md` | What behaviors follow from the model? | Traceable user stories, acceptance criteria, and non-functional requirements |
| `05-architecture/architecture.md` | What components and flows could support those behaviors? | Inferred architecture, aggregates, flows, and coherence review |
| `auditoria.md` | What duplication, contradictions, and limitations were found? | Findings, applied treatment, and explicit open items |
| `spec/data-model.md` | What is the technical source for the reconstruction? | Reference data model and rules |

The README defines the production order as architecture → requirements → product → domain → context, followed by a final architecture coherence review. The table above provides a practical reading order, from context to solution.

## Interpretation and traceability rules

1. Technical claims should point to a section of the data model, for example `DM §2.3`.
2. Conclusions that do not follow directly from the source are labeled as assumptions or proposals; they are not presented as approved decisions or implemented features.
3. Domain rules are not confused with constraints guaranteed by the database. The model distinguishes database-enforced, domain-only, and pending validations (DM §§0, 4).
4. SQL queries dated 2026-09-19 predate the changes recorded in §13 on 2026-09-20. The later §13 status is used for documentary descriptions, but a new query is needed to verify the current database.
5. Hexagonal architecture is recommended as an interpretation compatible with the mentioned ports and adapters; it is not claimed to be the architecture already implemented.

## Open items that affect completion

- **T-11 (`category_name`):** one section leaves the column pending, while other sections use it. Its existence cannot be confirmed from the available source.
- **T-12 (`sold_by_user_id`):** the sale-to-user relationship remains pending in the documented status.
- **CA-06.1:** it requires one row per product, while DM §11.1 describes grouping by the frozen category label as well. This may produce multiple rows per product and requires an external decision.
- **Outdated evidence:** SQL queries should be regenerated to verify T-09, T-20, and the current column count; §13 describes changes made after the snapshot.
- **Unaligned text:** DM §9.2 retains an earlier description of anonymous registration, while §13 marks A-1 as closed with 401/403 responses.

These open items are not resolved by inference in this document; their details and other observations are recorded in `auditoria.md`.

## Status of this guide

This is an auxiliary synthesis derived from the existing documents. It adds no requirements, entities, or architecture decisions to the scope. If the technical source changes, this guide and the documents that depend on it should be reviewed together.
