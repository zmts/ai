# URL ownership and asynchronous navigation

- Decide whether URL, component or store owns filters/selection/page. Shareable navigation state often belongs in query/params; transient draft input need not. Avoid two watchers continuously rewriting each other.
- Normalize and compare query values before replace/push. Preserve unrelated query/hash values where intended. Use replace for non-history-worthy corrections and push for meaningful user navigation according to UX requirements.
- Define back/forward behavior, defaults, malformed values and canonical URLs. Do not erase valid zero/false values with truthy defaults.
- For route-driven loading, capture the intended key, invalidate old work and gate stale success/error/finally. A cancelled guard does not automatically abort arbitrary requests already started by application code.
- Choose fetch-before versus fetch-after navigation deliberately, including loading/error ownership. Do not leave the old resource actionable under the new route when that violates the contract.
- HTML5 history needs server fallback/deployment base configuration. Memory history requires explicit initial navigation; await readiness where initial-route-dependent rendering/tests need it.
- Browser checks are necessary for actual scroll restoration, focus, history/back-forward, navigation UI and hydration. Do not claim these from a memory-router test.
- SSR requires request-owned router/store instances and the correct initial location. Avoid cross-request state and browser-only APIs in server paths.

Sources: [Data fetching](https://router.vuejs.org/guide/advanced/data-fetching.html), [history modes](https://router.vuejs.org/guide/essentials/history-mode.html).
