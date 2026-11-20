# ADR-001 — Modular monolith before any distribution

- **Status:** Accepted, 2026-11-20
- **Context:** Aegis must demonstrate backend engineering, security
  and reliability. The fashionable default — microservices from day
  one — distributes every problem before the domain is understood:
  network failures between modules, distributed transactions for a
  single order, and security boundaries that exist on slides but not
  in running code.
- **Decision:** Build one Spring Boot application, one PostgreSQL
  database, internally divided into modules (inventory, orders,
  identity, audit, platform) with enforced boundaries:
  - a module never touches another module's tables or repositories;
  - cross-module calls go through the owning module's public API;
  - audit receives events and never calls back.
- **Consequences:**
  - Transactions stay local and simple — order placement is one
    database transaction, which is exactly what invariant 4 needs.
  - The whole system runs and debugs on one machine with Docker
    Compose.
  - The module walls are the future service boundaries: if scale
    ever earns distribution, the seams already exist and the split
    is mechanical, not archaeological.
  - The cost is discipline: boundaries hold only because the rules
    above are enforced in review and tests, not by the network.
