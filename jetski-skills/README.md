# sddw — Antigravity Skills

Spec-Driven Development Workflow for [Antigravity](https://antigravity.codes).

A structured 8-step pipeline where **specifications are the source of truth** and code is a verified artifact. The agent guides you through every step — researches, proposes options, confirms your decisions. Every step produces exactly one spec type. Every step reads specs from previous steps.

Also available for [Gemini CLI](https://github.com/christianmerkwirth/sddw-gemini) and [Claude Code](https://github.com/sermakarevich/sddw).

## Pipeline

```
sddw requirements  →  sddw code-analysis  →  sddw design  →  sddw taskify  →  sddw implement  →  sddw task-review  →  sddw verify  →  sddw self-improve
     (Step 1)            (Step 2, optional)     (Step 3)         (Step 4)         (Step 5)            (Step 6)           (Step 7)         (Step 8)
```

Steps 5 and 6 form a per-task loop: implement a task, then review it. Repeat for every task before moving on to feature-level verification.

| Step | Trigger Phrase | Output |
|------|---------------|--------|
| Requirements | `sddw requirements <feature>` | `.sddw/<feature>/requirements.md` |
| Code Analysis | `sddw code-analysis <feature>` | `.sddw/code-analysis.md` |
| Design | `sddw design <feature>` | `.sddw/<feature>/design/design.md` |
| Taskify | `sddw taskify <feature>` | `.sddw/<feature>/design/tasks/task-N-*.md` |
| Implement | `sddw implement <feature> --task N` | Code + `.sddw/<feature>/implement/tasks/task-N-*.done.md` |
| Task Review | `sddw task-review <feature> --task N` | `.sddw/<feature>/task-review/task-N-*.review.md` |
| Verify | `sddw verify <feature>` | `.sddw/<feature>/verify/report.md` |
| Self-Improve | `sddw self-improve <feature>` | `.sddw/<feature>/self-improve/report.md` |

Additional skills:
- `sddw design-and-taskify <feature>` — Combined alias: run design and taskify in one shot
- `sddw chat <feature>` — Fast-track interaction with a feature (quick edits, questions, updates)
- `sddw help` — Show workflow overview and feature status

## Install

### Prerequisites

- [Antigravity](https://antigravity.codes) installed and working
- Git

### Clone the repo

```bash
git clone https://github.com/christianmerkwirth/sddw-gemini.git
cd sddw-gemini
```

### Option A: Global install (recommended)

Makes sddw skills available in **all projects**. Run this script and select your environment:

```bash
echo "Select your environment:"
echo "1) Antigravity (~/.gemini/antigravity)"
echo "2) Antigravity CLI (~/.gemini/antigravity-cli)"
echo "3) Jetski (~/.gemini/jetski)"
echo "4) Jetski CLI (~/.gemini/jetski-cli)"
read -p "Choose (1-4): " choice

case $choice in
  1) D="antigravity";;
  2) D="antigravity-cli";;
  3) D="jetski";;
  4) D="jetski-cli";;
  *) echo "Invalid choice"; exit 1;;
esac

DEST="$HOME/.gemini/$D/skills"
mkdir -p "$DEST"

# Symlink each skill + the shared resources directory
for d in jetski-skills/sddw-*/; do
  ln -sf "$(pwd)/$d" "$DEST/$(basename "$d")"
done

# Also symlink the shared resources (specs and common rules)
ln -sf "$(pwd)/jetski-skills/sddw-common" "$DEST/sddw-common"

echo "Successfully installed to $DEST"
```

### Option B: Per-project install

Makes sddw skills available only in a **specific project**. Skills are installed into `<project>/.agent/skills/`.

```bash
# Run from your target project directory
PROJECT_DIR=$(pwd)
SDDW_DIR=/path/to/sddw-gemini  # adjust to where you cloned the repo

mkdir -p "$PROJECT_DIR/.agent/skills"

for d in "$SDDW_DIR"/jetski-skills/sddw-*/; do
  ln -sf "$d" "$PROJECT_DIR/.agent/skills/$(basename "$d")"
done

ln -sf "$SDDW_DIR/jetski-skills/sddw-common" "$PROJECT_DIR/.agent/skills/sddw-common"
```

> **Note:** The `sddw-common/` symlink is required — skills reference shared spec templates and common rules via `../sddw-common/` relative paths.

### Verify installation

After installing, restart Antigravity (or your IDE) to pick up the new skills. You can verify they're loaded by asking:

```
sddw help
```

The agent should display the workflow overview with all 8 steps.

## Uninstall

### Global

```bash
echo "Select your environment:"
echo "1) Antigravity (~/.gemini/antigravity)"
echo "2) Antigravity CLI (~/.gemini/antigravity-cli)"
echo "3) Jetski (~/.gemini/jetski)"
echo "4) Jetski CLI (~/.gemini/jetski-cli)"
read -p "Choose (1-4): " choice

case $choice in
  1) D="antigravity";;
  2) D="antigravity-cli";;
  3) D="jetski";;
  4) D="jetski-cli";;
  *) echo "Invalid choice"; exit 1;;
esac

rm -f "$HOME/.gemini/$D/skills"/sddw-*
echo "Successfully uninstalled from $HOME/.gemini/$D/skills"
```

### Per-project

```bash
rm -f .agent/skills/sddw-*
rm -f .agent/skills/sddw-common
```

## Skills

| Skill | Description |
|-------|-------------|
| `sddw-requirements` | Generate requirements spec through guided dialog |
| `sddw-code-analysis` | Analyse existing codebase patterns and conventions (optional) |
| `sddw-design` | Produce cross-cutting design.md with architecture and contracts |
| `sddw-taskify` | Break design into hybrid task files |
| `sddw-design-and-taskify` | Combined design + taskify in one flow |
| `sddw-implement` | Implement a single task with TDD, commit, and deviation protocols |
| `sddw-task-review` | Review a single implemented task against its criteria, design, and conventions |
| `sddw-verify` | Verify implementation against requirements, create remediation tasks |
| `sddw-self-improve` | Analyse execution and propose workflow improvements |
| `sddw-chat` | Fast-track interaction — quick edits, questions, status |
| `sddw-help` | Workflow overview and feature status |

## Interaction Modes

Every step supports two interaction modes:

| Mode | Flag | Behavior |
|------|------|----------|
| **Interactive** | *(default)* | Full guided dialog — one question at a time, every section confirmed |
| **Auto** | `--auto` | Fully autonomous — no questions, best-judgment decisions |

Pass `--auto` in your message to the agent, e.g.: `sddw requirements my-feature --auto`

## Directory Structure

```
sddw-skills/
├── sddw-common/                        # Shared resources (not a skill itself)
│   ├── common-rules.md                  # Interaction rules, path resolution, anti-patterns
│   └── specs/                           # Output format templates
│       ├── requirements.md
│       ├── code-analysis.md
│       ├── design.md
│       ├── design-task.md
│       ├── task-completion.md
│       ├── task-review-report.md
│       ├── verification-report.md
│       └── improvement-report.md
│
├── sddw-requirements/
│   ├── SKILL.md                         # Step instructions
│   └── references/questionnaire.md      # 3-phase dialog flow
│
├── sddw-code-analysis/
│   ├── SKILL.md
│   └── references/questionnaire.md
│
├── sddw-design/
│   ├── SKILL.md
│   └── references/questionnaire.md
│
├── sddw-taskify/
│   ├── SKILL.md
│   └── references/questionnaire.md
│
├── sddw-design-and-taskify/
│   ├── SKILL.md
│   └── references/questionnaire.md
│
├── sddw-implement/
│   ├── SKILL.md                         # Includes TDD, commit, deviation protocols
│   └── references/questionnaire.md
│
├── sddw-task-review/
│   ├── SKILL.md                         # Per-task quality gate; verdict + findings
│   └── references/questionnaire.md
│
├── sddw-verify/
│   ├── SKILL.md
│   └── references/questionnaire.md
│
├── sddw-self-improve/
│   ├── SKILL.md
│   └── references/questionnaire.md
│
├── sddw-chat/
│   └── SKILL.md                         # No questionnaire (freeform)
│
└── sddw-help/
    └── SKILL.md                         # No questionnaire (utility)
```

## Output Structure

All workflow artifacts live under `.sddw/` in your project root:

```
.sddw/
  <feature>/
    requirements.md           # Step 1 output
    design/
      design.md               # Cross-cutting design artefact
      tasks/
        task-1-<slug>.md      # One file per implementation task
        task-2-<slug>.md
    implement/
      tasks/
        task-1-<slug>.done.md # Completion reports
    task-review/
      task-1-<slug>.review.md # Per-task review reports (verdict + findings)
    verify/
      report.md               # FR-by-FR pass/fail, test results
    self-improve/
      report.md               # Findings, proposals, applied changes
  code-analysis.md            # Optional shared codebase analysis
```

## License

MIT
