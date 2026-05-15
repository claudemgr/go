# {PROJECT_NAME} Specification

**Name**: {project_name}

**About this file:** `APPLICATION.md` is the master template. When applied to a project, this file is copied (or symlinked) into the project as `AI.md`. Throughout this document, all references to `AI.md` refer to that resulting file in a real project.

**Note:** `{PROJECT_NAME}` and `{project_name}` in this file are reference tokens, not setup-time text replacements. Their values are resolved from `IDEA.md ## Project variables` while `AI.md` remains read-only.

---

# 🆕 FIRST-TIME PROJECT SETUP

**`AI.md` is a read-only specification. Project-specific values live in `IDEA.md ## Project variables`, and the placeholders in this file are resolved from there.**

## Detecting Unconfigured Project Setup

```bash
# Project is not configured until IDEA.md exists and has required variables
[ ! -f IDEA.md ] && echo "SETUP NEEDED - IDEA.md missing"

have_name=$(grep -cE '^project_name:[[:space:]]*.+$' IDEA.md 2>/dev/null || true)
have_org=$(grep -cE '^project_org:[[:space:]]*.+$' IDEA.md 2>/dev/null || true)
have_internal_name=$(grep -cE '^internal_name:[[:space:]]*.+$' IDEA.md 2>/dev/null || true)
have_internal_org=$(grep -cE '^internal_org:[[:space:]]*.+$' IDEA.md 2>/dev/null || true)

[ "$have_name" -eq 0 ] || [ "$have_org" -eq 0 ] || [ "$have_internal_name" -eq 0 ] || [ "$have_internal_org" -eq 0 ] && \
  echo "SETUP NEEDED - IDEA.md project variables incomplete"
```

## Auto-Detecting Project Values

**Project name and org can be inferred automatically:**

| Value | Primary Source | Fallback |
|-------|----------------|----------|
| `{project_name}` | IDEA.md `## Project variables` | Existing long-form `CLAUDE.md` / `.claude/CLAUDE.md` project details, then `basename "$PWD"` |
| `{project_org}` | IDEA.md `## Project variables` | Existing long-form `CLAUDE.md` / `.claude/CLAUDE.md` project details, then `basename "$(dirname "$PWD")"` |
| `{internal_name}` | IDEA.md `## Project variables` (always — set once at first run, never edited after) | Existing long-form `CLAUDE.md` / `.claude/CLAUDE.md` project details, then first-time setup: copy from `{project_name}` |
| `{internal_org}` | IDEA.md `## Project variables` (always — set once at first run, never edited after) | Existing long-form `CLAUDE.md` / `.claude/CLAUDE.md` project details, then first-time setup: copy from `{project_org}` |
| `{plist_name}` | **Derived (not stored)**: `io.github.{internal_org}.{internal_name}` | — |

**Detection commands (use commands — never guess):**
```bash
# Project name: current directory name
project_name=$(basename "$PWD")

# Project org: parent directory name (assumes ~/org/project structure)
project_org=$(basename "$(dirname "$PWD")")

# Internal name: same as project_name on first run, frozen forever after
internal_name="$project_name"

# Internal org: same as project_org on first run, frozen forever after
internal_org="$project_org"

# Plist name: derived from frozen internal_org + internal_name (Bundle ID convention)
plist_name="io.github.${internal_org}.${internal_name}"
```

**Why separate `{internal_name}` and `{internal_org}`:** if a project or its org renames itself later, the new names apply to user-visible places (app title, docs, packaging names, store listing, website). But the frozen pair stays stable forever, keeping config/data/cache paths, desktop bundle identifiers, package IDs, keychain entries, and updater channels stable.

**Rule:** `{internal_name}` and `{internal_org}` are set ONCE at first-time setup and are immutable for the life of the project. Editing either later is a migration, not a routine rename. **Anything written to disk on the user's machine (config dirs, data dirs, Bundle IDs, package identifiers) MUST use the frozen pair, never `{project_name}` or `{project_org}`.**

## First-Time Setup Flow

```
AI reads AI.md for the first time
│
├─► Check: Does IDEA.md exist with required `## Project variables` entries?
│   │
│   ├─► NO (setup needed)
│   │   │
│   │   ├─► 1. Check if IDEA.md exists
│   │   │   ├─► YES: Read `## Project variables`; if incomplete, fill only the missing required values
│   │   │   └─► NO: Check existing `CLAUDE.md` and `.claude/CLAUDE.md` for valid project-specific details, then fall back to directory structure commands — never guess
│   │   │
│   │   ├─► 2. Create IDEA.md if it doesn't exist
│   │   │   - If a long-form/project-specific `CLAUDE.md` or `.claude/CLAUDE.md` already exists, MIGRATE its valid project description, project variables, and business logic into IDEA.md first
│   │   │   - Do NOT copy loader-only instructions, duplicated AI.md rules, or stale implementation text into IDEA.md
│   │   │   - On creation, write `internal_name: <project_name>` and `internal_org: <project_org>` to `## Project variables` and warn that both are frozen forever
│   │   │
│   │   ├─► 3. Create or update IDEA.md `## Project variables`
│   │   │   - project_name  → actual project name (lowercase)
│   │   │   - project_org   → actual org name (lowercase)
│   │   │   - internal_name → on first run = project_name; afterwards read from IDEA.md, IMMUTABLE
│   │   │   - internal_org  → on first run = project_org;  afterwards read from IDEA.md, IMMUTABLE
│   │   │   - Derived UPPERCASE placeholders are computed from these values when referenced
│   │   │   - {plist_name} is derived as io.github.{internal_org}.{internal_name} and is NOT stored
│   │   │
│   │   └─► 4. Proceed with normal operation once IDEA.md is valid
│   │
│   └─► YES (already configured)
│       └─► Proceed with normal operation - read PART 0 first and resolve placeholders from IDEA.md as needed
```

## Placeholder Reference

**These placeholders are reference tokens used by the spec. They are resolved from `IDEA.md ## Project variables` and are not meant to be manually rewritten throughout `AI.md` during project setup.**

| Placeholder | Case | Mutability | Example |
|-------------|------|------------|---------|
| `{project_name}` | lowercase | Mutable (project may rename) | `myapp` |
| `{PROJECT_NAME}` | UPPERCASE | Mutable | `MYAPP` |
| `{project_org}` | lowercase | Mutable | `myorg` |
| `{PROJECT_ORG}` | UPPERCASE | Mutable | `MYORG` |
| `{internal_name}` | lowercase | **Frozen** at first-time setup | `myapp` |
| `{INTERNAL_NAME}` | UPPERCASE | **Frozen** | `MYAPP` |
| `{internal_org}` | lowercase | **Frozen** at first-time setup | `myorg` |
| `{INTERNAL_ORG}` | UPPERCASE | **Frozen** | `MYORG` |
| `{plist_name}` | derived | Derived from `{internal_org}` + `{internal_name}` (both frozen) | `io.github.myorg.myapp` |

**Use the frozen pair (`{internal_org}`, `{internal_name}`) for anything stored on disk, registered with the OS, or used as a stable identifier (Bundle IDs, package IDs, keychain entries, dbus names, updater channels). Use the mutable pair (`{project_org}`, `{project_name}`) only for user-visible cosmetic surfaces (app title, README, store listing, website).**

**After setup, this section remains reference-only. The placeholders above are resolved from `IDEA.md ## Project variables`; `AI.md` itself stays read-only.**

---

# PROJECT DESCRIPTION

**See `IDEA.md` for project-specific details.**

---

# SOURCE OF TRUTH AND IDEA.md PRECEDENCE

**See `IDEA.md` for features, data models, and business rules.**

IDEA.md is the project PLAN. AI.md (this file) is the SOURCE OF TRUTH.

| File | Role | Update When |
|------|------|-------------|
| **AI.md** | SOURCE OF TRUTH - implementation rules | Optional→required policy changes only |
| **IDEA.md** | PROJECT PLAN - must follow AI.md | Features change, project variables change |

**Rule:** If IDEA.md conflicts with AI.md, AI.md wins. Fix IDEA.md.

## IDEA.md Required Layout

**Every IDEA.md MUST have exactly these three top-level sections, in this order. For a fillable template plus worked examples (Notes / Feeds / dotctl), see PART 13 → "IDEA.md REFERENCE".**

```markdown
## Project description

(Full project description — what the project is, who uses it, what problem it solves.)

## Project variables

(All project variables in `key: value` form. Required keys at minimum: `project_name`,
`project_org`, `internal_name`, `internal_org`. Add more as the project needs — `app_name`,
`official_site`, `maintainer_name`, `maintainer_email`, `module_path`, etc.)

Example:

    project_name:  notes
    project_org:   casjay
    internal_name: notes        # FROZEN — set once at first-time setup, never edit
    internal_org:  casjay       # FROZEN — set once at first-time setup, never edit
    app_name:      Notes
    module_path:   github.com/casjay/notes
    official_site: https://notes.example.com

## Business logic

(Full business spec — the WHAT, not the HOW. Features, data models, user flows,
permission rules, business invariants, platform targets, input modes, accessibility,
security assumptions, and any exceptions.)
```

**Rules for `## Project variables`:**
- One variable per line: `key: value`
- Keys are **lower_snake_case** only
- The setup flow renders `{KEY_UPPER}` automatically by uppercasing the lowercase key
- Never guess values: use commands and existing files
- If a placeholder referenced by AI.md has no entry in `## Project variables`, setup MUST stop and ask instead of inventing a value

**Rules for `## Business logic`:**
- It MUST define the actual product scope for THIS project - not generic boilerplate
- It MUST state which app surfaces exist: GUI, TUI, CLI, or a subset
- It MUST define user flows, stored data, trust boundaries, abuse cases, and platform constraints
- If a security-sensitive choice is intentionally allowed, the reason MUST be documented there

## Migrating Existing `CLAUDE.md` Into `IDEA.md`

**If a repository already has a pre-template `CLAUDE.md` or `.claude/CLAUDE.md` with real project details, those project details MUST be migrated into `IDEA.md`.**

**What belongs in `IDEA.md`:**
- project description / elevator pitch
- project-specific terminology
- project variables that can be expressed as `key: value`
- business logic, roles, flows, constraints, trust boundaries, abuse cases, and security exceptions

**What does NOT belong in `IDEA.md`:**
- generic Claude/Copilot usage instructions
- loader boilerplate whose job is only to point at `AI.md`
- duplicated global implementation rules that already live in `AI.md`
- stale code snippets, one-off notes, or tool chatter with no business/spec value

**Migration rules:**
1. Read existing `CLAUDE.md` and `.claude/CLAUDE.md` first - never overwrite blindly
2. Extract valid project-specific content and reorganize it into the required `IDEA.md` layout
3. Normalize discovered variables into lower_snake_case `key: value` entries
4. If `internal_name` cannot be proven, initialize it to `project_name` on first migration and treat it as frozen after that. Do the same for `internal_org` ← `project_org`.
5. If statements from `CLAUDE.md` or `.claude/CLAUDE.md` conflict with `AI.md`, `AI.md` wins
6. After migration, keep root `CLAUDE.md` and/or `.claude/CLAUDE.md` only as short efficient loaders and keep the real plan/spec in `IDEA.md`
7. Never silently discard meaningful project-specific content; migrate it, trim it, or explicitly ask where it belongs

---

# PART 0: CRITICAL RULES - READ FIRST

## THIS IS A STRICT SPECIFICATION - NOT GUIDELINES

- Every item in this specification MUST be followed exactly unless explicitly marked optional
- This is not a suggestion document
- There are no silent exceptions
- If the spec says X, do X - not "improved X"
- If something seems wrong, follow it and flag it; do not silently rewrite intent

## ⚠️ CRITICAL: File Paths and Project Root

- All paths are relative to the project root unless explicitly noted
- Do not scatter top-level files unnecessarily
- Runtime-generated files are not committed
- AI must not move the project root or invent sibling repositories

## ⚠️ CRITICAL: AI.md is the Source of Truth

- `AI.md` is read-only during routine work
- `IDEA.md` is where project-specific values and product rules live
- Loader files (`CLAUDE.md`, `.claude/CLAUDE.md`) stay short and point back to `AI.md`
- If a loader file and `AI.md` disagree, `AI.md` wins

## ⚠️ CRITICAL: Keep Documentation in Sync

Update these when their subject changes:
- `IDEA.md` when features or variables change
- `README.md` when install, usage, or packaging changes
- `LICENSE.md` when dependencies or attribution changes
- CI/CD docs or scripts when release mechanics change

## ⚠️ CRITICAL: One Coherent Product

This template defines **one Go application** with shared core logic and up to three presentation layers:
- GUI (preferred when available)
- TUI (fallback for interactive terminals)
- CLI (fallback for non-interactive/plain execution)

Single-process, single binary. The app may make outbound network calls to remote services it consumes; it does not host any.

## ⚠️ CRITICAL: No Host Toolchain or Binary Execution

The Go toolchain and any compiled artifact from this project MUST NOT run on the host machine.

- Never invoke `go`, `go build`, `go test`, `go run`, `go fmt`, `go vet`, `golangci-lint`, `govulncheck`, `go install`, or any project binary built from source directly on the host
- Every build, test, lint, format-check, doc-gen, run, install, package, and manual debug session executes inside a project-provided Docker container
- The host's role is limited to editing source files, version control, and orchestrating Docker
- CI and local commands documented in this spec are illustrative shapes; the **actual invocation** is always Docker-wrapped (see PART 5 → "Docker Rule")
- If a contributor's environment cannot run Docker, they cannot build/test this project — that is intentional, not a bug

This rule has no opt-out. There is no "just this once" exception for `go test` on the host.

## ⚠️ CRITICAL: Go-Only Application

This project's source code is **exclusively Go**.

- All application code, library code, build automation, and test code in this repository is written in Go
- No C, C++, Objective-C, Swift, Python, JavaScript, TypeScript, or shell-script source files contribute to the produced binary
- `Makefile` and `scripts/` are used for build automation — they orchestrate Docker invocations but MUST NOT contain application logic
- Small `docker/entrypoint.sh` and `docker/` shell helpers are tolerated because they orchestrate the container, not the application; they MUST NOT contain application logic
- `CGO_ENABLED=0` ALWAYS — pure Go, no C, no exceptions
- Third-party packages that are pure Go are strongly preferred; packages that vendor C are allowed only when (a) no pure-Go equivalent is viable, (b) the C code is statically compiled into the final binary (c) it does NOT require a system C lib at runtime, and (d) the dependency is documented in `IDEA.md` and `LICENSE.md`
- **Prefer pure-Go packages whenever a viable one exists.** Pure-Go packages are what make the single-static-binary rule and cross-platform GUI (Windows / macOS / Linux / BSD) achievable in practice — every C dependency dragged in becomes a portability and build-system tax. See PART 5 → "Pure-Go Library Stack" for the recommended package list
- Never introduce a build that requires a system C/C++ toolchain on the user's machine — the project Docker image is the only required build environment

## ⚠️ CRITICAL: Single Static Binary

The deliverable for each supported target is **one statically linked binary** that runs without external runtime dependencies on the user's system beyond the kernel and (where applicable) display server sockets.

**Required build outputs per target:**

| Target | Linkage | Notes |
|--------|---------|-------|
| `GOOS=linux GOARCH=amd64 CGO_ENABLED=0` | fully static | Default Linux release target |
| `GOOS=linux GOARCH=arm64 CGO_ENABLED=0` | fully static | Default ARM64 Linux release target |
| `GOOS=windows GOARCH=amd64 CGO_ENABLED=0` | fully static | Single `.exe` with no runtime DLL dependency |
| `GOOS=windows GOARCH=arm64 CGO_ENABLED=0` | fully static | Single `.exe` |
| `GOOS=darwin GOARCH=amd64 CGO_ENABLED=0` | system frameworks only (Apple does not allow static libSystem) | "Static" here means: no third-party dynamic libraries; only Apple-provided frameworks resolved at runtime |
| `GOOS=darwin GOARCH=arm64 CGO_ENABLED=0` | system frameworks only | Same rule as amd64 macOS |

**Rules:**
- No `glibc` runtime dependency on Linux release artifacts — `CGO_ENABLED=0` produces a fully static binary on Linux without needing musl
- No third-party `.so` / `.dylib` / `.dll` shipped alongside the binary
- No `LD_LIBRARY_PATH`, `DYLD_LIBRARY_PATH`, or wrapper-script tricks to find runtime libs
- `go build` inside the Docker image MUST produce a binary that passes a "no unexpected dynamic deps" check (`ldd`, `otool -L`, `dumpbin /dependents`) appropriate to the target
- Plugin systems, `dlopen` of arbitrary user code, and runtime extension loading from disk are forbidden unless IDEA.md explicitly defines a hardened plugin contract

## ⚠️ CRITICAL: Self-Contained Assets

The single binary contains **everything the app needs to function**. The user is not required to install fonts, themes, icons, templates, schemas, locales, or any other support file separately.

- Embed assets at build time using `//go:embed` (standard library `embed` package)
- Fonts, icons, theme data, default config, JSON/YAML schemas, SQL migrations, web assets (if any), localization catalogs, default templates, and licenses-to-display all live inside the binary
- The `assets/` directory in the repo is a **build-time source tree** for embedding — it is not a runtime install target
- The app may **read** user-provided config and data from per-user paths (PART 4 → "Path Rule"), and may **write** cache/state there, but it must run with full default functionality when those paths are empty
- Network/CDN fetches at first run to "download missing assets" are forbidden
- A new install on an air-gapped machine MUST work end-to-end with only the binary present

## Licensing & Features

| Rule | Description |
|------|-------------|
| **MIT License** | All project code is MIT licensed unless IDEA.md explicitly states an additional compatible license policy |
| **3rd party attribution** | All third-party licenses are listed in `LICENSE.md`. Generation, allowlist/denylist, IDEA.md exception path, and the user-visible licenses surface live in PART 11 → "License Compliance" |
| **GPL / AGPL / LGPL denied by default** | Static linking would relicense the distributed binary away from MIT. Allowed only via a documented IDEA.md exception (PART 11 → "License Compliance") |
| **Free & open source** | No paid tiers, enterprise gating, or artificial feature segmentation |
| **No premium features** | GUI, TUI, and CLI surfaces expose the same core product capabilities where applicable |
| **No activation gates** | No license keys, phone-home unlocks, or paywalled code paths |

**NEVER implement:**
- upgrade/paywall prompts for core functionality
- hidden features enabled by payment tier
- telemetry-based licensing enforcement
- artificial limits used for monetization

## Reduced Stop-and-Ask Behavior

AI should be autonomous by default.

**Ask only when one of these is true:**
1. a required project variable cannot be proven from files/commands
2. the requested change is destructive or irreversible and intent is ambiguous
3. a legal, licensing, or security policy conflict cannot be resolved from the spec
4. privileged/system-wide changes are needed but the user did not explicitly request them
5. a migration would discard meaningful existing project-specific content

**Do not stop just because there are multiple reasonable implementation details.** Pick the most spec-compliant option and continue.

---

# PART 1: PROJECT FILES & GOVERNANCE

## Project Files

| File | Purpose | Update When |
|------|---------|-------------|
| **AI.md** | Implementation spec (HOW) - SOURCE OF TRUTH | Optional→required rule changes only |
| **IDEA.md** | Project plan (WHAT) | Features or variables change |
| **TODO.AI.md** | Task tracking (AI-owned) | Tasks added/completed |
| **TODO.md** | Task tracking (human-owned) | AI may mark done; never delete/empty |
| **PLAN.AI.md** | Implementation plan (AI-owned) | Planning new work |
| **PLAN.md** | Implementation plan (human-owned) | AI may mark done; never rewrite wholesale |
| **README.md** | User-facing install/usage docs | Usage changes |
| **LICENSE.md** | Project + dependency licenses | Dependency set changes |
| **release.txt** | Canonical release version when present | Release version changes |
| **site.txt** | Optional official site/homepage URL | Official site changes |

## Mandatory Compliance Schedule

| When | Action | Purpose |
|------|--------|---------|
| Session start | Read AI.md completely | Understand full context |
| Before each task | Re-read relevant part(s) | Prevent drift |
| Every 3-5 changes | Stop and verify against spec | Catch drift early |
| Before task completion | Full compliance check | Ensure correctness |
| When uncertain | Re-read spec first | Never guess needlessly |

## Self-Validation Loop

**AI MUST verify its own work with real tools before reporting a task as done. Do not rely on "the code looks right."**

**This rule applies to EVERY change type covered by this template — library/API logic, GUI/TUI/CLI binaries, single-static-binary build, asset embedding, Docker, CI/CD, configuration, documentation, security — not only one category.** Whatever you touched, you verify.

Getting code correct on the first try is much harder than iterating with feedback. Close the loop every time. All execution goes through the project's containerised targets — never bare host Go toolchain.

| Change type | How to verify |
|-------------|---------------|
| Library / API logic | Run the project's test target inside the container; exercise the API directly; compare output against expected |
| Behavior-preserving refactor | Diff outputs of old vs. new path on representative inputs (don't trust that the diff "looks right") |
| CLI binary | Run the binary in the container; exercise relevant flags including `--help`/`--version`; check stdout, stderr, and exit code |
| TUI binary | Run in the container; verify rendering, keyboard input, and screen redraw on resize/exit |
| GUI binary | Run in the container; verify the rendered window under both X11 and Wayland forwarding; confirm input events reach the app |
| Single static binary requirement | Confirm the artifact is a single self-contained file and that it runs on a clean container with no extra runtime install |
| Asset embedding | Confirm assets are loaded from the binary itself (not from a host path); test on a container without the source tree mounted |
| Performance change | Measure before AND after — don't assume parallelism, caching, or "cleaner" code is faster |
| Bug fix | Reproduce the bug FIRST so you have a failing signal, then verify the fix makes it disappear; add a regression test where feasible |
| Configuration / settings | Start the binary with the new config; verify defaults; verify validation rejects bad input with a useful error |
| Docker / container build | Build the image; run the container; smoke-test the binary inside it; for GUI, verify display forwarding still works |
| CI/CD workflow | Run the workflow on a branch (or equivalent dry-run); verify each job's exit status, not just YAML validity |
| Logging / error paths | Trigger the error path; verify the log line/structured event was emitted with expected fields |
| Security-sensitive change (auth, crypto, input validation, plugin contracts) | Test both the success path AND attempted bypass paths; never assume a guard works without exercising it |
| Documentation / README | Render markdown locally; verify links, code samples, and example commands actually work |
| Type / lint / build correctness | The project's containerised check, golangci-lint, and build targets — green across all |

**Iteration rules:**
- A failed check is data, not failure — adjust and re-run until green
- Never report "done" while any verification is still red
- If a check reveals the change is wrong in a way that can't be patched, revert and re-plan; do not paper over a failing check
- When verification is genuinely impossible in this environment (no display, no DB, no network): say so explicitly. List what was checked and what could not be, so the user knows where to look

**Reference:** based on published guidance about AI coding agent self-validation (Eivind Kjosbakken, Towards Data Science, 2026) — when an AI agent is given verification tools (output diffing, browser/display, test runners) and allowed to iterate, one-shot success rate, run length, and task complexity all improve substantially.

## Commit Message File

If changes were made, update `.git/COMMIT_MESS` to reflect the actual current uncommitted changes.

**Rules:**
- Create/update it only when files changed
- It must match `git status --porcelain` and `git diff --stat`
- Recreate it if stale
- Never describe changes that are no longer present

## Loader Files

| Tool | Primary Loader | Alternate Loader | Personal Override |
|------|----------------|------------------|------------------|
| Claude Code | `CLAUDE.md` | `.claude/CLAUDE.md` | `CLAUDE.local.md` |

**Loader rule:** loader files stay short. Long-form product content belongs in `IDEA.md`; long-form implementation policy belongs in `AI.md`.

---

# PART 2: GO APPLICATION MODEL

## Product Model

This template targets a **single-binary, fully self-contained Go application** that may expose:
- a native GUI
- a terminal UI (TUI)
- a plain CLI

The application may make outbound network calls to consume remote services it depends on (APIs, databases, object stores, update endpoints, etc.).

**Distribution model:** one statically linked binary per supported target. Everything the app needs at runtime — UI assets, fonts, icons, default config, schemas, templates, locales — is embedded inside that binary. See PART 0 → "Single Static Binary" and "Self-Contained Assets."

## Architectural Rule

Use **one shared core application layer** and thin presentation adapters:

```text
main.go                     # auto-detects GUI/TUI/CLI mode unless explicitly overridden
                            # (or cmd/{project_name}/main.go for multi-binary workspaces)
internal/
├── app/                    # shared domain/application logic
├── config/                 # configuration loading + defaults
├── platform/               # OS/platform integration
├── state/                  # app state and persistence adapters
├── support/                # logging, errors, utilities
└── ui/
    ├── gui/                # optional native GUI
    ├── tui/                # optional terminal UI
    └── cli/                # optional plain CLI commands/output
```

(See PART 5 → "Project Layout" for the full repository tree, including `docker/`, `assets/`, `Makefile`, `scripts/`, etc.)

**Rules:**
- Core behavior MUST live in shared packages, not be duplicated across GUI/TUI/CLI
- UI layers are adapters over the same business rules
- Optional `cmd/*/main.go` utilities are allowed only for real auxiliary tools, not as a substitute for clean architecture

## Binary Model

| Binary | Status | Purpose |
|--------|--------|---------|
| `{project_name}` | REQUIRED | Primary application binary — single statically linked artifact |
| `cmd/*` helpers | DISCOURAGED | Permitted only when IDEA.md justifies a real auxiliary tool; each helper is itself a single static binary |

**Binary naming rules:**

Distribution artifact names follow the schema:

```
{project_name}-{platform}-{arch}{.ext}
```

| Token | Allowed values |
|-------|----------------|
| `{platform}` | `linux`, `windows`, `macos`, `freebsd`, `netbsd`, `openbsd` (lowercase, normalized OS name — never the raw `GOOS` value where it differs, e.g., `darwin` maps to `macos`) |
| `{arch}` | `x86_64`, `aarch64`, `armv7`, `i686`, `riscv64`, … (architecture name normalized to match the `GOARCH` output) |
| `{.ext}` | `.exe` on Windows; empty everywhere else |

**Normalization rules:**
- No `-musl` suffix on artifact names. `CGO_ENABLED=0` Go binaries are fully static on Linux without needing a musl target triple — there is nothing to call out in the filename.
- `GOOS=darwin` maps to platform token `macos`.
- `GOOS=windows` maps to platform token `windows`.
- `GOOS=linux` maps to platform token `linux`.
- Strip any OS suffix that does not correspond to a meaningful user-facing distinction.

**Worked examples:**

| GOOS / GOARCH | Artifact name |
|---------------|---------------|
| `linux` / `amd64` | `{project_name}-linux-x86_64` |
| `linux` / `arm64` | `{project_name}-linux-aarch64` |
| `linux` / `arm` (GOARM=7) | `{project_name}-linux-armv7` |
| `windows` / `amd64` | `{project_name}-windows-x86_64.exe` |
| `windows` / `arm64` | `{project_name}-windows-aarch64.exe` |
| `darwin` / `amd64` | `{project_name}-macos-x86_64` |
| `darwin` / `arm64` | `{project_name}-macos-aarch64` |
| `freebsd` / `amd64` | `{project_name}-freebsd-x86_64` |

**Other rules:**
- Local (in-tree) primary binary name: `{project_name}` (no platform/arch suffix during local development inside the Docker image)
- If optional helper binaries exist, use `{project_name}-{tool}` for the in-tree name and `{project_name}-{tool}-{platform}-{arch}{.ext}` for distribution
- Checksum files mirror the artifact name plus `.sha256` (e.g., `{project_name}-linux-x86_64.sha256`)

**Single-binary rule:** the default user experience is "download one file, run it." The primary binary MUST be self-sufficient (PART 0 → "Self-Contained Assets"). Helper binaries are not a substitute for putting features into the primary binary; if a feature can live behind a CLI subcommand of the primary binary, it MUST.

## GUI/TUI/CLI Capability Rule

- A project may implement one, two, or all three surfaces
- If GUI exists, it is the preferred interactive experience on capable local desktops
- If TUI exists, it is the preferred interactive fallback for capable terminals
- CLI is the universal fallback and the default for automation/non-interactive use
- IDEA.md MUST declare which surfaces are actually in scope

---

# PART 3: RUNTIME MODE SELECTION

## Selection Priority

**Automatic runtime selection priority is:**
1. **GUI**
2. **TUI**
3. **CLI**

**Override priority is:**
1. explicit CLI flag / subcommand (`--ui gui|tui|cli`, `gui`, `tui`, `cli`, etc.)
2. config file value
3. environment override
4. automatic detection using the priority above

## Smart Detect Rules

### GUI

Choose GUI when **all** are true:
- GUI support is compiled and enabled for the project
- the process is **not** running in SSH/MOSH/remote-shell style contexts
- a desktop/session display stack is available
- the invocation is interactive enough for windowed UX

**Treat these as remote-shell / non-local GUI blockers:**
- `SSH_CONNECTION`, `SSH_CLIENT`, `SSH_TTY`
- `MOSH_IP`, `MOSH_KEY`
- explicit headless environment from config or flag

**Treat these as positive GUI/display signals (platform-appropriate):**
- `WAYLAND_DISPLAY` — Wayland session (supported by the app on Linux/BSD via the GUI toolkit)
- `DISPLAY` — X11 / XWayland session (supported by the app on Linux/BSD via the GUI toolkit)
- a local Windows desktop session
- a local macOS Aqua/session launch

**Backend selection on Linux/BSD when both signals are present:**
- Prefer Wayland when `WAYLAND_DISPLAY` is set; if connecting to that Wayland socket fails at runtime, fall back to X11 instead of erroring out
- Fall back to X11 (`DISPLAY`) when `WAYLAND_DISPLAY` is unset or the user/config explicitly requests X11
- Both backends MUST be exercised in tests

### TUI

Choose TUI when GUI is not selected and **all** are true:
- TUI support is implemented
- stdin and stdout are TTYs
- `TERM` is not `dumb`
- output is interactive (not piped for machine consumption)
- the environment is not clearly no-format/no-interaction

**Avoid TUI when any of these are true:**
- `TERM=dumb`
- stdout or stdin is not a TTY
- `CI=true` or similar automation context
- `NO_COLOR=1` or another plain-output requirement **when the TUI depends on color/alternate-screen/cursor control**
- explicit `--plain`, `--json`, `--quiet`, or script-oriented modes

### CLI

Choose CLI when:
- GUI and TUI are unavailable or inappropriate
- the app is running non-interactively
- output is piped/redirected
- machine-readable output is requested
- automation or plain-text mode is requested

CLI is the **required** fallback and the default for non-interactive work.

## Reference Detection Logic

```go
type UIMode int

const (
    UIModeCLI UIMode = iota
    UIModeTUI
    UIModeGUI
)

func DetectUIMode(env Env, caps Capabilities) UIMode {
    if forced, ok := env.ForcedUIMode(); ok {
        return forced
    }

    remoteShell := env.HasAny("SSH_CONNECTION", "SSH_CLIENT", "SSH_TTY", "MOSH_IP", "MOSH_KEY")
    displayAvail := env.Has("WAYLAND_DISPLAY") || env.Has("DISPLAY") ||
        caps.LocalWindowsSession || caps.LocalMacOSSession

    if caps.GUISupported && !remoteShell && displayAvail && caps.InteractiveLaunch {
        return UIModeGUI
    }

    plainOrNoninteractive := !caps.StdinTTY || !caps.StdoutTTY ||
        env.Is("TERM", "dumb") || env.Truthy("CI") ||
        (env.Truthy("NO_COLOR") && caps.TUIRequiresFormatting) ||
        caps.MachineOutputRequested

    if caps.TUISupported && !plainOrNoninteractive {
        return UIModeTUI
    }

    return UIModeCLI
}
```

## Behavior Rules by Mode

| Mode | Primary UX | Good For | Avoid |
|------|------------|----------|-------|
| GUI | native windows/dialogs/desktop integration | local desktop use | headless/SSH automation |
| TUI | structured terminal app | interactive terminal workflows | `TERM=dumb`, plain-output contexts |
| CLI | plain commands/text/json | automation, scripts, CI, pipes | none |

---

# PART 4: PRIVILEGE ESCALATION & SYSTEM INTEGRATION

## Core Rule

Privilege escalation support is allowed, but it is **optional** and must be used **only when the user explicitly requests it**.

## NEVER Do

- Never auto-run `sudo`, `pkexec`, UAC elevation, or similar escalation just because a path is protected
- Never silently switch from user-scope install to system-scope install
- Never require elevation for normal app launch, per-user config, or per-user updates
- Never block core app usage behind root/admin requirements unless IDEA.md explicitly defines a genuinely privileged tool

## Allowed Uses (Explicit Request Only)

- system-wide install into protected directories
- system-wide uninstall or cleanup
- privileged file associations / shell integration / desktop registration when explicitly requested
- service registration only if the project truly implements a background service and IDEA.md says so

## Default Scope Rule

Default to **user scope**:
- user config
- user data
- user cache
- per-user desktop integration
- user-local updates/installations

## Path Rule

Prefer platform-standard user directories:

**On-disk paths use the frozen pair `{internal_org}` and `{internal_name}` only — never the mutable `{project_org}` / `{project_name}`. This is the rule that protects user data across project/org renames.**

| Purpose | Linux / BSD | macOS | Windows |
|---------|-------------|-------|---------|
| Config | `~/.config/{internal_org}/{internal_name}/` | `~/Library/Application Support/{internal_name}/config/` | `%AppData%\\{internal_org}\\{internal_name}\\config\\` |
| Data | `~/.local/share/{internal_org}/{internal_name}/` | `~/Library/Application Support/{internal_name}/data/` | `%LocalAppData%\\{internal_org}\\{internal_name}\\data\\` |
| Cache | `~/.cache/{internal_org}/{internal_name}/` | `~/Library/Caches/{internal_name}/` | `%LocalAppData%\\{internal_org}\\{internal_name}\\cache\\` |
| Logs | `~/.local/state/{internal_org}/{internal_name}/logs/` | `~/Library/Logs/{internal_name}/` | `%LocalAppData%\\{internal_org}\\{internal_name}\\logs\\` |

**Rule:** Both `{internal_name}` and `{internal_org}` anchor on-disk identifiers and stable OS-registered names (Bundle IDs, package IDs, dbus names, keychain entries, updater channels). A rename of `{project_name}` or `{project_org}` MUST NOT silently move user data or change those identifiers.

---

# PART 5: GO TOOLCHAIN, BUILD, AND PACKAGING

## Toolchain Rules

| Rule | Description |
|------|-------------|
| Go version | Use the current stable Go version declared in `go.mod` and pinned in `.go-version` |
| Version pin | Single-line `.go-version` file at project root (e.g., `go1.23.0`) — used by toolchain managers and CI |
| Targets | Build targets set via `GOOS`/`GOARCH` env vars inside the Docker image; no toolchain installation beyond the base `golang:alpine` image is required |
| Formatting | `gofmt` (or `goimports`) is required |
| Linting | `golangci-lint` is required |
| Testing | `go test` is required |
| CGO | `CGO_ENABLED=0` ALWAYS — pure Go, no C (PART 0 → "Go-Only Application") |
| Source language | Go only (PART 0 → "Go-Only Application") |

## Pure-Go Library Stack

This is the recommended starting point for satisfying common application needs with packages that do **not** drag in C dependencies. Pure-Go packages are what make a single static binary that works on Windows, macOS, Linux, and BSD achievable; every C-linked package becomes a portability tax across that target matrix.

**Rule:** when a capability below is needed, prefer the pure-Go option. Only deviate when (a) the pure-Go option is not viable for the project's actual requirements, AND (b) the deviation is documented in `IDEA.md` and `LICENSE.md`. All deviations from pure Go require both an IDEA.md exception and LICENSE.md attribution.

| Capability | Go preferred |
|------------|-------------|
| TLS | `crypto/tls` (standard library) |
| HTTP client | `net/http` (standard library) |
| Crypto | `crypto/*` (standard library); `golang.org/x/crypto` for extras |
| Compression | `compress/*` (standard library) |
| Serialization | `encoding/json`, `encoding/xml` (stdlib); `gopkg.in/yaml.v3`, `github.com/BurntSushi/toml` |
| Async / concurrency | goroutines + channels (language built-in) |
| CLI parsing | `github.com/spf13/cobra` + `github.com/spf13/pflag`; or stdlib `flag` |
| Logging | `log/slog` (stdlib, Go 1.21+); `go.uber.org/zap` |
| Error handling | `fmt.Errorf` + `%w` wrapping; `errors.Is` / `errors.As` (stdlib) |
| Date / time | `time` (standard library) |
| Config files | `github.com/spf13/viper`; or plain TOML/YAML |
| GUI (cross-platform) | `gioui.org` (Gio — pure Go, X11+Wayland+macOS+Windows); `fyne.io/fyne/v2` |
| TUI | `github.com/charmbracelet/bubbletea` + `github.com/charmbracelet/lipgloss` |
| Font rendering | handled by GUI toolkit |
| Image decoding | `image/*` (standard library) |
| Clipboard | `golang.design/x/clipboard` |
| Desktop notifications | `github.com/gen2brain/beeep` |
| File dialogs | `github.com/sqweek/dialog` |
| Theme detection | `github.com/adrg/xdg` + OS theme APIs |
| Secrets storage | `github.com/zalando/go-keyring` |
| Local SQL | `modernc.org/sqlite` (pure Go SQLite — `CGO_ENABLED=0` compatible, no bundled C needed) |
| UUID / hashing | `github.com/google/uuid`; `crypto/sha256` (stdlib) |
| Filesystem watching | `github.com/fsnotify/fsnotify` |
| Process / IPC | `net` (Unix socket / named pipe) from stdlib |
| Networking primitives | `net`, `net/http` from stdlib |

This list is not exhaustive; treat it as the starting point. When introducing a new dependency:

1. Search pkg.go.dev for a pure-Go option first
2. Check for any transitive C dependency (`go mod graph`, review each package's source)
3. If the only viable option has a C dependency, document the exception in `IDEA.md`, confirm it can be statically compiled into the final binary with `CGO_ENABLED=0`, and add attribution to `LICENSE.md`

## Go Commands

**All Go invocations execute inside the project Docker container — never on the host.** The table below shows the *logical* command; the **actual** invocation is wrapped (e.g., `docker compose run --rm dev <cmd>` or `docker run --rm -v "$PWD":/work -w /work <image> <cmd>`). See "Docker Rule" below.

| Logical Command | Purpose |
|-----------------|---------|
| `gofmt -l .` | Format verification (list files needing formatting) |
| `gofmt -w .` | Format code in-place |
| `golangci-lint run ./...` | Lint enforcement |
| `go test ./...` | Test suite |
| `go build ./...` | Debug build (all packages) |
| `go build -ldflags="-s -w" -o {project_name} .` | Optimized build |
| `go build -ldflags="-s -w -X main.Version=…" …` | Release build with version injection |
| `go vet ./...` | Vet checks |
| `govulncheck ./...` | Vulnerability scanning |
| `go run . -- [args]` | Execution (in container; with X11/Wayland forwarding if GUI) |

**Examples of the actual wrapped form:**

```bash
# Format check
docker compose run --rm dev gofmt -l .

# Lint
docker compose run --rm dev golangci-lint run ./...

# Tests
docker compose run --rm dev go test ./...

# Release build (artifacts land in a mounted dist/ inside the container)
docker compose run --rm \
  -e CGO_ENABLED=0 -e GOOS=linux -e GOARCH=amd64 \
  dev go build -ldflags="-s -w -X main.Version=$(cat release.txt)" \
  -o dist/{project_name}-linux-x86_64 .

# Run with display forwarding via the `gui` compose service, which mounts
# the X11 / Wayland sockets and exports DISPLAY / WAYLAND_DISPLAY (see
# "Docker Rule" → "X11 and Wayland Forwarding" for socket/env wiring).
docker compose run --rm gui go run . -- --ui gui
```

Bare `go …` invocations on the host are forbidden by PART 0 → "No Host Toolchain or Binary Execution."

## Build Rules

- **Pure Go by default** — `CGO_ENABLED=0` always; every dependency is pure Go unless a specific exception is documented in `IDEA.md` (PART 0 → "Go-Only Application")
- Release builds use `go build` with `-ldflags="-s -w"` to strip symbols, reducing binary size
- Use `-ldflags="-X main.Version=…"` (and similar `-X` flags) to inject version, commit ID, and build date at compile time
- The final artifact MUST be a single statically linked binary per target (PART 0 → "Single Static Binary")
- Set `GOOS`, `GOARCH`, and `CGO_ENABLED=0` as environment variables when cross-compiling inside the Docker image
- All runtime assets are embedded at compile time via `//go:embed`; the repo's `assets/` directory is build-time input only (PART 0 → "Self-Contained Assets")
- A static-binary self-check runs as part of CI (`ldd` on Linux artifacts must show "not a dynamic executable" or only kernel-vDSO; `otool -L` on macOS must show only Apple-provided frameworks; `dumpbin /dependents` on Windows must show no unexpected DLLs)

## Build Metadata

Inject version, commit ID, build date, and official site via `-ldflags`:

```go
// Version injected at build time via -ldflags
var (
    Version      = "dev"     // overridden by: -ldflags="-X main.Version=$(cat release.txt)"
    CommitID     = "unknown" // overridden by: -ldflags="-X main.CommitID=$(git rev-parse --short HEAD)"
    BuildDate    = "unknown" // overridden by: -ldflags="-X main.BuildDate=$(date -u +%Y-%m-%dT%H:%M:%SZ)"
    OfficialSite = ""        // overridden by: -ldflags="-X main.OfficialSite=$(cat site.txt)"
)
```

**Full release build command (logical form — run inside the Docker container, not on the host):**
```bash
go build \
  -ldflags="-s -w \
    -X main.Version=$(cat release.txt) \
    -X main.CommitID=$(git rev-parse --short HEAD) \
    -X main.BuildDate=$(date -u +%Y-%m-%dT%H:%M:%SZ) \
    -X main.OfficialSite=$(cat site.txt 2>/dev/null || true)" \
  -o dist/{project_name}-linux-x86_64 .
```

## Project Layout

```text
go.mod
go.sum
.go-version                 # single-line Go version pin, e.g. "go1.23.0"
main.go                     # entry point — auto-detects GUI/TUI/CLI mode
                            # (or cmd/{project_name}/main.go for multi-binary workspaces)
internal/
├── app/                    # shared domain/application logic
├── config/                 # configuration loading + defaults
├── platform/               # OS/platform integration
├── state/                  # app state and persistence adapters
├── support/                # logging, errors, utilities
└── ui/
    ├── gui/                # optional native GUI
    ├── tui/                # optional terminal UI
    └── cli/                # optional plain CLI commands/output
assets/                     # BUILD-TIME ONLY: source tree embedded via //go:embed
docker/                     # REQUIRED: Dockerfile, compose.yaml, entrypoint.sh, README.md
Makefile                    # build targets: build, release, docker, test, dev, clean
scripts/                    # optional host-side Docker wrapper scripts; MUST NOT contain application logic
tests/                      # integration tests
```

The `assets/` directory in the repo holds source files (fonts, icons, default themes, schemas, locales, default config). They are embedded into the binary at compile time via `//go:embed`. The directory is **never** shipped or installed alongside the binary on a user's machine.

## Naming Rules

| Item | Rule | Example |
|------|------|---------|
| Files | `snake_case` or flat lowercase | `config_loader.go`, `platform.go` |
| Package names | lowercase, no underscores | `package config`, `package ui` |
| Types/interfaces | `PascalCase` | `AppConfig`, `UIMode` |
| Functions | `camelCase` (unexported) / `PascalCase` (exported) | `detectUIMode`, `DetectUIMode` |
| Constants/vars | `SCREAMING_SNAKE_CASE` (package-level consts) | `DefaultTheme` (exported), `defaultTimeout` (unexported) |
| Go module path (`go.mod`) | lowercase, URL-style | `github.com/myorg/myapp` |
| Go package name | lowercase, no underscores, no hyphens | `package myapp` |

## Release Artifacts

- Primary binary follows the naming schema `{project_name}-{platform}-{arch}{.ext}` defined in PART 2 → "Binary Model" — single statically linked file, no OS suffix artifacts
- Each release MUST publish artifacts for at minimum: `{project_name}-linux-x86_64`, `{project_name}-linux-aarch64`, `{project_name}-windows-x86_64.exe`, `{project_name}-windows-aarch64.exe`, `{project_name}-macos-x86_64`, `{project_name}-macos-aarch64` (subset acceptable only when IDEA.md narrows platform scope)
- A static-linkage verification step is part of release: `ldd` / `otool -L` / `dumpbin /dependents` output is captured and checked against an allowlist (kernel vDSO, Apple system frameworks, Windows kernel32/user32 etc.) — anything outside the allowlist fails the release
- No companion files (no `.so`, `.dylib`, `.dll`, no asset bundles, no font directories) ship next to the binary
- Include SHA-256 checksums for every published artifact, named `{artifact}.sha256`
- Include release notes that describe actual changes
- Include an SBOM (always — generated via `cyclonedx-gomod`; see PART 10 → "Suggested CI Steps" for the invocation). Include provenance/attestation when the release platform supports it
- If GUI packaging exists (MSI, DMG, AppImage, deb, rpm, etc.), the package wraps the same single static binary plus desktop integration metadata; package metadata lives in `packaging/`

## Docker Rule

Docker is **REQUIRED**. The project MUST ship a working Docker setup, and every toolchain command and every produced binary runs inside it (PART 0 → "No Host Toolchain or Binary Execution").

### Required Docker Assets

Place Docker assets under `docker/`:

```text
docker/
├── Dockerfile              # builds the dev/build/test image using golang:alpine (rolling)
├── Dockerfile.runtime      # optional: minimal runtime image for produced binaries
├── compose.yaml            # services: dev (build/test/run), gui (X11/Wayland forwarding), runtime
├── entrypoint.sh           # sets non-root UID/GID, prepares cache dirs
└── README.md               # how to build the image, run tests, run GUI with display forwarding
```

A top-level `compose.yaml` symlink or shim is allowed for ergonomics, but the source of truth lives under `docker/`.

### Mandatory Image Properties

- Base image: `golang:alpine` (rolling — never pinned to a specific version tag)
- Go version matching `.go-version` is provided by the base image or set via `GOTOOLCHAIN` in the Dockerfile
- `gofmt` and `go vet` are included in the Go toolchain (no extra install needed)
- `golangci-lint`, `govulncheck`, `go-licenses`, and `cyclonedx-gomod` are pre-installed in the image
- A non-root user matching the host UID/GID by default (so files written into mounted volumes are not root-owned)
- Go module cache and build cache mounted as named volumes for build speed
- `CGO_ENABLED=0` set as a default environment variable in the image
- The image requires no C dev libraries — `CGO_ENABLED=0` Go builds need no C toolchain at all

### X11 and Wayland Forwarding (Mandatory for GUI/Display Testing)

GUI and display-aware test runs use the `gui` compose service (or equivalent `docker run` flags). Both X11 and Wayland forwarding MUST be supported; the spec does not pick one — the container detects what the host provides and forwards accordingly.

**X11 forwarding (host running Xorg or XWayland):**

```bash
xhost +SI:localuser:$(id -un)        # grant access to current user only; revoke when done

docker run --rm \
  -e DISPLAY="$DISPLAY" \
  -e XAUTHORITY=/tmp/.docker.xauth \
  -v /tmp/.X11-unix:/tmp/.X11-unix:rw \
  -v "$XAUTHORITY":/tmp/.docker.xauth:ro \
  --device /dev/dri \
  -v "$PWD":/work -w /work \
  "$PROJECT_IMAGE" go run . -- --ui gui

xhost -SI:localuser:$(id -un)        # revoke after the session
```

**Wayland forwarding (host running a Wayland compositor):**

```bash
docker run --rm \
  -e WAYLAND_DISPLAY="$WAYLAND_DISPLAY" \
  -e XDG_RUNTIME_DIR=/tmp/xdg \
  -v "$XDG_RUNTIME_DIR/$WAYLAND_DISPLAY":"/tmp/xdg/$WAYLAND_DISPLAY" \
  --device /dev/dri \
  -v "$PWD":/work -w /work \
  "$PROJECT_IMAGE" go run . -- --ui gui
```

**Rules:**
- Both forwarding modes MUST be documented and working — pick one based on what the host session exposes
- `xhost +` (wide open) is forbidden; use scoped grants (`+SI:localuser:…`) and revoke after
- Do not run the container as root just to make sockets work; map UID/GID instead
- GPU access (`/dev/dri`) is allowed when the GUI stack benefits from it; keep it opt-in via a compose profile when not always needed
- Sound/MIDI/input device forwarding is allowed only when IDEA.md justifies it
- Audio (PulseAudio/PipeWire) forwarding follows the same scoped-socket pattern when needed

### Forbidden

- Running `go`, `go build`, `go test`, or any built binary on the host
- Treating Docker as a "CI-only" or "release-only" concern
- Recommending `go install` on the host to "just try it quickly"
- `--privileged`, `--net=host` outside of clearly justified, documented profiles
- `xhost +` without a scoped target

---

# PART 6: VERSION, SITE, AND BUILD METADATA

## `release.txt` (Canonical Version When Present)

- `release.txt` is a single-line canonical version source when present
- It overrides derived/tag-based defaults and should stay in sync with `go.mod` module version if applicable
- Use semantic versions unless the project intentionally uses another documented scheme
- If `release.txt` exists, CI/CD and release scripts must use it rather than inventing a version

## `site.txt` (Optional Official Site)

- Optional single-line URL file in project root
- Use it for the official site, homepage, docs root, support page, or update landing page
- Never guess it
- If `site.txt` exists, it wins over `IDEA.md`, README, environment variables, and CI secrets
- If `site.txt` and `IDEA.md` disagree, `site.txt` wins; update IDEA.md to match or remove the stale variable

## Metadata Priority Rules

### Version
1. `release.txt` (when present)
2. `go.mod` module version / git tag
3. explicit documented fallback for bootstrap/dev builds

### Official Site
1. `site.txt` (when present)
2. `IDEA.md ## Project variables` (`official_site`)
3. explicit environment/CI override used only where documented
4. empty

## Build Metadata

Inject at build time via `-ldflags` `-X` flags:
- version
- commit ID
- build date
- official site (optional)

**Example variable declarations:**
```go
// Version injected at build time via -ldflags
var (
    Version      = "dev"
    CommitID     = "unknown"
    BuildDate    = "unknown"
    OfficialSite = ""
)
```

**Makefile pattern for version injection:**
```makefile
VERSION     := $(shell cat release.txt 2>/dev/null || echo "dev")
COMMIT_ID   := $(shell git rev-parse --short HEAD 2>/dev/null || echo "unknown")
BUILD_DATE  := $(shell date -u +%Y-%m-%dT%H:%M:%SZ)
SITE        := $(shell cat site.txt 2>/dev/null || true)

LDFLAGS := -s -w \
  -X main.Version=$(VERSION) \
  -X main.CommitID=$(COMMIT_ID) \
  -X main.BuildDate=$(BUILD_DATE) \
  -X main.OfficialSite=$(SITE)

build:
	go build -ldflags="$(LDFLAGS)" -o dist/{project_name} .
```

---

# PART 7: RUNTIME DETECTION, CONFIGURATION, AND OUTPUT

## Runtime Detection Rules

All machine-dependent settings MUST be detected at runtime on the target machine, never copied from the developer machine.

| Setting | Detection Method | NEVER Do |
|---------|------------------|----------|
| OS / architecture | `runtime.GOOS`, `runtime.GOARCH` | hardcode dev machine values |
| Home/config/data dirs | `os.UserHomeDir()`, `os.UserConfigDir()`, `os.UserCacheDir()` | hardcode absolute dev paths |
| Locale | `LANG`, platform locale APIs | assume English-only unless documented |
| Theme preference | OS theme APIs / config | assume dark/light universally |
| Terminal capability | TTY + TERM + negotiated features | assume ANSI/alt-screen support |
| Display stack | X11 (`DISPLAY`) and Wayland (`WAYLAND_DISPLAY`) detection — both backends supported, runtime-selected by GUI toolkit | assume GUI exists; ship X11-only or Wayland-only |
| CPU / memory | `runtime.NumCPU()`, runtime detection if needed | tune only for dev hardware |

## Configuration Rules

- Config files store user choices, not auto-detected machine facts
- Defaults must be sensible for a fresh per-user install
- Support config file + environment variable + CLI override layering
- CLI override wins over env; env wins over config; config wins over defaults
- Secrets must not be stored in world-readable files

## Output Rules

- Respect `NO_COLOR`
- Respect `TERM=dumb`
- Respect non-TTY stdout/stderr
- Provide machine-readable output when the app advertises it (`--json`, `--plain`, etc.)
- Help/version output must show the **actual invoked binary name**

## NO_COLOR Support

When `NO_COLOR` is set and non-empty, disable ANSI color output. If the TUI depends on richer formatting, fall back to CLI/plain output instead of forcing a degraded pseudo-TUI.

| Condition | GUI | TUI | CLI |
|-----------|-----|-----|-----|
| local desktop with display | allowed | fallback | fallback |
| SSH with TTY | no auto-GUI | allowed if terminal capable | fallback |
| `TERM=dumb` | no auto-GUI | avoid | use CLI |
| `NO_COLOR=1` | GUI unaffected unless project says otherwise | avoid color-dependent TUI | prefer CLI/plain |
| stdout piped | avoid GUI | avoid TUI | use CLI |

---

# PART 8: TESTING, QUALITY, AND DEBUGGING

## Required Quality Gates

All gates execute inside the project Docker container (PART 5 → "Docker Rule"). The "Logical Command" column shows the Go invocation; the actual command is Docker-wrapped.

| Gate | Logical Command | Wrapped Form (example) |
|------|-----------------|------------------------|
| Formatting | `gofmt -l .` | `docker compose run --rm dev gofmt -l .` |
| Linting | `golangci-lint run ./...` | `docker compose run --rm dev golangci-lint run ./...` |
| Tests | `go test ./...` | `docker compose run --rm dev go test ./...` |
| Vet | `go vet ./...` | `docker compose run --rm dev go vet ./...` |
| Vulnerability scan | `govulncheck ./...` | `docker compose run --rm dev govulncheck ./...` |
| License enumeration | `go-licenses report ./...` | see PART 10 → "Suggested CI Steps" |
| Attribution drift | `go-licenses report ./...` (output diffed against the GENERATED region of `LICENSE.md`) | see PART 10 → "Suggested CI Steps" |
| GUI smoke (X11) | `go run . -- --ui gui` against an X11 socket | see PART 5 → "X11 and Wayland Forwarding" |
| GUI smoke (Wayland) | `go run . -- --ui gui` against a Wayland socket | see PART 5 → "X11 and Wayland Forwarding" |

## Testing Rules

- Core business logic must have unit tests
- Integration tests go in `tests/`
- UI-specific code should keep as much logic as possible outside the UI layer so it is unit-testable
- GUI/TUI smoke tests are required when those surfaces exist
- Snapshot/golden tests are allowed for stable text output and rendering state, but must be intentionally reviewed
- Fixes must include a test when the behavior is testable

## Debug Rules

- Release builds default to user-friendly error messages
- Debug/development mode may include verbose logs and stack traces
- Do not expose sensitive data in logs
- Panic behavior must be intentional and documented (`recover` used deliberately, not to swallow all errors)
- GUI/TUI debug tooling must not leak into normal production UX by default

## Performance Rules

- Optimize based on measured behavior, not guesses
- Scale caches/worker counts from `runtime.NumCPU()` or documented defaults when relevant
- Keep minimal-system defaults sensible
- Avoid unnecessary allocations and duplicate state copies in hot paths
- Never spawn unbounded goroutines — always cap with a semaphore, worker pool, or context cancellation

---

# PART 9: SECURITY & PRIVACY

## Security-First Design

- Least privilege by default
- Per-user storage by default
- Explicit consent for networked, destructive, or privileged operations
- Validate all untrusted input
- Prefer pure-Go packages and audited dependencies
- Use `crypto/tls` (standard library) for TLS on all platforms. On macOS and Windows, the platform-native TLS APIs are also accessible through Go's standard library — these are acceptable. On Linux/BSD, `crypto/tls` covers all use cases without any C dependency
- For in-memory secret handling, avoid holding secrets in plain `string` or `[]byte` longer than necessary; zero memory after use where practical (`defer runtime.KeepAlive` patterns, explicit zeroing before `return`)
- For OS-level secret storage, use `github.com/zalando/go-keyring` (pure Go; speaks the platform protocol on Linux/macOS/Windows). Do not link to `libsecret` / `libgnome-keyring` / `kwallet` directly — those would require CGO and violate the `CGO_ENABLED=0` rule

## Secrets and Sensitive Data

- Never log raw secrets
- Redact tokens, passwords, API keys, auth headers, and personal data from logs and diagnostics
- Config files containing secrets must use restrictive permissions
- Export/import features must clearly label sensitive content

## Update / Download Safety

If the app downloads updates or remote content:

- require explicit user intent or documented auto-update policy
- verify integrity/signatures where supported
- never run downloaded code implicitly
- document trust assumptions in IDEA.md

Plugin downloads are an additional case and apply only when IDEA.md defines a hardened plugin contract per PART 0 → "Single Static Binary"; the four rules above apply to them as well.

## Dependency Governance

- Keep dependencies minimal
- Remove unused packages promptly
- Public repos should automate dependency updates for Go modules, GitHub Actions, and Docker if used
- Security advisories are blockers until triaged
- Run `govulncheck ./...` as part of CI to catch known vulnerabilities

## Telemetry Rule

No hidden telemetry. Any analytics, crash reporting, or update pings must be documented and controllable.

---

# PART 10: CI/CD, RELEASES, AND AUTOMATION

## CI/CD Rules

| Rule | Description |
|------|-------------|
| Explicit commands | CI must run explicit `go` / `make` commands, not hide core release logic behind undocumented wrappers |
| Least privilege | CI tokens/permissions default to read-only |
| Pinned actions | Third-party actions pinned to full commit SHAs |
| No unsafe fork secrets | Fork PRs do not receive secrets or publish permissions |
| Version precedence | `release.txt` wins when present |
| Site precedence | `site.txt` wins when present |
| Verifiable outputs | Releases publish checksums and an SBOM (always); provenance/attestation when the platform supports it |

## Minimum Public Repo Workflows

- build/test workflow
- release workflow
- security/dependency workflow

Equivalent Gitea/Forgejo/GitLab/Jenkins pipelines must enforce the same gates, not a weaker subset.

## Suggested CI Steps

CI runs every Go step inside the project's Docker image. CI MUST NOT install a Go toolchain on the runner and call `go` directly — it builds the project image (or pulls a cached one) and executes commands inside it.

```bash
# Prepare output directory for release artifacts (binaries, checksums, SBOM)
mkdir -p dist

# Build (or pull) the dev/build/test image
docker build -t "$PROJECT_IMAGE" -f docker/Dockerfile .

# Run quality gates inside the image
docker run --rm -v "$PWD":/work -w /work "$PROJECT_IMAGE" gofmt -l .
docker run --rm -v "$PWD":/work -w /work "$PROJECT_IMAGE" golangci-lint run ./...
docker run --rm -v "$PWD":/work -w /work "$PROJECT_IMAGE" go test ./...
docker run --rm -v "$PWD":/work -w /work "$PROJECT_IMAGE" go vet ./...
docker run --rm -v "$PWD":/work -w /work "$PROJECT_IMAGE" govulncheck ./...

# License enumeration and attribution-drift check:
# Regenerate the GENERATED region and compare to committed.
docker run --rm -v "$PWD":/work -w /work "$PROJECT_IMAGE" \
  go-licenses report ./... > LICENSE.generated.md
sed -n '/<!-- GENERATED:/,$p' LICENSE.md > LICENSE.committed-generated.md
diff LICENSE.committed-generated.md LICENSE.generated.md

# Build statically linked release binaries (one per supported target).
for TARGET in "linux/amd64" "linux/arm64" "windows/amd64" "windows/arm64" "darwin/amd64" "darwin/arm64"; do
  GOOS="${TARGET%/*}"
  GOARCH="${TARGET#*/}"

  # Map GOOS/GOARCH → artifact name
  case "$GOOS" in
    linux)   PLATFORM=linux ;;
    windows) PLATFORM=windows ;;
    darwin)  PLATFORM=macos ;;
  esac
  case "$GOARCH" in
    amd64) ARCH=x86_64 ;;
    arm64) ARCH=aarch64 ;;
  esac
  EXT=""
  [ "$GOOS" = "windows" ] && EXT=".exe"

  ARTIFACT="{project_name}-${PLATFORM}-${ARCH}${EXT}"

  docker run --rm -v "$PWD":/work -w /work \
    -e CGO_ENABLED=0 -e GOOS="$GOOS" -e GOARCH="$GOARCH" \
    "$PROJECT_IMAGE" \
    go build \
      -ldflags="-s -w -X main.Version=$(cat release.txt) -X main.CommitID=$(git rev-parse --short HEAD) -X main.BuildDate=$(date -u +%Y-%m-%dT%H:%M:%SZ)" \
      -o "dist/$ARTIFACT" .

  sha256sum "dist/$ARTIFACT" > "dist/$ARTIFACT.sha256"

  # Verify static linkage on Linux
  if [ "$GOOS" = "linux" ]; then
    docker run --rm -v "$PWD":/work -w /work "$PROJECT_IMAGE" \
      sh -c "ldd dist/$ARTIFACT 2>&1 | grep -qE 'not a dynamic executable|statically linked'"
  fi
done

# Generate the SBOM (CycloneDX JSON) — published alongside the release artifacts.
docker run --rm -v "$PWD":/work -w /work "$PROJECT_IMAGE" \
  cyclonedx-gomod app -json -output "dist/{project_name}-bom.json"
```

For GUI smoke tests in CI, use a virtual X server (e.g., `Xvfb`) and a headless Wayland compositor (e.g., `cage`, `weston --backend=headless`) **inside** the container or as a sidecar service — both backends MUST be exercised, not just one.

## Release Integrity

Tagged/manual releases should publish:
- binaries/packages
- SHA-256 checksum file
- release notes
- SBOM (`CycloneDX` from `cyclonedx-gomod`; `SPDX JSON` is acceptable if a project chooses that format instead) — always
- provenance / attestation — when the release platform supports it

If signing or attestation is required but keys/permissions are unavailable, stop and ask instead of faking compliance.

---

# PART 11: DOCUMENTATION & LICENSE

## README Minimum Sections

- About
- Features
- Installation
- Usage
- GUI/TUI/CLI mode behavior (GUI section MUST cover both X11 and Wayland)
- Configuration
- Development (MUST document the Docker-based workflow — no host `go` invocations)
- Testing (MUST show Docker-wrapped commands; GUI tests show X11 and Wayland forwarding)
- License

## LICENSE.md Requirements

`LICENSE.md` is the canonical attribution document. It MUST contain:

1. **Project license text** — verbatim copy of the MIT license (or whatever IDEA.md declares), with the project's copyright line
2. **Third-party dependency attributions** — for every direct and transitive module in `go.sum` whose license requires attribution (MIT, BSD-2/3-Clause, Apache-2.0, ISC, MPL-2.0, Zlib, BSL-1.0, OpenSSL, Unicode-DFS-2016, etc. — i.e., everything except true public domain). Public-domain / CC0-1.0 packages SHOULD still be listed for transparency
3. **Verbatim NOTICE / COPYING / LICENSE preservation** — preserve upstream notice files verbatim whenever the upstream license requires it. Common triggers: (a) packages that vendor C code — preserve the upstream C project's NOTICE / COPYING / LICENSE alongside the package's own license; (b) Apache-2.0 dependencies that ship a NOTICE file — preserve verbatim
4. **Embedded asset notices** — fonts, icons, theme data, sample data, locale catalogs etc. that are embedded in the binary; each shipped asset's license/attribution as required by its source
5. **Generation banner** — a header comment recording which tool generated the third-party section and from which `go.sum` revision, so reviewers can tell at a glance whether the file is current

**Rule:** `LICENSE.md` is generated, not hand-written, for the third-party section (see "License Compliance" below). The project license text and upstream NOTICE preservation pieces are hand-written and MUST be merged with the generated section, not overwritten by it.

## License Compliance

License compliance is enforced by tooling and gated in CI; it is not left to manual review.

### Required Tooling

| Tool | Purpose |
|------|---------|
| `govulncheck` | Security advisory scanning for known Go vulnerabilities |
| `go-licenses` | License enumeration and generation of the third-party attribution section of `LICENSE.md` from `go.sum` |
| `cyclonedx-gomod` | Generate the SBOM (CycloneDX format) referenced in PART 5 → "Release Artifacts" and PART 10 → "Release Integrity" |
| `golangci-lint` | Lint enforcement; configured to ban import patterns violating the pure-Go rule |

All tools run inside the project Docker image. The image MUST have them pre-installed.

### License Allowlist (default)

| Allowed | Notes |
|---------|-------|
| `MIT` | Project default; baseline |
| `BSD-2-Clause`, `BSD-3-Clause` | Permissive; require attribution |
| `Apache-2.0`, `Apache-2.0 WITH LLVM-exception` | Require NOTICE preservation when one is shipped |
| `ISC` | Permissive; require attribution |
| `MPL-2.0` | File-level copyleft only — compatible with static linking into MIT binaries |
| `Zlib`, `BSL-1.0`, `Unicode-DFS-2016`, `Unicode-3.0` | Permissive |
| `CC0-1.0`, `Unlicense` | Public-domain equivalents |

Compound SPDX expressions like `MIT OR Apache-2.0` are accepted automatically when any alternative in the expression is on the allowlist. No special handling required.

### License Denylist (default — deny without IDEA.md exception)

| Denied | Reason |
|--------|--------|
| `GPL-2.0-*`, `GPL-3.0-*` | Strong copyleft; static linking forces the entire distributed binary to GPL, silently relicensing it away from MIT |
| `AGPL-1.0-*`, `AGPL-3.0-*` | Strong copyleft + network-use clause; even non-distributed deployments incur source-release obligations |
| `LGPL-2.0-*`, `LGPL-2.1-*`, `LGPL-3.0-*` | LGPL §6 under static linking requires shipping object files / a written relink offer alongside the binary, breaking the single-binary deliverable |
| `SSPL-1.0` | Service-source copyleft; treated like AGPL for our purposes |
| `Commons-Clause`, `BUSL-*`, `Elastic-2.0`, `Confluent-Community-*` | Source-available with commercial-use restrictions; not OSI-approved; incompatible with the "Free & open source" rule (PART 0 → "Licensing & Features") |

### IDEA.md Exception Path

A project MAY use a denylisted module only if **all** of:

1. `IDEA.md` adds a `## License exceptions` subsection naming the module, the upstream license, and the rationale
2. The exception explicitly accepts the consequence — e.g., "this binary is distributed under GPL-3.0; the project's MIT claim applies only to the source we author, not the published binary" — and `IDEA.md ## Project variables` adds (or updates) the variable `distribution_license` accordingly. `distribution_license` is an exception-only variable: it is not part of the required-keys list and is present only in projects that have taken a license exception
3. README, `LICENSE.md`, and any user-visible "About" surface are updated to reflect the actual distribution license, not just the source license
4. Any tooling configuration (e.g., `golangci-lint` import ban config) is updated with a scoped allow entry for the specific module + version, not a blanket category unblock

There is no other path to use a denylisted module. Adding it without going through these steps is a spec violation.

### Required `go.mod` Metadata

Every `go.mod` MUST include a `module` path and a `go` version directive:

```
module github.com/{project_org}/{project_name}

go 1.23
```

And `README.md` / `LICENSE.md` MUST include:
- project license (SPDX expression or full text)
- module path
- maintainer info

### Generation Pattern

`LICENSE.md` has two regions:

```markdown
<!-- HAND-WRITTEN: project license, upstream NOTICE files, embedded asset notices -->

# Project License

(MIT text here)

# Embedded Assets

(font / icon / locale attributions here)

# Upstream NOTICE Files

(verbatim NOTICE / COPYING from any vendored C dependency)

<!-- GENERATED: third-party module attributions -->
<!-- Generated by go-licenses from go.sum @ <commit> on <date> — do not edit by hand -->

# Third-Party Module Attributions

(go-licenses output here)
```

The generated region is regenerated on every dependency change. The hand-written region is preserved across regenerations.

### User-Visible Attribution Surface

Embedded license data MUST be reachable by the user at runtime, not just shipped in the source repo:

- **CLI**: `--licenses` (alias: `--credits`) prints the full embedded `LICENSE.md` to stdout
- **TUI**: a "Licenses" / "Credits" entry in the help / about screen, scrollable
- **GUI**: an "About → Open Source Licenses" entry that opens a scrollable view of the full text

All three surfaces read the same `LICENSE.md` blob embedded at compile time via `//go:embed` (PART 0 → "Self-Contained Assets"). The version, commit ID, and build date (PART 6) are shown alongside.

### CI Gate (mandatory)

The canonical CI invocation lives in PART 10 → "Suggested CI Steps" (the `govulncheck` command and the `go-licenses report` + `sed`/`diff` drift check). This section defines the **policy**; PART 10 owns the script template. Do not duplicate the commands here when editing — keep PART 10 as the single source.

Drift between `go.sum` and the generated section of `LICENSE.md` is a CI failure, not a warning. The module graph is the source of truth for what licenses we ship, and `LICENSE.md` MUST match it.

## Documentation Style

- Keep docs precise and operational
- Prefer command snippets that actually match the Go project layout
- If GUI/TUI/CLI differ in behavior, document the differences explicitly

---

# PART 12: CHECKLISTS

## Bootstrap Checklist

- [ ] `IDEA.md` exists
- [ ] `IDEA.md` has `## Project description`
- [ ] `IDEA.md` has `## Project variables`
- [ ] `IDEA.md` has `## Business logic`
- [ ] `project_name`, `project_org`, `internal_name`, and `internal_org` exist
- [ ] If pre-template `CLAUDE.md` or `.claude/CLAUDE.md` existed, project-specific content was migrated into IDEA.md
- [ ] `CLAUDE.md` / `.claude/CLAUDE.md` are short loaders, not duplicate specs
- [ ] `release.txt` exists if the project is using explicit release versioning
- [ ] `site.txt` exists only if there is a real official site URL
- [ ] `docker/Dockerfile`, `docker/compose.yaml`, and `docker/entrypoint.sh` exist and build a working dev/test image
- [ ] Image is based on `golang:alpine` (rolling); `golangci-lint`, `govulncheck`, `go-licenses`, and `cyclonedx-gomod` are pre-installed
- [ ] `CGO_ENABLED=0` is set as default in the Docker image environment
- [ ] X11 forwarding sample command is documented and works against a real Xorg/XWayland session
- [ ] Wayland forwarding sample command is documented and works against a real Wayland compositor
- [ ] `.go-version` pins the Go version; `go.mod` declares the same version
- [ ] `go.mod` has the correct `module` path and `go` directive
- [ ] Repo has `assets/` (build-time source) and a `//go:embed` directive wiring it into the binary
- [ ] No source files in any language other than Go contribute to the binary (small Docker shell helpers excepted)
- [ ] `Makefile` exists with at minimum `build`, `test`, `dev`, `clean` targets
- [ ] `go-licenses` produces the third-party attribution section; `LICENSE.md` is up to date
- [ ] `govulncheck` passes clean
- [ ] `cyclonedx-gomod` is pre-installed in the Docker image and produces a valid SBOM

## Implementation Checklist

- [ ] Core logic is shared across GUI/TUI/CLI
- [ ] Runtime mode auto-detect follows GUI → TUI → CLI priority
- [ ] GUI is not auto-selected in SSH/MOSH/remote-shell contexts
- [ ] If GUI is in scope, both X11 and Wayland backends are supported and runtime-selected by the GUI toolkit
- [ ] TUI is avoided for `TERM=dumb`, non-TTY, and plain-output contexts
- [ ] CLI works for non-interactive/scripted use
- [ ] Privilege escalation is optional and explicit-request only
- [ ] Config/data/cache/log paths use per-user defaults unless system scope was explicitly requested
- [ ] All assets (fonts, icons, themes, default config, schemas, locales) are embedded at compile time via `//go:embed`
- [ ] Dependencies are pure Go where a viable pure-Go package exists (PART 5 → "Pure-Go Library Stack"); each C-dependent package is justified in IDEA.md
- [ ] `go mod graph` was reviewed — no surprise transitive C or CGO dependencies
- [ ] No GPL / AGPL / LGPL / SSPL / BUSL / source-available dep was added without an IDEA.md `## License exceptions` entry and an updated `distribution_license` (PART 11 → "License Compliance")
- [ ] User-visible licenses surface exists: CLI `--licenses`, TUI "Licenses" entry, GUI "About → Open Source Licenses" — all reading the same embedded `LICENSE.md`
- [ ] App runs end-to-end from the binary alone on an air-gapped machine — no first-run downloads
- [ ] No plugin or runtime extension loading from disk unless IDEA.md defines a hardened plugin contract
- [ ] `release.txt` / `site.txt` precedence is preserved
- [ ] Docs and examples use Go/make terminology — wrapped in Docker invocations
- [ ] No build/test/run instructions tell the user to invoke `go` on the host

## Quality Checklist

All gates run inside the project Docker image — never on the host.

- [ ] `gofmt -l .` reports no files needing formatting (Docker-wrapped)
- [ ] `golangci-lint run ./...` passes with no warnings (Docker-wrapped)
- [ ] `go test ./...` passes (Docker-wrapped)
- [ ] `go vet ./...` passes (Docker-wrapped)
- [ ] `govulncheck ./...` passes (Docker-wrapped)
- [ ] `go-licenses report ./...` output matches the generated region of `LICENSE.md` (Docker-wrapped diff check)
- [ ] If GUI is in scope: smoke test exercised under both X11 and Wayland forwarding
- [ ] New behavior includes tests where practical

## Security Checklist

- [ ] No hidden telemetry
- [ ] No secrets in logs
- [ ] No automatic privilege escalation
- [ ] No unsafe downloaded-code execution
- [ ] `govulncheck` and license checks remain enabled in CI

## Release Checklist

- [ ] Version comes from `release.txt` when present
- [ ] Official site comes from `site.txt` when present
- [ ] Release notes match actual changes
- [ ] Checksums (SHA-256) are published for every artifact
- [ ] Each release artifact is a single statically linked binary — no companion `.so` / `.dylib` / `.dll` / asset bundle
- [ ] Static-linkage check (`ldd` / `otool -L` / `dumpbin /dependents`) was run and recorded for every published target
- [ ] `LICENSE.md` regenerated and committed if `go.sum` changed; CI license-drift check is green
- [ ] Distributed binary's actual license matches what README and "About" claim (no silent GPL/LGPL relicensing via a transitive module)
- [ ] Artifact filenames follow `{project_name}-{platform}-{arch}{.ext}` with no OS suffix tokens
- [ ] Artifacts cover the supported target matrix declared in IDEA.md (defaults: `{project_name}-linux-{x86_64,aarch64}`, `{project_name}-windows-{x86_64,aarch64}.exe`, `{project_name}-macos-{x86_64,aarch64}`)
- [ ] SBOM generated via `cyclonedx-gomod` and published with the release; provenance/attestation included when the platform supports it
- [ ] Packaging metadata matches supported GUI/TUI/CLI surfaces

## Red Flags (Stop and Ask Human)

- required project variables cannot be proven
- migration would discard meaningful project-specific content
- privileged/system-wide action is needed but was not explicitly requested
- licensing or security policy conflict cannot be resolved from the spec
- requested behavior contradicts documented product scope in IDEA.md
- a proposed dependency would force CGO, dynamic linking into the release binary, or a runtime asset alongside the binary
- a feature request implies shipping multiple files, an installer-only flow, or a first-run download
- a proposed GUI toolkit only supports X11 or only supports Wayland — not both
- a proposed dependency is GPL / AGPL / LGPL / SSPL / BUSL / source-available — flag the license-relicensing implication before adding (PART 11 → "License Compliance")

## Success Criteria

A compliant Go project created from this template:
- is driven by `IDEA.md` project variables while `AI.md` stays read-only
- preserves the governance/documentation discipline of the original template
- models a single-binary, fully self-contained Go application built around GUI / TUI / CLI surfaces
- ships exclusively Go source code (small Docker shell helpers excepted)
- produces one statically linked binary per target with all assets embedded
- runs end-to-end from the binary alone on an air-gapped machine
- supports both X11 and Wayland on Linux/BSD when GUI is in scope (via the chosen pure-Go GUI toolkit)
- builds, tests, and runs only inside the project's Docker image — never on the host
- auto-selects GUI, then TUI, then CLI using environment-aware detection
- uses optional explicit privilege escalation only when requested
- uses Go tooling (via `make` targets and Docker) consistently across development, testing, and CI/CD
- maintains `CGO_ENABLED=0` across every build target without exception

---

# PART 13: IDEA.md REFERENCE

**NEVER modify this PART. Read-only reference for IDEA.md structure.**

The canonical layout, variable rules, immutability constraints, and migration procedure are defined at the top of this file under "IDEA.md Required Layout" (the section preceding PART 0). PART 13 here is the worked-example reference — every example below uses the same three-section structure.

---

## IDEA.md Structure

Every IDEA.md has exactly three top-level sections, in this order:

1. `## Project description` — free-form prose: what the project is, who uses it, what problem it solves
2. `## Project variables` — `key: value` lines that provide the canonical values AI.md resolves for `project_name`, `project_org`, `internal_name`, `internal_org`, etc.
3. `## Business logic` — features, data models, user flows, platform constraints, security assumptions (WHAT, not HOW)

See "IDEA.md Required Layout" at the top of this file for the authoritative rules: variable-key naming, the immutable `internal_name` / `internal_org` rule, the missing-value setup flow, and the migration procedure for legacy `CLAUDE.md` files.

---

## IDEA.md Template

```markdown
## Project description

{Brief description of what this project does, its primary users, and what problem it
solves. Free-form prose, 1–3 paragraphs.}

## Project variables

project_name:     {project_name}
project_org:      {project_org}
internal_name:    {project_name}        # FROZEN — equals project_name on first install, never changes
internal_org:     {project_org}         # FROZEN — equals project_org on first install, never changes
app_name:         {App Display Name}
module_path:      github.com/{project_org}/{project_name}
official_site:    {full URL with scheme, e.g., https://example.com — or empty}
maintainer_name:  {Maintainer Name}
maintainer_email: {maintainer@example.com}

## Business logic

**Target users:**
- {User type 1}
- {User type 2}

**Surfaces (declare which apply — see PART 2 → "GUI/TUI/CLI Capability Rule"):**
- GUI: {yes / no — and on which platforms}
- TUI: {yes / no}
- CLI: {yes / no}

**Features:**
- **{Feature category 1}**: {brief description}
- **{Feature category 2}**: {brief description}

**Data models:**
- {Entity}: {fields and meaning, where stored}

**User flows:**
- {Flow}: {step 1 → step 2 → outcome}

**Business rules:**
- {Validation, constraint, invariant}

**Platform constraints:**
- {OS support, hardware requirements, display-server requirements}

**Outbound network use (if any — see PART 9 → "Security-First Design"):**
- {Remote service consumed, purpose, auth model}

**Stored data location (per-user — see PART 4 → "Path Rule"):**
- {What is stored at the per-user paths}

**License exceptions (only if a denylisted or C-dependent module is used — see PART 11 → "License Compliance"):**
- {module path, upstream license, rationale, `distribution_license` consequence if applicable}
```

**Rules for the example contents above:**

- No implementation details — describe behavior, not algorithms or libraries. AI.md PARTs 0–11 define HOW; PART 12 verifies compliance.
- This template targets GUI / TUI / CLI applications (PART 0 → "One Coherent Product").
- Cross-reference AI.md PARTs by number for any pattern that already exists there (Docker → PART 5, security → PART 9, license exceptions → PART 11, etc.).
- `internal_name` and `internal_org` are immutable after first set (FIRST-TIME PROJECT SETUP → "Auto-Detecting Project Values").

---

## IDEA.md Examples

**Example 1: Notes (GUI + CLI, fully offline)**

```markdown
## Project description

A local-first notes application with a native GUI for daily use and a CLI for scripting.
Notes are stored on the user's machine — no cloud sync, no telemetry, fully offline.

## Project variables

project_name:     notes
project_org:      casjay
internal_name:    notes
internal_org:     casjay
app_name:         Notes
module_path:      github.com/casjay/notes
official_site:    https://notes.example.com
maintainer_name:  Jane Doe
maintainer_email: jane@example.com

## Business logic

**Target users:**
- Individual users wanting a fast, offline notes app
- Power users scripting note creation / search via shell

**Surfaces:**
- GUI: yes (Linux X11 + Wayland via Gio toolkit, macOS, Windows)
- TUI: no
- CLI: yes (`notes new`, `notes search`, `notes ls`)

**Features:**
- **Note management**: create, edit, archive, delete with markdown support
- **Search**: full-text search across all notes
- **Organization**: tag-based grouping; no folders
- **Export**: dump notes to a directory of `.md` files

**Data models:**
- Note: id, title, body (markdown), tags[], created_at, updated_at

**User flows:**
- Quick capture: Ctrl+N opens compose → write → Ctrl+S saves → return to list
- Search: Ctrl+K opens overlay → type → live-filter results

**Business rules:**
- Archive is reversible; delete is permanent
- All edits are local; no network
- Title can be empty; body cannot

**Platform constraints:**
- GUI requires X11 or Wayland on Linux/BSD (handled by Gio toolkit — supports both)
- No internet access required; works fully offline (PART 0 → "Self-Contained Assets")

**Outbound network use:** none

**Stored data location (per-user):**
- Notes database: `~/.local/share/casjay/notes/notes.db`
- Config: `~/.config/casjay/notes/config.toml`
- Cache: `~/.cache/casjay/notes/`

**License exceptions:**
- `modernc.org/sqlite` for the local notes database. Pure-Go SQLite implementation; compatible with `CGO_ENABLED=0`. No exception to the Go-only rule required. Distribution license remains MIT.
```

**Example 2: Feeds (TUI + CLI, consumes a remote API)**

```markdown
## Project description

A terminal-first RSS / Atom feed reader. Subscribes to feeds, fetches new items in the
background, and presents them in a keyboard-driven TUI. A `feeds` CLI exposes the same
operations for automation.

## Project variables

project_name:     feeds
project_org:      casjay
internal_name:    feeds
internal_org:     casjay
app_name:         Feeds
module_path:      github.com/casjay/feeds
official_site:
maintainer_name:  Jane Doe
maintainer_email: jane@example.com

## Business logic

**Target users:**
- Terminal-first users who follow many feeds
- Anyone wanting offline-friendly RSS reading without a hosted service

**Surfaces:**
- GUI: no
- TUI: yes (primary)
- CLI: yes (`feeds add`, `feeds sync`, `feeds export`)

**Features:**
- **Subscription**: add / remove / rename feeds; OPML import / export
- **Sync**: fetch all feeds in parallel; respect HTTP cache headers
- **Reading**: keyboard navigation; mark read / unread; star items
- **Filtering**: by feed, by tag, by unread state

**Data models:**
- Feed: id, url, title, last_etag, last_modified, last_synced
- Item: id, feed_id, guid, title, body, published_at, read, starred

**User flows:**
- Launch → background sync starts → pick a feed → arrow keys to navigate → Enter to read
- `feeds sync` from cron pulls new items without launching the TUI

**Business rules:**
- Items are deduplicated by GUID within a feed
- Sync respects ETag / Last-Modified to avoid re-fetching unchanged feeds
- Network failures degrade gracefully (read-only mode over cached items)

**Platform constraints:**
- TUI requires a TTY (PART 3 → "Smart Detect Rules"); for headless cron use the `feeds sync` CLI subcommand

**Outbound network use:**
- HTTPS GET to feed URLs the user adds. TLS via `crypto/tls` stdlib (PART 9 → "Security-First Design"). No telemetry, no tracking, no third-party analytics.

**Stored data location (per-user):**
- Feed / item database: `~/.local/share/casjay/feeds/feeds.db`
- Config: `~/.config/casjay/feeds/config.toml`
- Cache (article bodies, images): `~/.cache/casjay/feeds/`

**License exceptions:**
- `modernc.org/sqlite` for the local feeds database. Pure-Go SQLite; `CGO_ENABLED=0` compatible. Distribution license remains MIT.
```

**Example 3: dotctl (CLI only, manages dotfiles)**

```markdown
## Project description

A small CLI for managing user dotfiles: stow / unstow symlinks, version-pin individual
files, and verify that what is on disk matches what the user committed.

## Project variables

project_name:     dotctl
project_org:      casjay
internal_name:    dotctl
internal_org:     casjay
app_name:         dotctl
module_path:      github.com/casjay/dotctl
official_site:
maintainer_name:  Jane Doe
maintainer_email: jane@example.com

## Business logic

**Target users:**
- Developers managing dotfiles across multiple machines

**Surfaces:**
- GUI: no
- TUI: no
- CLI: yes (`dotctl link`, `dotctl unlink`, `dotctl status`, `dotctl diff`)

**Features:**
- **Link / unlink**: create or remove symlinks from a dotfiles repo into the user's home
- **Status**: report which managed files match, differ, or are missing
- **Diff**: show the diff between repo and on-disk versions
- **Atomic apply**: link operations are dry-run-able and reversible

**Data models:**
- ManagedFile: source_path (in repo), target_path (in home), state (linked / unlinked / divergent)

**User flows:**
- `dotctl status` shows the current state matrix
- `dotctl link --dry-run` previews what would change; `dotctl link` applies it

**Business rules:**
- Never overwrite a regular file without explicit `--force`
- Symlinks are created relative to the dotfiles repo so the home directory remains portable

**Platform constraints:**
- POSIX-style filesystem operations; Windows support uses junction-style links via the Go standard library (`os.Symlink` on Windows creates junctions where appropriate)

**Outbound network use:** none

**Stored data location (per-user — PART 4 → "Path Rule"):**
- Config: `~/.config/casjay/dotctl/config.toml`
- State (last known link map): `~/.local/share/casjay/dotctl/state.json`

**License exceptions:** none
```
