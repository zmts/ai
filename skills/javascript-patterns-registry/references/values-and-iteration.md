# Values, identity and iteration

- || defaults on all falsy values; ?? only on null/undefined. Destructuring defaults apply to undefined, not null. Preserve valid zero, false and empty strings.
- Spread copies one level of enumerable own properties, not prototypes or a complete graph. JSON round-tripping is not a generic clone. structuredClone has supported-type/runtime limits and does not replace domain serialization.
- Arrow functions capture lexical this; normal functions receive it through the call site/API. Detached methods may lose their receiver. Timer receivers are host-specific, not always undefined.
- Array sort mutates. Use a supported non-mutating method or shallow array copy when ownership requires it; element identities remain shared.
- Prefer a loop over a reduce that repeatedly copies the entire accumulator. Measure before memoization and define invalidation/retention limits.
- Generators provide lazy sequences. Early break calls return when present; use try/finally for owned cleanup in the iterator.
- Preserve numeric/serialization contracts: number is not exact decimal money and bigint cannot be directly serialized by JSON.stringify. Trace consumers before changing representation.

Source: [MDN JavaScript reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference).
