# ADR-003 — One database, one schema per module

**Status:** Accepted (Phase 0 design)
**Date:** 2026-11-21

## Context

Aegis is a modular monolith (ADR-001): one deployable, one
PostgreSQL database. The modules (Inventory, Orders, Identity,
Audit, Platform) must stay independent enough that a module can be
extracted later without untangling shared tables — but Phase 0–1
has no budget for distributed transactions or a message broker
(see non-goals in `docs/requirements.md`).

## Decision

- One PostgreSQL database, **one schema per module**:
  `inventory`, `orders`, `identity`, `audit`.
- A module may read and write **only its own schema**. Cross-schema
  foreign keys are forbidden.
- Cross-module references are stored as plain identifiers
  (e.g. `orders.order_line.product_id` holds the Inventory product
  id) with **no database-level FK** into another module's schema.
  Integrity across modules is enforced by application logic and
  covered by tests, not by constraints.
- Inside a module, foreign keys and constraints are used
  aggressively — that is where integrity is cheap and total.
- Each module owns its Flyway migrations, in its own location and
  version sequence, so migration history reads per module.
- Cross-module calls happen only through the owning module's
  public API (in-process interfaces in the monolith), never through
  direct table access.

## Consequences

- **Positive:** module boundaries are enforced by the database,
  not by convention alone. A module extraction later becomes a
  schema export plus an API swap, not a data surgery. Invariant 4
  (atomicity) stays achievable with ordinary local transactions
  inside a module, and with the outbox (ADR-004) across modules.
- **Negative:** cross-module joins are impossible; reporting that
  spans modules must compose data in application code. Accepted:
  Phase 0–1 has no cross-module reporting requirement.
- **Risk controlled:** a developer "just joining" across schemas
  is blocked by database grants — each module's datasource user
  has rights only on its own schema.
