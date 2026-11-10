---
name: vue-expert
description: Diagnose or implement Vue component reactivity, rendering, props/emits/v-model, composables, watchers and async lifecycle cleanup. Use for substantive Vue behavior; Pinia store ownership and Vue Router navigation have more specific skills, while styling and routine typed edits alone do not need this skill.
metadata:
  version: "2.0.0"
---

# Vue Expert

Correct Vue behavior with the smallest change that preserves project architecture and component contracts.

## Workflow

1. Inspect installed Vue/compiler versions, component style, rendering environment and relevant state owners. Do not assume current online APIs exist in the application.
2. Trace the affected props/state/effects and parent-child or async boundary before changing behavior. Diagnose the actual cause, not only the visible render symptom.
3. Read only the relevant reference below. Keep established Composition/Options API conventions and avoid incidental migration or mandatory component splitting.
4. Verify the changed update path and meaningful failure/cleanup case using existing project tools. Browser behavior and hydration require corresponding integration checks, not only a successful build.

Use pinia-patterns for substantive store lifecycle and vue-router-patterns for navigation/URL contracts, if available. frontend-developer owns broader UI/accessibility/responsive work. JavaScript/TypeScript/test skills load only when the task has that distinct need; do not activate every skill for a .vue file.

## References by concern

| Concern | Reference |
|---|---|
| ref/reactive/computed, props, v-model, identity and rendering diagnosis | [Reactivity and components](references/reactivity-and-components.md) |
| Watchers, stale requests, cleanup, composables and SSR boundaries | [Async and cleanup](references/async-and-cleanup.md) |
| Source revisions and local changes | [Sources](references/sources.md) |

Preserve domain names, units and input/output contracts. Avoid global state, deep watches, forced remounts or new dependencies as automatic fixes. Respect user authorization and analysis-only mode without adding another generic gate.

Report concrete behavior changes, checks actually run and material compatibility/integration limits, proportionally to the task.
