# The Architecture of `sddw` (Agent Skills edition)

`sddw` is a **spec-driven development workflow**. This directory packages it as a set of **Agent Skills** for Antigravity / Jetski (per the README, installed into `~/.gemini/<env>/skills/` or a project's `.agent/skills/`). Like the Gemini extension, its cleverness is almost entirely *organizational*: there is no runtime, no code — just markdown files arranged into a grid. Understanding it means understanding two axes.

> Note: this document describes the **skills** variant in `jetski-skills/`. A parallel **Gemini CLI extension** lives at the repository root (`commands/ instructions/ questionnaires/ specs/`); it expresses the same grid but composes each step from `.toml` command files using `!{cat ${extensionPath}/...}` inlining. The key difference here is that a step is a *skill folder*, discovered by trigger phrase, that **reads its companion files at runtime** rather than having them concatenated upfront.

## The Big Idea: a 2D grid of markdown

Think of the whole system as a **table**.

- **Rows = Steps** (the pipeline): `requirements → code-analysis → design → taskify → implement → task-review → verify → self-improve`, plus aliases (`design-and-taskify`, `chat`, `help`).
- **Columns = Concerns** (the layers): `command`, `instructions`, `questionnaire`, `spec`.

Every cell is one small markdown file. A "step" like `requirements` is not a single file — it's a **vertical slice** assembled from the skill's own files plus the shared `sddw-common/` resources:

```
                   SKILL.md         references/        sddw-common/specs/   sddw-common/
                   (cmd+instr)      questionnaire.md   <artifact>.md        common-rules.md
requirements   →   [entry+process]  [dialog]           [format]             ┐
design         →   [entry+process]  [dialog]           [format]             │
taskify        →   [entry+process]  [dialog]           [format]             ├ shared,
implement      →   [entry+process]  [dialog]           [format]             │ spans
task-review    →   [entry+process]  [dialog]           [format]             │ all rows
   ...                                                                       ┘
```

This is **separation of concerns applied to a prompt** — but with one structural twist versus the Gemini extension: the **command and instructions columns are fused into a single `SKILL.md`**, because in the Agent Skills model the skill's frontmatter *is* the invocation surface.

## The Columns — and how they map to skill files

| Column | Where it lives | Answers the question | Nature |
|--------|---------------|----------------------|--------|
| **Command** | `SKILL.md` frontmatter (`name`, `description`) | *How is this invoked?* | Discovery metadata |
| **Instructions** | `SKILL.md` body | *What must happen, in what order, under what rules?* | Process logic |
| **Questionnaire** | `<skill>/references/questionnaire.md` | *How do I talk to the human?* | Interaction script |
| **Spec** | `sddw-common/specs/<artifact>.md` | *What artifact must come out, in what shape?* | Output template |

Why this cut still holds: these things **change for different reasons and at different rates** — the classic test for a good module boundary.

- The **command** surface (frontmatter description / trigger phrases) rarely changes.
- The **spec** (output format) changes when you want differently-shaped artifacts.
- The **questionnaire** (dialog) changes when you want a smoother conversation.
- The **instructions** (rules) change when you discover process bugs.

You can improve *how the agent asks questions* without touching *what it produces*, and vice versa. That's the whole payoff.

### What each column actually contains

- **`SKILL.md`** — the skill entry point. Its **frontmatter** (`name`, `description`) is the discovery contract: the agent matches the user's request ("sddw requirements", "spec out a feature", …) against the `description` and loads the skill. Its **body** is the imperative process — a Setup block that says *"Read `../sddw-common/common-rules.md` and follow all rules"*, a phase sequence (Discover → Research & Propose → Confirm & Generate), hard constraints (`SHALL use RFC 2119`, `SHALL NOT include implementation details`), and where to write output. This single file is both the *command* (how it's triggered) and the *instructions* (what to do).
- **`references/questionnaire.md`** — a *staged conversation script*: Phase 1 Discover, Phase 2 Research & Propose (section by section), Phase 3 Confirm & Generate, plus anti-patterns to avoid (no interrogation, no checklist-walking). The `SKILL.md` points to it (*"Follow the three-phase flow defined in `./references/questionnaire.md`"*) and the agent reads it on demand. This is the "UX" layer, co-located with each skill.
- **`sddw-common/specs/<artifact>.md`** — a *fill-in-the-blanks template* with format blocks, rules, and worked examples for every section of the output document. This is the "schema" layer. Specs are named by **artifact**, not by step (`requirements.md`, `code-analysis.md`, `design.md`, `design-task.md`, `task-completion.md`, `task-review-report.md`, `verification-report.md`, `improvement-report.md`), and a single spec is reused by several steps — so the spec column is a shared artifact library, not a strict one-per-row mapping.

## The composition mechanism: progressive disclosure, not concatenation

This is the deepest difference from the Gemini extension. There is no build-time include. Instead, `SKILL.md` contains **plain-English pointers** that instruct the agent to read companion files *when it needs them*:

```
Read `../sddw-common/common-rules.md` and follow all rules throughout this step.
Follow the three-phase flow defined in `./references/questionnaire.md`.
Write the spec ... following the spec template at `../sddw-common/specs/requirements.md`.
```

So the questionnaire and spec are loaded lazily, on demand — the **progressive-disclosure** pattern of Agent Skills — rather than `cat`-ed into one giant prompt at command-construction time. The relative paths (`../sddw-common/...`, `./references/...`) resolve because every skill folder, plus `sddw-common`, is installed side by side in the skills directory.

## The Two Cross-Cutting Pieces

Two resources break the grid pattern on purpose, and both live in the shared **`sddw-common/`** skill:

1. **`sddw-common/common-rules.md`** — the *shared base class*. Rules every step inherits: interaction modes (`--auto` vs interactive), path resolution for `.sddw/` (relative to CWD, resolved once, never the git root), the one-question-at-a-time interaction contract, how to present options, and global anti-patterns. Every `SKILL.md` instructs the agent to read it first. This is **DRY for prompts**: the `--auto` semantics live in one place, not duplicated across steps.

2. **`sddw-self-improve`** — the *reflexive loop*. It reads a finished feature's artifacts (deviations, difficulties, remediation tasks) and proposes edits **back into the grid itself** — patching a `SKILL.md`, a `questionnaire.md`, or a spec template. The architecture is designed to be modified by its own output. Because concerns are cleanly separated, an improvement can target exactly one cell with a surgical diff.

### Packaging detail: self-symlinks and install

Each skill folder contains a self-symlink (`sddw-<step>/sddw-<step> -> .`) and the README installs skills by symlinking every `sddw-*/` folder — plus `sddw-common/` — into the target skills directory. The self-symlink keeps the relative `../sddw-common/...` references resolvable regardless of how the skill is nested when linked. (These are recent-relative-path symlinks; earlier absolute self-symlinks were a known footgun, since fixed.)

## The Horizontal Axis: a pipeline of artifacts

The rows aren't independent — they form a **pipeline connected through files on disk**, not through conversation memory:

```
requirements.md → (code-analysis.md) → design.md → tasks/*.md → *.done.md → task-review/*.review.md → verify/report.md → self-improve/report.md
```

Steps 5 and 6 (`implement` → `task-review`) form a **per-task loop**: implement one task, review it, repeat for every task before feature-level `verify`.

Three invariants make this work:

- **Every step produces exactly one spec type.** (one output shape per row)
- **Every step reads specs from previous steps.** (inputs come from disk, not chat)
- **A new conversation between steps.** Each step runs in a *fresh, focused context window*.

This is the real reason for the whole design: it's a strategy for **beating context-window limits**. A big feature won't fit in one conversation, so the work is chopped into stages where each stage reads only the artifacts it needs, does one job, writes one artifact, and clears. The filesystem (`.sddw/<feature>/...`) is the persistent memory between otherwise-amnesiac sessions.

## Two orthogonal interaction modes

Cutting across both axes is a mode switch (`--auto` vs interactive) defined once in `common-rules.md`. The principle: **all modes do the same work** (discover, research, propose, decide) — the *only* difference is whether user approval gates each step. Interactive follows the questionnaire literally; `--auto` runs it autonomously. So the questionnaire is simultaneously a human script *and* an autonomous checklist. Two safety carve-outs override `--auto`: a too-thin requirements description downgrades to interactive, and architectural deviations during `implement` still stop and ask.

## Inventory

- **11 skill folders** (each with a `SKILL.md`): `requirements`, `code-analysis`, `design`, `taskify`, `implement`, `task-review`, `verify`, `self-improve`, plus aliases `design-and-taskify`, `chat`, `help`.
- **9 questionnaires** (`references/questionnaire.md`): every step except the `chat` and `help` aliases, which need no staged dialog.
- **8 specs** (`sddw-common/specs/`): `requirements`, `code-analysis`, `design`, `design-task`, `task-completion`, `task-review-report`, `verification-report`, `improvement-report`.
- **1 shared rulebook** (`sddw-common/common-rules.md`).

The columns aren't equal height: command+instructions are fused into `SKILL.md`, `chat`/`help` carry no questionnaire or spec, and several specs are shared across steps.

## Why this architecture is good (in plain terms)

1. **Composability** — a step is assembled, not written. Building a new step means dropping a `SKILL.md` + a `questionnaire.md` into a new skill folder and (if it emits a new artifact) a spec into `sddw-common/specs/`.
2. **Single responsibility per file** — each file has one reason to change; you edit dialog without risking output format.
3. **Reuse** — `common-rules.md` and the `specs/` library are shared by every skill instead of being copied; `design.md` is referenced by every task rather than duplicated.
4. **Progressive disclosure** — `SKILL.md` stays lean; the questionnaire and spec are pulled in only when the step actually needs them, keeping the working context tight.
5. **Reviewability** — the *spec* is the deliverable, version-controlled and peer-reviewable before any code exists.
6. **Context discipline** — the pipeline + a new conversation per step keeps every step inside the window where the model is accurate.
7. **Self-evolution** — `self-improve` closes the loop by editing the grid that produced the work.

## One-sentence summary

> The skills edition of `sddw` is a **grid of small markdown files** — *steps × concerns* — packaged as **Agent Skills** where each step is a discovered `SKILL.md` (command + instructions fused) that **reads on demand** a co-located dialog script and a shared spec template plus a shared rulebook; the steps form a **file-based pipeline** run one-per-context-window, and the system can rewrite its own cells via a self-improvement step.
