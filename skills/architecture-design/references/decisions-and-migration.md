# Decisions, migration and evidence

- State the concrete requirement and known constraints: compatibility, latency/volume evidence, durability, operations, ownership, cost and timeline. Mark assumptions, especially about traffic and team size.
- Compare the current design, a local improvement and a larger change only when meaningful. Do not pad the response with irrelevant alternative stacks.
- Explain failure modes and the cost paid for each benefit. Prefer reversible steps where uncertainty is high, but do not call destructive data conversion reversible without a recovery strategy.
- For a significant decision, an optional compact ADR can contain context, decision, alternatives, consequences and verification/revisit criteria. Follow existing document locations/format. Do not create ADRs automatically for minor edits.
- Use a small Mermaid diagram if actors, stores and edges clarify ownership or sequence. Label synchronous calls versus async delivery; do not infer runtime isolation from folder names.
- Migration steps should preserve required old/new consumer compatibility. Identify data backfill, dual-write/read risks, cutover criteria, rollback limits and cleanup only where relevant.
- Validate the contract/invariant, not the diagram. Suitable evidence can be call-site inspection, a regression test, integration/fault-injection check or measured workload comparison.
- Planning approval does not authorize unrelated deployments, issue publication, broad refactors or new dependencies. Preserve existing user authorization and avoid extra generic approval gates.
