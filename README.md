# 🐹 claudemgr/go

Template specifications for CasjaysDev Go projects. Each file is a master template — copied into a generated project as `AI.md` — covering the full project lifecycle from layout and build system through CI/CD, testing, and release.

## Templates

| File | App type | When to use |
|------|----------|-------------|
| `API.md` | REST / JSON API server | HTTP services that expose a structured API; may include a companion CLI client |
| `APPLICATION.md` | Native GUI / TUI / CLI application | Single-binary desktop or terminal applications (Gio, Bubble Tea, Cobra) with no server component |
| `SERVER.md` | Full-stack web server | Server-side rendered HTML with optional REST endpoints; similar to API but ships a frontend |
| `HYBRID.md` | Native application + full server | Single-binary GUI/TUI/CLI application that also embeds a full server (frontend + backend, never called an "API server"); merges the entirety of `APPLICATION.md` and `API.md` |

## Files

| File | Purpose |
|------|---------|
| `API.md` | Go API server template — source of truth for API projects (PARTs 0–33) |
| `APPLICATION.md` | Go application template — source of truth for GUI/TUI/CLI projects (PARTs 0–13) |
| `SERVER.md` | Go web server template — source of truth for full-stack server projects (PARTs 0–37) |
| `HYBRID.md` | Go hybrid application+server template — source of truth for single-binary projects that are also a full server (PARTs 0–28) |
| `README.md` | This file |
| `LICENSE.md` | Repository license (WTFPL) |

## Related

`IDEA.md` and `CLAUDE.md` are not stored here — they are generated inside each project that uses one of these templates. Global implementation conventions live in `~/.claude/memory/` (go_conventions.md, makefile_conventions.md, testing_conventions.md, etc.).

## License

This repository (the templates themselves) is licensed under **WTFPL** — see [LICENSE.md](LICENSE.md).

Projects generated from these templates are licensed under **MIT** (each generated project ships its own `LICENSE.md`).
