## Task Review Report

Written after the task-review step reviews a single completed task. Stored at `.sddw/<feature-name>/task-review/task-<N>-<slug>.review.md`.

**Location:** `.sddw/<feature-name>/task-review/task-<N>-<slug>.review.md`

**Format:**
```
# Task Review: task-<N> <slug> (<feature-name>)

## Summary
- **Date:** [ISO date]
- **Commits reviewed:** [hash, hash]
- **Findings:** [blockers] blockers, [majors] majors, [minors] minors
- **Task tests:** [passed] passed, [failed] failed, [skipped] skipped
- **Verdict:** [APPROVED | CHANGES REQUESTED]

## Dimensions

### Criteria Conformance — [PASS | ISSUES]
**Done Criteria** (from task file):
- [x] [criterion] — met
- [ ] [criterion] — [why not met]

**Acceptance Criteria** (FR-XX):
- [x] [scenario] — covered by `[test name]`
- [ ] [scenario] — [uncovered / failing]

### Design Conformance — [PASS | ISSUES]
- [finding, or "Follows design.md contracts"]

### Convention Conformance — [PASS | ISSUES]
- [finding, or "Matches codebase conventions"]

### Code Quality — [PASS | ISSUES]
- [finding, or "Clean and in-scope"]

### Test Quality — [PASS | ISSUES]
- [finding, or "Tests present and passing per testing approach"]

### Deviation Integrity — [PASS | ISSUES]
- [finding, or "Deviations match completion report"]

## Findings
[Numbered list of all findings, or "None"]
1. **[Blocker | Major | Minor]** `path/to/file.py:42` — [what's wrong] ([dimension], [criterion/convention])
   - **Fix:** [what should change]

## Verdict
**[APPROVED | CHANGES REQUESTED]** — [one-line rationale]

[If CHANGES REQUESTED: "Re-run `sddw implement <feature> --task <N>` to address the blockers above, then re-review."]
```

**Rules:**
- SHALL be written after all review checks complete, including running the task's tests.
- SHALL review the actual diff (commits from the completion report), not just the task file.
- SHALL cite specific `file:line` and the criterion or convention for each finding.
- SHALL assign every finding a severity: Blocker, Major, or Minor.
- Verdict SHALL be APPROVED only if there are no Blocker and no Major findings.
- SHALL be a review record only — SHALL NOT imply code was modified by this step.
- SHALL be concise — this is a review, not a narrative.
- Re-running task-review for the same task SHALL overwrite the previous report.

**Example:**
> # Task Review: task-2 token-validation (password-reset)
>
> ## Summary
> - **Date:** 2026-03-25
> - **Commits reviewed:** `e4f5g6h`, `a1b2c3d`
> - **Findings:** 1 blockers, 1 majors, 1 minors
> - **Task tests:** 4 passed, 1 failed, 0 skipped
> - **Verdict:** CHANGES REQUESTED
>
> ## Dimensions
>
> ### Criteria Conformance — ISSUES
> **Done Criteria** (from task file):
> - [x] `is_valid()` method exists
> - [ ] Expired tokens rejected — `test_token_expiry` failing
>
> **Acceptance Criteria** (FR-02):
> - [ ] Failure path: expired token rejected — covered by `test_token_expiry` (FAILING)
>
> ### Design Conformance — PASS
> - Follows design.md token model contract
>
> ### Convention Conformance — PASS
> - Matches codebase conventions
>
> ### Code Quality — ISSUES
> - Helper `_now()` added but unused
>
> ### Test Quality — ISSUES
> - No test for the timezone-aware expiry path
>
> ### Deviation Integrity — PASS
> - Deviations match completion report
>
> ## Findings
> 1. **Blocker** `auth/token.py:31` — `is_valid()` uses `<` instead of `<=`, accepting tokens at the exact expiry instant (Criteria Conformance, FR-02 done criterion).
>    - **Fix:** Change comparison to `<=` and ensure `test_token_expiry` passes.
> 2. **Major** `tests/test_token.py` — no coverage for timezone-aware expiry (Test Quality, FR-02 acceptance criterion).
>    - **Fix:** Add a test exercising an aware `datetime`.
> 3. **Minor** `auth/token.py:9` — unused helper `_now()` (Code Quality).
>    - **Fix:** Remove dead code.
>
> ## Verdict
> **CHANGES REQUESTED** — boundary bug fails an acceptance criterion and a required scenario is untested.
>
> Re-run `sddw implement password-reset --task 2` to address the blockers above, then re-review.
