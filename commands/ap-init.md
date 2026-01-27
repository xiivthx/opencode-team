---
description: Bootstrap repo for autonomous agents: ensure root AGENTS.md, project runbook, and safe defaults.
subtask: true
---

## Skills
- engineering-principles
- git-workflow
- tdd-playbook
- qa-gates
- security-supply-chain
- ci-golden-path
- team-contract-ids

## Non-negotiable safety rules
- Never delete data or run destructive commands without explicit user confirmation.
- Never add secrets to repo. Never print secrets.
- Prefer minimal, reversible changes.
- If unsure, write TODO + safe default rather than inventing behavior.

## Snapshot (ground truth)
Root: !`pwd`
Top-level: !`ls -la | head -n 120`
Git: !`(git rev-parse --is-inside-work-tree >/dev/null 2>&1 && git status -sb && echo && git log --oneline -10) || echo "no git repo"`
Repo map: !`find . -maxdepth 2 -type f -not -path '*/.git/*' -not -path '*/node_modules/*' -not -path '*/target/*' -not -path '*/dist/*' 2>/dev/null | sed -n '1,180p'`

## Tasks
### 1) Ensure root AGENTS.md exists (REQUIRED)
Create or update `AGENTS.md` at project root with:
- setup commands (install/build/run/test)
- formatting/lint commands
- branch + commit conventions (reference git-workflow)
- “definition of done”: tests + qa-gates + security checks
- where specs/ADRs/plans live (docs/specs, docs/adr, .opencode/plans)
- how to invoke autopilot commands (short)

Keep it short and actionable (max ~150 lines). No essays.

### 2) Create minimal structure (if missing)
- `docs/specs/`  (specs)
- `docs/adr/`    (ADRs)
- `.opencode/plans/` (execution plans)

Do NOT create extra meta-docs unless needed.

### 3) Add repo-safe defaults (only if missing)
- `.gitignore` (minimal, do not break existing style)
- `.pre-commit-config.yaml` (hooks) and !`prek install` (if missing)
- If a formatter/linter is already configured but undocumented, document it in AGENTS.md.

### 4) Output contract
End with:
- ✅ Files created/updated (1-line purpose each)
- 🧪 Copy/paste verification commands (what you actually ran OR what user should run)
- ⚠️ Unknowns / assumptions
- Execute now (exact one-liner, usually `/ap-spec ...` or `/ap-plan ...`). **You must run this command.**


### Example .pre-commit-config.yaml
```yaml
# Optimized for prek (https://github.com/j178/prek)
# prek is a fast, dependency-free Rust-based pre-commit alternative.

repos:
  # Standard hooks using prek builtins (Rust-native, zero-setup)
  - repo: builtin
    hooks:
      - id: check-merge-conflict
      - id: check-toml
      - id: check-yaml
      - id: end-of-file-fixer
      - id: trailing-whitespace
      - id: mixed-line-ending

  # Rust-specific hooks
  - repo: local
    hooks:
      # Format check (fast, runs first)
      - id: cargo-fmt
        name: cargo fmt
        description: Format Rust code
        entry: cargo fmt --all --
        language: system
        types: [rust]
        pass_filenames: false

      # Clippy lint (catches bugs before tests)
      - id: cargo-clippy
        name: cargo clippy
        description: Lint Rust code
        entry: cargo clippy --workspace --all-targets -- -D warnings
        language: system
        types: [rust]
        pass_filenames: false

      # Tests (slowest, runs last)
      - id: cargo-test
        name: cargo test
        description: Run tests
        entry: cargo test --workspace
        language: system
        types: [rust]
        pass_filenames: false
        stages: [pre-push]  # Only on push, not every commit

      # Check for security vulnerabilities (optional)
      - id: cargo-audit
        name: cargo audit
        description: Check for security vulnerabilities
        entry: cargo audit
        language: system
        types: [rust]
        pass_filenames: false
        stages: [pre-push]

      # Check Cargo.lock is up to date
      - id: cargo-check-lock
        name: cargo check lock
        description: Ensure Cargo.lock is up to date
        entry: cargo check --locked
        language: system
        types: [rust]
        pass_filenames: false

# Configuration
default_stages: [pre-commit]
fail_fast: true  # Stop on first failure
```