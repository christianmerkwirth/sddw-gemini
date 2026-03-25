# sddw — Spec-Driven Development Workflow

sddw is a structured development workflow where **specifications are the source of truth** and code is a verified artifact. You follow a pipeline that produces durable `.md` specs before writing any code.

## Pipeline

```
/sddw:requirements  →  /sddw:code-analysis  →  /sddw:design  →  /sddw:implement
       (Step 1)              (Step 2, optional)      (Step 3)          (Step 4)
```

| Step | Command | Output |
|------|---------|--------|
| Requirements | `/sddw:requirements <feature>` | `.sddw/<feature>/requirements.md` |
| Code Analysis | `/sddw:code-analysis <feature>` | `.sddw/code-analysis.md` |
| Design | `/sddw:design <feature>` | `.sddw/<feature>/design/tasks/task-N-*.md` |
| Implement | `/sddw:implement <feature> --task N` | Code + `.sddw/<feature>/implement/tasks/task-N-*.done.md` |

Additional commands:
- `/sddw:chat <feature>` — Fast-track interaction with a feature (quick edits, questions, updates)
- `/sddw:help` — Show workflow overview and feature status

## Artifacts

All artifacts live under `.sddw/` in the project root:

```
.sddw/
  <feature>/
    requirements.md       # Step 1 output
    design/
      tasks/
        task-1-*.md       # One file per implementation task
    implement/
      tasks/
        task-1-*.done.md  # Completion reports
  code-analysis.md        # Optional shared codebase analysis
```

## Interaction

All steps support three modes via flags:
- *(default)* — Interactive: guided dialog, one question at a time
- `--critical-only` — Autonomous except for critical decisions
- `--auto` — Fully autonomous, no questions asked

For structured user interaction, use the `ask_user` tool with types:
- `choice` — present 2–4 mutually exclusive options
- `text` — open-ended question
- `yesno` — yes/no confirmation
