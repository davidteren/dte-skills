---
name: dte-pm
description: Project-management — turn a goal, spec, or plan index into TRACKED, broken-down, sequenced work (GitHub issues/tickets) with status and dependency order. Composes ce-plan/ce-brainstorm to decompose, dt-create-issue (or gh issue create) to file each ticket with the plain-language What/Why/How body + labels, and produces a tracking index. Use when the user says "track this", "create issues/tickets for this", "break this into tickets", "project-manage this", "file the backlog", "sequence this work", or "what's the status/order". Loops over a list of goals. Not for defining the what (use dte-spec), the implementation plan (use dte-arc-plan), or doing the work (use dte-arc-work).
---

# dte-pm — goal → tracked, sequenced issues

Read `../../references/conventions.md` first (looping, output shape). This skill turns work into a **tracked
backlog** — it owns *decomposition into tickets, sequencing, and status*, not the spec (that's `dte-spec`)
nor the implementation plan (that's `dte-arc-plan`). It composes the planner + the issue tool; it does not
invent a tracker.

## Input
- A goal/epic, a `dte-spec` output, a `dte-arc-plan` INDEX, or a rough list of work.
- Or a **list** of goals → handle each (serial by default — issue numbers + ordering interact).
- Read the project `CLAUDE.md`/`AGENTS.md`: the tracker in use, the **label taxonomy**, the current phase
  label, and the **plain-language What/Why/How ticket convention** (every ticket opens with it).

## Procedure (per goal)
1. **Decompose into shippable units.** Use **`ce-plan`** (or `ce-brainstorm` for a fuzzy goal, or reuse an
   existing INDEX/spec) to break the goal into independently-trackable work items. Don't file one giant
   ticket — file the units someone can actually pick up.
2. **Sequence.** Order the items by dependency; assign milestone/phase labels. Mark what blocks what.
3. **File each ticket.** Prefer the project's **`dt-create-issue`** if present (it already enforces the
   What/Why/How body + label taxonomy + phase); else **`gh issue create`** with a body that **opens with the
   plain-language What / Why / How** (a non-technical PM could follow it), then the technical detail below,
   plus labels + the dependency note. Never ship a bare/purely-technical ticket.
4. **Emit a tracking index.** Write `docs/pm/INDEX-<goal>-<date>.md`: every issue (number + link), its
   status, milestone, dependency order, and the critical path. This is the human-readable backlog board.
5. **Hand off.** Point at `dte-arc-plan` (plan the issues) → `dte-arc-work`/`dte-loop` (build them). The
   index doubles as a `dte-loop` worklist source.

## For a list of goals
One sub-agent per goal through 1–4 (serial), then a combined `docs/pm/INDEX-<batch>-<date>.md` with the
cross-goal sequence + critical path.

## Degrade gracefully
- No `gh`/GitHub access → write the backlog as a markdown index only, note tickets weren't filed.
- No `ce-plan` → decompose by hand from the spec/goal, note the breakdown is unvalidated.
- No `dt-create-issue` → use `gh issue create` and apply the What/Why/How + labels manually.

## Done when
The goal is broken into sequenced, tracked tickets (each opening with plain-language What/Why/How + labels),
with a tracking index showing status + dependency order + critical path, and a pointer to `dte-arc-plan`/
`dte-loop` to execute.
