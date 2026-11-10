# Consistency and failure recovery

- Define the invariant and observable failure states before choosing a mechanism. A local database transaction is often the simplest atomic boundary.
- Writing a row then publishing to a broker leaves a crash gap; publishing then writing leaves the opposite gap. Do not describe either order as atomic.
- A transactional outbox writes domain state and an event record in the same database transaction. A relay publishes committed records. A crash after publication but before marking delivery can produce duplicates: consumers need durable idempotency or another explicit duplicate strategy.
- Specify event identity, schema version, ordering scope, retries/dead-letter handling, retention and relay recovery. Deduplicate and apply a local consumer database effect in one transaction when possible. External effects require their own idempotency/reconciliation mechanism.
- Do not hold an outbox transaction open around slow broker IO without evaluating locks and throughput. Polling/CDC/queue technology is a deployment choice, not automatic infrastructure to add.
- Idempotency keys need a scope, request identity/fingerprint, atomic concurrent claim, in-progress/replay policy, retention and reconciliation of ambiguous outcomes. An in-memory map or unique job ID alone does not establish exactly-once business effects.
- Retry bounded, classified transient failures with appropriate backoff/jitter and deadlines. Retrying all errors can multiply load or duplicate effects. A timeout means the outcome may be unknown, not that the remote side did nothing.
- A saga coordinates local transactions and compensating actions; compensation may fail and is not time travel or a universal rollback. Use only for a real cross-boundary workflow.
- Circuit breakers/bulkheads isolate particular dependency failures; they do not remove the need for bounded resources, deadlines and a defined degraded response.
- Availability and consistency claims depend on failure assumptions, topology and recovery behavior. Avoid unqualified exactly-once/zero-downtime/no-data-loss claims.

Source: [AWS transactional outbox guidance](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/transactional-outbox.html). Validate product-specific guarantees against the actual installed infrastructure.
