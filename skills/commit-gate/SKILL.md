---
name: commit-gate
description: Inspect local git changes, select a commit scope and message, and present a compact plan before committing. Wait for explicit approval by default; with --auto, choose the best message and execute automatically. Supports --api, --ui, --all, --exclude, --fast, --a as an alias for --auto, and --aa as an alias for --all --auto without hard-coding project paths.
---

# Commit Gate

Prepare a safe commit plan before creating any commit.
Keep the workflow narrow and explicit. Require approval unless the user invokes `--auto`.

## Workflow

1. Resolve any requested argument scope.
2. Inspect the repository state.
3. Group the intended commit scope.
4. Check whether the commit requires a changelog entry.
5. Propose one primary commit message and meaningful alternatives when useful.
6. Present the commit plan in the user's language.
7. Wait for an explicit user choice, or select the primary message automatically with `--auto`.
8. Execute after approval or under the authorization provided by `--auto`.

## Argument Scope Rules

Interpret explicit user arguments before running git commands or proposing a commit scope.

Normalize `--a` to `--auto` and `--aa` to `--all --auto` before applying any rules below. Each alias has exactly the same scope and authorization as its expanded flag or flags, including when combined with `--api`, `--ui`, `--exclude`, or `--fast`.

Do not hard-code absolute project paths in this skill. Resolve the selected scope from the current workspace layout, repository names, local instructions, recent task context, and the user's wording.

- `--api` targets the backend/server/API repository or backend scope for the current workspace.
- `--ui` targets the frontend/client/UI repository or frontend scope for the current workspace.
- `--api --all` means inspect and commit the full approved backend/API scope, not only the last change you worked on.
- `--ui --all` means inspect and commit the full approved frontend/UI scope, not only the last change you worked on.
- `--exclude` removes one or more files or directories from the requested commit scope.
- `--fast` requests a quicker commit-gate pass with minimal code and diff reading, using the current task context as the main source for the commit proposal.
- `--auto` authorizes automatic selection of the most suitable commit message and execution within the resolved scope, without waiting for a menu choice.

If the selected scope is not already the current git repository, locate the matching git root before running git commands. If the workspace has several plausible backend or frontend repositories and the intended scope cannot be inferred safely, ask the user to choose the repo instead of guessing.

When `--api` or `--ui` is provided without `--all`, prefer the latest task scope you worked on inside that repo. Do not silently include older unrelated files from the same repo just because they are modified.

When `--all` is provided with `--api` or `--ui`, the scope expands from the latest task scope to the entire selected repo's current working tree. Still inspect the diff and call out risky or unrelated changes. Wait for normal approval unless `--auto` is provided.

Do not mix backend and frontend commits for scoped calls:
- if only `--api` is provided, do not inspect or commit frontend changes;
- if only `--ui` is provided, do not inspect or commit backend changes;
- if both `--api` and `--ui` are provided, stop and ask which one to commit first unless the user explicitly requested separate sequential commit plans.

If no argument scope is provided, use the normal narrow-scope behavior.

## Auto Mode Rules

Use `--auto` as explicit authorization to complete the normal commit workflow without asking the user to choose a message or confirm the commit again.

- Inspect the resolved scope and choose the most suitable type, module, and summary using the existing Message Rules and `[type][module]: action summary` format.
- Show a compact plan with the selected message, commit scope, changelog handling, and relevant exclusions before staging. State that `--auto` selects the primary message and proceeds; omit the choice menu and alternatives.
- If a matching changelog entry is missing, include its exact proposed text in the plan, then add it as part of the authorized commit. `--auto` does not authorize skipping changelog policy.
- Stage and commit the planned scope, then provide the Post-Commit Report.

`--auto` does not expand scope, imply `--all` or `--fast`, bypass safety checks, or authorize amending, pushing, or rewriting history. It combines with scope flags, `--exclude`, and `--fast`; `--fast --auto` keeps the reduced inspection rules and uses automatic execution.

If scope is ambiguous, risky unrelated changes cannot be separated safely, or the working tree changes after the plan, stop and explain the issue. Resolve the uncertainty before proceeding; do not treat `--auto` as permission to guess.

## Fast Mode Rules

Use `--fast` when the user wants a quicker commit-gate pass based mostly on the current task context.

`--fast` means:
- read less code and avoid broad diff exploration;
- build the commit proposal primarily from the current conversation context, latest task scope, and `git status --short`;
- inspect only the minimum diff needed to confirm scope, detect risky unrelated changes, validate changelog needs, and avoid staging excluded paths.

`--fast` does not mean:
- authorize auto-commit by itself (`--auto` is required for automatic execution);
- skip `git status --short`;
- skip changelog policy;
- skip `--exclude` safety;
- include unrelated files silently;
- replace `--api`, `--ui`, or `--all`.

When `--fast` is used:
- prefer the latest task scope when `--api` or `--ui` is provided without `--all`;
- if `--all` is also provided, inspect the full selected repo status but keep diff reading selective;
- if the current context is enough, present the commit plan without deep code review;
- if scope, risk, or changelog impact is unclear, say that `--fast` is not enough for that part and read only the necessary files or ask the user.

Before committing, still:
- wait for explicit menu approval unless `--auto` authorizes execution;
- stage only approved files or hunks;
- verify `git diff --cached --name-only`;
- verify no `--exclude` path is staged;
- run `git commit` only with the approved message.

## Exclude Rules

Use `--exclude` when the user wants specific files or folders to stay out of the commit, even if `--all` is provided.

Parse excludes as path arguments after `--exclude`:
- one path: `--exclude scripts/experiments1`;
- several path arguments: `--exclude scripts/experiments1 scripts/experiments2 test.js`;
- comma-separated paths: `--exclude scripts/experiments1, scripts/experiments2, test.js`;
- repeated flags: `--exclude scripts/experiments1 --exclude scripts/experiments2`;
- mixed form: `--exclude scripts/experiments1, scripts/experiments2 --exclude test.js`.

When one `--exclude` is followed by several non-option tokens, treat every token up to the next `--flag` as an exclude value.
Normalize each excluded path by trimming commas and surrounding whitespace.
Treat excluded paths as relative to the selected git repository root unless the user explicitly gives a path that resolves inside that repository.
For directories, exclude the directory and every path below it.
For files, exclude the exact file path.

Apply excludes consistently through the whole workflow:
- when reviewing status and diff, show matching excluded paths separately from the proposed commit scope;
- when building the plan, include an `Исключено по --exclude:` or equivalent note when excluded paths exist;
- when staging, stage only approved non-excluded paths or hunks;
- if an excluded path is already staged, unstage that path before committing or stop and ask the user before proceeding;
- before `git commit`, verify with `git diff --cached --name-only` that no excluded path is staged.

Do not use `--exclude` as a reason to ignore unrelated non-excluded changes silently. If non-excluded changes still look unrelated or risky, call them out in the normal commit plan.

## Inspect The Repository

Inspect the working tree before proposing a commit:
- Run `git status --short`.
- Inspect staged and unstaged diffs as needed.
- Notice untracked files.
- Notice unrelated or risky changes that should stay out of scope.

When `--fast` is provided, keep this inspection intentionally small:
- still run `git status --short`;
- rely on the current task context for the commit description when it is sufficient;
- read only targeted diff hunks needed for scope, risk, changelog, staged-state, or exclude safety.

Prefer the narrowest safe commit scope.
Do not assume that every modified file belongs in the same commit.

## Changelog Rules

For repositories that contain `CHANGELOG.md`, every commit created through this skill must keep the changelog current unless the user explicitly approves an exception.

Use the repository's `CHANGELOG.md` as a state log, not a release log:
- do not add release versions, SemVer sections, or `Unreleased`;
- group entries by date in `YYYY-MM-DD` format;
- if today's date already exists, add the new item under that date;
- if today's date does not exist, add a new `## YYYY-MM-DD` section near the top of the file;
- use category headings only when needed: `Added`, `Changed`, `Fixed`, `Removed`;
- describe the project state after the change, not a mechanical file list.

When the approved commit scope changes project behavior, contracts, UI, workflow, documentation, tests, configuration, or operational state, include the matching `CHANGELOG.md` update in the commit scope.

When the diff already includes a suitable changelog entry:
- mention it in the commit plan;
- keep it staged with the related change.

When the diff does not include a suitable changelog entry:
- propose the exact category and bullet text in the commit plan;
- ask for approval to add it before committing, or add it under `--auto` authorization;
- do not create the commit until the changelog entry is added or the user explicitly confirms that this commit should skip changelog.

If the repository does not contain `CHANGELOG.md`, do not invent one as part of commit-gate. Only enforce this rule where the file already exists or where the current task explicitly adds it.

## Present The Commit Plan

Always present a compact commit plan before any `git add` or `git commit`.

With `--auto`, use the compact plan described in Auto Mode Rules without a menu. The menu rules below apply to the default confirmation flow.

Use this structure:

```md
План коммита

Что коммитится:
- ...
- ...

Menu:
0. Cancel
1. Use message - `[fix][module]: adjust order validation for stop-loss flow`
2. Use message - `[minor][module]: refine order validation for stop-loss flow`
3. Use message - `[refactor][module]: simplify stop-loss validation path`
```

Include:
- a short list of files or logical change groups that belong in the commit;
- the changelog entry that will be included, or the explicit reason it is skipped;
- a note about untracked files when relevant;
- a note about unrelated changes excluded from the commit when relevant;
- a note about paths excluded by `--exclude` when relevant;
- one clear primary commit message in the required bracket format, attached to menu item `1`;
- up to two alternative messages when they materially help the user choose;
- more than two alternatives only when there is a real naming ambiguity, mixed intent in the change, or several equally valid commit angles worth presenting.
- keep the exact menu label `Menu:` in the visible plan.

Always map menu items to messages like this:
- `0` means cancel without changing git state;
- `1` means execute with the main message;
- `2` means execute with the first alternative message;
- `3` means execute with the second alternative message;
- additional numbered execute options are allowed only when there is a strong reason to present more alternatives.

## Wait For Approval

When `--auto` is provided, its explicit authorization replaces this menu approval step; proceed under Auto Mode Rules.

Otherwise:

Do not run `git add`.
Do not run `git commit`.
Do not stage all changes blindly.

Wait until the user explicitly chooses `Use message` or clearly instructs you to proceed.
If the user chooses `0` or `Cancel`, stop without changing git state.
If the user chooses an execute option, use the exact message attached to that menu item.

## Execute The Approved Commit

After approval or automatic selection with `--auto`:
- stage only the approved files or hunks;
- keep every `--exclude` path unstaged, including paths that were already staged before commit-gate execution;
- re-check that the staged scope still matches the plan;
- re-check that no `--exclude` path appears in `git diff --cached --name-only`;
- update the plan if new changes appeared before staging;
- before `git commit`, verify that the approved message follows `[type][module]: action summary`, with an allowed type in the first bracket and the selected module in the second; if it does not, correct the proposal before proceeding (obtain approval for the corrected message unless `--auto` applies);
- create the commit with the approved message;
- provide the Post-Commit Report.

Use non-interactive git commands only.
Do not amend commits or rewrite history unless the user explicitly requests that behavior.

## Post-Commit Report

After every successful commit, including `--auto` and `--fast`, report:
- the final commit hash and message;
- every file changed in that commit, with a `git diff --stat` style count and `+`/`-` bar;
- a final line in the exact format `Total: 15 files: +434 / −83`, using the actual file count, insertions, and deletions.

Read the statistics from the created commit's captured hash, not from the working tree, index, or a moving `HEAD`. Use `git show --format= --stat <commit-hash>` for the file listing and `git show --format= --numstat <commit-hash>` for totals. Keep full file paths visible and include all changed files without truncating the list. Count binary files in the file total, display their binary marker, and exclude their non-numeric `-` values from line totals.

Put the file listing and total in a fenced `text` block to preserve alignment. Replace Git's default summary line with the requested `Total:` line. For multiple commits, provide a separate report for each commit.

## Message Rules

Always use this format in every mode, including menu alternatives, `--fast`, and `--auto`:

```text
[type][module]: action summary
```

The first bracket MUST contain `type`; the second bracket MUST contain `module`. Never reverse this order. A module or component name must never occupy the first bracket, even when it is the main focus of the change.

Examples:
- `[fix][orders]: correct stop-loss validation for market entry`
- `[feature][signals]: add breakout confirmation filter`
- `[refactor][risk]: simplify position sizing calculation`
- `[feature][trend.service]: add backend trend line extension segment`
- `[docs-update][docs]: document strategy config overrides`
- `[minor][api]: rename pnl response field for clarity`
- `[test][tests]: add coverage for strategy warmup edge cases`
- `[ci][ci]: run lint before unit tests in pull requests`
- `[deps][deps]: bump ccxt to latest supported version`
- `[perf][engine]: reduce indicator recomputation in backtest loop`
- `[build][build]: align tsup output with node runtime target`
- `[security][auth]: harden token validation for websocket sessions`

### Message Parts

- `type` (first bracket): one of `feature`, `fix`, `refactor`, `minor`, `docs-update`, `test`, `chore`, `perf`, `ci`, `deps`, `build`, `security`;
- `module` (second bracket): the module, bounded context, package, or top-level area affected by the change, or a component/file name under Module Selection rules;
- `action summary`: a short phrase that states what changed;
- if useful, extend the action summary with the specific action, flow, endpoint, handler, or scenario that changed.

### Module Selection

Choose the most meaningful module name from:
- the main feature area;
- the bounded domain;
- the package or folder that contains most of the change;
- a component or file name only when that component or file is the clear semantic focus of the change;
- `docs` when the change is documentation-only.

Avoid generic modules such as `project`, `misc`, or `general` unless there is no better option.

Use a component or file name in `[module]` only when it identifies the main element being changed and makes the scope clearer than a broader area name. Supporting tests and small related edits in other files do not prevent this choice. Do not select the main element solely by changed-line counts.

For file names, prefer the semantic file stem without the extension, for example:
- `OrderForm.vue` -> `OrderForm`
- `trend.service.ts` -> `trend.service`
- `order-validation.handler.ts` -> `order-validation.handler`

When a large change spans several independent parts or is cross-cutting, use a broader module, domain, or area name instead of naming one component or file. If there is no clear semantic focus, also use the broader name.

Examples:
- Validation fix centered on `OrderForm.vue`, with related tests: `[fix][OrderForm]: correct order amount validation`.
- Limit-order support spanning several parts of order creation: `[feature][orders]: add limit order support`.

### Type Selection

- `feature`: new user-facing or system-facing capability;
- `fix`: bug fix or behavioral correction;
- `refactor`: internal restructuring without intended behavior change;
- `minor`: small safe improvement, rename, cleanup, or low-risk adjustment that does not fit better as `fix` or `refactor`;
- `docs-update`: documentation-only changes;
- `test`: tests only or primarily test coverage changes;
- `chore`: repository maintenance, housekeeping, or supporting changes without direct product behavior impact;
- `perf`: performance optimization with intended runtime or resource improvement;
- `ci`: CI/CD pipeline, workflow, or automation changes;
- `deps`: dependency updates or dependency-related maintenance;
- `build`: build system, bundling, compilation target, or packaging changes;
- `security`: vulnerability mitigation, hardening, or auth/permission/security behavior updates.

Do not invent additional commit types unless the user explicitly asks to extend the format further.

### Summary Rules

Make the summary:
- specific to the real change;
- short enough for readable history;
- action-oriented rather than vague.

Avoid vague summaries such as `update stuff`, `changes`, or `misc fixes`.

## Safety Checks

Stop and explain the issue when:
- there is nothing to commit;
- the working tree changed after the plan was shown;
- the commit scope is ambiguous because unrelated changes are mixed together;
- the requested commit would include risky or clearly unrelated files.

If needed, present a refreshed commit plan instead of guessing.
