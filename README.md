# Aegis Platform

**Secure Enterprise Order & Identity Platform**

Java 21 · Spring Boot 3 · Spring Security · Keycloak (OIDC) · PostgreSQL ·
Flyway · Testcontainers · Docker

A production-style backend platform for a fictional company — Sentinel
Commerce — that manages customers, products, orders, inventory and
employees. Its security is part of the architecture, not a layer added
at the end: strict authorization, transaction integrity under
concurrency, a full audit trail, and automated security tests that
prove the boundaries hold.

> Status: **Milestone 0 — design pack complete (Nov 2026).**
> Architecture, threat model, ADRs, module specifications, error
> catalogue, testing strategy and authorization matrix are written,
> and the repository carries an MIT license. No application code
> yet — code starts at Module A once the Java reactivation gate is
> passed.

## The problem it solves

The platform must stay correct and safe when things go wrong or turn
hostile:

- A customer can access only their own orders.
- A warehouse employee cannot grant themselves administrator
  privileges.
- Stock cannot go negative when several customers order the same
  product simultaneously.
- A failed operation never leaves an order half-processed.
- Security events are recorded without ever logging passwords, tokens
  or other secrets.
- Vulnerable dependencies and failing security checks are visible in
  the pipeline, not discovered in production.

## Architecture

A **modular monolith**: one deployable Spring Boot application,
internally divided into modules with explicit boundaries. Each module
owns its data and exposes a narrow interface to the others — the walls
that would become service boundaries later, if scale ever earns them.

```
src/main/java/com/aegis/platform/
├── inventory/   Module A — products, warehouses, stock
├── orders/      Module B — orders, payments, order lifecycle
├── identity/    Module C — users, roles, Keycloak/OIDC integration
├── audit/       Module D — security events, audit trail
└── platform/    Module E — shared config, security, observability
```

See [docs/architecture.md](docs/architecture.md) for the module map
and boundary rules, and [docs/threat-model.md](docs/threat-model.md)
for the threat model skeleton.

## Build order

| Module | Scope | Core concepts |
| --- | --- | --- |
| A — Inventory | Products, categories, warehouses, stock adjustments | JPA, constraints, pagination, validation |
| B — Orders | Order lifecycle, stock reservation, payments | Transactions, concurrency, idempotency |
| C — Identity | Authentication, roles, authorization | Spring Security, Keycloak, OIDC, RBAC |
| D — Audit | Security event log, admin audit trail | Append-only records, secret-safe logging |
| E — Platform | Hardening, CI security checks, observability | Semgrep, dependency checks, Actuator |

## Stack

| Component | Technology |
| --- | --- |
| Language | Java 21 LTS |
| Framework | Spring Boot 3.x |
| Security | Spring Security |
| Identity provider | Keycloak with OIDC |
| Database | PostgreSQL |
| Persistence | Spring Data JPA / Hibernate |
| Migrations | Flyway |
| Unit testing | JUnit 5, Mockito |
| Integration testing | Testcontainers |
| API documentation | OpenAPI / Swagger UI |
| Build | Maven |
| Local environment | Docker Compose |
| CI/CD | GitHub Actions |
| Security checks | Semgrep, OWASP Dependency-Check, secret scanning |
| Observability | Spring Boot Actuator, Micrometer |

## Author

Zakaria Chelouati — Backend Software Engineering (Java / Spring),
High-Tech Rabat, Génie Logiciel.
Portfolio: https://zakaria-portfolio-inky.vercel.app/
