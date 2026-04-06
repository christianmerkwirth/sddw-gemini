# Task 3 Completion: Create TOML commands for all 6 steps

## Summary
Created 6 TOML command files in `commands/sddw/` — one per workflow step. Each uses `{{args}}` for argument injection and `!{cat ${extensionPath}/...}` to inline instructions, questionnaires, and specs from the content files created in task 2.

## Commits
- `fa5b592` feat(sddw-gemini): create TOML commands for all 6 steps (FR-02, FR-03)

## Deviations
None

## Difficulties
None

## Notes
- `chat` and `help` omit questionnaire and spec includes per the mapping table — they have no multi-phase dialog
- `implement` includes both `design-task.md` and `task-completion.md` specs
- `help` uses `<subcommand>` XML tag; all others use `<feature_name>`
- NEXT STEP guidance in each command omits the `/clear` suggestion present in the Claude Code source — Gemini CLI handles context differently
