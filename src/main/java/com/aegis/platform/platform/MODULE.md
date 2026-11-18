# Module E — Platform

**Scope:** the domain-free plumbing shared by all modules —
Spring Security configuration, global error handling, observability
(Actuator / Micrometer), shared configuration.

**Owns:** security filter chain configuration, error response
format, health and metrics endpoints, CI security-check wiring
(Semgrep, OWASP Dependency-Check, secret scanning).

**Key rules:**

- Only genuinely domain-free code lives here. If it knows what an
  order is, it does not belong in this module.
- Error responses never leak stack traces, SQL, or internal class
  names to clients.
- Health endpoints expose liveness, not internal detail, to
  unauthenticated callers.

**Status:** bones only — no code yet.
