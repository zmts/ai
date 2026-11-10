# Bootstrap, lifecycle and testing

- Use the existing config module or startup validation; reject invalid config before listening. ConfigService generics do not themselves validate environment variables.
- Async provider factories and initialization hooks can gate readiness. Handle failed partial startup and release resources already opened.
- app.close triggers application cleanup; signal-driven shutdown hooks require the appropriate enableShutdownHooks setup. Do not layer duplicate custom handlers over it without a single coordinated owner.
- Determine which hook runs before HTTP closure and which resources still serve in-flight work. Do not end DB/Redis prematurely. A server.setTimeout call controls socket behavior, not an application shutdown deadline.
- Request-scoped providers do not have the same application-lifecycle hook behavior as singleton providers. Check installed version and owner before putting global cleanup there.
- Reuse existing TestingModule and test runner. Override the actual token and compile the relevant module graph; avoid mocks that bypass the failing import/export contract.
- HTTP integration fixtures should include production global pipes/filters/guards when relied upon. Tests with a new empty app do not verify bootstrap policy automatically. Initialize and close the fixture; use Fastify readiness/injection when appropriate to the adapter.
- Choose focused tests for the changed behavior: missing/forbidden principal, malformed DTO, shared-provider identity or cleanup. No blanket test-per-service policy, fixed coverage quota or framework installation in the application just for this skill.
- Use task-owned external-resource substitutes unless a real integration environment is explicitly intended. Unit mocks cannot prove real transaction/queue semantics.

Sources: [Lifecycle events](https://docs.nestjs.com/fundamentals/lifecycle-events), [testing](https://docs.nestjs.com/fundamentals/testing), [configuration](https://docs.nestjs.com/techniques/configuration).
