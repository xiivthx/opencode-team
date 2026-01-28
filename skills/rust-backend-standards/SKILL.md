---
name: rust-backend-standards
description: High-performance Rust patterns for APIs, stable error contracts, and structured observability.
compatibility: opencode
metadata:
  owner: torin
  domain: backend
  freedom: low
---

# Rust Backend Standards (Torin)

This skill enforces backend conventions for correctness, operability, and long-term maintainability.

## When to use me
Use this skill when you:
- Implement or modify an HTTP/gRPC endpoint
- Change request/response shapes, status codes, or error handling
- Touch persistence (SQL migrations, queries, transactions)
- Add a backend dependency (crate) or change runtime behavior
- Need to instrument tracing/logging for debuggability

## Related skills (routing)
- For writing tests first (unit/integration/contract) from AC/API behavior: skill({ name: "tdd-playbook" }).
- For service ops discipline (config/logging/process model): skill({ name: "twelve-factor-service" }).
- For architectural layering decisions beyond this file: skill({ name: "hex-architecture" }).

## Non-negotiables (runtime safety)
### No panics in runtime paths
- **Forbidden in production code paths:** `unwrap()`, `expect()`, `panic!()`
- **Allowed:** in tests, prototypes/spikes (but remove before merge)
- Prefer typed errors (`thiserror`) and explicit propagation with `?`

### No `println!` for logs
- Use `tracing::{info,warn,error,debug}`.
- Logs must be structured (key-value fields), not free-form dumps.

---

# Contract & Traceability (IDs)
- Any behavior change must link to at least one upstream reference ID:
  - `AC-xxx` (Spec), `ADR-xxx` (Architecture), `API-xxx` (Contract change), `TEST-xxx` (QA), `SEC-xxx` (Security)
- **If request/response/error shape changes:** create or update an `API-xxx` reference in the PR/task.

---

# API Compatibility Rules (what counts as breaking)
Treat these as **breaking changes** unless explicitly versioned/flagged:
- Removing fields or changing field meaning
- Changing error codes or HTTP statuses for existing scenarios
- Tightening validation that rejects previously-valid inputs
- Changing default values

Usually safe (non-breaking):
- Adding new optional fields (with defaults)
- Adding new error codes for new functionality (must be documented)

When in doubt:
- Write/Update an `API-xxx` and notify UI/QA owners in the task description.

---

# Error Code Contract (stable boundary behavior)
## Goals
- Errors are **predictable** for UI + tests
- Codes are **stable** (do not rename casually)
- Each error clearly indicates whether retry makes sense

## Canonical error response shape
Pick one shape and keep it consistent across endpoints. Example:

```json
{
  "error": {
    "code": "RESOURCE_CONFLICT",
    "message": "Safe message suitable for users",
    "retryable": false,
    "details": {
      "hint": "Optional; non-sensitive"
    }
  }
}
````

### Rules

* `code` is **stable**, UPPER_SNAKE_CASE
* `message` must be safe (no secrets, no SQL, no stack traces)
* `retryable` is **explicit**
* `details` is optional and must never contain sensitive data

## Error classification (recommended)

* **User** (4xx): invalid input, permission, not found, conflict
* **System** (5xx): unexpected failures, dependency down
* **Retryable**: transient dependency issues, timeouts (only when retry is meaningful)

## HTTP status mapping (example)

* `INVALID_ARGUMENT` → 400
* `UNAUTHORIZED` → 401
* `FORBIDDEN` → 403
* `NOT_FOUND` → 404
* `RESOURCE_CONFLICT` → 409
* `RATE_LIMITED` → 429
* `UPSTREAM_TIMEOUT` → 504 (retryable: true)
* `INTERNAL` → 500 (retryable: false by default)

> Keep the mapping in one place (module) so it doesn't drift.

## Rust error type pattern

Use typed errors at the boundary:

```rust
#[derive(thiserror::Error, Debug)]
pub enum ApiError {
    #[error("invalid argument: {0}")]
    InvalidArgument(String),

    #[error("unauthorized")]
    Unauthorized,

    #[error("forbidden")]
    Forbidden,

    #[error("not found")]
    NotFound,

    #[error("resource conflict")]
    Conflict,

    #[error("rate limited")]
    RateLimited,

    #[error("upstream timeout")]
    UpstreamTimeout,

    #[error("internal")]
    Internal,
}
```

### Converting internal errors

* Internal errors may use `anyhow::Error` or concrete error types internally,
  but must be mapped to `ApiError::Internal` (or appropriate public code).
* Always log internal diagnostics via tracing (see below), **not** in the response.

---

# Observability: Tracing (required)

## Minimum tracing requirements

* Every request should have a request identifier (`request_id`)
* Every public service method or handler should start a span
* Logs should include:

  * `request_id`
  * `error_code` (when failing)
  * `status` or `http_status`
  * Optional: `user_id` (only if non-sensitive and policy allows)
  * Do **not** log secrets/tokens/PII

## Example instrumentation pattern

* Use spans for handlers/services.
* Use structured fields:

```rust
tracing::info!(
  request_id = %request_id,
  endpoint = "POST /v1/foo",
  "request_received"
);
```

On error:

```rust
tracing::warn!(
  request_id = %request_id,
  error_code = "RESOURCE_CONFLICT",
  retryable = false,
  "request_failed"
);
```

## No sensitive logging

Never log:

* auth tokens, passwords, private keys
* raw DB rows containing PII
* full request bodies by default (allowlisted fields only)

---

# Database & Migrations (SQL discipline)

## Migrations

* All schema changes must use migrations (no manual DB drift).
* Migration files must be deterministic and reversible when feasible.
* If backfill is needed: document it (spec/ADR) and provide a plan.

## Queries

* Prefer compile-time checked queries when possible.
* Keep DB access behind a small repository/service boundary.
* Use transactions for multi-step writes that must be atomic.

## Concurrency expectations

* If concurrent writes can conflict:

  * define the strategy (optimistic concurrency / unique constraint / explicit lock)
  * map conflicts to `RESOURCE_CONFLICT` (409)

---

# Dependency policy (backend-specific)

* Avoid adding crates without a clear justification.
* Prefer well-maintained, widely-used crates.
* If adding a crate that touches security/crypto/serialization/parsing:

  * require a security review entry (`SEC-xxx`) or a checklist note
* Ensure workspace dependency versions remain consistent (avoid duplicate tokio versions, etc.).

---

# Testing minimums (backend)

For each endpoint/behavior change:

* At least one integration test covering the happy path
* At least one test for each **documented** sad path (from spec)
* If error codes change: add/adjust tests that assert `code` + `retryable` + status

> If a change introduces a new error code, tests must assert it.

---

# Pre-merge checklist (backend)

* [ ] No `unwrap/expect/panic` in runtime paths
* [ ] No `println!` used for logging
* [ ] Error codes/states follow contract (stable `code`, explicit `retryable`)
* [ ] Tracing includes request_id and error_code on failure
* [ ] Migrations added for schema changes (if any)
* [ ] Tests cover AC + sad paths; failures are deterministic
* [ ] If contract changed: `API-xxx` created/updated and linked in PR/task
