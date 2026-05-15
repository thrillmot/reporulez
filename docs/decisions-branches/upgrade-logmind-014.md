## 2026-05-15 12:57 - Initialize logmind decision tracking

**Reasoning:** Starting structured decision logging for this project to maintain clear documentation of architectural choices and provide context for AI agents.

**Alternatives considered:** Manual decision documentation, ADR (Architecture Decision Records)

**Implications:**
- All significant decisions should now be logged using `logmind.log()`
- AI agents will have access to decision history via docs/decisions.md
- Git history will serve as an audit trail for all decisions

---
## 2026-05-15 12:59 - Upgrade logmind 0.1.3 → 0.1.4

**Reasoning:** Pick up 0.1.4's scoped staging on logmind log (no longer sweeps unrelated working-tree changes into decision commits), feature-branch tree-snapshot suppression (no more PR-vs-main file-structure.md conflicts), and improved aggregator template that natively supports the LOGMIND_BOT_PAT fallback (GH_TOKEN: PAT || GITHUB_TOKEN, with a workflow-level warning when PAT isn't set).

**Alternatives considered:** Stay on 0.1.3 — misses the scoped-staging behavior and the future-tree-conflict fix, Cherry-pick only the version pin bump without regenerating workflows — would miss 0.1.4's PAT-fallback design in the aggregator

**Implications:**
- Workflows regenerated from 0.1.4 templates. Three customizations re-applied: (1) actions/checkout@v4 → @v6 (Node 20 deprecation), (2) pip install logmind → pip install 'logmind==0.1.4' (reproducibility), (3) check-doc-links.yml path filters removed so the check runs on every PR and can stay in required_status_checks. The previous PAT-on-checkout customization is DROPPED — 0.1.4 stock handles PAT correctly on gh pr create, which is the only step that actually needs a non-Actions identity. Live ruleset's required checks (clud-bug-review, check-decisions, check-links) are unchanged.

---
