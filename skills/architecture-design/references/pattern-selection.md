# Pattern selection by actual pressure

These are decision aids, not a checklist to apply in every task.

| Pressure | Candidate | Simpler alternative / reason to skip |
|---|---|---|
| Real interchangeable behavior with recurring branching | Strategy via functions or objects | Keep a small clear conditional when variation is limited |
| Third-party interface mismatches local needs | Adapter | Direct call if it already matches and isolation adds no benefit |
| State transitions have invalid combinations or complex rules | Explicit state model/state machine | Union plus a small transition function before a class hierarchy |
| Dependencies hidden in construction or shared globals | Explicit dependency injection | Function parameters or existing framework DI before a new container |
| Real interchangeable construction/lifecycle | Factory | Direct construction when configuration is simple |
| Callers coordinate many subsystem operations | Facade | Keep existing API if the facade would merely rename each operation |
| Persistence details obscure domain operations | Repository/data mapper | Existing ORM API when it already provides the needed boundary |
| Reused nontrivial domain value invariants | Value object | Validated primitive when wrapping adds ceremony |
| Several independently useful behaviors decorate one contract | Decorator/composition | One explicit call chain when combinations are not needed |
| One deployment needs stronger internal ownership | Modular monolith | Improve existing modules without introducing service boundaries |
| Domain must stay separate from several delivery/integration mechanisms | Ports/adapters | A local adapter before full hexagonal architecture |
| Different clients require substantially different backend contracts | BFF | Local client mapping or an existing endpoint when adequate |

For caching, specify ownership, key scope, invalidation, staleness, memory limits and concurrent misses before adding memoization/cache-aside. A cache is not a fix for an incorrect data model.

For observers/events, decide whether delivery is in-process best effort or durable and whether ordering/backpressure/failure isolation matters. An event emitter does not make a cross-process workflow reliable.

Microservices require an actual need for independent ownership/deployment/scaling that outweighs network, consistency and operations costs. CQRS need not mean separate services or event sourcing. Event sourcing needs a deliberate event evolution/replay model, not just an audit-log requirement.

Use existing project conventions unless changing them addresses the stated pressure. No arbitrary maximum pattern count, mandatory new architecture or speculative future scale.
