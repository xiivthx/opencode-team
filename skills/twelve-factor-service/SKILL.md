---
name: twelve-factor-service
description: Twelve-Factor service playbook tailored for this repo: config/secrets, stateless processes, logs/metrics, backing services, build-release-run, concurrency, disposability, and dev/prod parity. Use when designing or operating backend services, writing ADRs, or shaping CI/release workflows.
compatibility: opencode
metadata:
  owner: kai
  reviewers: [torin]
  domain: service-ops
  freedom: medium
---

# Twelve-Factor Service (Repo Playbook)

This skill applies **Twelve-Factor** principles in a practical, enforceable way for our services.
It focuses on outcomes: reproducibility, operability, and safe evolution.

## When to use me
Use this skill when:
- creating a new backend service or worker
- changing runtime configuration, deploy process, or observability
- writing ADRs that affect deploy/ops shape (process model, state, dependencies)
- designing endpoints/jobs that must behave well under failure, scaling, or restarts
- updating CI/release workflows (`ci-golden-path`)

## Related skills (routing)
- CI parity, pipelines, containers: skill({ name: "ci-golden-path" })
- Backend contract, error codes, tracing: skill({ name: "rust-backend-standards" })
- Architecture boundaries: skill({ name: "hex-architecture" })
- Quality gates and determinism: skill({ name: "qa-gates" })
- Security policies for deps/secrets: skill({ name: "security-supply-chain" })
- Git/PR conventions: skill({ name: "git-workflow" })

---

# What I must produce (for a new or changed service)
At minimum:
1) A **service runbook** section (how to run locally + required env vars + ports)
2) A **config contract** (required/optional config, defaults, validation)
3) An **observability contract** (logs/spans/metrics + identifiers)
4) A **deploy/release plan** (build vs release vs run, migration/admin steps)

If any of these are missing, the design is incomplete.

---

# The Twelve-Factor checklist (practical version)

## 1) Codebase
- One codebase tracked in version control; many deploys (dev/stage/prod).
- No environment-specific code branches.
- Environment differences must be config only.

## 2) Dependencies
- Declare all dependencies explicitly.
- Avoid “works on my machine” by using pinned toolchains and reproducible environments.
- Prefer devcontainer parity with CI (`ci-golden-path`).

## 3) Config (strict)
### Rule
All environment-specific settings come from **environment variables** (or an equivalent injected config provider), not committed files.

### Requirements
- Validate config at startup and fail fast with a clear error.
- Provide a single place listing required config keys (docs + code).
- Secrets must never be logged.
- Do not require interactive prompts at runtime.

### Recommended config pattern
- `APP_ENV=dev|staging|prod`
- `PORT=...`
- `DATABASE_URL=...`
- `LOG_LEVEL=...`
- `RUST_LOG=...` (if used)
- `SERVICE_NAME=...`
- `REQUEST_ID_HEADER=...` (if applicable)

## 4) Backing services
Treat DBs, caches, queues, and external APIs as attached resources.
- No hardcoded endpoints/credentials.
- Switching a backing service should be config change, not code rewrite.
- Make timeouts explicit; define retry behavior intentionally.

## 5) Build, release, run
### Rule
Separate:
- **Build**: compile artifacts
- **Release**: combine build + config
- **Run**: execute the release in an environment

Practical requirements:
- Build produces immutable artifacts (binary/container image).
- Release injects config and secrets (CI secrets manager or runtime injection).
- Run is identical across environments aside from config.

## 6) Processes (stateless)
- Service processes must be stateless and share-nothing.
- Persist state only in backing services (DB, cache, object store).
- In-memory caches are optional optimizations and must tolerate restart.

## 7) Port binding
- Bind to a port provided by config (e.g., `PORT`).
- Expose health endpoints (or equivalent) for orchestration:
  - `/healthz` (liveness)
  - `/readyz` (readiness) when dependencies are required

## 8) Concurrency
- Scale via processes/instances, not by manually adding “special singleton” nodes.
- If background jobs exist, run them as separate process types (worker vs web).
- Avoid hidden global locks. If you need distributed locking, make it explicit.

## 9) Disposability (fast startup, graceful shutdown)
### Requirements
- Startup should be fast and deterministic.
- Shutdown should be graceful:
  - stop accepting new work
  - finish in-flight work up to a timeout
  - flush logs/telemetry best-effort
- Handle SIGTERM/SIGINT correctly (or platform equivalent).

## 10) Dev/prod parity
- Keep dev/staging/prod as similar as possible.
- Avoid “dev-only” code paths that never run in prod.
- Use the same container/devcontainer toolchain as CI when feasible.

## 11) Logs as event streams
### Rule
The service writes logs to stdout/stderr; the platform routes them.
- No “log files” as the primary mechanism.
- Use structured logs/spans; no `println!`.
- Include identifiers:
  - request_id / trace_id
  - error_code (when failing)
  - service_name, version/commit

(Backend specifics live in `rust-backend-standards`.)

## 12) Admin processes
Run one-off tasks (migrations, backfills, repair scripts) as **separate admin processes**:
- same codebase
- same config injection
- same observability
- explicitly invoked (not “startup side effect”)

Rule: DB migrations should not be a hidden side effect of boot unless explicitly designed and approved.

---

# Operational guardrails (recommended)
## Timeouts and retries (must be intentional)
- Every external call must have a timeout.
- Retries must be limited and categorized (idempotent vs non-idempotent).
- Log retry decisions with structured fields (no spam).

## Health and readiness
- Liveness: process is up.
- Readiness: process can serve traffic (dependencies ready).
- If readiness fails, expose why (safe message).

## Versioning / identity
- Every deploy should expose:
  - build version (commit SHA)
  - service name
  - environment (dev/stage/prod)

---

# “Done” definition for Twelve-Factor compliance (service)
Before calling a service change complete:
- [ ] Config contract documented and validated at startup
- [ ] No secrets in logs; logs are structured to stdout/stderr
- [ ] Stateless process model; state lives in backing services
- [ ] Health checks exist and are meaningful
- [ ] Graceful shutdown implemented (with timeouts)
- [ ] Build/release/run separation is respected in CI/CD
- [ ] Dev/prod parity: devcontainer and CI use the same toolchain
- [ ] Admin tasks (migrations/backfills) are explicit and observable
