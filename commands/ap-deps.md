---
description: Add/update dependencies with supply-chain rubric + minimal risk, including tests and security notes.
subtask: true
---

## Skills
- security-supply-chain
- qa-gates
- engineering-principles
- (Rust) rust-backend-standards

Args: $ARGUMENTS
Expected: `<dependency> <reason>` (but handle missing gracefully)

## Hard rules
- No new dependency without justification.
- If dependency touches crypto/auth/parsing/network: treat as high-risk.
- Update lockfiles properly (do not hand-edit lockfiles).
- Run tests after change (or state why you could not).

## Snapshot
Root: !`pwd`
Git: !`(git rev-parse --is-inside-work-tree >/dev/null 2>&1 && git status -sb) || echo "no git repo"`
Rust manifests: !`(test -f Cargo.toml && echo "Cargo.toml:" && sed -n '1,120p' Cargo.toml) || echo "no Cargo.toml"`

## Procedure
1) Identify where deps are declared.
2) Apply Viktor rubric:
   - maintainer/reputation signals
   - maintenance/activity signals
   - transitive risk
   - MSRV impact (Rust)
3) Propose change + alternative (at least one).
4) Implement minimal safe update.
5) Run relevant checks/tests.
6) Record security notes (where your repo keeps them; if none, include in final report).

## Output
- Dependency change summary
- Justification (short)
- Tests run + outcome
- Security notes + follow-ups (SEC-* if needed)
