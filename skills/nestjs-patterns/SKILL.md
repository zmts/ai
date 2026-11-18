---
name: nestjs-patterns
description: Design, implement, diagnose, or review NestJS modules, dependency injection, request handling, transports, and lifecycle. Use when a NestJS feature, integration, or bug depends on the module graph, request or message flow, testing setup, or the boundaries of a use case.
metadata:
  version: "1.2.0"
---

# NestJS Patterns

1. Check the project's lockfile, adapter, bootstrap, and registration scopes. Trace the real entry point and failure path. The registry uses NestJS 12.5.1 as its baseline.
2. Select the relevant heading in the [full registry](references/nestjs-patterns-registry.md#table-of-contents). Resolve the file relative to this `SKILL.md`. To read selectively, locate headings with `rg -n '^#{2,4} '` and read only the applicable section or card.
3. Choose the smallest Nest mechanism that owns the behavior. Verify it through the real entry point and report material version or adapter constraints.

| Topic | Registry sections |
| --- | --- |
| HTTP lifecycle and routing | [Request map and registration order](references/nestjs-patterns-registry.md#http-request-map) · [Adapters, parsing, and routing](references/nestjs-patterns-registry.md#ingress-and-routing) |
| Modules, DI, and bootstrap | [Application composition](references/nestjs-patterns-registry.md#application-design-and-composition) |
| Request enhancers | [Middleware](references/nestjs-patterns-registry.md#middleware) · [Guards](references/nestjs-patterns-registry.md#guards) · [Interceptors](references/nestjs-patterns-registry.md#interceptors) · [Arguments, DTOs, and pipes](references/nestjs-patterns-registry.md#arguments-and-pipes) |
| Response and errors | [Serialization and response ownership](references/nestjs-patterns-registry.md#successful-response) · [Exception boundaries and filters](references/nestjs-patterns-registry.md#errors-and-exception-filters) |
| Use cases, data, and side effects | [Business layer, transactions, and outbox](references/nestjs-patterns-registry.md#handler-and-business-layer) |
| GraphQL, WebSocket, RPC, jobs, and scheduling | [Transport differences and messaging](references/nestjs-patterns-registry.md#transport-variants) |
| Tests, startup, and shutdown | [Testing and operations](references/nestjs-patterns-registry.md#testing-and-operations) |
| Worked example and source lookup | [End-to-end endpoint](references/nestjs-patterns-registry.md#end-to-end-example) · [Topical source index](references/nestjs-patterns-registry.md#sources-and-topical-index) |
