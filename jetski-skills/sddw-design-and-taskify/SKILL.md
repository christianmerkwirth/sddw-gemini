---
name: sddw-design-and-taskify
description: >
  Combined design and task decomposition in one flow. Use when the user says
  "sddw design and taskify", "design and break down tasks", or wants the
  one-shot path for smaller features. Produces both design.md and task files.
  Requires requirements.md. Supports --auto flag for fully autonomous mode.
---

# sddw Design and Taskify Step

Generate the cross-cutting `design.md` plus hybrid task files for a feature in a single combined dialog. End artefacts are structurally equivalent to running `sddw design` then `sddw taskify`.

## Setup

1. **Parse arguments:** Extract `<feature-name>` and `--auto` flag from the user's message.
2. **Read common rules:** Read `../sddw-common/common-rules.md` and follow all rules throughout this step.

## Goal

Produce both `design.md` (the shared architecture, data models, interface contracts, and design decisions) and the hybrid task files (`task-<N>-<slug>.md`) that reference it. This command provides a single monolithic flow for smaller features where design and task decomposition can be approved together.

## Prerequisites

Read the requirements spec:
`<resolved-sddw-path>/<feature-name>/requirements.md`

If the requirements spec does not exist, inform the user and suggest running `sddw requirements <feature-name>` first.

Check if `<resolved-sddw-path>/code-analysis.md` exists. If it does, read it and use it to ground design decisions. If it does not exist, that is fine — perform lightweight codebase scanning as needed.

Check if `<resolved-sddw-path>/<feature-name>/design/design.md` already exists:
- **Interactive mode:** Present options: `Overwrite` / `Edit existing` / `Abort` before proceeding.
- **`--auto` mode:** refuse with message "design.md already exists at <path>; re-run interactively or delete it first"

## Process

Follow the monolithic 3-phase flow defined in `./references/questionnaire.md`.

## Rules

- SHALL produce artefacts structurally equivalent to running the split flow (`sddw design` then `sddw taskify`).
- SHALL read and reference the requirements spec — every design element traces to an FR
- SHALL use the Project path from the requirements spec as the target codebase for analysis
- SHALL write `design.md` before any task files, ensuring `design.md` is preserved if task generation aborts mid-flow.
- SHALL use `.sddw/code-analysis.md` if it exists, but SHALL NOT require it
- SHALL analyse the actual codebase if code-analysis is absent
- Every task SHALL trace to one or more FR-IDs
- Every FR SHALL appear in at least one task
- Tasks SHALL be dependency-ordered (independent first) with explicit `Depends on:` field
- Tasks SHALL declare `Depends on:` for sequencing.
- Tasks SHALL use the hybrid format from `../sddw-common/specs/design-task.md`, referencing `design.md` for cross-cutting concerns.
- SHALL NOT silently overwrite an existing `design.md` (see Prerequisites).
- SHALL NOT proceed to generation without user approval in interactive mode. `--auto` mode may proceed without approval.

## Output

```
.sddw/<feature-name>/design/
├── design.md
└── tasks/
    ├── task-1-<slug>.md
    ├── task-2-<slug>.md
    └── ...
```

## Next Step

After generating the design and task files, suggest:
> Run `/clear` to free up context, then `sddw implement <feature-name> --task 1`
