# reporulez

Drop-in GitHub repository rulesets tuned for AI-driven development.

The goal: let an AI agent branch → push → open a PR → wait for checks → merge, without
bypassing safety. Force pushes and default-branch deletions are blocked, every PR is
reviewed (by Copilot or an external AI reviewer), every review thread must be resolved,
and all required status checks must pass before merge. Optionally require a human approval.

## Quickstart

```sh
# Default: Copilot auto-review enabled, no human approval required (full AI auto-mode).
curl -fsSL https://raw.githubusercontent.com/thrillmot/reporulez/main/bin/apply.sh \
  | bash -s -- owner/repo

# With a human-in-the-loop approval gate:
curl -fsSL https://raw.githubusercontent.com/thrillmot/reporulez/main/bin/apply.sh \
  | bash -s -- owner/repo copilot --human-review

# If you already use a non-Copilot AI reviewer (Claude Code Review, CodeRabbit, Cursor, …):
curl -fsSL https://raw.githubusercontent.com/thrillmot/reporulez/main/bin/apply.sh \
  | bash -s -- owner/repo external
```

Requires the [`gh`](https://cli.github.com) CLI authenticated against the target repo, and `jq`.

## Variants

| Variant | Copilot auto-review | Use when |
|---|---|---|
| `copilot` (default) | enabled via the `copilot_code_review` ruleset rule | You want GitHub's built-in reviewer to comment on every PR. |
| `external` | not included | You've installed a non-Copilot AI reviewer GitHub App that already comments on every PR. |

Both variants are otherwise identical: PR required, force push and deletion blocked,
linear history, squash-only merges, dismiss stale reviews, last-push approval, all
threads must resolve, status checks must pass (with strict / branch-up-to-date enforced).

### Human approval flag

`--human-review` patches `required_approving_review_count` from `0` to `1`. Approvals
must come from a **human** — both GitHub Copilot code review and Anthropic's Claude
Code Review GitHub App submit *Comment* reviews only, never *Approve*, so they cannot
satisfy this count. (Bots that *can* approve, like CodeRabbit's auto-approve, do.)

## What gets configured

The installer applies two things:

1. **A repository ruleset** (`reporulez-default`) targeting the default branch:
   - PR required, with last-push approval, thread resolution, stale-review dismissal
   - Required status checks (strict mode; you fill in the check names afterward)
   - Block default-branch deletion
   - Block force pushes
   - Require linear history
   - Allowed merge methods: `squash`
   - (copilot variant only) Copilot code review on every push, not on drafts
2. **Repository settings** that rulesets can't control:
   - Auto-merge enabled
   - Squash-only merging
   - Delete head branch on merge
   - Squash commit title = PR title, message = PR body

The script is idempotent — running it twice updates the existing ruleset instead of creating a duplicate.

## After install — manual steps

1. **Add CI status check names** to the ruleset. The installer leaves
   `required_status_checks` empty because we can't know your workflow names.
   Settings → Rules → Rulesets → `reporulez-default` → "Require status checks to pass".
2. **Drop in templates** if you want:
   ```sh
   curl -fsSL https://raw.githubusercontent.com/thrillmot/reporulez/main/templates/CODEOWNERS \
     -o .github/CODEOWNERS
   curl -fsSL https://raw.githubusercontent.com/thrillmot/reporulez/main/templates/pull_request_template.md \
     -o .github/pull_request_template.md
   ```
3. **Verify entitlement / app install:**
   - `copilot` variant: the repo must have Copilot code review available (Pro / Pro+ / Business).
   - `external` variant: an AI reviewer GitHub App must be installed and configured.

## Hand-import without the script

If you don't want to run a shell script (e.g. inside CI), import the JSON directly:

```sh
gh api --method POST repos/owner/repo/rulesets \
  --input rulesets/copilot.json
```

To require a human approval in this path, edit the JSON's `required_approving_review_count` to `1` first.

## Out of scope (for now)

- Org-level rulesets (use repo-level for now; org-level lives at a different API path)
- Tag protection
- Push rulesets (file paths, file sizes, etc.)
- Required signed commits — high friction for AI agents without signing keys
- Environment / deployment protection rules

## License

MIT — see [LICENSE](LICENSE).
