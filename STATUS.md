# STATUS — dte-skills

_Living tracker for the plugin. Last updated: 2026-06-28._

**What this is:** a Claude Code plugin of `dte-*` orchestration skills (review → plan → work → audit)
that compose the existing AI-engineering toolchain. **Public:** github.com/davidteren/dte-skills ·
site davidteren.github.io/dte-skills. **Maintainer guide:** [AGENTS.md](AGENTS.md).

**Current:** **v1.4.6**, 18 skills, published. Install: `claude plugin marketplace add davidteren/dte-skills`
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
into `dte-deep-reviewer`/`dte-arc-review`/`conventions.md`.** · **v1.4.2** wired the
**hotwire-rails-toolkit** deterministic checkers in as an Optional composed dependency — the
deterministic-linter layer the suite lacked (everything else is LLM-driven). Its `lint_stimulus`/
`lint_turbo_streams`/`lint_morphing`/`lint_bridge_contract` checkers now back the Front-end lens in reviews,
`audit_token_auth` + the stream-eavesdrop check feed `dte-security-sweep`, and `upgrade_audit` + the
LazyRouteSet flake detector feed `dte-migrate` on Rails 7→8 bumps. (Toolkit `lint_stimulus` typed-Values
false-positive fixed + shipped as toolkit v1.0.1 in the same pass; validated against `miela_app`.)

---

## Backlog 🔭

Status: ☐ todo · ◐ designed/parked · ▶ next.

### New skills to build
- ◐ **`dte-discovery`** *(parked — owner building in a dedicated session)* — onboard/understand a codebase
  or feature: map it (Augment + cubic wiki), trace the real flow, surface entry points, conventions, risks
  → a discovery/onboarding doc.

_(Shipped in v1.4.0: `dte-pm`, `dte-pwa`, `dte-skill-audit`, and the front-end-lens-in-reviews enhancement.)_

### Open follow-ups
- ✅ **`dte-pwa` runtime dogfood** — run on `miela_app` (Rails 8.1.3) 2026-06-28. Found it **already a
  correctly-configured installable PWA** (manifest + icons + maskable + SW registered, both layouts) needing
  **zero changes** — and its SW deliberately skips offline caching (a `respondWith` once broke file
  downloads). Surfaced a real skill gap → **v1.4.3** added a step-1 *honor-an-existing-deliberate-SW* guard +
  a `respondWith`-vs-downloads warning to `dte-pwa`. **Re-run on 1.4.3 (2026-06-28):** guard fired correctly
  → "already installable, zero changes" (the old path would have re-broken downloads). **Second finding when
  trying for a live number:** Lighthouse **removed the standalone PWA category in v12** — there is no "PWA
  score" anymore → **v1.4.6** rewrote `dte-pwa` step 5/6 to verify via the **Chrome DevTools Application
  panel** (Manifest + Service Workers installability verdict) + best-practices/accessibility Lighthouse +
  manual HTTP fallback, and dropped the dead "Lighthouse PWA pass" claim (also fixed on the site).
- ✅ **`dte-skill-audit` first run** — done 2026-06-28 (`docs/reviews/skill-audit-2026-06-28.md`): suite
  9/10 → **10/10** after v1.4.1 applied the 1 🟠 + 3 🟡 wording/boundary fixes.
- ✅ **Auto-pull the FE lens** — **partially confirmed by dogfood** (2026-06-28): ran `dte-deep-reviewer`
  on `miela_app`'s Hotwire/Stimulus slice; the **hotwire-rails-toolkit checker pre-pass fired and caught
  the one real finding** (`search_form` timer leak) before the LLM lens. Two caveats remain: (1) the session
  loaded the **1.2.0** skill body, so the checkers ran by explicit instruction, not the `conventions.md`
  auto-trigger — **still to confirm after a restart to ≥1.4.2** that a Stimulus/Turbo diff pulls them in on
  its own; (2) the run surfaced a real checker gap — `lint_turbo_streams` is blind to controller-response
  `*.turbo_stream.erb` templates → filed as hotwire-rails-toolkit **issue #1**. Review doc:
  `miela_app:docs/reviews/deep-review-hotwire-slice-2026-06-28.md`.
  **Caveat (1) resolved 2026-06-28 (≥1.4.2 session):** on a real FE diff (`miela_app@fc2fec7`, an ARIA
  change) the FE-file detector fired the **experience lens autonomously** (4 FE files detected; a BE-only
  diff `@7a746a6` correctly skipped — both branches proven), and the lens returned real codebase-grounded
  findings, incl. **2 P1 a11y bugs the original ARIA commit missed** (Space-key page-scroll on the
  `role=button` div with no `preventDefault`; the `card` utility's `outline` suppressing the keyboard focus
  ring). Caveat (2) `lint_turbo_streams` gap stays open as hotwire-rails-toolkit issue #1.
  **Bugs filed (2026-06-28):** `miela_app` **#606** (Space-key page-scroll, WCAG 2.1.1) + **#607** (missing
  focus ring, WCAG 2.4.7) — labels `Bug Fix`/`ui`/`Phase 5`, plain-language What/Why/How bodies, each with a
  copy-paste **`/dte-debug` fix command attached as a comment** (root cause + file:line + exact fix + repro→
  regression-test, gate `bin/rails test && bin/rails test:system`). Full chain proven: FE-lens → real bug →
  tracked issue → executable fix command.

---

## Decisions

- **Naming:** plugin `dte-skills`; skills keep the `dte-` prefix; `dte-arc-*` for the review→plan→work arc.
- **License:** MIT. **Public** repo + GitHub Pages.
- **Mandatory way of working:** real work → `ce-plan` then `ie-validate-plan` (+ validators), gaps resolved,
  before execution; big tasks decompose into many validated plans. (Hard-wired in `dte-arc-plan`/`-work`.)
- **interactors-not-services** for these projects.
- Skills are recipes that **compose** installed tools — see [DEPENDENCIES.md](DEPENDENCIES.md) for what
  each needs. Keep the suite tight; reject overlap (the majestic 47-skill bloat lesson).
- **Deterministic checkers vs LLM lenses:** the composed toolchain is LLM-driven (coders/reviewers); the
  **hotwire-rails-toolkit** is the one source of runnable, exact linters (failures-that-produce-no-error).
  Where it applies (Hotwire/Stimulus/Turbo/native diffs, Rails 7→8, token auth) run its checker as a fast
  exact pre-pass that **grounds** the LLM lens — never replaces it. Always Optional ⚪: skip with a note if
  not installed. `audit_token_auth` assumes a DB-backed token design → advisory on Devise/other auth.

## How to evolve this safely
Read [AGENTS.md](AGENTS.md) before adding/editing a skill — it carries the design rules and the lessons
(why dark loops, self-critique, and unverified claims are banned). Bump version in both manifests, validate,
push, and `claude plugin update`.
