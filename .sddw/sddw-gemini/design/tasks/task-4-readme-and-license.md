# Task 4: Add README and LICENSE

## Trace
- **FR-IDs:** FR-01
- **Depends on:** task-1

## Files
- `README.md` — create
- `LICENSE` — create (copy from source repo)

## Architecture

### Components
- `README.md`: Installation and usage documentation — new
- `LICENSE`: MIT license — copied from source repo

## Contracts

### README structure
1. **Banner/Title**: sddw-gemini — Spec-Driven Development Workflow for Gemini CLI
2. **Overview**: What sddw is, the 4-step pipeline, link to original Claude Code version
3. **Installation**: `gemini extensions install <github-url>` + dev mode with `gemini extensions link .`
4. **Usage**: Command reference table with all 6 commands and their arguments
5. **Workflow**: Brief description of requirements → code-analysis → design → implement flow
6. **Output structure**: `.sddw/<feature>/` directory layout
7. **Modes**: `--auto`, `--critical-only`, interactive mode descriptions
8. **License**: MIT

## Acceptance Criteria

### FR-01: Extension manifest (install documentation)
- GIVEN the README
- WHEN a user follows the installation instructions
- THEN they can install the extension via `gemini extensions install`

## Done Criteria
- [ ] `README.md` exists at repo root
- [ ] README includes installation instructions with `gemini extensions install` command
- [ ] README includes usage examples for all 6 commands
- [ ] README describes the three interaction modes
- [ ] README does not reference Claude Code-specific features
- [ ] `LICENSE` exists at repo root with MIT license text
