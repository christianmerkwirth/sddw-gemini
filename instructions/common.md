# Common Rules

These rules apply to all sddw steps.

## Interaction Modes

Parse `--auto` or `--critical-only` from the arguments. Default to interactive mode if neither is present.

| Mode | Flag | Behavior |
|------|------|----------|
| **Interactive** | *(default)* | Full guided dialog. One question at a time, every section confirmed. |
| **Critical-only** | `--critical-only` | Research and propose autonomously. Pause only for critical decisions (see below). Present all non-critical sections as a batch for quick review. |
| **Auto** | `--auto` | Fully autonomous. No questions asked. Use best judgment for all decisions. |

### What counts as critical

Critical decisions are those that are hard to reverse or that fundamentally shape the output:

| Step | Critical decisions | Non-critical |
|------|-------------------|--------------|
| **Requirement** | Project path, scope boundaries (in/out), prohibitions | Purpose framing, user story wording, FR priority order, acceptance criteria details, testing approach |
| **Code analysis** | Ambiguous conventions, conflicting patterns | Clear-cut patterns, interfaces, flows, naming conventions |
| **Design** | Architecture approach (new vs modify vs reuse), design decisions with trade-offs, task breakdown and dependencies | Code analysis update, data model details, interface contract specifics |
| **Implement** | Architectural deviations (Rule 4), dependency conflicts | Task selection (pick next unblocked), implementation approach, TDD applicability |

### Mode rules

All modes perform the same work — discover, research, propose, decide. The difference is only in what requires user input.

- **Interactive**: Follow the questionnaire as written — one question at a time, wait for approval on every section.
- **Critical-only**: Perform all phases autonomously. Make decisions on non-critical items using best judgment. Pause and ask only for critical decisions. Present the final summary before generation.
- **Auto**: Perform all phases autonomously. Make all decisions — including critical ones — using best judgment. Generate output directly. Still follow all spec format rules and quality standards.

### Safety

- In `--auto` mode for the **requirements** step: warn the user that requirements quality depends on input detail. If the feature description in the arguments is less than ~20 words, downgrade to `--critical-only` and ask for a more detailed description.
- In `--auto` mode: architectural deviations (Deviation Rule 4) during implement still STOP and ask — this overrides `--auto` because the risk of silent architectural changes is too high.

## Interaction (Interactive mode)

- Ask ONE question at a time. Wait for the user's response before asking the next.
- Use the user's answer to shape the follow-up — do not follow a script.
- Build understanding incrementally — each answer narrows the next question.
- Never dump multiple questions in a single message.
- SHALL NOT dump the spec template or full output structure to the user. Use the spec as internal guidance. Present proposals in conversational form, one block at a time, and confirm each before moving on.

## Path Resolution

All `.sddw/` references are **relative to the project root** — the git root of the target codebase being worked on.

| Step | Path resolution |
|------|----------------|
| **Requirements** | If the user specifies a Project path other than `.`, resolve `.sddw/` relative to that path's git root. If the Project path is `.` or unspecified, `.sddw/` is relative to the current working directory's git root. **Create** `.sddw/` if it does not exist. |
| **All other steps** | Read the Project path from `.sddw/<feature-name>/requirements.md`. Resolve `.sddw/` relative to the same root used by the requirements step. If `.sddw/` cannot be found, tell the user and suggest running `/sddw:requirements` first. |

**Rules:**
- SHALL resolve the `.sddw/` base path **once** at the start of every step and use absolute paths for all reads and writes.
- SHALL NOT assume `.sddw/` is in the current working directory — always resolve from the project root.
- When writing file paths in output or logs, use the resolved absolute path.
- Step-specific path behavior (creating directories, fallback messages) is noted in each step's instructions.

## Tool Usage — ask_user

**CRITICAL:** All user-facing questions MUST use the `ask_user` tool. Do NOT use plain text conversation turns to ask questions or present options.

| Question type | How to ask |
|--------------|------------|
| **Options / choices** (2-4 items) | `ask_user` with `type: "choice"` and `options` array with `label` and `description`. Add "(Recommended)" to the preferred option. Use `multiSelect: true` when choices are not mutually exclusive. Use `description` field for context (no `preview` field available). |
| **Open-ended question** (no predefined choices) | `ask_user` with `type: "text"` and optional `placeholder`. |
| **Yes/No confirmation** | `ask_user` with `type: "yesno"`. |

**Rules:**
- SHALL use `ask_user` for every question in interactive and critical-only modes — both option-based and open-ended.
- SHALL NOT present questions or options as plain text in the conversation and wait for a reply.
- The only text output between questions should be brief context, summaries, or research findings that set up the next question.

## Anti-patterns

- **Multiple questions at once** — never ask more than one question per message
- **Checklist walking** — going through items in order regardless of what the user said
- **Shallow acceptance** — taking vague answers without probing ("good" means what? "users" means who?)
- **Premature constraints** — asking about tech details before understanding the idea
- **Interrogation** — firing questions without building on answers
- **Script following** — asking the same questions regardless of context

---

## TDD Protocol

Check the Testing Approach in `.sddw/<feature-name>/requirements.md` Constraints section. Follow the user's chosen approach:
- **TDD** — always write tests first
- **Test-after** — implement first, add tests after
- **Selective TDD** — TDD for business logic/APIs, skip for config/glue code
- **No tests** — skip testing entirely

If Selective TDD or no preference specified, use TDD for tasks involving business logic, APIs, validation, data transformations, or algorithms. Skip TDD for UI layout, configuration, glue code, and simple CRUD.

**Heuristic:** Can you write `expect(fn(input)).toBe(output)` before writing `fn`? If yes, use TDD.

**RED — Write failing test:**
1. Create test file following project conventions
2. Write test describing expected behaviour from the acceptance criteria
3. Run test — it MUST fail
4. If test passes: feature already exists or test is wrong — investigate

**GREEN — Implement to pass:**
1. Write minimal code to make the test pass
2. No cleverness, no optimisation — just make it work
3. Run test — it MUST pass

**REFACTOR (if needed):**
1. Clean up implementation if obvious improvements exist
2. Run tests — MUST still pass

**Rules:**
- Limit refinement to 3-5 iterations — if still failing, escalate to user
- SHALL NOT modify tests to make them pass — fix the implementation instead
- If tests reveal a spec gap, update the task file or requirements first, then re-implement

## Commit Protocol

One task = one commit. Commit after tests pass, never before.

**Stage individually:**
```
git add src/specific/file.py
git add tests/specific/test_file.py
```

**Never use** `git add .` or `git add -A`.

**Message format:**
```
type(feature-name): description (FR-01, FR-02)
```

**Commit types:**

| Type | When |
|------|------|
| `feat` | New functionality |
| `fix` | Bug fix |
| `test` | Test-only (TDD RED phase) |
| `refactor` | No behaviour change (TDD REFACTOR phase) |
| `chore` | Config, dependencies |

**TDD tasks produce 2-3 commits:**
1. `test(feature): add failing test for X (FR-01)`
2. `feat(feature): implement X (FR-01)`
3. `refactor(feature): clean up X (FR-01)` (optional)

**Rules:**
- Every commit message SHALL reference the FR-IDs from the task file
- SHALL NOT commit partial implementations that break tests
- SHALL NOT commit unrelated changes in the same commit

## Deviation Handling

Deviations during implementation are normal. Classify and handle:

| Rule | Trigger | Action | Permission |
|------|---------|--------|------------|
| **1: Bug** | Broken behaviour, errors, type errors, security vulnerabilities | Fix → test → verify → document | Auto |
| **2: Missing Critical** | Missing essentials: error handling, validation, auth checks, input sanitisation | Add → test → verify → document | Auto |
| **3: Blocking** | Prevents completion: missing deps, wrong types, broken imports, missing config | Fix blocker → verify → document | Auto |
| **4: Architectural** | Structural change: new DB table, schema change, new service, switching libraries, breaking API | STOP → present to user → document | Ask user |

**Priority:** Rule 4 (STOP) > Rules 1-3 (auto) > unsure → Rule 4.

**Rules:**
- ALL deviations SHALL be documented (rule number, what was found, what was done)
- Auto-fixed deviations (Rules 1-3) SHALL be mentioned in the commit message
- If a deviation reveals a spec gap, note it in the task file for the verification step

## Completion Report

After the task commit(s), write a completion report following the task-completion spec:
`.sddw/<feature-name>/implement/tasks/task-<N>-<slug>.done.md`

Create the `implement/tasks/` directory if it does not exist. The report documents what was done, deviations, and difficulties. This enables `/sddw:help status` to show task completion and provides context for future tasks.

## Status Logic

### Status detection
- `requirements.md` exists → requirements done
- `.sddw/code-analysis.md` exists → code analysis done
- `design/tasks/task-N-*.md` → design done (count total tasks)
- `implement/tasks/task-N-*.done.md` → count completed tasks
- `verify/report.md` exists → verification done (read result from Summary)
- `self-improve/report.md` exists → self-improve done (read applied/skipped counts from Summary)

### Feature Status (`status <feature-name>`)

Read the feature directory and show detailed progress:

```
Feature: <feature-name>

  Requirements:    ✓ completed
    └─ .sddw/<feature-name>/requirements.md

  Code Analysis:   ✓ exists (last updated: <date>)
    └─ .sddw/code-analysis.md
    (or: ○ skipped — no code-analysis.md found)

  Design:          ✓ completed
    └─ .sddw/<feature-name>/design/tasks/

  Tasks:           2 of 4 complete
    1. task-1-<slug>    ✓ done
    2. task-2-<slug>    ✓ done
    3. task-3-<slug>    ○ pending
    4. task-4-<slug>    ○ pending (Depends on: task-3)

  Verification:      ✓ PASS (2026-03-25)
    └─ .sddw/<feature-name>/verify/report.md
    (or: ○ not yet run)
    (or: ✗ FAIL — 1 FR failed, 1 partial — 2 remediation tasks created)

  Self-Improve:      ✓ done — 2 applied, 1 skipped (2026-03-26)
    └─ .sddw/<feature-name>/self-improve/report.md
    (or: ○ not yet run)
```

For completed tasks, if a `.done.md` file exists, show a brief summary from it.

For pending tasks with dependencies, show the `Depends on:` field.
