---
name: team-contract-ids
description: Enforce cross-team reference IDs (AC/ADR/API/TEST/SEC) and a single mapping thread so Elias, Vera, Silas, Torin, Lyra, Quinn, Viktor stay in sync across specs, architecture, code, tests, and security notes.
compatibility: opencode
metadata:
  audience: squad
  domain: process
---

# Team Contract IDs

This skill defines the **reference ID system** used across the team to keep Specs (Vera), Architecture (Silas), Implementation (Torin/Lyra/Alex), QA (Quinn), Security (Viktor), and Tracking (Elias) aligned.

## What I do
- Define a **single ID scheme** for: `AC`, `ADR`, `API`, `TEST`, `SEC`
- Define where IDs must appear (spec/adr/pr/task.md/commit)
- Provide a **mapping format** so each requirement is traceable to code/tests/security notes
- Define “what to do when things change” (migration & renumbering rules)

## When to use me
Use this skill when:
- Creating a new feature spec or updating acceptance criteria (Vera’s output is scenario-driven) :contentReference[oaicite:2]{index=2}
- Writing or updating an ADR (Silas defines contracts/traits/types) :contentReference[oaicite:3]{index=3}
- Implementing/altering backend contracts or error handling (Torin must match Spec and avoid breaking API) :contentReference[oaicite:4]{index=4}
- Writing tests that must enforce the spec contract (Quinn is “Contract Enforcer”) :contentReference[oaicite:5]{index=5}
- Adding deps / security notes / gatekeeping decisions (Viktor blocks deploy on critical issues) :contentReference[oaicite:6]{index=6}
- Updating `task.md` as the single source of truth (Elias requires it) :contentReference[oaicite:7]{index=7}

## ID scheme (canonical)
Use **uppercase prefix + 3 digits**, zero-padded:

- `AC-001` … Acceptance Criteria / Gherkin scenarios (Vera)
- `ADR-001` … Architecture Decision Record (Silas)
- `API-001` … Public API contract changes (Torin; may include error codes)
- `TEST-001` … QA test plan item / implemented test reference (Quinn)
- `SEC-001` … Security finding / mitigation / policy decision (Viktor)

### Scope & uniqueness rules
- IDs are unique **within a project repo**.
- Never reuse an ID after it’s published (even if removed).
- Prefer monotonic increasing numbers; do not renumber old IDs.

## Where IDs MUST appear
### 1) Spec (Vera)
- Each Acceptance Criterion line MUST be labeled: `AC-xxx: ...`
- Each sad-path or edge case MUST reference the same AC number when possible. :contentReference[oaicite:8]{index=8}

### 2) ADR (Silas)
- ADR title includes `ADR-xxx`.
- ADR “Contract” section must cite related `AC-xxx` IDs it satisfies.
- If ADR introduces breaking contract change, it must mention the impacted `API-xxx`.

### 3) Backend/API changes (Torin)
- PR description MUST include at least one of: `AC-xxx`, `ADR-xxx`, `API-xxx`
- If response shape/status/error changes: create or update an `API-xxx` entry.

### 4) QA (Quinn)
- Each test (unit/integration/snapshot/bench) references:
  - the `AC-xxx` it validates, and
  - a `TEST-xxx` label for reporting coverage. :contentReference[oaicite:9]{index=9}

### 5) Security (Viktor)
- Findings are logged as `SEC-xxx`.
- Any “BLOCK DEPLOY” must reference at least one `SEC-xxx`. :contentReference[oaicite:10]{index=10}

### 6) Tracking (Elias / task.md)
- Each task item MUST include at least one reference ID (minimum: `AC-xxx`).
- If a task is implementation-only, it must link upstream (`ADR-xxx` or `AC-xxx`) to stay grounded. :contentReference[oaicite:11]{index=11}

## Git/PR linkage (lightweight)
- PR title or PR description MUST include at least one relevant reference ID (AC/ADR/API/TEST/SEC).
- Commit messages SHOULD include IDs for traceability when practical (especially for contract changes).
- For commit/branch/PR conventions, use skill({ name: "git-workflow" }).

## Mapping format (single source of traceability)
Maintain a mapping block in either:
- `docs/traceability.md` (recommended), or
- the relevant feature spec doc (if you prefer co-location)

Use this table format:

| AC | ADR | API | TEST | SEC | Notes |
|---|---|---|---|---|---|
| AC-001 | ADR-003 | API-002 | TEST-010 | SEC-004 | Retry behavior + error UX |
| AC-002 | ADR-003 | API-002 | TEST-011 |  | Concurrent edit conflict |

### Minimum requirement
For every `AC-xxx`, there must be at least:
- one `TEST-xxx` (or explicit “N/A with justification”),
- and if relevant, an `API-xxx`.

## Change management rules
### If an AC changes
- Keep the old ID; append “(superseded)” in notes rather than deleting history.
- Update mapping row; add a new `TEST-xxx` if behavior changed.

### If a contract breaks (types/errors/endpoints)
- Create or update `API-xxx`.
- Ensure Lyra imports shared types rather than duplicating; breaking changes must be explicit. :contentReference[oaicite:12]{index=12}

### If security posture changes
- Record a new `SEC-xxx` entry with severity and verification step.
- If critical: label **BLOCK DEPLOY** with the minimum unblock checklist. :contentReference[oaicite:13]{index=13}

## Working agreement (short)
- Vera owns `AC-*` labels. :contentReference[oaicite:14]{index=14}
- Silas owns `ADR-*` labels and contract sections. :contentReference[oaicite:15]{index=15}
- Torin owns `API-*` labels and guarantees “no break without explicit note.” :contentReference[oaicite:16]{index=16}
- Quinn owns `TEST-*` labels and enforces spec/contract. :contentReference[oaicite:17]{index=17}
- Viktor owns `SEC-*` labels and deploy gates. :contentReference[oaicite:18]{index=18}
- Elias ensures `task.md` remains the single execution ledger. :contentReference[oaicite:19]{index=19}
