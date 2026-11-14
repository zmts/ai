---
name: iterate-gate
description: Bring an implementation into compliance with a specified document through repeated review, fixes, and verification. Use when the user asks to iteratively resolve requirement gaps and report issues, solution options, and verification results.
---

# Iterate Gate

Invocation: `$iterate-gate <document-path> [task-description]`.

Bring the implementation into compliance with the document's requirements within the task scope. Continue the cycle autonomously while actionable gaps remain. Respond in the user's language; keep the final block labels `Issue`, `Solution`, and `Status`.

## Preparation

1. Read the specified document, applicable project instructions, and the related implementation. Follow references in the document when needed to understand its requirements.
2. Determine the task scope from the user's description and the document. If no description is provided, use the document's requirements that apply to the current project. Do not guess about material ambiguities that change expected behavior or scope: clarify them while continuing independent work.
3. Build a working checklist of verifiable requirements with references to document sections or lines. For each requirement, track the corresponding implementation, verification evidence, and state: met, unmet, or unverified. Do not present assumptions as requirements.
4. Account for and preserve the user's existing changes. Invoking the skill authorizes fixes within the task scope but does not override explicit user restrictions, project instructions, or permission requirements. With `--plan`, review-only instructions, or another restriction on changes, perform only the permitted part of the workflow and state the limitation explicitly.

## Iterative Cycle

### 1. Review Against Requirements

Review the current implementation against the entire working checklist, including relevant behavior outside the current diff. For each real issue, identify the document requirement, implementation location, and observed gap. Distinguish confirmed defects from open questions and unavailable checks.

Do not add style comments or hypothetical improvements unless they affect task requirements, change correctness, or regressions.

### 2. Report Before Making Fixes

For the current iteration, show the user the issue or related group of issues:

- **Issue:** the requirement, location, and nature of the gap.
- **Solution options:** reasonable fixes and their material differences. If only one option is suitable, present only that option.
- **Chosen solution:** what you will change and why the approach fits the task.

Keep updates concise and concrete. Update the user when new findings emerge, the approach changes, or a blocker appears. Do not defer all findings to the final response. Presenting options does not itself require pausing for confirmation; continue authorized work.

### 3. Fix and Verify

Fix the causes of gaps, following KISS, DRY, and project conventions. Do not expand scope, introduce unnecessary dependencies, or rewrite document requirements to match existing code. Modify the source document only with separate user authorization.

Run checks appropriate to the changed behavior: existing tests, builds, type checks, integration checks, or manual verification as needed. Add tests where they verify meaningful behavior and regression risks; do not create tests merely to mirror the implementation. Record what actually ran and its results.

After each fix, report the verification result and the issue's current state. An unavailable or skipped check does not count as successful.

### 4. Review Again

After making fixes, revisit the entire set of requirements and check for regressions in affected behavior. Do not limit the review to previously found issues or treat the absence of new findings as sufficient evidence of completion.

If actionable gaps remain or new ones appear, continue the cycle. Consolidate repeated findings with the same cause into one issue, retaining its current solution and status.

## Completion and Blockers

Mark the task complete only when all requirements within its scope are met, the repeat review finds no remaining gaps, and all necessary checks pass.

If progress is blocked by missing access, a material ambiguity, a required user decision, or an unavailable mandatory check, complete independent authorized work and report the specific blocker. Do not declare the whole task complete while requirements remain unverified.

If the same issue recurs without progress, reconsider the hypothesis and verification approach. Do not endlessly repeat an unchanged failed attempt. When further progress requires an external change or user decision, stop the dependent work and explain exactly what is needed. Do not stop solely because of the iteration count if there is a justified next step.

## Assess Additions to the Source Document

At the end of the work, assess whether ambiguities discovered, important decisions, or missing conditions should be explicitly captured in the source document's contract. Propose only additions justified by the completed work; do not turn incidental implementation details into contract requirements or adjust requirements to fit the code.

For each necessary addition, provide:

- what should be clarified or added and in which section;
- why it matters to the contract;
- concrete wording ready to insert into the document.

These are proposals for the user, not automatic document changes. Apply them only with separate user authorization. If a proposed clarification changes requirements or depends on a decision that has not been made, state that explicitly; the proposal does not resolve an existing gap or establish task completion.

If no additions are needed, say so briefly.

## Final Result

Briefly state the overall outcome and checks performed. Then provide one block per unique issue, including resolved and outstanding issues. Do not replace these blocks with a table.

**Issue**
The specific gap against a document requirement; link to the requirement and implementation when available.

**Solution**
The applied solution or proposed fix for an unresolved issue. Include material alternatives only when they help explain the choice.

**Status**
Fixed and verified / fixed, verification unavailable / unresolved. For an unverified or unresolved result, state the reason, blocker, and required action.

If no issues were found, say so directly and list the checks performed and material verification limitations. Do not invent issues to fill the format.

After the issue blocks, briefly report the source document assessment: necessary proposals with rationale and ready-to-insert wording, or a statement that no additions are needed.
