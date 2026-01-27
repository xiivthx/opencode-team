---
description: Produce a safe release plan: notes, version suggestion, checklist, rollback (no tagging/publishing).
subtask: true
---

## Skills
- twelve-factor-service
- engineering-principles
- qa-gates
- security-supply-chain

Release intent: $ARGUMENTS

## Snapshot
Root: !`pwd`
Git: !`(git rev-parse --is-inside-work-tree >/dev/null 2>&1 && git status -sb && echo && git log --oneline -25) || echo "no git repo"`
Diff summary: !`(git rev-parse --is-inside-work-tree >/dev/null 2>&1 && git diff --stat) || echo "no git diff"`

## Output contract
# Release notes (draft)
- Added
- Changed
- Fixed
- Security

# Versioning suggestion
- semver recommendation + rationale

# Release checklist
- Pre-merge checks (include qa-gates)
- Merge strategy
- Tagging/publishing steps (written, not executed)
- Rollback plan
- Post-release verification
