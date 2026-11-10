# Sources and local adaptation

Reviewed 2026-10-20. Local versions identify these curated instructions, not upstream releases.

- [vuejs-ai/skills — vue-router-best-practices](https://github.com/vuejs-ai/skills/tree/bc922e485acd752954972a24dc63e3c9051b4bfd/skills/vue-router-best-practices), pinned commit `bc922e485acd752954972a24dc63e3c9051b4bfd`. Inspected: skills/vue-router-best-practices/SKILL.md.

## Local changes

Scoped routing to actual navigation issues. Added parameter reuse, guard/failure handling, canonical URL ownership and stale loading. Router 4-origin guidance is checked against the installed Router 5 version; no automatic file-routing migration.

Local text/examples are independently written from primary documentation and the task requirements; upstream files are not vendored. The inspected upstream skill frontmatters declare MIT. Retain upstream notices if copying substantial content in future.

The existing Vue skill had no recorded upstream provenance; its local revision is not claimed to be an upstream upgrade. Public doc links can target newer majors: check installed versions and compiler/runtime alignment.

Update by comparing pinned sources/references, preserving local scope, existing conventions and authorization. Do not overwrite automatically based on a newer date.

## Verification

Executable memory-history checks on the installed project libraries verified parameter updates on the same matched record and aborted navigation. They do not prove actual DOM component reuse, browser back/forward, scroll/focus, server fallback or hydration. Structural validation, UI metadata and local links also passed. Tests ran from a task-owned temporary directory and did not change application source or install application dependencies.
