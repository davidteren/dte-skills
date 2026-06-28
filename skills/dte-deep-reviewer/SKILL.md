---
name: dte-deep-reviewer
description: Deep multi-lens review of a pull request, a branch diff, or a whole codebase, producing a written findings doc. Runs compound-engineering's cross-model code review, the intent-engineering lenses, majestic's Rails code-review orchestrator, and cubic review/scan, then adversarially cross-checks where they disagree. Use when the user says "review this PR / branch / diff", "deep review", "check this codebase", or hands over PR numbers/branch names. Loops over a list of PRs or branches. Not for architecture-level structure (use dte-arc-review) or for writing/fixing code (use dte-arc-work).
---

# dte-deep-reviewer — PR / branch / codebase review → findings doc

Read `../../references/conventions.md` first. Output is a findings doc; it does not change code.

## Input
- A target: a PR (number/URL), a branch (diff vs base), or a codebase/path.
- Or a **list** of PRs/branches → review each (parallel sub-agents; `--serial` to slow it down).
- Read the project `CLAUDE.md`/`AGENTS.md` first.

## Procedure (per target)
1. **Resolve & contextualize.** Get the diff (`gh pr diff`, `git diff <base>...<branch>`, or the path).
   Use Augment `codebase-retrieval` to pull the *surrounding* code the diff touches (callers, callbacks,
   tests) so review isn't diff-blind.
2. **Run the review lenses in parallel** (one sub-agent each; collect structured findings):
   - `/ce-code-review` — correctness, security, reliability, performance, testing personas **+ the
     cross-model second opinion** (the 3.15.0 win). Pass the plan/base when known.
   - `/ie-review` — predictability, convention, simplicity, experience, architecture lenses.
   - majestic `review:rails-code-review` — Rails-specific orchestrated review.
   - cubic `/run-review` + `/scan` (if available) — team-learned patterns + security scan.
   - **Front-end lens (conditional).** If the diff touches FE files (`.erb`/`.haml`/`.slim`, ViewComponent,
     JS/TS, Stimulus, CSS/Tailwind — see conventions "Front-end lens"), add **`ie-experience-reviewer`** +
     **frontend-design**/ui.sh (interaction states, a11y, keyboard/focus, look-and-feel). No FE files → say so, skip.
3. **Adversarial cross-check.** Compare the lenses: where two or more agree → **confirmed**; where they
   disagree on severity or existence → **disputed** (surface both, don't merge or average).
4. **Synthesize.** Dedupe, severity-rank (🔴/🟠/🟡), each finding with `file:line` + why + concrete fix.
   Apply owned-convention (interactors-not-services). Heuristic findings → "suspected"; only
   single-line-decidable ones get a hard claim.
5. **Write** `docs/reviews/deep-review-<target>-<date>.md` in the conventions shape, with the
   **Confirmed vs disputed** section and **tools used / unavailable**.

## For a list
One sub-agent per PR/branch → per-target docs + a roll-up `docs/reviews/deep-review-rollup-<date>.md`
ranking targets by risk and listing cross-PR themes.

## Done when
Each target has a findings doc: one-line verdict, confirmed findings with evidence + fixes, the disputed
section, and (for PRs) a ready-to-paste summary comment. Hand fixes to `dte-arc-plan` / `dte-arc-work`.
