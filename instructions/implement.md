# Implement Step Instructions

Implement a single task from the design spec. This is Step 3 of the sddw workflow. The user specifies which task to execute.

## Input

- `<feature-name>` — the feature being implemented
- `--task <N>` — the task number to execute (e.g., `--task 1`)

If no `--task` is provided, list available tasks and ask the user which to execute.

## Prerequisites

Read the task file: `<resolved-sddw-path>/<feature-name>/design/tasks/task-<N>-*.md`

Use the Project path from `<resolved-sddw-path>/<feature-name>/requirements.md` as the working directory for implementation.

Check `Depends on:` — if dependencies are not yet complete, warn the user.

Reference only if needed:
- `<resolved-sddw-path>/code-analysis.md` — for codebase patterns and conventions (may not exist)
- `<resolved-sddw-path>/<feature-name>/requirements.md` — if acceptance criteria need clarification

## Process

Follow the three-phase flow defined in the questionnaire:

1. **Discover** — Identify the task, check dependencies, ask if there's any context since the design was written. *In `--critical-only`: pick next unblocked task if none specified, but ask the user if dependencies are incomplete. In `--auto`: perform discovery fully autonomously.*

2. **Research & Propose** — Scan codebase for current state of files, research test patterns and library usage. Propose implementation approach and TDD applicability. User confirms. *In `--critical-only`: research and decide approach autonomously. In `--auto`: same.*

3. **Execute & Report** — Implement following the TDD Protocol, Commit Protocol, and Deviation Handling (see Common Rules). Report completion (see Completion Report in Common Rules). *Note: Deviation Rule 4 (architectural) always asks the user, even in `--auto` mode.*

---

## Completion Report

After the task commit(s), write a completion report (see Common Rules). This enables `/sddw:help status` to show task completion and provides context for future tasks.

## Output

- Implemented code for the specified task
- Commit(s) with descriptive messages referencing FR-IDs
- Completion report (`.done.md`) in `implement/tasks/`
