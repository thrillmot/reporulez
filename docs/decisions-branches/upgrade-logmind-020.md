## 2026-05-15 17:13 - Initialize logmind decision tracking

**Reasoning:** Starting structured decision logging for this project to maintain clear documentation of architectural choices and provide context for AI agents.

**Alternatives considered:** Manual decision documentation, ADR (Architecture Decision Records)

**Implications:**
- All significant decisions should now be logged using `logmind.log()`
- AI agents will have access to decision history via docs/decisions.md
- Git history will serve as an audit trail for all decisions

---
## 2026-05-15 17:15 - Upgrade logmind 0.1.4 → 0.2.0 (derived-file architecture)

**Reasoning:** Pick up v0.2's derived-file architecture: docs/timeline.md regenerated deterministically on every PR. Retires the per-merge aggregator workflow that caused our LOGMIND_BOT_PAT / required-checks pain across PRs #4 and #5. The new regen-timeline.yml is verify-only (fail-fast, contents:read), no auto-commit, no token gymnastics.

**Alternatives considered:** Stay on 0.1.4 — keeps the aggregator workflow that uses LOGMIND_BOT_PAT and is no longer maintained

**Implications:**
- logmind-aggregate.yml deleted. regen-timeline.yml added with our standard customizations (checkout v6, pin to logmind==0.2.0). AGENTS.md block bumped v2-slim -> v3-slim (adds docs/timeline.md to required reading). Contributor Setup section's PAT step replaced with timeline-regen instructions. Workflow permissions flipped to 'write' (not strictly needed for the verify-only regen-timeline but per the v0.2 release-notes recommendation). LOGMIND_BOT_PAT secret was already absent on this repo; no cleanup needed. Required-checks update on the live ruleset (adding check-derived-docs) deferred to a follow-up after merge.

---
