---
name: dte-loop
description: Set up an autonomous, gated loop over a batch of work — turn "I have many plans to implement / reviews to run / bugs to fix" into a worklist doc the loop iterates, with every predictable decision asked UP FRONT, then emit the exact command to trigger it. Each item runs on its own branch, sequentially, gated, with approve→merge criteria you set. Composes the built-in /loop, dte-arc-plan (INDEX worklist), dte-arc-work (gated executor), and dte-deep-reviewer/dte-arc-review for review batches. Use when the user says "loop over these", "run these autonomously", "implement all these plans each on its own branch", "set up a loop", "batch these reviews and merge the approved ones", or "what command do I run to loop this". Not for doing one task end-to-end (use dte-arc-work) or building one plan (use dte-arc-plan).
---

# dte-loop — build the worklist, front-load the questions, hand back the command

Read `../../references/conventions.md` first — especially **Decomposition**, **Surface key forks early**,
**Runnable gate**, and **Progress: report every phase**. This skill is the **front door to an autonomous
loop**: it produces the reference doc the loop iterates and the command you paste to start it. It does
**not** silently start changing code — it sets up, asks everything it can predict, then hands you the trigger.

The point: a long autonomous loop must not run dark and must not stall mid-way asking a question it could
have asked at the start. So this skill **moves every foreseeable decision to the front**, records the
defaults it had to assume, and only then loops.

## Input
- A **batch** of work, in any of these shapes:
  - **plans** — a `docs/plans/INDEX-<topic>.md`, a folder of plans, or "plan + implement these N things".
  - **reviews** — "review these PRs/branches/slices; merge the ones that meet <criteria>".
  - **tasks/bugs** — a list of fixes/features to grind through.
- Read the project `CLAUDE.md`/`AGENTS.md` first (gate, branch naming, interactors-not-services, test framework).

## Procedure

### 1. Classify the batch & confirm the loop contract
Decide which kind of loop this is (plan-impl / review-merge / task / bug). State it back in one line.
The contract for every kind: **one item per branch, serial by default** (shared codebase → no merge/index
races), **gate before each commit**, **report after each item**.

### 2. Build the worklist doc (the thing /loop iterates)
This is the reference the loop reads each pass. **Don't reinvent it where one already exists:**
- **plan-impl** → if there's no INDEX yet, call **`dte-arc-plan`** to decompose + validate the batch into
  `docs/plans/INDEX-<topic>.md` (each plan ce-plan-built + ie-validate-plan-validated). Reuse it as the worklist.
- **review-merge / task / bug** → write `docs/loops/INDEX-<topic>.md` in the worklist shape below.

```
# Loop worklist — <topic>
type: plan-impl | review-merge | task | bug
gate: <the runnable gate command>           # e.g. bin/rails test && bin/rails zeitwerk:check
branch: <pattern>                            # e.g. feat/<id>-<slug>
merge-policy: manual | auto-if(<criteria>)   # review-merge only; auto-merge is opt-in + outward-facing
serial: true
decisions: see "Assumed defaults" below

- [ ] <id> · <target> · status:todo · branch:- · result:-
- [ ] <id> · <target> · status:todo · branch:- · result:-
```
Order items by dependency. Every item independently shippable.

### 3. Front-load EVERY predictable question (the whole point)
Before emitting the command, surface — via **AskUserQuestion** — every fork the loop would otherwise hit
mid-run. Predict them from the batch:
- **granularity / scope** (one plan or many; how big a slice per branch),
- **gate** (which command counts as green; if none, default test-suite + boot/eager-load check),
- **branch + commit policy** (`--ship` auto-commit/PR, or stop-for-review per item),
- **review-merge criteria** (what "approved" means: all CI green + N reviewers + no 🔴 findings) and
  **whether merge is automatic** — auto-merge is irreversible/outward-facing, so it is **opt-in and
  explicitly confirmed here**, never assumed,
- **stop conditions** (max items, stop on first red gate, time/iteration budget).
Ask these as ONE batched set of questions up front.

### 4. Record assumed defaults (after-the-fact safety valve)
For anything genuinely unforeseeable at setup, the loop proceeds with the **recommended option** and
**logs it** — it does not stall. Seed that log now: an **"Assumed defaults"** section in the worklist doc
listing each decision taken and the chosen default, so the user can **retroactively override** any wrong
call. Format: `- <decision> → went with <default> (recommended). Override: <how>.`

### 5. Emit the trigger command (don't auto-start)
Hand the user the exact command and stop. Two forms — recommend the first:
- **Direct (serial, gated, reports each item):**
  `/dte-arc-work docs/plans/INDEX-<topic>.md --ship`  ← dte-arc-work already loops the INDEX, one branch +
  gate + report per item. Best when you're at the keyboard.
- **Resilient / unattended (built-in /loop wrapper, self-paced, survives a hang):**
  `/loop /dte-arc-work docs/loops/INDEX-<topic>.md --ship` ← re-fires until every box is ticked.
For a **review-merge** batch the inner command is `/dte-deep-reviewer` (or `/dte-arc-review`) per item, with
merge gated on the step-3 criteria.

Print: the worklist path, the command, the assumed-defaults summary, and the stop conditions. **Then stop** —
the user pastes the command to begin. (If the user said "and run it", proceed — but only after the front-loaded
questions are answered.)

## For the loop itself (what the emitted command does)
`dte-arc-work` owns execution: take the first unchecked dependency-ready item → branch → work/review →
verify → **gate** → on green tick the box + report + (next or pause per policy); on red **revert the item,
log, stop**. dte-loop's job is done once the worklist + command exist.

## Degrade gracefully
- No `dte-arc-plan` → build the worklist by hand from the user's list (note plans are unvalidated).
- No `/loop` skill → emit the direct `/dte-arc-work` command only, note resilience wrapper unavailable.

## Done when
There is a worklist doc (ordered, gated, with merge policy + assumed-defaults log), every predictable
decision has been asked up front, and the user has the exact one-line command to start the autonomous loop —
with stop conditions stated. No code changed by this skill.
