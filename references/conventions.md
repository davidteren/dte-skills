# dte-* shared conventions

Every `dte-*` skill follows these. Read this once; each skill assumes it.

## Compose, don't reinvent
A `dte-*` skill orchestrates existing tools. It does **not** re-implement review or planning logic.
Prefer invoking a skill (`/ce-code-review`, `/ie-review`, `/layered-rails ...`) over hand-rolling.

## Tool availability — detect and degrade (never silent)
Optional tools may be absent in a given project/session. Detect, use if present, and **state in the
output** when one was unavailable. Never silently skip a step.

| Capability | Primary | Fallback if absent |
|---|---|---|
| Context retrieval | Augment `codebase-retrieval` | cubic `codebase-context`/`wiki`; else Grep/Glob + read |
| Rails code review | `/ce-code-review` (cross-model) | `/ie-review`; majestic `review:rails-code-review` |
| Architecture | `/layered-rails` analyzers + `layered-rails-reviewer` | `/ie-audit` (architecture lens) |
| Planning | `/ce-plan` + `/ie-validate-plan` | `/ie-plan-assist` only |
| Tests | `rails-testing` + SimpleCov | majestic `minitest-coder`; coverage skipped w/ note |
| Mutation testing | `mutant` | skip + flag "mutation not run (mutant not set up)" |
| History/insights | MemPalace `search`/`kg`, `/ce-sessions` | skip + note |
| Elegance pass | `ponytail` / `/ce-simplify-code` | `ie-simplicity-reviewer` |

> **3.15.0 note:** the standalone `ce-*-reviewer` agents no longer exist — those reviewers are personas
> inside `/ce-code-review`. Invoke the skill, not the old agents.

## Looping
Every skill accepts a **single target** or a **list**. For a list:
- Independent items → dispatch **parallel sub-agents** (default), one per item; accumulate + roll-up.
- `--serial` → one at a time (cheaper, slower).
- A list of 20+ items → consider the Workflow engine for fan-out; otherwise batch the sub-agents.

## Owned-convention awareness
- Honor the project's `CLAUDE.md`/`AGENTS.md`. For these projects: **interactors-not-services** — never
  recommend "extract a service object"; recommend an interactor.
- **Don't double-fire overlapping skills.** Several testing skills exist (`rails-testing`, majestic
  `minitest-coder`, baseline `ce-dhh` testing) and two `layered-rails`. Pick **one** per job by this
  precedence: project-local skill > `rails-testing`/`layered-rails` (palkan) > majestic. Say which you used.

## Front-end lens (reviews must not be back-end-only)
A review that only runs back-end/architecture lenses misses half of a front-end change. **Any review skill
must detect front-end diffs and pull the front-end lens in:**
- **Detect FE files** in the target/diff: `.erb`/`.haml`/`.slim` views, `app/components` (ViewComponent),
  JS/TS, Stimulus controllers, `*.css`/Tailwind, `app/javascript`, asset config.
- **When present, pull in:** the **ui.sh** skills + **frontend-design** (look-and-feel, intentional vs
  templated) + **`ie-experience-reviewer`** (interaction states loading/empty/error/disabled, keyboard/
  focus/back-button, accessibility, information architecture). For a heavier pass hand the slice to **`dte-ux`**.
- **Run the deterministic checkers** when the diff touches Stimulus/Turbo/Hotwire Native — the
  **hotwire-rails-toolkit** linters (`lint_stimulus`, `lint_turbo_streams`, `lint_morphing`,
  `lint_bridge_contract`) catch the failures that produce no error (undeclared Values/Targets, a `connect()`
  with no `disconnect()` cleanup, a stream channel that broadcasts without authorizing, a bridge payload key
  dropped on one native side). They're a fast, exact pre-pass that grounds the LLM lens above; feed their
  output into the same findings doc. Skip with a note if the plugin isn't installed (Optional).
- No FE files in scope → state "no front-end changes detected" and skip (don't fabricate FE findings).

## Output shape (findings + plans)
Findings → `docs/reviews/<skill>-<target>-<YYYY-MM-DD>.md`. Plans → `docs/plans/` (ce-plan owns the name).
Findings docs open with:

```
# <skill> — <target>
_Date · tools used · tools unavailable_

## Verdict        (one line + a 0–10 health score where it makes sense)
## Findings       (severity-ranked: 🔴 high / 🟠 med / 🟡 low; each: what · where (file:line) · why · fix)
## Confirmed vs disputed   (where lenses disagreed — don't merge, surface)
## What this would replace / overlap   (so adoption stays honest)
```

## Verify, don't advise
Where a check can be made runnable (coverage %, mutation survivors, grep-based pattern wiring), prefer
the runnable signal over prose. Report "suspected" for heuristic findings; reserve hard claims for
single-line-decidable ones.

## Planning is mandatory and validated (ways-of-working — non-negotiable)
Once analysis is done and **real work** will follow, you do not free-hand the work:
1. **Every plan is created with `/ce-plan`** (compound-engineering). Never hand-roll a plan when ce-plan
   is available — it is the planning engine.
2. **Every plan is validated before it is "done"** — run `/ie-validate-plan` (intent-engineering) and
   any other validators in the library that apply (e.g. a cross-model `/ce-code-review` read of the
   plan, `layered-rails-planner` for architecture). **Fold the gaps back in; a plan with unresolved
   validation gaps is not ready to execute.**
3. Validation is a **separate pass** from drafting — different lens, ideally different model. Do not
   treat the same workflow that wrote the plan as its own critic.

## Decomposition: a massive task → many small plans (then complete many)
A big task is not one giant plan. **Break it down and loop:**
1. Decompose the task into independently-shippable sub-areas (by slice/pack/concern/dependency).
2. **For each sub-area, loop:** `/ce-plan` → validate (above) → a small plan in `docs/plans/`.
3. Emit a **plan index / worklist** — `docs/plans/INDEX-<topic>.md` — listing every plan, ordered by
   dependency, with a checkbox each (the worklist `dte-arc-work` consumes).
4. `dte-arc-work` then **completes many** of them in a loop (gate + report per plan; see below).
This is first-class: `dte-arc-plan` can produce *one* plan or a *batch*; `dte-arc-work` can complete
*one* or the *whole index*.

## Progress: report every phase (no dark loops)
The #1 failure of a bespoke loop is running dark — the human asking "how far are we?". Every looping
skill **reports after each item/phase** (what it did, the gate result, what's next) and, by default,
**pauses for the go-ahead** between phases. Never run a long loop silently.

## Runnable gate before each commit (in a loop)
When a project defines a gate (tests, boot/zeitwerk check, lint, packwerk validate), **derive and run
it before every commit in a loop**, and treat a red gate as **revert-the-phase**, not patch-around-it.
Detect the gate from the project (CI config, the plan's stated gate, `bin/rails`/`bin/rake` tasks); if
none exists, at minimum run the test suite + a boot/eager-load check. The gate is the real safety net.

## Surface key forks early
Decisions that materially change the work (granularity, scope, approach) go to the user via
AskUserQuestion **before** deep work — not discovered mid-loop. ce-plan's scoping synthesis does this;
honor it.
