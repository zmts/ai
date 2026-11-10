---
name: javascript-patterns
description: Diagnose JavaScript language behavior involving promises, async ordering, cancellation, ESM/CommonJS interoperability, iterators, closures, or mutation. Use for JS semantics in JavaScript or TypeScript; ordinary Vue rendering, CSS, and routine typed edits do not need this skill.
metadata:
  version: "1.0.0"
---

# JavaScript Patterns

Resolve the actual language or module issue with the smallest compatible change.

## Workflow and scope

1. Inspect the relevant package.json, lockfile, runtime/browser targets and execution path: Node, browser, SSR or build tool.
2. Trace values, ownership, async ordering and rejection handling before changing behavior. Preserve observable contracts and project conventions.
3. Read only the matching reference below. A recent development runtime does not establish deployment compatibility.
4. Verify behavior and its meaningful failure path using existing project commands. Analysis-only requests remain read-only.

Use the framework skill for component lifecycle/reactivity, `nodejs-pro` for Node service lifecycle and `typescript-patterns` for static contracts, if available. Do not load every skill for routine edits.

## Decision rules

- Await, return or explicitly handle promises. A floating void expression does not handle rejection.
- Choose sequential execution for dependencies; bound parallel work to resource capacity.
- Handle errors where recovery or translation is possible. Do not turn failures into empty successes without an explicit contract.
- Preserve missing/null/empty/zero/false distinctions. Use ?? when only null/undefined should trigger a default.
- Prefer const and simple control flow; retain callbacks required by APIs. Avoid unrelated syntax modernization.
- Spread is shallow. Preserve identity where callers/frameworks rely on it; do not force immutable copying or class hierarchies.
- Preserve precise domain names and units such as durationMs/changePct; retain canonical contract fields and avoid incidental renaming.
- Keep existing dependencies, module format and test tools unless the task needs a change. No mandatory coverage target or abstraction.

## Load references by task

| Task | Reference |
|---|---|
| Promises, deadlines, cancellation, concurrency and cleanup | [Async and resources](references/async-and-resources.md) |
| Module formats, exports, extensions and runtime resolution | [Modules](references/modules.md) |
| Default values, identity, this and iteration | [Values and iteration](references/values-and-iteration.md) |
| Upstream revision and local adaptations | [Sources](references/sources.md) |

Report the behavior changed, checks actually run and material compatibility limits, proportionally to the task.
