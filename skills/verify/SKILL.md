---
name: "verify"
description: "Verify implementation matches change artifacts. Use when the user wants to validate that implementation is complete, correct, and coherent before archiving."
metadata:
  kit: osf
---

SCOPE DISCIPLINE

Parallel sessions may share this branch. When briefing osf-verify, include these rules verbatim so the subagent has them in its prompt:

- Scope = files in the change's tasks.md / proposal.md / design.md, plus files the caller named in this input
- Verify is report-only — never delete, edit, or "clean up" any file
- Code outside scope that looks like spec drift may belong to another session — report as "out-of-scope code present, cannot verify ownership", NOT as CRITICAL
- Do not recommend deletion of unfamiliar files, even when they seem to violate the spec
- Unfamiliar code = another session's in-progress work, not drift

TDD CHECK

If `openspec/changes/<change-name>/tdd.md` exists, add these rules to the brief:

- Recompute `git hash-object` for every file under `## Lock` in tdd.md; any mismatch or missing file → CRITICAL "test lock broken"
- Run the test command and the E2E command from tdd.md; any failing locked test (e2e included) → CRITICAL
- Every AC in the traceability table must still have a `[AC-n]`-tagged test → missing → CRITICAL
- Run the coverage command; line/branch below the tdd.md target on changed production files → WARNING (not CRITICAL)

Before launching the subagent, gather context from the current conversation:

1. If an OpenSpec change name exists (from a prior spec or implementation):
   - Pass the change name — the subagent reads spec artifacts automatically
2. If no spec but implementation was just done:
   - Summarize what was implemented and what the expected behavior should be
3. If user provides explicit arguments:
   - Pass those directly

Brief the user, then launch Agent tool with `subagent_type: "osf-verify"`.