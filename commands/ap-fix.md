---
description: Reproduce, diagnose, patch, and add regression tests (TDD). Stops if unreproducible.
subtask: false
---

## Skills
- tdd-playbook
- qa-gates
- engineering-principles
- (backend) rust-backend-standards
- (UI) rust-ui-patterns
- (deps/security) security-supply-chain

Bug report / symptom: $ARGUMENTS

## Snapshot
Root: !`pwd`
Git: !`(git rev-parse --is-inside-work-tree >/dev/null 2>&1 && git status -sb && echo && git log --oneline -6) || echo "no git repo"`
Recent diff: !`(git rev-parse --is-inside-work-tree >/dev/null 2>&1 && git diff --stat && echo && git diff --unified=3 | head -n 220) || echo "no git diff"`

## Protocol
### 1) Reproduce-first (max 3 attempts)
- Find the smallest repro command/steps.
- If you cannot reproduce within 3 attempts:
  - list 2–3 hypotheses
  - list the exact evidence you need (logs, inputs, environment)
  - STOP (do not “fix by vibes”).

### 2) Diagnose
- Pinpoint root cause at code level.
- Prefer experiments over opinions.

### 3) Patch + proof
- Smallest fix that addresses root cause.
- Add/extend regression tests.
- Run fast checks then one broader check.

## Output (mandatory)
- ✅ Repro steps (before) + (after)
- 🧪 Tests/checks run + results
- 🧩 Root cause (2–4 sentences)
- 🔧 Patch summary + files changed
- 🛡️ Regression coverage (what it prevents)
