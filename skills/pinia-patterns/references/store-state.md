# Store state and ownership

- A store is reactive; destructuring primitive state/getters loses its live connection. Use storeToRefs for state/getters when needed. Actions can be destructured because Pinia binds them; storeToRefs is not an action extractor.
- Preserve mutation conventions. Pinia permits direct writes and $patch; forcing every assignment through a new action is not intrinsically safer. Actions are useful for domain transitions or coordinated operations.
- Expose all state that belongs to a Setup Store so SSR/devtools/plugins can observe it. Do not hide private reactive store state that those mechanisms depend on.
- Option Stores have a built-in $reset based on the state factory. Setup Stores need an explicit reset for owned refs; restoring only one field can leave stale selection/error/loading state.
- Resetting values does not cancel an in-flight action. Invalidate/abort prior requests on logout/account change so late results cannot restore the previous user's data.
- $dispose stops the store scope/removes its registration; it is not a complete state reset or a persisted-storage purge. Handle those explicitly when required by the session contract.
- Keep ephemeral form inputs local unless sharing/retention requires a store. URL query can own shareable filters; server data needs an explicit freshness/invalidation strategy, not necessarily a new library.
- Cache keys and retained data must include user/tenant/query scope where relevant. Avoid duplicating the same canonical state in component, store and URL with competing writers.

Sources: [Pinia core concepts](https://pinia.vuejs.org/core-concepts/), [state and reset](https://pinia.vuejs.org/core-concepts/state.html).
