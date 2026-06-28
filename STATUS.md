# STATUS — dte-skills

_Living tracker for the plugin. Last updated: 2026-06-28._

**What this is:** a Claude Code plugin of `dte-*` orchestration skills (review → plan → work → audit)
that compose the existing AI-engineering toolchain. **Public:** github.com/davidteren/dte-skills ·
site davidteren.github.io/dte-skills. **Maintainer guide:** [AGENTS.md](AGENTS.md).

**Current:** **v1.4.0**, 18 skills, published. Install: `claude plugin marketplace add davidteren/dte-skills`
→ `claude plugin install dte-skills@dte-skills-marketplace`. Gaps + decisions log: [GAPS.md](GAPS.md).

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
| `dte-loop` | batch → worklist doc + autonomous gated loop command (questions front-loaded) |
| `dte-spec` | fuzzy idea → validated spec/PRD (ce-brainstorm + ie-validate-plan) → feeds dte-arc-plan |
| `dte-ux` | review **and** produce front-end (ui.sh + frontend-design + ie-experience-reviewer) |
| `dte-feature` | full-stack feature — plan/build/verify **both** back-end and front-end |
| `dte-perf` | measured perf review → ranked evidence-backed fixes (AppSignal, EXPLAIN, Lighthouse) |
| `dte-security-sweep` | security posture / PR review — traced vulns + Brakeman/bundler-audit |
| `dte-debug` | bug → runnable repro → root cause → fix once at the shared path → regression test |
| `dte-migrate` | safe upgrade / schema-data migration — small reversible gated steps + rollback |
| `dte-pm` | goal/spec → tracked, sequenced GitHub issues (What/Why/How) + tracking index |
| `dte-pwa` | make a Rails app installable/offline on Rails 8 native PWA + ui.sh + Lighthouse |
| `dte-skill-audit` | audit the dte-* suite against the AGENTS.md quality bar + trigger-overlap matrix |

**Releases:** v1.0.0 the 6 core skills · v1.1.0 hard-wired plan-then-validate + decomposition + gated
loops + claim-verify + per-phase reporting (lessons from the ongela refactor session) · v1.2.0 added
`dte-arc-auditor` · then renamed `dte-workflows → dte-skills` and published public (MIT) · **v1.3.0 added
8 skills: `dte-loop` (autonomous-loop driver), `dte-spec`, `dte-ux`, `dte-feature`, `dte-perf`,
`dte-security-sweep`, `dte-debug`, `dte-migrate` + the [GAPS.md](GAPS.md) tracker · **v1.4.0** dogfooded
`/dte-loop` on the backlog → added `dte-pm`, `dte-pwa`, `dte-skill-audit` + wired the **front-end lens**
into `dte-deep-reviewer`/`dte-arc-review`/`conventions.md`.**

---

## Backlog 🔭

Status: ☐ todo · ◐ designed/parked · ▶ next.

### New skills to build
- ◐ **`dte-discovery`** *(parked — owner building in a dedicated session)* — onboard/understand a codebase
  or feature: map it (Augment + cubic wiki), trace the real flow, surface entry points, conventions, risks
  → a discovery/onboarding doc.

_(Shipped in v1.4.0: `dte-pm`, `dte-pwa`, `dte-skill-audit`, and the front-end-lens-in-reviews enhancement.)_

### Open follow-ups
- ☐ **`dte-pwa` runtime dogfood** — authored + validated, but not yet run against a real Rails 8 app
  (no app in this repo). Verify the Lighthouse PWA pass on a live app. (Logged in [GAPS.md](GAPS.md).)
- ☐ **`dte-skill-audit` first run** — built but not yet run on the suite. Run it to get the baseline
  health score + trigger-overlap matrix across all 18 skills.
- ☐ **Auto-pull the FE lens** — the rule + conditional step are wired; confirm in practice that reviews
  actually trigger `ie-experience-reviewer`/ui.sh on a real FE diff.

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
