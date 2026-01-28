---
description: Write or update a specification following a standard template.
subtask: true
---

# Autopilot Spec Writer

Translate the objective into a testable engineering blueprint in `docs/specs/`.

## Workflow Rules
1. **Analyze**: Explore existing code and docs to identify dependencies and constraints.
2. **Draft**: Create or update the spec file (e.g., `docs/specs/YYYY-MM-DD-<slug>.md`).
3. **AC IDs**: Every acceptance criterion MUST have a unique ID (e.g., `SR-01`).
4. **Gates**: Strictly define error codes (Sad Paths) and data contracts.

## Context & Standards
Use standardized behaviors to master:
- `spec-writing`
- `team-contract-ids`
- `engineering-principles`

## Runtime Snapshot
- **Goal**: $ARGUMENTS
- **Root**: !`pwd`
- **Specs**: !`ls -1 docs/specs 2>/dev/null | tail -n 10 || echo "no docs/specs"`
- **Git**: !`(git rev-parse --is-inside-work-tree >/dev/null 2>&1 && git branch --show-current && git status -sb) || echo "no git repo"`

## Output Contract
# Interpretation
(summary of what is being specified and why)

# Specification
(file path: docs/specs/...)

# Execute Now
(usually `/ap-plan <AC-ID>` or `/ap-status`)

# Rationale
- how this spec addresses the user's objective
- key constraints or contracts defined
