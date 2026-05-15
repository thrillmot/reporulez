# AGENTS.md

This is the canonical instruction file for AI coding agents working in this
repository. Tools that understand `AGENTS.md` (Cursor, Codex, Windsurf,
Claude Code, Cline, Continue, Aider, ...) read this file directly. Per-tool
files like `CLAUDE.md` or `.cursorrules` are stubs that point here so the
guidance lives in one place.

<!-- logmind-start -->
<!-- logmind-block-version: v3-slim -->
## Decision logging — see the `logmind` skill

This project uses [logmind](https://logmind.dev). The full procedure
(when to log, how to log, what counts as a decision, branch routing) lives
in the **`logmind` agent skill** which your runtime should auto-load.

If the skill isn't loaded for some reason, install it once:

```bash
npx skills add https://github.com/thrillmot/agent-skills --skill logmind
```

### Project-specific paths

- **[docs/timeline.md](docs/timeline.md)** — auto-generated chronological overview across all branches; start here.
- Recent decisions on the default branch: **[docs/decisions.md](docs/decisions.md)**
- Per-branch decisions (in-flight feature work): **docs/decisions-branches/**
- Archived decisions: **[docs/decisions-archive.md](docs/decisions-archive.md)**
- Project tree (regenerated on main-branch logs + post-PR-merge): **[docs/file-structure.md](docs/file-structure.md)**

### Quick reference

```bash
logmind log "decision summary" -r "why" -a "alternative" -i "implication"
logmind show               # recent decisions on the current branch
logmind search "keyword"   # full-text across recent + archive
```

**Use `logmind log` for the commit, not `git add` + `git commit`.** The
`log` command writes the decision file, stages the decision log + its
companion files, and creates the commit in one step. Use
`--stage all` to also stage the rest of the working tree.

**Read `docs/decisions.md` and the matching `docs/decisions-branches/<branch>.md` (if any) before starting any non-trivial task.** The team has likely already decided things you'd otherwise re-litigate.
<!-- logmind-end -->

## Project Overview

`reporulez` ships opinionated GitHub **repository rulesets** for AI-driven
development. The installer (`bin/apply.sh`) tunes target-repo settings
(auto-merge, squash-only, delete-on-merge) and POSTs a ruleset that requires
PRs, blocks force pushes and default-branch deletions, enforces thread
resolution, and (in the `copilot` variant) wires up GitHub Copilot
auto-review. Two variants — `copilot` and `external` — plus a
`--human-review` flag. See `README.md` for the full design.

This repo itself uses the `external` variant: clud-bug is the AI reviewer,
logmind enforces decision logging, and CI gates every PR on `clud-bug-review`,
`check-decisions`, and `check-links` (all currently passing).

## Development Commands

```bash
# Validate ruleset JSON
jq . rulesets/*.json

# Syntax-check the installer
bash -n bin/apply.sh

# Apply rulesets to a target repo (dogfood / end-to-end test)
./bin/apply.sh <owner/repo> [copilot|external] [--human-review]
```

There is no test suite. End-to-end validation is "apply against a throwaway
repo and inspect via the GitHub UI / API."

## Contributor Setup

After cloning, contributors should:

1. **Install logmind** locally (CLI tool used to log decisions):
   ```bash
   pipx install logmind   # or: brew install thrillmot/logmind/logmind
   logmind install-hook   # .git/hooks/pre-commit
   ```
   The same enforcement runs on every PR via `.github/workflows/check-decisions.yml`
   (shipped by logmind itself), so skipping the local hook only delays the failure —
   it still blocks merge.

2. **When CI flags `docs/timeline.md` as stale,** regenerate it locally and push:
   ```bash
   logmind timeline --write docs/timeline.md
   git add docs/timeline.md
   git commit -m "regen: docs/timeline.md"
   git push
   ```
   `docs/timeline.md` is a derived file — auto-regenerated chronological overview
   across all branches. The `check-derived-docs` workflow (`.github/workflows/regen-timeline.yml`)
   fails fast when it's stale. Running `logmind timeline --write` locally before
   pushing avoids the red CI run.
