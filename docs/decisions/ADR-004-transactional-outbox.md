# ADR-004 — Transactional outbox for cross-module events

**Status:** Accepted (Phase 0 design)
**Date:** 2026-11-21

## Context

Invariant 4 requires an order placement to commit completely —
order, lines, stock reservation, audit event — or not at all.
Inventory and Orders are separate modules with separate schemas
(ADR-003), and Audit must record state transitions it does not
own. Writing to two schemas in one transaction would quietly
re-couple the modules the schemas were meant to separate.

## Decision

- When a module changes state that other modules must react to,
  it writes a **domain event row into its own schema's outbox
  table in the same transaction** as the state change. The event
  either exists with the change, or not at all — never one without
  the other.
- A single in-process relay polls the outbox tables and delivers
  each event to the subscribed modules' public APIs (in-process
  calls in the monolith). Delivery is **at-least-once**; consumers
  must be idempotent and deduplicate on the event id.
- Events are facts in the past tense (`OrderPlaced`,
  `StockReserved`, `OrderCancelled`), carry the acting user's id,
  a correlation id, and the minimal payload the consumers need —
  never secrets (invariant 6).
- Audit is a consumer like any other: it subscribes to the events
  it must record. Audit never reaches into another module's
  tables to reconstruct history.
- No message broker in Phase 0–1. The outbox table *is* the
  queue; if a broker is ever justified, the relay is the only
  component that changes.

## Consequences

- **Positive:** atomicity holds without cross-schema transactions;
  module coupling stays at the API level; the event log doubles
  as debugging evidence for the concurrency tests in Milestone 2.
- **Negative:** consumers see events slightly after commit
  (eventual consistency inside the process). Accepted: no Phase
  0–1 requirement needs synchronous cross-module reaction — stock
  reservation itself happens through Inventory's synchronous
  public API inside the order transaction's orchestration, and
  the outbox carries the *record* of it, not the reservation.
- **Rule for consumers:** a consumer that fails must not block
  the relay; failed deliveries are retried with backoff and land
  in a dead-letter row after the retry budget, visible to
  operations (Module E).
