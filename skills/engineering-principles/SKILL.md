---
name: engineering-principles
description: Core decision heuristics for simplicity, decoupled architecture, and maintainable code boundaries.
compatibility: opencode
metadata:
  owner: elias
  domain: engineering
  freedom: medium
---

# Engineering Principles (Practical, Repo-Scoped)

This skill is a compact set of **decision heuristics** used during:
- spec writing (what is the true requirement?)
- ADR design (where do boundaries belong?)
- implementation (what is the simplest correct solution?)
- PR review (is this maintainable and safe to evolve?)

It is intentionally practical: if a principle doesn’t change decisions, it doesn’t belong here.

## When to use me
Use this skill when:
- reviewing or writing a spec/ADR
- evaluating a refactor or architecture change
- PR review (especially “clever” code)
- deciding whether to introduce a dependency or framework
- deciding whether code belongs in core vs adapter layer

## Related skills (routing)
- Spec templates and sad paths: skill({ name:"spec-writing" })
- ADR structure and migration/compat: skill({ name:"adr-writing" })
- Hex boundaries: skill({ name:"hex-architecture" })
- TDD and tests-from-AC: skill({ name:"tdd-playbook" })
- Quality gates: skill({ name:"qa-gates" })
- Git/PR hygiene: skill({ name:"git-workflow" })

---

# The Complexity Budget
Complexity is a currency. Spend it only when you must.

## Rule
- If two solutions satisfy the requirements, choose the simpler one.
- If you add complexity, you must state what it buys (performance, safety, extensibility) and how it will be tested.

## Complexity signals (red flags)
- more than one new abstraction introduced “just in case”
- indirection without clear boundaries
- “framework types” leaking into core logic
- configuration explosion
- new dependency without measurable benefit

---

# Contracts over vibes
A contract is a testable promise: types, interfaces, error codes, invariants.

## Rules
- Make contracts explicit at boundaries:
  - API schema + error codes
  - ports (traits) between core and infrastructure
  - UI state machine + error UX mapping
- If a contract changes, the change must be:
  - documented (AC/ADR/API)
  - tested (TEST)
  - communicated (PR)

---

# Cohesion and Coupling
## Prefer high cohesion
Keep related code together so it can change together.

## Minimize coupling
A module should have as few reasons to change as possible.

### Signals of bad coupling
- a small change forces edits across many unrelated modules
- shared “god types” used everywhere
- bidirectional dependencies
- UI and backend duplicating “almost the same” type system

---

# Boundaries are design, not bureaucracy
Boundaries prevent entropy.

## Rules of boundaries
- Core logic should not depend on infrastructure details.
- Infrastructure is replaceable; core logic is not.
- Put IO at the edges. Keep policy inside.

(Apply `hex-architecture` when boundaries are unclear.)

---

# Software design principles

## Classic design principles (compact)
- **SRP (Single Responsibility):** a module should have one primary reason to change.
- **OCP (Open/Closed):** extend behavior via new modules/implementations behind contracts, not by editing many call sites.
- **LSP (Substitutability):** trait implementations must preserve the trait’s semantic promises (not just compile).
- **ISP (Interface Segregation):** prefer small, purpose-built traits over “god traits”.
- **DIP (Dependency Inversion):** core depends on abstractions (ports), adapters depend on core (see `hex-architecture`).

## Public surface discipline
- Minimize `pub` surface area; keep internals private.
- Prefer stable DTOs/contracts at boundaries; avoid leaking framework/DB types.
- Prefer type-driven constraints (“make illegal states unrepresentable”) at boundaries.

## Least knowledge (Law of Demeter)
- Avoid deep chaining and reaching through multiple objects/modules.
- If you need `a.b().c().d()`, you likely need a boundary or a helper with a clearer contract.

## Side effects at the edges
- Keep core logic deterministic and testable.
- Isolate I/O and global state (time, randomness, network, DB) behind ports/adapters.

---

# DRY, but not at gunpoint
Duplicate code is sometimes cheaper than a bad abstraction.

## Guidance
- Duplicate tiny pieces until the pattern is stable.
- Abstract when:
  - you have 3+ real uses, or
  - the abstraction reduces risk/bugs, or
  - it enforces a contract you must not violate.

Avoid “mega utils” and “helpers” that become dumping grounds.

---

# Make failure modes explicit
Every system fails. Mature systems fail predictably.

## Rules
- Define sad paths in specs.
- Encode failure expectations in tests.
- Ensure user-facing behavior is safe and recoverable.
- Observability is part of correctness (logs/spans/metrics).

---

# Optimize for debugging
Future-you is a detective with bad coffee and limited time.

## Rules
- Make changes easy to bisect (small PRs, readable history).
- Prefer structured logs at boundaries (request_id, error_code).
- Keep error messages safe and stable for UX mapping.
- Avoid nondeterministic behavior without instrumentation.

---

# Performance: measure, don’t guess
## Rules
- Don’t optimize blindly.
- When you touch hot paths, add benchmarks.
- Accept a performance trade-off only if you can justify it and keep it visible.

---

# Security: default to suspicion
## Rules
- Treat new dependencies as new attack surface.
- Avoid logging sensitive data.
- Make privilege boundaries explicit.
- If unsure, create `SEC-xxx` and ask for a security pass.

---

# Review playbook (fast checklist)
Use this to review a spec/ADR/PR quickly.

## Requirements and scope
- [ ] Is the requirement concrete and testable?
- [ ] Are non-goals explicit?
- [ ] Does it avoid unnecessary features?

## Design and boundaries
- [ ] Are boundaries clear (core vs adapters)?
- [ ] Are contracts explicit (types/errors/invariants)?
- [ ] Is coupling minimized?

## Tests and determinism
- [ ] Are tests derived from ACs?
- [ ] Are sad paths covered?
- [ ] Is anything flaky/nondeterministic?

## Operability
- [ ] Can we debug this in production (tracing/logs)?
- [ ] Are configs/secrets handled properly?
- [ ] Is rollout/rollback considered?

## Complexity budget
- [ ] Is this the simplest correct solution?
- [ ] If complex, does the value justify it?
- [ ] Is complexity localized and documented?

---

# Common anti-patterns (call these out explicitly)
- “We might need it later” abstraction (YAGNI)
- Leaky layers (DB/HTTP types in core)
- Utility dumping grounds (“helpers” that do everything)
- Hidden breaking changes (contract shifts without `API-xxx`)
- Inconsistent error handling (UI cannot map reliably)
- Flaky tests accepted as “normal”
- “Works on my machine” parity gaps (dev != CI)
