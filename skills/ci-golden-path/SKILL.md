---
name: ci-golden-path
description: Build a fast, reproducible “golden path” CI/CD where local dev matches CI via DevContainers, Rust toolchain is pinned, caches are correct, and required gates run (fmt/clippy/test/security). Use when creating or updating CI workflows, devcontainer config, or build/release pipelines.
compatibility: opencode
metadata:
  owner: kai
  domain: devops
  freedom: low
---

# CI Golden Path (Kai)

You are building a CI/CD pipeline that is:
1) **Reproducible** (local == CI)
2) **Fast** (aggressive caching, minimal rebuilds)
3) **Secure** (dependency + policy gates)
4) **Boring to operate** (predictable steps, clear failure modes)

## When to use me
Use this skill when:
- Creating or editing CI workflows (`.github/workflows/*`, `.gitlab-ci.yml`, etc.)
- Introducing DevContainers or fixing “works on my machine”
- Adding cross-compilation targets (server / wasm / embedded)
- Making required checks (quality/security gates) consistent across team
- Optimizing CI runtime with caches

## Related skills (routing)
- For what gates mean and when to add perf/contract tests: skill({ name: "qa-gates" }).
- For dependency security policy requirements: skill({ name: "security-supply-chain" }).
- For service operational principles that shape CI steps: skill({ name: "twelve-factor-service" }).
- For branching/PR conventions that integrate with CI: skill({ name: "git-workflow" }).

## Expected outputs
- A clear pipeline plan and/or config changes in:
  - `.devcontainer/devcontainer.json` (+ optional Dockerfile)
  - CI workflow file(s)
  - optional `Makefile`/`Justfile` “one-liner” commands
- A single “how to reproduce CI locally” instruction set

---

# 1) The Golden Rule: Local Parity
**Local dev and CI MUST use the same toolchain and OS-level deps.**

## 1.1 DevContainer is the source of truth
- Prefer putting the canonical environment in `.devcontainer/`.
- CI should use the same container image (or build from the same Dockerfile) when feasible.
- If CI cannot run inside devcontainer, pin versions so results match.

## 1.2 Pin everything that can drift
Minimum pins:
- Rust toolchain channel + version
- Targets (wasm32/embedded)
- Node/tooling versions (if UI build exists)
- System deps (openssl, pkg-config, clang, etc.)

---

# 2) Standard CI stages (required order)
This ordering minimizes wasted compute and provides fast feedback.

## Stage A — Hygiene (fast fail)
- Format: `cargo fmt --check`
- Lint: `cargo clippy -- -D warnings`

## Stage B — Tests (correctness)
- `cargo test` (workspace-aware)
- If available and approved, prefer nextest for speed (but keep determinism).

## Stage C — Security gates (policy)
- Run dependency checks (coordinate with `security-supply-chain`):
  - `cargo audit` (RustSec advisories)
  - `cargo deny check` (licenses/bans/duplicates/sources)
  - `cargo vet` (if enabled)

## Stage D — Build artifacts (release)
- `cargo build --release`
- Produce artifacts once and reuse downstream (immutability)

## Stage E — Packaging (if applicable)
- Docker multi-stage build for backend binaries
- WASM/static packaging for UI
- Firmware artifacts for embedded targets

---

# 3) Caching strategy (Rust-heavy)
Rust compiles are slow; caching is mandatory.

## 3.1 Cache directories (typical)
- Cargo registry cache: `~/.cargo/registry`
- Cargo git deps: `~/.cargo/git`
- Build outputs: `target/` (or a profile-specific target dir)

## 3.2 Cache correctness rules
- Cache keys must include:
  - OS
  - Rust toolchain version
  - Lockfile hash (`Cargo.lock`)
  - Feature flags / target triple (if applicable)
- Avoid cache poisoning across incompatible targets (separate cache per target triple if needed).

## 3.3 Optional acceleration
- Use sccache (or equivalent) if CI runners are stable and benefit is clear.
- Keep it optional: correctness first.

---

# 4) Cross-compilation matrix (IoT + Web + Cloud)
When the repo supports multiple outputs, build in a matrix:

- **Cloud/server**: `x86_64-unknown-linux-gnu`
- **Web UI**: `wasm32-unknown-unknown`
- **Firmware**: e.g. `thumbv7em-none-eabihf` (project-specific)

Rules:
- Build matrix jobs must share caches safely (per target).
- Fail fast: do not let optional targets hide failures.

---

# 5) DevContainer recipe (baseline)
Goal: a devcontainer that can run the same commands as CI.

## 5.1 Required components
- Rust toolchain (pinned)
- Required system libs
- Optional: Node (if UI build exists)
- Optional: embedded toolchain (probe-rs / llvm tools) behind a feature flag

## 5.2 devcontainer.json guidelines
- Prefer a stable base image (or Dockerfile) and keep it deterministic.
- Document what the container provides and which commands should succeed:
  - `cargo fmt --check`
  - `cargo clippy -- -D warnings`
  - `cargo test`
  - security checks (if installed)

## 5.3 “One command to match CI”
Add a project-level entrypoint:
- `just ci` OR `make ci` that runs the same stage sequence.

Example (conceptual):
- `make ci`: fmt → clippy → test → security → build

---

# 6) Required checks and ownership
This pipeline is cross-team contract enforcement.

- Quinn owns the QA gates behavior (determinism, contract tests, perf policy).
- Viktor owns supply-chain policy and “BLOCK DEPLOY” decisions.
- Kai owns CI mechanics, caching, and local parity.
- Torin/Lyra/Alex must be able to reproduce failures locally in devcontainer.

---

# 7) Failure triage (playbook)
When CI fails:
1) Identify which stage failed (A/B/C/D/E).
2) Reproduce inside DevContainer using the same command.
3) If non-reproducible:
   - suspect cache key mismatch
   - suspect toolchain drift
   - suspect missing OS dependency
4) Fix root cause (do not “rerun until green”).

---

# 8) Minimal GitHub Actions example (adapt)
Use this as a starting point; customize targets and security tools per repo.

```yaml
name: ci

on:
  push:
  pull_request:

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Toolchain install (pin version in your repo policy)
      - name: Install Rust
        uses: dtolnay/rust-toolchain@stable

      # Cache (ensure you use a supported cache action version)
      - name: Cache
        uses: actions/cache@v4
        with:
          path: |
            ~/.cargo/registry
            ~/.cargo/git
            target
          key: ${{ runner.os }}-rust-${{ hashFiles('**/Cargo.lock') }}

      - name: fmt
        run: cargo fmt --check

      - name: clippy
        run: cargo clippy -- -D warnings

      - name: test
        run: cargo test

      # Optional security gates
      # - name: audit
      #   run: cargo audit
      # - name: deny
      #   run: cargo deny check
````

---

# Checklist (Kai sign-off)

* [ ] DevContainer exists and is the canonical environment
* [ ] Rust toolchain is pinned and matches CI
* [ ] Cache keys include lockfile + toolchain + (targets if relevant)
* [ ] Required gates run in order: fmt → clippy → test → security → build
* [ ] Cross-target builds are explicit (server/wasm/firmware as needed)
* [ ] Clear local reproduction instructions exist (prefer `make ci` / `just ci`)
* [ ] No secrets in repo; use CI secrets manager / env injection
