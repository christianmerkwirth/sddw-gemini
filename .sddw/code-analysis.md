## Codebase Analysis

### Relevant Patterns

- **Command-as-Markdown with YAML frontmatter**: Each of the 6 commands is a `.md` file in `commands/` with `---` delimited YAML frontmatter (`name`, `description`, `argument-hint`), followed by argument injection via `<feature_name> #$ARGUMENTS </feature_name>` and `@` file includes. This is the entry point contract for the entire workflow.
- **Modular three-layer content**: Commands compose themselves from three content layers — `instructions/` (process rules), `questionnaires/` (dialog scripts), `specs/` (output templates) — all referenced via `@~/.claude/sddw/` includes. This separation enables independent evolution of each layer.
- **Shared common.md**: A single `instructions/common.md` is included by every command. It defines interaction modes (`--auto`, `--critical-only`, interactive), path resolution, `AskUserQuestion` tool usage rules, critical decision tables per step, and anti-patterns. This is the behavioral contract all steps share.
- **Three-phase dialog**: Every step follows Discover → Research & Propose → Confirm & Generate. Questionnaires script the dialog flow; instructions enforce the rules. This pattern must be preserved in any port.
- **Self-contained task files**: Design produces task files that embed all context (architecture, contracts, models, decisions, acceptance criteria) inline. The implementation agent needs only the task file — no cross-file references required.
- **Traceability chain**: `FR-01`/`FR-02` identifiers flow from requirements → design task `Trace: FR-IDs` → commit messages `type(feature): desc (FR-01)`. This end-to-end traceability must be preserved.
- **Install-to-home**: `bin/install.sh` copies `commands/*.md` to `~/.claude/commands/sddw/`, making them available as `/sddw:<name>` slash commands. Supports `--local` (symlink for development) and remote (git clone/pull) modes.

### Key Interfaces

- **Command frontmatter contract**: `name: sddw:<step>`, `description: <text>`, `argument-hint: "<args>"` — the registration interface for Claude Code. The Gemini port must replace this with its own command registration mechanism (TOML or `gemini-extension.json` commands).
- **`#$ARGUMENTS` injection**: Arguments passed to the command are substituted into `#$ARGUMENTS`, wrapped in XML tags (`<feature_name>` for most commands, `<subcommand>` for help). The port needs `{{args}}` or equivalent Gemini placeholder.
- **`@` file include syntax**: `@~/.claude/sddw/<path>` inlines referenced file content into the command context. The port needs `@{path}` or equivalent Gemini include mechanism.
- **`AskUserQuestion` tool contract**: All user-facing questions use this tool with structured options (2-4 choices, `(Recommended)` suffix, `multiSelect`, `preview`), open-ended questions, or yes/no pairs. The port must use `ask_user` with `choice`/`text`/`yesno` types instead.
- **`.sddw/` output directory structure**: `<feature>/requirements.md` (per-feature), `code-analysis.md` (shared at root), `<feature>/design/tasks/task-<N>-<slug>.md` (per-feature), `<feature>/implement/tasks/task-<N>-<slug>.done.md` (per-feature). This structure is an output contract shared across both versions.
- **FR-ID format**: `FR-01`, `FR-02` etc. — used in requirements spec, task `Trace:` fields, and commit messages. Numbering is sequential, uppercase.
- **`.claude-plugin/plugin.json`**: Plugin metadata (`name`, `version`, `description`, `author`) — the port needs a `gemini-extension.json` equivalent with Gemini-specific fields.

### Existing Flows

- **Command invocation flow**: User types `/sddw:<step> <feature> [flags]` → Claude loads command `.md` → YAML frontmatter registers the command → `#$ARGUMENTS` substituted → `@` includes inline content → Claude executes instructions with questionnaire and spec as context.
- **Requirements flow**: Parse project path → Discover (open question → follow-ups) → Research (web search, domain) → Propose sections one-by-one (Purpose → User Stories → FRs → Acceptance Criteria → Constraints → Testing Approach) → Confirm summary → Write `.sddw/<feature>/requirements.md`.
- **Code analysis flow**: Read requirements → Discover focus areas → Scan codebase → Propose sections one-by-one (Patterns → Interfaces → Flows → Conventions) → Confirm summary → Write `.sddw/code-analysis.md`.
- **Design flow**: Read requirements + optional code-analysis → Discover architecture preferences → Propose sections one-by-one (Architecture → Data Models → Contracts → Design Decisions → Task Breakdown) → Confirm → Write individual task files to `design/tasks/`.
- **Implement flow**: Select task (or parse `--task N`) → Check dependencies → Discover context → Propose implementation plan (with TDD applicability) → Execute (RED → GREEN → REFACTOR) → Commit (staged individually, message with FR-IDs) → Write `.done.md` completion report → Suggest next task.
- **Chat flow**: Load all feature artifacts silently → Ask what user needs → Classify request (Question / Spec Update / Quick Implementation / Status) → Act accordingly → Redirect to full workflow if change is too large (>3 files or >1 commit heuristic).
- **Help flow**: Route on subcommand — no args → show workflow overview, `list` → scan `.sddw/` for features with status indicators, `status <name>` → show detailed progress with task completion status.
- **Install flow**: `bin/install.sh` → clone/pull repo to `~/.claude/sddw/` (or symlink with `--local`) → copy `commands/*.md` to `~/.claude/commands/sddw/` → print available commands.

### Conventions

- **File naming**: Commands, instructions, questionnaires, and specs use `<step-name>.md` (kebab-case). Exceptions: `common.md` (shared instructions), `design-task.md` and `task-completion.md` (domain-specific specs).
- **Directory structure**: `commands/`, `instructions/`, `questionnaires/`, `specs/` at repo root. Not all steps have questionnaires — `chat` and `help` have none. Not all steps have specs — `chat` and `help` have none.
- **Command composition order**: Every command includes `common.md` first, then step-specific instructions, then optionally questionnaire and spec. The `implement` command includes two spec files (`design-task.md` + `task-completion.md`).
- **RFC 2119 keywords**: SHALL, SHOULD, MAY, SHALL NOT used consistently in instructions and in generated requirements output.
- **One question at a time**: Enforced across all interactive flows — never multiple questions in one message. Anti-pattern list in common.md explicitly prohibits this.
- **Section-by-section approval**: Propose one spec section, wait for approval, then move to next. Locked-in sections are not revisited unless user explicitly requests changes.
- **Mode parsing**: `--auto` and `--critical-only` extracted from arguments. Default is interactive. Safety downgrade: `--auto` requirements with <20 words → `--critical-only`. Chat defaults to `--critical-only`.
- **Deviation Rule 4 override**: Architectural deviations always STOP and ask the user, even in `--auto` mode — this is a safety invariant that overrides all mode settings.
- **Commit message format**: `type(feature-name): description (FR-01, FR-02)` with types: `feat`, `fix`, `test`, `refactor`, `chore`. TDD tasks produce 2-3 commits (test → feat → optional refactor).
- **Task file naming**: `task-<N>-<slug>.md` for design, `task-<N>-<slug>.done.md` for completion reports. N is sequential, slug is human-readable kebab-case.
- **Path resolution**: `.sddw/` resolved once at step start relative to project root (git root). All subsequent reads/writes use absolute paths. Requirements step creates `.sddw/` if absent; other steps error if not found.
