---
description: Self-contained autonomous sprint engine. Executes Spec -> Plan -> Task Loop -> Review -> Release. Supports no-argument discovery.
subtask: true
---

# Autopilot Magik Engine

Your mission is to drive the current objective to completion by executing the full sprint lifecycle autonomously.

## 🔄 The Magik Loop
1. **Analyze State**: If `$ARGUMENTS` is empty, scan `.opencode/plans/` and `docs/specs/` to find the most recent active goal or AC-ID.
2. **Execute Phase**: Based on the detected phase, execute the corresponding protocol below.
3. **Self-Trigger**: After every action, evaluate progress. If incomplete, loop back by self-triggering `/ap-magik <detected-goal>`.

---

### Phase 1: Discovery (Spec)
*Trigger: On `main` with a new goal (or goal found in docs).*
1. **Analyze**: Explore code/docs to identify dependencies and global constraints.
2. **Draft**: Create/update `docs/specs/YYYY-MM-DD-<slug>.md`.
3. **Acceptance**: Define unique **AC-IDs** (e.g., `AC-01`) and strict data contracts/error codes.

### Phase 2: Architecture (Plan)
*Trigger: Spec exists but no plan in `.opencode/plans/`.*
1. **Decompose**: Synchronize with specs/ADRs to create an atomic task list (T01, T02...).
2. **Draft**: Create `.opencode/plans/YYYY-MM-DD-<AC-ID>.md`.
3. **Context**: Each task MUST have a "Why", "Files", "Commands", and "Acceptance" criteria.

### Phase 3: Iteration (Task Loop) & Fix
*Trigger: Plan exists with pending tasks.*
1. **Prepare**: If T01, create a feature branch (`feat/AC-xx-slug`). Ensure you are NOT on `main`.
2. **TDD Loop (Worker)**:
   - **RED**: Write a failing test.
   - **GREEN**: Smallest implementation to pass.
   - **CLEAN**: Refactor and verify (build + test).
3. **Self-Correction (Fixer)**: If tests fail, diagnose root cause and patch with regression tests. STOP after 3 failed repro attempts.
4. **Commit**: Suggest `git commit -m "feat(scope): <desc> [AC-xxx]"` on success.

### Phase 4: Validation (Review)
*Trigger: All tasks in the active plan are [x] Done.*
1. **Audit**: Verify test evidence, security (no secrets/dangerous patterns), and contract adherence.
2. **Verdict**: ✅ ship / ⚠️ needs fixes.

### Phase 5: Finalization (Release)
*Trigger: Quality Gate gives a ✅ verdict.*
1. **Merge**: Switch to `main`, pull latest, merge feature branch using `--no-ff`.
2. **Verify**: Full build/test on `main`. 
3. **Safety**: If failure on `main`, immediately rollback to feature branch and trigger **Fix**.

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
