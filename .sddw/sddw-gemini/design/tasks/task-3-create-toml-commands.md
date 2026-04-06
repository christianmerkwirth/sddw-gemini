# Task 3: Create TOML commands for all 6 steps

## Trace
- **FR-IDs:** FR-02, FR-03
- **Depends on:** task-1, task-2

## Files
- `commands/sddw/requirements.toml` — create
- `commands/sddw/code-analysis.toml` — create
- `commands/sddw/design.toml` — create
- `commands/sddw/implement.toml` — create
- `commands/sddw/chat.toml` — create
- `commands/sddw/help.toml` — create

## Architecture

### Components
- `commands/sddw/`: 6 TOML command files — entry points for `/sddw:<step>` slash commands — new

### Data Flow
User types `/sddw:<step> <args>` → Gemini loads `commands/sddw/<step>.toml` → `{{args}}` substituted with user arguments → `!{cat ${extensionPath}/...}` inlines content files → Gemini executes combined prompt

## Contracts

### TOML command format
Each command follows this template (using requirements as example):

```toml
prompt = """
<feature_name> {{args}} </feature_name>

If no feature name is provided, ask the user to describe the feature they want to build.

# COMMON RULES
!{cat ${extensionPath}/instructions/common.md}

# INSTRUCTIONS
!{cat ${extensionPath}/instructions/requirements.md}

# QUESTIONNAIRE
!{cat ${extensionPath}/questionnaires/requirements.md}

# SPECS
!{cat ${extensionPath}/specs/requirements.md}

# NEXT STEP
After the user approves the requirements, suggest:
> Run `/sddw:code-analysis <feature-name>` to ground design decisions in the actual code.
> Otherwise, go straight to `/sddw:design <feature-name>`.
"""
```

### Command-to-content mapping

| Command | Instructions | Questionnaire | Specs | Notes |
|---------|-------------|---------------|-------|-------|
| requirements | common.md + requirements.md | requirements.md | requirements.md | |
| code-analysis | common.md + code-analysis.md | code-analysis.md | code-analysis.md | |
| design | common.md + design.md | design.md | design-task.md | |
| implement | common.md + implement.md | implement.md | design-task.md + task-completion.md | Two spec files |
| chat | common.md + chat.md | *(none)* | *(none)* | No questionnaire or spec |
| help | common.md + help.md | *(none)* | *(none)* | No questionnaire or spec |

### Argument wrapping per command

| Command | XML tag | Source equivalent |
|---------|---------|-------------------|
| requirements | `<feature_name>` | `<feature_name> #$ARGUMENTS </feature_name>` |
| code-analysis | `<feature_name>` | same |
| design | `<feature_name>` | same |
| implement | `<feature_name>` | same |
| chat | `<feature_name>` | same |
| help | `<subcommand>` | `<subcommand> #$ARGUMENTS </subcommand>` |

## Design Decisions

### Include mechanism: !{cat ${extensionPath}/...}
- **Chosen:** Use `!{cat ${extensionPath}/path}` shell execution to inline content files
- **Rationale:** This is Gemini CLI's native include mechanism. `${extensionPath}` resolves to the extension's install directory, so paths work regardless of installation location
- **Rejected:** Embedding content directly in TOML — would duplicate content and make syncing with Claude Code version difficult

## Acceptance Criteria

### FR-02: Six commands
- GIVEN a successfully installed extension
- WHEN the user types `/sddw:requirements`
- THEN Gemini CLI recognizes and executes the command

- GIVEN each of the 6 commands
- WHEN invoked
- THEN each SHALL load its corresponding instructions, questionnaire, and spec content

### FR-03: Same output artifacts
- GIVEN the user completes `/sddw:requirements myfeature`
- WHEN the spec is generated
- THEN it SHALL be written to `.sddw/myfeature/requirements.md` with the same sections (Purpose, User Stories, FRs, Acceptance Criteria, Constraints)

## Done Criteria
- [ ] All 6 TOML files exist in `commands/sddw/`
- [ ] Each TOML file has a valid `prompt` field with triple-quoted string
- [ ] `{{args}}` is used for argument injection in all commands
- [ ] `!{cat ${extensionPath}/...}` correctly references all content files per the mapping table
- [ ] Help command uses `<subcommand>` tag, all others use `<feature_name>` tag
- [ ] Implement command includes both `design-task.md` and `task-completion.md` specs
- [ ] Chat and help commands omit questionnaire and spec includes
- [ ] Each command includes a NEXT STEP section with appropriate guidance
