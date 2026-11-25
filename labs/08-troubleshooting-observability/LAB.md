# Lab 08 — Diagnose production-style failures

**Status:** Prepared — gated; Lab 01 remains active pending learner evidence and assessment
**Exercise when activated:** Level 1 — Guided troubleshooting
**Region:** `ap-southeast-1` (Singapore), unless a required service or integration is unavailable

## Learning objectives

- Establish a baseline for a serverless API using health checks, structured logs, service metrics, custom metrics, alarms, and traces.
- Use CloudWatch Logs Insights, CloudWatch metrics, X-Ray, and deployment events to narrow a symptom to a testable cause.
- Separate application defects from API integration, IAM, configuration, and deployment failures.
- Make the smallest safe correction in an isolated development stack and verify recovery end to end.
- Explain what each evidence source proves, its limitations, and how to avoid leaking sensitive request data into telemetry.
- Remove lab-created resources and verify cleanup.

## Scenario

The Order Check team has released a serverless validation API to a development environment. A new release has produced intermittent 5xx responses and an unhealthy request path, while the deployment output contains warnings and failures that are not yet understood. The on-call engineer must identify the causes quickly and leave a clear evidence trail without touching production.

## Problem statement

Use the assessed Order Check application from Labs 06–07 as the workload, deploying it into a fresh, isolated development stack. The instructor will provide a deliberately faulty revision and a sanitized symptom packet when Lab 08 is activated. Instrument the stack with structured Lambda logs, API Gateway and Lambda service metrics, at least one application-level custom metric, CloudWatch alarms with an SNS notification path, and AWS X-Ray tracing. Investigate the supplied symptoms across these evidence sources, record a ranked root-cause explanation, repair the faulty development revision, and verify recovery with synthetic API requests.

The exercise includes a controlled deployment/configuration failure diagnostic as well as an application/service-integration diagnostic. Any failure injection is limited to the lab's development stack and synthetic test requests. Production stacks, data, and credentials are out of scope. The fault packet identifies observable symptoms and time windows but does not identify the faulty components or fixes.

## Prerequisite knowledge

- Labs 06 and 07 assessed: Order Check source, SAM/CloudFormation deployment, API Gateway, Lambda, IAM roles, and basic release/rollback behavior. These labs remain prepared and gated; Lab 01 is still the active assignment.
- Node.js, AWS CLI with named profiles/temporary credentials, CloudFormation, API Gateway REST APIs, Lambda, CloudWatch, and basic X-Ray concepts.
- AWS SAM CLI and Docker installed locally; Git access to the assessed Order Check source.
- A dedicated AWS sandbox account and a temporary-credential identity authorized for the explicitly approved lab resources.
- A learner-approved email endpoint for the SNS alarm notification, if email notification is used. Do not commit the address or use a personal address without approval.
- Review the official CloudWatch Logs Insights, CloudWatch alarms, Lambda/X-Ray, API Gateway tracing, CloudFormation events, and pricing references below before activation.

## Functional requirements

1. Deploy a **new development-only** Order Check stack. Use a distinct stack name, API endpoint, log groups, alarm names, and resource tags identifying Lab 08. Do not reuse or modify a production stack.
2. Record a healthy baseline for `GET /health` and `POST /validate`: expected HTTP status/body, Lambda invocations/errors/duration/throttles, API Gateway request/error/latency metrics where available, and a representative X-Ray trace.
3. Emit structured application logs with a correlation identifier and safe diagnostic fields (route, outcome, stage, and duration as relevant). Do not log full request bodies, tokens, credentials, or sensitive values. Preserve enough context to correlate an API request with its Lambda log events and trace.
4. Emit at least one useful application-level custom metric for a business outcome or failure class. Include a dimension strategy that avoids unbounded/high-cardinality dimensions such as request IDs, order IDs, or user IDs.
5. Enable Lambda active tracing and API Gateway stage tracing where supported by the selected API type. Verify that a synthetic request produces a trace or service-map evidence. If an integration does not propagate a trace, document the limitation and use the Lambda trace as the service-side evidence; do not claim an absent trace proves the fault is absent.
6. Configure a CloudWatch alarm against a meaningful health signal and connect its alarm action to an SNS topic. Verify one notification using the learner-approved endpoint, or document and validate the approved non-email subscription path. Keep thresholds, evaluation periods, and test traffic bounded.
7. Use the instructor-provided fault packet to investigate at least one application/service-integration symptom and one deployment/configuration symptom. Build a timestamped timeline, query the relevant logs, inspect metrics and alarm history, inspect traces, and review SAM/CloudFormation output/events. Record hypotheses and evidence before changing the code or configuration.
8. Apply the smallest correction to the development revision. Deploy using the approved SAM/CloudFormation workflow, verify the change-set/deployment result, and run healthy and invalid synthetic requests. Confirm the expected HTTP behavior and that alarms/metrics/logs/traces reflect recovery.
9. Capture sanitized evidence for every criterion in `TESTS.md`. Do not submit account IDs where avoidable, endpoint secrets, email addresses, tokens, environment dumps, complete event payloads, or trace/log fields containing sensitive data.

## Non-functional requirements

- Use least-privilege temporary credentials for the human identity and scoped service roles for Lambda, API Gateway logging/tracing, and deployment. Review the effective policy before deployment.
- Keep all diagnostics in the isolated development stack. Use synthetic order data only, low request volume, short function timeouts, and short log retention (three days or less).
- Use structured logs with a stable schema and avoid logging sensitive input. Treat traces and logs as potentially sensitive operational data.
- The fix must be supported by evidence, be minimal, and preserve the documented Order Check API contract. Do not suppress alarms, disable logging/tracing, broaden IAM, or weaken validation merely to produce a green check.
- Verify service state directly. A successful `sam deploy`, a green alarm, a single successful request, or a dashboard screenshot by itself is not proof of end-to-end recovery.

## Constraints and cost controls

- In scope: AWS SAM/CloudFormation, one API Gateway REST API (or the assessed API type if its observability features are verified), one Lambda function, CloudWatch Logs/metrics/alarms/dashboard as needed, one SNS topic/subscription, and AWS X-Ray.
- No production deployments, databases, NAT gateways, custom domains, sustained load tests, paid third-party observability, or unrelated resources.
- Prefer `ap-southeast-1`. Confirm the selected services and API tracing support in-region; do not change region without recording why and obtaining approval.
- Use a three-day-or-shorter log retention period, bounded X-Ray sampling and short capture windows, no more than 100 synthetic API requests in total, a small Lambda memory/timeout configuration, and one alarm/dashboard unless a reviewed need is documented.
- Keep this lab's request volume and deployment exercises to one baseline deployment, one controlled failing deployment/diagnostic, one corrected deployment, and only necessary retries. Do not use production traffic or real customer data.
- **Estimated duration:** 3–4 hours for instrumentation, baseline, investigation, repair, validation, evidence, and cleanup after Labs 06–07 are assessed.
- **Estimated cost:** approximately **$0.25–$5 USD** under the bounded plan above, depending on CloudWatch log ingestion/retention, custom metrics/alarms/dashboard, X-Ray trace volume, API Gateway/Lambda requests, regional rates, and account pricing terms. This is an estimate, not a quote; check current `ap-southeast-1` prices. Use **$5 USD as the maximum spend expectation** and pause if the reviewed plan may exceed it. Do not assume free-tier eligibility.
- No AWS connection, subscription, stack, alarm, dashboard, or API deployment may be created before Lab 08 is active and the learner explicitly approves the account, identity, region, exact resources, notification endpoint/path, request/deployment bounds, maximum spend expectation, duration, and cleanup plan.

## Four exercise levels

### Level 1 — Guided troubleshooting (first exercise when activated)

Investigate the supplied sanitized fault packet using the guided milestones and progressive hints in `HINTS.md`. The fault packet is issued only when the lab is activated. Complete all checks in `TESTS.md`, explain the causes and evidence, and clean up.

### Level 2 — Independent implementation

After Level 1 is assessed, receive an outcome-only incident brief for a different AWS application. Choose the evidence sources and investigation sequence, identify and repair the causes, and demonstrate recovery. The application, symptoms, and acceptance criteria will be supplied when Level 2 is activated; no diagnosis or solution design is provided upfront.

### Level 3 — Troubleshooting

After Level 2 is assessed, diagnose a new fault set from sanitized logs, metrics, traces, alarm history, and deployment output. Several symptoms may be correlated or misleading. Identify the smallest sufficient root-cause set, make minimal repairs, and substantiate the result. The fault set and evidence packet will be provided when Level 3 begins; this assignment does not reveal the faults or repairs.

### Level 4 — Optimization

After Level 3 is assessed, improve the observability of a working application against a supplied baseline. Reduce diagnostic time, telemetry noise, or telemetry cost while preserving signal, privacy, alert usefulness, and recovery verification. The workload and baseline will be provided when Level 4 is activated.

## Guided implementation milestones — Level 1

1. **Review the incident boundary:** identify the development stack, test window, synthetic request set, account and region. Draw the request path from API Gateway through Lambda and its telemetry. Mark production and user data as out of scope.
2. **Establish the baseline:** deploy the known-good application once, record its revision and stack outputs, exercise health and validation routes, and write down expected status codes and service metrics.
3. **Instrument safely:** add correlation-friendly structured logs, one bounded-cardinality custom metric, Lambda tracing, API access/execution tracing where supported, an alarm, and an SNS action. Keep retention, sampling, and request volume within the cost controls.
4. **Review before provisioning:** verify caller identity, account, region, stack changes, permissions, log retention, alarm/SNS destinations, trace configuration, expected charges, run bounds, and cleanup. Obtain explicit approval before creating or invoking AWS resources.
5. **Build an incident timeline:** align the symptom timestamps with request outcomes, Lambda/API metrics, alarm state transitions, log events, trace segments, and deployment events. Capture each source's time range and query/metric dimensions.
6. **Test hypotheses:** explain how each observation supports or weakens candidate causes. Distinguish where the failure enters the request path and whether the deployment itself completed. Do not change multiple variables before collecting baseline evidence.
7. **Repair and verify:** apply the smallest supported fix to the development revision, review the change set, deploy, and repeat the same synthetic requests. Compare before/after behavior and telemetry; verify the production stack was not touched.
8. **Report and clean up:** complete `TESTS.md`, explain uncertainty and evidence limits, then follow `CLEANUP.md` and verify resource removal and delayed billing review.

## Acceptance criteria

1. **AC1 — Isolated scope and baseline:** a distinct Lab 08 development stack and revision are recorded; health and validation routes have documented baseline results; no production resource was modified.
2. **AC2 — Safe, useful telemetry:** structured logs correlate a synthetic request without exposing sensitive inputs; at least one useful custom metric is emitted with bounded dimensions; retention and trace sampling are limited.
3. **AC3 — Observability coverage:** Lambda/API metrics and alarm history, a trace or documented tracing limitation, and searchable logs are captured for the same request or incident window. The learner explains what each source can and cannot establish.
4. **AC4 — Alarm notification:** a meaningful CloudWatch alarm transitions on a controlled, bounded signal and its SNS notification path is verified using the approved endpoint/path; alarm recovery is recorded.
5. **AC5 — Root-cause analysis:** the learner identifies causes for both an application/service-integration symptom and a deployment/configuration symptom, supports each with a time-aligned evidence chain, and rejects at least one plausible but unsupported hypothesis.
6. **AC6 — Verified repair:** the smallest justified development-only fix is deployed successfully; health and valid/invalid synthetic requests meet the API contract; logs, metrics, traces, and alarm state support recovery; production remains unchanged.
7. **AC7 — Security, cost, and cleanup:** no credentials or sensitive inputs appear in submitted telemetry; activity stays within the reviewed region, request/deployment limits, and $5 expectation; Lab 08 resources are deleted or explicitly documented as shared/retained, with absence checks and billing follow-up.
8. **AC8 — Reasoning:** the learner explains the diagnostic sequence, correlation strategy, root causes, competing hypotheses, service boundaries, alert trade-offs, and remaining uncertainty.

## Official references

- [DVA-C03 exam guide — Domain 4: Troubleshooting and Optimization](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03-domain4.html)
- [Task 4.1 — Assist in a root cause analysis](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03-domain4.html#dva-03-task4.1)
- [Task 4.2 — Apply logging, monitoring, and observability best practices](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03-domain4.html#dva-03-task4.2)
- [CloudWatch Logs Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/AnalyzingLogData.html)
- [CloudWatch Logs Insights query syntax](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CWL_QuerySyntax.html)
- [CloudWatch metrics and dimensions](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch_concepts.html)
- [CloudWatch alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html)
- [CloudWatch Embedded Metric Format](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Embedded_Metric_Format.html)
- [AWS Lambda X-Ray tracing](https://docs.aws.amazon.com/lambda/latest/dg/services-xray.html)
- [API Gateway X-Ray tracing](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-xray.html)
- [AWS X-Ray sampling](https://docs.aws.amazon.com/xray/latest/devguide/xray-console-sampling.html)
- [Amazon SNS email subscriptions](https://docs.aws.amazon.com/sns/latest/dg/sns-email-notifications.html)
- [View CloudFormation stack events](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/view-stack-events.html)
- [AWS SAM CLI — `sam deploy`](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-deploy.html)
- [AWS IAM security best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [CloudWatch pricing](https://aws.amazon.com/cloudwatch/pricing/) · [X-Ray pricing](https://aws.amazon.com/xray/pricing/) · [API Gateway pricing](https://aws.amazon.com/api-gateway/pricing/) · [Lambda pricing](https://aws.amazon.com/lambda/pricing/) · [SNS pricing](https://aws.amazon.com/sns/pricing/)

## Estimate

**Duration:** 3–4 hours, after Labs 06–07 are assessed.  
**Cost:** approximately $0.25–$5 USD for the constrained development-only run described above. Confirm current regional prices and the account's billing terms before approval; stop if the approved plan may exceed $5.

## DVA-C03 mapping

- **Domain 4 — Troubleshooting and Optimization** (22% in the current exam guide; check the guide before exam preparation).
- **Task 4.1 — Assist in a root cause analysis:** Skills 4.1.1–4.1.4: debug application and integration defects; interpret metrics, logs, and traces; query logs; troubleshoot deployments using output logs. Skill 4.1.5 (AWS AI troubleshooting tools) is not required for this lab.
- **Task 4.2 — Apply logging, monitoring, and observability best practices:** Skills 4.2.1–4.2.5: structured logs, custom metrics, alert notifications, tracing, and health monitoring.
- This lab emphasizes evidence-led diagnosis and operations; performance/cost optimization is reserved for Lab 09.
