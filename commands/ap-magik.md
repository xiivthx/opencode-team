---
description: Self-contained autonomous sprint engine. Executes Spec -> Plan -> Task Loop -> Review -> Release. Supports no-argument discovery.
subtask: true
---

# Autopilot Magik Engine

Your mission is to drive the current objective to completion by executing the full sprint lifecycle autonomously.

## 🔄 The Magik Loop
1. **Analyze State**: If `$ARGUMENTS` is empty, scan for the most recent goal in `.opencode/plans/` and `docs/specs/`. 
2. **Proactive Discovery**: If the most recent goal is **[x] Done** or the plan is empty:
   - **SEARCH** the repository for `ROADMAP.md`, `BACKLOG.md`, `TODO.md`, or the "Future Work" section in `AGENTS.md` and `README.md`.
   - **IDENTIFY** the very next logical high-level feature or improvement.
   - **COMMENCE** a new **Phase 1 (Discovery)** for this new goal automatically. **NEVER ASK THE USER FOR A NEW GOAL**—draft the new spec and proceed.
3. **Autonomous Research**: If a technical decision or error requires user input, **SEARCH THE INTERNET** (via `search_web`) to identify best practices or known solutions. Choose the most appropriate path and proceed without asking the user.
4. **Execute Phase**: Based on the detected phase, execute the corresponding protocol below.
5. **Self-Trigger**: After every action, evaluate progress. If incomplete, loop back by self-triggering `/ap-magik <detected-goal>`.

---

### Phase 1: Discovery (Spec)
*Trigger: On `main` with a new goal.*
1. **Delegation**: Create a task for **Vera** (`agents/vera.md`) to analyze the repo and draft `docs/specs/YYYY-MM-DD-<slug>.md`.
2. **Acceptance**: Ensure Vera defines unique **AC-IDs** and strict data contracts.

### Phase 2: Architecture (Plan)
*Trigger: Spec exists but no plan in `.opencode/plans/`.*
1. **Delegation**: Create a task for **Silas** (`agents/silas.md`) to create an atomic task list (T01, T02...) in `.opencode/plans/`.
2. **Constraint**: Ensure each task links to an AC-ID and has clear "Why/Files/Acceptance" criteria.

### Phase 3: Iteration (Task Loop) & Fix
*Trigger: Plan exists with pending tasks.*
1. **Preparation**: Create a feature branch (`feat/AC-xx-slug`).
2. **Delegation**: Assign the task to the appropriate specialist:
   - **Backend**: Torin (`agents/torin.md`)
   - **Frontend**: Lyra (`agents/lyra.md`)
   - **Firmware**: Alex (`agents/alex.md`)
3. **Execution**: The specialist MUST follow the TDD loop (skill: `tdd-playbook`). 
4. **Self-Correction**: If tests fail, the specialist patches the code. If stuck, they escalate to Silas.

### Phase 4: Validation (Review)
*Trigger: All tasks in the active plan are [x] Done.*
1. **Delegation**: Assign the final audit to **Quinn** (`agents/quinn.md`) for QA and **Viktor** (`agents/viktor.md`) for Security.
2. **Verdict**: Only proceed to Phase 5 if both provide a ✅ verdict.

### Phase 5: Finalization (Release)
*Trigger: Quality Gate gives a ✅ verdict.*
1. **Action**: The Orchestrator (Elias) performs the merge to `main`.
2. **Verification**: Run final integration build/test on `main`. 
3. **Safety**: Rollback immediately on failure.

---

## 🛡️ Hard Safety Gates
- **STOP** if TDD fails 3 times or merge conflict occurs.
- **STOP** if a Security Critical issue (auth/leak) is detected.

## 📦 Runtime Snapshot
- **Goal (Arg)**: $ARGUMENTS
- **Root**: !`pwd`
- **Context**: !`(git rev-parse --is-inside-work-tree >/dev/null 2>&1 && git branch --show-current && git status -sb && echo && ls -R docs/specs .opencode/plans 2>/dev/null) || echo "init state"`

## 📋 Output Contract
# Sprint State: [Phase X: Name]
(Analysis: Goal detected from [source], phase progress, next step)

# Continuous Magik
/ap-magik <detected-goal>

# Rationale
- why the current action is the correct movement
- evidence of progress
