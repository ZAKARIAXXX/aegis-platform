# Module C — Identity

**Scope:** user accounts, roles, permission mapping, Keycloak/OIDC
integration.

**Owns:** the mapping between Keycloak identities and platform
users, role assignments, account status.

**Key rules:**

- Passwords live in Keycloak, never in this platform's database.
- No user can grant themselves a higher role — role changes require
  an administrator and produce an audit event.
- Authorization is two-layered: role (what kind of action) and
  ownership (on whose objects). Both are checked server-side.
- Disabled accounts fail authentication immediately, not at next
  login attempt by luck.

**Status:** bones only — no code yet.
