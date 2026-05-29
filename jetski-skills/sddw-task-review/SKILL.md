---
name: sddw-task-review
description: >
  Review a single completed implementation task before moving on — a focused
  code review against the task's done/acceptance criteria, design.md contracts,
  and codebase conventions. Use when the user says "sddw task-review",
  "review task N", "task review", or wants to quality-gate a task after
  implement and before verify. Requires the task's completion report to exist.
  Produces .sddw/<feature>/task-review/task-N-<slug>.review.md.
  Supports --task N flag and --auto flag.
---

# sddw Task Review Step

Review a single completed implementation task before moving on. This is Step 6 of the sddw workflow. It is a focused, task-level quality gate that runs after `sddw implement` and before the feature-level `sddw verify`.

## Setup

1. **Parse arguments:** Extract `<feature-name>`, `--task <N>`, and `--auto` flag from the user's message.
2. **Read common rules:** Read `../sddw-common/common-rules.md` and follow all rules throughout this step.

## Input

- `<feature-name>` — the feature being reviewed
- `--task <N>` — the task number to review (e.g., `--task 1`)

If no `--task` is provided, list tasks that have a completion report but no review report and ask the user which to review.

## Goal

Confirm that one task's implementation actually satisfies its done and acceptance criteria, conforms to the design and codebase conventions, is clean and in-scope, and is properly tested — *before* the next task builds on it. Catching issues at task granularity is cheaper than discovering them at feature-level verification.

This step is a **review, not a rewrite**: it SHALL NOT modify code. When changes are needed, it routes back to `sddw implement` for the same task.

## Prerequisites

Read the feature artifacts from `<resolved-sddw-path>/<feature-name>/`:

| Artifact | Path | Required |
|----------|------|----------|
| Task file | `<feature-name>/design/tasks/task-<N>-*.md` | Yes |
| Completion report | `<feature-name>/implement/tasks/task-<N>-*.done.md` | Yes |
| Design | `<feature-name>/design/design.md` | Yes |
| Requirements | `<feature-name>/requirements.md` | Yes |
| Code analysis | `code-analysis.md` | No |

If the task file does not exist, stop and suggest running `sddw taskify` first.
If the completion report does not exist, the task has not been implemented — stop and suggest running `sddw implement <feature> --task <N>` first.
If `design.md` is missing, stop and inform the user they must regenerate the feature tasks via `sddw design` then `sddw taskify`.

Use the Project path from `<resolved-sddw-path>/<feature-name>/requirements.md` as the working directory.

## Process

Follow the three-phase flow defined in `./references/questionnaire.md`:

1. **Assess** — Load the task file, the completion report, and the design. Identify exactly what changed for this task: read the commit hashes recorded in the completion report and inspect their diffs (`git show <hash>`). If no VCS is in use, review the files listed in the task file. *In `--auto`: assess fully autonomously.*

2. **Review** — Run the tests relevant to this task, then review the diff across the Review Dimensions below. Classify each finding by severity. *In `--auto`: classify all findings autonomously.*

3. **Report & Decide** — Produce a task review report following the template at `../sddw-common/specs/task-review-report.md`, ending in a verdict of **APPROVED** or **CHANGES REQUESTED**. If CHANGES REQUESTED, summarise the blocking findings and route back to `sddw implement` for the same task. *In `--auto`: write the report and verdict directly; do not modify code.*

---

## Review Dimensions

For the task under review:

### 1. Criteria Conformance
- Each **Done Criterion** in the task file is actually satisfied by the code (file exists, function behaves, constraint met).
- Each **Acceptance Criterion** for the task's FR-IDs (from `requirements.md`) is met and exercised.

### 2. Design Conformance
- Code follows `design.md` — architecture, data models, and interface contracts.
- No architectural deviation (new table/service, schema change, library switch, breaking API) was introduced without being approved and documented (Deviation Rule 4).

### 3. Convention Conformance
- Matches the patterns, naming, structure, and error-handling style captured in `code-analysis.md` (if present) and the surrounding codebase.
- Lint and formatting rules are respected.

### 4. Code Quality
- Readable, well-named, appropriately commented; no dead code or debug leftovers.
- **No scope creep** — changes are confined to what the task calls for.

### 5. Test Quality
- Tests exist per the Testing Approach in `requirements.md` Constraints and cover the task's acceptance criteria.
- Tests pass. Tests were not weakened or modified merely to pass.

### 6. Deviation Integrity
- Deviations visible in the diff are all documented in the completion report (`.done.md`).
- Auto-fixed deviations (Rules 1-3) are reflected in commit messages.

---

## Severity and Verdict

| Severity | Condition | Action |
|----------|-----------|--------|
| **Blocker** | Done/acceptance criterion unmet, failing tests, design contract violated, undocumented architectural deviation | Must fix before proceeding |
| **Major** | Acceptance criterion not covered by tests, convention violation, scope creep, missing error handling/validation | Should fix |
| **Minor** | Style, naming, comment clarity, small cleanup | Optional |
| **Pass** | Dimension fully satisfied | None |

**Verdict:**
- **APPROVED** — no Blocker and no Major findings. Minor findings may remain as notes.
- **CHANGES REQUESTED** — one or more Blocker or Major findings.

---

## Rules

- SHALL run the actual tests relevant to the task, not assume results.
- SHALL review the real diff (commits from the completion report), not just the task file.
- SHALL reference specific `file:line` and the criterion or convention each finding relates to.
- SHALL classify every finding by severity and end in an explicit verdict.
- SHALL be a review only — SHALL NOT modify code, tests, task files, or completion reports.
- SHALL NOT create remediation task files — that is the feature-level verify step's responsibility. Task-level fixes route back to `sddw implement` for the same task.
- SHALL overwrite the previous review report for the task when re-run (idempotent).

## Output

```
.sddw/<feature-name>/task-review/
└── task-<N>-<slug>.review.md
```

## Next Step

After the review:
- If **APPROVED** and unblocked tasks remain:
  > Task <N> approved. Run `/clear` to free up context, then `sddw implement <feature> --task <next-N>`.
- If **APPROVED** and all tasks are implemented and reviewed:
  > All tasks approved. Run `/clear` to free up context, then `sddw verify <feature>` to check everything works against requirements.
- If **CHANGES REQUESTED**:
  > Re-run `sddw implement <feature> --task <N>` to address the blocking findings, then re-run `sddw task-review <feature> --task <N>`.
