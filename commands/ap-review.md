---
description: Strict review using team principles + skills. Produces a scored verdict with blocking issues.
subtask: true
---

## Skills
- engineering-principles
- qa-gates
- security-supply-chain
- git-workflow
And stack-specific:
- rust-backend-standards / rust-ui-patterns / tailwind-design-system / twelve-factor-service (as relevant)

## Context
Root: !`pwd`
Git: !`(git rev-parse --is-inside-work-tree >/dev/null 2>&1 && git status -sb && echo && git log --oneline -10) || echo "no git repo"`
Diff:
!`(git rev-parse --is-inside-work-tree >/dev/null 2>&1 && git diff --stat && echo && git diff --unified=3 | head -n 360) || echo "no diff"`

## Review gates (BLOCK if violated)
- No evidence of tests for changed behavior (when testable).
- New dependency without security justification.
- Contract/API changed without spec/ADR update.
- Nondeterministic tests introduced.
- Security-critical issue (authz/input validation/secret leakage).

## Scoring (0–2 each, total /18)
- Correctness
- Contracts & IDs (AC/ADR/API/SEC alignment)
- Tests (TDD + determinism)
- Security & privacy
- Performance (and benchmarks if hot path)
- Observability (logs/tracing)
- Readability
- Maintainability
- CI/Dev parity

## Output contract
- Overall verdict: ✅ ship / ⚠️ needs changes / 🛑 do not ship
- Score breakdown + total
- Top 5 issues (most important first)
- Minimal patch suggestions (concrete)
- Follow-up tests recommended
- Next command (one line): `/ap-do ...` or `/ap-fix ...` or `/ap-release ...`
