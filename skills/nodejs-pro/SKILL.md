---
name: nodejs-pro
description: Diagnose Node.js backend services, server APIs, process lifecycle, IO, resource ownership, concurrency, or backend integration contracts. Use for Node-specific runtime and service behavior; pure JavaScript semantics, TypeScript inference, Vue reactivity, and ordinary frontend changes have more specific skills.
metadata:
  version: "2.1.0"
---

# Node.js Pro

Resolve Node.js runtime and service problems within the existing architecture.

## Working mode

1. Establish the actual Node version, package manager, module format, startup command and deployment path from project files.
2. Trace the request/job through state, external dependencies and response. Identify the concrete cause before implementing.
3. Apply the smallest coherent fix. Verify the changed path, a relevant failure path and integration behavior where practical.
4. Report behavior changes, checks actually performed and material verification limits.

Use javascript-patterns for substantive promise/module/value issues and typescript-patterns for substantive type-system issues, if available. Load only skills warranted by the problem; routine backend edits do not need all three. Framework-specific frontend behavior belongs with its framework skill.

For NestJS-specific modules, DI or request lifecycle, prefer nestjs-patterns if available. Read the references below only for the affected backend concern; generic service guidance does not require a separate framework or architecture migration.

## References by task

| Concern | Reference |
|---|---|
| Request handling, middleware order, auth and response ownership | [HTTP and middleware](references/http-and-middleware.md) |
| Error mapping, logs, configuration and startup failures | [Errors and configuration](references/errors-and-config.md) |
| Connection pools, transaction ownership and release | [Database lifecycle](references/database-lifecycle.md) |
| Workers, retries, cancellation and delivery semantics | [Jobs and workers](references/jobs-and-workers.md) |
| Readiness, draining, signals and ordered cleanup | [Shutdown](references/shutdown.md) |
| Pinned sources and adaptation decisions | [Sources](references/sources.md) |

## Service and resource correctness

- Align handlers, domain flow and persistence with established project roles. Add a layer only when it resolves real coupling or variation.
- Validate external inputs and configuration at the existing boundary. Do not add a configuration framework for a small script.
- Trace ownership of sockets, timers, streams, subscriptions, database clients and transactions. Release owned resources on success, failure and cancellation.
- For shutdown changes, stop admitting work, bound draining, close owned resources and preserve appropriate exit status. Reuse framework lifecycle hooks.
- Bound concurrency to downstream capacity. Respect stream backpressure and propagate stream failures; async syntax does not make CPU work non-blocking.
- Retry identified transient failures within a bounded policy. Writes/jobs need established idempotency before retrying.
- Preserve transaction boundaries and shared-state ownership; parallel promises do not make writes atomic.
- Prefer async IO on request paths. Synchronous startup and small CLI operations can be appropriate when their blocking behavior is understood.

## Contracts and local conventions

- Compare documented API/schema, implementation and callers before choosing which side to fix. Preserve compatibility unless a contract change is authorized.
- Keep display formatting in the presentation layer when that is the agreed contract. Deliberate server aggregation/pagination can be appropriate.
- Keep precise domain names and measured units such as ttlSec, durationMs, changePct and volumeUsd. Preserve canonical fields such as open/close/high/low and offset/limit/total/count.
- Follow public module boundaries; avoid barrels that create cycles. Export only intentional module API.
- Handle, translate or propagate errors intentionally. Log useful context without secrets and avoid repeated logging of one failure at each layer.
- Preserve repository conventions and scope. Avoid speculative architecture, opportunistic cleanup, new dependencies and unrelated renames.

## Validation

Use existing scripts for typecheck, lint and relevant tests. Verify actual startup/integration behavior when affected, using task-owned resources. Do not change coverage policy, test framework or deployment configuration for a generic checklist. Analysis-only requests remain read-only; distinguish observed behavior from assumptions.

Language rules formerly bundled here are maintained in the JavaScript/TypeScript skills. See [migration scope](references/sources.md).
