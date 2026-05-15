## 2026-05-15 02:08 - Initialize logmind decision tracking

**Reasoning:** Starting structured decision logging for this project to maintain clear documentation of architectural choices and provide context for AI agents.

**Alternatives considered:** Manual decision documentation, ADR (Architecture Decision Records)

**Implications:**
- All significant decisions should now be logged using `logmind.log()`
- AI agents will have access to decision history via docs/decisions.md
- Git history will serve as an audit trail for all decisions

---
## 2026-05-15 02:24 - Switch logmind-aggregate from direct push to peter-evans/create-pull-request, add logmind-decision-check CI gate

**Reasoning:** The reporulez ruleset on this repo blocks direct pushes to main (no bypass_actors). The default logmind-aggregate workflow pushed straight to main and would silently fail every merge. Per Clud Bug review on PR #4. Also adds a PR-level decision-log enforcement check to mirror the local pre-commit hook.

**Alternatives considered:** Add github-actions[bot] to bypass_actors with bypass_mode=always — weakens security since any workflow could then push to main, Skip aggregation entirely and rely on manual logmind aggregate runs — defeats automation

**Implications:**
- Each merge of a feature PR now opens/updates an auto-generated PR on branch logmind-aggregate. Concurrency group prevents non-fast-forward races. logmind-decision-check workflow gates every PR on having a decision logged when >20 lines changed.

---
## 2026-05-15 03:17 - Use LOGMIND_BOT_PAT for aggregator workflow; make logmind-check always run with conditional no-op for aggregator branch

**Reasoning:** peter-evans/create-pull-request invoked with secrets.GITHUB_TOKEN doesn't trigger downstream pull_request workflows (GitHub design). With required status checks planned, the auto-generated aggregator PR would be permanently unmergeable. Per Clud Bug re-review of fc81a0e.

**Alternatives considered:** Add github-actions[bot] to ruleset bypass_actors — weakens repo security, Skip required status checks entirely — defeats the enforcement goal

**Implications:**
- Repo secret LOGMIND_BOT_PAT must be set (fine-grained PAT with contents:write + pull_requests:write). logmind-check.yml restructured so it always runs; the aggregator-branch shortcut is now a step-level conditional that reports success rather than skipped. Decision-doc check now counts lines added (min 3) instead of files touched, closing the whitespace-bypass.

---
