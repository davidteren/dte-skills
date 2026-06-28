---
name: dte-spec
description: Turn a fuzzy idea or goal into a right-sized, validated specification (PRD) — the WHAT and WHY, with scope, user stories, acceptance criteria, edge cases, and non-goals — before any planning. Composes ce-brainstorm (explore/scope the idea), ce-ideate, and ie-validate-plan (validate the spec for clarity, simplicity, predictability, UX completeness). Use when the user says "spec this out", "write a PRD", "define the requirements", "what should this do", "scope this feature", or hands a one-line idea that needs shaping. Loops over a list of ideas. Not for the HOW/implementation plan (use dte-arc-plan) or building it (use dte-feature/dte-arc-work).
---

# dte-spec — idea → validated spec/PRD (the WHAT, before the HOW)

Read `../../references/conventions.md` first (looping, output shape, **Surface key forks early**). This
skill produces a **specification**, not a plan. It defines *what* to build and *why* and *what done means*
— then **validates that definition** before it feeds `dte-arc-plan`. It composes the brainstorm/ideate +
validation tools; it does not invent its own requirements framework.

## Input
- An idea/goal — anything from one line to a rough brief.
- Or a **list** of ideas → spec each (parallel sub-agents by default; `--serial` to think in sequence).
- Read the project `CLAUDE.md`/`AGENTS.md` + existing features so the spec fits reality, not greenfield.

## Procedure (per idea)
1. **Explore & right-size.** Run **`ce-brainstorm`** to open the idea up — who it's for, the job it does,
   the smallest version that delivers value. Use **`ce-ideate`** if multiple approaches are worth weighing.
   **Surface the material forks early via AskUserQuestion** (scope, must-haves vs nice-to-haves, platforms)
   — don't bake assumptions silently.
2. **Write the spec.** `docs/specs/<idea>-<date>.md`, opening with the plain-language **What / Why / How**
   block (per the user's ticket/PR convention — non-technical PM could follow it), then:
   - **Problem & goal** · **Users & their job** · **Scope (in)** · **Non-goals (out)**
   - **User stories / flows** · **Acceptance criteria** (testable) · **Edge cases & failure states**
   - **UX notes** (key screens/states the feature implies) · **Open questions / assumptions**
   - **Dependencies & risks**.
3. **Validate the spec (separate pass).** Run **`ie-validate-plan`** against the spec — clarity, simplicity
   (no speculative scope), predictability, and **UX completeness** (states/flows/IA/accessibility decided,
   not hand-waved). It returns 0–10 ratings + gaps. **Fold the gaps back in**; a spec with unresolved
   validation gaps is not ready. Optionally `ie-plan-assist` for a lighter advisory pass.
4. **Mark readiness.** State the score and "ready for `dte-arc-plan`" — or list what's still open.

## For a list of ideas
One sub-agent per idea through 1–4, then an index `docs/specs/INDEX-<batch>.md` listing each spec, its
readiness score, and dependency order (feeds `dte-loop`/`dte-arc-plan`).

## Degrade gracefully
- No ce-brainstorm/ce-ideate → shape the spec by hand from the idea + codebase context, note it.
- No ie-validate-plan → self-review against the section checklist, flag that validation was not a separate pass.

## Done when
Each idea has a spec doc with the plain-language What/Why/How, scope + non-goals, testable acceptance
criteria, edge/UX states, and a **validation score with gaps resolved** — ready to hand to `dte-arc-plan`.
