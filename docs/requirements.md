# Aegis Platform — Requirements

Phase 0 design document. What the platform must do, and the business
invariants it must never break. Implementation follows these rules;
when a rule and an implementation disagree, the implementation is
wrong.

## Business invariants (non-negotiable)

1. **Ownership.** A customer can read and act on only their own
   orders, addresses and payment records. Ownership is checked
   server-side on every request — never inferred from the client.
2. **No self-escalation.** No user can grant themselves a higher
   role, directly or through any combination of API calls.
3. **Stock integrity.** Available stock never goes negative, and the
   sum of reserved + fulfilled quantities never exceeds physical
   stock — including under simultaneous orders for the same product.
4. **Atomicity.** An order placement either commits completely
   (order, lines, stock reservation, audit event) or not at all. A
   failed operation leaves no partial writes.
5. **Idempotent retries.** Repeating an order-creation request with
   the same idempotency key returns the original result — never a
   second order, never a second reservation.
6. **Secret hygiene.** Passwords, tokens and secrets never appear in
   logs, error responses or audit records.
7. **Auditability.** Every privilege change, denied-access attempt,
   stock override and order state transition produces an audit
   record: actor, action, target, timestamp, outcome.

## Functional requirements by module

### Module A — Inventory

- Create, update, search and archive products; categories and
  warehouses.
- SKU is unique; stock levels are tracked per product per warehouse.
- Every stock adjustment records a reason and the acting user.
- Read and write permissions are separate: viewing the catalogue is
  not permission to adjust stock.

### Module B — Orders

- Order lifecycle: `DRAFT → PLACED → CONFIRMED → FULFILLED`, with
  `CANCELLED` reachable from `DRAFT`, `PLACED` and `CONFIRMED`.
- Prices and totals are validated on the server at placement time;
  client-supplied prices are never trusted.
- Placing an order reserves stock atomically; cancelling releases
  the reservation where the lifecycle allows it.
- Order creation accepts an idempotency key (see invariant 5).

### Module C — Identity

- Authentication is delegated to Keycloak (OIDC); the API never
  handles passwords.
- Roles: `CUSTOMER`, `SUPPORT_AGENT`, `WAREHOUSE_OPERATOR`,
  `ADMINISTRATOR`. Authorization combines role (what kind of action)
  with ownership (on which objects).
- Deny by default: an endpoint with no explicit permission rule is
  closed, not open.

### Module D — Audit

- Audit records are append-only: no update, no delete, for any role.
- Records are queryable by actor, target, action and time range.
- Audit storage is separate from ordinary application logs.

### Module E — Platform

- Health and metrics endpoints for operations.
- CI runs tests and security checks on every pull request; a red
  security check blocks the merge.

## Non-goals (Phase 0–1)

- No microservices, no message broker, no Kubernetes.
- No payment-provider integration — payments are modelled as records,
  not processed.
- No public storefront UI; the deliverable is the API and its proof.
