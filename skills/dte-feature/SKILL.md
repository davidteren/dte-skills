---
name: dte-feature
description: Full-stack new feature spanning front-end AND back-end — plan, implement, and verify across architecture/back-end AND UI/UX/front-end, making sure BOTH are solid. Composes dte-arc-plan (validated plan), dte-arc-work (gated build), dte-ux + the ui.sh family for the front-end, and dte-deep-reviewer + ie-experience-reviewer to verify both layers. Use when the user says "build this feature", "ship this idea end to end", "add <feature> with a UI", or describes something that needs both a model/controller/job AND a screen. Loops over a list of features. Not for a back-end-only change (use dte-arc-work), pure UI (use dte-ux), or just defining the what (use dte-spec).
---

# dte-feature — full-stack feature: plan → build → verify, both layers solid

Read `../../references/conventions.md` first — **Planning is mandatory and validated**, **Runnable gate**,
**Progress: report every phase**. This skill **changes code on both sides of the stack**. It ensures the
back-end *and* the front-end each get a real plan, real build, and real verification — neither half is an
afterthought. Default: **stop for review after each phase**; `--ship` to auto commit/PR.

## Input
- A feature idea/spec (or a `dte-spec` output). If it's still fuzzy, run **`dte-spec`** first to get a
  validated spec, then come back.
- Or a **list** of features → loop (serial by default on a shared codebase).
- Read the project `CLAUDE.md`/`AGENTS.md`: interactors-not-services, test framework, design system.

## Establish the gate FIRST
Derive the runnable gate (tests + boot/eager-load check; add a JS/asset build + a Lighthouse/a11y check if
the feature ships UI) and confirm green on the current tree. It runs before every commit; red = revert.

## Cycle (per feature)
1. **Branch** — `feat/<slug>`.
2. **Plan BOTH layers.** Run **`dte-arc-plan`** → a validated plan that explicitly covers the back-end
   (models/controllers/jobs/interactors, data, authorization) **and** the front-end (screens, components,
   states, responsiveness, a11y). A plan missing either layer is not done — send it back.
3. **Build back-end.** `/ce-work` the back-end slice. Interactors not services; follow existing patterns;
   tests as you go.
4. **Build front-end.** Hand the UI slice to **`dte-ux --build`** (ui.sh family + frontend-design) — real
   interaction states (loading/empty/error), responsive, dark-mode-aware, accessible. Wire it to the
   back-end.
5. **Elegance.** `/ce-simplify-code` + `ponytail` over the whole diff.
6. **Verify BOTH.** `dte-deep-reviewer` on the diff (correctness/architecture) **and** **`dte-ux --review`**
   / `ie-experience-reviewer` on the UI (states, a11y, look-and-feel). Feed 🔴/🟠 back into 3–4 (≤2 passes,
   then surface what remains). `dte-test-auditor` on changed code.
7. **Gate.** Run the gate. Green → commit + **report the phase** (or `--ship` → `/ce-commit-push-pr`).
   Red → revert the phase, log why, stop.
8. **Report + pause** — what shipped on each layer, both verify verdicts, gate result, branch, what's next.

## For a list of features
One branch per feature; serial by default. Or hand the list to **`dte-loop`** to build a worklist + run it
autonomously. Report + pause per feature.

## Guardrails
- **Both layers or it's not done.** A feature with a working back-end and an untested/inaccessible UI (or
  vice-versa) fails the bar — say so, don't ship half.
- Stop-and-ask before push/PR/deploy unless `--ship`. High-sev findings open after 2 passes → not done.

## Done when
The feature is implemented on a branch with back-end **and** front-end built, simplified, **both verified**
(code review + UX/a11y review green-ish), and **gate-green**, with a phase report. Shipped only if `--ship`.
