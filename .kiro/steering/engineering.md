# Engineering steering

## Repository conventions

- Lab artifacts live together in `labs/<number>-<slug>/` and stay consistent with `LAB.md`, `HINTS.md`, `TESTS.md`, `CLEANUP.md`, `LEARNING.md`, and `COMPETENCY.md`.
- `agents.md` is the detailed project policy and is explicitly loaded by `opencode.json`; keep it canonical.
- Prefer implementation approaches that build AWS-specific skills rather than reteaching general programming.
- Use AWS CLI, AWS SDKs, and IaC where appropriate; explain which tool the learner is expected to use.
- Keep generated and learner-owned artifacts intact. Do not claim unobserved tests or runtime outcomes.

## Evidence and quality

- Make acceptance criteria measurable and map each to an automated or manual check.
- Require runtime evidence for deployed behavior; source code alone does not prove AWS service integration behavior.
- Link to current, official AWS documentation and label cost/region assumptions.
- Include actionable cleanup and verification for every lab that provisions resources.
