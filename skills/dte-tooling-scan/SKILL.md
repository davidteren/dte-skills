---
name: dte-tooling-scan
description: Mine session history and recent work to recommend which tools, skills, plugins, or agents to adopt next — and flag overlap with what is already installed. Composes ce-sessions (prior-session synthesis), MemPalace history/knowledge-graph, the overlap logic from the skill-vet design, and light web research for candidates. Use when the user asks "what tools should we add", "scan our history for gaps", "what are we missing", "what would help based on what we do", or "tooling review". Optional loop over time windows or topics. Produces a recommendations doc; it does not install anything.
---

# dte-tooling-scan — history → "what to adopt next" doc

Read `../../references/conventions.md` first. Output is a recommendations doc; **never installs** anything
(installation is a separate, user-confirmed step).

## Input
- Optional: a time window ("last 2 weeks"), a topic ("testing", "review"), or nothing (recent history).
- Optional **loop** over several windows/topics.

## Procedure
1. **Synthesize what we actually do.** Run `/ce-sessions` over recent sessions + query MemPalace
   (`mempalace_search`, `kg_query`, `kg_timeline`) for recurring friction, repeated manual steps, and past
   "I wish we had X" moments. If MemPalace/sessions are unavailable, say so and use the repo's
   `STATUS.md` / git history instead.
2. **Inventory what we own.** List installed plugins/skills/agents (`claude plugin list` + the skill
   catalogue). This is the "already have" baseline.
3. **Find the gaps.** Cross the friction themes (step 1) against the inventory (step 2). A real gap is a
   recurring need with **no owned tool** that covers it. Use light web/Augment research only to name
   candidate tools for a confirmed gap — not to window-shop.
4. **Apply the overlap rule (from skill-vet).** For each candidate, state **what it overlaps / replaces**
   among owned tools. Reject or downgrade anything that mostly duplicates what we have (the majestic
   bloat lesson). Weight by *our* conventions + actual usage, not theoretical coverage.
5. **Write** `docs/reviews/tooling-scan-<date>.md`: ranked recommendations, each with — the friction it
   solves, the candidate, **what it overlaps/replaces**, effort (install vs build), and a clear
   adopt / bookmark / skip call. Plus a short "gaps with no good candidate yet" list.

## Done when
The doc gives a short, ranked, overlap-aware adoption shortlist grounded in what we actually do — not a
catalogue dump. Each item is install-or-build with a one-line justification.
