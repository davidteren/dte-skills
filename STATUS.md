# STATUS — dte-skills

_Living tracker for the plugin. Last updated: 2026-06-28._

**What this is:** a Claude Code plugin of `dte-*` orchestration skills (review → plan → work → audit)
that compose the existing AI-engineering toolchain. **Public:** github.com/davidteren/dte-skills ·
site davidteren.github.io/dte-skills. **Maintainer guide:** [AGENTS.md](AGENTS.md).

**Current:** **v1.2.0**, 7 skills, published. Install: `claude plugin marketplace add davidteren/dte-skills`
→ `claude plugin install dte-skills@dte-skills-marketplace`.

---

## Shipped ✅

| Skill | Job |
|---|---|
| `dte-arc-auditor` | audit a whole app → target architecture + validated migration roadmap + worklist |
| `dte-arc-review` | review a slice / structure → architectural findings |
| `dte-deep-reviewer` | review a PR / branch / diff → cross-model code findings |
| `dte-test-auditor` | judge a test suite — coverage + quality + mutation |
| `dte-arc-plan` | findings/spec/bug → ce-plan-built, ie-validate-plan-validated plans (decomposes big → many) |
| `dte-arc-work` | do the work — gated, looped, report+pause per phase |
| `dte-tooling-scan` | mine history → overlap-aware adoption shortlist |

**Releases:** v1.0.0 the 6 core skills · v1.1.0 hard-wired plan-then-validate + decomposition + gated
loops + claim-verify + per-phase reporting (lessons from the ongela refactor session) · v1.2.0 added
`dte-arc-auditor` · then renamed `dte-workflows → dte-skills` and published public (MIT).

---

## Backlog 🔭

Status: ☐ todo · ◐ designed/parked · ▶ next.

### New skills to build
- ◐ **`dte-discovery`** *(parked — owner building in a dedicated session)* — onboard/understand a codebase
  or feature: map it (Augment + cubic wiki), trace the real flow, surface entry points, conventions, risks
  → a discovery/onboarding doc. Will be the 8th skill.
- ☐ **`dte-pm`** — project-management skill: turn an idea/goal into tracked, broken-down work
  (issues/tickets), status, and sequencing. Compose `dt-create-issue` / GitHub issues + `ce-plan`.
- ☐ **`dte-ux`** (designer / UX-UI specialist) — review and produce UI/UX: design fidelity, interaction
  states, look-and-feel, accessibility. Compose the **ui.sh family** (`ui`, `ui-design`, `ui-componentize`,
  `ui-make-responsive`, …), `frontend-design`, `ie-experience-reviewer`, and the ce design agents.
- ☐ **`dte-feature`** (full-stack new feature) — an idea spanning front-end **and** back-end: plan +
  implement + verify across architecture/back-end **and** UI/UX/front-end, making sure **both** are solid.
  Compose `dte-arc-plan`/`dte-arc-work` + `dte-ux` + the ui.sh skills + `ie-experience-reviewer`.
- ☐ Parked siblings (noted in the origin plan): `dte-debug`, `dte-migrate`, `dte-security-sweep`,
  `dte-perf`, `dte-spec` — build as needed.

### Enhancements to existing skills
- ☐ **Front-end lens in reviews** — `dte-deep-reviewer` and `dte-arc-review` must detect **front-end
  changes** and pull in the **ui.sh** skills + `frontend-design` + `ie-experience-reviewer` (UI/UX,
  accessibility, look-and-feel), not just back-end/architecture lenses. Add to `references/conventions.md`
  + both review skills.
- ☐ **`skill-audit`** *(later)* — a way to **audit the dte-* skills themselves** for solidity: do they
  compose (not reinvent), loop, degrade gracefully, have sharp non-overlapping triggers, fire correctly?
  The keystone/skill-vet concept turned inward. Until built, audit by hand against AGENTS.md "Quality bar".

---

## Decisions

- **Naming:** plugin `dte-skills`; skills keep the `dte-` prefix; `dte-arc-*` for the review→plan→work arc.
- **License:** MIT. **Public** repo + GitHub Pages.
- **Mandatory way of working:** real work → `ce-plan` then `ie-validate-plan` (+ validators), gaps resolved,
  before execution; big tasks decompose into many validated plans. (Hard-wired in `dte-arc-plan`/`-work`.)
- **interactors-not-services** for these projects.
- Skills are recipes that **compose** installed tools — see [DEPENDENCIES.md](DEPENDENCIES.md) for what
  each needs. Keep the suite tight; reject overlap (the majestic 47-skill bloat lesson).

## How to evolve this safely
Read [AGENTS.md](AGENTS.md) before adding/editing a skill — it carries the design rules and the lessons
(why dark loops, self-critique, and unverified claims are banned). Bump version in both manifests, validate,
push, and `claude plugin update`.
