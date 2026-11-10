# Async actions, subscriptions and isolation

- Store async actions need the same stale-response protection as components. Use a request generation/scope or cooperative cancellation that invalidates late success, errors and loading updates. Do not retain an AbortController in serialized public state without checking hydration/plugin behavior.
- $subscribe/$onAction have ownership and disposal behavior. Retain unsubscribe functions for manually managed/detached subscriptions. Avoid duplicate registrations after navigation/HMR; preserve existing HMR integration.
- Persistence is an application contract: choose allowed fields, schema/version, expiry and logout cleanup. Browser storage is untrusted input; do not blindly hydrate privileged fields or secrets. Storage writes can fail and SSR has no localStorage.
- Outside component setup, call useStore after Pinia installation or supply the correct pinia instance. Module-scope store creation can select the wrong app or run before initialization.
- SSR requires a fresh app/Pinia per request and hydration of the correct request state. A process-global singleton can leak user data. Escape/serialize hydration payloads safely; do not assume JSON.stringify alone makes inline HTML safe.
- Mocked/stubbed actions test consumers, not the real action's behavior. Test real state transitions/reset and the relevant persistence/SSR boundary separately.

Sources: [Outside components](https://pinia.vuejs.org/core-concepts/outside-component-usage.html), [SSR](https://pinia.vuejs.org/ssr/), [subscriptions](https://pinia.vuejs.org/core-concepts/state.html#subscribing-to-the-state).
