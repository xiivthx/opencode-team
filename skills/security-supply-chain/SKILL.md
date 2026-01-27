---
name: security-supply-chain
description: Supply-chain security playbook for Rust dependencies: vet new crates and updates, run RustSec audits, enforce license/source/duplicate policies, and record SEC-xxx decisions with severity gates. Use when adding/updating deps, reviewing PRs, or configuring CI security checks.
compatibility: opencode
metadata:
  owner: viktor
  domain: security
  freedom: low
---

# Security Supply Chain (Viktor)

This skill defines the **dependency trust and supply-chain policy** for Rust projects:
- approving new crates
- updating versions
- enforcing advisories/licenses/sources policies
- recording decisions as `SEC-xxx`
- blocking deploy on critical issues

## When to use me
Use this skill when:
- Adding a new dependency (direct or workspace-level)
- Updating `Cargo.lock` / dependency versions
- Introducing crates that touch: crypto, auth, parsing, serialization, networking, build scripts
- Setting up CI security gates
- Responding to advisories or suspicious dependency behavior

## Related skills (routing)
- For dependency update PR conventions (scope, commit/PR labeling): use skill({ name: "git-workflow" }).
- For CI wiring of security gates: use skill({ name: "ci-golden-path" }).

## Outputs you must produce
- A `SEC-xxx` entry (in PR notes, `docs/security/`, or traceability table)
- Clear verdict: **ALLOW / ALLOW WITH CONDITIONS / BLOCK**
- If allow: list required follow-ups (tests, gates, deny/vet config changes)
- If block: minimum checklist to unblock

---

# 1) Policy: minimize risk without freezing progress
The goal is not “zero dependencies”. The goal is:
- fewer surprises
- auditable third-party code
- consistent tooling and gates

Keep policies strict where it matters (security-sensitive crates),
and pragmatic for low-risk crates (formatting, small utilities).

---

# 2) Intake checklist for NEW crates (direct deps)
For any new crate, collect:

## 2.1 Project hygiene
- What does it do? Why is it needed (vs standard library / existing crates)?
- Is there a smaller alternative already in use?
- Does it pull in heavy transitive deps? (check `cargo tree -d`)

## 2.2 Maintainer & activity signal
- Is it maintained? (recent commits/releases, issue responsiveness)
- Is it published by a reputable org/maintainer or “drive-by”?
- Is it a squatted name / suspicious similarity to popular crates?

## 2.3 Risk surface
Mark as **High Risk** if any:
- crypto / auth / TLS / credential handling
- parsers for untrusted input
- networking stacks
- build scripts (`build.rs`) / proc macros
- `unsafe` heavy code

High Risk crates require deeper review and stronger gates.

## 2.4 License + source constraints
- License must be compatible with the project policy.
- Source must be acceptable (crates.io preferred; restrict git/path sources unless justified).

---

# 3) Required tooling checks (Rust standard stack)
## 3.1 Known vulnerabilities: RustSec (cargo-audit)
Run vulnerability scanning against `Cargo.lock`:
- `cargo audit`

If any advisories are found:
- create/update `SEC-xxx`
- decide: upgrade, patch, remove, or accept risk (rare; must be explicit)

## 3.2 Policy enforcement: cargo-deny
Use cargo-deny to enforce:
- advisories
- banned crates / duplicate versions
- licenses
- sources (e.g., ban arbitrary git sources)

Suggested checks:
- `cargo deny check advisories`
- `cargo deny check bans`
- `cargo deny check licenses`
- `cargo deny check sources`

Maintain policy in `deny.toml` at repo root.

## 3.3 Auditing trust: cargo-vet (when enabled)
Use cargo-vet for “trusted-audit coverage”:
- `cargo vet`

If gaps exist:
- decide whether to:
  - add a local audit entry
  - rely on a trusted importer / shared audit
  - block until audited for High Risk deps

> Team guideline: do not make vet requirements so onerous that updates stop.
> Prefer incremental audits focused on the highest-risk crates first.

---

# 4) Severity rubric (BLOCK rules)
Record severity in `SEC-xxx`:

## Critical (BLOCK DEPLOY)
Examples:
- RCE / credential leakage / signing key compromise risk
- malicious or obviously compromised dependency
- advisory with active exploitation or direct high-impact path
Action:
- **BLOCK DEPLOY**
- remove/replace dependency or pin to safe version
- add CI gate to prevent reintroduction

## High (FIX BEFORE MERGE)
Examples:
- known vulnerability with realistic exploit in our context
- supply-chain risk in high-risk area (crypto/parser/build scripts)
Action:
- fix or replace before merge
- if temporary allow, must be timeboxed and documented

## Medium / Low (TRACKED)
Examples:
- non-exploitable advisory in our context
- license edge-case with mitigation
Action:
- track with issue, document justification and monitoring plan

---

# 5) “Allow with conditions” patterns
Approve a dependency with explicit conditions, e.g.:
- “Allow only if `cargo deny` policies updated”
- “Allow only with feature flags disabling risky defaults”
- “Allow only with additional tests / fuzzing / benchmarks”
- “Allow only if used behind a narrow wrapper module”

---

# 6) CI integration (security gates)
Coordinate with CI owner (Kai) to enforce:
- `cargo audit` on every PR touching Cargo.toml/Cargo.lock
- `cargo deny check` per policy
- `cargo vet` (if enabled) on dependency changes

Rules:
- Any dependency change PR must run security gates.
- High-risk crates require explicit Viktor review (SEC-xxx).

---

# 7) Security decision template (SEC-xxx)
Copy/paste into PR description or `docs/security/SEC-xxx.md`:

## SEC-XXX: <Title>
**Change:** Add/Update crate `<name>` `<old→new>`  
**Why:** <business/technical justification>  
**Risk:** Low/Med/High/Critical  
**Surface:** (crypto/auth/parser/net/build/proc-macro/unsafe)  
**Checks:**
- cargo-audit: <pass/fail + notes>
- cargo-deny: <pass/fail + notes>
- cargo-vet: <covered/gap + notes> (if enabled)
**Verdict:** ALLOW / ALLOW WITH CONDITIONS / BLOCK  
**Conditions / Follow-ups:** <list>  
**Owner:** Viktor (or delegate)  
**Review date:** <date>

---

# 8) Rules of engagement (team)
- Do not “sneak in” deps via transitive changes without review.
- If a dependency update is large, split it:
  - tooling-only updates
  - runtime deps updates
  - high-risk deps updates
- If something looks suspicious, block first and investigate second.
