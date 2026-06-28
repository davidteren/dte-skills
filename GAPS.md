# GAPS — dte-skills

_Where the suite is thin, and the decisions taken to fill it. Companion to [STATUS.md](STATUS.md)
(tracker) and [AGENTS.md](AGENTS.md) (how to build a skill safely). Last updated: 2026-06-28._

A **gap** = a job a Rails engineer hits that no `dte-*` skill (or composed tool) covers yet. Each row
says what's missing, whether an existing skill/tool already covers it, and the decision (build / log / extend).

---

## Open gaps

### PWA (Progressive Web App) for Rails — ◐ logged, build later
**Gap:** no skill for making a Rails app installable/offline-capable (manifest, service worker, install
prompt, offline cache, push). **What exists:** **Rails 8 ships native PWA** — `app/views/pwa/manifest.json.erb`
+ `service-worker.js`, routes commented in `config/routes.rb`. The **ui.sh** family + **frontend-design**
cover the UI; **ie-experience-reviewer** the UX. No skill stitches these into a PWA recipe.
**Decision:** **log now, build `dte-pwa` later.** When built it should *compose* Rails 8 native PWA (don't
hand-roll a service worker), the ui.sh skills (install UI / offline states), frontend-design, and
ie-experience-reviewer — plus a Lighthouse PWA audit via chrome-devtools `lighthouse_audit`. Sharp trigger:
"make this a PWA / installable / work offline / add a service worker / web app manifest". Not for general
front-end work (use dte-ux).

### Front-end lens in reviews — ☐ todo (carried from STATUS backlog)
`dte-deep-reviewer` + `dte-arc-review` detect back-end/architecture smells but **not front-end changes**.
**Decision:** extend both to detect FE diffs (erb/haml/slim, JS/TS, Stimulus, ViewComponent, CSS/Tailwind)
and pull in **ui.sh** + **frontend-design** + **ie-experience-reviewer**. Add the rule to
`references/conventions.md` once, reference from both skills. (Now partly covered by `dte-ux`, but reviews
should auto-pull the FE lens, not require invoking dte-ux by hand.)

### skill-audit — ☐ later (carried from STATUS backlog)
No mechanical way to audit the `dte-*` skills themselves (compose-not-reinvent, loop, degrade, sharp
non-overlapping triggers, gate+report). **Decision:** build `skill-audit` later; until then audit by hand
against the AGENTS.md "Quality bar".

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
