# HTTP lifecycle and middleware

- Identify the installed framework, version and server adapter first. Hono is not Express; avoid copying req/res/next conventions between them. Preserve the framework used by the project.
- Trace middleware registration order, path matching, short-circuit responses and before/after-next behavior. Verify errors with the actual framework error handler: Hono can turn a downstream throw into a response before upstream middleware resumes.
- Validate parsed body/query/params with existing tooling. Type annotations do not validate input. Preserve content-type handling and raw bytes needed for webhook signatures; do not consume a streaming body twice.
- Authentication establishes a principal; authorization checks access to the specific resource/action. Include an authenticated-but-forbidden case when changing access control. CORS is not authorization, and cookie authentication may require CSRF protection.
- Exactly one layer owns the final response. Check framework-native response helpers and serialization before manually manipulating sockets/headers. After a stream starts, an error usually cannot become a new JSON response.
- Bound body sizes, upstream waits and concurrency where relevant to the task; select limits from requirements. Do not add compression, rate limits, DI or a gateway to every endpoint by default.
- Respect trusted-proxy configuration when using IP, protocol or forwarding headers. Do not trust arbitrary incoming request IDs as unsanitized log data.

## Hono on Node

- Verify app.route/basePath registration, middleware scope and adapter startup. Use await next() for middleware that needs downstream completion; preserve early-return responses.
- Use app.onError/app.notFound and existing HTTPException mapping deliberately. Check behavior of mounted apps and retained error envelopes.
- c.req.json() parses JSON; it does not establish a schema. Existing validators may depend on Content-Type; test real request headers and malformed payloads.
- app.request tests route behavior without a network listener. Use a real adapter/listener check when investigating disconnects, streaming, server timeouts or shutdown.

Sources: [Hono middleware](https://hono.dev/docs/guides/middleware), [validation](https://hono.dev/docs/guides/validation), [Node adapter](https://hono.dev/docs/getting-started/nodejs), [Node HTTP](https://nodejs.org/api/http.html). Match documentation to installed versions.
