# Boundaries and architecture review

- Follow one representative execution path before drawing system boxes. Record entrypoints, input/output contracts, data writes, transaction scope, external calls, background work and deployment/resource owners.
- Find where business invariants live and who may mutate state. Shared tables, direct internal imports and hidden globals can invalidate a diagram claiming isolated services.
- Prefer cohesive responsibilities and a useful public interface over mechanical controller/service/repository splitting. A wrapper with the same complexity as its dependency may add no useful boundary; an adapter can still be justified by a real external contract.
- Evaluate how a concrete change propagates: callers changed, knowledge duplicated, tests required and operational dependencies introduced. Separate necessary domain coupling from accidental implementation coupling.
- Move volatile integration details behind a narrow boundary when it reduces real change cost. Do not promise that swapping an entire database/framework is effortless merely because an interface exists.
- Domain/persistence/transport models need separation where semantics differ, not three copies by convention. A transaction or invariant can span several methods; do not split it blindly by file ownership.
- Test through observable public contracts where practical. Pure unit tests do not replace database/queue integration tests when correctness depends on their behavior.
- For requested review, report only evidenced problems with concrete impact and verified file/line links. Label uncertain hypotheses and avoid rewriting the system to match a favorite style.
- Suggest a bounded refactor candidate with the affected callers, preserved behavior, expected benefit and verification. Do not turn a local task into a whole-repository cleanup.
