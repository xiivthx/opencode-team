---
description: Execute a single task (T01/T02/...) from a plan file with gates.
subtask: false
---

Args:
- plan file: $1
- task id: $2
Fallback:
- if only one arg: treat as task id and pick the most recent plan from .opencode/plans.

## Snapshot
Root: !`pwd`
Plans: !`ls -1 .opencode/plans 2>/dev/null | tail -n 20 || echo "no .opencode/plans"`
Git: !`(git rev-parse --is-inside-work-tree >/dev/null 2>&1 && git status -sb) || echo "no git repo"`

## Rules
- Apply: tdd-playbook, qa-gates, git-workflow, engineering-principles.
- Do NOT do other tasks unless required to satisfy acceptance criteria of this task.
- If blocked, stop and report the minimum info needed to proceed.

## Output
- Task id + status (done/blocked)
- Files changed
- Commands run
- Next suggested command (one line)
