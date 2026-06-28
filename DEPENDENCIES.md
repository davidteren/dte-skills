# dte-skills — what each skill needs

`dte-skills` is an orchestrator: it invokes other plugins' skills/agents. This page lists, **per skill**,
what it composes — **Required** (the skill can't do its core job without it), **Recommended** (materially
better), **Optional** (used if present, gracefully skipped with a note if not).

> Most skills degrade gracefully. A 🟢 Required item missing means the skill falls back to a weaker path
> (named in its output) or asks you to install it. Optional items never block.

## Plugins referenced

| Short name | Plugin / source | Provides |
|---|---|---|
| compound-engineering | [EveryInc/compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin) | `/ce-plan`, `/ce-work`, `/ce-code-review` (cross-model), `/ce-simplify-code`, `/ce-debug`, `/ce-sessions` |
| intent-engineering | [davidteren/intent-engineering](https://github.com/davidteren/intent-engineering) | `/ie-validate-plan`, `/ie-review`, `/ie-audit`, `/ie-plan-assist`, `ie-*-reviewer` agents |
| layered-rails | [palkan/skills](https://github.com/palkan/skills) | `/layered-rails`, `analyze-callbacks/gods/services`, `layered-rails-planner/reviewer` |
| majestic-rails | majestic-marketplace | `pragmatic-rails-reviewer`, `data-integrity-reviewer`, `review:rails-code-review`, `minitest-coder`, … |
| rails-testing | visuality `rails-testing-v8` | idiomatic Rails 8 + Minitest + fixtures guidance |
| hotwire-rails-toolkit | [davidteren/hotwire-rails-toolkit](https://github.com/davidteren/hotwire-rails-toolkit) | runnable Hotwire/Stimulus/Turbo **checkers** (CI-friendly bash): `lint_stimulus`, `lint_turbo_streams`, `lint_morphing`, `audit_token_auth`, `upgrade_audit` + LazyRouteSet flake detector, `lint_bridge_contract`/`lint_path_config` — deterministic linters that flag failures-with-no-error |
| ponytail | ponytail | the lazy-senior elegance / over-engineering pass |
| cubic | cubic | `/run-review`, `/scan`, `codebase-context`, `wiki` |
| Augment | auggie MCP | semantic `codebase-retrieval` |
| MemPalace | mempalace MCP | history, knowledge-graph, decisions |
| mutation tester | `mutant` (licensed) or **brutus** (homegrown) | "does a test fail when code breaks" signal |

## Per-skill matrix

### `/dte-arc-auditor` — app audit → migration roadmap
| Required 🟢 | Recommended 🟡 | Optional ⚪ |
|---|---|---|
| compound-engineering (`ce-plan`), intent-engineering (`ie-validate-plan`), layered-rails | majestic-rails, ponytail | Augment, cubic, MemPalace |
Composes `dte-arc-review` + `dte-arc-plan` internally. Without ce-plan/ie-validate-plan it cannot produce
a *validated* roadmap — its whole point.

### `/dte-arc-review` — architectural findings
| Required 🟢 | Recommended 🟡 | Optional ⚪ |
|---|---|---|
| layered-rails **or** intent-engineering (`ie-audit`) | majestic-rails, ponytail | Augment, cubic, **layer-boundary-lint** (deterministic layer-violation pre-pass) |
Needs at least one architecture lens. With both layered-rails and ie-audit it cross-checks. When
**layer-boundary-lint** is present, run it FIRST as a no-LLM grep gate (Current-in-models, request/params
in the domain incl. interactors, raw queries in controllers/views, I/O in `after_*` callbacks) → `file:line`
facts that ground the lenses. (Incubating in `Rails_Skills_Analysis/adoption/skills/`; not yet a published
plugin — path-referenced, skipped-with-a-note when absent.)

### `/dte-deep-reviewer` — PR / branch / diff review
| Required 🟢 | Recommended 🟡 | Optional ⚪ |
|---|---|---|
| compound-engineering (`ce-code-review`) **or** intent-engineering (`ie-review`) | majestic-rails (`review:rails-code-review`) | cubic (`run-review`/`scan`), Augment, **hotwire-rails-toolkit** (Stimulus/Turbo/bridge linters when the diff has FE/native files) |
Cross-model second opinion comes from `ce-code-review` (compound-engineering ≥ 3.14). On a diff touching
Stimulus/Turbo/Hotwire Native, run the **hotwire-rails-toolkit** checkers first (per the Front-end lens in
`conventions.md`) — deterministic findings that ground the LLM review.

### `/dte-test-auditor` — test value / quality / coverage
| Required 🟢 | Recommended 🟡 | Optional ⚪ |
|---|---|---|
| a test framework + SimpleCov (coverage) | compound-engineering (testing persona), rails-testing, majestic `minitest-coder` | a mutation tester (`mutant` or **brutus**), **rails-test-smell-checker** (deterministic smell pre-pass) |
When **rails-test-smell-checker** is present, run it FIRST — a no-LLM static pass (Minitest AND RSpec)
flagging high-confidence smells (`sleep` in system tests, missing `disable_net_connect!`, stubbing the SUT,
`has_css?` inside an assertion) with `file:line`; the quality lens then reads deeper. (Incubating in
`Rails_Skills_Analysis/adoption/skills/`; not yet a published plugin — skipped-with-a-note when absent.)
Coverage + quality run without mutation; the "does a test catch a bug" signal needs a mutation tester
(flagged as not-run when absent).

### `/dte-arc-plan` — findings/spec/bug → validated plan(s)
| Required 🟢 | Recommended 🟡 | Optional ⚪ |
|---|---|---|
| compound-engineering (`ce-plan`), intent-engineering (`ie-validate-plan`) | intent-engineering (`ie-plan-assist`), layered-rails (`layered-rails-planner`) | — |
Hard requirement by design: plans are built with `ce-plan` and validated with `ie-validate-plan`.

### `/dte-arc-work` — build, gated and looped
| Required 🟢 | Recommended 🟡 | Optional ⚪ |
|---|---|---|
| compound-engineering (`ce-work`) | `ce-simplify-code` + ponytail, `dte-deep-reviewer`, `dte-test-auditor` | `ce-debug`, Augment |
Executes the plan via `ce-work`; the verify step uses the review/test skills; the elegance step uses
ponytail/ce-simplify-code.

### `/dte-tooling-scan` — what to adopt next
| Required 🟢 | Recommended 🟡 | Optional ⚪ |
|---|---|---|
| — | compound-engineering (`ce-sessions`), MemPalace | Augment, web search |
Runs on whatever history sources exist; with none, falls back to repo `STATUS`/git history (and says so).

### `/dte-loop` — batch → worklist + autonomous gated loop
| Required 🟢 | Recommended 🟡 | Optional ⚪ |
|---|---|---|
| `dte-arc-work` (the executor) | `dte-arc-plan` (builds a validated INDEX worklist), built-in `/loop` (resilient wrapper) | `dte-deep-reviewer`/`dte-arc-review` (review-merge batches) |
Sets up the worklist + emits the trigger command; the actual loop is `dte-arc-work`. Without `dte-arc-plan`
the worklist is built by hand (plans unvalidated); without `/loop` only the direct command is emitted.

### `/dte-spec` — idea → validated spec/PRD
| Required 🟢 | Recommended 🟡 | Optional ⚪ |
|---|---|---|
| intent-engineering (`ie-validate-plan`) | compound-engineering (`ce-brainstorm`, `ce-ideate`) | Augment/cubic (existing-feature context) |
Validation is the hard requirement — a spec isn't done until `ie-validate-plan` gaps are resolved.

### `/dte-ux` — review & produce front-end (UI/UX)
| Required 🟢 | Recommended 🟡 | Optional ⚪ |
|---|---|---|
| the **ui.sh** family (`ui`, `ui-design`, `ui-componentize`, `ui-make-responsive`, `ui-add-dark-mode`, `ui-markup-from-image`) **or** `frontend-design` + `ie-experience-reviewer` | both of the above together | chrome-devtools (`lighthouse_audit`, snapshots), Augment, **hotwire-rails-toolkit** (`lint_stimulus` + `stimulus-patterns`/`turbo-*` guidance for Stimulus/Turbo work) |
Review runs on frontend-design + ie-experience-reviewer; building needs the ui.sh skills. No browser → a11y
is heuristic, not measured.

### `/dte-feature` — full-stack feature (both layers)
| Required 🟢 | Recommended 🟡 | Optional ⚪ |
|---|---|---|
| `dte-arc-plan`, `dte-arc-work`, `dte-ux` | `dte-deep-reviewer` + `dte-test-auditor` (verify), `ce-simplify-code`/ponytail | `dte-spec` (shape a fuzzy idea first), chrome-devtools, **hotwire-rails-toolkit** (Stimulus/Turbo/bridge checkers in the verify step) |
Composes the arc skills for the back-end and `dte-ux` for the front-end; both layers must verify.

### `/dte-perf` — measured performance review
| Required 🟢 | Recommended 🟡 | Optional ⚪ |
|---|---|---|
| majestic-rails (`performance-reviewer`, `database-optimizer`) | AppSignal MCP (production numbers), chrome-devtools (FE traces + Lighthouse), `database-admin` | Augment, a load/benchmark tool |
Measures first. No AppSignal → EXPLAIN/logs/benchmarks; no chrome-devtools → FE findings flagged heuristic.

### `/dte-security-sweep` — security posture / PR review
| Required 🟢 | Recommended 🟡 | Optional ⚪ |
|---|---|---|
| the `security-audit` **or** `security-review` skill | **Brakeman** + **bundler-audit**, majestic `privacy-reviewer`/`data-integrity-reviewer`, `ie-predictability-reviewer` | Augment (source→sink tracing), **hotwire-rails-toolkit** (`audit_token_auth` enumeration/plaintext-token/opt-in-auth checks; `lint_turbo_streams` broadcast-eavesdropping check) |
Scanners give the runnable signal; absent → manual sink-tracing, flagged "scanners not run". On a Hotwire app,
the hotwire-rails-toolkit auth + stream linters add two encoded vuln classes (user enumeration, stream
eavesdropping) the generic scanners miss — `audit_token_auth` assumes a DB-backed token design, so on Devise/
other auth treat its output as advisory.

### `/dte-debug` — root-cause bug fix
| Required 🟢 | Recommended 🟡 | Optional ⚪ |
|---|---|---|
| compound-engineering (`ce-debug`) **or** majestic `rails-debugger` | a test framework (repro + regression test), Augment (trace the path) | AppSignal (production error context) |
Needs a runnable repro before and after. No AppSignal → logs/local repro for production bugs.

### `/dte-migrate` — safe upgrade / schema-data migration
| Required 🟢 | Recommended 🟡 | Optional ⚪ |
|---|---|---|
| `dte-arc-plan`/`ce-plan` + `ce-work` | majestic `gemfile-upgrade`, `gemfile-organize`, `database-admin`, `database-optimizer`, `gem-research`, **hotwire-rails-toolkit** (`upgrade_audit` + LazyRouteSet flake detector — for a Rails 7→8 bump) | `dte-loop` (run the stepwise plan autonomously) |
Every step reversible + gate-green (incl. migrate↔rollback round-trip). No data migration without a tested rollback.
For a **Rails 7→8** upgrade specifically, run hotwire-rails-toolkit's `upgrade_audit` (version/defaults/risky-gem
pre-flight) and `lint_route_test_helper` (the intermittent `LazyRouteSet` route-test flake that hides behind
single green runs) as part of the readiness + verify steps.

### `/dte-pm` — goal → tracked, sequenced issues
| Required 🟢 | Recommended 🟡 | Optional ⚪ |
|---|---|---|
| `gh` CLI (GitHub issues) **or** the project's `dt-create-issue` | compound-engineering (`ce-plan`/`ce-brainstorm` to decompose) | Augment/cubic (existing-feature context) |
No `gh` → writes a markdown backlog index, notes tickets weren't filed. Honors the project's What/Why/How + label taxonomy.

### `/dte-pwa` — installable / offline Rails app
| Required 🟢 | Recommended 🟡 | Optional ⚪ |
|---|---|---|
| **Rails 8** (native PWA scaffolding) | the ui.sh family + frontend-design (install/offline UI), ie-experience-reviewer | chrome-devtools (`lighthouse_audit`), Augment |
Pre-Rails-8 → manifest + SW added by hand from the Rails 8 template. No browser → manual PWA checklist, score unmeasured.

### `/dte-skill-audit` — audit the dte-* suite
| Required 🟢 | Recommended 🟡 | Optional ⚪ |
|---|---|---|
| this repo's `AGENTS.md` + `references/conventions.md` (the rubric) + `claude plugin validate` | — | — |
Self-contained + read-only — reads the skills, AGENTS.md, and DEPENDENCIES.md; needs no external plugin.

## Minimum vs full

- **Minimum useful set:** compound-engineering + intent-engineering (+ layered-rails for the Rails
  architecture skills). That unlocks plan/validate/work/review.
- **Full Rails set:** add majestic-rails, rails-testing, ponytail, and a mutation tester (mutant or brutus).
- **Best-in-class:** also Augment (context), cubic (team patterns), MemPalace (history).
