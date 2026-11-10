---
name: typescript-patterns
description: Diagnose TypeScript inference, narrowing, generics, discriminated unions, public type contracts, or compiler/module-resolution issues. Use for substantive type-system work; routine typed component edits, styling, and Node runtime issues alone do not need this skill.
metadata:
  version: "1.0.0"
---

# TypeScript Patterns

Make static contracts reflect runtime truth with the simplest adequate types.

## Workflow and scope

1. Inspect installed TypeScript, effective tsconfig, package boundaries and the failing producer/consumer. Distinguish compiler, runtime and bundler errors.
2. Reproduce the contract issue with a minimal relevant example when useful. Trace consumers before public type changes.
3. Read only the relevant reference below. Prefer inference, narrowing and built-in utilities before generic machinery.
4. Check valid and meaningful invalid usage using project compiler/test commands. External data and changed behavior also need runtime verification. Analysis-only requests remain read-only.

Keep `nodejs-pro` for Node service/runtime issues and the Vue skill for reactivity/lifecycle. Load `javascript-patterns` only for a separate JS semantics issue, if available. A .ts extension alone does not warrant this skill.

## Contract rules

- Types, casts, generics and satisfies do not validate JSON or SDK responses at runtime.
- Use unknown at untrusted boundaries and narrow before access. Reuse existing validators or guards that prove the required fields.
- Use discriminated unions when they remove invalid state combinations; preserve existing null/error semantics.
- Preserve domain names and units. Use top-level import type for type-only dependencies and established public type modules unless they create cycles.
- Do not add brands, generic utilities, layers or dependencies without concrete benefit. Do not cast away a real inconsistency.
- Preserve compiler/module settings. Do not enable new strict flags, migrate enums or rewrite module structure incidentally.
- Separate DTO/domain/persistence models where meanings differ; avoid mechanical shape duplication.

## Load references by task

| Task | Reference |
|---|---|
| Untrusted data, guards and state unions | [Boundaries and narrowing](references/boundaries-and-narrowing.md) |
| Generic relationships, utilities and type tests | [Generics and inference](references/generics-and-inference.md) |
| Configuration, emit and runtime resolution | [Compiler and modules](references/compiler-and-modules.md) |
| Upstream revision and local corrections | [Sources](references/sources.md) |

Report the corrected contract and checks actually run. Distinguish static guarantees from runtime validation.
