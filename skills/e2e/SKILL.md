---
name: "e2e"
description: "End-to-end tests for user journeys (UI or API) after implementation. Picks up the journeys osf:tdd left pending, or a spec/flow you name, writes tests with the project's e2e framework, runs them against a local app, and turns mismatches into fix rounds. Args: [change-name] [flow description] [--red-only]."
metadata:
  kit: osf
---

You orchestrate end-to-end testing: **journeys → tests → run against the real app → fix app defects → lock**. You do not write tests or code yourself — osf-e2e writes and runs tests, osf-apply fixes the app. You read, check, and route.

This is not the `browser` skill: `browser` explores and reproduces by hand and leaves nothing behind; `e2e` leaves automated tests in the repo that run in CI. A flow found with `browser` is a good input for `e2e`.

## TEST LOCK (verbatim rules)

Whenever you brief osf-apply after e2e tests exist, include this block verbatim (merge with any lock the `tdd` skill already recorded):

```
TEST LOCK — tests were written first and are the specification.
- Locked test files: <list>
- Do NOT edit, delete, rename, skip (.skip/xit/@Ignore/pytest.mark.skip) or weaken any locked test or its fixtures
- Do NOT change test runner/coverage config to exclude locked tests
- Make tests pass by changing production code only
- If a locked test looks wrong, STOP and report it as "Test dispute: <file> › <test> — <reason>" instead of changing it
```

---

## Step 0: Parse arguments and find the journeys

Arguments (all optional): `[change-name] [flow description] [--red-only]`
- `--red-only`: write and run tests, report, no fix rounds
- Input containing `CALLER_CONTEXT: autopilot-pipeline` behaves like `--red-only` with the autopilot output contract (Step 4)

Find the source, in this order:
1. **`openspec/changes/<name>/tdd.md` with `## E2E pending`** (change name from args, conversation, or the single active change in `openspec list --json`) → those ACs
2. **OpenSpec change** without tdd.md → the journey scenarios in its specs
3. **Flow description** in args or conversation (including a flow just walked with the `browser` skill)
4. **Neither** → print one line and stop:
   `osf:e2e needs a change name or a flow description — e.g. /osf e2e add-user-auth, or /osf e2e "guest checkout with a coupon".`

## Step 1: Check the e2e stack

Look for an e2e framework (`playwright.config.*`, `cypress.config.*`, WebdriverIO, Detox/Maestro, API suites such as supertest/`WebApplicationFactory`/RestAssured/Newman) and how the app is started for it.

- **Found** → proceed to Step 2
- **None, interactive** → recommend one that fits the project (Playwright for web UI unless the ecosystem says otherwise; for API-only, the HTTP test client the stack already uses) and ask once: "Project chưa có framework e2e. Cài `<command>` không?" Yes → pass `Setup approved: <command>` to osf-e2e. No → stop and say which ACs stay pending
- **None, autopilot** → do not install. Print ONLY:
  `⚠️ E2E skipped: <change-name> — no e2e framework (install: <command>)`
  and return in the same turn so the pipeline continues to verify

## Step 2: Launch osf-e2e

Brief the user, then launch Agent tool with `subagent_type: "osf-e2e"`:

```
Change name: <change-name>            (spec mode)
Flow: <description>                   (flow mode)
Acceptance criteria:                  (only when you derived them yourself)
- AC-n [ui|api] <journey> → <expected outcome>
Setup approved: <command>             (only if the user agreed)
Docs: <paths>
App start / base URL: <if already known from the project>
```

## Step 3: Read the report and act

Read `## 🧪 E2E Report`.

1. **Env blockers** → show them with the exact error. Interactive: stop and let the user fix the environment, then re-run `/osf e2e`. Autopilot: carry them as a warning and continue to Step 4
2. **Test faults remaining after 3 rounds** → show them; stop (interactive) or carry as warning (autopilot)
3. **App defects** (tests that fail because the product diverges from the spec) — this is the point of e2e:
   - `--red-only` / autopilot → report them (autopilot's verify step will see the failing locked tests)
   - Otherwise → launch osf-apply with the SCOPE DISCIPLINE rules from the `apply` skill, the TEST LOCK block (locked e2e files + any tdd lock), the defect list (expected vs actual), and `Done = E2E command passes`. Then re-run the E2E command yourself and compare `git hash-object` of the locked e2e files to the recorded lock. Hash differs → **CRITICAL: test lock broken**, do not accept. Test dispute reported → show the user and stop. Max 2 fix rounds, then report what remains
4. **Lock** — confirm the `## E2E lock` hashes in tdd.md match the files on disk. You read and compare; osf-e2e writes the lock

## Step 4: Output

**Autopilot** (`CALLER_CONTEXT: autopilot-pipeline`): print ONLY this line and return in the same turn — no summary, no next-step suggestions:
```
✅ E2E complete: <change-name> (<pass>/<n> passing, <n> app defects, <n> env blocked)
```

**Otherwise**:
```
## ✅ E2E Complete

**Source:** <change-name | flow>
**Stack:** <framework> · cmd `<…>` · app start `<…>`
**Journeys:** <covered>/<total> AC · <n> tests · all passing · lock intact ✓
**Fix rounds:** <n>
**Warnings:** <env blockers, still-pending ACs, test faults — or "none">
```

Do not run verify or archive yourself — the user (or autopilot) routes that.

## Guardrails

- Never write tests or production code yourself — osf-e2e and osf-apply do
- Never install a framework without the user's go-ahead; never in autopilot
- Local app only — never shared/staging/production URLs
- Never edit a locked test to get green, and never let a subagent do it
- Never commit

The following is the user's request:
