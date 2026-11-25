# AWS Practical Skills Curriculum

**Framework:** AWS Certified Developer - Associate (DVA-C03)  
**Region preference:** `ap-southeast-1` when services are available  
**Progression:** One lab at a time; assess the learner before moving on.

Each lab has four exercise levels—guided implementation, independent implementation, troubleshooting, and optimization. Its assignment and supporting artifacts live together in its numbered folder under `labs/`. A lab may be prepared on request before it becomes active; preparation does not change the one-lab-at-a-time progression gate or mark skills demonstrated.

## Lab sequence

| No. | Lab | DVA-C03 mapping | Status |
|---:|---|---|---|
| 01 | [Reliable asynchronous ticket intake](labs/01-asynchronous-ticket-intake/LAB.md) | Domain 1 — Development with AWS Services; Tasks 1.1 and 1.2 (SQS, Lambda, SDKs, event-driven processing, retries) | **Active — Level 1** |
| 02 | [Data access patterns and lifecycle management](labs/02-data-access-patterns-lifecycle/README.md) | Domain 1 — Task 1.3 (DynamoDB/S3 selection, access patterns, indexes, TTL, lifecycle, caching) | Planned |
| 03 | [Secure application access and configuration](labs/03-secure-access-configuration/README.md) | Domain 2 — Tasks 2.1 and 2.2 (AWS credentials, IAM roles and resource permissions, encryption, secrets) | **Prepared — gated** |
| 04 | [Authenticated application API](labs/04-authenticated-application-api/README.md) | Domains 1 and 2 — API development and authentication/authorization (API Gateway, application identity, least privilege) | **Prepared — gated** |
| 05 | [Managed AI service integration with privacy controls](labs/05-managed-ai-privacy/README.md) | Domains 1 and 2 — Task 1.1 and Task 2.3 (AWS SDK integration, data privacy, input/output handling, access control) | **Prepared — gated** |
| 06 | [Package and test an AWS application](labs/06-package-and-test/README.md) | Domain 3 — Tasks 3.1 and 3.2 (SAM artifacts, environment configuration, test events, automated validation, AI-assisted testing) | Planned |
| 07 | [Release safely with CI/CD](labs/07-cicd-safe-releases/README.md) | Domain 3 — Task 3.3 (IaC, pipeline stages, deployment strategies, rollback, source control) | Planned |
| 08 | [Diagnose production-style failures](labs/08-troubleshooting-observability/README.md) | Domain 4 — Tasks 4.1 and 4.2 (integration/deployment troubleshooting, logs, metrics, traces, alerts) | Planned |
| 09 | [Optimize a working AWS application](labs/09-application-optimization/README.md) | Domain 4 — Task 4.3 (performance, concurrency, messaging, caching, cost and resource optimization) | Planned |

## Current progress

- **Active:** Lab 01, Level 1 — guided implementation.
- **Evidence:** No learner implementation or assessment evidence has been submitted yet.
- **Next gate:** Complete the active lab, submit runtime evidence, and explain the architecture and failure behavior. The next lab is selected after assessment.
- Labs 03, 04, and 05 were prepared on request; none is active until the preceding labs and assessment gates are completed.

## Official exam framework

- [DVA-C03 exam guide](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03.html)
- [Domain 1 — Development with AWS Services (30%)](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03-domain1.html)
- [Domain 2 — Security (26%)](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03-domain2.html)
- [Domain 3 — Testing and Deployment (22%)](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03-domain3.html)
- [Domain 4 — Troubleshooting and Optimization (22%)](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03-domain4.html)

Weights are those in the current official guide and may change when AWS revises the exam.
