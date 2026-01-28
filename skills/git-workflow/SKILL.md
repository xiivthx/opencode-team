---
name: git-workflow
description: Team Git standards for branch naming, commit traceability, and disciplined PR review workflows.
compatibility: opencode
metadata:
  owner: elias
  domain: workflow
  freedom: low
---

# Git Workflow (Team Standard)

This skill defines how the team uses Git: branching, commits, PRs, reviews, merges, conflicts, and traceability via reference IDs (`AC/ADR/API/TEST/SEC`).

## When to use
Use this skill when:
- starting new work (creating a branch)
- opening or reviewing a PR
- rewriting history (rebase / squash)
- resolving merge conflicts
- doing dependency updates / security fixes
- ensuring changes are traceable to spec/ADR/tests

## Related skills (routing)
- Reference IDs and mapping: skill({ name: "team-contract-ids" })
- TDD process: skill({ name: "tdd-playbook" })
- Quality gates: skill({ name: "qa-gates" })
- CI parity and pipeline: skill({ name: "ci-golden-path" })
- Supply-chain review: skill({ name: "security-supply-chain" })

---

# 1) Non-negotiables
1) Every PR must clearly state **why**, **what**, and reference at least **one ID**.
2) PRs must pass all relevant gates (fmt/clippy/test/security/perf as applicable).
3) Any behavior change requires tests, or an explicit N/A with justification.
4) Avoid multi-topic PRs that are hard to review. Split work.

---

# 2) Branch naming
## Standard format
`<type>/<short-desc>--<ID>`

Examples:
- `feat/login-refresh--AC-012`
- `fix/timeout-retry--API-007`
- `test/contract-errors--TEST-031`
- `chore/deps-update--SEC-014`
- `ci/cache-key-fix--AC-090`

## Common types
- `feat` feature
- `fix` bug fix
- `refactor` refactor (behavior unchanged or protected by tests)
- `test` tests only
- `docs` docs only
- `chore` maintenance / deps
- `ci` pipeline/devcontainer
- `sec` security-only changes (optional if you want separation from chore)

Rule: a branch should communicate **intent** and **traceability**.

---

# 3) PR contract
## 3.1 PRs must include reference IDs
PR description must include at least one:
- `AC-xxx` / `ADR-xxx` / `API-xxx` / `TEST-xxx` / `SEC-xxx`

If changing user-visible behavior or a public contract, include `AC` or `API` explicitly.

## 3.2 Recommended PR description template
**Summary**
- What changed and why

**References**
- AC-xxx / ADR-xxx / API-xxx / TEST-xxx / SEC-xxx

**Changes**
- Bullet list of meaningful changes

**Testing**
- Commands run + what they cover
- If something wasn’t run: explain why + risk

**Risk & Rollback**
- Primary risks
- Rollback plan / feature flag / revert plan

**Evidence** (as applicable)
- UI screenshots (loading/error states)
- Firmware logs or HiL report snippet

---

# 4) Commit hygiene
## 4.1 Suggested commit message format
`<type>: <imperative summary> [<ID>]`

Examples:
- `feat: add retry classification for timeouts [API-007]`
- `fix: prevent panic on malformed payload [AC-012]`
- `test: add contract test for RESOURCE_CONFLICT [TEST-031]`
- `chore: bump serde to x.y.z [SEC-014]`

## 4.2 Commit count
- Small changes: 1–3 commits is ideal (test → impl → refactor).
- Larger changes: split by “units of meaning” to improve review.
- Avoid long-lived “wip” noise; clean up before merge.

---

# 5) Merge policy
## Default: Squash merge
- One PR = one commit on main
- Keep history readable and bisect-friendly

## Exceptions (require justification)
- Migration sequences where commit separation matters
- Cases where merge commits are needed for tooling/release flow

Rule: optimize for future debugging, not short-term convenience.

---

# 6) Updating a branch (keeping up with main)
## Prefer rebase when:
- you want clean history
- you need to resolve conflicts without merge commits

## Prefer merging main into branch when:
- the team explicitly wants merge history for that PR, or
- your tooling requires it (document why)

Rules:
- Don’t rebase a branch others are actively using without coordination.
- If you must: announce it in the PR and coordinate.

---

# 7) Conflict resolution (systematic approach)
1) Identify file ownership and contract source of truth:
   - backend contracts/error codes → Torin
   - UI state/styling → Lyra + Luna
   - CI/devcontainer → Kai
   - security policy → Viktor
2) Resolve conflicts by honoring the contract:
   - `AC/ADR/API` are the source of truth
3) Run minimum validation:
   - fmt + clippy + tests (and security gates if relevant)
4) If results are flaky or non-reproducible: treat as a blocker.

---

# 8) Dependency updates
## 8.1 Small safe bump (patch/minor)
- One PR is fine
- Run tests + security checks

## 8.2 Large bumps / lockfile churn
Split into multiple PRs:
- tooling-only
- runtime deps
- high-risk deps (crypto/parsers/proc-macros/build.rs)

High-risk updates require `SEC-xxx` per `security-supply-chain`.

## 8.3 PR title guidance for deps
- `chore: deps update [SEC-xxx]`
- or explicitly note “no security impact” only if validated.

---

# 9) Hotfix / urgent fixes
- Use `fix/...--<ID>` or `sec/...--SEC-xxx`
- Keep PR minimal
- Must include tests or clear reproduction steps + rollback plan
- Follow with a hardening PR if needed (tests/cleanup)

---

# 10) Review standard (what reviewers check)
Reviewers should verify:
- PR summary is clear + includes IDs
- Behavior changes have tests (or justified N/A)
- Contract changes include `API-xxx` and contract tests
- CI/devcontainer parity is preserved (if relevant)
- Security gates pass for dependency/security-sensitive changes

Authors should:
- keep PRs small
- explain risks and rollback
- address feedback with changes (not debate-by-vibes)

---

# Pre-merge checklist
- [ ] PR includes at least one reference ID
- [ ] Scope is tight (not a multi-topic blob)
- [ ] fmt/clippy/tests pass (security/perf gates if applicable)
- [ ] Contract changes have `API-xxx` + contract tests
- [ ] Dependency changes follow `security-supply-chain` and include `SEC-xxx` when needed
- [ ] Risk + rollback/mitigation is documented for risky changes
