# Lab 05 — Progressive hints

Use these only for **Level 1**. Hints progress from conceptual to implementation-focused; pause at the first hint that gets you unstuck. Level 2–4 solution details will be supplied only as those levels are activated.

## Level 1 — Guided implementation

### Hint 1 — Start with the data boundary

Draw the flow before coding. Identify the exact moment raw text leaves the local process, the API response data you need, and what should be allowed into output. Keep all examples fabricated even if the service documentation shows realistic-looking samples.

### Hint 2 — Verify the AWS identity and region

Use a named profile backed by temporary credentials. Confirm the caller with STS and check that both the profile and SDK client target `ap-southeast-1`. A successful local unit test does not prove the AWS identity has service permission.

### Hint 3 — Ask for the smallest service permission

The runtime needs the `DetectPiiEntities` operation. Check the Amazon Comprehend service authorization reference before deciding whether the action supports resource-level scoping. If it does not, use the narrow action with the required wildcard resource; do not turn that into `comprehend:*`.

### Hint 4 — Separate detection from redaction

The API returns offsets and entity types rather than a pre-redacted real-time copy. Apply masks to the original string locally. Think about how changing an early span could invalidate offsets for later spans; make the algorithm safe for multiple detections and test with mocked offsets.

### Hint 5 — Make the safe path the only output path

Do not log the input before calling the service. If the API fails, return a generic error and no original text. If it finds nothing, report “no detections” rather than “PII-free”; the detector is not a complete guarantee.

### Hint 6 — Check content-use claims precisely

Read the Comprehend FAQ's data privacy section for the Detect PII exception, then read the relevant service terms. Do not generalize a statement about model improvement into a claim that content is never processed or retained. This lab uses synthetic data regardless.

### Hint 7 — Keep live calls bounded

The pricing page counts text in 100-character units with a per-request minimum. Use mocked tests for edge cases and keep real service calls to the approved short list; do not create an endpoint or training job.
