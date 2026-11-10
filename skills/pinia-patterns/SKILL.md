---
name: pinia-patterns
description: Diagnose or implement Pinia store ownership, state/getter/action reactivity, subscriptions, reset, persistence boundaries or SSR store isolation. Use for substantive Pinia issues; local component state, generic Vue rendering and ordinary API fetching alone do not need this skill.
metadata:
  version: "1.0.0"
---

# Pinia Patterns

Use stores for genuinely shared application state while preserving the project's existing store style.

## Workflow

1. Inspect installed Pinia/Vue versions, owning pinia instance, store definition and actual consumers. Identify local UI state, shared client state, URL state and server data separately.
2. Trace writes, getters, async actions, subscriptions and identity/session transitions. Find which owner is responsible for reset/cancellation.
3. Read the relevant reference below and implement the smallest consistent change. Do not move all component state/API responses into stores or install caching/persistence plugins automatically.
4. Verify updates, failure/obsolete-response behavior and reset/isolation where affected, using existing project tools. Unit mocks do not establish persistence or SSR behavior.

Use vue-expert for separate component/reactivity issues and vue-router-patterns for navigation/URL synchronization, if available. Do not load all three for a routine store call.

## References

| Concern | Reference |
|---|---|
| Destructuring, getters/actions, ownership and reset | [Store state](references/store-state.md) |
| Async actions, subscriptions, persistence and SSR | [Lifecycle and isolation](references/lifecycle-and-isolation.md) |
| Pinned source and adaptations | [Sources](references/sources.md) |

Preserve current Options/Setup Store conventions, contracts and authorization. Report checks actually performed and unresolved integration limits. Analysis-only requests remain read-only.
