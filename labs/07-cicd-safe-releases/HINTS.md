# Lab 07 — Progressive hints

Use these only for **Level 1**. Hints move from broad concepts to more concrete prompts; stop when unstuck. Levels 2–4 receive their own hints only when activated. Do not treat the hints as implementation evidence.

## Level 1 — Guided implementation

### Hint 1 — Draw the promotion boundary

Separate building an artifact from deploying it. Identify which stages can run automatically, which stage needs a human decision, and which failures must prevent production from changing.

### Hint 2 — Keep one artifact moving forward

The development and production environments should receive the same packaged application bytes from the same source revision. Compare artifact identity/digest and template references, not just matching version labels.

### Hint 3 — Assign a role to each actor

The pipeline orchestrates, the build project tests/packages, CloudFormation and CodeDeploy deploy, and the Lambda role runs the application. Start with a resource/action matrix; avoid one broad role for every stage.

### Hint 4 — Make the approval useful

An approval is a release decision only if the reviewer can identify the commit, change-set summary, environment, and development verification. A pipeline approval does not replace the learner's provisioning approval.

### Hint 5 — Design the rollback proof before the failure

Decide which metric will alarm, how the canary will receive enough synthetic traffic to evaluate it, what the prior version should do after rollback, and how you will show production was untouched. Keep the failure injection in development.

### Hint 6 — Read the CloudFormation change set

Before execution, compare resources, environment parameters, IAM changes, and replacements. A successful template validation does not tell you whether a proposed change is safe to apply.

### Hint 7 — Understand a green pipeline's limits

A successful run proves the configured checks ran for a particular revision; it does not prove every permission boundary, alarm threshold, traffic pattern, or production dependency is correct. Verify the endpoint and deployment state directly.

### Hint 8 — Treat cleanup as a release stage

Inventory resources by stack and tags before deletion. Confirm the exact artifact bucket and objects belong to this lab, remove active pipelines promptly, and verify absence rather than assuming stack deletion removed every object or log group.
