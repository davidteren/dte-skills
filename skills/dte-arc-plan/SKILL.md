---
name: dte-arc-plan
description: Turn architectural findings, a spec, a bug report, or an "improve area X" request into concrete, validated implementation plan(s) — always built with compound-engineering's ce-plan and validated with intent-engineering's ie-validate-plan. For a massive task it decomposes the work into many small, independently-shippable plans (loop), each ce-planned and validated, and emits a plan index that dte-arc-work then completes. Use when the user says "plan the fixes", "make a plan(s) from these findings", "plan this bug/refactor/feature", "break this big thing into plans", or hands over a dte-arc-review doc. Not for executing plans (use dte-arc-work) or producing findings (use dte-arc-review).
---

# dte-arc-plan — findings/spec/bug → validated plan(s)

Read `../../references/conventions.md` first — especially **"Planning is mandatory and validated"** and
**"Decomposition"**. This skill produces **plans**, not code.

## Two non-negotiables (ways of working)
1. **Plans are created with `/ce-plan`.** Never hand-roll a plan. ce-plan is the engine.
2. **Plans are validated before "done"** with `/ie-validate-plan` + any other applicable validators
   (cross-model `/ce-code-review` read of the plan, `layered-rails-planner` for architecture). **Gaps
   get folded back in.** A plan with open validation gaps is not ready.

## Input
- One of: a `dte-arc-review` findings doc, a written spec, a bug report, or a freeform "improve X".
- Read the project `CLAUDE.md`/`AGENTS.md` first (interactors-not-services, etc.).

## Single plan (a bug, a bounded change)
1. **Frame.** Restate problem + scope from the input. For a findings doc, carry its severity-ranked items
   forward as the requirements. **Verify** any load-bearing claim you'll plan around (don't trust it blind).
2. **`/ie-plan-assist`** — inject least-astonishment/convention/simplicity/UX considerations before drafting.
3. **`/ce-plan`** — draft the plan from the framed problem + the intent checklist. Architecture-shaped →
   also `layered-rails-planner`. Honor owned-convention (interactors, not service objects).
4. **Validate (separate pass).** `/ie-validate-plan` over the draft → predictability/convention/simplicity/UX
   ratings + gaps. Optionally a cross-model `/ce-code-review` read of the plan. **Resolve the gaps**, then finalize.
5. **Output** in `docs/plans/` (ce-plan owns the name); add a `→ plan: <path>` pointer atop the source doc.
   Right-size: a bug gets a light plan, not a phased epic (ponytail).

## Batch — a massive task → MANY plans (loop)
When the task is too big for one plan (a whole refactor, a multi-slice feature, a backlog):
1. **Decompose** into independently-shippable sub-areas — by slice / pack / concern / dependency. State the
   decomposition and the dependency order; surface the granularity fork to the user early (AskUserQuestion)
   if it materially changes the breakdown.
2. **Loop over the sub-areas** (parallel by default; `--serial` to slow down): for each, run the **Single
   plan** flow above (ce-plan → validate → resolve gaps → `docs/plans/`). Report after each (no dark loop).
3. **Emit the plan index** — `docs/plans/INDEX-<topic>.md`: every plan listed, **dependency-ordered**, one
   checkbox each, plus the project gate command. This is the worklist `dte-arc-work` consumes to **complete
   many**. Mirror the proven `PROGRESS.md` + gate shape.

## Degrade gracefully
- No `ce-plan` → draft the plan against the conventions output shape, **note it wasn't ce-plan-built**.
- No `ie-validate-plan` → fall back to `ie-plan-assist`; if neither, self-review against the section
  checklist and **flag that validation was not a separate pass** (a plan-then-validate gap, per conventions).

## Done when
Every input has a **validated** plan in `docs/plans/` an implementer can start from (validation gaps
resolved, not dangling). For a batch: a dependency-ordered `INDEX` worklist ready for `dte-arc-work`.
