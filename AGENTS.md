# Global Autopilot System

This document serves as the **Single Source of Truth** for the Autopilot squad. It defines the standards, safety protocols, and coordination logic for the entire repository.

## Modular Rules (Skills)
Standardized behaviors are loaded via the `skill({ name: "..." })` tool. Documentation for specific standards should be referenced by their skill name.

### Core Engineering
- `engineering-principles`: Code Quality & Mentality
- `git-workflow`: Branching/Commit Standards
- `tdd-playbook`: Red-Green-Refactor
- `qa-gates`: Verification Requirements

### Architecture & Domain
- `hex-architecture`: Backend Structure
- `rust-backend-standards`: Production Rust
- `rust-ui-patterns`: Performant UI
- `tailwind-design-system`: Atomic CSS

### Management & Security
- `team-contract-ids`: Reference & Traceability
- `adr-writing`: Decision Logs
- `security-supply-chain`: Safe Dependencies

## Sprint Workflow (Hard Gates)
Coordination follows a strict state machine defined in `commands/`.

1. **Spec (/ap-spec)**: MUST define unique AC-IDs (e.g., `AC-01`). Ambiguity results in a BLOCK.
2. **Plan (/ap-plan)**: MUST break work into atomic tasks (T01...). MUST link tasks to AC-IDs.
3. **Task (/ap-task)**: **TDD is non-negotiable**. Every change must correspond to a commit referencing an AC-ID. 
4. **Review (/ap-review)**: Reviewers MUST verify test evidence and security before a ✅ verdict.
5. **Release (/ap-release)**: Merge via `--no-ff`. Auto-rollback on any post-merge test failure.

## Coordination Protocols
- **task.md**: Represents the current ground truth. Agents MUST update it after every task completion.
- **AC-IDs**: All implementation work must be traceable to an Acceptance Criterion ID.
- **Delegation**: Primary agents (e.g., Elias) delegate atomic units of work to specialists (Subagents).

## Safety & Hygiene
- **Zero Secrets**: Never commit, print, or leak environment variables or keys.
- **Strict Git**: No direct commits to `main`. Use descriptive feature branches (`feat/AC-xx-slug`).
- **Non-Destructive**: Never delete project data or infrastructure without explicit user approval.
- **Evidence-First**: "It should work" is not a valid status update. Provide test output or build logs.
