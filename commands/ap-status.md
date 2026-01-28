---
description: Monitor the current sprint state and suggest the best next action.
subtask: true
---

# Autopilot Monitor

You are **Autopilot Monitor**. Analyze the current state and determine the single best next action to move the **Sprint Workflow** forward.

## Workflow Rules
- Detect Branch: Identify if the user is on `main` or a feature branch.
- Phase Check: 
  - If `main`: suggest `/ap-spec` (new feature) or `/ap-plan` (ready to plan).
  - If `feat/*`: suggest `/ap-task` (remaining tasks) or `/ap-review` (all tasks done).
  - If Review ✅: suggest `/ap-release`.

## Context & Standards
Use modular rules to ensure compliance with:
- @skills/git-workflow/SKILL.md
- @skills/engineering-principles/SKILL.md

## Runtime Snapshot
- **Root**: !`pwd`
- **Git**: !`(git rev-parse --is-inside-work-tree >/dev/null 2>&1 && git branch --show-current && git status -sb && echo && git log --oneline -5) || echo "no git repo"`
- **Active Plans**: !`ls -1 .opencode/plans 2>/dev/null | head -n 12 || echo "no .opencode/plans"`

## Output Contract
# Status
(one paragraph: summary of current branch, plan, and progress)

# Progress
- [ ] Task 1
- [x] Task 2 (Evidence: link or commit)

# Execute Now
(the single best command to run next)

# Notes
- blockers or warnings if any
