## 2026-05-15 11:35 - Initialize logmind decision tracking

**Reasoning:** Starting structured decision logging for this project to maintain clear documentation of architectural choices and provide context for AI agents.

**Alternatives considered:** Manual decision documentation, ADR (Architecture Decision Records)

**Implications:**
- All significant decisions should now be logged using `logmind.log()`
- AI agents will have access to decision history via docs/decisions.md
- Git history will serve as an audit trail for all decisions

---
## 2026-05-15 11:40 - Fresh-install logmind 0.1.3, retire custom logmind-check.yml in favor of stock check-decisions.yml

**Reasoning:** logmind 0.1.3 ships its own check-decisions.yml CI workflow plus an aggregator with smart fallback. Standardizing on stock workflows reduces upstream drift. Keep two project-specific tweaks: (1) actions/checkout v4 -> v6 (Node 20 deprecation), (2) aggregator's PAT instead of GITHUB_TOKEN (downstream-workflow trigger). PAT is still needed because GITHUB_TOKEN-authored PRs don't trigger pull_request events under GitHub's design — same root cause as in 0.1.1.

**Alternatives considered:** Keep my custom logmind-check.yml — diverges from upstream, harder to maintain, Use stock as-is without checkout bump or PAT — Node 20 deprecation + fallback PR unmergeable under required checks

**Implications:**
- Live ruleset's required_status_checks updated: dropped logmind-decision-check, added check-decisions. PR #5 must run check-decisions on its own branch (workflow file present) for the PR to merge.

---
