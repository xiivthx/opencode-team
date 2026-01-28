---
name: hex-architecture
description: Hexagonal architecture playbook (ports & adapters) for this repo: define domain + application core, isolate infrastructure behind ports, enforce dependency direction, and keep contracts stable. Use when designing modules/crates, writing ADRs, or refactoring for testability and maintainability.
compatibility: opencode
metadata:
  owner: silas
  domain: architecture
  freedom: medium
---

# Hex Architecture (Ports & Adapters)

This skill defines how we apply **hexagonal architecture** in this repo:
- keep business logic independent of frameworks and infrastructure
- define explicit boundaries (ports) as stable contracts
- implement infrastructure as adapters behind those ports
- enforce dependency direction to prevent “framework creep”

## When to use me
Use this skill when:
- writing an ADR that introduces new services, modules, or boundaries
- refactoring to improve testability or reduce coupling
- adding a new external dependency (DB, network API, queue, filesystem)
- your codebase is drifting into “everything depends on everything”

## Related skills (routing)
- Decision records and migration plans: skill({ name: "adr-writing" })
- Backend contract + error code discipline: skill({ name: "rust-backend-standards" })
- UI async state + error UX mapping: skill({ name: "rust-ui-patterns" })
- Engineering heuristics: skill({ name: "engineering-principles" })
- Tests and determinism: skill({ name: "tdd-playbook" }), skill({ name: "qa-gates" })
- Traceability IDs: skill({ name: "team-contract-ids" })

---

# Core idea
**Inside is policy, outside is details.**
The application core should not know (or care) about:
- databases, HTTP frameworks, queues, filesystems
- OS/environment specifics
- serialization formats as a primary type system

---

# Vocabulary (use consistently)
- **Domain**: business rules, invariants, entities/value objects.
- **Application**: use cases (orchestrates domain + ports).
- **Port**: an interface the core depends on (usually a trait).
  - inbound port: "what the app can do" (use-case API)
  - outbound port: "what the app needs" (DB, clock, external service)
- **Adapter**: implementation of a port (DB repo, HTTP client, API handler).
- **Infrastructure**: frameworks, databases, runtime glue (outside the hex).

---

# Dependency direction (non-negotiable)
Dependencies must point **inward**.

Allowed:
- adapters → ports (implement)
- adapters → application/domain (call/use)
- application → domain (use)
- application → ports (depend on interfaces)

Forbidden:
- domain → adapters/infrastructure
- application → concrete DB/client/framework types
- ports → adapters

If you need a framework type inside core, you probably need an adapter.

---

# Layer responsibilities
## Domain layer (pure)
Must be:
- framework-free
- deterministic
- heavily unit-tested

Contains:
- entities/value objects
- invariants and validation
- domain errors (business meaning)

## Application layer (use cases)
Contains:
- orchestration of steps and policies
- transaction boundaries (as concepts, not DB-specific code)
- calls to outbound ports
- mapping domain results to application results

## Ports (interfaces)
Contains:
- trait definitions (outbound ports)
- DTOs owned by the core (inputs/outputs of use cases)
- boundary errors (stable, intentional)

## Adapters (infrastructure)
Contains:
- DB repositories
- HTTP handlers/controllers
- external API clients
- serialization + transport mapping
- logging/tracing glue
- time providers, random providers, etc.

---

# Rust-specific patterns (recommended)
## Port definition (outbound)
Define ports as traits in a `ports` module or crate:

```rust
#[async_trait::async_trait]
pub trait FooRepo {
    async fn get(&self, id: FooId) -> Result<Option<Foo>, RepoError>;
    async fn save(&self, foo: Foo) -> Result<(), RepoError>;
}
````

## Application service depends on ports (not adapters)

```rust
pub struct FooService<R: FooRepo> {
    repo: R,
}

impl<R: FooRepo> FooService<R> {
    pub async fn create(&self, input: CreateFoo) -> Result<Foo, UseCaseError> {
        // business logic + calls to repo
        // no SQL, no HTTP here
        todo!()
    }
}
```

## Adapters implement ports

```rust
pub struct PostgresFooRepo { /* pool */ }

#[async_trait::async_trait]
impl FooRepo for PostgresFooRepo {
    async fn get(&self, id: FooId) -> Result<Option<Foo>, RepoError> { todo!() }
    async fn save(&self, foo: Foo) -> Result<(), RepoError> { todo!() }
}
```

---

# Project structure (pick one and be consistent)

## Option A: Single crate, clear modules

```
src/
  domain/
  application/
  ports/
  adapters/
  main.rs (composition root)
```

## Option B: Workspace crates (recommended for larger repos)

```
crates/
  core-domain/
  core-application/
  core-ports/
  adapter-http/
  adapter-db/
  adapter-external-foo/
  app-server/ (composition root)
```

Rule: “composition root” (wiring) lives at the edge: `main.rs` or `app-*`.

---

# Error boundaries (avoid leaking infrastructure)

## In core (domain/application)

Errors should be meaningful and stable (business terms).
Do not expose:

* SQL error strings
* HTTP client internals
* framework types

## At the adapter boundary

Map:

* infra errors → boundary errors (or `UseCaseError`)
* boundary errors → API error codes (`rust-backend-standards`)

Rule: errors crossing the boundary must be stable enough for UI + tests.

---

# Testing strategy (hex makes this easy)

## Domain tests

* unit tests for invariants and pure logic

## Application tests

* unit tests with fake/mock ports (no DB/network)
* deterministic, fast

## Adapter tests

* integration tests for DB adapters and HTTP wiring
* contract tests ensuring adapter preserves port semantics

## Contract tests (API)

* assert response shape + error codes for client stability
* required when changing contracts

---

# Migration guidance (how to refactor safely)

Use the “strangler” approach:

1. Introduce ports and application service around existing implementation
2. Wrap old code in an adapter (even if ugly)
3. Add tests at the port boundary
4. Replace internals incrementally with cleaner adapters
5. Keep outward contracts stable; document changes with `ADR-xxx` / `API-xxx`

---

# Common anti-patterns (spot and fix)

* **God service**: one module owns everything (DB + HTTP + business rules)
* **Leaky adapters**: core depends on `sqlx::Row`, `reqwest::Response`, framework request types
* **Anemic domain**: “domain” is just data structs; all rules live in handlers
* **Ports that mirror DB schema**: ports should reflect use-case needs, not table columns
* **Adapters calling adapters**: keep edges separate; share through core ports/use cases

---

# Review checklist (Silas-style)

Before approving an ADR or major refactor:

* [ ] Clear boundaries: domain vs application vs adapters
* [ ] Ports exist for infra dependencies; core depends on traits, not concrete types
* [ ] Dependency direction points inward (no framework types in core)
* [ ] Error mapping is defined at boundaries (core errors → API error codes)
* [ ] Test plan exists at the right layers (mock ports for core, integration for adapters)
* [ ] Migration/compat plan exists if contracts or persistence are touched
