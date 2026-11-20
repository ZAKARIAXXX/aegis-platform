# Aegis Platform — API Conventions

Phase 0 design document. Every endpoint follows these rules so the
API behaves as one system, not a pile of controllers.

## Versioning

- URL versioning: all endpoints live under `/api/v1/`.
- A breaking change ships as a new version; `/api/v1` is never
  silently changed under existing clients.

## Resources and verbs

- Nouns, plural, lowercase: `/api/v1/products`, `/api/v1/orders`.
- `GET` reads, `POST` creates or triggers an action, `PUT` replaces,
  `PATCH` partially updates, `DELETE` archives or removes.
- Actions that are not CRUD use a sub-resource:
  `POST /api/v1/orders/{id}/cancel`.

## Error format

Every error response uses one envelope. No stack traces, no internal
class names, no SQL text — ever (invariant 6).

```json
{
  "error": {
    "code": "ORDER_STOCK_INSUFFICIENT",
    "message": "Requested quantity exceeds available stock.",
    "details": { "productId": "…", "available": 2 }
  }
}
```

- `code` is a stable machine-readable string; `message` is for
  humans and may be reworded without notice.
- HTTP status carries the class: 400 validation, 401 unauthenticated,
  403 authenticated but not allowed, 404 not found **or not yours**
  (another customer's resource is indistinguishable from a missing
  one), 409 state conflict, 422 business-rule violation.

## Pagination, filtering, sorting

- List endpoints accept `page` (0-based), `size` (default 20, max
  100), `sort` (`field,asc|desc`) and resource-specific filters.
- List responses return `content`, `page`, `size`,
  `totalElements`, `totalPages`.

## Writes and idempotency

- `POST /api/v1/orders` requires an `Idempotency-Key` header. A
  retry with the same key returns the original response; the same
  key with a different body is a 409.
- Mass assignment is impossible by construction: request DTOs
  contain only client-settable fields. `role`, `ownerId`, internal
  prices and status are never accepted from the client.

## Authentication

- `Authorization: Bearer <OIDC access token>` on every non-public
  endpoint. Missing, expired or wrongly signed tokens are rejected
  with 401 before any business logic runs.
