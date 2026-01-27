---
description: Produce an executable plan in .opencode/plans/ tied to contract IDs and team skills.
subtask: true
---

## Skills
- team-contract-ids
- engineering-principles
- tdd-playbook
- qa-gates
- git-workflow
- (as needed) rust-backend-standards, rust-ui-patterns, tailwind-design-system, security-supply-chain, ci-golden-path, firmware-testing

Goal / Input: $ARGUMENTS

## Snapshot (read-only)
Root: !`pwd`
Git: !`(git rev-parse --is-inside-work-tree >/dev/null 2>&1 && git status -sb && echo && git log --oneline -8) || echo "no git repo"`
Specs: !`ls -1 docs/specs 2>/dev/null | tail -n 15 || echo "no docs/specs"`
ADRs: !`ls -1 docs/adr 2>/dev/null | tail -n 15 || echo "no docs/adr"`

## Plan file rules
Create: `.opencode/plans/{YYYYMMDD}-{slug}--{PRIMARY-ID}.md`
- PRIMARY-ID should be one of: AC-*, ADR-*, API-*, SEC-* (prefer AC/ADR)
- If no ID is available: ask ONE question; else fallback to `ID-TBD`.

## Plan format (must follow exactly)
# Primary ID
# Goal
# Non-goals
# Inputs (spec/ADR links if any)
# Assumptions (each with confidence high/med/low)

# Skill routing (mandatory)
- Backend: rust-backend-standards
- UI: rust-ui-patterns + tailwind-design-system
- Testing: tdd-playbook + qa-gates
- CI/Dev env: ci-golden-path
- Dependencies: security-supply-chain
- Architecture changes: hex-architecture + adr-writing

# Task List (executable)
Use this checklist format:
- [ ] T01: <task title>
  - Owner: (Torin | Lyra | Luna | Kai | Alex | Quinn | Viktor | Silas | Vera | Elias)
  - Why:
  - Files:
  - Commands:
  - Acceptance:
  - Risks:

# Verification
- Fast checks (run every task)
- Full checks (run before declaring done)
- Security checks (if applicable)
- Perf checks (if applicable)

# Rollback
(what to revert, how to restore)

## Hard gates
- Each task MUST have Acceptance criteria that is testable.
- If task changes API/DTOs: add explicit contract test task for Quinn.
- If task adds deps: add SEC task for Viktor.

## Output contract (chat)
- Plan file path
- Next command (exact one line): `/ap-do <plan-file-path>`
