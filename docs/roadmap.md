# Aegis Platform — Roadmap

Milestones, not deadlines. Each milestone ends with something that
runs and can be explained, not a pile of half-finished features.
Estimates assume focused part-time work alongside studies.

## Milestone 0 — Design ✅ (Phase 0)

- [x] Requirements and business invariants — `docs/requirements.md`
- [x] Architecture and module boundaries — `docs/architecture.md`
- [x] Threat model skeleton — `docs/threat-model.md`
- [x] Data model — `docs/data-model.md`
- [x] API conventions — `docs/api-conventions.md`
- [x] Architecture decisions — `docs/decisions/`
- [x] Security test plan — `security/security-test-plan.md`
- [ ] Local environment ready: JDK 21, Maven, Docker

## Milestone 1 — Java & API foundation (~1 week)

- Maven project, Spring Boot skeleton, PostgreSQL via Docker Compose.
- Module A: products and categories — DTOs, validation, Flyway
  migrations, global error handling per the API conventions.

## Milestone 2 — Inventory & orders (~1–2 weeks)

- Stock levels and adjustments with audit reasons.
- Module B: order lifecycle, atomic stock reservation, idempotency.
- Concurrency tests: rollback, cancellation, duplicate requests,
  the last-unit race.

## Milestone 3 — Identity & security (~1–2 weeks)

- Keycloak in Docker Compose; Spring Security as resource server.
- Roles and ownership checks on every sensitive operation.
- Negative authorization tests from the security test plan.

## Milestone 4 — Quality & observability (~1 week)

- Module D: audit events, queryable trail, redaction rules.
- Integration tests with Testcontainers (PostgreSQL + Keycloak).
- Structured logging, Actuator health and metrics.

## Milestone 5 — DevSecOps & release (~1 week)

- GitHub Actions CI: tests, Semgrep, OWASP Dependency-Check,
  secret scanning, container image scan.
- OpenAPI documentation, synthetic demo data, setup guide.
- Signature security demonstrations written up with evidence.

## Definition of done (whole platform)

- [ ] A fresh clone starts with documented commands.
- [ ] Schema is created entirely by migrations.
- [ ] Every critical business rule has an automated test.
- [ ] Customers cannot access other customers' resources.
- [ ] Unauthorized administrative actions are rejected and audited.
- [ ] Concurrent-order tests preserve the stock invariants.
- [ ] Duplicate order requests are handled safely.
- [ ] Logs and errors expose no credentials or sensitive data.
- [ ] CI runs tests and security checks on every pull request.
- [ ] README documents architecture, setup, API and security
      decisions.
- [ ] Demo uses synthetic data with a repeatable security
      walkthrough.
