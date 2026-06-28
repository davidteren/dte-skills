---
name: dte-ux
description: Designer / UX-UI specialist — review AND produce front-end work: design fidelity, interaction states, look-and-feel, responsiveness, dark mode, and accessibility, ending in a findings doc and/or applied UI. Composes the ui.sh family (ui, ui-design, ui-componentize, ui-make-responsive, ui-add-dark-mode, ui-markup-from-image), frontend-design, and ie-experience-reviewer. Use when the user says "review the UI/UX", "design this screen", "improve the look-and-feel", "check accessibility", "make it responsive", "build this component", "turn this mockup/screenshot into UI", or "does this feel polished". Loops over a list of screens/components. Not for back-end/architecture (use dte-arc-review) or a full front+back feature (use dte-feature).
---

# dte-ux — review & produce front-end (UI/UX) → findings doc + applied UI

Read `../../references/conventions.md` first (tool detection, looping, output shape). This skill is a
**recipe**: it conducts the ui.sh skills + frontend-design + the experience reviewer. It can **review**
(write findings) or **produce** (build/refine UI), or both — say which.

## Input
- A target: a screen, a component, a flow, a mockup/screenshot, or a slice of front-end.
- Or a **list** → handle each (parallel sub-agents by default; `--serial` for a shared design system).
- Read the project `CLAUDE.md`/`AGENTS.md` + any design tokens/component library before judging.
- Flags: `--review` (findings only, default if code already exists) · `--build` (produce UI) · both.

## Procedure (per target)
1. **Map the surface.** Find the views/components/styles in play (Augment `codebase-retrieval`; fallback
   Grep/Glob over `app/views`, `app/components`, JS/Stimulus, Tailwind config). Note the design system in use.
2. **From a mockup/screenshot?** Run **`ui-markup-from-image`** to get semantic unstyled markup first, then
   style it with the project's system.
3. **Review lens (always).** Run **`ie-experience-reviewer`** over the target — missing interaction states
   (loading/empty/error/disabled), inconsistent look-and-feel, broken keyboard/focus/back-button,
   accessibility gaps, weak information architecture, AI-slop defaults. Pull **frontend-design** for
   aesthetic direction (typography, spacing, intentional vs templated).
4. **Produce (`--build`).** Use the right ui.sh skill for the job — **`ui-design`**/**`ui`** for new UI,
   **`ui-componentize`** to extract reusable components, **`ui-make-responsive`** for breakpoints,
   **`ui-add-dark-mode`** for theming. Follow the existing component patterns; don't introduce a second
   design language.
5. **Verify it's real.** Where possible run it in a browser (chrome-devtools `take_snapshot` /
   `take_screenshot`) and a **`lighthouse_audit`** for accessibility/best-practices score — prefer the
   runnable a11y signal over prose. State if the browser wasn't available.
6. **Synthesize.** Severity-rank findings (🔴 broken/inaccessible · 🟠 inconsistent · 🟡 polish), each with
   `file:line` + a concrete fix. Surface where frontend-design and ie-experience-reviewer disagreed.

## For a list of targets
One sub-agent per screen/component through steps 1–6, then a roll-up `docs/reviews/ux-rollup-<date>.md`
ranking the worst surfaces + cross-cutting themes (inconsistent states, missing a11y, design drift).

## Degrade gracefully
- No ui.sh skills → fall back to frontend-design + ie-experience-reviewer for review; note build is manual.
- No browser/chrome-devtools → skip the Lighthouse/snapshot step, flag a11y as "heuristic, not measured".

## Done when
The target has a UX findings doc (verdict + a11y/polish score, severity-ranked findings with `file:line`,
disputed section, tools used/unavailable) and/or built UI that matches the project's design system, with
interaction states + accessibility handled and (where possible) a Lighthouse score recorded.
