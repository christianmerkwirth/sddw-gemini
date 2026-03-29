# `sddw-gemini`

Spec-Driven Development Workflow for [Gemini CLI](https://github.com/google-gemini/gemini-cli).

- Write **requirements**, optionally **analyse the codebase**, then **design** (as self-contained task files), then **implement** each task separately, then **verify** the result, then **self-improve** the workflow
- The agent guides you through every step — researches, proposes options, confirms your decisions
- Every step produces exactly one spec type. Every step reads specs from previous steps.
- Three interaction modes: guided dialog (default), `--critical-only`, or fully `--auto`
- Lightweight and easily customizable — just markdown and TOML files, no runtime dependencies

Also available for [Claude Code](https://github.com/sermakarevich/sddw).

## Why

The standard way to use AI coding agents is short, interactive prompts: describe what you want, get code, fix it, repeat. This works for small tasks but breaks down for anything non-trivial — context gets lost between sessions, architectural decisions live only in chat history, and there's no artifact a teammate can review before code is written.

`sddw` inverts this. Instead of prompting for code, you collaborate with the agent to write **specifications** — requirements, architecture, interface contracts, task breakdowns. The specs become the primary artifact: reviewable by peers, version-controlled, persistent across sessions. Code generation is then a mechanical step guided by approved specs, not a creative leap from a vague prompt.

Detailed specifications reduce AI code errors by up to 50% (Piskala, 2026), security defects by 73% (Marri, 2026), and architecture-misaligned PRs by 60% (GitHub Spec Kit). `sddw` is designed for medium to large projects that don't fit into a single context window. By splitting work into discrete steps — requirements, codebase analysis, design, per-task implementation — each step operates within a focused context where models are more accurate, rather than a sprawling conversation where critical details get lost.

## Install

```bash
gemini extensions install https://github.com/christianmerkwirth/sddw-gemini
```

For development (link from local repo):

```bash
git clone https://github.com/christianmerkwirth/sddw-gemini
cd sddw-gemini
gemini extensions link .
```

## Commands

| Command | Description |
|---------|-------------|
| `/sddw:requirements <feature> [--auto \| --critical-only]` | Generate requirements spec |
| `/sddw:code-analysis <feature> [--auto \| --critical-only]` | Analyse existing codebase (optional) |
| `/sddw:design <feature> [--auto \| --critical-only]` | Generate self-contained task files |
| `/sddw:implement <feature> --task <N> [--auto \| --critical-only]` | Implement a single task |
| `/sddw:verify <feature> [--auto \| --critical-only]` | Verify implementation against requirements |
| `/sddw:self-improve <feature> [--auto \| --critical-only]` | Analyse execution and improve workflow |
| `/sddw:chat <feature> [--auto \| --critical-only]` | Fast-track interaction with an existing feature |
| `/sddw:help [list \| status <feature>]` | Workflow overview and feature status |

## Interaction Modes

Every step supports three interaction modes:

| Mode | Flag | Behavior |
|------|------|----------|
| **Interactive** | *(default)* | Full guided dialog — one question at a time, every section confirmed |
| **Critical-only** | `--critical-only` | Autonomous research and proposals, pauses only for critical decisions |
| **Auto** | `--auto` | Fully autonomous — no questions, best-judgment decisions |

Example: `/sddw:design my-feature --critical-only`

## Workflow

### 1. Requirements

```
/sddw:requirements <feature-name> [--auto | --critical-only]
```

Collaboratively produce a requirements spec through guided dialog:

- **Discover** — understand the feature through one-at-a-time questions
- **Research & Propose** — research SOTA, codebase, domain; propose each section with ranked options
- **Confirm & Generate** — user approves each block, spec is written

Output: `.sddw/<feature-name>/requirements.md`
Sections: Purpose, User Stories, Functional Requirements, Acceptance Criteria, Constraints

### 2. Code Analysis (optional)

```
/sddw:code-analysis <feature-name> [--auto | --critical-only]
```

Analyse the existing codebase to ground design decisions in reality:

- **Discover** — understand which areas of the codebase matter most
- **Research & Propose** — scan for patterns, interfaces, flows, conventions
- **Confirm & Generate** — user approves each section, analysis is written

Output: `.sddw/code-analysis.md` (shared across features)

Skip this step for greenfield projects with no existing codebase.

### 3. Design

```
/sddw:design <feature-name> [--auto | --critical-only]
```

Produce self-contained task files through guided dialog:

- **Discover** — understand architectural preferences and constraints
- **Research & Propose** — propose architecture, data models, contracts, decisions, and task breakdown
- **Confirm & Generate** — user approves each block, task files are written

Output:

```
.sddw/<feature-name>/
└── design/
    └── tasks/
        ├── task-1-<slug>.md  # self-contained: architecture, models, contracts, decisions, criteria
        ├── task-2-<slug>.md
        └── ...
```

Each task file includes all relevant design details inline so the implementation agent needs only that single file.

### 4. Implement

```
/sddw:implement <feature-name> --task <N> [--auto | --critical-only]
```

Execute a single task from the design spec:

- **Discover** — select task, check dependencies, gather context
- **Research & Propose** — scan codebase, propose implementation approach and TDD applicability
- **Execute** — implement following TDD protocol, commit protocol, and deviation handling

After each task, a completion report (`task-N-<slug>.done.md`) is written to `implement/tasks/`.

### 5. Verify

```
/sddw:verify <feature-name> [--auto | --critical-only]
```

Verify the implementation against requirements after all tasks are complete:

- **Assess** — load artifacts, detect test runner, check task completion status
- **Verify** — run test suite, cross-check each FR's acceptance criteria, review done criteria
- **Report & Remediate** — produce verification report, create remediation tasks if issues found

Output:

```
.sddw/<feature-name>/
└── verify/
    └── report.md    # FR-by-FR pass/fail, test results, deviations, warnings
```

If issues are found, remediation tasks are created as additional task files in `design/tasks/`. These can be executed with `/sddw:implement` and then verified again.

### 6. Self-Improve

```
/sddw:self-improve <feature-name> [--auto | --critical-only]
```

Analyse the completed feature's execution across all workflow steps. Identify what went wrong (or could be better) and propose concrete improvements to the workflow itself:

- **Analyse** — extract signals: deviations, difficulties, remediation task origins, spec gaps
- **Diagnose** — classify findings by workflow step, identify patterns, propose improvements
- **Apply** — present proposals with diff previews, apply approved changes to workflow files

Output:

```
.sddw/<feature-name>/
└── self-improve/
    └── report.md    # findings, proposals, applied/skipped changes
```

### Chat

```
/sddw:chat <feature-name> [--auto | --critical-only]
```

Fast-track interaction with a feature that already has artifacts. Skips the full questionnaire ceremony — load context and talk.

- **Questions** — ask anything about the feature
- **Spec updates** — edit requirements, FRs, acceptance criteria, or task files in-place
- **Quick implementation** — small code changes following TDD and commit protocols
- **Status** — check feature progress

### Help

```
/sddw:help [list | status <feature-name>]
```

- `/sddw:help` — workflow overview
- `/sddw:help list` — list all features with progress indicators
- `/sddw:help status <feature-name>` — detailed feature status

## Output Structure

All artifacts live under `.sddw/` in the project root:

```
.sddw/
  <feature>/
    requirements.md           # Step 1 output
    design/
      tasks/
        task-1-<slug>.md      # One file per implementation task
        task-2-<slug>.md
    implement/
      tasks/
        task-1-<slug>.done.md # Completion reports
    verify/
      report.md               # FR-by-FR pass/fail, test results
    self-improve/
      report.md               # Findings, proposals, applied changes
  code-analysis.md            # Optional shared codebase analysis
```

## License

MIT
