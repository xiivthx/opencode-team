---
name: tdd-playbook
description: Test-driven development playbook for this repo: derive tests from AC-xxx, write failing tests first, implement minimal code, refactor safely, and keep tests deterministic. Use when changing behavior, adding features, or creating/adjusting QA gates and contract tests.
compatibility: opencode
metadata:
  owner: quinn
  domain: engineering-process
  freedom: low
---

# TDD Playbook (Team Standard)

This skill defines a repo-wide TDD workflow that is:
- **Traceable** (AC-xxx → TEST-xxx)
- **Deterministic** (no flaky tests)
- **Contract-aware** (API/error codes are asserted)
- **Maintainable** (tests clarify intent, not implementation details)

## When to use me
Use this skill when:
- Implementing or changing behavior (backend/UI/firmware logic)
- Writing tests for sad paths or edge cases
- Changing API error codes / response shapes
- You need a repeatable “how do we test this?” process

## Related skills (routing)
- For pass/fail gates and perf policy: skill({ name: "qa-gates" })
- For writing specs that become tests: skill({ name: "spec-writing" })
- For backend error contract and tracing: skill({ name: "rust-backend-standards" })
- For UI async/error UX patterns: skill({ name: "rust-ui-patterns" })
- For AC/ADR/API/TEST/SEC reference discipline: skill({ name: "team-contract-ids" })

---

# Core rules (non-negotiable)
1) **Behavior changes require tests.** If behavior changes and no test exists, the PR is blocked unless explicitly waived with justification.
2) **Tests must be deterministic.** If it’s flaky, it’s failing.
3) **Tests should assert contracts, not internal implementation.**
4) **Every new feature must map AC → TEST.** (Even a lightweight mapping is required.)

---

# Traceability: AC → TEST
## Required mapping
- Every `AC-xxx` must have at least one `TEST-xxx`, or an explicit “N/A” with justification.
- Add/update a mapping table (preferred location: `docs/traceability.md` or within the spec).

Suggested mapping table:
| AC | TEST | Type | Notes |
|---|---|---|---|
| AC-001 | TEST-010 | integration | happy path |
| AC-001 | TEST-011 | integration | conflict sad path |
| AC-002 | TEST-012 | unit | invariant |

---

# TDD loop (Red → Green → Refactor)
## Phase 0: Clarify the contract (before writing tests)
- Identify the requirement source:
  - Spec AC(s): `AC-xxx`
  - Architecture contract(s): `ADR-xxx` (if needed)
  - Backend contract(s): `API-xxx` (if request/response/errors change)
- Identify the observable outcome:
  - output data
  - state transition
  - emitted error code/status
  - UI state / displayed message mapping
  - firmware signal (log/LED/counter)

If you can’t describe the observable outcome, you’re not ready to write a test.

## Phase 1: RED (write a failing test)
Write the smallest test that fails for the right reason.
Rules:
- Name the test after the behavior, not the function name.
- Use Arrange → Act → Assert.
- Assert on stable outputs (contracts), not internal steps.

## Phase 2: GREEN (minimal implementation)
Make the test pass with the smallest reasonable change.
Rules:
- Avoid premature abstractions.
- Do not add “extra features” not covered by AC.

## Phase 3: REFACTOR (improve design safely)
Now clean up:
- reduce duplication
- improve naming
- isolate boundaries (ports/adapters if needed)
- remove temporary hacks

Rule: Refactor only while tests stay green.

## Phase 4: Commit discipline
Prefer 1–3 commits:
- `TEST:` add failing test(s)
- `IMPL:` minimal implementation
- `REFACTOR:` cleanup

Include reference IDs in PR description at minimum; commits can also include IDs if helpful.

---

# Choosing the right test type (fast decision)
## Unit tests
Use when:
- Pure logic, invariants, state machines, parsing/formatting
- No I/O and no external dependencies
Goal:
- Fast feedback; high signal; deterministic.

## Integration tests
Use when:
- Endpoint behavior, DB interactions, serialization boundaries, auth flows
Goal:
- Assert the system boundary as the user/client experiences it.

## Contract tests (required for APIs)
Use when:
- Backend response shape, status codes, error code contract (`error.code`, `retryable`)
Goal:
- Freeze external expectations so UI and other clients don’t break silently.

## UI tests (state + mapping)
Use when:
- Async state transitions (Loading/Empty/Ready/Error)
- Error code mapping to user messages and recovery actions
Goal:
- Prevent regressions in UX around failures and retries.

## Performance benchmarks (Criterion)
Use when:
- Hot paths (parsers, crypto, alloc-heavy transforms, tight loops)
Goal:
- Detect >10% regressions per repo policy (see `qa-gates`).

---

# Determinism checklist (must pass)
A test is unacceptable if it depends on:
- Wall-clock time, timezones, real sleeps
- Randomness without fixed seeds
- External network/services
- Shared global state across tests
- Execution order (tests must pass under parallel runs)

Fixes:
- Inject a clock / mock time
- Seed randomness
- Use test-local temp dirs/DB fixtures
- Use hermetic fakes/stubs, not real services

---

# Backend TDD pattern (Torin-aligned)
When changing endpoints or error behavior:
1) Write/adjust contract test asserting:
   - status code
   - response shape
   - failure response includes stable `error.code` and `retryable`
2) Add integration test for sad path(s) defined in spec:
   - conflict → 409 + `RESOURCE_CONFLICT`
   - timeout → 504 + `UPSTREAM_TIMEOUT` (retryable)
3) Only then implement/adjust handler/service code.

Rule:
- Never ship a new error code without a test asserting it.

---

# UI TDD pattern (Lyra-aligned)
When building a screen that calls an API:
1) Write test(s) for the UI state machine:
   - `Loading → Ready/Empty/Error`
2) Write test(s) for error mapping:
   - backend `error.code` → `UiError` (message/action/presentation)
3) Implement the component/screen minimal path.
4) Refactor to avoid prop drilling and centralize mapping.

Rule:
- UI must never display raw backend/internal error strings.

---

# Firmware TDD pattern (Alex-aligned)
Default:
- Test core logic on host (x86_64) using mocks for `embedded-hal` traits.
- Only then validate on-target for interrupts/DMA timing integration.

If on-target confirmation is required:
- Use `hardware-hil-testing` to run a structured human-in-the-loop protocol.

---

# Common pitfalls (avoid these)
- Over-mocking: tests that mirror implementation, not behavior.
- Testing private methods instead of outcomes.
- “Happy path only” tests with no sad paths.
- Flaky timing tests (sleep-based).
- Making tests so broad they become hard to diagnose when failing.

---

# Minimal PR checklist (TDD compliance)
- [ ] Behavior change is linked to AC-xxx (or justified as bug fix)
- [ ] At least one new/updated TEST-xxx exists and is mapped
- [ ] Tests are deterministic (no time/random/order dependence)
- [ ] API changes have contract tests (shape + error.code + retryable)
- [ ] UI changes have state + mapping coverage
- [ ] Hot-path changes include benchmarks or justification
