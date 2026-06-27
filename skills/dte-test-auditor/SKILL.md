---
name: dte-test-auditor
description: Audit a test suite for real value, not just presence — coverage gaps AND test quality (weak assertions, over-mocking, implementation-coupled/brittle tests, missing edge/error/integration paths), plus mutation testing where available to prove tests actually fail when code breaks. Composes SimpleCov, rails-testing (Minitest), the ce-code-review testing persona, and mutant. Use when the user says "audit the tests", "are these tests any good", "check test coverage/quality", "do the tests actually test anything", or after a feature lands. Loops over a list of files/modules. Not for writing new tests (use minitest-coder / rails-testing) — this judges existing ones.
---

# dte-test-auditor — test value, quality & coverage → findings doc

Read `../../references/conventions.md` first. Judges existing tests; it does not write them (hand fixes
to `rails-testing` / majestic `minitest-coder` via `dte-arc-work`).

## Why this exists
Coverage % lies. A line can be "covered" by a test that asserts nothing real. This audit checks three
things: **is it covered, is the test good, and does it actually catch a bug if one is introduced.**

## Input
- A target: a module/area, specific test files, or the whole suite.
- Or a **list** of files/modules → audit each (parallel; `--serial` to slow it down).

## Procedure (per target)
1. **Coverage.** Run the suite under **SimpleCov** (if present) → uncovered lines/branches for the target.
   If SimpleCov is absent, note it and fall back to mapping tests→code by name/reference.
2. **Quality lens** (read the tests, don't just count them) — flag:
   - **Weak assertions** — `assert true`, asserting a mock was called and nothing else, snapshot-only.
   - **Over-mocking** — every collaborator stubbed, so the test proves logic *in isolation* but nothing
     about the real interaction. Require ≥1 integration test through the real callback/middleware chain.
   - **Implementation-coupled / brittle** — asserting CSS classes, internal call order, private methods,
     exact prose. Prefer semantic / `data-test-id` / behavioural assertions.
   - **Missing categories** — for each feature-bearing unit, check happy-path + edge + error/failure +
     integration. Name the specific missing case.
   Use the `/ce-code-review` testing persona + `rails-testing` idioms (Minitest: `assert_difference`,
   `assert_enqueued_with`, `assert_broadcast_on`, system tests) as the quality bar.
3. **Mutation (the real signal).** If **mutant** is set up, run it on the target → **survivors** =
   "this code can change/break and no test notices." List each survivor with the mutated operator. If
   mutant is not set up, **flag the audit as coverage+quality-only** and recommend wiring it (don't fake it).
4. **Synthesize** → `docs/reviews/test-audit-<target>-<date>.md`: coverage holes, quality defects
   (severity-ranked, `file:line`), and mutation survivors. Each finding names the missing/weak case and
   the concrete test to add. Apply owned-convention (the project's chosen test framework only — don't
   recommend RSpec into a Minitest suite).

## Done when
The doc tells the user, per target: what's untested, which existing tests are theatre, and (if mutant ran)
exactly which code is unguarded — ranked, with the specific tests to add.
