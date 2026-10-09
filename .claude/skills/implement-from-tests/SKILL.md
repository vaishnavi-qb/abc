---
name: implement-from-tests
description: >-
  Step 2 of the test-first pipeline. Given a ticket plus its already-committed,
  failing tests, write the implementation that makes those frozen tests pass —
  without ever modifying the tests. Use only AFTER a human has reviewed,
  confirmed RED, and committed the tests produced by the `draft-tests` skill.
---

# implement-from-tests — make the frozen tests pass

You are running **step 2 of a two-skill, test-first pipeline**. The tests are
already written, reviewed by a human, confirmed failing (RED), and committed as
a diff anchor. Your job is to write the **implementation** that turns them green
— and nothing that touches the tests.

## Inputs

Ask the user for both before doing anything else:

- **Ticket context.** Read `ticket.md` from the repository root. Use it for
  intent the tests don't spell out literally — not as licence to add untested
  behaviour. If `ticket.md` is missing or unclear, ask the user to clarify
  before proceeding.
- **A reference to the test-only commit.** The human commits the reviewed tests
  before invoking this skill and provides a ref — anything from a full hash
  (`a3f9c12`) to a shorthand like `"latest commit"` or `HEAD`. Resolve it:
  - `"latest commit"` / `HEAD` → use `HEAD`
  - anything else → treat it as a git ref as-is
    Then run:
  ```bash
  git show <ref> --stat
  ```
  to see exactly which test files were added or changed in that commit, then read
  those files. This is the authoritative frozen contract — do not look for tests
  anywhere else.

## Procedure

1. **Confirm RED first.** Extract the test files from the commit and run only
   those — not the full suite. This is fast and proves you are implementing
   against a real failing contract.

   ```bash
   git diff <ref>^..<ref> --name-only   # lists changed files
   ```

   Then run:
   - BE: inspect `ecommerce-backend/package.json` for the test script and run it
     scoped to the target file (e.g. `cd ecommerce-backend && npm test -- <test-file>`)
   - FE: `cd ecommerce-frontend && npm run test -- <test-file>`

2. **Propose a short plan, then wait.** Before editing anything, lay out a brief
   plan and hand it back for a quick OK:
   - which file(s) you'll change and the approach in a sentence or two,
   - the order you'll make the failing tests pass.
     Keep it proportional — a couple of lines for a small ticket; don't restate the
     tests. Pause here so the human can redirect _before_ code is written; for a
     trivial change they'll just wave it through.

3. **Implement in the smallest steps** that make failing tests pass, one at a
   time, following the agreed plan. Prefer the simplest change that satisfies the
   assertions.

4. **Stay in scope.** Build what the tests require, informed by the ticket. Do
   not add features, endpoints, or options no test covers.

5. **Inner loop — iterate fast until target tests are green.**
   After each change, run only the target tests (not the full suite). This keeps
   the feedback loop under a few seconds.
   - BE: `cd ecommerce-backend && npm test -- <target-test-file>`
   - FE: `cd ecommerce-frontend && npm run test -- <test-file>`
   - Repeat until all target tests are green before moving to step 6.

6. **Final gate — run once, after target tests are green.** Do not hand back
   until both sub-steps pass:

   a. **Full test suite** (catches regressions in unrelated tests):
   - BE: `cd ecommerce-backend && npm test`
   - FE: `cd ecommerce-frontend && npm run test`

   b. **Typecheck** (FE only, if available):
   - Check `ecommerce-frontend/package.json` for a `tsc` or `typecheck`
     script and run it if present.

7. **Report back**: the implementation files changed, confirmation that tests
   are green, and a reminder that this is the second commit (production code)
   sitting on top of the test-only diff anchor.

## Hard rules

- **NEVER modify, skip, delete, weaken, or rename a test** to get to green. The
  committed tests are frozen.
- If a test looks **wrong** (bad assertion, impossible expectation), **STOP and
  flag it for the human** with your reasoning — do not edit it yourself. Fixing
  a test is a new `draft-tests` + review + commit cycle, not part of this turn.
- Do not write new tests here. New behaviour needs new tests authored in the
  `draft-tests` step first.
- Keep going until the full suite is green; don't stop at a partial pass.
