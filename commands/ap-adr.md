---
description: Write/update an ADR using Silas template, including migration + compatibility plan.
subtask: true
---

## Skills
- adr-writing
- engineering-principles
- hex-architecture (when boundaries matter)
- twelve-factor-service (for deploy/ops shape)
- security-supply-chain (if deps/crypto/auth touched)

Input: $ARGUMENTS

## Snapshot (read-only)
Root: !`pwd`
Existing ADRs: !`ls -1 docs/adr 2>/dev/null | tail -n 30 || echo "no docs/adr"`
Git (optional): !`(git rev-parse --is-inside-work-tree >/dev/null 2>&1 && git log --oneline -10) || true`

## Output requirement: write an ADR file
Create/Update: `docs/adr/<ADR-ID>-<slug>.md`

### If no ADR-ID is provided
- Ask ONE question: “What ADR-ID should we use?”
- If unknown: create `docs/adr/ADR-TBD-<slug>.md` and mark **RENAME REQUIRED BEFORE MERGE**.

## ADR format (must include all headings)
# ADR-ID
# Title
# Status (Proposed/Accepted/Superseded)
# Context
# Decision
# Options considered (with pros/cons)
# Consequences (positive/negative)
# Migration plan
# Compatibility / versioning
# Operational impact (12-factor concerns)
# Security considerations
# Testing/verification plan
# Links (spec, PRs, docs)

## Hard gates
- If decision changes public contracts: explicitly list breaking changes + mitigation.
- If introducing new crates/services: create SEC follow-up and apply supply-chain rubric.

## Output contract (in chat)
- ADR file path
- “Decision in one sentence”
- “Next command” one-liner: usually `/ap-plan <ADR-ID> ...`
