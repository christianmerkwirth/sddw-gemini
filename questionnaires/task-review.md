# Task Review Questionnaire

Three-phase dialog for reviewing a single completed task before moving on.

---

## Phase 1: Assess

*In `--auto`: assess fully autonomously.*

Understand what the task was supposed to do and what actually changed.

**Step 1 — Task selection:**

If no `--task` flag provided:
List tasks that have a completion report (`.done.md`) but no review report (`.review.md`), then use `AskUserQuestion` with each as an option:
- "Task 1: [name]" — implemented, not yet reviewed
- "Task 2: [name]" — implemented, not yet reviewed
Question: "Which task would you like to review?"

If `--task` flag provided, confirm the task is reviewable:
> "Task [N]: [name]. Completion report found. I'll review the changes from commits [hashes]."
> If no completion report exists: tell the user the task hasn't been implemented yet and suggest `/sddw:implement <feature> --task <N>`.

**Step 2 — Scope the diff:**

- Read the commit hashes from the completion report and inspect their diffs.
- If no VCS is in use, fall back to the files listed in the task file.
- Note the set of changed files to review.

---

## Phase 2: Review

Run checks and present findings, grouped by Review Dimension.

### 2.1 Test Execution

Run the tests relevant to this task. Report results:
> "Task tests: [pass] passed, [fail] failed, [skip] skipped."
> If failures: list failing tests with brief error summaries.

### 2.2 Dimension-by-Dimension Review

For each dimension, present the result:

> **Criteria Conformance** — [PASS | findings]
> - Done criteria: [met / unmet items]
> - Acceptance criteria (FR-XX): [covered / uncovered]
>
> **Design Conformance** — [PASS | findings]
> **Convention Conformance** — [PASS | findings]
> **Code Quality** — [PASS | findings]
> **Test Quality** — [PASS | findings]
> **Deviation Integrity** — [PASS | findings]

Each finding cites `file:line` and a severity (Blocker / Major / Minor).

*In `--auto`: classify all findings autonomously.*

If a finding's severity is ambiguous, use `AskUserQuestion` with options:
- "Blocker — [why it must be fixed before proceeding]"
- "Major — [why it should be fixed]"
- "Minor — [why it's optional]"

Wait for response for each ambiguous finding.

---

## Phase 3: Report & Decide

### 3.1 Summary

Present the review summary:

> **Task review summary for task-[N] (<feature-name>):**
> - Blockers: [count]
> - Majors: [count]
> - Minors: [count]
> - Task tests: [pass/fail/skip]
> - **Verdict: [APPROVED | CHANGES REQUESTED]**

### 3.2 Generate

Generate the review report to `.sddw/<feature-name>/task-review/task-<N>-<slug>.review.md`.

Do NOT modify code, tests, task files, or completion reports.

### 3.3 Route

- **APPROVED**, more tasks remain → suggest implementing the next unblocked task.
- **APPROVED**, all tasks implemented and reviewed → suggest `/sddw:verify <feature>`.
- **CHANGES REQUESTED** → suggest re-running `/sddw:implement <feature> --task <N>` to address the blocking findings, then re-reviewing.

*In `--auto`: write the report and verdict directly, then state the routing recommendation.*
