# Aegis Platform — Error Catalogue

Phase 0 design document. The stable error codes the API returns,
mapped to the business invariants in `docs/requirements.md`.
Format and envelope rules live in `docs/api-conventions.md`;
this catalogue fixes the *codes* so clients, tests and the
security test plan all speak the same language.

## Rules

- Codes are stable once published: a code's meaning never
  changes, codes are never reused.
- Error responses never reveal whether a resource exists when
  the caller has no right to know (see `NOT_FOUND` vs
  `FORBIDDEN` below).
- No error message contains secrets, stack traces, SQL, or
  internal identifiers beyond the resource the caller addressed
  (invariant 6).

## Catalogue

| Code | HTTP | Meaning | Invariant |
|---|---|---|---|
| `VALIDATION_FAILED` | 400 | Request body or parameters fail validation; `details` lists every offending field. | — |
| `UNAUTHENTICATED` | 401 | No valid access token. | 1 |
| `FORBIDDEN` | 403 | Authenticated, but the role does not permit this kind of action at all. | 1, 2 |
| `NOT_FOUND` | 404 | Resource does not exist — **or exists but belongs to another customer**. Ownership failures answer 404, never 403, so ids cannot be probed. | 1 |
| `SKU_CONFLICT` | 409 | A product with this SKU already exists. | 3 |
| `INSUFFICIENT_STOCK` | 409 | Requested quantity exceeds available stock at placement time. | 3 |
| `INVALID_STATE_TRANSITION` | 409 | The order lifecycle does not allow this transition from the current state (see `docs/order-lifecycle.md`). | 4 |
| `IDEMPOTENCY_KEY_REUSED` | 409 | The key was already used with a *different* payload. Same key + same payload replays the original result instead (invariant 5). | 5 |
| `RATE_LIMITED` | 429 | Too many requests; `Retry-After` header is set. | — |
| `INTERNAL` | 500 | Unexpected failure. The response carries a correlation id and nothing else useful to an attacker. | 6 |

## Notes for implementers

- Ownership checks run **before** existence responses are
  composed: the 404 for "not yours" and the 404 for "does not
  exist" must be indistinguishable in body and timing as far as
  reasonably possible.
- Denied-access attempts (401/403 and ownership 404s on
  sensitive resources) produce audit records (invariant 7) —
  the audit event is written even though the request failed.
- The security test plan (`security/security-test-plan.md`)
  references these codes directly; a test that cannot name its
  expected code is not yet a test.
