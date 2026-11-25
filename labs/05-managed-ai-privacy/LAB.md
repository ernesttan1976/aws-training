# Lab 05 — Managed AI service integration with privacy controls

**Status:** Prepared — gated; Lab 01 remains active pending learner evidence and assessment  
**Exercise when activated:** Level 1 — Guided implementation  
**Region:** `ap-southeast-1` (Singapore), unless a required feature is unavailable

## Learning objectives

- Call a managed AWS AI service from a Node.js application using AWS SDK for JavaScript v3 and temporary AWS credentials.
- Request the minimum Amazon Comprehend permission needed for PII detection and explain the service action's resource-scope limitations.
- Minimize and classify data before sending it to an AI service, understand the service-specific data handling terms, and avoid sending real personal or confidential information in this lab.
- Redact detected PII from text before printing, persisting, or otherwise using application output; treat detection as fallible rather than a complete privacy boundary.
- Keep content out of logs and errors, validate behavior with evidence, and remove any lab-only local or IAM artifacts safely.

## Scenario

A support team wants to route short customer messages to downstream tools. Before a message is displayed or used for analytics, the application should identify and mask common PII. The team needs a working managed-service integration and clear evidence about what data is sent, what is returned, and what is allowed into logs.

## Problem statement

Build a small local Node.js CLI called **Safe Intake**. It accepts one short text fixture, calls Amazon Comprehend's real-time `DetectPiiEntities` API, masks the returned PII spans, and prints only the redacted text plus non-sensitive metadata such as entity types and counts. Use AWS SDK for JavaScript v3 and the standard credential provider chain.

This is a privacy-control exercise, not a claim that PII detection is complete or that detected spans are safe to use without review. **Use only fabricated examples and synthetic PII** (for example, reserved example domains and clearly fake phone numbers). Do not paste real customer, employee, health, payment, or account data into the program or AWS console.

Amazon Comprehend is a managed AI service. Its FAQ states that the content-use-for-improvement practice described for Comprehend text inputs does not apply to Comprehend Detect PII; read the current service terms and FAQ before use. That exception is not a general guarantee of zero retention, zero processing, or regulatory compliance. This lab still prohibits real sensitive data and requires data minimization.

## Prerequisite knowledge

- Node.js, npm, JSON, AWS SDK for JavaScript v3, and basic CLI use.
- A dedicated AWS sandbox account and named profile using temporary credentials (prefer IAM Identity Center or an approved assume-role flow). Do not use root or create long-lived application keys.
- Permission to call `comprehend:DetectPiiEntities` in the selected region, or an administrator who can review/approve a narrowly scoped permission change. Do not broaden an existing role to make the lab work.
- Lab 01 remains the active assignment. This package is prepared for later use; do not call AWS or change IAM until the instructor activates the lab and the learner explicitly approves the expected charges and any IAM change.

## Functional requirements

1. Implement a CLI that reads text from standard input or a local fixture file; do not pass message contents as command-line arguments or environment variables. Keep live-test fixtures synthetic and no larger than 2 KiB each; reject empty input and input larger than the exercise's 2 KiB limit before sending it. The service's hard request limit is higher; the exercise limit is an intentional minimization control.
2. Use `DetectPiiEntities` with an explicit supported language code (use English for the required fixture) through the SDK v3 client. Configure the client to the approved region and use the SDK's default credential provider chain.
3. Build a sanitized result by masking the text spans identified by the service. Preserve unaffected text, handle multiple spans safely, and do not include the original text or span content in the result, logs, exception messages, or saved artifacts.
4. Output only the redacted text, an entity-type/count summary, and a simple status. Do not print entity offsets alongside the original message, request/response payloads, confidence values tied to identifiable text, credentials, or raw SDK errors.
5. Make privacy handling explicit: classify data before submission, minimize the payload, send it only to the selected regional endpoint, and make clear that detection can miss PII or produce false positives. Downstream systems must not use the unredacted input based solely on this demonstration.
6. Use a short-lived profile/role whose effective permission allows `comprehend:DetectPiiEntities` and no unrelated Comprehend operations for this exercise, subject to the account's permission boundaries. Do not request IAM administration permissions for the application. If the action cannot be restricted to a specific resource ARN, explain the wildcard resource required for that action while keeping the action list narrow.
7. Handle missing permission, unsupported language, oversized input, and service errors without logging the source text or returning an unredacted fallback. Fail closed: if detection or redaction fails, do not emit the original input as a substitute.
8. Keep request count small: no more than 12 live API requests, including retries. Use local unit tests for most cases; live calls are limited to the approved synthetic fixtures.

## Non-functional requirements

- No real PII, credentials, secrets, private customer text, or confidential data in source control, fixtures, CLI arguments, environment variables, logs, screenshots, or evidence. Keep temporary synthetic input files outside the repository.
- Use HTTPS service endpoints and temporary AWS credentials; never embed credentials or log request payloads.
- Store no original input in application logs or output files. Do not enable any content-logging middleware or SDK debug logging.
- Treat the managed detector as probabilistic and language/type limited. The application must not claim that a message is PII-free simply because no entities were returned.
- No long-running infrastructure, model endpoint, training job, S3 bucket, Lambda function, or other persistent AWS resource is needed for this lab.

## Constraints

- Use Node.js and AWS SDK for JavaScript v3; use the `@aws-sdk/client-comprehend` package and the real-time `DetectPiiEntities` operation.
- Run locally against the regional Comprehend endpoint in `ap-southeast-1`. The endpoint list includes Singapore; confirm current region and operation availability before starting.
- Do not use Comprehend custom models, training, asynchronous jobs, S3 input/output, Lambda, API Gateway, VPC endpoints, or any always-on resource in this exercise.
- Limit testing to 12 API calls and 2 KiB per request. Use local unit tests and mocked SDK responses for normal, edge, and failure cases.
- Do not change IAM or incur service charges without the learner's explicit approval. Ask the sandbox administrator for the minimum permission if needed; no IAM policy is to be created by this assignment itself.
- **Estimated duration:** 1.5–2.5 hours for implementation, privacy analysis, testing, and evidence.
- **Estimated cost:** approximately **$0–$0.05 USD** for up to 12 requests of at most 2 KiB each, using current public pricing as a planning basis and excluding any available free tier. The public pricing page describes 100-character billing units and a 300-character minimum per request, and lists $0.0001/unit for its example; regional prices, current pricing tiers, and account eligibility can differ. Check current Singapore pricing before approval. Keep a **$1 USD maximum spend expectation** and stop if the current estimate or actual usage exceeds it. No persistent infrastructure charges are expected.

## Four exercise levels

### Level 1 — Guided implementation (first exercise when activated)

Implement Safe Intake as specified above. Use the Level 1 hints in `HINTS.md`, validate against every criterion in `TESTS.md`, and explain the data flow, permission boundary, redaction behavior, and privacy limitations.

### Level 2 — Independent implementation

After Level 1 is assessed, design and implement a different managed-AI integration for a synthetic document workflow. Select an AWS service and operation available in the sandbox region, define the minimum data sent and output allowed to leave the application, and prove that errors cannot leak raw input. Requirements will be provided when Level 2 begins; do not reuse this exercise's solution design or receive a solution upfront.

### Level 3 — Troubleshooting

After Level 2 is assessed, diagnose a supplied deliberately faulty application and sanitized evidence packet. Symptoms may include overly broad access, raw input in logs, unsafe fallback behavior, incorrect span masking, or wrong-region requests. Identify each root cause, make the smallest safe repair, and prove it with tests and service evidence. The faulty starter and symptom packet will be issued when this level begins; no repair recipe is included here.

### Level 4 — Optimization

After Level 3 is assessed, optimize a working integration against a supplied synthetic workload and baseline. Measure API calls, latency, units processed, and application-side handling. Improve cost, latency, or exposure surface without weakening redaction and fail-closed behavior. Consider whether local safeguards or batching are appropriate; do not introduce paid infrastructure without approval.

## Guided implementation milestones — Level 1

1. **Review the service boundary:** read the PII detection guide, API reference, Comprehend FAQ privacy section, and pricing page. Record the allowed data class, selected region, API permission, request-size limit, and what the service documentation does and does not promise about content use.
2. **Plan the flow:** sketch local synthetic fixture/stdin → size/language validation → regional `DetectPiiEntities` call → in-memory masking → sanitized output. Pass only a fixture path (never message text) on the command line. Mark every boundary where raw content exists and confirm none writes to logs or disk.
3. **Verify identity and permission:** select the named sandbox profile, run an STS identity check, and verify `ap-southeast-1`. Have the effective policy reviewed for the single Comprehend action; request an approved change if necessary. Do not attach or broaden policies yourself unless explicitly authorized.
4. **Implement locally:** write the SDK call, safe span-masking logic, input validation, sanitized output, and fail-closed error behavior. Keep errors generic and useful without including input or raw service response.
5. **Test before a live call:** run local unit tests against synthetic examples and mocked responses, including adjacent/overlapping spans, no detections, malformed responses, and service failures. Inspect captured console/log output for the fixture string.
6. **Review and request approval:** state the maximum of 12 live requests, request sizes, likely cost, region, identity, data class, and cleanup plan. Do not send any live request until approval is explicit.
7. **Exercise and prove:** run the approved synthetic fixture, confirm the returned redacted text, inspect process output and AWS identity/region, and gather evidence in `TESTS.md`. Finish with `CLEANUP.md`.

## Acceptance criteria

1. **AC1 — Correct service integration:** the local app uses SDK v3 and `DetectPiiEntities` through the approved regional endpoint with temporary credentials; the successful call is evidenced without recording raw text.
2. **AC2 — Redacted result:** the synthetic fixture's returned PII detections are masked before output. Unit tests prove unaffected text is preserved and multiple detections do not corrupt offsets or expose spans.
3. **AC3 — Privacy boundary:** evidence shows only synthetic, minimized text was sent; original input and entity content are absent from stdout/stderr, logs, screenshots, and saved evidence. The learner accurately explains the current Comprehend Detect PII data-use statement and does not overclaim retention or compliance guarantees.
4. **AC4 — Least privilege:** the calling identity can perform the one required operation and has no unrelated Comprehend permissions for this lab, to the extent the account's effective policy controls permit. The learner explains why a wildcard resource may be needed for a non-resource-scoped action and identifies any account-level boundary not proven by policy simulation.
5. **AC5 — Safe failures and limitations:** invalid/oversized input, service failure, and no-detection cases never cause raw text to be emitted as a fallback. The app and evidence state that the detector can miss sensitive content and is not a complete privacy guarantee.
6. **AC6 — Cost and cleanup:** no persistent AWS resources were created; API use stayed within the approved call/size limit and cost expectation; any lab-only IAM grant is removed or expires, and local scratch material is deleted.
7. **AC7 — Reasoning:** the learner explains input minimization, regional processing, permission scope, data handling terms, redaction and failure semantics, and a measured cost/privacy trade-off.

## Official references

- [DVA-C03 exam guide — Domain 2: Security](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03-domain2.html)
- [DVA-C03 exam guide — Domain 1: Development with AWS Services](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03-domain1.html)
- [Amazon Comprehend — Detecting PII entities](https://docs.aws.amazon.com/comprehend/latest/dg/how-pii.html)
- [Amazon Comprehend — DetectPiiEntities API](https://docs.aws.amazon.com/comprehend/latest/APIReference/API_DetectPiiEntities.html)
- [Amazon Comprehend endpoints and quotas](https://docs.aws.amazon.com/general/latest/gr/comprehend.html)
- [Amazon Comprehend FAQ — data privacy](https://aws.amazon.com/comprehend/faqs/)
- [Amazon Comprehend IAM guidance](https://docs.aws.amazon.com/comprehend/latest/dg/security_iam_service-with-iam.html)
- [Amazon Comprehend actions and resource types in the IAM Service Authorization Reference](https://docs.aws.amazon.com/service-authorization/latest/reference/list_amazoncomprehend.html)
- [AWS SDK for JavaScript v3 — credential provider chain](https://docs.aws.amazon.com/sdk-for-javascript/v3/developer-guide/setting-credentials-node.html)
- [AWS SDK for JavaScript v3 API reference — Comprehend](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/client/comprehend/)
- [IAM policy evaluation logic](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html)
- [Amazon Comprehend pricing](https://aws.amazon.com/comprehend/pricing/)
- [AWS Organizations — managing AI services opt-out policy](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_ai-opt-out.html)

## Estimate

**Duration:** 1.5–2.5 hours, including local tests, a small approved live test, evidence, and cleanup.  
**Cost:** approximately $0–$0.05 USD for the constrained test volume under the pricing assumptions above. This is not a quote; verify current `ap-southeast-1` rates, free-tier status, and account billing before the live call.

## DVA-C03 mapping

- **Domain 2 — Security** (26% of scored content in the current guide; check for exam-guide updates).
- **Task 2.3 — Identify and mitigate security risks associated with using and integrating AI services into application development:** Skills 2.3.1 (request/manage AI-service access), 2.3.2 (privacy controls for data transmitted to AI services), 2.3.3 (filter/control AI inputs and outputs, including PII redaction), and 2.3.5 (protect sensitive content in monitoring logs).
- **Task 2.2 — Implement encryption and manage sensitive data by using AWS services:** Skill 2.2.6 (identify, mask, and limit access to sensitive data).
- **Domain 1 — Development with AWS Services:** Task 1.1, practical application integration through AWS SDK APIs.
