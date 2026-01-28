---
name: spec-writing
description: Guidelines for drafting testable requirement specifications with labeled Acceptance Criteria (AC).
compatibility: opencode
metadata:
  owner: vera
  domain: product-spec
  freedom: medium
---

# Spec Writing (Vera)

This skill produces **testable**, **unambiguous** specifications with labeled Acceptance Criteria (`AC-xxx`) and explicit failure scenarios (“sad paths”).

## When to use me
Use this skill when:
- Starting a new feature
- Changing behavior (even if small) that could break clients or tests
- You need clarity on edge cases, concurrency, retry behavior, or data validation
- QA needs a concrete plan and pass/fail criteria

## Related skills (routing)
- For test-first flow and structuring tests from AC: use skill({ name: "tdd-playbook" }).
- If the feature implies architecture boundaries (ports/adapters, layers): use skill({ name: "hex-architecture" }).
- If this spec describes a long-running service (config/logs/deploy concerns): use skill({ name: "twelve-factor-service" }).
- For shared team heuristics (simplicity, coupling, naming): use skill({ name: "engineering-principles" }).

## Outputs (what you must produce)
One Markdown spec document containing:
- Overview and scope
- Explicit Acceptance Criteria labeled `AC-xxx`
- Sad paths and failure modes
- Data contract and error contract expectations (at the level needed by backend/UI)
- Test strategy mapping (at least a draft)
- Traceability pointers (link to ADR/API/TEST/SEC IDs when they exist)

> If your repo has a preferred location, write specs there (e.g., `docs/specs/<feature>.md`). If not, create `docs/specs/`.

---

# Spec Levels (choose one)
## Lite Spec (fast path)
Use for: small changes, bug fixes, limited blast radius.
Must include: Overview, AC list, 3–5 sad paths, and test notes.

## Full Spec (default for new features)
Use for: new features, API changes, workflow changes, concurrency, security-sensitive behavior.
Includes everything in this skill template.

---

# Template (copy/paste)
## Title
**<Feature Name>** — Spec

### Status
- Draft / In Review / Approved
- Owner: Vera
- Stakeholders: (Elias / Silas / Torin / Lyra / Quinn / Viktor / Kai / Alex)

### References
- Traceability: (link or pointer to mapping doc/table if available)
- Related ADR(s): ADR-xxx (if known)
- Related API change(s): API-xxx (if known)
- Related Security item(s): SEC-xxx (if known)

---

## 1) Problem
What user/system problem are we solving? One paragraph.

## 2) Goals
Bulleted, concrete, measurable where possible.

## 3) Non-Goals
Explicitly list what this spec does NOT cover (prevents scope creep).

## 4) Assumptions
E.g., expected load, latency budget, auth assumptions, data availability.

## 5) User Stories / Use Cases
List primary flows in plain language.

---

## 6) Acceptance Criteria (AC)
Write each AC as a **testable** scenario. Label each item.

### AC-001: <Short name>
**Given** ...
**When** ...
**Then** ...

**Notes**
- Data requirements:
- Permissions/auth:
- Observability expectation (optional): log/metric/event

### AC-002: ...
...

> Rules:
> - Every “must/should” statement must live inside an AC (or be removed).
> - Avoid vague words: “fast”, “robust”, “user-friendly”, “as needed”.
> - If you can’t test it, it’s not an AC.

---

## 7) Data & Contract
### 7.1 Entities / Fields
Define what data exists and how it behaves.
- Field names, types, constraints (min/max/format), defaults
- Example payloads (request/response) if relevant

### 7.2 Invariants
Rules that must always hold (great for tests).

### 7.3 Compatibility / Migration
If this change impacts clients or persisted data:
- What breaks?
- Can old and new coexist?
- Migration strategy (if any)

---

## 8) Failure Analysis (Sad Paths)
For each category, define:
- Trigger condition
- Expected behavior
- User-visible outcome (UI)
- Retry behavior (if any)
- Logging/metrics requirement (if any)

### 8.1 Network
- Timeout
- Partial failure
- Retry/backoff rules (or “no retry”)

### 8.2 Concurrency
- Concurrent edits
- Race conditions
- Locking/last-write-wins/optimistic concurrency expectations

### 8.3 Data validation
- Null/malformed fields
- Out-of-range values
- Missing required data

### 8.4 Permissions / Auth
- Unauthorized
- Forbidden (role mismatch)
- Token expired/session invalid

### 8.5 External dependency failure (if any)
- Upstream service unavailable
- Rate limits

> Requirement:
> - Each sad path must map back to at least one AC (`AC-xxx`) or you need to add an AC.

---

## 9) Error Contract (backend ↔ UI expectations)
Define at the level necessary for consistent UI and tests:
- Stable error code(s) and classification
- Which errors are retryable
- Minimum fields in error responses

Example (shape only; adapt to your project):
```json
{
  "error": {
    "code": "RESOURCE_CONFLICT",
    "message": "Human-readable (safe)",
    "retryable": false
  }
}
````

> UI rule:
>
> * UI must not show raw internal errors to users.
> * UI maps codes → curated messages and recovery actions.

---

## 10) Test Plan (draft)

Map ACs to tests (Quinn will refine, but you must provide the first pass).

* Unit tests: ...
* Integration tests: ...
* Contract tests: ...
* Performance tests (if relevant): ...

Suggested mapping format:

* AC-001 → TEST-xxx (unit + integration)
* AC-002 → TEST-yyy (contract + snapshot)
* ...

---

## 11) Open Questions

List unresolved decisions as explicit questions.
Each question must have an owner and a due phase (Spec / ADR / Implementation).

---

# Writing Rules (Vera-style constraints)

* **No ambiguity**: replace fuzzy language with thresholds, conditions, or examples.
* **Define terms once**: add a mini-glossary if needed.
* **Prefer examples**: one example payload beats 10 sentences.
* **Be explicit on defaults**: especially for optional fields and missing data.
* **Document trade-offs**: if you choose “simple” over “perfect”, say so.

---

# Handoff to other agents (how this spec gets used)

## To Silas (Architecture / ADR)

* Identify which ACs require architectural decisions.
* Provide constraints: latency/load/security assumptions.

## To Torin (Backend)

* The spec is the contract: request/response behavior, error codes, retryability.

## To Lyra (UI)

* Provide UI states required (loading/disabled/error) and error mappings by code.

## To Quinn (QA)

* AC list is the test source of truth.
* Sad paths must have tests or explicit justification.

## To Viktor (Security)

* Any auth/PII/secrets/deps changes must create/update SEC-xxx.

---

# Checklist before marking "Approved"

* [ ] Every requirement is covered by at least one `AC-xxx`
* [ ] Sad paths are defined (network + concurrency + data + auth at minimum)
* [ ] Data constraints and defaults are explicit
* [ ] Error expectations are defined enough for UI/test mapping
* [ ] Test plan draft exists (AC → tests)
* [ ] Compatibility/migration note included when contracts/data change
