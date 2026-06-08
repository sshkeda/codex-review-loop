---
name: codex-review-loop
description: Use when an agent should run the official Codex CLI `codex review` command and infer whether to review uncommitted changes, a branch diff, a commit, or the whole repo.
---
# Codex Review Loop
Use the official Codex CLI review command from the repo root.

Infer target from the prompt:
- WIP/current/my changes/staged/unstaged -> `codex review --uncommitted`
- SHA/commit/HEAD/last commit -> `codex review --commit <sha>`
- branch/PR/against main/trunk -> `codex review --base <base-branch>`
- whole repo/codebase -> `codex review "Review this repository for actionable correctness, security, maintainability, and test issues."`
If target is unclear, check `git status --short`, `git branch --show-current`, and `git log --oneline -5`.
Default to `--uncommitted` when the working tree has changes; otherwise ask one short clarifying question.
Pass any requested review lens as the optional prompt, e.g. `codex review --uncommitted "Focus on regressions and missing tests."`
Report findings first, ordered by severity, with file/line refs when available. State the exact command run.
