# Request lifecycle and validation

- Trace middleware, guards, interceptors, pipes, handler/service and exception filters in the real registration scope. Interceptors wrap execution and unwind in reverse order; errors can skip later stages. Use official lifecycle documentation for parameter-pipe ordering and scoped filter behavior.
- Use guards for principal/resource authorization. A valid JWT does not prove permission to the requested record. Never trust a tenant/user identifier supplied by the body over the authenticated context.
- Reuse the existing validation approach: ValidationPipe with class-validator or established schema pipes. Type annotations/interfaces and Swagger metadata do not validate input. DTO classes needed by runtime reflection must remain value imports.
- Inspect whitelist/forbidNonWhitelisted/transform/skip-missing settings before editing global pipes. Altering them changes every route's contract. Nested DTOs need the corresponding runtime transformation/validation configuration.
- Avoid blanket implicit conversion: strings such as 'false' can become truthy under generic Boolean conversion. Define accepted boolean/number syntax and bounds deliberately; route parameter parsers must match API semantics.
- Keep domain errors transport-independent if services are shared across HTTP/jobs/messages. Use framework HTTP exceptions/filters at an appropriate boundary; preserve public envelope/status and internal cause without leaking secrets.
- Native @Res handling and passthrough change response ownership. Verify serialization/interceptor behavior before mixing manual response writes and returned objects. Express and Fastify adapter objects are not interchangeable.
- An Observable returned by an interceptor is not a Promise. Preserve completion/error/cancellation semantics and verify RxJS behavior when changing reactive flows.

Sources: [Request lifecycle](https://docs.nestjs.com/faq/request-lifecycle), [validation](https://docs.nestjs.com/techniques/validation), [interceptors](https://docs.nestjs.com/interceptors), [filters](https://docs.nestjs.com/exception-filters).
