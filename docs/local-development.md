# Aegis Platform — Local Development

Phase 0 design document. The environment every milestone
assumes. The goal: a fresh clone starts with documented
commands and no tribal knowledge (definition of done,
`docs/roadmap.md`).

## Required tools

| Tool | Version | Why |
|---|---|---|
| JDK | 21 (LTS) | Language and runtime baseline. |
| Maven | 3.9+ (or the project wrapper `./mvnw`) | Build and dependency management. The wrapper is preferred — it pins the version for every machine. |
| Docker + Docker Compose | current | PostgreSQL and Keycloak run as containers; nothing is installed natively. |
| Git | current | Source control. |

## Services (Docker Compose, planned)

| Service | Port (host) | Notes |
|---|---|---|
| PostgreSQL 16 | 5432 | One database, one schema per module (ADR-003). Data in a named volume; `down -v` wipes it deliberately, never by accident. |
| Keycloak | 8080 | Realm `aegis` imported from a versioned realm file in the repo — no manual clicking in the admin console. Test users for each role (`CUSTOMER`, `SUPPORT_AGENT`, `WAREHOUSE_OPERATOR`, `ADMINISTRATOR`) are seeded by the same file, with clearly synthetic credentials for local use only. |

## Everyday commands (targets, to be wired in Milestone 1)

```bash
docker compose up -d          # start PostgreSQL + Keycloak
./mvnw verify                 # compile, unit + integration tests
./mvnw spring-boot:run        # run the API locally
docker compose down           # stop services (data survives)
```

## Configuration rules

- No secrets in the repository. Local configuration comes from
  environment variables or an uncommitted `.env` file; the repo
  ships `.env.example` with synthetic values only (invariant 6).
- The application fails fast at startup when a required variable
  is missing — a half-configured run is worse than no run.
- Profiles: `local` (default, synthetic data allowed) and
  `test` (Testcontainers manage their own services; nothing
  local is touched).

## First-run checklist (Milestone 1 acceptance)

1. Clone → `docker compose up -d` → `./mvnw verify` is green.
2. API answers its health endpoint.
3. A seeded `CUSTOMER` token can read the catalogue and nothing
   outside its permissions.
4. README's setup section matches these steps exactly — if it
   drifts, the README is the bug.
