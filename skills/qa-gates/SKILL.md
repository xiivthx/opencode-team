---
name: qa-gates
description: Non-negotiable quality metrics for determinism, contract enforcement, and performance regression testing.
compatibility: opencode
metadata:
  owner: quinn
  domain: qa
  freedom: low
---

# QA Gates (Quinn)

This skill defines **non-negotiable quality gates** for changes that could affect correctness, contracts, or performance.

## When to use me
Use this skill when:
- Creating/updating tests or QA plans
- Reviewing PR quality and deciding pass/fail
- Adding/changing endpoints, response shapes, or error codes
- Introducing hot-path logic (parsers, crypto, state machines, tight loops)
- Making CI required checks / gate policies

## Core principles
- **Determinism beats heroics**: flaky tests are failing tests.
- **Contracts are executable**: specs and error codes must be asserted in tests.
- **Perf regressions are bugs**: treat measurable slowdowns as failures.

## Related skills (routing)
- For the step-by-step TDD workflow and test structuring: use skill({ name: "tdd-playbook" }).
- For git/PR conventions that help traceability: use skill({ name: "git-workflow" }).

---

# 1) Required gates (baseline)
These are the default “must-pass” gates for merge.

## 1.1 Lint / format gate
- `cargo fmt --check`
- `cargo clippy -- -D warnings`

## 1.2 Test gate
- `cargo test` (or workspace equivalent)
- Tests must be deterministic (see section 2)

## 1.3 Contract gate (when APIs/behavior change)
- Contract tests assert:
  - HTTP status
  - response shape
  - `error.code` and `retryable` fields (if failure scenario)
- Any change to public behavior must link an upstream ID (`AC-xxx`, `ADR-xxx`, `API-xxx`) and a test ID (`TEST-xxx`).

## 1.4 Performance gate (when hot paths change)
- Micro-benchmarks using Criterion for hot paths.
- Fail the build if regression > 10% (policy in section 4).

---

# 2) Determinism rules (no flaky tests)
A test is flaky if it can pass/fail with the same code.

## Forbidden patterns (unless explicitly justified)
- Sleeping for timing (`sleep`, “wait 200ms then assert”)
- Depending on wall clock time or timezones
- Randomness without fixed seeds
- Shared mutable global state across tests
- Network calls to real external services
- Order-dependent tests (must pass under parallel execution)

## Required practices
- Use fixed seeds for any pseudo-random input.
- If time is involved, inject a clock / mock time source (no real time in assertions).
- Isolate data per test (unique IDs / fresh temp DB / clean fixtures).
- Make failures descriptive: assert the contract, not “it broke”.

---

# 3) Traceability: AC → TEST mapping (mandatory for features)
For every `AC-xxx` you must provide:
- at least one `TEST-xxx` that validates it, OR
- a written “N/A” with justification (rare)

Recommended table (keep it in the spec doc or `docs/traceability.md`):

| AC | TEST | Type | Notes |
|---|---|---|---|
| AC-001 | TEST-010 | integration | happy path |
| AC-001 | TEST-011 | integration | conflict sad path |
| AC-002 | TEST-012 | unit | pure logic invariant |

---

# 4) Performance policy (Criterion micro-benchmark gate)
## When benchmarks are required
Add/Update benchmarks when changes touch:
- parsers/formatters/serialization
- crypto/hashing/compression
- alloc-heavy transformations
- state machines / tight loops
- DB query shaping or result processing on hot endpoints

## Criterion setup (standard)
- Put benchmarks in `benches/*.rs`.
- Keep benchmark inputs stable (fixed dataset, no randomness unless seeded).
- Avoid I/O; benchmark pure computation or well-controlled in-memory work.

## Regression threshold (default)
- If a change introduces > **10%** regression vs baseline on the target metric, mark as **FAIL**.

### Enforcement notes (practical)
- Benchmarks are sensitive to machine variance.
- Prefer enforcing the fail gate on a **consistent CI runner** (same CPU class).
- If CI hardware is not stable, treat regression as a **blocker for investigation**, not “merge anyway”.

## What to report in PR/QA notes
- Which benchmarks ran
- Baseline vs new measurement (the specific stat you compare)
- Whether regression exceeds threshold
- If “no perf impact expected”, state why (and link code paths)

---

# 5) Contract tests (API + error codes)
Use contract tests whenever the backend contract matters to UI.

## Must assert
- Success: required fields exist, types/format constraints enforced
- Failure: `error.code` is stable, `retryable` matches policy, message is user-safe
- Status codes match expectations for each sad path

## Snapshot policy (optional but recommended)
- Snapshot response shapes for stability.
- Breaking changes require:
  - explicit `API-xxx`
  - snapshot update
  - UI mapping update (if error codes changed)

---

# 6) Review checklist (Quinn’s PR gate)
Before approving:
- [ ] Tests are deterministic (no timing/random/order dependence)
- [ ] AC coverage is traceable (AC-xxx → TEST-xxx)
- [ ] Contract tests exist for any API/behavior changes
- [ ] Error-code contract is asserted where relevant
- [ ] Benchmarks added/updated for hot-path changes
- [ ] Any perf regression is explained and either fixed or formally accepted by owner

---

# 7) Coordination rules
- Specs define `AC-xxx` (Vera). Architecture defines `ADR-xxx` (Silas).
- Backend behavior changes create/update `API-xxx` (Torin).
- QA creates `TEST-xxx` and blocks merge if gates fail (Quinn).
- Security issues create `SEC-xxx` and may block deploy (Viktor).
