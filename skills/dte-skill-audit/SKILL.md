---
name: dte-skill-audit
description: Audit the dte-* skills themselves against the AGENTS.md quality bar — does each skill compose (not reinvent), loop, degrade gracefully, have sharp non-overlapping triggers, read conventions first, and write a useful doc? Cross-checks trigger overlap across the whole suite and produces a scored report. Read-only — it does not edit skills. Use when the user says "audit the dte skills", "are these skills solid", "skill-audit", "check the suite for overlap/bloat", or after adding several skills. Loops over the whole skills/ directory by default. Not for reviewing application code (use dte-deep-reviewer) or app architecture (use dte-arc-review).
---

# dte-skill-audit — audit the dte-* suite against its own quality bar

Read `../../references/conventions.md` **and** `../../AGENTS.md` ("Quality bar" + "How to ADD a skill")
first — they are the rubric. This skill is the keystone concept turned inward: it checks the `dte-*` skills
the way the skills check code. **Read-only** — it reports, it does not rewrite a SKILL.md.

## Input
- Default target: every `skills/dte-*/SKILL.md` in this repo.
- Or a **list** / single skill name → audit just those.
- Read `AGENTS.md`, `references/conventions.md`, `DEPENDENCIES.md` as the source of truth for what "solid" means.

## Per-skill checks (the quality bar, made runnable where possible)
For each skill, score each dimension pass / weak / fail with evidence (`file:line`):
1. **Composes, doesn't reinvent** — the body *names the tools it calls* (`/ce-plan`, `ie-*`, `ui-*`, …) and
   contains no hand-rolled review/planning *logic*. A procedure step with no named tool is a smell.
2. **Loops** — handles a **single target AND a list** (per the conventions "Looping" rule).
3. **Degrades gracefully** — has a "Degrade gracefully" path naming fallbacks; never a silent skip.
4. **Sharp trigger** — `description` has explicit **trigger phrases** *and* a **"Not for X (use Y)"** clause.
5. **Reads conventions first** — the body opens with the "Read `../../references/conventions.md` first" line.
6. **Writes a useful doc / concrete output** — states where output lands + a "Done when" bar.
7. **Honors owned conventions** — interactors-not-services; no double-firing overlapping tools.

## Suite-level checks (cross-skill)
- **Trigger-overlap matrix.** Compare every pair of descriptions — do two skills fire for the same phrase
  without a "Not for…use Y" boundary? Flag collisions (the routing bugs).
- **DEPENDENCIES coverage.** Every skill has a row in `DEPENDENCIES.md`; every tool it names is listed.
- **Bloat check (the majestic 47-skill lesson).** Any skill that's a thin wrapper or fully subsumed by
  another → recommend merge/removal.
- **Manifest sanity.** `claude plugin validate .` passes; skill count matches STATUS.

## Output
Write `docs/reviews/skill-audit-<date>.md` in the conventions shape:
- **Verdict** — suite health score (0–10) + the count of skills failing any dimension.
- **Per-skill table** — skill × the 7 dimensions, pass/weak/fail with evidence.
- **Trigger-overlap matrix** — collisions surfaced (not merged).
- **Findings** — severity-ranked (🔴 mis-routes/reinvents · 🟠 weak degrade/loop · 🟡 wording), each with
  the fix and a pointer (edit the SKILL.md / edit conventions / update DEPENDENCIES).
- **Tools used / unavailable.**

## Done when
There is a scored audit doc covering every dte-* skill on all 7 dimensions + the suite-level overlap/bloat/
manifest checks, with severity-ranked, actionable findings — and `claude plugin validate .` recorded as pass/fail.
