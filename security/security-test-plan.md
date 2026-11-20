# Aegis Platform — Security Test Plan

Phase 0 design document. "Secure" is a claim; this plan is how the
claim gets proven. Every threat from the threat model maps to an
automated test with an expected result. A control without a test is
a hope, not a control.

References: OWASP Top 10, OWASP ASVS. Intentionally vulnerable
examples live only in disposable test branches, never in production
configuration.

## Threat → test mapping

| Threat | Test | Expected result |
| --- | --- | --- |
| Broken object-level authorization (BOLA/IDOR) | Customer A requests customer B's order by ID | 404; no data from B's order in the response |
| Broken function-level authorization | Warehouse operator calls an administrator endpoint | 403; attempt written to the audit trail |
| Privilege escalation | Any user submits a role change for themselves, incl. via extra JSON fields | Rejected; role unchanged; attempt audited |
| Mass assignment | Order/product payloads include `role`, `ownerId`, internal price fields | Fields ignored or request rejected; stored values are server-set |
| Injection | Search and filter parameters carry SQL fragments | Parameterised queries only; no error, no extra rows |
| Authentication failures | Requests with missing, expired, tampered or wrong-issuer tokens | 401 before business logic runs |
| Race conditions | N parallel orders compete for the last unit of stock | Exactly the orders that fit available stock succeed; stock never negative; losers get a clean 409/422 |
| Duplicate submission | Same order POST retried with the same idempotency key | One order, one reservation, identical response |
| Sensitive data exposure | Force errors across endpoints; inspect logs and audit records | No stack traces, tokens, passwords or SQL in any output |
| Audit tampering | Application roles attempt UPDATE/DELETE on audit records | Denied at the database grant level |
| Vulnerable dependencies | CI dependency scan on every pull request | Known-vulnerable dependency fails the check and blocks merge |
| Brute force / abuse | Repeated failed logins and denied requests | Rate limiting at the edge; patterns visible in audit queries |

## Signature demonstrations

Three scenarios get a written walkthrough (setup, expected, actual,
evidence) once their modules land:

1. **Authorization** — a customer attempts to read another
   customer's order; access is denied and the denial is audited.
2. **Concurrency** — simultaneous requests race for the last unit;
   the stock invariant holds and only valid orders succeed.
3. **Dependency security** — a known vulnerable dependency is
   introduced on a disposable branch; the scanner detects it and
   the CI policy blocks it.

## Cadence

- Unit and integration tests run on every pull request.
- Security tests are part of the same suite — not a separate,
  optional phase.
- Findings are recorded with severity, fix and any accepted
  exception, in the pull request that resolves them.
