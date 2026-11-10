# Sources and local adaptation

Reviewed 2026-10-20. Local version 1.0.0 is not an upstream release.

- [wshobson/agents — api-design-principles](https://github.com/wshobson/agents/tree/be57c0b2e3c05c528ca6132b87410b385718775f/plugins/backend-development/skills/api-design-principles), pinned commit `be57c0b2e3c05c528ca6132b87410b385718775f`. Inspected: plugins/backend-development/skills/api-design-principles/SKILL.md, plugins/backend-development/skills/api-design-principles/references/details.md.

## Adaptation

Removed categorical bans on domain actions, automatic protocol/version migrations and claims that GraphQL resolves all fetching issues. Added HTTP method semantics, nullable/missing fields, pagination under concurrent changes and explicit retry outcomes.

Local text is independently written; upstream code and reference files are not vendored. Preserve the original license/notice if upstream material is copied in a future update. Primary standards/product documentation are linked where applicable.

Use available Codex file/search tools and Markdown/Mermaid as appropriate. No dependency on an unavailable Skill tool, HTML reporting system, issue tracker, mandatory subagents or extra approvals is introduced.

For updates, compare sources against the pinned revisions, inspect references and preserve local user intent, scope and tested corrections. Newer source dates alone do not establish better behavior.

## Verification scope

Structural checks and a manual scenario walkthrough cover local-fix routing, a coupled-module refactor, API compatibility and database-write/event publication. This is a static instruction review, not an independent model benchmark or a running application reliability test.

Walkthrough: an endpoint rename preserves known consumers or defines a migration; a timestamp cursor requires a tie-breaker and explicit mutation semantics; retried create requests need atomic durable identity handling; changed errors/nullability are treated as contract changes.
