# 🐹 claudemgr/go

Comprehensive Go project specification for Claude AI — `API.md` is the single source of truth for how any CasjaysDev Go project is built, structured, and deployed.

## 📄 What's in `API.md`

A ~43,000-line spec covering the full project lifecycle across 34 PARTs:

| Topic | Coverage |
|-------|----------|
| **Project layout** | All Go source under `src/`; singular package dirs (`handler/`, `model/`, `middleware/`) |
| **Build system** | Makefile for local dev / AI use; CI/CD uses direct `go build` commands — never `make` |
| **Docker** | `casjaysdev/go:latest` only; `CGO_ENABLED=0`; `-buildvcs=false`; module cache bind-mounts |
| **Binary naming** | `{name}-{GOOS}-{GOARCH}` — Go terms only (`linux/darwin/windows`, `amd64/arm64`) |
| **LDFLAGS** | `-s -w -trimpath` required on `build`, `release`, `docker` targets |
| **Auth** | Sessions, API tokens, RBAC, password hashing, CSRF/XSS/SSRF guards |
| **Server** | HTTP handlers, middleware chain, server-side Go templates (no client-side rendering) |
| **CLI / TUI** | Flag standards (`--debug`, `--color auto/yes/no`), NO_COLOR, cobra/pflag |
| **Config & logging** | `log/slog`, structured text handler, no ANSI in log files |
| **CI/CD** | GitHub Actions, GitLab CI, Gitea/Forgejo — SHA-pinned, direct cargo/go commands |
| **Release** | Cross-platform binaries, Docker image, SBOM, signed artifacts |
| **Testing** | Docker + Incus patterns, coverage enforcement (≥ 80%) |

## 🔑 Core Rules

- **Makefile = local development and AI-driven work only.** Targets: `build`, `release`, `docker`, `test`, `dev`, `clean`.
- **CI/CD never uses `make`.** All pipeline jobs run direct `go build`/`go test` inside `casjaysdev/go:latest`.
- **Never build on the host.** All compilation happens inside Docker.
- `CGO_ENABLED=0` on every `go build` invocation — no exceptions.
- `-buildvcs=false` required inline on all `go build` calls (and via `-e GOFLAGS=-buildvcs=false` in `GO_DOCKER`).

## 📦 Module Cache

| Variable | Local default | Container path |
|----------|--------------|----------------|
| `GO_CACHE` | `~/go/pkg/mod` | `/usr/local/share/go/pkg/mod` |
| `GO_BUILD` | `~/.cache/go-build` | `/usr/local/share/go/cache` |

## 📁 Files

| File | Purpose |
|------|---------|
| `API.md` | Full Go project specification — source of truth |
| `IDEA.md` | Project-scope description and variables |
| `CLAUDE.md` | Project loader — points Claude at `AI.md` and `IDEA.md` |

## 👤 Author

**Jason Hempstead** · [GitHub](https://github.com/casjay) · [Casjays Developments](https://casjaysdev.pro)

## 📄 License

MIT — see [LICENSE.md](LICENSE.md)
