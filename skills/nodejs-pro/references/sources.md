# Migration scope

Updated 2026-10-20 under the approved JavaScript/TypeScript split. This is a local revision; the prior skill had no recorded upstream origin.

- Moved language-level async/module/value guidance into a dedicated language skill; it is now consolidated in [javascript-patterns-registry](../../javascript-patterns-registry/SKILL.md#language-diagnosis-references) as of 2026-11-12.
- Moved narrowing, inference, type-only imports and static/runtime contract guidance into typescript-patterns.
- Retained measured-unit naming, public module boundaries, scoped changes and meaningful error handling.
- Replaced frontend-wide activation with Node service/runtime scope. Resolve API disagreement by inspecting the contract and both sides.
- Removed repeated quality checklists and fixed validation counts; clarified resource ownership without adding frameworks.

The split does not require all three skills for every task. Local versions identify these adaptations, not upstream releases.

## Backend expansion — 2026-10-20

Local version 2.1.0 supplements the language split with task-specific backend references.

- Upstream: [nodejs-backend-patterns](https://github.com/wshobson/agents/tree/be57c0b2e3c05c528ca6132b87410b385718775f/plugins/javascript-typescript/skills/nodejs-backend-patterns).
- Pinned commit: `be57c0b2e3c05c528ca6132b87410b385718775f`. Inspected SKILL.md, references/details.md and references/advanced-patterns.md.
- MIT notice retained in [LICENSE.txt](../LICENSE.txt). Detailed local guidance and examples were rewritten rather than installing Express/Fastify templates.
- Primary sources: Node HTTP/process/errors/environment documentation, Hono middleware/validation/Node adapter, node-postgres pooling/transactions and BullMQ shutdown/idempotency. Links are placed with the relevant guidance. Check installed versions.
- Adaptations: no mandatory DI container, HTTP framework, logger, schema library, CORS policy or test suite. Corrected unsafe dynamic SQL identifier interpolation, transaction resource ownership, pre-insert uniqueness assumptions and unbounded shutdown assumptions.
- Hono guidance was included after verifying a Hono dependency and entrypoint in the user's backend; it is conditional and does not redefine Node.js as Hono-only. NestJS remains a separate skill at the user's explicit request.

Compare future upstream changes against this revision and preserve local corrections, scope and authorization. SQL optimization and broad architecture design remain separate tasks.

Validation: strict port-parser and transaction examples executed with success/failure/release checks using a fake pool; Hono middleware/error unwinding checked against the installed project package without opening a listener. No production database, queue or server was contacted. Deployment shutdown remains guidance, not a production drain test.
