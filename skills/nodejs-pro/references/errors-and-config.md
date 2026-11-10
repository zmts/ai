# Errors, logging and configuration

- Preserve the existing public error contract: distinguish invalid input, unauthenticated, forbidden, missing resources, conflict and unexpected failure. Check installed framework propagation rules; an Express 4 async handler and Express 5 do not have identical rejection handling.
- Translate errors where transport/domain semantics are known. Unknown exceptions should not expose stacks, SQL, tokens or credentials. Retain internal cause/context for diagnosis without logging secrets.
- Avoid catching only to return null/empty or logging the same failure repeatedly at each layer. Background callbacks need an explicit rejection owner.
- Use the established structured logger, correlation and redaction policy. Do not log full request bodies or authorization/cookie headers by default.
- Parse and validate configuration once in the existing startup boundary. Distinguish missing/empty/zero and bound numeric values. Do not use Boolean('false') to parse a flag or parseInt to accept partial numeric strings accidentally.
- A TypeScript config type does not validate process.env. Startup failures should identify the setting without printing its secret value. Reject invalid configuration before accepting traffic or jobs.
- Keep application config separate from resource ownership: failed partial startup must release resources already opened. Do not implement a new config framework for a small CLI.

Illustrative strict TCP port parser, not a replacement for an existing config layer. Port 0 is intentionally excluded here; allow it separately for ephemeral test listeners.

```javascript
function parsePort(raw) {
  if (typeof raw !== 'string' || !/^[0-9]+$/.test(raw)) {
    throw new Error('PORT must contain decimal digits');
  }
  const port = Number(raw);
  if (!Number.isInteger(port) || port < 1 || port > 65535) {
    throw new Error('PORT must be between 1 and 65535');
  }
  return port;
}
```

Sources: [Node environment variables](https://nodejs.org/api/environment_variables.html), [Node errors](https://nodejs.org/api/errors.html).
