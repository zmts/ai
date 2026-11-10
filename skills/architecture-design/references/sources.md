# Sources and local adaptation

Reviewed 2026-10-20. Local version 1.0.0 is not an upstream release.

- [netsky-prod/architecture-patterns — architecture-patterns](https://github.com/netsky-prod/architecture-patterns/tree/1aacb24751b9cbdec4b1c86f8776365987c8b8ed/skills/architecture-patterns), pinned commit `1aacb24751b9cbdec4b1c86f8776365987c8b8ed`. Inspected: skills/architecture-patterns/SKILL.md, skills/architecture-patterns/patterns/system/outbox.md, skills/architecture-patterns/patterns/code/strategy.md.
- [Jeffallan/claude-skills — architecture-designer](https://github.com/Jeffallan/claude-skills/tree/958e60fb998e008507c978b7943211f337def6a6/skills/architecture-designer), pinned commit `958e60fb998e008507c978b7943211f337def6a6`. Inspected: skills/architecture-designer/SKILL.md, skills/architecture-designer/references/architecture-patterns.md.
- [mattpocock/skills — codebase-design](https://github.com/mattpocock/skills/tree/321658273cb1d20b76026717d027d505790106d4/skills/engineering/codebase-design), pinned commit `321658273cb1d20b76026717d027d505790106d4`. Inspected: skills/engineering/codebase-design/SKILL.md.
- [mattpocock/skills — improve-codebase-architecture](https://github.com/mattpocock/skills/tree/d80fa0f4ebe0c5714af0adf8670336065233ecc6/skills/engineering/improve-codebase-architecture), pinned commit `d80fa0f4ebe0c5714af0adf8670336065233ecc6`. Inspected: skills/engineering/improve-codebase-architecture/SKILL.md.

## Adaptation

Removed mandatory terminology, HTML reports, fixed pattern counts and automatic ADR creation. Retained concrete trade-offs and plain-solution alternatives. Clarified transactional outbox duplicates, durable idempotency and migration limits.

Local text is independently written; upstream code and reference files are not vendored. Preserve the original license/notice if upstream material is copied in a future update. Primary standards/product documentation are linked where applicable.

Use available Codex file/search tools and Markdown/Mermaid as appropriate. No dependency on an unavailable Skill tool, HTML reporting system, issue tracker, mandatory subagents or extra approvals is introduced.

For updates, compare sources against the pinned revisions, inspect references and preserve local user intent, scope and tested corrections. Newer source dates alone do not establish better behavior.

## Verification scope

Structural checks and a manual scenario walkthrough cover local-fix routing, a coupled-module refactor, API compatibility and database-write/event publication. This is a static instruction review, not an independent model benchmark or a running application reliability test.

Walkthrough: an ordinary bug fix stays with its implementation skill; a module split traces callers/data ownership before adding layers; an API rename routes to api-design with compatibility analysis; a write/publish crash gap considers outbox only when durable delivery is required and retains duplicate-consumer handling.
