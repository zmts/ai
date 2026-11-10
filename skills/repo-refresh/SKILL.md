---
name: repo-refresh
description: Use when the user says they changed repository files outside Codex, especially from an IDE, and wants Codex to refresh context, resync with the repo, reread changed files, or understand what changed before continuing work.
---

# Repo Refresh

Use this skill when the user indicates that repository state changed outside the current Codex flow and wants the assistant to update its working context before answering or continuing implementation.

Common triggers:
- "я поменял в IDE, обнови контекст"
- "обнови контекст репы"
- "перечитай изменения"
- "синхронизируйся с репозиторием"
- "refresh repo context"

## Workflow

1. Confirm the current repository root with `pwd` and, when needed, `git rev-parse --show-toplevel`.
2. Inspect the working tree without mutating it:
   - `git status --short`
   - `git diff --name-only`
   - `git diff --stat`
3. Identify files relevant to the user's current task:
   - Prefer changed files from `git diff --name-only`.
   - Include untracked files from `git status --short` when they look task-relevant.
   - Do not scan the whole repository unless changed-file context is insufficient.
4. Reread the relevant files or diffs:
   - Use `git diff -- <path>` for modified tracked files.
   - Use `sed -n`, `rg`, or another read-only command for untracked files and nearby context.
   - Use `rg` first for searches.
5. Summarize the refreshed context:
   - Which files changed.
   - What behavior, contracts, or assumptions are now different.
   - Any conflicts with the previous plan or implementation.

## Guardrails

- Do not modify files during refresh unless the user separately asks to implement or fix something.
- Do not run git write operations such as `git add`, `git commit`, `git checkout`, `git reset`, or stash commands.
- Never revert external edits unless the user explicitly asks.
- Treat external changes as user-owned work and adapt to them.
- If the refreshed state changes the task scope, stop and explain the new scope before implementing.

## Response Style

Keep the refresh report concise and concrete:

```md
Контекст обновил.

Changed files:
- path/to/file.ts: ...

What changed:
- ...

Impact:
- ...
```
