# Aegis Platform — Threat Model (skeleton)

Phase 0 skeleton: assets, actors, trust boundaries and the security
invariants the design must hold. The full STRIDE pass is written
module by module as each module is built.

## Assets

- Customer accounts and personal data.
- Order and payment records.
- Inventory and stock levels (business integrity).
- Employee accounts and role assignments.
- The audit trail itself — an attacker who can rewrite history owns
  the system.

## Actors

- **Customer** — authenticated, owns their orders only.
- **Warehouse employee** — manages stock, never roles.
- **Support agent** — reads incidents and customer orders within scope.
- **Administrator** — manages users and roles; every action audited.
- **Unauthenticated visitor** — public catalogue surface only.
- **Hostile authenticated user** — a valid account probing for other
  users' data, privilege escalation, and race conditions. This is the
  actor the design is built against.

## Trust boundaries

1. Client ↔ API (all input untrusted).
2. API ↔ Keycloak (token validation; the API never sees passwords).
3. API ↔ PostgreSQL (parameterised access only, least-privilege user).
4. Application ↔ audit log (append-only; no update, no delete).

## Security invariants (must always hold)

1. A customer can access only their own orders — enforced server-side
   on every request, never by hiding UI elements.
2. No user can grant themselves a higher role.
3. Stock never goes negative, including under simultaneous orders.
4. A failed operation leaves no partial writes.
5. Passwords, tokens and secrets never appear in logs or audit
   records.
6. Every privilege change and denied-access attempt is auditable.

## STRIDE pass

_To be completed per module during its build phase._

| Category | Module A | Module B | Module C | Module D |
| --- | --- | --- | --- | --- |
| Spoofing | | | | |
| Tampering | | | | |
| Repudiation | | | | |
| Information disclosure | | | | |
| Denial of service | | | | |
| Elevation of privilege | | | | |
