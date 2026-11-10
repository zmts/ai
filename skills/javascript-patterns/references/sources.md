# Sources and local adaptation

Reviewed/adapted 2026-10-20. Local version 1.0.0 is not an upstream release number.

- Upstream repository: https://github.com/wshobson/agents
- Pinned commit: `be57c0b2e3c05c528ca6132b87410b385718775f`
- Source skill and supporting references: [modern-javascript-patterns](https://github.com/wshobson/agents/tree/be57c0b2e3c05c528ca6132b87410b385718775f/plugins/javascript-typescript/skills/modern-javascript-patterns). Inspected SKILL.md and references/details.md; JavaScript advanced-patterns.md was also consulted for module/iteration guidance.
- Upstream MIT notice is retained in [LICENSE.txt](../LICENSE.txt).
- Primary documentation links are attached to the relevant local reference. Check API availability against project versions, not the current documentation's default runtime.

## Local differences

Replaced introductory syntax tutorials with task routing. Corrected timer receiver assumptions; added HTTP status handling, timer cleanup and cooperative cancellation. Preserved runtime/module compatibility and local naming rules. No mandatory ESM migration, framework, dependency or coverage threshold.

These are locally curated instructions, not an unchanged upstream installation. For updates, compare the pinned source directory and its references with upstream; preserve local scope, runtime constraints, user authorization and tested corrections. Do not overwrite automatically or infer improved behavior from a newer date alone.
