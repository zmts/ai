# Sources and local adaptation

Reviewed 2026-10-20. Local versions identify these curated instructions, not upstream releases.

- [vuejs-ai/skills — vue-best-practices](https://github.com/vuejs-ai/skills/tree/691bc006619f8f5c4c5951a0c57394df883b0000/skills/vue-best-practices), pinned commit `691bc006619f8f5c4c5951a0c57394df883b0000`. Inspected: skills/vue-best-practices/SKILL.md, skills/vue-best-practices/references/reactivity.md, skills/vue-best-practices/references/composables.md.
- [vuejs-ai/skills — vue-debug-guides](https://github.com/vuejs-ai/skills/tree/691bc006619f8f5c4c5951a0c57394df883b0000/skills/vue-debug-guides), pinned commit `691bc006619f8f5c4c5951a0c57394df883b0000`. Inspected: skills/vue-debug-guides/SKILL.md.
- [antfu/skills — vue](https://github.com/antfu/skills/tree/d02c48452d782231e4c32d7069cde731a4c7db42/skills/vue), pinned commit `d02c48452d782231e4c32d7069cde731a4c7db42`. Inspected: skills/vue/SKILL.md.

## Local changes

Removed framework-wide activation, forced component splitting and mandatory style migration. Added Vue 3.5 reactive-props distinction, version gates, stale-result protection and cleanup ownership.

Local text/examples are independently written from primary documentation and the task requirements; upstream files are not vendored. The inspected upstream skill frontmatters declare MIT. Retain upstream notices if copying substantial content in future.

The existing Vue skill had no recorded upstream provenance; its local revision is not claimed to be an upstream upgrade. Public doc links can target newer majors: check installed versions and compiler/runtime alignment.

Update by comparing pinned sources/references, preserving local scope, existing conventions and authorization. Do not overwrite automatically based on a newer date.

## Verification

Executable checks on installed Vue 3.5.30 verified captured primitive versus reactive toRefs, the documented latest-result composable against stale success/error and scope disposal, and SFC compilation with reactive props destructure/defineModel. This is not a mounted-component or browser hydration test. Structural validation, UI metadata and local links also passed. Tests ran from a task-owned temporary directory and did not change application source or install application dependencies.
