---
name: dte-arc-auditor
description: Full architectural audit of a whole Rails app (or a major subsystem) that produces not just findings but a complete, validated, loop-executable migration roadmap — current-state classification, a proposed target architecture, a phased plan with gates, a risks/decisions doc, and a PROGRESS worklist. This is the "re-architect this app" deliverable (e.g. service-junk-drawer → modular monolith / Packwerk / interactors). Use when the user says "audit the architecture", "do a full architectural audit", "re-architect / restructure the app", "plan a modular-monolith / Packwerk migration", or "assess this app and give me a migration plan". Composes dte-arc-review (current state) + a target-design step + dte-arc-plan (the validated roadmap). Not for a single slice's findings (use dte-arc-review), a PR/diff (use dte-deep-reviewer), or executing the roadmap (use dte-arc-work).
---

# dte-arc-auditor — app audit → target architecture → validated migration roadmap

Read `../../references/conventions.md` first — **all** of it applies (planning-is-mandatory-and-validated,
decomposition, verify-claims, report-every-phase, runnable-gate, surface-forks-early, ponytail counterweight).

This is the comprehensive assessment skill. It **produces a doc set + worklist**; it does **not** change
code (hand the worklist to `dte-arc-work`). It is built to do, properly, what a one-off bespoke workflow
does badly: it **composes** structured skills, **verifies** its own claims, **validates** the plan in a
separate pass, **surfaces the big forks up front**, and **keeps the honest "don't over-engineer" voice**.

## Input
- A whole app, or a major subsystem (e.g. `app/services`).
- Read the project `CLAUDE.md`/`AGENTS.md` first — owned conventions (interactors-not-services) drive the
  target design, not generic defaults.

## Procedure

1. **Current-state audit (compose, then verify).** Run `dte-arc-review` across the app to map structure
   and surface issues, and build a **component inventory** — every relevant file classified
   (command / pure-PORO / query / adapter / value-object / organizer) with "what it is" and "what it
   should become." **Verify the load-bearing facts with runnable checks** — counts, `require`/autoload
   usage, `zeitwerk:check`, "X already does Y", "zero of Z". Wrong classification here moves the wrong file.

2. **Surface the big forks EARLY (AskUserQuestion).** Before designing deep, put the decisions that change
   the whole shape to the user: granularity (e.g. 3 / 7 / 11 packs), adopt-a-gem-vs-keep-what-exists,
   how-much-to-convert. Do not discover these mid-roadmap.

3. **Design the target architecture.** Propose boundaries / packs / patterns that honor the owned
   conventions. Produce a **dependency DAG** and an explicit **do-not-convert / leave-alone list** (forcing
   pure logic into the new shape is the classic anti-pattern — name it). Keep what's already correct.

4. **Build the migration roadmap (via `dte-arc-plan`, validated).** Decompose into **phased, independently-
   shippable steps**, each: goal, concrete moves, a `stop_here_if` gate, risk. Establish the project's
   **runnable gate** (tests + boot/`zeitwerk:check` + any pack/lint validate) — the real safety net,
   especially if deploys aren't test-gated. Each phase/plan is **validated with `/ie-validate-plan`**
   (separate pass; gaps resolved). Emit a **`PROGRESS.md` worklist** (one checkbox per phase + the gate
   command) that `dte-arc-work` consumes.

5. **Risks & decisions — a separate adversarial pass.** Run an independent critic over the roadmap
   (different lens, ideally cross-model `/ce-code-review`), plus a **ponytail / `ie-simplicity-reviewer`
   counterweight**: is any of this over-engineering for the app's actual stage? Record the honest "healthy
   codebase, don't over-build" verdict if that's the truth. Capture the open decisions and the
   must-fix-before-start list.

6. **Write the doc set** under `docs/<topic>/` (e.g. `docs/refactor/`): `00-README` (orientation + headline
   verdict + how the loop runs), `01-current-state`, `02-target-architecture` (the DAG + do-not-convert),
   `03-migration-plan` (phased, gated), `04-risks-and-decisions`, `appendix-inventory`, and `PROGRESS.md`.
   Use the conventions output shape; mark **tools used / unavailable** and **verified vs assumed** claims.

7. **Report + hand off.** Summarize the verdict and the first phase, confirm the gate is green today, and
   **stop for the user's go-ahead** — execution is `dte-arc-work` on the `PROGRESS` worklist, not here.

## Done when
There's a validated doc set whose `PROGRESS.md` worklist `dte-arc-work` can execute phase-by-phase: a
target architecture with a real DAG + leave-alone list, a phased gated plan with each phase ie-validated,
the load-bearing claims **verified not assumed**, the big forks already decided with the user, and an
honest over-engineering check on the record.
