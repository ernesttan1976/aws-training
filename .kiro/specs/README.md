# Change specs

Create one folder per multi-file feature or architecture change. Use `/specify <outcome>` to draft it and `/spec-review <folder>` for a read-only quality check.

Each spec contains:

- `requirements.md` — user/business outcomes and numbered, testable acceptance criteria.
- `design.md` — architecture and decisions, interfaces/data flow, security, failure behavior, operations, validation, and rollout/rollback where relevant.
- `tasks.md` — ordered checklist items traced to requirement IDs, each with a verifiable completion condition.

Copy from `_template/`. Keep specs focused on repository or product changes; learner AWS exercises remain learner-owned and continue to use the lab artifact format.
