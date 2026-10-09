---
name: draft-tests
description: >-
  Step 1 of the test-first pipeline. Given a ticket from ticket.md, draft
  FAILING tests only — never the implementation. Use when starting any new
  ticket test-first. Pairs with the `implement-from-tests` skill, which runs
  in a SEPARATE turn after a human has reviewed and committed these tests.
---

# draft-tests — author the failing tests, nothing else

You are running **step 1 of a two-skill, test-first pipeline**. Your single job
is to turn a ticket's acceptance criteria into **failing tests**. You do **not**
write, scaffold, or even sketch the implementation in this turn. A human reviews
your tests, confirms they fail (RED), and commits them _before_ the
`implement-from-tests` skill is invoked in a separate turn. That separation is
the whole point — it is what stops tests and code being written to match each
other (the "tautology trap").

## Procedure

1. **Get the spec from the ticket.**
   - Read `ticket.md` from the repository root using the file read tool.
   - Parse the title, description, and **acceptance criteria** from it.
   - **If `ticket.md` is missing, empty, or has no acceptance criteria**, ask
     the user to supply the ticket details before proceeding.

2. **If the ticket has no acceptance criteria (or they're vague), DON'T guess —
   propose and confirm.** Many real tickets are just a title + a paragraph.
   In that case:
   - Derive a list of **candidate testable behaviours** from the description
     (happy path + the edge cases the description implies).
   - **Present them back to the user as proposed acceptance criteria / a
     test list, and ask targeted questions** for anything ambiguous (expected
     status codes, boundary values, error messages, null/empty handling).
   - **Wait for the user to confirm or correct** before drafting any tests.
   - Never _silently_ invent acceptance criteria and start coding tests from
     them — surfacing and confirming the intent is the point of this step.

3. **Locate where the tests belong.** Find the existing test suite and match its
   conventions exactly:
   - Backend (`ecommerce-backend/`): place tests under `ecommerce-backend/tests/`,
     using the test framework already present in the project (inspect
     `ecommerce-backend/package.json` for the test runner and scripts).
   - Frontend (`ecommerce-frontend/`): Vitest, co-located `*.test.jsx` / `*.test.js`
     next to the component or module under test inside `ecommerce-frontend/src/`,
     `describe`/`it` style.

4. **Draft failing tests — one behaviour per test.**
   - Derive each test directly from an acceptance criterion (confirmed in step 2
     if the ticket lacked them). Cover the happy path **and** the edge cases the
     ticket names (and obvious ones it implies).
   - **Assert behaviour / output, never implementation shape.** A test must stay
     true no matter how the code is written. If your assertion just restates how
     you imagine the function is built, rewrite it to assert the observable
     result instead.
   - Tests will reference functions/fields/endpoints that **do not exist yet** —
     that is correct. They must fail for the right reason (missing behaviour),
     not a typo.
   - **For frontend features, test at every layer the ticket touches.** If a
     ticket introduces a utility function that drives UI behaviour (e.g. controls
     button visibility, label text, enabled state), draft tests at **both**
     levels: a unit test for the function AND a component-level test (React
     Testing Library) asserting the UI actually reflects the function's output.
     A utility function test alone is not enough — if the component never calls
     the function, the unit tests will pass and the feature will still be broken.
     **The component test must be at the level users actually see.** If the
     behaviour is delivered through a sub-component, also test the parent
     component that renders it — not the sub-component in isolation. A
     sub-component test alone is not enough: if the parent never uses the
     sub-component, tests will pass and the feature will still be broken.

5. **Confirm RED using only the new tests.** Running the full suite here is
   unnecessary and slow — you only need to see your new tests fail.
   - **Run only the new tests** (not the full suite):
     - BE: inspect `ecommerce-backend/package.json` for the test script and run
       it scoped to the new test file (e.g. `cd ecommerce-backend && npm test -- <test-file>`)
     - FE: `cd ecommerce-frontend && npm run test -- <test-file>`

6. **STOP. Hand back for human review.** End your turn with:
   - the list of test files you created/changed,
   - a one-line-per-test summary mapping each test to the acceptance criterion
     it encodes,
   - the exact command to run them and watch them fail,
   - an explicit note: _"Review these, confirm RED, then commit the tests alone
     before running `implement-from-tests`. Pass a reference to that commit (a
     hash, `HEAD`, or just 'latest commit') — the agent will use `git show <ref>`
     to read the frozen tests directly."_

## Hard rules

- **Do NOT** write or modify any implementation/production file in this turn.
- **Do NOT** make a test pass by weakening it. If you cannot make a test fail
  against current code, say so — the behaviour may already exist (a finding the
  human needs).
- **Do NOT** commit. Committing the reviewed tests is the human's step — it is
  the diff anchor.
- One behaviour per test; behaviour-based assertions only.
