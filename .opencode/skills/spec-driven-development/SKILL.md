---
name: spec-driven-development
description: Use when defining a multi-file product, infrastructure, or tooling change as explicit requirements, a design, and traceable implementation tasks before coding.
---

# Spec-driven development

Use this workflow for multi-file features, architecture changes, and infrastructure changes. Skip it for routine, localized fixes unless the user requests a spec.

## Workflow

1. Read the applicable `.kiro/steering/` documents and inspect only the code/docs needed to understand the change.
2. Create or update `.kiro/specs/<short-kebab-case-name>/` with `requirements.md`, `design.md`, and `tasks.md` from the templates.
3. Resolve material ambiguity with a small number of focused questions. Record safe, low-impact assumptions explicitly.
4. Make requirements numbered, observable, and testable. Prefer Given/When/Then acceptance criteria.
5. Map every requirement to at least one design decision and one implementation/verification task.
6. Include failure behavior, security/privacy, observability, cost/region, deployment approval, and cleanup when relevant.
7. Review the spec with `/spec-review <spec-folder>` or the `spec-reviewer` agent.
8. Implement only when the user requests implementation and the repository's learning rules allow it. Update task checkboxes as work is verified; do not mark tasks complete based on intention.

## Spec lifecycle

- `requirements.md` describes the outcome and acceptance criteria, not a preselected implementation.
- `design.md` records architecture, tradeoffs, interfaces, data flow, error behavior, security, and validation approach.
- `tasks.md` breaks work into ordered, independently verifiable steps with links to requirement IDs.
- Preserve prior decisions when refining a spec; note superseded decisions rather than silently erasing rationale.
- A spec is ready to implement only when its acceptance criteria are measurable, blocking questions are resolved, and verification/rollback expectations are clear.

## AWS training boundary

These specs are for repository/tooling or product changes. A learner's assigned AWS lab remains learner-owned. Do not implement their exercise, deploy billable infrastructure, or claim mastery without learner evidence and an explanation. Follow `agents.md` for the full assessment contract.
