---
name: dte-security-sweep
description: Security review of a Rails app, a PR/diff, or a slice — find real vulnerabilities (authz gaps, injection, mass-assignment, secrets, unsafe deserialization, SSRF, CSRF, privacy/PII leaks) with evidence and ranked fixes. Composes the security-audit (whole-app posture) and security-review (PR-scoped) skills, majestic privacy-reviewer + data-integrity-reviewer, intent-engineering's predictability lens, and Brakeman/bundler-audit if present. Use when the user says "security review", "security sweep", "is this safe", "check for vulnerabilities", "audit auth/authorization", "scan for secrets/PII", or before shipping a sensitive change. Loops over a list of slices/PRs. Not for general correctness review (use dte-deep-reviewer) or architecture (use dte-arc-review).
---

# dte-security-sweep — security posture / PR review → ranked, evidence-backed findings

Read `../../references/conventions.md` first (tool detection, looping, output shape, **Verify don't
advise**). This skill conducts the existing security skills + scanners; it does **not** invent its own
vuln taxonomy. Findings are evidence-backed — a real sink/source path or a scanner hit, not vibes.

## Input
- A target: the whole app (posture), a **PR/branch/diff** (scoped), or a slice (auth, a controller, a job).
- Or a **list** → sweep each (parallel sub-agents by default; `--serial` for a focused audit).
- Read the project `CLAUDE.md`/`AGENTS.md`; note auth library (Devise? action-policy?), and trust boundaries.

## Procedure (per target)
1. **Pick the right lens by scope.**
   - **Whole app** → run the **`security-audit`** skill (posture across auth, sessions, config, deps).
   - **PR/diff** → run the **`security-review`** skill (scoped to the change + its blast radius).
2. **Run the scanners (runnable signal first).** **Brakeman** (`brakeman -q`) for Rails-specific sinks,
   **bundler-audit**/`bundle audit` for vulnerable gems, secret-scan the diff (high-entropy strings, keys).
   Prefer a scanner hit with a line number over a prose suspicion. State which scanners weren't installed.
3. **Run the review lenses.** majestic **`privacy-reviewer`** (PII/data exposure, logging of sensitive
   fields) + **`data-integrity-reviewer`**, and **`ie-predictability-reviewer`** (silent failures, auth
   that doesn't fail closed, hidden side effects on a trust boundary).
4. **Trace each finding.** Confirm a real **source → sink** path (user input reaching SQL/HTML/system/SSRF)
   or a real missing authorization check — grep all callers, don't patch one path and leave siblings open.
   Mark heuristic findings "suspected".
5. **Rank by exploitability × impact.** 🔴 exploitable now · 🟠 conditional · 🟡 hardening. Each: what ·
   where (`file:line`) · the path/evidence · the fix (fail-closed, allowlist, strong params, escape, rotate).

## For a list of targets
One sub-agent per target through 1–5, then a roll-up `docs/reviews/security-rollup-<date>.md` ranking by
exploitability, with the must-fix-before-ship items at the top and a "tools/scanners unavailable" note.

## Degrade gracefully
- No Brakeman/bundler-audit → fall back to the review skills + manual sink tracing; flag scanners not run.
- No security-audit/security-review skills → use ie-predictability + privacy-reviewer + Brakeman, note it.

## Done when
The target has a security doc with severity-ranked, **traced** findings (source→sink or missing-authz
evidence at `file:line`), scanner results (or "not run"), tools used/unavailable, and a clear must-fix list
gating the ship — pointer to `dte-arc-plan`/`dte-arc-work` to remediate.
