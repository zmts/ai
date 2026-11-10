# Sources and local adaptation

Reviewed 2026-10-20. Local versions identify these curated instructions, not upstream releases.

- [antfu/skills — pinia](https://github.com/antfu/skills/tree/d02c48452d782231e4c32d7069cde731a4c7db42/skills/pinia), pinned commit `d02c48452d782231e4c32d7069cde731a4c7db42`. Inspected: skills/pinia/SKILL.md, skills/pinia/references/core-stores.md.

## Local changes

Preserved existing Options/Setup Store conventions. Added reset versus disposal/cancellation distinctions, session boundaries, persistence validation and request-owned SSR state. No mandatory server-state cache plugin.

Local text/examples are independently written from primary documentation and the task requirements; upstream files are not vendored. The inspected upstream skill frontmatters declare MIT. Retain upstream notices if copying substantial content in future.

The existing Vue skill had no recorded upstream provenance; its local revision is not claimed to be an upstream upgrade. Public doc links can target newer majors: check installed versions and compiler/runtime alignment.

Update by comparing pinned sources/references, preserving local scope, existing conventions and authorization. Do not overwrite automatically based on a newer date.

## Verification

Executable checks on the installed project libraries verified storeToRefs, a destructured bound action, Option Store reset and explicit Setup Store reset. Two independent SSR apps rendered different request-owned stores correctly. Persistence plugins and full browser hydration were not tested. Structural validation, UI metadata and local links also passed. Tests ran from a task-owned temporary directory and did not change application source or install application dependencies.
