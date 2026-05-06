# AI.md Project Guide

`AI.md` is the project specification and the single source of truth for this project.

`IDEA.md` contains the project-specific description, variables, and business logic. `TODO.AI.md` is optional task tracking only and never overrides `AI.md`.

## Overview

This project is driven by specification-first workflow.

- `./AI.md` defines how work must be done.
- `./IDEA.md` defines what this specific project is supposed to do.
- `./TODO.AI.md` tracks work when needed.

If any file conflicts with `./AI.md`, `./AI.md` wins.

## Project Files

| File | Purpose |
|------|---------|
| `AI.md` | Project specification and source of truth |
| `IDEA.md` | Project-specific description, variables, and business logic |
| `TODO.AI.md` | Optional AI task tracking |

## AI Quick Start

Use the block below as a copy/paste prompt for GitHub Copilot, Claude Code, or another AI tool.

```text
You are working in the current project directory.

The project specification is ./AI.md. It is the single source of truth.

Before doing anything else:
1. Fully read ./AI.md from the top through PART 5.
2. Follow ./AI.md exactly as written.
3. Do not guess or assume when ./AI.md says to detect, read, verify, or ask.
4. If ./TODO.AI.md exists, review and update it as needed, but use ./AI.md for the actual rules and process.
5. Commit all COMMIT, NEVER, and MUST rules from ./AI.md to memory.
6. Before making changes, read the relevant sections of ./AI.md and follow them exactly.

If any file conflicts with ./AI.md, ./AI.md wins.
```

## Workflow

1. Read `./AI.md` through **PART 5** before starting work.
2. Review the relevant sections of `./AI.md` before making changes.
3. Use `./IDEA.md` for project-specific scope and business logic.
4. Update `./TODO.AI.md` when task tracking is needed.
5. Keep work aligned with the rules defined in `./AI.md`.

## Operating Rules

- Always treat `./AI.md` as authoritative.
- Never guess or assume values when the spec says to detect, read, verify, or ask.
- Read the relevant section of `./AI.md` before making changes.
- Use `./TODO.AI.md` only for task tracking, never as the source of truth.

## Disclaimer

This project guidance is provided "as is" without warranty of any kind.

- **No Warranty**: Use of the project and its instructions is at your own risk.
- **Project Rules Apply**: Operational, implementation, and workflow decisions must follow `./AI.md`.
- **Human Review Recommended**: Validate important changes before relying on them in production or other critical environments.

## License

MIT - See [LICENSE.md](LICENSE.md)
