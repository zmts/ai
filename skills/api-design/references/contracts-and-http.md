# Contracts and HTTP semantics

- Define required versus optional versus nullable fields, omission/default behavior, exact numeric representation, units, timestamps/timezones and accepted ranges. Missing PATCH fields and explicit null need not mean the same thing.
- Schema validation is not business validation or authorization. Identify who may request each action on each resource; a valid identifier/token is insufficient by itself.
- Keep request DTOs and response representations distinct where meanings differ. Returning an ORM row can expose credentials/internal fields or silently couple API evolution to storage.
- GET/HEAD are safe by intended semantics; GET request bodies lack generally defined semantics and may be rejected by intermediaries. PUT/DELETE are idempotent by intended effect; POST can support application idempotency. PATCH is not automatically idempotent. A repeated DELETE may return a different status while remaining idempotent.
- Preserve project status/error conventions while respecting protocol requirements. For 401 responses, consider the required authentication challenge; distinguish 403, 404 information-hiding choices, 409 conflicts and input-validation errors according to the actual contract.
- 202 means accepted, not completed: provide a defined way to observe eventual success/failure where needed. 204 has no response content. An error after streaming starts cannot simply become a fresh JSON error response.
- Problem Details (RFC 9457) is an available standard, not a mandatory global migration. If used, keep HTTP status and problem fields consistent; do not expose internal stacks/SQL/secrets in detail.
- Conditional requests/ETags can protect against lost updates when the server atomically enforces the condition. An ETag alone without write validation does not provide concurrency control.
- Preserve content type, caching and authorization semantics. Do not cache user-specific responses under a shared key without the relevant isolation policy.

Sources: [HTTP semantics RFC 9110](https://www.rfc-editor.org/rfc/rfc9110.html), [PATCH RFC 5789](https://www.rfc-editor.org/rfc/rfc5789.html), [Problem Details RFC 9457](https://www.rfc-editor.org/rfc/rfc9457.html).
