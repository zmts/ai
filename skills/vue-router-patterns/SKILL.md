---
name: vue-router-patterns
description: Diagnose or implement Vue Router params/query, reused route components, guards and redirects, navigation failures or URL/state synchronization. Use for actual routing behavior; generic component rendering, styling and backend authorization alone do not need this skill.
metadata:
  version: "1.0.0"
---

# Vue Router Patterns

Preserve navigation and URL contracts using the installed router version and existing route configuration.

## Workflow

1. Inspect Vue Router/Vue versions, route declarations, history/base, guards, owning components and store dependencies. Router 4 guidance is not a blanket guarantee for Router 5+ APIs/tooling.
2. Reproduce the navigation including parameter/query changes and back/forward behavior where relevant. Distinguish reused components, failed navigation and stale data.
3. Read the relevant reference below. Correct the actual route/state dependency instead of forcing remounts, duplicating canonical state or adding guards everywhere.
4. Verify successful navigation plus the affected cancel/redirect/error case. Memory-history tests do not prove browser history fallback, scroll/focus or SSR hydration behavior.

Use vue-expert for separate watcher/component lifecycle issues and pinia-patterns for store ownership, if available. Client guards improve UX; authorization still belongs on the backend.

## References

| Concern | Reference |
|---|---|
| Params/query, reuse, guards, redirects and failures | [Navigation](references/navigation.md) |
| URL ownership, loading races, browser/SSR boundaries | [URL and async state](references/url-and-async-state.md) |
| Pinned source and adaptations | [Sources](references/sources.md) |

Preserve project conventions, user scope and selected history mode. Report evidence and browser/environment limitations. Analysis-only requests stay read-only.
