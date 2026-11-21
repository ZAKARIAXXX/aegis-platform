# Aegis Platform — Observability

Phase 0 design document. Module E's contract: when something
fails at 3 a.m., the logs must explain it — and when an auditor
asks, the trail must prove it — without ever leaking a secret
(invariant 6).

## Two different records, two different jobs

| | Application logs | Audit records (Module D) |
|---|---|---|
| Purpose | Diagnose behaviour and failures | Prove who did what |
| Lifetime | Operational (rotated, short) | Append-only, retained |
| Content | Technical events, timings, errors | Actor, action, target, outcome |
| Mutability | Disposable | No update, no delete, any role |

Logs are never used as audit evidence, and audit records are
never written to the log stream. Audit is fed by outbox events
(ADR-004), not by log scraping.

## Structured logging rules

- JSON, one event per line, with a fixed field set:
  `timestamp`, `level`, `service`, `correlationId`, `actorId`
  (when authenticated), `module`, `event`, `durationMs`.
- **Correlation ids:** accepted from the `X-Correlation-Id`
  header or generated at the edge; propagated to every module
  call and stamped on every outbox event, so one request can be
  followed across the whole monolith.
- **Redaction is structural, not careful:** passwords, tokens,
  authorization headers and payment-record fields are excluded
  by the logging configuration's allow-list of fields — never
  by developers remembering not to print them. A field that is
  not on the allow-list does not reach the log, including inside
  exception messages and request/response dumps (which are off
  by default).
- Errors log the correlation id and the error code from
  `docs/error-catalogue.md` — the same id the client received
  in its `INTERNAL` response, so a user report can be traced
  to exactly one log trail.

## Health and metrics (Spring Boot Actuator)

- `/actuator/health` — liveness/readiness for operations;
  detailed health components are **not** exposed publicly
  (they leak topology).
- `/actuator/metrics` and `/actuator/prometheus` — restricted
  to the operations network/role; business counters worth
  watching from day one: orders placed/cancelled per hour,
  `INSUFFICIENT_STOCK` rejections, denied-access attempts
  (401/403/ownership-404 rate — a spike is a probing signal),
  outbox dead-letter count (ADR-004).
- **Alerting seeds (Phase 1):** dead-letter count > 0,
  denied-access spike, health down. Three alerts, each with an
  owner and a runbook line — an alert nobody acts on is noise
  and gets deleted.

## The test

Milestone 4 acceptance includes a redaction test: a scripted
run of the signature flows (including failed logins and denied
actions) after which the complete log output is grepped for
seeded secrets — zero hits is the only passing result.
