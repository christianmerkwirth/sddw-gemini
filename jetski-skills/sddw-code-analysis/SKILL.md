---
name: sddw-code-analysis
description: >
  Analyse the existing codebase to extract patterns, interfaces, flows, and
  conventions. Use when the user says "sddw code analysis", "analyse the codebase",
  or wants to ground design decisions in existing code before running sddw design.
  This is an optional step between Requirements and Design.
  Produces .sddw/code-analysis.md (shared across features).
  Supports --auto flag for fully autonomous mode.
---

# sddw Code Analysis Step

Analyse the existing codebase to extract patterns, interfaces, flows, and conventions. This is an optional step between Requirements and Design. It grounds design decisions in reality rather than assumptions.

## Setup

1. **Parse arguments:** Extract `<feature-name>` and `--auto` flag from the user's message.
2. **Read common rules:** Read `../sddw-common/common-rules.md` and follow all rules throughout this step.

## Goal

Produce a shared, project-level codebase analysis.

## Prerequisites

Read the requirements spec produced by Step 1:
`<resolved-sddw-path>/<feature-name>/requirements.md`

If the requirements spec does not exist, inform the user and suggest running `sddw requirements <feature-name>` first.

Use the Project path from the requirements spec as the target codebase for analysis.

## Process

Follow the three-phase flow defined in `./references/questionnaire.md`:

1. **Discover** — Ask the user what areas of the codebase are most relevant, whether there are known conventions or gotchas. *In `--auto`: perform discovery fully autonomously.*

2. **Research & Propose** — Scan the codebase for patterns, interfaces, flows, and conventions relevant to the feature. Propose each section with findings. User accepts, modifies, or provides their own input. *In `--auto`: decide all sections autonomously.*

3. **Confirm & Generate** — Summarise what will be written. User confirms. Write the spec to `.sddw/code-analysis.md` following the template at `../sddw-common/specs/code-analysis.md`. *In `--auto`: generate directly.*

## Rules

- SHALL analyse the actual codebase, not assume patterns
- SHALL identify reusable components before proposing new ones in design
- SHALL document conventions the implementation must follow
- SHALL NOT propose patterns that conflict with existing codebase conventions
- SHALL note dependency direction between existing modules
- If `.sddw/code-analysis.md` already exists, SHALL review and update it rather than recreate
- SHALL add new sections relevant to the current feature without removing existing content
- SHALL NOT proceed to generation without user approval (interactive mode). `--auto` mode may proceed without approval.

## Output

```
.sddw/code-analysis.md
```

## Next Step

After the user approves the code analysis, suggest:
> Run `/clear` to free up context, then `sddw design <feature-name>`
