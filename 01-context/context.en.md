# Context — Simple Stock Flow

## Derived purpose

The system maintains a catalog and inventory, records sales made by internal operators, and allows sales to be queried by date range. This purpose is derived from the entities, rules, and access patterns in `spec/data-model.md` (DM §§1–2, 6); the repository does not directly describe the business problem.

## Scope

- Products with a name, price, stock, category, and optional image (DM §§1, 3).
- Five seeded, read-only categories (DM §§2.1, 9).
- Internal users with `admin` / `seller` roles and password hashes (DM §§1, 2.5).
- Immutable sales with one or more line items and historical product values (DM §§1, 2.3–2.4, 7.1).
- Date-range queries and aggregates calculated when read (DM §§1, 6).
- External image binaries; PostgreSQL stores an opaque key (DM §§1, 3, 7.1).

## Out of scope

There are no customers/buyers, payments, multiple currencies, category CRUD, physical product deletion, editing/deleting sales, reports by seller, `created_at`/`updated_at` history, or undeclared extra product fields (DM §§1, 2.1, 7.1, 8, 12).

## Actors and dependencies

| Actor/system | Relationship | Boundary |
|---|---|---|
| Administrator | `admin` role; initial account provisioned at application startup | §13 marks A-1 closed (401/403); §9.2 retains an earlier description that has not been updated (DM §§2.5, 9.2, 13) |
| Seller | `seller` role, an operator who records sales | Endpoint-by-endpoint privileges are not specified (DM §§1, 2.5) |
| PostgreSQL | `simple_stock_flow` database, `sales` schema; version 16.14 is reported | SQL measurements dated 2026-09-19; later changes are recorded in DM §13 |
| File storage | Stores images by opaque key | Technology, protocol, and cleanup are unspecified (DM §§3, 7.1, 11 H-2) |
| API | Referred to by an external `api-contract.md` | The contract is not included in the activity repository (DM §12) |

## Data boundaries and constraints

PostgreSQL stores the catalog, users, sales, and sale items; images remain external; totals and reports are calculated and not stored. Some invariants are validated only in the domain, not by the database engine (DM §§0, 3–4, 7.1). No microservices architecture is asserted.

## Assumptions

- **CX-01:** `admin` and `seller` belong to a business's staff; the source only says they are internal operators.
- **CX-02:** an API exists, inferred from the reference to `api-contract.md`; its transport and endpoints are unknown.
- **CX-03:** the product aims to keep sales and inventory consistent; this is inferred from the rules, not a finding from interviews.

## Uncertainties

The model includes queries dated 2026-09-19 and a change log dated 2026-09-20. For T-09, T-20, and A-1, the later §13 status is used; the queries were not updated. T-11 (`category_name`) remains contradictory in the only available version. See `auditoria.md`.
