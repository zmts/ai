# Collections, ordering and pagination

- Establish filter semantics, allowed fields/operators, maximum page sizes, stable ordering and empty-result representation. Parameterize values and validate sort/filter identifiers.
- Offset pagination is reasonable for bounded/small collections or random page access. Keyset pagination helps deep sequential traversal, but requires a stable total ordering and suitable query/index design.
- For descending (created_at, id), continuation compares the same key tuple in the same direction. Non-unique timestamps alone can skip/duplicate rows. NULL keys and mixed sort directions require explicit treatment.
- A cursor should identify the relevant sort position and query context; base64 is encoding, not tamper protection or authorization. Validate opaque cursor input and bind scope appropriately; use integrity protection if needed by the contract.
- Concurrent inserts/deletes/updates affect traversal. Cursor pagination alone does not guarantee a snapshot. Specify snapshot/watermark or best-effort semantics only when required; do not imply perfect stability under mutable sort keys.
- Totals can be expensive or inconsistent with a separate page query. Define whether total is exact/estimated/omitted and under which consistency guarantee.
- Use half-open time ranges with explicit timezone/units and deterministic boundaries. Avoid silently converting a local calendar day to a fixed 24-hour UTC interval.
- Batch endpoints need bounded input and explicit per-item versus atomic failure semantics. A successful HTTP response is not evidence that every item succeeded unless that is the contract.
