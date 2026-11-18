# Module B — Orders

**Scope:** order placement, order lifecycle, payments, order history.

**Owns:** orders, order lines, payment records, order status
transitions.

**Depends on:** Inventory's public API for availability and stock
reservation — never its tables.

**Key rules:**

- Placement is one transaction: validate, reserve stock, persist —
  or nothing at all.
- Simultaneous orders for the same product cannot drive stock
  negative; the database enforces it, the module handles the loser
  cleanly.
- A customer reads and acts on their own orders only (enforced here,
  server-side, on every request).
- Retried submissions must not create duplicate orders
  (idempotency on placement).

**Status:** bones only — no code yet.
