---
name: nestjs-patterns
description: Implement or diagnose NestJS modules, providers and injection scopes, guards/pipes/interceptors/filters, DTO validation, application hooks, and Nest testing. Use for actual NestJS projects or explicit NestJS requests; Hono/Express/Fastify without Nest and generic language issues do not need this skill.
metadata:
  version: "1.0.0"
---

# NestJS Patterns

Preserve the project's Nest architecture while correcting the specific framework contract.

## Workflow

1. Inspect installed @nestjs versions, platform adapter, compiler/decorator configuration, bootstrap and affected module graph. Current online docs may describe a newer major.
2. Trace provider registration/scope and the request/job lifecycle before changing code. Identify whether failure is DI, validation, authorization, transport mapping or resource ownership.
3. Read only the relevant reference below. Prefer native mechanisms already used by the project; no automatic migration to a new ORM, validator, event bus or architecture.
4. Validate the changed provider/route and relevant failure case with existing tests. Reproduce production pipes/guards/interceptors in integration tests when behavior depends on them.

Use nodejs-pro for separate shared resource/transaction/worker concerns, [javascript-patterns-registry](../javascript-patterns-registry/SKILL.md#language-diagnosis-references) for language semantics and typescript-patterns for static type issues, if available. Do not load them all for every Nest edit.

## Decision rules

- Import a provider's owning module and exported token instead of duplicating its registration accidentally.
- A TypeScript interface is erased; use an existing class or explicit runtime token for injection.
- Keep request/user state out of shared singleton fields. Request scope can propagate to consumers; changing it can affect lifecycle and cost.
- DTO types and Swagger decorators alone do not validate runtime input. Preserve existing validators and explicit conversion semantics.
- Guards authorize; pipes validate/transform; interceptors wrap execution; filters map exceptions. Keep domain errors independent of HTTP where the current architecture requires it.
- Reuse application hooks. Do not add parallel signal handlers or assume socket timeouts bound shutdown.
- Respect user scope and repository conventions. Avoid mandatory repositories/events/CQRS, blanket global pipe changes or unrelated strictness upgrades.

## References by concern

When a concrete pattern choice needs examples, use [javascript-patterns-registry](../javascript-patterns-registry/SKILL.md). Read only the linked entry relevant to the task; check its applicability and simpler alternative before introducing a pattern. Apply these general examples through the existing Nest provider scopes, injection tokens and lifecycle mechanisms.

| Concern | Reference |
|---|---|
| Module exports, injection tokens, scope and circular dependencies | [Modules and DI](references/modules-and-di.md) |
| Request order, DTO validation, authorization and exception mapping | [Request lifecycle](references/request-lifecycle.md) |
| Configuration, shutdown hooks and testing | [Lifecycle and tests](references/lifecycle-and-tests.md) |
| Shared provider state versus per-request context examples | [Singleton](../javascript-patterns-registry/references/js-patterns-registry.md#singleton), [Context](../javascript-patterns-registry/references/js-patterns-registry.md#context) |
| Explicit dependencies and delegation examples | [IoC and DI](../javascript-patterns-registry/references/js-patterns-registry.md#explicit-dependencies--ioc-and-di), [Composition over inheritance](../javascript-patterns-registry/references/js-patterns-registry.md#composition-over-inheritance) |
| Pinned upstream and local corrections | [Sources](references/sources.md) |

Report the corrected Nest behavior, checks actually run and any environment/version limits. Analysis-only requests stay read-only.
