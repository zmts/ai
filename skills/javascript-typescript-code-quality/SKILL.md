---
name: javascript-typescript-code-quality
description: Review or improve JavaScript and TypeScript code quality using KISS, DRY, clear domain intent, safe data handling, and proportionate verification. Use for code-quality reviews, readability and maintainability improvements, and scoped cleanup or refactoring; isolated framework or type-system diagnosis alone does not need this skill.
metadata:
  version: "1.0.0"
---

# JavaScript / TypeScript Code Quality

Apply the rules in [code-quality.md](references/code-quality.md). This skill provides the workflow for using those rules when reviewing, writing, modifying, or refactoring JavaScript and TypeScript.

## Workflow

1. Establish the requested mode and scope: review, explanation, implementation, or refactoring. Review and analysis remain read-only; implementation requests authorize the relevant edits without an additional approval gate introduced by this skill.
2. Read the target project's instructions and relevant code. For a diff review, inspect the actual diff and enough surrounding producers, callers, and tests to establish behavior. For implementation, trace the affected execution path and existing reusable logic before choosing a change.
3. Inspect package scripts, runtime targets, compiler/linter settings, and existing conventions when relevant. Reuse the project's dependencies, validators, error handling, and test tools; preserve existing module boundaries.
4. Read the applicable sections of the quality rules using the table below. For a broad code-quality audit, read the whole document. Apply the project-context guidance below alongside the rules.
5. In review mode, report supported findings within the requested scope. In implementation mode, make the smallest cohesive change and verify the behavior it affects. Preserve behavior during refactoring unless the user requests a behavior change.

## Read the rules by concern

| Concern | Sections in code-quality.md |
|---|---|
| KISS, DRY, reuse, proportional changes | 1. Priorities, Simplicity, and Reuse |
| Domain names, units, intermediate values, comments | 2. Readability, Naming, and Comments |
| Responsibilities, early returns, parameters, conditions, mutation | 3. Functions and Control Flow |
| External data, state, collections, equality and falsy values | 4. Data, State, and JavaScript Semantics |
| Promise ownership, execution order, named awaited results, errors | 5. Asynchronous Work and Error Handling |
| unknown, guards, assertions, modeling, imports, compiler settings | 6. TypeScript-Specific Rules |
| Cohesion, exports, functions versus classes, public boundaries | 7. Modules and Public API |
| Context-specific protection and safe logging | 8. Security and Logging |
| Complexity, repeated work, concurrency, memory, measurements | 9. Performance and Algorithmic Complexity |
| Meaningful tests, existing automation, safe cleanup | 10. Testing, Tooling, and Refactoring |

The quality document contains the substantive rules and examples. Do not replace it with a generic checklist or load unrelated references by default.

## Applying the rules in the target project

- User requirements and local project instructions take precedence. General JavaScript rules apply to TypeScript; skip TypeScript-only requirements for JavaScript. Do not migrate the language or introduce a framework to apply the skill.
- Use the target project's architecture and contracts to determine layers, folder names, model ownership, and test placement.
- Preserve public names and wire formats. A preference for explicit units does not authorize a breaking contract rename; use clear internal names or boundary conversions where appropriate.
- Match shared types and reusable logic by domain meaning and responsibility. Avoid both mechanical duplication and coupling independently owned contracts just because shapes match.
- Follow the quality rules on meaningful intermediate values and named awaited results while respecting existing project conventions. When extracting expressions, preserve short-circuiting, side-effect order, repeated loop evaluation, and the difference between synchronous throws and rejected Promises. Do not move guarded work into an unconditional preparation step.
- Preserve existing compiler/module settings and dependencies in scoped tasks. The strict compiler configuration in section 6 is a recommendation for new projects, not a requirement to migrate existing ones.
- The 100-element array guideline is project-specific. Judge performance using actual bounds, frequency, memory, and the whole execution path; do not treat that number as a universal cutoff.
- Validate untrusted runtime values even in TypeScript; types and assertions do not validate data. A `void` expression alone does not handle a Promise rejection. Local mutation may be appropriate when it respects ownership and avoids unnecessary copying.
- Before removing apparently unused code, check deliberate public exports, framework discovery, and external callers as relevant. An absent local reference alone does not prove a public export is unused.

## Review output

Lead with actionable findings, ordered by priority. For each, give a precise file/line reference, the triggering condition, its concrete consequence, and a concise correction. Distinguish confirmed behavior from unresolved assumptions.

- Prioritize correctness, security, contract violations, and regression risk. Include readability, complexity, or duplication findings when they demonstrate a concrete maintenance burden or address the requested quality review.
- Do not manufacture findings from stylistic preferences, hypothetical scale, or a preferred abstraction. Explain the impact of a tooling diagnostic rather than merely repeating it.
- For diff reviews, focus on issues introduced or worsened by the change. Label a pre-existing issue when it materially affects the change.
- If there are no actionable findings, say so. State material verification gaps without claiming the code is proven correct.

## Implementation and verification

Use existing checks proportionate to the change. For TypeScript, use the relevant project type-check command; for behavior changes, verify meaningful normal and error/edge cases. Compiler success alone does not establish runtime validation or async correctness.

Do not add dependencies, test frameworks, coverage targets, broad formatting, or repository-wide cleanup merely to satisfy the skill. Keep checks read-only in review/analysis mode and avoid commands that auto-fix files.

Report what changed and why, checks actually performed, and material remaining uncertainty. Distinguish theoretical complexity from measured speedups; do not claim tests or benchmarks that were not run.
