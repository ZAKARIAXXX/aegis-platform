# Aegis Platform — Authorization Matrix

Phase 0 design document. Who may do what, per role, before a
line of security code exists. Derived from
`docs/requirements.md` (Module C) and the error catalogue.
Where this matrix and an implementation disagree, the
implementation is wrong.

## Roles

`CUSTOMER` · `SUPPORT_AGENT` · `WAREHOUSE_OPERATOR` ·
`ADMINISTRATOR`

Authorization = **role** (what kind of action) **× ownership**
(on which objects). Role alone never grants access to another
customer's data.

## Matrix

| Action | CUSTOMER | SUPPORT_AGENT | WAREHOUSE_OPERATOR | ADMINISTRATOR |
|---|---|---|---|---|
| Search / view catalogue | ✔ | ✔ | ✔ | ✔ |
| Create / update product | — | — | — | ✔ |
| Archive product | — | — | — | ✔ |
| View stock levels | own orders only* | ✔ | ✔ | ✔ |
| Adjust stock (with reason) | — | — | ✔ | ✔ |
| Override stock (break-glass) | — | — | — | ✔ (audited) |
| Create order | ✔ (self) | ✔ (on behalf, audited) | — | ✔ |
| View order | own only | ✔ | fulfilment view† | ✔ |
| Cancel order | own, while `PLACED` | ✔ | — | ✔ |
| Confirm / fulfil order | — | ✔ | ✔ (fulfil) | ✔ |
| View audit records | — | — | — | ✔ |
| Manage users / roles | — | — | — | ✔ |

\* A customer sees availability ("in stock / low / out of
stock"), never exact warehouse quantities.
† The fulfilment view shows lines and quantities, not prices
or payment records.

## Hard rules (from the invariants)

1. **Ownership before existence.** A customer asking for
   another customer's order gets `NOT_FOUND`, not `FORBIDDEN`
   (invariant 1, error catalogue).
2. **No self-escalation.** Role changes are administrator-only,
   and an administrator cannot act on their own role record;
   every grant or revocation is audited (invariants 2 and 7).
3. **Deny by default.** An endpoint with no row in this matrix
   is closed to everyone until the matrix says otherwise.
4. **Read ≠ write.** Catalogue visibility never implies stock
   or order write access (Module A requirement).
5. **Denied attempts are audited** on sensitive resources,
   including the ownership 404s (invariant 7).
6. **Support "on behalf" is explicit.** When a support agent
   creates or cancels an order for a customer, the audit record
   names both the agent (actor) and the customer (owner) —
   never the agent alone.

## Test hooks

Every "—" cell is a negative test in
`security/security-test-plan.md`; every "own only" cell is a
cross-customer test. The matrix is complete when every cell
has a corresponding passing test in CI.
