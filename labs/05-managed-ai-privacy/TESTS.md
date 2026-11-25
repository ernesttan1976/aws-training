# Lab 05 — Validation and evidence

Do not run a live AWS request until the lab is activated, the profile/region and estimated cost are reviewed, and the learner explicitly approves the call. Use fabricated text only. Keep evidence sanitized: do not include original fixture text, detected substrings, raw request/response payloads, or credentials.

| Criterion | Validation procedure | Evidence to submit |
|---|---|---|
| AC1 — Correct service integration | Run unit tests with the SDK client mocked. After approval, make one live `DetectPiiEntities` call with the small synthetic fixture. Confirm the SDK v3 operation, configured region, and caller identity. | Sanitized test output; STS caller ARN/account redacted as appropriate; region; API operation and successful status without request text. |
| AC2 — Redacted result | Unit-test masking with at least two returned spans, spans near the beginning/end, and unaffected surrounding text. Confirm the emitted text has placeholders instead of detected values. | Test report and a sanitized output example using fictional/reserved values only. Do not include offsets paired with source text. |
| AC3 — Privacy boundary | Capture stdout/stderr for both a successful call and an API error. Search the captured output and test logs for a unique synthetic marker that existed only in the source fixture. Inspect the app for SDK request/debug logging and writes of raw input. | Sanitized console capture, test assertion that the marker is absent, and a brief data-flow note naming exactly what was sent and what was output. |
| AC4 — Least privilege | Review the active role/profile's effective permissions and the approved policy. Where available, simulate `comprehend:DetectPiiEntities` as allowed and a different Comprehend operation as denied. Document policy simulator limitations, permission boundaries, SCPs, session policies, or inherited grants that affect the result. | Sanitized policy excerpt or administrator confirmation; policy simulation output; explanation of the action/resource scope. Never include access-key material. |
| AC5 — Safe failures and limitations | Mock an authorization error, unsupported language, oversized input, malformed response, and a successful response with no detections. Assert no path returns or logs the original fixture. | Automated test results for each case and the user-facing generic failure/status text. |
| AC6 — Cost and cleanup | Count live calls and request sizes; inspect billing/usage if available after its reporting delay. Confirm no persistent AWS resources were created and any lab-only permission is removed or expires. | A small call-count/size ledger, cost estimate and assumption, cleanup checklist, and confirmation of no lab-only IAM grant remaining. |
| AC7 — Reasoning | Explain why the input is synthetic/minimized, what the API's result means, why a no-detection result is not a guarantee, and how the policy was constrained. | Learner's written explanation and answers to instructor follow-up questions. |

## Local test cases

1. Valid synthetic English text returns a sanitized result and a summary only.
2. Multiple non-overlapping detections are masked without changing unaffected text.
3. A fixture with no detections returns a neutral status, not a “PII-free” assertion.
4. Empty text, unsupported language, and text above the exercise's 2 KiB input limit are rejected before the SDK is called. Input text is not passed through command-line arguments or environment variables.
5. A mocked API authorization/service error produces a generic failure; source text and raw SDK error details are absent.
6. Malformed or inconsistent offsets fail safely and never emit raw input.
7. A sentinel string in the original fixture is absent from captured stdout/stderr after success and failure.
8. SDK client configuration uses the approved region and standard credential provider chain; no literal credentials occur in the repository.

## Evidence checklist

- [ ] No real personal, health, financial, account, or confidential information was used.
- [ ] No live API request happened before explicit approval.
- [ ] SDK call, region, identity, and permission evidence is sanitized.
- [ ] Redaction and fail-closed behavior are proven by automated tests.
- [ ] No raw input or credentials appear in logs, output, screenshots, or evidence.
- [ ] Request count, maximum request size, estimate, and cleanup are recorded.
- [ ] Learner explains service limitations and current content-use terms accurately.
