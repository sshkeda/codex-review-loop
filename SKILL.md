---
name: codex-review-loop
description: Collect Codex CLI review findings, then fix actionable issues and rerun the Codex review until no issues remain.
---
# Codex Review Loop
From the repo root, infer review target from prompt and context.

Collect findings only with the first-class Codex CLI review command:
- Current working tree: `codex review --uncommitted`
- Specific commit: `codex review --commit <sha>`
- Branch comparison: `codex review --base <branch>`
- Repo-wide review: `codex review "Check repository for correctness, security, maintainability, and tests."`

The diff-target flags (`--uncommitted`, `--commit`, `--base`) do **not** accept a
custom `[PROMPT]` — they are mutually exclusive. Only the bare repo-wide form
takes instructions. So you cannot feed Codex a per-run "ignore this" prompt; each
run re-reports the **full** finding set for the target. De-dup and durable
suppression therefore happen on your side, two ways:

Do not run any non-Codex reviewer. Do not spawn a subagent or ask another agent
to review; run the Codex CLI review command directly.

## 1. Session false-positive ledger (stops re-litigating across reruns)

Keep a running ledger of findings you have validated as NOT actionable this
session, each with a one-line reason. On every rerun Codex will re-report them —
recognize ledger items immediately and skip them; only investigate and act on
**new** findings. **Convergence = no new actionable findings** (every remaining
finding is on the ledger). Do not chase "zero findings"; a stable ledger is done.

## 2. Durable scope via AGENTS.md (stops Codex raising them at all)

Codex reads the repo's `AGENTS.md` as standing review context. For a *class* of
false positives that will recur (most commonly vendored/generated code), add a
short scope note to `AGENTS.md` — but only with the user's explicit OK, since
editing that file is governance. Recommended note:

> Treat agent-patches-vendored snapshots as read-only dependency code: paths
> under `third_party/`, or any directory matching an entry in
> `agent-patches.vendor.json` / a sibling `.agent-patches-vendor.json`. Review
> only how this repo *consumes* them, not their internal design — fixes to their
> internals belong upstream in the canonical repo, then `agent-patches vendor sync`.

## Loop

1. Run the review for the chosen target.
2. Independently validate each **new** finding (not on the ledger). Actionable =
   a real defect in first-party code this repo ships or runs. You are the editor —
   Codex only finds; never let it write the fix. Confirm it is not a false
   positive before touching anything.
3. Fix confirmed-actionable findings yourself, then re-verify each fix
   (typecheck / lint / test / run as appropriate).
4. When you judge a finding a false positive, add it to the ledger with the reason
   (e.g. "vendored CLI box path — the consumer uses @scope/sdk, unaffected").
5. Rerun. Stop when every remaining finding is on the ledger.

Never "fix" a finding by hand-editing agent-patches-vendored files — that breaks
the vendor lock (`agent-patches vendor check --locked` fails). Change the
canonical source and re-sync, or record it on the ledger when the consumer is
unaffected.
