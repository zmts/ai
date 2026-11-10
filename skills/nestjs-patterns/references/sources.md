# Sources and local adaptation

Reviewed 2026-10-20. Local version 1.0.0 is not an upstream release.

- Compared [Kadajett/agent-nestjs-skills](https://github.com/Kadajett/agent-nestjs-skills/tree/9c553c7b217e6f1b657b06a717db06345c689749/skills/nestjs-best-practices), pinned commit `9c553c7b217e6f1b657b06a717db06345c689749`, declared skill version 1.2.0.
- Inspected SKILL.md and rules for circular dependencies, module sharing, injection scope, validation and graceful shutdown.
- Local instructions/examples are independently written from the task requirements and official Nest documentation linked in each reference; upstream code/rule files are not vendored. Upstream frontmatter declares MIT, but no standalone license file was present in the inspected tree.
- Corrected scope propagation direction and the distinction between socket timeouts and shutdown deadlines. Avoided duplicated shutdown handlers, mandatory implicit conversion, repositories, events, new dependencies and unsupported claims about zero downtime.
- Current official Nest documentation can target a newer major than the application. Verify installed versions, adapter and compiler setup before applying version-sensitive APIs.

For updates, compare upstream against the pinned revision and primary docs, preserving local scope and authorization. Do not replace these curated files with the entire upstream collection automatically.

Validation: structural checks passed; isolated @nestjs/core 11.2.7 fixture verified shared exported providers, rejection of missing exports, request-scope consumer identity and initialization/destruction hooks. This is not a full HTTP-adapter or multi-version compatibility evaluation.
