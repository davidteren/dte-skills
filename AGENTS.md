# AGENTS.md — developing & maintaining dte-skills

Agentic guidance for **building, editing, and updating the `dte-*` skills**. Skill *usage* lives in each
`skills/<name>/SKILL.md`; this file is the meta-workflow that keeps them solid as the suite grows.

If you change a skill, read this first. The rules here are not style — they are the reasons the skills
work. They were learned the hard way (see **Lessons** below).

---

## What this repo is

A Claude Code **plugin** (`dte-skills`) of orchestration skills that **compose** the user's existing
AI-engineering tools — intent-engineering, compound-engineering, layered-rails, majestic, ponytail,
cubic, MemPalace, Augment — into disciplined `review → plan → work → audit` flows. The skills are
**recipes that conduct other tools**, not reimplementations.

Layout:
```
.claude-plugin/{plugin.json, marketplace.json}   # plugin + local-marketplace manifests (root layout, source "./")
skills/dte-*/SKILL.md                            # one recipe per skill
references/conventions.md                        # the shared rules EVERY skill obeys (source of truth)
README.md · DEPENDENCIES.md · docs/index.html    # public docs + GitHub Pages site
STATUS.md                                        # the project tracker (shipped + backlog + decisions)
```

---

## Core philosophy (do not regress these)

1. **Compose, don't reinvent.** A skill invokes `/ce-plan`, `/ie-validate-plan`, `/layered-rails`, the
   ui-*/frontend-design skills, etc. If you find yourself writing review or planning *logic* into a
   SKILL.md, stop — call the tool that already does it.
2. **Plan, then validate.** Real work is planned with `ce-plan` and validated with `ie-validate-plan`
   (+ other validators) before execution. Validation is a **separate pass** from drafting.
3. **Verify, don't trust.** A plan's load-bearing claims get checked against the code, not assumed.
4. **Gate + report every step.** A runnable gate before every commit; a loud report + pause after every
   phase. No dark loops, no silent skips.
5. **Loop at scale.** Big task → many small validated plans → completed one by one.
6. **Degrade gracefully.** Optional tools used if present, their absence stated. Never silently skip.
7. **Honor the target project's `CLAUDE.md`/`AGENTS.md`.** Local conventions (e.g. interactors-not-services)
   win over generic defaults.

The full, authoritative version is `references/conventions.md`. **Every skill reads it first.** When you
add a cross-cutting rule, put it there — not copy-pasted into each skill.

---

## How to ADD a skill

1. **Earn it.** A new skill must be a *distinct job* not covered by an existing one. Map it against the
   suite (see STATUS / README table) — if it overlaps, extend an existing skill instead.
2. **Create `skills/dte-<name>/SKILL.md`** with YAML frontmatter:
   - `name: dte-<name>` (keep the `dte-` prefix; `dte-arc-*` for the review/plan/work arc).
   - `description:` — what it does + **trigger phrases** + a **"Not for X (use Y)"** clause so it doesn't
     fire over a sibling. This is how Claude routes to it; invest in it.
3. **Body = a recipe**, in this shape (mirror an existing skill):
   - "Read `../../references/conventions.md` first."
   - **Input** (single target **and** a list — every skill loops).
   - **Procedure** — numbered steps that *name the tools they compose* and their fallbacks.
   - **Output** — `docs/reviews/` (findings) or `docs/plans/` (plans), in the conventions output shape.
   - **Done when** — a concrete completion bar.
4. **Compose, loop, degrade, honor-conventions** — all four, every skill.
5. **Validate + install:** `claude plugin validate .` → bump version (below) → `marketplace update` +
   `plugin update`. Test the trigger by describing a task and confirming the right skill fires.

## How to EDIT a skill safely

- Don't break the **compose contract** — if you change which tool a skill calls, update its row in
  `DEPENDENCIES.md` and any cross-reference in other skills (e.g. `dte-arc-work` lists what it composes).
- Keep the **description triggers** accurate — a stale trigger mis-routes.
- Re-validate + reinstall. Skills activate on the **next Claude Code restart**.
- Prefer editing `references/conventions.md` for cross-cutting changes over touching seven files.

## Versioning & release

SemVer in **both** `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json` (kept in sync).
- **Patch** — wording/fix. **Minor** — new skill or new capability. **Major** — breaking rename/removal.
Release: `claude plugin validate .` → bump both manifests → commit + push → consumers
`claude plugin update dte-skills@dte-skills-marketplace` (restart to apply).

---

## Lessons (the WHY — don't let edits regress these)

Learned from analysing a real one-off "refactor my app" workflow (the ongela session). Each lesson is
already encoded as a rule above; keep it that way:

| What went wrong in the bespoke workflow | The rule it produced |
|---|---|
| Ran dark — the human asked "how far are we?" four times | **report + pause every phase** |
| Critiqued itself in the same context/model | **validation is a separate pass** (ie-validate-plan, cross-model) |
| Asserted facts ("10 already X") nobody re-checked | **verify claims against the code** |
| Reinvented planning + critique from scratch | **compose ce-plan / ie-validate-plan, don't reinvent** |
| Granularity fork surfaced late, mid-loop | **surface key forks early (AskUserQuestion)** |
| No safety net but a green boot (deploy ungated) | **runnable gate before every commit; red = revert** |

## Quality bar — "keep them solid"

A skill is solid when: it composes (doesn't reinvent), loops, degrades gracefully, has sharp triggers
(fires for its job, not a sibling's), and writes a useful doc. A dedicated **skill-audit** to check this
mechanically across the suite is on the roadmap (`STATUS.md`) — until then, audit by hand against this list.

## Conventions specific to these projects
- **interactors-not-services** — never recommend service objects; recommend interactors.
- Don't double-fire overlapping installed skills (several Minitest skills exist); pick one by precedence
  and say which. See `references/conventions.md`.
