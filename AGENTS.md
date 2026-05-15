# AGENTS.md

This is the canonical instruction file for AI coding agents working in this
repository. Tools that understand `AGENTS.md` (Cursor, Codex, Windsurf,
Claude Code, Cline, Continue, Aider, ...) read this file directly. Per-tool
files like `CLAUDE.md` or `.cursorrules` are stubs that point here so the
guidance lives in one place.

<!-- logmind-start -->
<!-- logmind-block-version: v2-slim -->
## Decision logging — see the `logmind` skill

This project uses [logmind](https://logmind.dev). The full procedure
(when to log, how to log, what counts as a decision, branch routing) lives
in the **`logmind` agent skill** which your runtime should auto-load.

If the skill isn't loaded for some reason, install it once:

```bash
npx skills add https://github.com/thrillmot/agent-skills --skill logmind
```

### Project-specific paths

- Recent decisions on the default branch: **[docs/decisions.md](docs/decisions.md)**
- Per-branch decisions (in-flight feature work): **docs/decisions-branches/**
- Archived decisions: **[docs/decisions-archive.md](docs/decisions-archive.md)**
- Project tree (auto-regenerated on every log): **[docs/file-structure.md](docs/file-structure.md)**

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
   brew install thrillmot/logmind/logmind   # or: pipx install logmind
   logmind install-hook                     # .git/hooks/pre-commit
   ```
   The same enforcement runs on every PR via `.github/workflows/check-decisions.yml`
   (shipped by logmind itself), so skipping the local hook only delays the failure —
   it still blocks merge.

2. **Set repo secret `LOGMIND_BOT_PAT`** (one-time, per fork/clone of this repo):
   The `logmind-aggregate` workflow opens a PR with the aggregated
   `docs/decisions.md` update on every merge. PRs opened with the default
   `GITHUB_TOKEN` don't trigger downstream `pull_request` workflows
   ([known GitHub design](https://github.com/peter-evans/create-pull-request/blob/main/docs/concepts-guidelines.md#triggering-further-workflow-runs)),
   so the aggregator PR would never get its required checks and would be
   permanently unmergeable under this repo's ruleset.

   Create a **fine-grained PAT** scoped to this repo with
   `contents: write` + `pull_requests: write`, and add it as the repo secret
   **`LOGMIND_BOT_PAT`**. Without it, the aggregator step fails clearly
   rather than silently.
