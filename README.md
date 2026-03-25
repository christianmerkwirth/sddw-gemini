# `sddw-gemini`

Spec-Driven Development Workflow for [Gemini CLI](https://github.com/google-gemini/gemini-cli).

- Write **requirements**, optionally **analyse the codebase**, then **design** (as self-contained task files), then **implement** each task separately
- The agent guides you through every step — researches, proposes options, confirms your decisions
- Every step produces exactly one spec type. Every step reads specs from previous steps.
- Three interaction modes: guided dialog (default), `--critical-only`, or fully `--auto`
- Lightweight and easily customizable — just markdown and TOML files, no runtime dependencies

Also available for [Claude Code](https://github.com/sermakarevich/sddw).

## Install

```bash
gemini extensions install https://github.com/sermakarevich/sddw-gemini
```

For development (link from local repo):

```bash
git clone https://github.com/sermakarevich/sddw-gemini
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
  code-analysis.md            # Optional shared codebase analysis
```

## License

MIT
