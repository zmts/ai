---
name: iterate-gate
description: Bring an implementation into compliance with a specified requirements document through iterative review, fixes, and verification. Use when the task is to satisfy that document's requirements, rather than for routine bug fixes or general code review.
---

# Iterate Gate

Invocation: `$iterate-gate <document-path> [task-description]`

Bring the implementation into compliance with the source document within the authorized task scope.

Operate autonomously through **Review → Fix → Test → Fresh Review → Coverage & Regression → Repeat** until all in-scope requirements are verified or further progress is blocked.

Respond in the user's language. Preserve the final block labels `Issue`, `Solution`, and `Status`.

## 1. Preparation

1. Read the source document, applicable project instructions, relevant implementation, and existing tests.
2. Establish the task scope from the document and user instructions.
3. Extract verifiable requirements, including:
   - Functional behavior and acceptance criteria.
   - Edge cases and error handling.
   - Invariants and regression risks.
   - Explicit non-functional constraints.
4. For each requirement, track:
   - Source document section or line reference.
   - Implementation location.
   - Verification method.
   - Verification evidence.
   - Status: `met`, `unmet`, or `unverified`.
5. Preserve existing user changes and project conventions.

Do not invent requirements, silently resolve material ambiguities, or modify the source document without separate authorization. Respect plan-only, review-only, and other explicit restrictions on changes.

## 2. Stack-Aware Verification

Before fixing issues, inspect the project's existing verification infrastructure.

### Environment Detection

Identify:

- Language and runtime.
- Package manager and lockfile.
- Build and test frameworks.
- Available verification scripts.
- CI configuration.
- Required integration dependencies.

For Node.js/TypeScript projects, inspect `package.json`, lockfiles, workspace configuration, test and TypeScript configuration, and CI workflows. Do not assume npm, Jest, or any particular framework.

### Check Selection

Prefer existing project commands and conventions. Choose checks appropriate to the affected behavior:

- Targeted unit tests.
- Relevant integration tests.
- Type checking.
- Linting.
- Build verification.
- Regression tests.
- Runtime or manual verification where required.

For monorepos, select affected packages and relevant dependents. Use the detected package manager and existing scripts. Do not install dependencies, modify configuration, or introduce tooling solely for verification unless necessary and authorized.

### Verification Rules

1. Run targeted checks during development where possible.
2. Run broader regression checks before completion.
3. Record the exact commands actually executed and their results.
4. Distinguish `passed`, `failed`, `skipped`, and `unavailable`.
5. Never count skipped or unavailable checks as passed.
6. Never weaken or bypass tests to obtain a passing result.

If mandatory checks require unavailable services, credentials, or infrastructure, report the limitation and leave affected requirements unverified.

## 3. Iterative Cycle

### Step A — Review

Review the current implementation against the **entire** requirement checklist, including relevant behavior outside the current diff.

For each confirmed gap, identify:

- Violated requirement.
- Relevant code location.
- Observable defect or missing behavior.
- Reproduction steps or supporting evidence.
- Possible regression impact.

Separate confirmed defects from assumptions, optional improvements, and unavailable checks. Group issues sharing the same root cause.

### Step B — Plan Fixes

Choose the smallest reliable solution consistent with KISS, DRY, and project conventions. Consider alternatives only when they involve material trade-offs.

Briefly report confirmed issues, the chosen approach, and its rationale when alternatives matter. Continue authorized work without asking for confirmation for routine implementation decisions.

### Step C — Implement and Test

1. Fix confirmed defects without expanding scope.
2. Add or update meaningful tests when existing coverage is insufficient for changed behavior or regression risks.
3. Run applicable checks selected through Stack-Aware Verification.
4. Recheck previously fixed scenarios.
5. Record actual execution results.

Do not change requirements to match existing code, introduce unnecessary dependencies, or rewrite tests merely to make them pass.

### Step D — Fresh Requirement Review

After fixes, the **same agent** performs a distinct verification pass against the **original source document** and current implementation.

- Re-read the original requirements instead of relying on prior review conclusions.
- Reassess the full requirement checklist, including behavior outside the latest diff.
- Validate each reported gap against the requirements and observable implementation behavior.
- Treat previous conclusions as hypotheses, not established facts.
- Do not delegate the review or create separate implementer/reviewer roles.

Do not treat speculative findings as confirmed defects without evidence.

### Step E — Requirement Coverage and Regression Verification

Verify **requirement coverage and evidence freshness together** after each round of fixes:

1. Reevaluate the entire checklist: original requirements, previously fixed issues, affected integration boundaries, and relevant edge cases.
2. For each requirement, determine whether a meaningful behavioral check exists. Passing existing tests does not prove compliance with requirements those tests do not exercise.
3. Identify missing verification coverage. Add focused tests where feasible and appropriate; otherwise use suitable runtime/manual evidence or mark the requirement `unverified`.
4. When subsequent changes affect verified behavior, dependencies, or integration boundaries, invalidate the corresponding earlier evidence and rerun the affected checks.
5. Before declaring `DONE`, run the necessary broader regression checks against the final implementation state. Record what ran and which requirements the results support.

A fixed defect does not automatically verify its entire associated requirement. Repeat the cycle while actionable gaps remain.

## 4. Persistent Checkpoint

Maintain a persistent checkpoint when the task involves multiple iterations, significant changes, or risk of context loss.

### Location and Git Isolation

Store checkpoints under the repository root:

`.codex/iterate-gate/<task-id>.md`

Derive a stable, collision-resistant task ID from the source document path and task scope.

**Configure Git isolation lazily, only when a persistent checkpoint is actually needed.** Do not create the directory or modify Git metadata during routine short tasks.

When a checkpoint is required:

1. Identify the Git repository root and the intended checkpoint path.
2. Check whether that path is already ignored (for example, with `git check-ignore`). If it is, do not change Git exclusions.
3. Check whether any relevant checkpoint paths are already tracked in the Git index. Ignore rules do not untrack existing files; do not overwrite or delete tracked checkpoints.
4. Only if an exclusion is needed, resolve the repository-local exclude file using `git rev-parse --git-path info/exclude` and append the following rule without duplicating an existing entry:

   ```gitignore
   /.codex/iterate-gate/
   ```

5. Create the checkpoint directory under the repository root and confirm the intended checkpoint path is ignored using `git check-ignore`.

Do not modify `.gitignore`, unrelated exclude entries, or tracked project files for checkpoint management. If safe local Git isolation is unavailable, use an existing ignored persistent location or report the limitation.

For Git worktrees, resolve Git metadata dynamically; do not assume `.git` is a directory. Avoid unrelated Git metadata changes. Do not commit, push, or deploy as part of checkpoint management.

### Contents

Record only information necessary to resume:

- Source document and task scope.
- Current iteration and progress.
- Requirement checklist and statuses.
- Confirmed unresolved defects.
- Applied fixes and affected files.
- Verification commands and actual results.
- Remaining regression checks.
- Blockers and next actions.

Keep entries concise and evidence-based. Never store credentials, tokens, secrets, or unnecessary source content.

### Update Policy

Update the checkpoint:

- After each completed iteration.
- After significant implementation changes.
- When blockers appear.
- Before ending an incomplete task.

Avoid updating it after every minor action.

### Recovery

Before starting a new cycle:

1. Look for a checkpoint matching the document and task scope.
2. Validate it against the current source document and working tree.
3. Reinspect affected files.
4. Invalidate evidence made stale by document, implementation, dependency, or integration changes, and rerun relevant checks before relying on it.
5. Resume from remaining requirements and defects.

A checkpoint is a recovery aid, not proof that previously verified behavior remains correct.

### Cleanup

After successful completion:

- Remove only the checkpoint created for the completed task.
- Preserve unrelated checkpoints and user-created files.
- Retain the local Git exclusion rule for subsequent runs.
- Include relevant verification evidence in the final report.

If the task is `BLOCKED`, preserve its checkpoint for later continuation.

## 5. Progress Control

Track progress across iterations using:

- Confirmed unresolved defects.
- Verified requirements.
- Failing checks.
- Recurring failures.
- Newly introduced regressions.

Use **five iterations as a progress checkpoint**, not an automatic completion condition.

At that checkpoint:

1. Continue if specific actionable work and a credible verification path remain.
2. Reconsider the approach if the same failure recurs.
3. Stop as `BLOCKED` when progress depends on unavailable resources, external decisions, or no credible new approach remains.

Never repeat an unchanged unsuccessful fix without revising the hypothesis. Do not invent issues to justify another iteration. Do not stop solely because an iteration count was reached when there is a concrete path to progress.

## 6. Completion Gate

Declare `DONE` only when **all** conditions hold:

1. Every in-scope requirement is implemented.
2. Every requirement has explicit verification evidence.
3. All mandatory applicable checks pass.
4. No confirmed in-scope defects remain.
5. Previously fixed behavior passes relevant regression checks.
6. A final full review finds no outstanding gaps.

A clean review without evidence is insufficient. A second clean verification pass is recommended for complex or regression-prone changes.

Declare `BLOCKED` when required implementation or verification cannot be completed. Never report partial compliance as `DONE`.

## 7. Source Document Assessment

After implementation, assess whether the source document needs clarification. Propose additions only for:

- Material ambiguities.
- Missing acceptance criteria.
- Important behavioral constraints.
- Decisions necessary to define the contract.

For each proposal provide:

- Target document section.
- Why the addition matters.
- Ready-to-insert wording.

Do not modify the source document without separate user authorization. Do not use proposed changes to retroactively justify the implementation or claim completion. If no additions are needed, say so briefly.

## 8. Reporting

Provide concise progress updates after meaningful iteration rounds, changes in approach, or blockers. Avoid repetitive per-fix narration.

### Final Report

Include:

1. Overall result: `DONE` or `BLOCKED`.
2. Requirement coverage summary.
3. Checks actually executed and their results.
4. One block for each unique confirmed issue, including resolved and outstanding issues:

**Issue**

The violated requirement, its document reference, relevant code, and observed gap.

**Solution**

The applied fix or proposed resolution; include alternatives only for material trade-offs.

**Status**

Use one of:

- `Fixed and verified` — the issue is resolved and verified.
- `Fixed but unverified` — the change was made, but required verification is unavailable.
- `Unresolved` — the issue remains open.

For unverified or unresolved issues, state the reason, blocker, and required action.

If no issues were found, say so explicitly and list the checks performed and material verification limitations. Do not invent issues to fill the format.

#### Source Document Assessment

Include necessary proposals with rationale and ready-to-insert wording, or confirmation that no additions are needed.
