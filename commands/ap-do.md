---
description: Execute a plan end-to-end with TDD, contract gates, and clean git history (skill-aware).
subtask: false
---

## Skills
- git-workflow
- tdd-playbook
- qa-gates
- engineering-principles
And conditionally:
- rust-backend-standards (backend)
- rust-ui-patterns + tailwind-design-system (UI)
- security-supply-chain (deps)
- ci-golden-path (CI/devcontainer)
- firmware-testing + hardware human-in-loop (firmware/hardware)

Input: $ARGUMENTS
- If it’s a plan file path: read it fully and execute.
- Otherwise: create a minimal plan inline (same structure as /ap-plan) BEFORE editing.

## Hard safety rules (absolute)
- Never force-push. Never push to remote unless user explicitly asks.
- Never run destructive commands (rm -rf, dropping DB, migrations against prod, deploys) without explicit confirmation.
- Never modify secrets files (.env, creds) unless user explicitly requests AND you redact sensitive output.
- Never claim “done” without showing evidence (tests/checks output or explicit statement you could not run them).

## Snapshot
Root: !`pwd`
Git: !`(git rev-parse --is-inside-work-tree >/dev/null 2>&1 && git status -sb && echo && git log --oneline -5) || echo "no git repo"`
Changed files: !`(git rev-parse --is-inside-work-tree >/dev/null 2>&1 && git diff --name-only) || true`

## 0) Pre-flight
1) Identify stack (Rust/Node/Python/etc). Prefer repo evidence over guessing.
2) If git exists:
   - Ensure you are NOT on main/master.
   - Create a branch that follows git-workflow (include PRIMARY-ID).
3) Identify the “fast check” command (lint/test/format). Run it early if possible.

## 1) Execution loop (per task)
For each task:
1) Search/read existing code first. Do not invent file paths.
2) TDD loop:
   - Write/adjust test (red)
   - Implement minimal change (green)
   - Refactor (clean)
3) Run the task’s listed commands; if missing, run the smallest relevant fast check.
4) If a gate fails, stop and fix immediately (no “move on with red tests”).

## 2) Multi-agent delegation (when it reduces risk)
Delegate *only* when needed and with tight instructions:
- Backend-heavy work -> Torin
- UI patterns/states -> Lyra (and Luna if design tokens/states)
- CI/devcontainer -> Kai
- Tests strategy -> Quinn
- Security/deps/auth -> Viktor
- Architecture conflicts -> Silas
- Spec ambiguity -> Vera

If a subagent disagrees, resolve via evidence:
- ask for a minimal reproduction / spike
- do not decide by opinion

## 3) Git hygiene
- Keep commits 1–3 max.
- Commit messages must include PRIMARY-ID and follow git-workflow.
- Working tree should be clean at end (or explain why not).

## 4) Done definition (do not skip)
You may only say “DONE” if:
- Tests/checks passed OR you explicitly state what you could not run and why.
- Contract changes documented (spec/ADR updated if required).
- No new deps without security review notes (SEC gate).

## Final report (mandatory)
- ✅ What changed
- 🧾 Files touched (added/modified)
- 🧪 Commands run + outcomes
- 🧠 Key decisions + why (short)
- 🛡️ Security/deps notes (if any)
- 🚧 Remaining risks / TODOs (if any)
- Execute now (one line): usually `/ap-review` or `/ap-status`. **You must run this command.**
