---
name: dte-arc-work
description: The full cycle for a task, bug, area, or a whole plan index — review, plan, do the work, then verify — with quality gates and progress reporting throughout. Composes dte-arc-review (or a light scan), dte-arc-plan, ce-work, ce-simplify-code/ponytail, and dte-deep-reviewer + dte-test-auditor to verify. Loops over a list of tasks/bugs OR over a dte-arc-plan INDEX worklist to complete many plans (one branch + gate + report each). Use when the user says "fix this", "do the work", "implement this plan", "work through these plans/tasks", or "handle this end to end". Stops for review after each item by default; --ship to auto commit/PR. --quick for a small bug, --deep for a real change.
---

# dte-arc-work — review → plan → work → verify (looping, gated, reported)

Read `../../references/conventions.md` first — especially **"Progress: report every phase"**,
**"Runnable gate"**, and **"Decomposition"**. This skill **changes code**. Default: **stop for the
user's review** after each item; only commit/PR if `--ship`.

## Input
- A single target (task/bug/area), **or** a list, **or** a `docs/plans/INDEX-<topic>.md` worklist from
  `dte-arc-plan` (the "complete many plans" path).
- Read the project `CLAUDE.md`/`AGENTS.md`; honor interactors-not-services + the project's test framework.

## Establish the gate FIRST (before any change)
Derive the project's runnable gate and confirm it's **green on the current tree**. Detect from: the
plan/INDEX's stated gate, CI config, `bin/rails`/`bin/rake` tasks. If none, use **test suite + a
boot/eager-load check** (e.g. `zeitwerk:check`, `CI=1`). This gate runs before **every** commit. A red
gate is **revert-the-phase**, never patch-around. The gate is the only real safety net — especially if
deploys are not test-gated.

## Cycle (per item / phase)
1. **Branch.** Meaningful name (`fix/...`, `feat/...`). Never the default branch.
2. **Review.** `--deep`: `dte-arc-review` on the affected slice. `--quick`: focused read + `/ce-debug` to
   the root cause (fix the cause, grep all callers first). _If the item already has a validated plan
   (INDEX/`docs/plans`), skip to step 4._
3. **Plan.** `dte-arc-plan` → a **validated** plan (ce-plan + ie-validate-plan; gaps resolved).
4. **Work.** `/ce-work` the plan. Follow existing patterns; write/update tests as you go.
5. **Elegant.** `/ce-simplify-code` + `ponytail` over the diff — the ladder, not just "less code".
6. **Verify.** `dte-deep-reviewer` on the diff + `dte-test-auditor` on changed code. Feed 🔴/🟠 back into
   step 4 (bounded: ≤2 fix passes, then surface what remains).
7. **Gate.** Run the project gate. Green → (commit + tick the worklist box + **report this phase**) or, if
   `--ship`, `/ce-commit-push-pr`. Red → **revert this phase, log why, stop.**
8. **Report + pause.** Print a phase report — what changed, verify verdict, gate result, branch, what's
   next — and **pause for go-ahead** (default cadence). Never advance silently.

## Completing MANY plans (an INDEX worklist)
Treat it like the proven loop: read `INDEX-<topic>.md` → take the **first unchecked, dependency-ready**
plan → run steps 1/4–8 for it (each plan already validated by `dte-arc-plan`) → on green, tick its box +
update the pointer + report + pause → next. One branch per plan unless the INDEX says otherwise. Maintain
an iteration log. **Serial by default** for a shared codebase (avoid index/merge races).

## Guardrails
- Stop-and-ask before anything irreversible/outward-facing (push, PR, deploy) unless `--ship`.
- High-severity findings still open after 2 passes → **not done**; surface them.
- Trace the whole problem before picking a fix; a small diff in the wrong place is a second bug.

## Done when
Each item/plan is implemented on a branch, simplified, **verified**, and **gate-green**, with the
worklist updated and a phase report given. Shipped only if `--ship`.
