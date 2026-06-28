---
name: dte-perf
description: Performance review of a Rails app, endpoint, query, job, or front-end page — find the real bottlenecks (N+1s, missing indexes, slow queries, fat allocations, slow front-end paint/bundle) with measured evidence, and propose ranked fixes. Composes majestic performance-reviewer + database-optimizer + database-admin, AppSignal production data, chrome-devtools performance traces + Lighthouse, and Augment for context. Use when the user says "it's slow", "find the bottleneck", "perf review", "optimize this query/endpoint/page", "why is this N+1", or "check performance". Loops over a list of endpoints/queries/pages. Not for general architecture (use dte-arc-review) or fixing without measuring (this measures first).
---

# dte-perf — measured performance review → ranked, evidence-backed fixes

Read `../../references/conventions.md` first (tool detection, looping, output shape, **Verify don't
advise**). This skill **measures before it advises**: a perf finding without a number is a guess. It
conducts the perf reviewers + profilers; it does not blindly rewrite code.

## Input
- A target: an endpoint, a query, a background job, a model hotspot, or a front-end page/route.
- Or a **list** → handle each (parallel sub-agents by default; `--serial` to be gentle on a shared DB).
- Read the project `CLAUDE.md`/`AGENTS.md`; note the DB, caching (Solid Cache?), and front-end stack.

## Procedure (per target)
1. **Get real numbers first.** Prefer measured signal over reading code:
   - **Back-end:** **AppSignal** MCP (slow transactions, query time, throughput, error rate) if connected;
     else logs / `EXPLAIN ANALYZE` / a benchmark. Map the slow path with Augment.
   - **Front-end:** chrome-devtools **`performance_start_trace`** + **`performance_analyze_insight`** and a
     **`lighthouse_audit`** (LCP, CLS, TBT, bundle size).
   State the baseline numbers. If nothing measurable is available, say so and mark findings "suspected".
2. **Run the perf lenses.** majestic **`performance-reviewer`** (N+1, eager-load gaps, allocations, caching
   opportunities), **`database-optimizer`** (slow queries, missing/unused indexes, query shape), and
   **`database-admin`** for index/schema actions. For FE: bundle/asset weight, render-blocking, image perf.
3. **Attribute cost.** Tie each finding to a number from step 1 (this query = X ms × N calls; this route's
   LCP = Y s). No number → mark "suspected (heuristic)", don't claim a win you can't measure.
4. **Rank by impact.** 🔴 measured hot path · 🟠 likely · 🟡 micro. Each: what · where (`file:line`) · the
   measured cost · the fix (and its expected delta). Interactors-not-services if extraction is suggested.

## For a list of targets
One sub-agent per target through 1–4, then a roll-up `docs/reviews/perf-rollup-<date>.md` ranking the
worst offenders by measured cost, with the cheapest-high-impact fixes at the top.

## Degrade gracefully
- No AppSignal → use EXPLAIN/logs/benchmarks, note production data unavailable.
- No chrome-devtools → skip FE traces, mark FE findings heuristic.
- No mutation/load tooling → don't claim throughput numbers you didn't measure.

## Done when
The target has a perf doc with **baseline numbers**, severity-ranked findings each tied to a measured (or
explicitly-suspected) cost and a fix with an expected delta, tools used/unavailable, and a pointer to
`dte-arc-plan`/`dte-arc-work` to apply the top fixes.
