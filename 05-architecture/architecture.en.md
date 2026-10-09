# Architecture — Simple Stock Flow

## Scope and evidence

This document derives the architecture solely from `spec/data-model.md` (DM). The activity repository provides that model and the README, but no code, API, migrations, or ADRs. Inferred decisions are labeled as assumptions.

## Architectural style

The model mentions aggregates and C# value objects, ports for hashing, repositories and read access, plus adapters for PostgreSQL/EF Core and external image storage (DM §§1–2, 6–7, 9, 12). This is compatible with hexagonal architecture. **Assumption ARQ-01:** organize the system this way; the model does not prove that this architecture is implemented or that it is the only option. There is no evidence of microservices.

## Inferred components

| Component | Responsibility | Evidence / limit |
|---|---|---|
| Domain | `Product`, `Sale`, `SaleItem`, `User`, `Category`; invariants and `Money`, `Quantity` value objects | DM §§1–2 |
| Application | Coordinate use cases, ports, authorization, and the sale/stock operation | DM §§2.2–2.3, 7, 9.2. **Assumption ARQ-02:** this layer is not described through classes |
| Persistence | Map aggregates to the `sales` schema with EF Core and migrations | DM §§0, 3, 5, 9, 12 |
| API | Expose operations and enforce authentication/authorization | DM §§7, 9.2, 12 mention an external contract that was not provided. **Assumption ARQ-03:** an API transport exists |
| Image storage | Store external binary files; the database keeps an opaque `image_key` | DM §§1, 3, 7.1 |
| Queries/reports | Query sales by date range and aggregate data at read time; no report is persisted | DM §§1, 6, 11.1 |

## Aggregates and flows

- `Product` is the catalog aggregate root; it validates price, stock, category, and image (DM §2.2).
- `Sale` is the sales aggregate root; it contains `SaleItem`; it cannot be confirmed empty, cannot repeat products, and is immutable (DM §§2.3–2.4).
- `User` is the identity aggregate root; username is normalized, role is constrained, and password hash is protected (DM §§1, 2.5, 7).
- `Category` is a seeded, read-only reference (DM §§2.1, 9).
- A sale line copies the product name and price at the time of sale; subtotal and total are calculated (DM §§1, 2.4).

### Sale (inferred mechanism)

1. Authenticate the operator and check authorization (DM §§7, 9.2; endpoints were not provided).
2. Load products and enforce quantity and availability invariants.
3. Add a line with frozen values and decrease stock as one domain operation (DM §§2.2–2.4).
4. Persist the sale and stock changes together. **Assumption ARQ-04:** a PostgreSQL transaction implements that atomicity; the DM requires the logical operation but does not describe the transactional code.

### Images

First clear or change `image_key` and commit the database transaction; then delete the external binary. If the second step fails, an orphaned file remains, but the database does not point to a missing file. Orphan cleanup is undefined (DM §§7.1, 11 H-2).

### Report

Query a date range, grouping by product and frozen values, without persistence or a breakdown by seller (DM §§1, 6, 7.1, 11.1, 12). There is an external conflict: CA-06.1 says “one row per product,” while DM §11.1 decides to group by the frozen category label as well.

## Source consistency issues

| Topic | Conflict | Treatment |
|---|---|---|
| `deleted_at` | §13 records T-09 as completed and says the column/filter were applied on Sep 20; the attached query is dated Sep 19 (DM §§3, 10, 13) | Use §13 as the later documented state; regenerate the query to verify the current physical database |
| `sale_item.sale_id` | The Sep 19 table/query marks it nullable; §13 says T-20 made it `NOT NULL` on Sep 20 (DM §§3, 10, 13) | Use the later §13 state for this exercise; mark §10's output as stale |
| T-20 FK/index | §13 says they are applied; Sep 19 queries show the earlier state (DM §§5, 10, 13) | Treat them as applied according to the latest written status; measure again to certify the current database |
| T-11 `category_name` | The physical table marks it pending, but invariants/report query treat it as present (DM §§2.4, 3, 11.1) | Unresolved in the only version; do not assume the column was applied |
| Column count | §10 lists 21 columns on Sep 19; §13 later declares `product.deleted_at`, implying 22 documented columns (DM §§3, 10, 13) | The later count is inferred from §13; an updated query is missing |
| Authorization | §9.2 still describes anonymous creation as broken; §13 records A-1 as closed with 401/403 (DM §§9.2, 13) | Use §13 as the later documented state and flag §9.2 as outdated text |

## Assumptions

ARQ-01 recommends hexagonal architecture; ARQ-02 assumes an application layer for use cases; ARQ-03 assumes an API exists; ARQ-04 assumes a transaction for sale/stock. None is presented as a verified implementation.

**Source:** `spec/data-model.md`, §§0–13. The external documents referenced there were not available.

## Consistency review after completing the other documents

- **Context:** scope covers catalog, stock, operators, sales, and queries; images remain outside PostgreSQL. Customers, payments, and multiple currencies were not added.
- **Domain:** aggregate roots `Product`, `Sale`, and `User`, read-only `Category`, and internal `SaleItem` are the same entities used by the requirements and flows.
- **Product:** goals and vision are limited to capabilities supported by the model; business impact and metrics are not asserted.
- **Requirements:** criteria derive from invariants and queries Q1–Q8. Unspecified permissions are labeled assumptions; A-1 follows the later status in §13.
- **Persistence:** T-09 and T-20 follow §13; Sep 19 SQL outputs are treated as earlier snapshots. T-11 and CA-06.1 remain open and are not resolved by inference.
- **Duplication and logic:** requirements do not recast invariants as separate features; sale/stock is described as one logical operation without claiming implementation details; domain events are conceptual proposals.

Hexagonal architecture and its layers remain an inferred recommendation, not a confirmed description of the deployed system. Verifying implementation requires migrations, source code, and an updated database query.
