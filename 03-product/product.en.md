# Problem and Product Vision — Simple Stock Flow

## Documentable problem

The model describes a system that must maintain products and stock, record sales with their lines, and support date-range queries. Sale lines retain the product name and price at the time of sale, so historical records do not depend on the current catalog (DM §§1–2, 6).

**Limit:** the input does not explain the business's current process, industry, organizational users, impact, or metrics. No losses or delays are presented as established findings.

## Provisional vision

> For internal operators who maintain a catalog and record sales, Simple Stock Flow keeps products and stock, records sales with historical values, and supports date-based queries while enforcing domain consistency rules.

This is a vision derived for the exercise, not a statement approved by a product owner (**assumption PROD-01**). It is based on DM §§1–2, 6.

## Supported objectives

1. Maintain products with a name, price, stock, category, and optional image (DM §§1, 2.2).
2. Prevent negative stock and withdrawals beyond available stock (DM §2.2).
3. Preserve immutable sales with lines and product values from the time of sale (DM §§1, 2.3–2.4, 7.1).
4. Query sales by date range and calculate aggregates without storing them (DM §§1, 6).
5. Protect internal operators' identities and secrets (DM §§1, 2.5, 7).

## Out of scope according to the model

Customers/buyers, payments, multiple currencies, category management, physical product deletion, sale editing/deletion, seller reports, `created_at`/`updated_at` auditing, and extra fields such as description/SKU (DM §§1, 2.1, 7.1, 8, 12).

## Users and value

Identified roles: `admin` and `seller`; the model describes them as internal operators (DM §§1, 2.5). **Assumption PROD-02:** they work in a business that sells products. The expected value is consistent inventory and historical queries; no impact metrics are available.

## Decisions to resolve

- A report grouped by frozen category can produce multiple rows for a product after a category rename; CA-06.1 says one row per product (DM §11.1). Resolve with the product owner.
- Authorization A-1 is marked closed in DM §13 (no token → 401; seller → 403), while §9.2 retains the earlier description of anonymous creation; use §13 as the later status.
- T-09 and T-20 are marked applied on Sep 20 in §13; the queries dated Sep 19 are earlier evidence. T-11 (`category_name`) remains contradictory in the only version; do not assume it was applied (DM §§3, 10, 13).
