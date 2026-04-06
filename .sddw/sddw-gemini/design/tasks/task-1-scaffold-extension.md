# Task 1: Scaffold extension manifest and context file

## Trace
- **FR-IDs:** FR-01
- **Depends on:** none

## Files
- `gemini-extension.json` — create
- `GEMINI.md` — create

## Architecture

### Components
- `gemini-extension.json`: Extension manifest — required entry point for `gemini extensions install` — new
- `GEMINI.md`: Ambient context file — auto-loaded by Gemini CLI when extension is active — new

### Data Flow
`gemini extensions install <url>` → reads `gemini-extension.json` → registers extension → `GEMINI.md` loaded into context when active

## Contracts

### gemini-extension.json
```json
{
  "name": "sddw",
  "version": "0.1.0",
  "description": "Spec-Driven Development Workflow for Gemini CLI. A 3-step pipeline: Requirement → Design → Implement. Specifications are the source of truth, code is a verified artifact.",
  "contextFileName": "GEMINI.md"
}
```

### GEMINI.md
Lightweight overview of the sddw workflow:
- What sddw is (spec-driven development workflow)
- The 4-step pipeline: requirements → code-analysis (optional) → design → implement
- Available commands: `/sddw:requirements`, `/sddw:code-analysis`, `/sddw:design`, `/sddw:implement`, `/sddw:chat`, `/sddw:help`
- Note that `.sddw/` directory holds all artifacts
- Reference to `ask_user` tool for structured interaction

## Acceptance Criteria

### FR-01: Extension manifest
- GIVEN the sddw-gemini repo
- WHEN a user runs `gemini extensions install`
- THEN the extension installs successfully and commands are available

- GIVEN the `gemini-extension.json` file
- WHEN parsed
- THEN it SHALL contain valid `name`, `version`, `description`, and `contextFileName` entries

## Done Criteria
- [ ] `gemini-extension.json` exists at repo root with valid JSON and all required fields
- [ ] `GEMINI.md` exists at repo root with workflow overview and command list
- [ ] `contextFileName` in manifest matches the actual filename `GEMINI.md`
