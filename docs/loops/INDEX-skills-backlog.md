# Loop worklist — dte-skills backlog (v1.4.0)

type: task (skill-authoring in this repo)
gate: `claude plugin validate .` + per-skill frontmatter/trigger sanity (no test suite in a skills plugin)
branch: main (direct) — solo public docs repo, matches the v1.3.0 release flow; one commit per item
merge-policy: n/a (task loop, not review-merge)
serial: true
cadence: run-straight-through → single report + one version bump (1.3.0 → 1.4.0) + push + plugin update at end
decisions: see "Assumed defaults" below

## Worklist (build in order)

- [x] **PM** · `dte-pm` — idea/goal → tracked GitHub issues + sequencing · status:done · result: `skills/dte-pm/SKILL.md`
- [x] **PWA** · `dte-pwa` — make a Rails app installable/offline · status:done · result: `skills/dte-pwa/SKILL.md` (runtime dogfood pending)
- [x] **FE-LENS** · enhancement — front-end lens in reviews · status:done · result: `references/conventions.md` + `dte-deep-reviewer` + `dte-arc-review`
- [x] **AUDIT** · `dte-skill-audit` — audit the dte-* suite · status:done · result: `skills/dte-skill-audit/SKILL.md` (first run pending)

## Assumed defaults (unforeseeable-at-setup forks — recommended taken; override anytime)

- **`dte-pwa` runtime verify** → went with **author + validate only** (no Rails 8 app in this repo to run a
  real Lighthouse PWA audit against). Override: dogfood it against a real app later.
- **`skill-audit` name** → went with **`dte-skill-audit`** (keeps the `dte-` prefix per AGENTS.md). Override: rename.
- **`skill-audit` shape** → went with **read-only report** (no auto-fix of skills). Override: add a `--fix` pass later.
- **`dte-pm` issue backend** → went with **`gh issue create`** generic + `dt-create-issue` if the target
  project has it (Miela-specific). Override: wire a different tracker.
- **Version bump** → **minor, 1.3.0 → 1.4.0** (4 new/changed skills, additive). Override: major if you consider it breaking.

## Status
Built by `/dte-loop` (dogfood). Execution: run-straight-through per the cadence answer. Boxes ticked as each lands.
