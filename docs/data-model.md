# Aegis Platform — Data Model

Phase 0 design document. The entities each module owns, their key
fields and the constraints that protect the business invariants in
[requirements.md](requirements.md). Field lists are design intent,
not migrations — Flyway scripts are written with each module.

Conventions: every table has a surrogate `id` (UUID) primary key,
`created_at` / `updated_at` timestamps, and foreign keys to the
owning module's tables only. Cross-module references are by ID
through the owning module's API — never a join across module
boundaries.

## Module A — Inventory

| Table | Key fields | Constraints |
| --- | --- | --- |
| `category` | id, name, parent_id | name unique per parent |
| `product` | id, sku, name, category_id, base_price, archived | **sku unique**; base_price ≥ 0 |
| `warehouse` | id, code, name, active | code unique |
| `stock_level` | product_id, warehouse_id, quantity_on_hand, quantity_reserved | unique (product_id, warehouse_id); **quantity_on_hand ≥ 0**; quantity_reserved ≥ 0; quantity_reserved ≤ quantity_on_hand |
| `stock_adjustment` | id, product_id, warehouse_id, delta, reason, actor_id, created_at | immutable once written |

Stock integrity lives here: the `stock_level` checks plus atomic
conditional updates (`UPDATE ... SET quantity_reserved =
quantity_reserved + :n WHERE quantity_on_hand - quantity_reserved >=
:n`) are what make invariant 3 hold under concurrency.

## Module B — Orders

| Table | Key fields | Constraints |
| --- | --- | --- |
| `customer_order` | id, customer_id, status, total_amount, idempotency_key, placed_at | **idempotency_key unique per customer**; total_amount ≥ 0 |
| `order_line` | id, order_id, product_id, quantity, unit_price | quantity > 0; unit_price captured at placement, never re-read from `product` |
| `payment_record` | id, order_id, method, amount, status | amount = order total at capture |
| `stock_reservation` | id, order_id, product_id, warehouse_id, quantity, released | one active reservation per order line |

Order status is a closed set: `DRAFT`, `PLACED`, `CONFIRMED`,
`FULFILLED`, `CANCELLED`. Transitions are validated in the service
layer and recorded in the audit trail.

## Module C — Identity

Identity data is owned by Keycloak (users, credentials, sessions).
Aegis stores only the mapping it needs for authorization:

| Table | Key fields | Constraints |
| --- | --- | --- |
| `app_user` | id, keycloak_subject, display_name, active | keycloak_subject unique |
| `role_assignment` | user_id, role, granted_by, granted_at | role ∈ {CUSTOMER, SUPPORT_AGENT, WAREHOUSE_OPERATOR, ADMINISTRATOR}; a user cannot be their own `granted_by` for an elevation |

## Module D — Audit

| Table | Key fields | Constraints |
| --- | --- | --- |
| `audit_event` | id, actor_id, action, target_type, target_id, outcome, detail (JSONB), created_at | **append-only** — the application DB role has INSERT and SELECT only; no UPDATE or DELETE grants |

`detail` is a redacted summary (IDs, amounts, outcomes) — never
request bodies, tokens or personal data beyond what the
investigation needs.

## Module E — Platform

No domain tables. Owns configuration, error types and observability
plumbing only.
