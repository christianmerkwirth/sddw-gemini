# Verify Questionnaire

Three-phase dialog for verifying implementation against requirements.

---

## Phase 1: Assess

*In `--auto`: assess fully autonomously.*

Understand the current state of the feature. Load artifacts and identify the verification scope.

**Step 1 — Status overview:**

Present a status table showing:
- Total tasks: [count design tasks]
- Completed: [count .done.md files]
- Reviewed: [count APPROVED .review.md files]; list tasks not reviewed or with CHANGES REQUESTED
- Pending: [list incomplete tasks]

If there are pending tasks, use `ask_user` with `type: "choice"` and options:
- "Verify completed tasks only (Recommended)" — verify what's been implemented so far
- "Wait — implement remaining tasks first" — redirect to implement

Wait for response.

**Step 2 — Test runner detection:**

Scan the project for test configuration:
- `package.json` (scripts.test), `jest.config.*`, `vitest.config.*`
- `pytest.ini`, `pyproject.toml` ([tool.pytest]), `setup.cfg`
- `Makefile` (test targets), `.github/workflows/*`
- Other common test runners

If detected:
> "I'll run tests using [runner]. Any specific test command or flags I should use instead?"

Use `ask_user` with `type: "choice"` and options:
- "Yes, use [detected command] (Recommended)"
- "I'll provide a different command"

Wait for response.

If not detected, use `ask_user` with `type: "choice"` and options:
- "No tests — skip test execution"
- "I'll provide the test command"

Wait for response.

---

## Phase 2: Verify

Run checks and present findings. Group by FR.

### 2.1 Test Execution

Run the detected test command. Report results:
> "Test results: [pass] passed, [fail] failed, [skip] skipped."
> If failures: list failing tests with brief error summaries.

### 2.2 FR-by-FR Verification

For each FR, present the verification result:

> **FR-01: [title]** — [PASS | FAIL | PARTIAL]
> - Acceptance criteria: [N/M covered by tests]
> - Done criteria: [checked / unchecked items]
> - Tests: [pass / fail count for related tests]
> - Issues: [list specific problems, or "none"]

*In `--auto`: classify all results autonomously.*

If there are ambiguous results, use `ask_user` with `type: "choice"` and options:
- "Pass — [reason why it could be acceptable]"
- "Fail — [reason why it should be fixed]"
- "Partial — [explanation]"

Wait for response for each ambiguous result.

---

## Phase 3: Report & Remediate

### 3.1 Summary

Present the verification summary:

> **Verification summary for <feature-name>:**
> - FRs passed: [count]
> - FRs failed: [count]
> - FRs partial: [count]
> - Warnings: [count]
> - Test suite: [pass/fail/skip]

### 3.2 Remediation (if issues found)

If any FRs are fail or partial, propose remediation tasks:

> "I'd create these remediation tasks:"
> 1. Task [N+1]: Fix [description] (FR-01) — fixes [specific issue]
> 2. Task [N+2]: Add [description] (FR-03) — covers [uncovered criteria]
> "Each task follows the same format as design tasks and can be run with `/sddw:implement`, then reviewed with `/sddw:task-review`."

Use `ask_user` with `type: "choice"` and options:
- "Create all remediation tasks (Recommended)"
- "Create selected tasks — I'll specify which"
- "Skip — no remediation tasks needed"

Wait for response.

If "selected": use `ask_user` with `type: "choice"` and `multiSelect: true`, with each proposed task as an option.

### 3.3 Generate

Generate the verification report to `.sddw/<feature-name>/verify/report.md`.

If remediation tasks were approved, generate them to `.sddw/<feature-name>/design/tasks/`.

*In `--auto`: create all proposed tasks directly.*
