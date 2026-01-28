---
description: Create or update an executable plan in .opencode/plans/.
subtask: true
---

# Autopilot Planner

Break down specifications into atomic, testable execution steps in `.opencode/plans/`.

## Workflow Rules
1. **Analyze**: Synchronize with existing Specs and ADRs to identify dependencies.
2. **Decompose**: Create a task list with unique IDs (e.g., `T01`).
3. **Draft**: Create the plan file (e.g., `.opencode/plans/YYYY-MM-DD-<AC-ID>.md`).
4. **Gates**: Each task must have clear "Why", "Files", "Commands", and "Acceptance" criteria.

## Context & Standards
Use standardized behaviors to master:
- `team-contract-ids`
- `engineering-principles`
- `qa-gates`
- `hex-architecture`

## Runtime Snapshot
- **AC ID**: $ARGUMENTS
- **Root**: !`pwd`
- **Plans**: !`ls -1 .opencode/plans 2>/dev/null | tail -n 12 || echo "no .opencode/plans"`
- **Git**: !`(git rev-parse --is-inside-work-tree >/dev/null 2>&1 && git branch --show-current && git status -sb) || echo "no git repo"`

## Output Contract
# Plan
(file path: .opencode/plans/...)

# Execute Now
(single command: `/ap-task <plan-file> T01`)

# Rationale
- why this task breakdown is optimal
- how it handles identified risks or dependencies
