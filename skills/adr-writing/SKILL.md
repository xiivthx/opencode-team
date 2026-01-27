---
name: adr-writing
description: Create or update an Architecture Decision Record (ADR-xxx) with a contract-first Rust design (types/traits/errors), explicit trade-offs, and a migration/compat plan. Use when an AC needs architectural decisions or when contracts/shared types change.
compatibility: opencode
metadata:
  owner: silas
  domain: architecture
  freedom: medium
---

# ADR Writing (Silas)

Write ADRs that are **actionable**, **contract-first**, and **migration-aware**.
This skill is about making decisions legible and implementable — not writing a novel.

## When to use me
Use this skill when:
- A Spec introduces new flows that require architectural decisions (AC-xxx)
- You are adding/changing shared types, service boundaries, storage shape, or API behavior
- You need to justify a trade-off (performance vs simplicity, consistency vs availability, etc.)
- You need a migration/compatibility plan (anything that can break Torin/Lyra/Alex/Quinn)

## Related skills (routing)
- For hexagonal boundaries (ports/adapters, dependency direction): use skill({ name: "hex-architecture" }).
- For service operational decisions (config, logs, statelessness, disposability): use skill({ name: "twelve-factor-service" }).
- For git/PR conventions on ADR updates: use skill({ name: "git-workflow" }).
- For general design heuristics: use skill({ name: "engineering-principles" }).

## What I must produce
A Markdown ADR document with:
- `ADR-xxx` identifier
- Links to related `AC-xxx` (spec) and any existing `API-xxx / SEC-xxx / TEST-xxx`
- A **Contract Block** (Rust signatures/types/errors)
- Trade-offs and alternatives
- Migration/compat plan + rollout notes
- Clear implementation boundaries (who does what)

> Rule: This ADR defines interfaces and invariants. It does **not** implement function bodies.

---

# ADR levels (pick one)
## Lite ADR (default for small decisions)
Use when: confined change, low blast radius, no storage migration.
Must include: Decision, Contract Block, Risks, Compatibility note.

## Full ADR (for large / risky changes)
Use when: breaking changes, migrations, multi-service flows, concurrency, security-sensitive.
Includes everything in the template.

---

# Template (copy/paste)

## ADR-XXX: <Title>
### Status
- Draft / In Review / Approved / Superseded
- Owner: Silas
- Reviewers: (Torin / Lyra / Quinn / Viktor / Kai / Alex as relevant)

### References
- Spec: AC-xxx, AC-yyy
- API: API-xxx (if contract exposed)
- Tests: TEST-xxx (if already planned)
- Security: SEC-xxx (if applicable)

---

## 1) Context
What’s happening, what’s changing, and why this decision exists.

## 2) Decision
One paragraph: what we will do.

## 3) Goals and Non-Goals
- Goals: bullet list
- Non-Goals: bullet list

## 4) Constraints
- Performance/latency budget (if any)
- Storage constraints
- Environment constraints (no_std, wasm, etc.)
- Operational constraints (CI, deploy cadence)

---

## 5) Contract Block (Rust-first, no bodies)
Define the **public interfaces** that implementations must satisfy.

### 5.1 Data types
```rust
// Example shapes; replace with your actual types.
pub struct FooId(pub uuid::Uuid);

#[derive(Debug, Clone)]
pub struct Foo {
    pub id: FooId,
    pub name: String,
}
````

### 5.2 Errors (canonical)

```rust
#[derive(thiserror::Error, Debug)]
pub enum FooError {
    #[error("not found")]
    NotFound,
    #[error("conflict")]
    Conflict,
    #[error("unauthorized")]
    Unauthorized,
    #[error("internal")]
    Internal,
}
```

### 5.3 Traits / service interfaces

```rust
#[async_trait::async_trait]
pub trait FooService {
    async fn create(&self, input: CreateFoo) -> Result<Foo, FooError>;
    async fn get(&self, id: FooId) -> Result<Foo, FooError>;
}
```

### 5.4 Invariants

List invariants that must always hold (great for tests).

* Example: “Foo.name is non-empty and max 80 chars.”
* Example: “create is idempotent for the same request key.”

> Requirement:
>
> * Every invariant should be testable.
> * If exposed externally, errors should map to stable API codes (API-xxx).

---

## 6) Architecture

Describe the high-level design:

* Components/crates and responsibilities
* Data flow (optional diagram)
* Where state lives (DB, cache, memory)
* Concurrency strategy (locks, optimistic concurrency, etc.)

### 6.1 Dependency and crate boundaries

* No circular dependencies between crates
* Shared types live in one shared crate, imported by backend/UI as needed
* Prefer minimal public surfaces; keep private modules private

---

## 7) Alternatives considered

List 2–3 viable alternatives:

* Option A: …
* Option B: …
  For each: pros/cons + why rejected.

## 8) Risks and mitigations

* Performance risks
* Operational risks
* Complexity risks
* Security risks (link SEC-xxx if needed)

---

## 9) Compatibility and Migration Plan (required if anything can break)

### 9.1 Compatibility

* Who breaks? (Backend / UI / Firmware / Tests)
* Can old + new coexist?
* Versioning strategy (if applicable)

### 9.2 Data migration (if persistent data changes)

* Forward migration steps
* Backfill strategy
* Rollback plan (or “no rollback; mitigate via …”)

### 9.3 Rollout plan

* Feature flags (if needed)
* Incremental deploy steps
* Observability requirements (what logs/metrics must exist)

---

## 10) Implementation notes (boundaries, not code)

Assign responsibilities clearly:

* Torin: implement backend interfaces + error mappings + tracing
* Lyra: UI states + error UX mapping
* Quinn: contract tests + regression gates
* Viktor: security review + supply chain checks
* Kai: CI/devcontainer parity + pipeline gates
* Alex: firmware constraints if relevant

> Rule: ADR defines the “what and why” and the interface contracts.
> Implementers own the “how” inside those interfaces.

---

## 11) Open questions

List unresolved items with owners and due phase:

* Q1 (Owner: …, Due: Spec/ADR/Impl): …
* Q2 …

---

# Authoring rules (Silas-style)

* Prefer **types and signatures** over paragraphs.
* Be explicit about error taxonomy and invariants.
* If a change can break clients, write a migration/compat plan.
* Keep the ADR tight: remove commentary that doesn’t affect implementation.
