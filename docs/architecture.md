# Aegis Platform — Architecture

## Shape

One Spring Boot application (modular monolith), one PostgreSQL
database, one identity provider (Keycloak). No microservices, no
Kubernetes: the goal is a system that can be designed, secured,
tested and debugged end-to-end by one engineer.

## Modules and boundaries

| Module | Package | Owns | May depend on |
| --- | --- | --- | --- |
| A — Inventory | `com.aegis.platform.inventory` | Products, categories, warehouses, stock levels, stock adjustments | platform (config only) |
| B — Orders | `com.aegis.platform.orders` | Orders, order lines, payments, order status | inventory (via its public API), platform |
| C — Identity | `com.aegis.platform.identity` | Users, roles, permission mapping, Keycloak integration | platform |
| D — Audit | `com.aegis.platform.audit` | Security events, audit records (append-only) | nothing — it receives events |
| E — Platform | `com.aegis.platform.platform` | Security configuration, error handling, observability, shared types | nothing domain-specific |

## Boundary rules

1. A module never reaches into another module's tables or repositories.
   Cross-module calls go through the owning module's public service
   interface.
2. A module never imports another module's internal classes — only its
   published API package.
3. Audit receives events; it never calls back into domain modules.
   Auditing must not be able to break the operation it records.
4. Shared code lives in `platform` only when it is genuinely
   domain-free (configuration, error types, security plumbing).
   "Used by two modules" is not enough — it must also mean the same
   thing in both.

## Data and transactions

- PostgreSQL is the single source of truth.
- Schema changes are Flyway migrations, versioned with the code.
- The transaction boundary is the application service method: an
  order placement either commits completely or not at all.
- Stock integrity under concurrency is enforced in the database
  (constraints + atomic updates), not by hope in application code.

## Security architecture (summary)

- Authentication is delegated to Keycloak (OIDC); the API validates
  tokens with Spring Security.
- Authorization is role- and ownership-based: a role decides what kind
  of action is allowed, ownership decides on which objects.
- Every security-relevant event (login, privilege change, denied
  access, stock override) produces an audit record that contains no
  secrets.
- Detail: [threat-model.md](threat-model.md).
