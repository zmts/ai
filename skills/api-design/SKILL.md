---
name: api-design
description: Design or review external API request/response contracts, error semantics, pagination, idempotency, compatibility and protocol evolution. Use for substantive contract decisions in HTTP, GraphQL, events or real-time APIs; routine route implementation, internal function types and styling alone do not need this skill.
metadata:
  version: "1.0.0"
---

# API Design

Design contracts around actual producers, consumers and failure behavior while preserving the project's chosen protocol and conventions.

## Workflow

1. Inspect existing schema/specification, handlers, consumers, authentication and serialization. Establish the requested behavior and which compatibility promises apply.
2. Specify input/output meaning, units, nullability, validation, authorization, ordering and relevant failure/retry semantics. Distinguish desired contract from observed implementation.
3. Read only the relevant reference below. Compare a compatible local change with alternatives; do not change protocol, global error format or versioning scheme incidentally.
4. Validate affected valid/invalid requests and existing consumers using available project tooling. A schema-only check cannot prove runtime behavior.
5. Implement/update specifications only within authorized scope. Analysis/review stays read-only; reuse existing approval rather than introducing another menu.

Use architecture-design for a genuine change to system boundaries/consistency. Framework/database/language skills own implementation detail when needed and available. Do not automatically activate all of them for an endpoint edit.

## Contract rules

- Source of truth is the agreed contract checked against both sides; neither current backend output nor UI expectations are automatically correct.
- Preserve protocol and established error/response envelopes. HTTP resource naming is a useful convention, not a ban on explicit domain actions.
- Distinguish authentication from resource/action authorization. Validate inputs and explicitly select public output fields; do not expose persistence records wholesale.
- Define timeout, retry and ambiguous outcomes where they matter. Transport idempotency does not guarantee exactly-once execution or identical responses.
- Avoid mandatory GraphQL, a new OpenAPI generator, a gateway, new major versions or broad redesign for a local change.

## References by concern

| Concern | Reference |
|---|---|
| Input/output schema, errors, HTTP methods and status | [Contracts and HTTP](references/contracts-and-http.md) |
| Pagination, filtering, deterministic ordering and time | [Collections](references/collections.md) |
| Idempotency, compatibility, async and protocol evolution | [Evolution and reliability](references/evolution-and-reliability.md) |
| Pinned sources and standards | [Sources](references/sources.md) |

Return the proposed contract/change, rationale, compatibility effect and actual validation limits. Keep examples and specification work proportional to the decision.
