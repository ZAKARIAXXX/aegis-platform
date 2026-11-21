# Aegis Platform — Order Lifecycle

Phase 0 design document. The state machine behind Module B,
expanded from `docs/requirements.md`. Every transition lists
its guard (what must be true), its effects (what changes), and
its audit record. A transition not listed here does not exist.

## States

`DRAFT → PLACED → CONFIRMED → FULFILLED`
`CANCELLED` is reachable from `DRAFT`, `PLACED`, `CONFIRMED`.
`FULFILLED` and `CANCELLED` are terminal.

## Transitions

### Create → `DRAFT`
- **Guard:** authenticated customer (or support on behalf,
  audited per the authorization matrix).
- **Effects:** order and lines persisted; **no stock is
  reserved** — a draft holds nothing.
- **Prices:** line prices are snapshots of the catalogue price
  at the moment the line is added; totals are always computed
  server-side.

### `DRAFT → PLACED`
- **Guard:** at least one line; idempotency key supplied;
  server re-validates every price at placement time —
  client-supplied prices are never trusted.
- **Effects (one transaction, invariant 4):** stock reserved
  atomically through Inventory's public API; order, lines,
  reservation and the `OrderPlaced` outbox event commit
  together or not at all.
- **Failure:** insufficient stock → `INSUFFICIENT_STOCK`, the
  order stays `DRAFT`, no partial reservation survives.

### `PLACED → CONFIRMED`
- **Guard:** role `SUPPORT_AGENT` or `ADMINISTRATOR`.
- **Effects:** reservation stands; `OrderConfirmed` event.

### `CONFIRMED → FULFILLED`
- **Guard:** role `WAREHOUSE_OPERATOR` or `ADMINISTRATOR`.
- **Effects:** reserved quantities become fulfilled; physical
  stock decreases by the fulfilled amounts. Invariant 3 is
  checked at this boundary as well as at reservation.

### `DRAFT | PLACED | CONFIRMED → CANCELLED`
- **Guard:** the owning customer (from `PLACED` only while not
  yet `CONFIRMED` — a confirmed order is cancelled by support
  or an administrator), or `SUPPORT_AGENT` / `ADMINISTRATOR`.
- **Effects:** any reservation is released in the same
  transaction; `OrderCancelled` event carries the reason.
- **Terminal:** a cancelled order can never transition again;
  re-ordering means a new order.

## Cross-cutting rules

- **Idempotent placement (invariant 5):** repeating
  `DRAFT → PLACED` with the same idempotency key and the same
  payload returns the original result; same key, different
  payload → `IDEMPOTENCY_KEY_REUSED`.
- **Illegal transitions** are rejected with
  `INVALID_STATE_TRANSITION` (error catalogue), including
  transitions attempted concurrently — the cancel-vs-fulfil
  race in `docs/testing-strategy.md` proves exactly one wins.
- **Every transition is audited** (invariant 7): actor, action,
  target order, previous state, new state, timestamp, outcome
  — including rejected attempts on sensitive transitions.
