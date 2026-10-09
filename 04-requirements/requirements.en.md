# Requirements — Simple Stock Flow

References such as `DM §n` point to `spec/data-model.md`, the only technical input provided. Statements not supported by the model are labeled assumptions or open items.

## Actors

- **Administrator:** `admin` role; the initial administrator is provisioned at application startup (DM §§1, 2.5, 9.2). Exact privileges by operation are unspecified.
- **Seller:** `seller` role, an internal operator who records sales (DM §§1, 2.5).
- **Image storage:** external system that stores binary files under opaque keys (DM §§3, 7.1). Inferred technical actor.

## User stories

### US-01 Browse the catalog

As an application user, I want to search active products by text or category so that I can find catalog items.

- Partial text search, category filter, name ordering, and pagination (DM §6 Q1).
- Retrieve a product by ID and load active products in batches of IDs (DM §6 Q2–Q3).
- **Assumption:** the model does not define which role may browse or the response for a missing ID.

### US-02 Maintain products

As an authorized user, I want to maintain products so that the catalog represents the items available for sale.

- Product fields: name, price, stock, required category, and optional image; do not add description, SKU, or other fields (DM §§1, 3, 12).
- Name is trimmed and non-empty; price is strictly positive; stock is never negative; withdrawals cannot exceed available stock (DM §2.2).
- Category must exist. A missing image is `NULL`, not an empty string (DM §2.2).
- Categories: five seeded, read-only values; there is no category CRUD (DM §§2.1, 9).
- A discontinued product is not physically deleted; T-09 is marked applied as of Sep 20 (DM §§2.2, 7.1, 13). The attached SQL result is from the previous day.
- **Assumption:** the source does not define which role is authorized to manage the catalog.

### US-03 Record a sale

As an authenticated seller, I want to record a sale so that the system stores what was sold and updates stock.

- A sale stores its operator and timestamp. Every confirmed sale has at least one line (DM §§1, 2.3).
- Each line has a product, positive quantity, and frozen product name/price; subtotal is calculated (DM §§1, 2.4).
- A product cannot appear more than once in a sale; an empty sale cannot be confirmed (DM §2.3).
- Adding a line and decreasing stock form one operation; reject the sale if stock is insufficient (DM §§2.2–2.3).
- Total is calculated, not stored. The system uses one currency (DM §§1, 3).
- A recorded sale is immutable and cannot be deleted (DM §§1, 2.3, 7.1).
- **Assumption:** the concrete transaction mechanism for saving the sale and stock must be verified in code.

### US-04 View sales and reports

As an authorized user, I want to query sales by date range so that I can review sales activity.

- Search by date range, sort by descending date, and provide paginated/non-paginated variants (DM §6 Q7–Q8).
- End date cannot precede start date (DM §1).
- Report is calculated at query time, not persisted, and has no seller breakdown (DM §§1, 7.1, 12).
- Use frozen historical values. The business decision is to group by frozen label (DM §11.1), but storing `category_name` depends on T-11, whose status conflicts within the only version (§§2.4, 3).
- **Open conflict:** external acceptance criterion CA-06.1 requires “one row per product,” which conflicts with grouping by frozen label when a category changes. Keep the §11.1 decision and document CA-06.1 as unresolved; do not invent a resolution.

### US-05 Manage users and access

As an administrator, I want to manage internal users so that I can control who operates the system.

- Username is unique, trimmed, and lowercase; role is either `admin` or `seller` (DM §§1, 2.5).
- The domain never receives a plaintext password; retain the hash produced by a port; do not expose the hash in logs, responses, or errors (DM §§1, 7, 9.2).
- Initial administrator is provisioned at startup from environment credentials; do not insert a fixed hash through SQL (DM §9.2).
- No one assigns `admin` during normal use; deployment provisions that role (DM §11 H-3).
- The latest model status says A-1 is closed: no token → 401; `seller` attempting an administrative operation → 403 (DM §13). §9.2 retains an earlier description and should be updated.

### US-06 Manage images

As an authorized user, I want to associate an image with a product so that it can be displayed in the catalog.

- The database stores an opaque `image_key`; bytes and file path are stored externally (DM §§1, 3).
- On replacement or deletion: clear the reference and commit the database change before deleting the binary (DM §7.1).
- **Operational open item:** cleanup of orphaned files (DM §11 H-2). Formats, sizes, and permissions are **unspecified assumptions**.

## Non-functional requirements

| ID | Derived requirement | Evidence and limit |
|---|---|---|
| NFR-01 | Database engine must prevent negative stock, including direct SQL | `CHECK` is reported (DM §§2.2, 4); related query is a Sep 19 snapshot |
| NFR-02 | Validate domain-only rules in every use case | DM §§0, 2, 4; direct SQL can bypass them |
| NFR-03 | Stock concurrency follows the optimistic strategy mentioned in ADR-002 | DM §§0, 2.2, 3; detailed mechanism is not included |
| NFR-04 | Do not leak credentials or password hashes | DM §§2.5, 7 |
| NFR-05 | Keep images external; prioritize a consistent reference even if cleanup fails | DM §§3, 7.1, 11 H-2 |
| NFR-06 | Retain sales indefinitely; use logical product deletion | DM §§2, 7.1, 13; T-09 is marked applied in the Sep 20 update |
| NFR-07 | Timestamps include a time zone; server is reported as UTC | DM §3; evidence dated Sep 19, 2026 |
| NFR-08 | Restrict personal data in username/operator and role; hash never leaves the system | DM §7 |
| NFR-09 | Indexes must be justified by access patterns Q1–Q8 | DM §§6, 13; Sep 20 T-20 status supersedes the Sep 19 physical list |

## Exclusions to prevent scope expansion

No end customers, payments/cards, multiple currencies, sale edits/deletions, category CRUD, physical product deletion, extra product fields, seller reports, or `created_at`/`updated_at` (DM §§1, 2.1, 7.1, 8, 12).

## Traceability items to resolve

Before finalizing acceptance criteria: update SQL snapshots after T-09 and T-20; resolve T-11 `category_name`; resolve T-12 `sold_by_user_id`; reconcile CA-06.1 with grouping by frozen label. Details are in `auditoria.md`.
