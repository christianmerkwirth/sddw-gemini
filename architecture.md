# The Architecture of `sddw`

`sddw` is a **spec-driven development workflow**, packaged here as a **Gemini CLI extension** (`gemini-extension.json`, `GEMINI.md`). Its cleverness is almost entirely *organizational*: there is no runtime, no code — just markdown files (and thin `.toml` command wrappers) arranged into a grid. Understanding it means understanding two axes.

> Note: this document describes the top-level Gemini extension. A parallel Antigravity (Jetski) port lives under `jetski-skills/`. It expresses the same grid as skill folders: each `SKILL.md` reads its questionnaire and the shared specs at run time, instead of the `.toml` + `!{cat ...}` inlining described below.

## The Big Idea: a 2D grid of markdown

Think of the whole system as a **table**.

- **Rows = Steps** (the pipeline): `requirements → code-analysis → design → taskify → implement → task-review → verify → self-improve`, plus aliases (`design_and_taskify`, `chat`, `help`).
- **Columns = Concerns** (the layers): `command`, `instructions`, `questionnaire`, `spec`.

Every cell is one small markdown file. A "step" like `requirements` is not a single file — it's a **vertical slice** assembled from one file in each column:

```
                 commands/sddw/  instructions/   questionnaires/   specs/
requirements  →  [entry.toml]    [process]        [dialog]          [format]
design        →  [entry.toml]    [process]        [dialog]          [format]
taskify       →  [entry.toml]    [process]        [dialog]          [format]
implement     →  [entry.toml]    [process]        [dialog]          [format]
task-review   →  [entry.toml]    [process]        [dialog]          [format]
   ...
common        →                  [shared rules]   ← spans all rows
```

This is **separation of concerns applied to a prompt**. Instead of one giant prompt per command, each command is decomposed by *what kind of knowledge it is*.

## The Four Columns

Each step is split into four files, each answering a different question:

| Column | File | Answers the question | Nature |
|--------|------|----------------------|--------|
| **Command** | `commands/sddw/<step>.toml` | *How is this invoked?* | Thin glue + wiring |
| **Instructions** | `instructions/<step>.md` | *What must happen, in what order, under what rules?* | Process logic |
| **Questionnaire** | `questionnaires/<step>.md` | *How do I talk to the human?* | Interaction script |
| **Spec** | `specs/<artifact>.md` | *What artifact must come out, in what shape?* | Output template |

(The first three columns are keyed by *step name*; the spec column is keyed by *artifact name* and is shared across steps — see below.)

Why this particular cut? Because these four things **change for different reasons and at different rates** — the classic test for a good module boundary:

- The **command** rarely changes (just the arg capture + `!{cat ...}` includes).
- The **spec** (output format) changes when you want differently-shaped artifacts.
- The **questionnaire** (dialog) changes when you want a smoother conversation.
- The **instructions** (rules) change when you discover process bugs.

You can improve *how the agent asks questions* without touching *what it produces*, and vice versa. That's the whole payoff.

### What each column actually contains

- **Command** (`commands/sddw/requirements.toml`) — A near-empty shell. It is a single `prompt = """..."""` string that captures `{{args}}`, then *inlines* the other files via Gemini's `!{cat ...}` shell-injection, and ends with a "NEXT STEP" hint. It is *pure composition*:
  ```toml
  prompt = """
  # COMMON RULES
  !{cat ${extensionPath}/instructions/common.md}        ← shared rules
  # INSTRUCTIONS
  !{cat ${extensionPath}/instructions/requirements.md}  ← process
  # QUESTIONNAIRE
  !{cat ${extensionPath}/questionnaires/requirements.md} ← dialog
  # SPECS
  !{cat ${extensionPath}/specs/requirements.md}         ← output format
  """
  ```
  A command may inline *several* specs when its job touches multiple artifacts (e.g. `implement` pulls both `design-task.md` and `task-completion.md`).
- **Instructions** — The imperative rules: the goal, the phase sequence, hard constraints (`SHALL use RFC 2119`, `SHALL NOT include implementation details`), and where to write output. This is the "policy" layer.
- **Questionnaire** — A *staged conversation script*: Phase 1 Discover, Phase 2 Research & Propose (section by section), Phase 3 Confirm & Generate. It even prescribes anti-patterns to avoid (no interrogation, no checklist-walking). This is the "UX" layer.
- **Spec** — A *fill-in-the-blanks template* with format blocks, rules, and worked examples for every section of the output document. This is the "schema" layer. Unlike the other columns, specs are named by **artifact**, not by step (`requirements.md`, `code-analysis.md`, `design.md`, `design-task.md`, `task-completion.md`, `task-review-report.md`, `verification-report.md`, `improvement-report.md`), so a single spec can be reused by several steps — the spec column is a shared artifact library, not a strict one-per-row mapping.

## The Two Cross-Cutting Pieces

Two files break the grid pattern on purpose:

1. **`instructions/common.md`** — the *shared base class*. Rules every step inherits: interaction modes (`--auto` vs interactive), path resolution for `.sddw/`, the mandate to use the `ask_user` tool, and global anti-patterns. Every command inlines it first. This is **DRY for prompts**: the `--auto` semantics live in one place, not duplicated across steps. (Counts: 11 commands, 12 instruction files, 9 questionnaires, 8 specs. `common` is the extra instruction file; the alias steps `help`/`chat` need neither questionnaire nor spec, and several specs are shared across steps — which is why the columns aren't equal height.)

2. **`self-improve`** — the *reflexive loop*. It reads a finished feature's artifacts (deviations, difficulties, remediation tasks) and proposes edits **back into the grid itself** — patching an instruction, a questionnaire, or a spec. The architecture is designed to be modified by its own output. Because concerns are cleanly separated, an improvement can target exactly one cell with a surgical diff.

## The Horizontal Axis: a pipeline of artifacts

The rows aren't independent — they form a **pipeline connected through files on disk**, not through conversation memory:

```
requirements.md → (code-analysis.md) → design.md → tasks/*.md → *.done.md → task-review/report.md → verify/report.md → self-improve/report.md
```

Three invariants make this work:

- **Every step produces exactly one spec type.** (one output shape per row)
- **Every step reads specs from previous steps.** (inputs come from disk, not chat)
- **`/clear` between steps.** Each step runs in a *fresh, focused context window*.

This is the real reason for the whole design: it's a strategy for **beating context-window limits**. A big feature won't fit in one conversation, so the work is chopped into stages where each stage reads only the artifacts it needs, does one job, writes one artifact, and clears. The filesystem (`.sddw/<feature>/...`) is the persistent memory between otherwise-amnesiac sessions.

## Two orthogonal interaction modes

Cutting across both axes is a mode switch (`--auto` vs interactive) defined once in `common.md`. The principle: **all modes do the same work** (discover, research, propose, decide) — the *only* difference is whether user approval gates each step. Interactive follows the questionnaire literally; `--auto` runs it autonomously. So the questionnaire is simultaneously a human script *and* an autonomous checklist.

## Why this architecture is good (in plain terms)

1. **Composability** — a step is assembled, not written. Building a new step means dropping four small files into four folders.
2. **Single responsibility per file** — each file has one reason to change; you edit dialog without risking output format.
3. **Reuse** — `common.md` is inherited everywhere; `design.md` is referenced by every task instead of being copied (the "hybrid task file" idea).
4. **Reviewability** — the *spec* is the deliverable, version-controlled and peer-reviewable before any code exists.
5. **Context discipline** — the pipeline + `/clear` keeps every step inside the window where the model is accurate.
6. **Self-evolution** — `self-improve` closes the loop by editing the grid that produced the work.

## One-sentence summary

> `sddw` is a **grid of small markdown files** — *steps × concerns* — where each command is composed (not coded) from a shared rulebook, a process spec, a dialog script, and an output template; the steps form a **file-based pipeline** run one-per-context-window, and the system can rewrite its own cells via a self-improvement step.
