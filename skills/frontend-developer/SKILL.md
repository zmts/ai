---
name: frontend-developer
description: Implement scoped frontend changes and UI bug fixes with production-level behavior and quality. Use when a task needs focused work on routes, components, client state, rendering logic, async UI behavior, accessibility, or user-visible interaction bugs without expanding into unrelated redesign or refactor work.
---

# Frontend Developer

Own frontend changes as user-visible product behavior plus state integrity.

## Working Mode

1. Map the route, component, state, and data boundaries for the target flow.
2. Implement the smallest coherent UI change that solves the requested problem.
3. Validate behavior, accessibility, and the nearest likely regressions.

For substantive Vue component behavior, use vue-expert; Pinia ownership and Vue Router navigation use pinia-patterns and vue-router-patterns when available. Load only the skill needed for the concrete issue; do not require all of them for every frontend edit.

## Focus Areas

- Keep component and state ownership clear.
- Prefer explicit state transitions over hidden side effects.
- Preserve rendering correctness during async updates and conditional paths.
- Align frontend behavior with backend and API contracts.
- Preserve the established design system, layout patterns, and interaction conventions.
- Keep loading, empty, and error states consistent with surrounding UI.
- Check keyboard and focus behavior for interactive elements.

## Implementation Checks

- Avoid introducing new abstractions unless they remove repeated complexity.
- Keep diffs scoped, reviewable, and local to the changed path.
- Preserve behavior outside the target flow.

## Validation Checks

- Verify the exact user flow that was fixed or implemented.
- Test one high-risk edge transition such as an async race, stale data path, or conditional re-render.
- Confirm there is no obvious accessibility regression.
- Call out cache, API, or runtime assumptions that still require integration verification.

## Return Format

When reporting the work, include:

- The changed UI path and touched files.
- A concise summary of the behavior change.
- The validation performed.
- Any residual UI, accessibility, or integration risk.

## Guardrails

- Do not broaden into unrelated redesign work unless the user explicitly asks for it.
- Do not broaden into cleanup refactors unless they are required to land the scoped change safely.
- Prefer shipping the smallest correct change over opportunistic improvements.
