---
name: javascript-patterns-registry
description: Diagnose JavaScript runtime behavior or choose, explain, implement and review engineering patterns in JavaScript and TypeScript. Use for promise ordering, cancellation, module interoperability, value/identity/mutation issues, named patterns or concrete choices about composition, collections, state and resources. Pure type-system errors, ordinary UI styling and framework-specific behavior alone belong to their specialized skills.
metadata:
  version: "1.1.0"
---

# JavaScript Patterns Registry

Use the smallest solution that satisfies the actual contract. Keeping a direct call, function, conditional or built-in collection is a valid outcome.

## When to use

- The issue involves JavaScript language or module behavior: promises, async ordering, cancellation, ESM/CommonJS interoperability, this, closures, iteration, defaults, identity or mutation, including in TypeScript code.
- The user names a pattern and asks to explain, compare, implement or review it in JavaScript or TypeScript.
- A concrete design choice needs comparison: how objects are created, how behavior is composed, where state lives, or which data representation fits the operations.
- A resource, error-handling or performance change needs an example showing the relevant invariant, failure mode and simpler alternative.

Choose between language diagnosis and pattern selection below. They are entry points for a concrete task, not a checklist to apply to every task. A `.js` or `.ts` extension alone does not warrant this skill.

## Workflow

1. Establish the observed problem, caller expectations and relevant constraints. Inspect package.json, the lockfile, runtime/browser targets and the actual execution path when compatibility matters; a recent development runtime does not establish deployment compatibility.
2. For a behavior issue, trace values, ownership, async ordering, rejection handling or module initialization before choosing a fix. Read only the matching language reference below. A language bug does not automatically require a design pattern.
3. For a pattern question, select the matching category or named entry. Compare applicability, excessive-use cases and simpler alternatives against the actual task. Do not load the whole registry by default; use its table of contents and pattern headings to locate the relevant material.
4. Treat the good/bad examples as illustrations of contracts and failure modes. Adapt them to the project's runtime, types, dependencies and conventions rather than copying them mechanically. Preserve the stated limitations: teaching fragments are not complete production implementations.
5. Apply the smallest compatible change when authorized. Verify the relevant behavior and meaningful edge or failure case with existing project commands. For explanation or review, report the choice and tradeoff without implying permission to change code.

## Language diagnosis references

| Behavior or compatibility issue | Read |
|---|---|
| Promise combinators, async callbacks, ordering, cancellation, deadlines or owned cleanup | [Async and resources](references/async-and-resources.md) |
| ESM/CommonJS interoperability, exports, emitted extensions, initialization cycles or runtime resolution | [Modules and compatibility](references/modules.md) |
| Defaults, this, shallow copies, mutation, lazy iteration or numeric/serialization semantics | [Values, identity and iteration](references/values-and-iteration.md) |

## Shared correctness rules

- Await, return or explicitly handle promises. A floating void expression does not handle rejection. Use sequential execution for dependencies and bound concurrent work to resource capacity.
- Recover or translate errors where the layer can do so meaningfully; otherwise propagate them. Do not turn a failure into an empty success without an explicit contract.
- Preserve missing, null, empty, zero and false distinctions. Use ?? when only null/undefined should trigger a default.
- Spread is shallow. Preserve identity and input ownership where callers depend on them; copying and class hierarchies are not universal requirements.
- Preserve precise domain names, units, public fields, module format, existing dependencies and test tools. Prefer const and simple control flow and retain callbacks required by APIs; avoid unrelated syntax modernization. Do not introduce mandatory coverage targets or test abstractions.

## Categories and entry points

| Decision or question | Read |
|---|---|
| Factories versus constructors, builders, cloning, Singleton/Multiton or request context | [Object creation and state scope](references/js-patterns-registry.md#1-object-creation-and-state-scope) |
| Adapter/Wrapper/Decorator/Proxy distinctions, Bridge versus Strategy, composite structures or component coordination | [Interface boundaries, composition and behavior management](references/js-patterns-registry.md#2-interface-boundaries-composition-and-behavior-management) |
| Shared immutable data, object/resource pools, leases, bounded memory or subscription cleanup | [Memory sharing and resource management](references/js-patterns-registry.md#3-memory-sharing-and-resource-management) |
| Object/Map/Set/Array selection, weak collections, binary layouts, aliases or snapshot semantics | [Data representation and collection selection](references/js-patterns-registry.md#4-data-representation-and-collection-selection) |
| FIFO/LIFO, rings, linked or persistent lists, trees, heaps, tries, traversal, LRU or replicated counters | [Data structures, search, traversal and distributed state](references/js-patterns-registry.md#5-data-structures-search-traversal-and-distributed-state) |
| Error classification/translation, recovery ownership, safe retry or process shutdown | [Errors and recovery](references/js-patterns-registry.md#6-errors-and-recovery) |
| Decomposition, mutation and input ownership, effect order, explicit dependencies or domain/I/O separation | [Code organization and data ownership](references/js-patterns-registry.md#7-code-organization-and-data-ownership) |
| Profiled object-shape, array-representation or allocation concerns, and valid benchmark comparisons | [Performance and measurement](references/js-patterns-registry.md#8-performance-and-measurement) |

## Scope and boundaries

- This skill owns JavaScript behavior diagnosis and pattern selection, not a mandatory architecture or style. Do not add classes, layers, custom collections, caches, pools or dependencies solely to match a pattern name.
- JavaScript and TypeScript use the same runtime principles. Add appropriate static contracts in TypeScript; types and readonly do not replace runtime validation or deep immutability.
- Keep static type-system issues with typescript-patterns and substantive system boundaries with architecture-design when those separate concerns arise and the skills are available. Vue/component reactivity, store ownership and routing belong to their framework skills. Do not load every related skill for an ordinary JavaScript or pattern question.
- Node service lifecycle belongs with nodejs-pro; Nest provider scopes, injection tokens and hooks belong with nestjs-patterns. General examples must respect the existing framework mechanisms rather than introduce parallel implementations.
- Do not infer a performance win from a preferred syntax or data structure alone. Use the target runtime and workload, and distinguish correctness requirements from optional optimizations.
- Preserve the user's analysis/review/implementation mode and requested scope. Report relevant assumptions, verification actually performed and material limitations proportionally to the task.
