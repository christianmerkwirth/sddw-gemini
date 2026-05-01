# Common Rules

These rules apply to all sddw steps.

## Interaction Modes

Parse `--auto` from the user's message. Default to interactive mode if not present.

| Mode | Flag | Behavior |
|------|------|----------|
| **Interactive** | *(default)* | Full guided dialog. One question at a time, every section confirmed. |
| **Auto** | `--auto` | Fully autonomous. No questions asked. Use best judgment for all decisions. |

### Mode rules

All modes perform the same work — discover, research, propose, decide. The difference is only in what requires user input.

- **Interactive**: Follow the questionnaire as written — one question at a time, wait for approval on every section.
- **Auto**: Perform all phases autonomously. Make all decisions using best judgment. Generate output directly. Still follow all spec format rules and quality standards.

### Safety

- In `--auto` mode for the **requirements** step: warn the user that requirements quality depends on input detail. If the feature description in the arguments is less than ~20 words, downgrade to interactive mode and ask for a more detailed description.
- In `--auto` mode: architectural deviations (Deviation Rule 4) during implement still STOP and ask — this overrides `--auto` because the risk of silent architectural changes is too high.

## Interaction (Interactive mode)

- Ask ONE question at a time. Wait for the user's response before asking the next.
- Use the user's answer to shape the follow-up — do not follow a script.
- Build understanding incrementally — each answer narrows the next question.
- Never dump multiple questions in a single message.
- SHALL NOT dump the spec template or full output structure to the user. Use the spec as internal guidance. Present proposals in conversational form, one block at a time, and confirm each before moving on.

### Presenting Options

When the instructions call for presenting choices to the user:

- **Options / choices (2-4 items):** Present as a numbered list with labels and descriptions. Mark the recommended option with "(Recommended)". Ask the user to pick one. When choices are not mutually exclusive, tell the user they can select multiple.
- **Open-ended question:** Ask the question directly. Wait for the user's response.
- **Yes/No confirmation:** Ask a clear yes/no question. Wait for the user's response.

## Path Resolution

All `.sddw/` references are **relative to the current working directory** — the directory from which the user invoked the command.

| Step | Path resolution |
|------|----------------|
| **Requirements** | If the user specifies a Project path other than `.`, resolve `.sddw/` relative to that path. If the Project path is `.` or unspecified, `.sddw/` is relative to the current working directory. **Create** `.sddw/` if it does not exist. |
| **All other steps** | Read the Project path from `.sddw/<feature-name>/requirements.md`. Resolve `.sddw/` relative to the same root used by the requirements step. If `.sddw/` cannot be found, tell the user and suggest running the requirements step first. |

**Rules:**
- SHALL resolve the `.sddw/` base path **once** at the start of every step and use absolute paths for all reads and writes.
- SHALL resolve `.sddw/` from the current working directory, NOT the git root.
- When writing file paths in output or logs, use the resolved absolute path.
- Step-specific path behavior (creating directories, fallback messages) is noted in each step's instructions.

## Anti-patterns

- **Multiple questions at once** — never ask more than one question per message
- **Checklist walking** — going through items in order regardless of what the user said
- **Shallow acceptance** — taking vague answers without probing ("good" means what? "users" means who?)
- **Premature constraints** — asking about tech details before understanding the idea
- **Interrogation** — firing questions without building on answers
- **Script following** — asking the same questions regardless of context
