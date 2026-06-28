---
name: dte-migrate
description: Plan and execute a safe upgrade or migration — a Rails version bump, gem upgrades, or a schema/data migration — incrementally, reversibly, gated at every step. Composes gemfile-upgrade + gemfile-organize (dependency moves), database-admin + database-optimizer (schema/data), gem-research (is an upgrade safe/maintained), ce-plan/ce-work, and the project gate after each step. Use when the user says "upgrade Rails/this gem", "bump dependencies", "migrate the schema/data", "backfill this column", "zero-downtime migration", or "update our dependencies". Loops over a list of upgrades. Not for an architecture/structure migration (use dte-arc-work from a dte-arc-auditor roadmap) or a code bug (use dte-debug).
---

# dte-migrate — safe, incremental, reversible upgrades & data migrations

Read `../../references/conventions.md` first (**Planning is mandatory and validated**, **Runnable gate**,
**Progress: report every phase**). This skill **changes dependencies, schema, or data** — the riskiest,
hardest-to-reverse work. So: one change per step, the gate green before each commit, reversible by design.
It composes the upgrade/DB tools; it does not free-hand a mass version bump.

## Input
- A migration: a Rails/Ruby version bump, a set of gem upgrades, or a schema/data migration (add/rename
  column, backfill, change type, move data).
- Or a **list** → handle each (serial by default — dependency/schema changes are not independent).
- Read the project `CLAUDE.md`/`AGENTS.md`; note the DB, deploy model, and whether deploys are test-gated.

## Establish the gate FIRST
Confirm the runnable gate (full test suite + boot/`zeitwerk:check`; for DB work add a migrate↔rollback
round-trip and a seed/load check) is **green on the current tree** before touching anything. It runs before
every commit; a red gate after a step = **revert that step**, never push through.

## Procedure (per migration)
1. **Research safety first.** For an upgrade, run **`gem-research`** — is the target version maintained,
   what breaks, what's the changelog/deprecation path? For a Rails bump, identify the upgrade guide steps.
   Don't bump blind.
2. **Plan it (validated).** Run **`dte-arc-plan`** / `ce-plan` → a validated, **stepwise** plan: smallest
   reversible increments (one major gem at a time; dual-write/backfill/verify/cutover for data; deprecation
   warnings cleared before the bump). Every step has its own rollback.
3. **Execute step by step** via `ce-work`:
   - **Deps:** **`gemfile-upgrade`** for the bump, **`gemfile-organize`** to keep the Gemfile clean;
     `bundle update <gem>` one at a time, run the gate, fix deprecations.
   - **Schema/data:** **`database-admin`** for the migration (reversible `up`/`down`, safe for large tables —
     add column nullable → backfill in batches → add constraint), **`database-optimizer`** to check the new
     index/query shape. Keep migrations **idempotent and reversible**.
4. **Gate after EVERY step.** Run the gate (incl. migrate→rollback→migrate round-trip for DB steps). Green →
   commit + **report the step**. Red → **revert the step, log why, stop.**
5. **Report + pause** per step — what changed, gate result, what's next, the rollback if it goes wrong.

## For a list of migrations
Serial; or hand the stepwise plan/INDEX to **`dte-loop`** to run gated + autonomous. One branch per
migration unless coupled. Report + pause per step.

## Degrade gracefully
- No `gemfile-upgrade`/`gemfile-organize` → bump with `bundle update <gem>` + manual Gemfile edits, noted.
- No `database-admin`/`database-optimizer` → hand-write reversible migrations + `EXPLAIN` by hand, noted.
- No `gem-research` → check the changelog/release notes manually; never bump blind regardless.

## Guardrails
- **Reversible or it doesn't ship.** No data migration without a tested rollback / backfill-verify.
- Stop-and-ask before anything irreversible (a destructive `down`, dropping a column, prod data change)
  unless explicitly authorized. Never run a big bump as one commit.

## Done when
The migration is done in **small reversible steps**, each **gate-green** (incl. rollback round-trip for DB),
deprecations cleared, dependencies clean, with a step-by-step report and the rollback path documented.
