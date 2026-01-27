---
description: Current state, progress, blockers, and the single best next action.
subtask: true
---

## Snapshot
Root: !`pwd`
Git: !`(git rev-parse --is-inside-work-tree >/dev/null 2>&1 && git status -sb && echo && git log --oneline -12 && echo && git diff --stat) || echo "no git repo"`
Plans: !`ls -1 .opencode/plans 2>/dev/null | tail -n 20 || echo "no .opencode/plans"`
Specs: !`ls -1 docs/specs 2>/dev/null | tail -n 10 || echo "no docs/specs"`
ADRs: !`ls -1 docs/adr 2>/dev/null | tail -n 10 || echo "no docs/adr"`

## Output contract
# Status
(one paragraph)

## Progress
- bullets

## Blockers
- bullets (or "None")

## Next best action (choose ONE)
- what
- why
- exact command (one line)

## Notes
- optional bullets
