# Module A — Inventory

**Scope:** products, categories, warehouses, stock levels, stock
adjustments.

**Owns:** product catalogue data, stock quantities per warehouse,
the stock adjustment history (reason + actor on every change).

**Public API (planned):** product search (paginated, filtered,
sorted), stock availability query, stock adjustment operation —
the only door through which Orders may touch stock.

**Key rules:**

- SKUs are unique, enforced by a database constraint.
- Stock adjustments always record who did them and why.
- Products are archived, never hard-deleted, once they appear on an
  order.

**Status:** bones only — no code yet.
