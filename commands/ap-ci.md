---
description: CI golden path + devcontainer parity. Updates CI safely and documents how to run checks.
subtask: true
---

## Skills
- ci-golden-path
- qa-gates
- security-supply-chain
- twelve-factor-service (for deploy/runtime concerns)

Intent: $ARGUMENTS

## Safety rules
- Prefer additive changes.
- Do not introduce secrets into workflows.
- If changing CI that could break builds: explain rollback.

## Snapshot
Root: !`pwd`
Git: !`(git rev-parse --is-inside-work-tree >/dev/null 2>&1 && git status -sb) || echo "no git repo"`
CI hints: !`(ls -1 .github/workflows 2>/dev/null | sed -n '1,80p') || echo "no .github/workflows"`
Devcontainer hints: !`(find .devcontainer -maxdepth 2 -type f 2>/dev/null | sed -n '1,80p') || echo "no .devcontainer"`

## Tasks
1) Detect stack and existing CI conventions (do not replace blindly).
2) Enforce parity:
   - Document exact local commands in AGENTS.md
   - Ensure devcontainer toolchain matches CI toolchain (Rust version, node version, etc.)
3) Add or improve:
   - format/lint stage
   - test stage
   - caching (where safe)
   - security scan hooks (as policy, do not force tools not installed)

## Output
- ✅ Files changed
- 🧪 Verification commands (local + CI expectations)
- ⚠️ Risks
- Next command (one line)
