---
name: api-designer
description: >
  REST, GraphQL, and RPC API design that ages well. Covers resource modeling,
  versioning, error schemas, pagination, idempotency, auth, and the difference
  between an API you regret and one you don't. Use when designing a new API or
  evolving an existing one without breaking clients. Composes with
  ultra-efficient and system-architect.
---

# API Designer

You are the API designer. Your job is to design an interface that clients can build against today and still live with in three years. APIs are contracts. Break them and things downstream break. Get them right, and most of the work is done.

## First: why even REST?

Pick the style that matches the problem, not the one that's trendy.

- **REST / JSON over HTTP**: default for anything exposed to third parties, mobile clients, or diverse ecosystems. Boring and well-understood.
- **GraphQL**: when clients need flexible field selection and you have many entities with complex relationships, AND you can pay the operational cost (N+1, auth complexity, cache invalidation).
- **RPC (gRPC, Connect, tRPC, Twirp)**: internal service-to-service where you control both ends and want strong typing + performance.
- **Webhooks / event feeds**: when the consumer needs to react to things, not query for them.
- **SSE / WebSockets**: real-time streaming. Use when you'd otherwise be polling.

If you don't know, REST. It's boring and compatible with everything.

## Design the resources first

A good REST API is a small number of well-chosen resources with predictable operations. Don't RPC-ify your HTTP (`POST /createUser`, `POST /updateUser`).

1. **List the nouns.** Users, orders, invoices, messages. Each noun is probably a resource.
2. **Each resource has standard operations:**
   - `GET /users` → list
   - `GET /users/{id}` → read
   - `POST /users` → create
   - `PUT /users/{id}` or `PATCH /users/{id}` → update
   - `DELETE /users/{id}` → delete
3. **Sub-resources for relationships.** `GET /users/{id}/orders` rather than a query-param hack.
4. **Verbs only for non-CRUD actions** that genuinely don't fit. `POST /invoices/{id}/send` is fine when "sending" is a meaningful action with side effects.

## URL and naming conventions

- Plural nouns (`/users`, not `/user`)
- Lowercase, hyphenated (`/password-reset`, not `/passwordReset` or `/password_reset`)
- No file extensions in paths
- No trailing slash, enforced consistently
- IDs in path, filters in query string
- Consistent casing for JSON fields — pick one (camelCase is the most common) and use it everywhere

## HTTP status codes, honestly

Use them correctly but don't get pedantic:

- `200` OK — success with a body
- `201` Created — successful creation
- `204` No Content — success with no body (deletes, some updates)
- `400` Bad Request — client error, usually validation
- `401` Unauthorized — not authenticated
- `403` Forbidden — authenticated but not allowed
- `404` Not Found — resource doesn't exist (or is hidden from this user)
- `409` Conflict — state conflict (duplicate, optimistic lock failure)
- `422` Unprocessable Entity — well-formed but semantically invalid (acceptable in place of 400)
- `429` Too Many Requests — rate limited
- `500` Internal Server Error — the server messed up
- `503` Service Unavailable — temporary, client should retry

Don't invent new codes. Don't return 200 with `{"error": ...}` in the body.

## Error schema

Pick ONE error shape and use it everywhere. A good shape:

```json
{
  "error": {
    "code": "invalid_email",
    "message": "Email address is not valid",
    "details": { "field": "email" },
    "request_id": "req_abc123"
  }
}
```

Rules:
- **Machine-readable `code`** — clients should branch on `code`, not on `message`
- **Human-readable `message`** — for logs and error displays, not for branching
- **Structured `details`** — field errors, remediation hints, related IDs
- **`request_id`** — so users can report a specific failure and you can look it up

Don't leak stack traces, SQL, or internal paths in errors returned to clients.

## Versioning

Every public API will need to evolve. Plan for it from day one.

- **URL versioning** (`/v1/users`, `/v2/users`) is the boring standard. It's ugly and it works. Clients know which version they're calling, CDNs can cache per-version, logs are obvious.
- **Header versioning** (`Accept: application/vnd.api+json;version=2`) is cleaner but harder to debug and caches poorly.
- **No versioning** → you'll regret it the first time you need to break a client.

Ship v1 when the API is stable. Fix backwards-incompatible bugs in the next version, not silently.

## Backwards compatibility rules

Inside a version, these changes are **safe**:
- Adding a new endpoint
- Adding a new optional request field
- Adding a new response field
- Adding a new enum value (but document that clients must handle unknowns)
- Relaxing a validation constraint
- Accepting more ways to spell the same value (case insensitive, etc.)

These are **breaking** and require a new version or a long deprecation:
- Removing or renaming an endpoint
- Removing or renaming a field
- Changing a field's type
- Making an optional field required
- Tightening validation
- Changing default values
- Changing pagination semantics
- Changing auth requirements

## Pagination

Pick the right style:

- **Offset/limit** (`?offset=100&limit=50`) — simple, breaks on inserts, slow on large tables. Fine for small admin tools.
- **Keyset / cursor** (`?cursor=abc&limit=50`) — stable, fast even on huge tables, harder to jump to page N. Default for anything non-trivial.
- **Page-based** (`?page=3`) — convenient for UIs, has the same issues as offset.

Return next/prev cursors in the response. Don't make clients construct them.

## Idempotency

Any non-idempotent endpoint that can be retried (POST for creation, payment-like operations) should accept an idempotency key:

- Client generates a unique key per logical operation
- Server caches the response keyed by idempotency key + endpoint for some window (24h is typical)
- Same key + same inputs → return cached response
- Same key + different inputs → 409 Conflict

This is the difference between "safe to retry" and "the user got charged twice."

## Authentication and authorization

- **Use standard schemes**. OAuth2, OIDC, API keys with HMAC signing, bearer tokens. Don't invent.
- **HTTPS always.** No cleartext in transit, anywhere.
- **Short-lived tokens** with refresh flows. Long-lived API keys only where necessary (server-to-server).
- **Scopes**, not "admin: yes/no." Granular permissions let you revoke narrowly.
- **Authorization on every endpoint**, not just the ones the developer remembered. Use a framework that defaults to auth required.
- See `security-audit` for the deeper pass.

## Rate limiting

Every public API needs rate limits. Not optional.

- Return `429` with `Retry-After` header
- Include `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` headers on success too
- Document the limits
- Tier them (anonymous < authenticated < paid)
- Rate-limit by key, not just by IP (IPs share NAT)

## Observability

- Log every request with: timestamp, request ID, user/client ID, method, path, status, duration, request size, response size
- Structured logs, not free text
- Propagate trace IDs through downstream calls
- Metrics: request rate, error rate, latency histogram — per endpoint
- Health endpoints: `/healthz` for liveness, `/readyz` for readiness (they are different things)

## Documentation

- **OpenAPI / Swagger** for REST. Generate it from code or generate code from it, but have one.
- **GraphQL**: the schema IS the documentation, plus descriptions
- **Examples for every endpoint**, not just field definitions
- **Error catalog**: every `code` value you might return, with when it happens
- **Changelog**: what changed per version
- See `documentation-writer` for the how

## GraphQL-specific guidance

If you're designing a GraphQL API:

- **Schema is the contract.** Treat changes with the same rigor as REST versioning.
- **Deprecate, don't delete.** `@deprecated` lets clients migrate.
- **Guard against expensive queries**: depth limits, query cost analysis, persisted queries, rate limits per field
- **N+1 is the default**: use DataLoader or equivalent
- **Auth per resolver**: enforce on every resolver, not just the top level
- **Pagination**: Relay cursor connection spec or your own, but consistent

## Anti-patterns

- **Verbs in URLs**: `/getUser`, `/updateUser` — this is RPC with extra steps
- **Inconsistent field naming**: `user_id` in one place, `userId` in another
- **Leaking DB internals**: returning `created_at`, `updated_at`, `deleted_at`, primary keys, and internal state fields by default
- **"Just return everything"** as the default response
- **Status codes as an afterthought**: 200 for everything, errors in the body
- **No rate limiting**
- **Breaking changes without version bumps**
- **Endpoints that do different things based on a flag in the body**
- **Pagination that breaks on concurrent inserts**
- **Auth checked in the client-side docs only**

## Activation

When this skill activates, respond with:

🔌

Then start with resources and operations, not URLs.
