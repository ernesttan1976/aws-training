# Lab 08 — Progressive hints

Use these only for **Level 1**. Hints move from broad concepts to more concrete investigation prompts; stop when unstuck. The fault packet is issued only when the lab is activated. Levels 2–4 receive their own hints only when activated. Hints are not implementation or assessment evidence.

## Level 1 — Guided troubleshooting

### Hint 1 — Fix the incident window first

Write down when the symptom began, which route/request was involved, the deployment revision, and the exact stack/environment. An investigation without a shared time window is easy to mis-correlate.

### Hint 2 — Start at the request boundary

Compare the observed HTTP status and response with the API contract. Then follow one request through API Gateway, Lambda, and the application. Ask where the evidence first diverges from the expected path.

### Hint 3 — Use each evidence source for its strength

Logs carry event detail, metrics show rates and trends, traces show the path and relative time spent, and CloudFormation/deployment output describes resource changes and failures. No single green or red signal explains the whole incident.

### Hint 4 — Correlate without logging secrets

Use a request/correlation identifier and timestamps to connect API access records, Lambda logs, and trace IDs. Keep identifiers out of metric dimensions if they create high cardinality. Never solve correlation by logging the full request.

### Hint 5 — Query narrowly before broadening

Choose the relevant log group, a short time range, and fields that identify route, outcome, and failure class. Start with counts and representative events; widen the query only if the first result does not distinguish hypotheses.

### Hint 6 — Read the deployment timeline

Inspect the exact stack events around the failed action, including the first resource event that failed and the events that followed it. A terminal rollback message is usually less diagnostic than the earliest failure.

### Hint 7 — Form competing hypotheses

For every candidate cause, write one observation that would support it and one that would weaken it. Compare the same request window across metrics, traces, logs, and deployment output before editing.

### Hint 8 — Make one justified change

Capture the pre-fix evidence, change the smallest number of relevant variables, and repeat the same synthetic request. If multiple settings change at once, you lose the ability to show which change fixed the symptom.

### Hint 9 — Prove recovery, not just deployment

Check the API response, Lambda/API metrics, relevant logs, trace behavior, alarm history, and deployed stack state after the change. State what remains uncertain and verify that the production stack was untouched.

### Hint 10 — Keep telemetry useful and bounded

Use short log retention and short trace windows. A metric dimension should describe a small, stable category—not a unique request, order, or user. An alarm should detect a useful failure signal without being permanently noisy or too slow for the lab.
