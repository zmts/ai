# Boundaries and narrowing

Assign JSON.parse/response.json results to unknown before validation. An assertion only changes compiler beliefs. Reuse existing validation tools rather than adding a library for a small boundary.

```typescript
type User = { id: string; active: boolean };

function isUser(input: unknown): input is User {
  return typeof input === 'object'
    && input !== null
    && 'id' in input
    && typeof input.id === 'string'
    && 'active' in input
    && typeof input.active === 'boolean';
}

function parseUser(json: string): User {
  const input: unknown = JSON.parse(json);
  if (!isUser(input)) throw new Error('Invalid user payload');
  return input;
}
```

This guard accepts extra/inherited properties; require own properties or schema restrictions where the boundary needs them. Domain constraints such as non-empty IDs need additional checks. Map errors through existing project policy.

```typescript
type LoadState<T> =
  | { status: 'loading' }
  | { status: 'ready'; data: T }
  | { status: 'failed'; error: Error };

function readState(state: LoadState<string>): string {
  switch (state.status) {
    case 'loading': return 'Loading';
    case 'ready': return state.data;
    case 'failed': throw state.error;
    default: {
      const unreachable: never = state;
      throw new Error(`Unexpected state: ${String(unreachable)}`);
    }
  }
}
```

- Truthiness does not mean non-null when zero/false/empty strings are valid.
- A predicate annotation is trusted, not proven; test malformed inputs.
- Prefer discriminants over optional fields shared by several variants.
- readonly restricts static access; it is not Object.freeze or runtime deep immutability.
- Avoid non-null assertions across await boundaries; capture stable values or recheck mutable state.

Source: [TypeScript narrowing](https://www.typescriptlang.org/docs/handbook/2/narrowing.html).
