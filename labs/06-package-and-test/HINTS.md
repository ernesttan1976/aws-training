# Lab 06 — Progressive hints

Use these only for **Level 1**. Hints move from broad concepts to concrete next steps; stop as soon as you are unstuck. Level 2–4 details will be provided only when those levels are activated.

## Level 1 — Guided implementation

### Hint 1 — Separate the stages

Keep the workflow explicit: unit tests → SAM template validation → SAM build → local Lambda/API event tests → package inspection → approved S3 upload. A successful local build is not the same as an uploaded package, and packaging is not deployment.

### Hint 2 — Treat the event as an interface

An API Gateway HTTP API payload-v2 event has a specific shape. Use a representative fixture and test both the HTTP method/path and body handling. Keep your fixture focused on fields your handler is expected to consume.

### Hint 3 — Keep configuration non-secret

Use SAM parameters/environment variables for values like stage names and limits. Test at least two settings and prove they alter behavior. Environment variables are configuration, not a safe place to put secrets in source control.

### Hint 4 — Test boundaries, not implementation trivia

Turn each acceptance rule into a test: missing fields, invalid JSON, type/range boundaries, and each configuration. Prefer assertions on status and safe response shape over asserting private helper names or exact internal code structure.

### Hint 5 — Compare local and cloud evidence

`sam local` is useful for a fast feedback loop, but it cannot prove a deployed role's permissions, service-side API Gateway configuration, or CloudFormation behavior. State those limits clearly rather than overclaiming.

### Hint 6 — Inspect what will be uploaded

SAM packaging uploads template-referenced local artifacts to S3. Read the built artifact contents before approving the command. A bucket name and prefix are not sufficient evidence that the uploaded contents are safe.

### Hint 7 — Stop before deployment

The deliverable is a packaged template plus an uploaded artifact that is removed after validation. Avoid `sam deploy`, `sam sync`, or CloudFormation commands: they create cloud infrastructure outside this lab's scope.

### Hint 8 — Review AI-suggested tests

If you use an AI assistant to suggest tests, give it only the contract and synthetic examples. Check that each suggestion is relevant, deterministic, and safe; generated tests do not count as evidence until they run and pass.
