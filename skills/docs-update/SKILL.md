---
name: docs-update
description: Prepare documentation updates before editing files. Use when Codex needs to add or update project documentation such as README files, docs/ pages, ADRs, onboarding notes, runbooks, or inline documentation, and should first show an editorial preview with affected files, semantic changes, ready-to-insert text blocks, and wait for explicit user approval or cancellation. Show raw diffs only when the user explicitly asks for a diff.
---

# Docs Update

Prepare documentation changes as a visible editorial proposal before writing files.
Use read-only inspection if needed to understand the current docs and draft the update, but do not edit any file until the user chooses `1. Apply`.

## Workflow

### 1. Restate the request briefly

Begin with one short line in the user's language.
State what documentation change is being prepared.

### 2. Inspect docs in read-only mode if needed

Read only the minimum context needed to identify:
- which documentation files should be updated;
- which new documentation files should be added;
- what semantic changes and ready-to-insert text blocks should be proposed for each file.

Do not edit files during this stage.

### 3. Show the documentation plan

Render a normal visible assistant answer that contains:
- a short one-line understanding;
- a flat list of documentation files that will be updated or added;
- a compact semantic-change table;
- the ready-to-insert Markdown blocks for changed or new sections;
- a short list of direct replacements when useful.

For the file list:
- Mark each file as `update` or `add`.
- Prefer concrete repository paths.
- Include every affected documentation file, even if only one file is affected.

For the semantic-change table:
- Use columns like `File`, `Section`, and `Change`.
- Keep rows short and concrete.
- Focus on what will change in meaning, not line-by-line patch mechanics.

For ready-to-insert text blocks:
- Show normal Markdown, not `diff`, unless the user explicitly asks for raw diff.
- Group blocks by file path and section heading.
- Include enough surrounding heading context so the user can understand where each block goes.
- For replacements, label whether the block replaces an existing section or is inserted as a new section.
- For very small edits, a direct replacement list can be enough.

For raw diffs:
- Do not show raw fenced `diff` blocks by default.
- Show raw diffs only when the user explicitly asks for `diff`, `покажи diff`, `raw diff`, or similar.
- If showing raw diff, keep it scoped to relevant lines and still include the stable approval menu.

### 4. Offer a stable approval menu

After the proposed editorial preview, always show this menu in the user's language:

```md
Выберите:
1. Apply
2. Cancel
```

Do not replace the menu with buttons or different labels.

### 5. Wait before editing

Do not edit files, run write operations, or apply patches until the user chooses `1. Apply` or explicitly says to proceed.
If the user chooses `2. Cancel`, stop without making changes.

### 6. Execute after approval

After approval:
- apply the documentation changes that match the proposed editorial preview;
- keep the final edit aligned with what was shown to the user;
- only diverge when the user explicitly asks for a different revision.

If scope changes materially after approval, show an updated documentation plan and repeat the same menu.

## Response Shape

Use this compact structure for the gate output:

~~~md
[One-line understanding]

Документы:
- update: path/to/existing-doc.md
- add: path/to/new-doc.md

Смысловые изменения:
| File | Section | Change |
|---|---|---|
| path/to/existing-doc.md | Section name | What changes in meaning |

Готовые блоки текста:

path/to/existing-doc.md — Section name
```md
## Section name

Ready-to-insert Markdown text.
```

Короткие замены:
```text
old phrase -> new phrase
```

Выберите:
1. Apply
2. Cancel
~~~

## Style Rules

- Match the user's language.
- Keep the proposal concrete and easy to review.
- Prefer editorial previews over raw diffs.
- Keep ready-to-insert text readable as documentation, not as a patch.
- Keep the affected-doc list complete.
- Keep the menu stable so the user can reply with a single number.
