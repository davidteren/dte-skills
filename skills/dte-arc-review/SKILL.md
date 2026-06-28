---
name: dte-arc-review
description: Architectural review of a Rails app, a subsystem, or a vertical slice, producing a written findings doc. Composes Augment context retrieval, layered-rails analyzers, intent-engineering audit, majestic Rails reviewers, and a ponytail over-engineering pass into one severity-ranked report. Use when the user asks to "review the architecture", "do an arc review", "assess the structure/design", "find architectural issues / god objects / fat models / layering problems", or to review a slice/subsystem. Loops over a list of slices when given several. Not for line-by-line PR/diff review (use dte-deep-reviewer) or for planning fixes (use dte-arc-plan).
---

# dte-arc-review — architectural review → findings doc

Read `../../references/conventions.md` first (tool detection, looping, output shape, owned-convention
rules). This skill is a **recipe**: it orchestrates existing tools and writes a findings doc. It does
not fix anything (hand off to `dte-arc-plan` / `dte-arc-work`).

## Input
- A target: an app root, a subsystem (e.g. `app/models/billing`), or a vertical slice (a feature).
- Or a **list** of targets → review each (parallel sub-agents by default; `--serial` to slow it down).
- Read the project's `CLAUDE.md`/`AGENTS.md` before judging — local conventions win.

## Procedure (per target)
1. **Scope, map & verify claims.** Map the slice with Augment `codebase-retrieval` (entry points, models,
   services, jobs, callbacks, dependencies; fallback cubic `codebase-context`/`wiki`, else Grep/Glob).
   **If reviewing against an existing plan/spec, run the cheap runnable checks that confirm its
   load-bearing factual claims** — file/class counts, `require` usage, boot/`zeitwerk:check`, "X already
   does Y", "zero of Z" — and **flag every mismatch**. Don't trust the analysis that wrote the plan; the
   classification error you catch here is the file the migration would otherwise move wrong.
2. **Run the lenses in parallel** (one sub-agent each; collect structured findings):
   - `/layered-rails` → `analyze-callbacks`, `analyze-gods`, `analyze-services` + `layered-rails-reviewer`
     (fat models/controllers, god objects, callback-heavy operations, layer leaks).
   - `/ie-audit` scoped to the target → architecture, convention, simplicity, predictability lenses.
   - majestic `pragmatic-rails-reviewer` + `data-integrity-reviewer` for Rails-specific structural smells.
   - **Front-end lens (conditional).** If the slice includes FE files (views, ViewComponent, JS/Stimulus,
     CSS/Tailwind — see conventions "Front-end lens"), add **`ie-experience-reviewer`** + **frontend-design**/
     ui.sh (component structure, interaction states, a11y, look-and-feel), or hand the slice to **`dte-ux`**.
     No FE files → state "no front-end changes detected", skip.
3. **Ponytail pass.** Run `ponytail`/`ie-simplicity-reviewer` over the findings: which "problems" are
   actually fine, and which proposed structure is over-engineering? Mark over-builds explicitly.
4. **Synthesize.** Dedupe across lenses, severity-rank (🔴/🟠/🟡), and apply owned-convention:
   for these projects recommend **interactors, not service objects**; flag any layered-rails `services`
   suggestion and rewrite it as an interactor recommendation.
5. **Write** `docs/reviews/arc-review-<target>-<date>.md` in the conventions output shape, including the
   **Confirmed vs disputed** section (where lenses disagreed — surface, don't merge) and **tools used /
   unavailable**.

## For a list of targets
Dispatch one sub-agent per target through steps 1–4, then write per-target docs **plus** a roll-up
`docs/reviews/arc-review-rollup-<date>.md` ranking the worst slices and the cross-cutting themes.

## Done when
Each target has a findings doc with a one-line verdict + health score, severity-ranked findings with
`file:line` evidence, the disputed section, and an explicit "what to fix next → `dte-arc-plan`" pointer.
