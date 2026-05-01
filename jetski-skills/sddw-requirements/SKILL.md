---
name: sddw-requirements
description: >
  Generate a requirements specification for a feature using the sddw
  (Spec-Driven Development Workflow) pipeline. Use when the user says
  "sddw requirements", "create requirements for", "spec out a feature",
  or wants to start the sddw pipeline for a new feature.
  Produces .sddw/<feature>/requirements.md with Purpose, User Stories,
  Functional Requirements, Acceptance Criteria, and Constraints.
  Supports --auto flag for fully autonomous mode.
---

# sddw Requirements Step

Generate a requirements specification for a feature. This is Step 1 of the sddw workflow.

## Setup

1. **Parse arguments:** Extract `<feature-name>` and `--auto` flag from the user's message.
2. **Read common rules:** Read `../sddw-common/common-rules.md` and follow all rules throughout this step.
3. If no feature name is provided, ask the user to describe the feature they want to build.

## Goal

Produce a precise, clear, and complete requirements spec.

## Process

Follow the three-phase flow defined in `./references/questionnaire.md`, adapted to the interaction mode:

1. **Discover** — Ask the user to describe the feature. Follow the thread, challenge vagueness, make the abstract concrete. Gather enough context for Purpose, User Stories, and Constraints. *In `--auto`: perform discovery fully autonomously.*

2. **Research & Propose** — Based on discovery, research the problem space (web search, codebase analysis, domain knowledge). For each spec section, propose 2-3 ranked options with rationale. User accepts, modifies, or provides their own input. *In `--auto`: decide all sections autonomously.*

3. **Confirm & Generate** — Summarise what will be written. User confirms. Write the spec to `.sddw/<feature-name>/requirements.md` following the spec template at `../sddw-common/specs/requirements.md`. *In `--auto`: generate directly.*

## Rules

- SHALL use RFC 2119 keywords (SHALL, SHOULD, MAY, SHALL NOT)
- SHALL NOT include implementation details — that belongs in the design step
- SHALL NOT include task decomposition — that belongs in the design step
- Every FR SHALL be atomic, testable, and user-centric
- Every FR SHALL have at least one acceptance criterion
- Include explicit prohibitions (SHALL NOT) to prevent unwanted agent behaviour
- SHALL NOT proceed to generation without user approval on all sections (interactive mode). `--auto` mode may proceed without approval.

**Path note:** The requirements step is responsible for **creating** the `.sddw/` directory and the feature subdirectory if they do not exist.

## Output

```
<resolved-sddw-path>/<feature-name>/requirements.md
```

## Next Step

After the user approves the requirements, suggest:
> Run `/clear` to free up context.
> If you have an existing codebase, run `sddw code-analysis <feature-name>` to ground design decisions in the actual code.
> Otherwise, go straight to `sddw design-and-taskify <feature-name>` (recommended default).
>
> *Alternative for design review iteration:* Run `sddw design <feature-name>` to establish architecture, then `sddw taskify <feature-name>` to generate tasks.
