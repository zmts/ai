---
name: architecture-design
description: Design or review substantive system/module boundaries, dependency direction, data ownership, consistency, failure recovery, or architectural pattern choices. Use for explicit architecture work or structural trade-offs; routine fixes, renames and changes already covered by project conventions do not need architectural review.
metadata:
  version: "1.0.0"
---

# Architecture Design

Choose the smallest structure that meets the actual constraints. Keeping the current design is a valid outcome.

## Workflow

1. Establish the requested outcome and scope. For existing systems, trace actual callers, state/data owners, dependencies, deployment units and failure paths; distinguish observations from inferred intent.
2. Identify the concrete pressure: correctness, change coupling, operational load, independent deployment, recoverability or a measured bottleneck. Ask only about unknowns that materially affect the choice.
3. Compare retaining/improving the current design with realistic alternatives using the relevant reference below. State benefits, costs and evidence; do not invent traffic/team/SLA assumptions as facts.
4. Recommend a coherent option with contracts/invariants, migration steps and validation proportional to risk. Diagrams and ADRs are useful when they clarify a significant decision, not mandatory output files.
5. Implement only when authorized. Planning/review alone remains read-only; prior implementation approval does not need another generic approval menu.

## Boundaries

- api-design owns external API contracts. nodejs-pro/nestjs-patterns/postgresql-patterns and language skills own implementation details when needed and available. Do not load all skills for a local change.
- A local correction that follows established boundaries should proceed through the appropriate implementation skill; no mandatory architecture report, refactor or line-count threshold.
- Preserve project vocabulary, infrastructure and conventions unless a concrete problem warrants a change. No compulsory layers, repository interfaces, DDD, event bus, CQRS or microservices.
- A pattern must resolve an observed need and justify its operational/maintenance cost. A simple function, conditional or direct call is often sufficient.
- Prefer evidence about responsibilities and change coupling over file size/count alone. Moving code between files does not establish a better boundary.

## References by decision

| Decision | Reference |
|---|---|
| Existing architecture review, module interfaces and refactor scope | [Boundaries and review](references/boundaries-and-review.md) |
| Code/component pattern selection and simpler alternatives | [Pattern selection](references/pattern-selection.md) |
| Outbox, idempotency, failure recovery and distributed boundaries | [Consistency and reliability](references/consistency-and-reliability.md) |
| Alternatives, migration and decision records | [Decisions and migration](references/decisions-and-migration.md) |
| Pinned sources and local adaptations | [Sources](references/sources.md) |

Lead with the recommendation and its reason. Separate verified facts, assumptions and unresolved questions. Do not claim performance/reliability improvements without suitable evidence.
