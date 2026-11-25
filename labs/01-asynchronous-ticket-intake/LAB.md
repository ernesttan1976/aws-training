# Lab 01 — Reliable asynchronous ticket intake

**Status:** Ready to start · **Current exercise:** Level 1 — Guided implementation  
**Region:** `ap-southeast-1` (Singapore), unless a required feature is unavailable

## Learning objectives

- Send messages to Amazon SQS from application code using AWS SDK for JavaScript v3.
- Connect an SQS queue to an AWS Lambda function with an event source mapping.
- Explain acknowledgement, retry, visibility-timeout, and dead-letter-queue behavior.
- Apply least-privilege permissions and verify behavior with AWS evidence.

## Scenario

A small support team receives ticket submissions in bursts. A synchronous intake service should not have to wait for every ticket to be processed, and a malformed ticket must not block valid work indefinitely. You are building a small, asynchronous intake path for a prototype service.

## Problem statement

Build a command-line producer that submits support-ticket messages to an SQS Standard queue. A Lambda consumer validates each message and emits a structured acceptance record to CloudWatch Logs. Configure a dead-letter queue (DLQ) so messages that repeatedly fail can be inspected without retrying forever.

This lab deliberately stops at observable processing; it does not require an API, database, or production ticketing system.

## Prerequisite knowledge

- Node.js and TypeScript or JavaScript; npm; basic AWS CLI use.
- An AWS sandbox account and a named AWS CLI profile with permission to deploy and inspect this lab's resources.
- Do not use root credentials, put credentials in source control, or create access keys for the application.
- Read the AWS SDK v3 credential-provider guidance before running the producer.

## Functional requirements

1. Provision one SQS Standard work queue and one SQS Standard DLQ.
2. Configure redrive so a message is moved to the DLQ after a small, finite number of failed receives.
3. Deploy one Lambda consumer, invoked by an SQS event source mapping. Use batch size 1 for deterministic first-pass behavior.
4. The producer sends a JSON ticket containing a unique `ticketId`, a non-empty `category`, and a non-empty `message`.
5. The consumer validates these fields, logs a structured accepted record for valid messages, and fails invalid messages so they can be retried and eventually sent to the DLQ.
6. Use separate, least-privilege permissions for sending messages and for the Lambda consumer. Keep all resource names and configuration in deployment parameters or environment configuration, not hard-coded credentials.

## Non-functional requirements

- A valid message is processed asynchronously; the producer does not invoke Lambda directly.
- The configured visibility timeout is at least six times the Lambda timeout, as recommended for an SQS event source mapping.
- Failed messages have a bounded retry path and are observable in the DLQ.
- Keep logs useful and avoid logging secrets or unnecessary ticket message contents.
- Use a short log-retention period for this temporary lab.

## Constraints

- Use AWS SAM for infrastructure and deployment, AWS CLI for inspection, and Node.js with AWS SDK for JavaScript v3 for the producer. The Lambda runtime may use the built-in AWS SDK only if you document the choice; otherwise bundle the SDK dependency.
- Use only SQS, Lambda, CloudWatch Logs, and the deployment resources SAM requires. No NAT gateway, API Gateway, database, or always-on compute.
- Use `ap-southeast-1` where available. Do not deploy anything billable until you have explicitly approved provisioning.
- Target at most 20 test messages and a one-hour live-resource window. Set an AWS Budget or other spend alert if one is already available in the sandbox; do not create a paid monitoring resource solely for this lab.
- Estimated infrastructure cost: **$0 to about $0.10 USD** for the stated small, short-lived test in Singapore, assuming no NAT gateway and modest log volume. This is an estimate, not a quote; account free-tier eligibility, CloudWatch usage, deployment artifacts, and current regional prices can change it. Set a **$1 USD maximum spend expectation** for this exercise and stop if estimates exceed it.

## Level progression for this competency

- **Level 1 — Guided implementation (current):** implement this lab using the milestones below and the hints in `HINTS.md`.
- **Level 2 — Independent implementation:** implement the same outcome from requirements alone, using a different small event such as an image-processing request. Do not reuse the Level 1 deployment template. Explain your retry and permission choices.
- **Level 3 — Troubleshooting:** diagnose a deliberately faulty version of this pipeline. The exercise will be issued when Level 3 begins; it will include a broken implementation and observable symptoms, not a solution.
- **Level 4 — Optimization:** improve a working pipeline for higher throughput and operational clarity. Measure before and after; consider batching, partial batch responses, idempotency, and log volume. Do not add paid services without approval.

## Guided implementation milestones — Level 1

1. **Plan:** sketch producer → SQS work queue → Lambda event source mapping → structured log; show the DLQ branch for repeated failure. Identify who needs which permissions.
2. **Bootstrap:** verify the active AWS profile and region, inspect the SAM template/deployment plan, and confirm no unintended resources (especially NAT gateways) are included.
3. **Implement:** define the queues, redrive policy, Lambda function, event source mapping, timeout/visibility-timeout relationship, log retention, and least-privilege roles. Implement the producer and validation behavior.
4. **Review before deployment:** explain what happens after a successful invocation, a function error, and a message that exceeds the receive threshold. Check the estimated spend and request provisioning approval before deploying.
5. **Deploy and exercise:** deploy the stack only after approval. Submit one valid and one invalid message. Observe successful processing and the invalid message's bounded retry/DLQ path.
6. **Prove and clean up:** collect the evidence in `TESTS.md`, delete the stack and any lab-only deployment artifact bucket you created, then complete the cleanup verification in `CLEANUP.md`.

## Acceptance criteria

1. **AC1 — Asynchronous producer:** a CLI invocation sends a valid ticket to the work queue and reports success without invoking Lambda directly.
2. **AC2 — Successful processing:** the valid ticket is removed from the work queue and a structured acceptance record is visible in the Lambda's CloudWatch log stream.
3. **AC3 — Bounded failure:** an invalid ticket is retried only up to the configured receive threshold and then appears in the DLQ; a valid ticket continues to process.
4. **AC4 — Security:** the producer identity can send to the work queue but cannot administer it; the Lambda role has only the queue operations and log permissions it needs. No credentials are committed.
5. **AC5 — Safe operation:** the deployed resources match the approved region/scope, costs remain within the stated expectation, and cleanup verification shows the lab resources are removed.

## Official references

- [DVA-C03 exam guide — Domain 1: Development with AWS Services](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03-domain1.html)
- [AWS SAM Developer Guide](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html)
- [Using Lambda with Amazon SQS](https://docs.aws.amazon.com/lambda/latest/dg/with-sqs.html)
- [Configuring an SQS event source mapping](https://docs.aws.amazon.com/lambda/latest/dg/services-sqs-configure.html)
- [Using dead-letter queues in Amazon SQS](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-dead-letter-queues.html)
- [AWS SDK for JavaScript v3 — SQS examples](https://docs.aws.amazon.com/sdk-for-javascript/v3/developer-guide/javascript_sqs_code_examples.html)
- [AWS SDK for JavaScript v3 — credential provider chain](https://docs.aws.amazon.com/sdk-for-javascript/v3/developer-guide/setting-credentials-node.html)
- [AWS Lambda pricing](https://aws.amazon.com/lambda/pricing/) · [Amazon SQS pricing](https://aws.amazon.com/sqs/pricing/) · [CloudWatch pricing](https://aws.amazon.com/cloudwatch/pricing/)

## Estimate

**Duration:** 2–3 hours, including deployment, evidence collection, and cleanup.  
**Cost:** see the estimate and assumptions under Constraints. Verify current prices for `ap-southeast-1` before deployment.

## DVA-C03 mapping

- **Domain 1 — Development with AWS Services** (30% of scored content in the current guide)
- **Task 1.1:** Develop code for applications hosted on AWS — especially Skills 1.1.1 (decoupling), 1.1.2 (asynchronous patterns), 1.1.3 (fault tolerance and retries), 1.1.5 (AWS APIs/SDKs), and 1.1.8 (event-driven patterns).
- **Task 1.2:** Develop code for AWS Lambda — especially Skill 1.2.3 (event lifecycle and errors) and Skill 1.2.4 (integrate Lambda with AWS services).
