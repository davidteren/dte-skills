# GAPS — dte-skills

_Where the suite is thin, and the decisions taken to fill it. Companion to [STATUS.md](STATUS.md)
(tracker) and [AGENTS.md](AGENTS.md) (how to build a skill safely). Last updated: 2026-06-28._

A **gap** = a job a Rails engineer hits that no `dte-*` skill (or composed tool) covers yet. Each row
says what's missing, whether an existing skill/tool already covers it, and the decision (build / log / extend).

---

## Open gaps

### PWA (Progressive Web App) for Rails — ✅ built v1.4.0 (`dte-pwa`), runtime dogfood pending
**Gap:** no skill for making a Rails app installable/offline-capable (manifest, service worker, install
prompt, offline cache, push). **What exists:** **Rails 8 ships native PWA** — `app/views/pwa/manifest.json.erb`
+ `service-worker.js`, routes commented in `config/routes.rb`. The **ui.sh** family + **frontend-design**
cover the UI; **ie-experience-reviewer** the UX. No skill stitches these into a PWA recipe.
**Decision:** **built as `dte-pwa` in v1.4.0** — composes Rails 8 native PWA (manifest + service worker,
not hand-rolled), the ui.sh skills (install UI / offline states), frontend-design, ie-experience-reviewer,
and a Lighthouse PWA audit. **Open follow-up:** authored + validated only — not yet run against a real
Rails 8 app (none in this repo); dogfood the live Lighthouse pass later. See STATUS "Open follow-ups".

### Front-end lens in reviews — ✅ done v1.4.0
**Decision:** rule added **once** to `references/conventions.md` ("Front-end lens"); a conditional FE-detect
step added to `dte-deep-reviewer` + `dte-arc-review` that pulls **ui.sh** + **frontend-design** +
**ie-experience-reviewer** when the diff touches erb/haml/slim, JS/TS, Stimulus, ViewComponent, CSS/Tailwind.
**Open follow-up:** confirm it actually triggers on a real FE diff in practice.

### skill-audit — ✅ built v1.4.0 (`dte-skill-audit`), first run pending
**Decision:** built as **`dte-skill-audit`** (kept the `dte-` prefix) — read-only, scores each skill on the
7 quality-bar dimensions + a cross-skill trigger-overlap matrix → a report. **Open follow-up:** run it to
get the baseline health score across all 18 skills.

### dte-pm (project management) — ☐ todo (carried from STATUS backlog)
Idea/goal → tracked, broken-down work (issues), status, sequencing. Composes `dt-create-issue` / GitHub
issues + `ce-plan`. Overlaps `dte-spec` (which defines the *what*) — keep dte-pm to *tracking/sequencing*.

### dte-discovery — ◐ parked (owner building in a dedicated session)
Onboard/understand a codebase or feature → discovery doc. Tracked in STATUS; left to the owner's session.

---

## Decisions log (forks resolved with recommended defaults — override anytime)

This session built the loop driver + the sibling skills. Where a fork was open and the user asked to
proceed, the **recommended default** was taken and recorded here (mirrors the `dte-loop` "assumed defaults"
pattern). Override by saying so.

| Fork | Went with (recommended) | Override |
|---|---|---|
| Loop-driver skill name | **`dte-loop`** (matches the built-in `/loop` it drives) | rename in frontmatter + manifests |
| Build order | **`dte-loop` first**, then ux, feature, perf, spec, security-sweep, debug, migrate | reorder |
| PWA now or later | **Log here, build `dte-pwa` later** | say "build dte-pwa now" |
| `dte-spec` meaning | **idea → validated spec/PRD** (ce-brainstorm + ie-validate-plan); feeds dte-arc-plan | redefine as test-spec authoring |
| `dte-migrate` meaning | **Rails/gem/data upgrades** (gemfile-upgrade, database-admin, ce-plan/work) | redefine as arch-migration executor |
| Merge in autonomous loops | **manual by default; auto-merge opt-in only** (outward-facing) | set `merge-policy: auto-if(...)` per loop |

---

## How to close a gap
Follow **AGENTS.md → "How to ADD a skill"**: earn it (distinct job, no overlap), write
`skills/dte-<name>/SKILL.md` (read conventions first, compose/loop/degrade/honor-conventions), add its
row to `DEPENDENCIES.md`, bump both manifests, validate, move the row here from "open" to STATUS "Shipped".
