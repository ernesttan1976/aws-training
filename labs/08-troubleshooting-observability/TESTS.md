# Lab 08 — Validation and evidence

Lab 08 is prepared but gated. Do not create an SNS subscription, stack, API, Lambda function, alarm, dashboard, or run AWS requests until the curriculum gate is reached and the learner explicitly approves the reviewed identity, account, region, resources, notification path, request/deployment bounds, duration, maximum spend expectation, and cleanup plan. Use synthetic data only and sanitize all submitted evidence.

| Criterion | Validation procedure | Evidence to submit |
|---|---|---|
| AC1 — Isolated scope and baseline | Record caller identity/account and region before provisioning. Inspect the planned change set and tags. Deploy the distinct development stack; exercise `GET /health` and valid/invalid `POST /validate`; record revision, stack outputs, expected responses, and baseline metrics. Inspect production stack state before/after. | Sanitized identity/region, change-set and stack names, source revision, endpoint test summary, baseline metric screenshots/query output, and production unchanged evidence. Do not include credentials or unredacted account information. |
| AC2 — Safe, useful telemetry | Submit a synthetic request and inspect structured Lambda log events. Confirm correlation fields and useful outcome data while checking that request content/secrets are not logged. Verify the custom metric namespace/name/dimensions and log retention. | Redacted log event, metric definition/dimensions, retention setting, and source/revision reference. |
| AC3 — Observability coverage | For a single request or incident window, inspect API Gateway and Lambda metrics, Logs Insights results, X-Ray trace/service-map evidence, and the alarm state/history. If API-to-Lambda trace propagation is unavailable for the chosen integration, document the verification and limitation, then show the Lambda trace. | Time-aligned metric graphs/query results, redacted Logs Insights query/results, trace ID or sanitized trace screenshot, alarm history, and a short statement of what each source proves/does not prove. |
| AC4 — Alarm notification | Trigger the documented alarm condition using the approved bounded test. Verify state transition and SNS delivery through the approved subscription path; verify recovery after the condition clears. Avoid broad load tests. | Alarm configuration/history, notification receipt with private address/message details redacted, delivery path, and recovery timestamp. |
| AC5 — Root-cause analysis | Use the supplied symptom packet to identify one application/service-integration cause and one deployment/configuration cause. Correlate source timestamps and propose competing hypotheses before changing anything. Review first relevant CloudFormation failure events and deployment command output. | Incident timeline; hypotheses with supporting/contradicting evidence; sanitized log query, metrics, trace and stack event references; learner's cause statement and reason for rejecting an alternative. |
| AC6 — Verified repair | Deploy the minimal fix only to the Lab 08 development stack. Repeat the same health, valid, invalid, and fault-relevant synthetic requests. Inspect post-change logs, metrics, traces, alarm state/history, CloudFormation status, and production state. | Commit/revision, reviewed change summary, deployment result, before/after test results and telemetry, alarm recovery, and production unchanged evidence. |
| AC7 — Security, cost, and cleanup | Review IAM scopes and telemetry for sensitive data; compare actual request/deployment count and resources to the approved plan. Follow `CLEANUP.md`; verify stack/resource absence and record a delayed billing review date. | Sanitized permission review, resource/run ledger, cleanup evidence, absence checks, and billing review source/date. |
| AC8 — Reasoning | Explain the request path, evidence correlation, root causes, alternatives, alert trade-offs, and uncertainty. Answer assessor follow-up questions about the first failing component and why the selected correction is sufficient. | Learner explanation and assessor notes. |

## Minimum validation matrix

1. The Lab 08 application is deployed in an isolated development stack with no production changes.
2. Baseline health and validation route results are recorded for the deployed revision.
3. Structured logs provide a correlation identifier and outcome information without logging the full request or secrets.
4. At least one bounded-cardinality custom metric is visible; Lambda/API service metrics are inspected.
5. A trace for a synthetic request is inspected, or the chosen integration's tracing limitation is verified and documented without overstating the evidence.
6. A bounded alarm test produces an observable state transition and a notification through the approved SNS path; the alarm recovers.
7. A timestamped evidence chain supports the application/service-integration diagnosis and the deployment/configuration diagnosis.
8. The corrected revision passes the same synthetic request checks, and relevant telemetry supports recovery.
9. CloudFormation/SAM output is reviewed for the deployment diagnostic, including the earliest meaningful failed resource event.
10. No production stack was altered; AWS activity stayed within the approved request/run bounds and $5 maximum spend expectation.
11. Lab-created resources are removed or explicitly listed as shared/retained with justification; absence and billing follow-up are recorded.

## Evidence checklist

- [ ] Lab 08 was active and explicit provisioning approval was recorded before creating resources or invoking the API.
- [ ] Account, caller identity, region, notification path, maximum spend expectation, request/deployment bounds, duration, and cleanup plan were approved.
- [ ] A distinct development stack and revision are recorded; production remained unchanged.
- [ ] Baseline and post-fix health/validation responses are captured using synthetic input.
- [ ] Structured logs, custom metric, service metrics, trace/limitation, and alarm notification/recovery are evidenced.
- [ ] The timeline supports diagnoses for both requested fault categories and includes at least one rejected hypothesis.
- [ ] Deployment output and CloudFormation events are sanitized and reviewed.
- [ ] No credentials, private notification endpoint, or sensitive request data are present in the evidence.
- [ ] Request/deployment volume and resource inventory remained within the approved plan.
- [ ] Lab resources were deleted or explicitly justified as retained/shared; absence and delayed billing review are recorded.
- [ ] Learner explains evidence limitations and remaining uncertainty.
