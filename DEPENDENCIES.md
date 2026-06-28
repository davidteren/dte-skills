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
| layered-rails **or** intent-engineering (`ie-audit`) | majestic-rails, ponytail | Augment, cubic |
Needs at least one architecture lens. With both layered-rails and ie-audit it cross-checks.

### `/dte-deep-reviewer` — PR / branch / diff review
| Required 🟢 | Recommended 🟡 | Optional ⚪ |
|---|---|---|
| compound-engineering (`ce-code-review`) **or** intent-engineering (`ie-review`) | majestic-rails (`review:rails-code-review`) | cubic (`run-review`/`scan`), Augment |
Cross-model second opinion comes from `ce-code-review` (compound-engineering ≥ 3.14).

### `/dte-test-auditor` — test value / quality / coverage
| Required 🟢 | Recommended 🟡 | Optional ⚪ |
|---|---|---|
| a test framework + SimpleCov (coverage) | compound-engineering (testing persona), rails-testing, majestic `minitest-coder` | a mutation tester (`mutant` or **brutus**) |
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
| the **ui.sh** family (`ui`, `ui-design`, `ui-componentize`, `ui-make-responsive`, `ui-add-dark-mode`, `ui-markup-from-image`) **or** `frontend-design` + `ie-experience-reviewer` | both of the above together | chrome-devtools (`lighthouse_audit`, snapshots), Augment |
Review runs on frontend-design + ie-experience-reviewer; building needs the ui.sh skills. No browser → a11y
is heuristic, not measured.

### `/dte-feature` — full-stack feature (both layers)
| Required 🟢 | Recommended 🟡 | Optional ⚪ |
|---|---|---|
| `dte-arc-plan`, `dte-arc-work`, `dte-ux` | `dte-deep-reviewer` + `dte-test-auditor` (verify), `ce-simplify-code`/ponytail | `dte-spec` (shape a fuzzy idea first), chrome-devtools |
Composes the arc skills for the back-end and `dte-ux` for the front-end; both layers must verify.

### `/dte-perf` — measured performance review
| Required 🟢 | Recommended 🟡 | Optional ⚪ |
|---|---|---|
| majestic-rails (`performance-reviewer`, `database-optimizer`) | AppSignal MCP (production numbers), chrome-devtools (FE traces + Lighthouse), `database-admin` | Augment, a load/benchmark tool |
Measures first. No AppSignal → EXPLAIN/logs/benchmarks; no chrome-devtools → FE findings flagged heuristic.

### `/dte-security-sweep` — security posture / PR review
| Required 🟢 | Recommended 🟡 | Optional ⚪ |
|---|---|---|
| the `security-audit` **or** `security-review` skill | **Brakeman** + **bundler-audit**, majestic `privacy-reviewer`/`data-integrity-reviewer`, `ie-predictability-reviewer` | Augment (source→sink tracing) |
Scanners give the runnable signal; absent → manual sink-tracing, flagged "scanners not run".

### `/dte-debug` — root-cause bug fix
| Required 🟢 | Recommended 🟡 | Optional ⚪ |
|---|---|---|
| compound-engineering (`ce-debug`) **or** majestic `rails-debugger` | a test framework (repro + regression test), Augment (trace the path) | AppSignal (production error context) |
Needs a runnable repro before and after. No AppSignal → logs/local repro for production bugs.

### `/dte-migrate` — safe upgrade / schema-data migration
| Required 🟢 | Recommended 🟡 | Optional ⚪ |
|---|---|---|
| `dte-arc-plan`/`ce-plan` + `ce-work` | majestic `gemfile-upgrade`, `gemfile-organize`, `database-admin`, `database-optimizer`, `gem-research` | `dte-loop` (run the stepwise plan autonomously) |
Every step reversible + gate-green (incl. migrate↔rollback round-trip). No data migration without a tested rollback.

## Minimum vs full

- **Minimum useful set:** compound-engineering + intent-engineering (+ layered-rails for the Rails
  architecture skills). That unlocks plan/validate/work/review.
- **Full Rails set:** add majestic-rails, rails-testing, ponytail, and a mutation tester (mutant or brutus).
- **Best-in-class:** also Augment (context), cubic (team patterns), MemPalace (history).
