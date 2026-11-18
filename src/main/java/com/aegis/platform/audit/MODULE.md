# Module D — Audit

**Scope:** security events and the administrative audit trail.

**Owns:** audit records — append-only by design. No update, no
delete path exists in this module's API.

**Receives events from:** Identity (logins, role changes, denied
access), Orders (overrides, cancellations), Inventory (stock
adjustments), Platform (security configuration changes).

**Key rules:**

- Records contain no secrets: no passwords, no tokens, no full
  request bodies.
- Auditing never breaks the operation it records: if the audit write
  fails, the failure itself is surfaced loudly, not swallowed.
- Every record carries actor, action, target, timestamp and outcome.

**Status:** bones only — no code yet.
