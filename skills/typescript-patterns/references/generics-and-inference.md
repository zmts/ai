# Generics and inference

Express a real input/output relationship; prefer inference to decorative parameters.

```typescript
function getProperty<T extends object, K extends keyof T>(input: T, key: K): T[K] {
  return input[key];
}

const user = { id: 'u1', active: true };
const id = getProperty(user, 'id');
const exactId: string = id;
// @ts-expect-error Unknown keys are rejected.
getProperty(user, 'missing');
// @ts-expect-error Selected value is string, not boolean.
const wrongId: boolean = id;
```

- A generic appearing only in the return type can let callers claim an unvalidated type.
- Use Pick/Omit/Partial/Record/ReturnType/Awaited before inventing equivalents. Partial is shallow and may weaken update invariants.
- Conditional types distribute over a naked type parameter. Tuple wrapping suppresses distribution when that is the intended relationship.
- Mapped/template literal types help with finite keys. Recursive property paths require handling arrays, functions, optional fields and compiler limits; avoid them without a real need.
- satisfies checks assignability, not runtime data; it does not preserve literals universally. as const deliberately narrows literal values and marks literal properties readonly.
- Brands can distinguish IDs, but conversion must enforce domain invariants. Do not brand every primitive.

## Type tests

Use project tooling or a small compiler fixture with positive assignment/call cases and @ts-expect-error negative cases. The directive fails if the next line stops producing an error; keep it minimal so unrelated errors cannot satisfy it. Avoid home-grown universal equality helpers: any/never/unions have subtle behavior.

Sources: [Generics](https://www.typescriptlang.org/docs/handbook/2/generics.html), [Conditional types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html), [Mapped types](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html).
