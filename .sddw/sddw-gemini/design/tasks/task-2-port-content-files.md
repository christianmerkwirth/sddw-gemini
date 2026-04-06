# Task 2: Port instructions, questionnaires, and specs

## Trace
- **FR-IDs:** FR-04, FR-05, FR-06, FR-07, FR-08
- **Depends on:** none

## Files
- `instructions/common.md` — create (adapted from source)
- `instructions/requirements.md` — create (adapted from source)
- `instructions/code-analysis.md` — create (adapted from source)
- `instructions/design.md` — create (adapted from source)
- `instructions/implement.md` — create (adapted from source)
- `instructions/chat.md` — create (adapted from source)
- `instructions/help.md` — create (adapted from source)
- `questionnaires/requirements.md` — create (adapted from source)
- `questionnaires/code-analysis.md` — create (adapted from source)
- `questionnaires/design.md` — create (adapted from source)
- `questionnaires/implement.md` — create (adapted from source)
- `specs/requirements.md` — create (adapted from source)
- `specs/code-analysis.md` — create (adapted from source)
- `specs/design-task.md` — create (adapted from source)
- `specs/task-completion.md` — create (adapted from source)

## Architecture

### Components
- `instructions/`: Process rules for each step — adapted from source, `ask_user` replaces `AskUserQuestion` — new
- `questionnaires/`: Dialog scripts for 4 steps (requirements, code-analysis, design, implement) — adapted — new
- `specs/`: Output templates for generated artifacts — adapted — new

### Data Flow
TOML commands `!{cat ${extensionPath}/instructions/...}` → content inlined into prompt → Gemini executes with `ask_user` for interaction

## Contracts

### ask_user tool (replacing AskUserQuestion)

All user-facing questions SHALL use the `ask_user` tool. Parameter mapping from source:

| Source (AskUserQuestion) | Target (ask_user) |
|---|---|
| `options` array with `label`, `description` | `type: "choice"`, `options` array with `label`, `description` |
| Open-ended `question` (no options) | `type: "text"`, optional `placeholder` |
| Yes/No with two options | `type: "yesno"` |
| `multiSelect: true` | `multiSelect: true` on choice type |
| `preview` field on options | Not available — omit, use description for context |
| `header` (max 12 chars) | `header` (max 16 chars) |
| `(Recommended)` suffix on label | `(Recommended)` suffix on label — preserved |

## Design Decisions

### Tool reference adaptation: Adapt inline
- **Chosen:** Rewrite the Tool Usage section in `common.md` to reference `ask_user` with `choice`/`text`/`yesno` types, keeping the same behavioral rules
- **Rationale:** Ensures instructions reference actual Gemini CLI tool parameters, avoiding confusion from non-existent parameter names
- **Rejected:** Minimal find-and-replace — would leave references to AskUserQuestion-specific parameters that don't exist in Gemini CLI

### Path reference handling: Strip @-paths
- **Chosen:** Remove all `@~/.claude/sddw/` include references from instruction content
- **Rationale:** Gemini CLI handles inclusion at the TOML level via `!{cat ${extensionPath}/...}`, so instruction files don't need path references. Stripping them avoids dangling references to Claude Code paths
- **Rejected:** Replacing with descriptive notes — adds noise without value since the TOML commands document the inclusion structure

### Adaptations to apply across all files

1. **`AskUserQuestion`** → `ask_user` with type-specific parameters (choice/text/yesno)
2. **`@~/.claude/sddw/...`** references → strip entirely (TOML handles includes)
3. **`#$ARGUMENTS`** → `{{args}}` (handled in TOML commands, but any references in instructions should use `{{args}}`)
4. **Claude-specific tool names** → remove or replace with Gemini equivalents
5. **YAML frontmatter** in commands → not applicable (TOML handles metadata)
6. **`preview` field** on options → omit (not available in Gemini `ask_user`), use `description` for context instead
7. **Preserve**: three-phase dialog, interaction modes (`--auto`, `--critical-only`, interactive), critical decision tables, anti-patterns, RFC 2119 keywords, section-by-section approval, `.sddw/` output structure, FR-ID format, all behavioral rules

## Acceptance Criteria

### FR-04: ask_user adaptation
- GIVEN the instructions reference user interaction
- WHEN proposing options
- THEN they SHALL reference `ask_user` with `choice`, `text`, or `yesno` types instead of `AskUserQuestion`

### FR-05: Interaction modes
- GIVEN a command invoked with `--auto`
- WHEN executed
- THEN it SHALL run fully autonomously without pausing for input

- GIVEN `--auto` on requirements with <20 words description
- WHEN invoked
- THEN it SHALL downgrade to `--critical-only`

### FR-06: Three-phase dialog
- GIVEN any step command in interactive mode
- WHEN executed
- THEN it SHALL follow Discover -> Research & Propose -> Confirm & Generate phases

### FR-07: No new steps
- GIVEN the extension
- WHEN reviewing commands
- THEN it SHALL contain exactly the same 6 commands with no additional workflow steps

### FR-08: No Claude-specific dependencies
- GIVEN any file in the extension
- WHEN searched for `AskUserQuestion`, `#$ARGUMENTS`, or Claude YAML frontmatter
- THEN zero matches SHALL be found

## Done Criteria
- [ ] All 7 instruction files exist in `instructions/`
- [ ] All 4 questionnaire files exist in `questionnaires/`
- [ ] All 4 spec files exist in `specs/`
- [ ] Zero occurrences of `AskUserQuestion` in any file
- [ ] Zero occurrences of `#$ARGUMENTS` in any file
- [ ] Zero occurrences of `@~/.claude/` in any file
- [ ] Zero occurrences of Claude YAML frontmatter (`---\nname:`) in any file
- [ ] All `ask_user` references use `choice`, `text`, or `yesno` types
- [ ] Three-phase dialog pattern preserved in all questionnaires
- [ ] Interaction mode rules (`--auto`, `--critical-only`, interactive) preserved in `common.md`
- [ ] `.sddw/` output directory structure unchanged in all instructions and specs
