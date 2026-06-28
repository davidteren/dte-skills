# dte-skill-audit — all 18 dte-* skills
_2026-06-28 · tools used: Read, Grep, manual rubric (AGENTS.md "Quality bar" + conventions). Plugin: `claude plugin validate .` ✔ pass · tools unavailable: the `/dte-skill-audit` slash command itself (not hot until restart — recipe run by hand)._

## Verdict
**Suite health: 9/10.** 18 skills, all read conventions first, all define a Done-when bar, all compose
(no reinvented review/plan logic found). **One real routing gap** (`dte-arc-work` has no "Not for" boundary)
and **two minor degrade-section omissions** (`dte-feature`, `dte-migrate`). No bloat, no duplication, no
skill subsumed by another. Manifest count matches STATUS (18). Trigger overlap is well-controlled — the
risky clusters ("review", "audit", "plan/spec/track") are all object-qualified with cross-pointing
"Not for…use Y" clauses, **except `dte-arc-work`**.

## Per-skill dimensions
`Y` pass · `~` weak · `✗` fail. Dimensions: **Cmp** composes·not-reinvent · **Loop** single+list ·
**Deg** degrades-gracefully · **Trig** trigger phrases · **NotFor** sharp boundary · **Conv** reads
conventions first · **Done** done-when bar.

| Skill | Cmp | Loop | Deg | Trig | NotFor | Conv | Done |
|---|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| dte-arc-auditor | Y | ~¹ | Y | Y | Y | Y | Y |
| dte-arc-plan | Y | Y² | ~³ | Y | Y | Y | Y |
| dte-arc-review | Y | Y | Y | Y | Y | Y | Y |
| dte-arc-work | Y | Y | ~⁴ | Y | **✗⁵** | Y | Y |
| dte-debug | Y | Y | Y | Y | Y | Y | Y |
| dte-deep-reviewer | Y | Y | Y | Y | Y | Y | Y |
| dte-feature | Y | Y | **✗⁶** | Y | Y | Y | Y |
| dte-loop | Y | Y | Y | Y | Y | Y | Y |
| dte-migrate | Y | Y | **✗⁶** | Y | Y | Y | Y |
| dte-perf | Y | Y | Y | Y | Y | Y | Y |
| dte-pm | Y | Y | Y | Y | Y | Y | Y |
| dte-pwa | Y | Y | Y | Y | Y | Y | Y |
| dte-security-sweep | Y | Y | Y | Y | Y | Y | Y |
| dte-skill-audit | Y | Y | Y | Y | Y | Y | Y |
| dte-spec | Y | Y | Y | Y | Y | Y | Y |
| dte-test-auditor | Y | Y | Y⁷ | Y | Y | Y | Y |
| dte-tooling-scan | Y | Y | Y⁷ | Y | ~⁸ | Y | Y |
| dte-ux | Y | Y | Y | Y | Y | Y | Y |

¹ Audits **one app** (no "list of apps" loop) — acceptable by design; it loops *internally* over phased sub-areas.
² Loops via decomposition ("Loop over the sub-areas"), not a "list of targets" — correct for its job.
³ No local degrade note; ce-plan/ie-validate fallback is governed by `conventions.md` (ie-plan-assist only). Covered, not stated.
⁴ Degrades the **gate** ("if none, test suite + boot check") but not ce-work absence. Partial.
⁵ **No "Not for…use Y" clause** in the description — the one real boundary gap (see findings).
⁶ No explicit "Degrade gracefully" section (its same-session siblings perf/security/debug/pwa all have one).
⁷ Degrades correctly ("If SimpleCov absent, fall back…" / "If MemPalace unavailable, say so…") — grep missed it (no literal header).
⁸ No "Not for" clause, but the job is unique (history-mining adoption advice) — low collision risk.

## Trigger-overlap matrix (collisions surfaced, not merged)
| Phrase cluster | Skills that fire | Resolved? |
|---|---|---|
| **"review …"** | arc-review(arch) · deep-reviewer(PR/diff) · security-sweep · perf · ux · test-auditor · skill-audit | ✅ all object-qualified + cross-pointed |
| **"audit …"** | arc-auditor(arch) · test-auditor · skill-audit · security-sweep(auth) · arc-review("arc review") | ✅ object-qualified; arc-auditor↔arc-review cross-point |
| **what→how→track** | spec(WHAT) · arc-plan(HOW) · pm(TRACK) | ✅ clean triad, all three cross-reference |
| **"fix this" / "end to end"** | **arc-work** · debug("fix this bug") · feature("ship end to end") | ⚠️ debug+feature point *to* arc-work; arc-work points **back at neither** → bare "fix this"/"end to end" can mis-route |
| **loop/batch** | loop(set up) · arc-work(execute) | ✅ loop says "Not for one task (use arc-work)"; minor one-way asymmetry |

## Findings (severity-ranked)
- 🟠 **`dte-arc-work` description has no "Not for…use Y" boundary.** It's the execution catch-all ("fix this",
  "do the work", "handle this end to end"), so it competes with **dte-debug** (bare "fix this"), **dte-feature**
  ("ship end to end"), and **dte-migrate** (upgrades) — and offers nothing to route *away*. _Fix:_ add to its
  description — *"Not for a pure bug hunt (use dte-debug), a full-stack feature with UI (use dte-feature), or
  an upgrade/data migration (use dte-migrate)."* Edit `skills/dte-arc-work/SKILL.md` frontmatter.
- 🟡 **`dte-feature` + `dte-migrate` lack an explicit "Degrade gracefully" section.** Every other build/review
  skill has one; these lean on composed skills degrading silently. _Fix:_ add a one-line degrade note each
  (feature: "no ui.sh → dte-ux falls back to frontend-design+ie-experience-reviewer"; migrate: "no
  gemfile-upgrade/database-admin → ce-work + manual steps, noted").
- 🟡 **`dte-tooling-scan` has no "Not for" clause.** Low risk (unique job) — add one pointing away from
  `dte-deep-reviewer`/`dte-arc-review` for completeness.
- 🟡 **`dte-arc-plan` degrade is implicit** (conventions-governed). Optional: add a one-line local note.

## What this would replace / overlap
Nothing to remove — no skill is subsumed. The suite is tight; the "majestic 47-skill bloat" failure mode is
**not** present. All findings are wording/boundary fixes, not structural.

## Done
18/18 skills scored on all 7 dimensions + the suite-level overlap/bloat/manifest checks. `claude plugin
validate .` = **pass**. 1 🟠 + 3 🟡 findings, all actionable as small SKILL.md description edits.

## Resolution (v1.4.1)
All 4 findings applied:
- 🟠 `dte-arc-work` — added "Not for…use dte-debug / dte-feature / dte-migrate / dte-loop" boundary.
- 🟡 `dte-feature` + `dte-migrate` — added explicit "Degrade gracefully" sections.
- 🟡 `dte-tooling-scan` — added "Not for" clause (→ deep-reviewer / arc-review / skill-audit).
- 🟡 `dte-arc-plan` — added local degrade note (ce-plan / ie-validate-plan fallbacks).

**Suite health after fixes: 10/10** — every dimension passes across all 18 skills.
