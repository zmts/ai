# Evolution, idempotency and asynchronous APIs

- Inventory deployed consumers, generated clients and documented promises before changing a field. Removing/renaming fields, changing units/nullability/ordering/defaults/errors or tightening validation can break clients.
- Additive changes are not universally safe: strict decoders, exhaustive enum handling and generated clients can break on new fields/variants. Test the actual affected consumers.
- Preserve the existing versioning/deprecation strategy. A new version is justified by a real incompatible contract, not every change; migrations need coexistence and removal criteria proportional to consumer ownership.
- For create/action retries, define idempotency key scope, atomic concurrent handling, request fingerprint, in-progress behavior, replay result and retention. Reusing a key with a different payload must follow an explicit policy.
- A timeout can leave an unknown server outcome. Give clients a reconciliation/status path for consequential operations instead of recommending blind replay.
- Event/queue APIs need message identity/version, delivery/ordering scope, duplicate handling and failure recovery. An HTTP ack may confirm acceptance rather than durable business completion; state which boundary is acknowledged.
- WebSocket/SSE reconnect behavior requires a defined replay/resume/gap policy if missed updates matter. Preserve the selected transport; do not replace it with GraphQL subscriptions by default.
- For GraphQL, inspect schema nullability and error propagation, pagination and authorization at the relevant resolvers. Client field selection does not automatically remove all overfetching/N+1/resource-cost problems.
- Validate contract examples and changed runtime behavior separately. Prefer existing OpenAPI/GraphQL/consumer tests; no mandatory new schema tooling for this skill.
