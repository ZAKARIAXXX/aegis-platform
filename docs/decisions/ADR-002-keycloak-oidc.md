# ADR-002 — Authentication delegated to Keycloak (OIDC)

- **Status:** Accepted, 2026-11-20
- **Context:** The platform needs login, roles and token-based API
  access. Hand-rolling password storage and token issuance is the
  classic way a portfolio project teaches itself the wrong lessons —
  and the classic way real systems get breached.
- **Decision:** Delegate authentication to **Keycloak** over
  **OpenID Connect**. The API is an OAuth 2.0 resource server:
  Spring Security validates access tokens (signature, expiry,
  issuer) on every request. The API never sees, stores or logs a
  password. Aegis keeps only the subject → role mapping it needs
  for authorization (see data model, Module C).
- **Consequences:**
  - Authentication and authorization stay separate concerns:
    Keycloak proves *who* you are; Aegis decides *what* you may do,
    including ownership checks no identity provider can know.
  - Token theft and expiry are handled by the protocol (short-lived
    access tokens, refresh at the provider), not by custom code.
  - Local development runs Keycloak in Docker Compose alongside
    PostgreSQL; integration tests use Testcontainers for both.
  - The cost is one more moving part to configure — accepted,
    because identity is the part of the system where "good enough"
    home-grown code fails hardest.
