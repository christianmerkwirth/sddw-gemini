# Task 2 Completion: Port instructions, questionnaires, and specs

## Summary
Created all 15 content files (7 instructions, 4 questionnaires, 4 specs) adapted from the Claude Code source with `AskUserQuestion` replaced by `ask_user` with `choice`/`text`/`yesno` types, `@~/.claude/` references stripped, and `preview` field usage removed.

## Commits
- `d3b089b` feat(sddw-gemini): port instructions, questionnaires, and specs (FR-04, FR-05, FR-06, FR-07, FR-08)

## Deviations
None

## Difficulties
None

## Notes
All 33 `ask_user` references in the ported files use explicit type parameters. The questionnaires adapt the source's inline `AskUserQuestion` calls to `ask_user` with contextually appropriate types — `choice` for option lists, `text` for open-ended prompts, `yesno` for confirmations. The `preview` field (not available in Gemini CLI) was replaced by using the `description` field for context where needed.
