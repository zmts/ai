---
name: diff
description: Use when the user asks for `diff`, `diff --ui`, `diff --api`, `diff --all`, or `diff --files` to inspect local git changes and explain them in plain language. Supports frontend/UI and API/backend scopes, with optional clickable affected files.
---

# Diff

Inspect local git changes and explain them clearly without modifying the working tree.

## Trigger

Use this skill when the user asks for:
- `diff`
- `diff --all`
- `diff --ui`
- `diff --api`
- `diff --files`
- a plain-language summary of local changes by scope

## Scope Rules

Resolve the requested scope before reading diffs.

- `--ui` targets the frontend/UI repository or frontend scope.
- `--api` targets the API/backend repository or backend scope.
- `--all` targets every relevant repository in the current workspace.
- `--files` adds a clickable affected-file list to the output.
- No key is equivalent to `--all`.

`--files` is an output modifier, not a scope. It can be combined with scope keys:
- `diff --api --files`
- `diff --ui --files`
- `diff --all --files`
- `diff --files` means all scopes with files.

Resolve scopes from the current workspace layout, repository names, local instructions, and the user's wording. Scopes may belong to separate repositories or to directories within one repository. For `--all`, include every relevant git root in the workspace.

If the selected scope cannot be inferred safely, ask one concise clarification question instead of guessing.

## Read-Only Guardrails

- Do not run `git add`, `git commit`, `git checkout`, `git reset`, `git stash`, or any write operation.
- Do not edit files.
- Do not revert user changes.
- Treat all local changes as user-owned unless the current conversation proves otherwise.

## Inspection Workflow

For each selected git root:

1. Confirm the root with `git -C <repo> rev-parse --show-toplevel` when needed.
2. Read state:
   - `git -C <repo> status --short`
   - `git -C <repo> diff --name-status`
   - `git -C <repo> diff --cached --name-status`
   - `git -C <repo> diff --stat`
   - `git -C <repo> diff --cached --stat`
3. Read targeted diffs only for changed files that need content-level understanding:
   - `git -C <repo> diff -- <path>`
   - `git -C <repo> diff --cached -- <path>`
4. Include untracked files from `status --short`; inspect their contents only when they look relevant and are reasonably small.
5. Summarize behavior, contracts, UI, docs, tests, config, and operational changes separately when the diff supports that distinction.

Prefer `rg`/`sed`/`git diff` reads over broad scans. Keep commands read-only.

## Output Format

Answer in the user's language.

Use this structure:

```md
Scope: <api | ui | all>

Изменения:
- <понятное описание изменения>
- <понятное описание изменения>

Отдельно:
- staged: <есть/нет, кратко>
- untracked: <есть/нет, кратко>
- риски/вопросы: <только если реально есть>
```

Rules:
- Group by meaningful change, not by raw file list.
- Do not include affected files unless the user provided `--files`.
- Say clearly when there are no changes in the selected scope.
- If there are many mechanical files, group them and list representative paths plus the count.
- Do not paste large diffs unless the user explicitly asks for raw diff output.

## `--files` Output

When `--files` is present, add affected files under each relevant change group:

```md
Изменения:
- <понятное описание изменения>
  Файлы:
  - [backend/src/example.ts](/path/to/workspace/backend/src/example.ts)
  - [frontend/src/example.vue](/path/to/workspace/frontend/src/example.vue)
```

Clickable file rules:
- Use Markdown links, not backticked paths, for file entries.
- Link targets must be absolute local filesystem paths so the Codex app can open them in Preview/side panel or IDE.
- Link labels should stay readable and repo-relative with the repository folder prefix, for example `backend/src/...`.
- Add `:line` only when a specific line is useful and known; otherwise link to the file path only.
- For deleted files, still link to the former absolute path when useful, but mark it as deleted in the label or description.

## Interpretation Hints

- `M` means modified.
- `A` means added.
- `D` means deleted.
- `R` means renamed.
- `??` means untracked.
- Staged and unstaged changes should both be reported, but keep them visually distinguishable when it matters.
