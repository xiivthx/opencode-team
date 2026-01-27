---
description: Write/update a spec using Vera’s template + sad paths, with contract IDs and test plan.
subtask: true
---

## Skills
- spec-writing
- team-contract-ids
- engineering-principles
- (if service): twelve-factor-service
- (if architecture change): hex-architecture + adr-writing

Input: $ARGUMENTS

## Snapshot (read-only)
Root: !`pwd`
Existing specs: !`ls -1 docs/specs 2>/dev/null | tail -n 30 || echo "no docs/specs"`
Existing ADRs: !`ls -1 docs/adr 2>/dev/null | tail -n 30 || echo "no docs/adr"`

## Output requirement: write a spec file
Create/Update: `docs/specs/<AC-ID>-<slug>.md`

### If no AC-ID is provided
- Ask ONE question: “What is the AC-ID for this work?”
- If user refuses/unknown: create `docs/specs/AC-TBD-<slug>.md` and clearly mark **RENAME REQUIRED BEFORE MERGE**.

## Spec format (must include all headings)
# AC-ID
# Title
# Summary
# Problem statement
# Goals (Acceptance Criteria)
# Non-goals
# User stories (optional)
# Design overview
# API/Contract changes (if any)
# Error codes & messages (backend + UI mapping)
# Data model / storage (if any)
# Observability (logs/tracing/metrics)
# Security & privacy considerations
# Failure scenarios (Sad paths)
# Test plan (unit/integration/contract/perf where applicable)
# Rollout / migration / compatibility (if relevant)
# Open questions

## Hard gates
- If you specify new error codes: reference `rust-backend-standards` and ensure they are stable + mappable to UX.
- If concurrency/async risks: explicitly list invariants + race handling.
- If introducing a dependency: add a `SEC-*` follow-up and reference `security-supply-chain`.

## Output contract (in chat)
- Spec file path
- 3-7 bullet “What changed / what is decided”
- “Next command” one-liner:
  - `/ap-adr ...` if architecture changes
  - else `/ap-plan <AC-ID> ...`
