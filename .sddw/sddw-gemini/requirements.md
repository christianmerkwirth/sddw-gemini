# Requirements: sddw-gemini

## 1. Project

- Path: `/home/cmerk/repos/sddw-gemini`

---

## 2. Purpose

Port the sddw spec-driven development workflow to Gemini CLI as a native Extension, enabling Gemini CLI users to follow the same structured requirements -> code-analysis -> design -> implement pipeline available to Claude Code users.

---

## 3. User Stories

- As a Gemini CLI user, I want to run `/sddw:requirements <feature>` to generate a requirements spec, so that I get the same structured discovery and proposal flow as in Claude Code.
- As a Gemini CLI user, I want to install sddw-gemini via `gemini extensions install`, so that setup is simple and follows Gemini CLI conventions.
- As a developer maintaining both versions, I want the Gemini port to follow the same modular structure (instructions, questionnaires, specs), so that changes to the workflow can be synced across both repos.

---

## 4. Functional Requirements

- FR-01: Extension SHALL include a `gemini-extension.json` manifest that enables installation via `gemini extensions install`.
- FR-02: Extension SHALL provide 6 commands matching the original sddw skills: `sddw:requirements`, `sddw:code-analysis`, `sddw:design`, `sddw:implement`, `sddw:chat`, `sddw:help`.
- FR-03: Each command SHALL produce the same output artifacts (`.sddw/<feature>/` structure) as the original sddw.
- FR-04: Instructions SHALL reference `ask_user` tool instead of `AskUserQuestion`, adapting parameter names to Gemini CLI's tool schema (`choice`, `text`, `yesno` types).
- FR-05: Commands SHALL support the same `--auto`, `--critical-only`, and interactive modes.
- FR-06: Instructions SHALL preserve the three-phase dialog pattern (Discover -> Research & Propose -> Confirm & Generate) for each step.
- FR-07: Extension SHALL NOT introduce new workflow steps or modify the existing step semantics.
- FR-08: Extension SHALL NOT depend on Claude Code-specific features (`AskUserQuestion`, `#$ARGUMENTS`, Claude YAML frontmatter, Claude-specific tool names).

---

## 5. Acceptance Criteria

### FR-01: Extension manifest

**Happy path:**
- GIVEN the sddw-gemini repo
- WHEN a user runs `gemini extensions install`
- THEN the extension installs successfully and commands are available

**Validation:**
- GIVEN the `gemini-extension.json` file
- WHEN parsed
- THEN it SHALL contain valid `name`, `version`, `description`, and `commands`/`skills` entries

### FR-02: Six commands

**Happy path:**
- GIVEN a successfully installed extension
- WHEN the user types `/sddw:requirements`
- THEN Gemini CLI recognizes and executes the command

**Coverage:**
- GIVEN each of the 6 commands
- WHEN invoked
- THEN each SHALL load its corresponding instructions, questionnaire, and spec content

### FR-03: Same output artifacts

**Happy path:**
- GIVEN the user completes `/sddw:requirements myfeature`
- WHEN the spec is generated
- THEN it SHALL be written to `.sddw/myfeature/requirements.md` with the same sections (Purpose, User Stories, FRs, Acceptance Criteria, Constraints)

### FR-04: ask_user adaptation

**Happy path:**
- GIVEN the instructions reference user interaction
- WHEN proposing options
- THEN they SHALL reference `ask_user` with `choice`, `text`, or `yesno` types instead of `AskUserQuestion`

### FR-05: Interaction modes

**Happy path:**
- GIVEN a command invoked with `--auto`
- WHEN executed
- THEN it SHALL run fully autonomously without pausing for input

**Edge case:**
- GIVEN `--auto` on requirements with <20 words description
- WHEN invoked
- THEN it SHALL downgrade to `--critical-only`

### FR-06: Three-phase dialog

**Happy path:**
- GIVEN any step command in interactive mode
- WHEN executed
- THEN it SHALL follow Discover -> Research & Propose -> Confirm & Generate phases

### FR-07: No new steps

**Validation:**
- GIVEN the extension
- WHEN reviewing commands
- THEN it SHALL contain exactly the same 6 commands with no additional workflow steps

### FR-08: No Claude-specific dependencies

**Validation:**
- GIVEN any file in the extension
- WHEN searched for `AskUserQuestion`, `#$ARGUMENTS`, or Claude YAML frontmatter
- THEN zero matches SHALL be found

---

## 6. Constraints

### In Scope
- Gemini Extension manifest and packaging (`gemini-extension.json`)
- All 6 commands ported as TOML commands and/or Agent Skills
- Modular instruction/questionnaire/spec files adapted for Gemini CLI
- `ask_user` tool references replacing `AskUserQuestion`
- `{{args}}` and `@{path}` replacing `#$ARGUMENTS` and `@` references
- README with installation and usage instructions

### Out of Scope
- Automated tests — manual validation only
- Install script — extension install via `gemini extensions install` is sufficient
- Changes to the workflow logic or step semantics — faithful port only
- Backward compatibility with Claude Code — this is a standalone repo

### Prohibitions
- SHALL NOT reference Claude Code-specific tools or features in any output file
- SHALL NOT modify the `.sddw/` output directory structure or artifact format
- SHALL NOT add, remove, or reorder workflow steps

### Testing Approach
- No tests — this is a configuration/prompt port consisting of markdown and TOML files, not executable code. Validation is manual by running commands in Gemini CLI.
