# Modules, providers and injection scope

- Inspect imports/providers/exports and token identity at the failing consumer. Registering one class independently in several modules can create several instances; importing its owning module is normally the intended sharing model.
- Interfaces and type-only imports cannot be runtime injection tokens. Use established class/symbol/string tokens with matching provider registration. For an alias to an existing provider, consider useExisting rather than constructing a duplicate via useClass.
- Prefer constructor injection where consistent with the project. Do not introduce a container on top of Nest's container.
- DEFAULT is shared within the relevant application/module provider context. REQUEST propagates to consumers that depend on it, not automatically to every dependency below it. TRANSIENT gives a dedicated instance per consumer; it is not universally per method call.
- Singleton does not mean safe for mutable request/user/tenant data. Async requests interleave. Keep such state request-local through established context mechanisms.
- Check transport and installed Nest version before scope claims about gateways, strategies and schedulers. Current documentation can introduce features unavailable in older majors.
- Trace the actual module/provider cycle before adding forwardRef. It can be a deliberate escape hatch; do not automatically create events, a shared mega-module or a new layer to remove a local cycle.

Sources: [Nest modules](https://docs.nestjs.com/modules), [custom providers](https://docs.nestjs.com/fundamentals/custom-providers), [injection scopes](https://docs.nestjs.com/fundamentals/injection-scopes), [circular dependencies](https://docs.nestjs.com/fundamentals/circular-dependency).
