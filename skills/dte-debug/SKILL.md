---
name: dte-debug
description: Track a bug to its ROOT CAUSE and fix it once where all callers route — for a failing test, an exception, a wrong result, or flaky behavior. Composes ce-debug (root-cause hypothesis loop), majestic rails-debugger, Augment for tracing call paths, AppSignal for production error context, and a runnable repro before and after the fix. Use when the user says "debug this", "why is this failing/broken", "fix this bug", "this exception", "find the root cause", or "this test is flaky". Loops over a list of bugs. Not for a broad code review (use dte-deep-reviewer) or a planned feature (use dte-feature).
---

# dte-debug — reproduce → root cause → fix once → prove it

Read `../../references/conventions.md` first (looping, **Runnable gate**, **Verify don't advise**). This
skill **finds the cause, not the symptom**, and fixes it where every caller routes through — the lazy fix
is the root-cause fix. It composes the debug tools; it does not patch the path the ticket names and walk away.

## Input
- A bug: a failing test, a stack trace/exception, a wrong output, or flaky behavior. Get a repro if missing.
- Or a **list** of bugs → handle each (serial by default — bugs in shared code interact).
- Read the project `CLAUDE.md`/`AGENTS.md`; honor interactors-not-services + the test framework.

## Procedure (per bug)
1. **Reproduce first.** Make it fail on demand — a failing test (write the smallest one if none exists), a
   console repro, or the exact request. **No repro → say so and stop guessing.** This same repro proves the
   fix at the end.
2. **Gather context.** Map the failing path with Augment `codebase-retrieval`. If it's a production error,
   pull **AppSignal** (backtrace, frequency, recent deploys, params). Read the actual flow end to end.
3. **Find the root cause.** Run **`ce-debug`** (hypothesis → test → narrow loop) and/or majestic
   **`rails-debugger`**. **Grep every caller** of the function you suspect — the bug usually lives in the
   shared function, and the report only named one symptom. Name the cause in one line before editing.
4. **Fix once, at the cause.** One guard/fix in the shared path beats a guard in every caller — and patching
   only the named path leaves sibling callers broken. Follow existing patterns; interactors not services.
5. **Prove it.** Re-run the step-1 repro → must pass. Run the project **gate** (tests + boot/eager-load
   check) → green. Add/keep a regression test that fails without the fix. A fix you can't demonstrate failing
   before and passing after is **not done**.

## For a list of bugs
One sub-agent per bug through 1–5 (serial by default), then a roll-up noting each bug's root cause, the
shared-path fix, and any sibling callers that were also affected. Report + pause per bug.

## Degrade gracefully
- No ce-debug → drive the hypothesis loop manually (bisect, log, narrow), note it.
- No AppSignal → use logs/local repro for production bugs, note production context unavailable.

## Done when
Each bug has a **runnable repro** that failed before and passes after, the **root cause named** (not just
the symptom), the fix applied **once at the shared path** with all callers checked, a regression test, and
a **green gate**. Surfaced (not silently): any caller you couldn't fix in scope.
