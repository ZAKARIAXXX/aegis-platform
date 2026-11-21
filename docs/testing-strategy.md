# Aegis Platform — Testing Strategy

Phase 0 design document. Aegis exists to produce *proof*: every
critical business rule in `docs/requirements.md` must be backed
by an automated test that would fail if the rule broke. A feature
without its proof is not done (see the definition of done in
`docs/roadmap.md`).

## The pyramid

1. **Unit tests** — the bulk. Domain rules in isolation:
   lifecycle transitions, quantity arithmetic, validation.
   Fast, no Spring context, no database.
2. **Module integration tests** — one module against real
   PostgreSQL via **Testcontainers**: Flyway migrations, JPA
   mappings, schema constraints, repository queries. Mocks of
   the database are forbidden at this level — the constraints
   are part of the design (ADR-003).
3. **API tests** — the whole application over HTTP with a real
   Keycloak test container: authentication, role and ownership
   authorization, error catalogue behaviour
   (`docs/error-catalogue.md`), idempotency replay.
4. **Security tests** — the negative cases from
   `security/security-test-plan.md`, run as ordinary API tests:
   cross-customer access, self-escalation attempts, denied
   actions producing audit records. Security tests are not a
   separate suite that can be skipped; they run in CI with
   everything else.

## The concurrency proofs (Milestone 2)

The tests this project is ultimately judged by:

- **Last-unit race:** N parallel orders for a product with one
  unit available → exactly one succeeds, stock never negative
  (invariant 3).
- **Duplicate submission:** the same idempotency key sent twice,
  concurrently and sequentially → one order, one reservation,
  identical responses (invariant 5).
- **Cancel vs fulfil race:** cancellation and fulfilment fired
  at the same order simultaneously → exactly one wins; the
  loser's transition is rejected with
  `INVALID_STATE_TRANSITION`; stock ends consistent either way.
- **Rollback purity:** a placement forced to fail mid-way
  leaves no order, no lines, no reservation, and exactly one
  audit record of the failure (invariants 4 and 7).

Concurrency tests must control their interleaving (latches and
barriers), not rely on sleeps and luck; a race test that passes
by timing is a false proof.

## Rules

- Tests are written against requirements and error codes, not
  against implementation details.
- Every bug found after the fact earns a regression test before
  its fix.
- CI runs the full suite on every pull request; a red security
  check blocks the merge (Module E requirement).
