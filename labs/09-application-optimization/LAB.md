# Lab 09 — Optimize a working AWS application

**Status:** Prepared — gated; Lab 01 remains active pending learner evidence and assessment  
**Exercise when activated:** Level 1 — Guided optimization  
**Region:** `ap-southeast-1` (Singapore), unless a required service is unavailable

## Learning objectives

- Establish a controlled performance and cost baseline for a read-heavy serverless API.
- Use request-level measurements and AWS service telemetry to distinguish client-observed latency, Lambda execution time, throttling, errors, and cold starts.
- Compare Lambda memory settings using duration, latency, and an estimated invocation-cost model rather than optimizing one metric in isolation.
- Evaluate a bounded, per-execution-environment in-memory cache against an uncached DynamoDB lookup, including cache hit rate and stale-data exposure.
- Explain concurrency limits, the behavior of per-instance caches under concurrency, and why a short synthetic test is not a capacity guarantee.
- Make and document a measured configuration decision, preserve correctness and least privilege, and verify cleanup.

## Scenario

The Catalog Lookup team serves a compact reference catalog through a serverless API. Traffic is read-heavy, requests are synthetic and repeat a small set of item IDs, and the team has no evidence that the current Lambda memory setting or repeated DynamoDB reads are appropriate. The team needs a cost-aware performance decision, not a blind increase in capacity or a new always-on cache service.

## Problem statement

Build and optimize a small **Catalog Lookup** service with an API Gateway HTTP API, a Node.js Lambda function, and one on-demand DynamoDB table. `GET /items/{itemId}` returns a seeded synthetic catalog record by partition key. The Lambda starts with an uncached lookup and a modest baseline memory setting. Measure it under a bounded, repeatable workload, compare at least two Lambda memory settings, and evaluate a configurable per-execution-environment cache with a short TTL and a hard entry limit.

The final recommendation must be based on before/after evidence. It may retain the baseline if measurements show that an alternative does not justify its cost, complexity, latency, or freshness trade-off. No managed cache such as ElastiCache or DAX is part of this lab.

The API is read-only. Seed at most 20 synthetic items before measurement. A separate controlled data update may be used to test cache freshness; do not use personal or customer data. This is a short-lived sandbox exercise, not a production capacity test.

## Prerequisite knowledge

- Labs 01–08 assessed, including Lambda/API Gateway, DynamoDB access patterns, IAM roles, SAM/CloudFormation, automated testing, safe releases, and CloudWatch evidence. Lab 01 remains active until its assessment gate is met; this package does not activate Lab 09.
- Node.js, AWS SDK for JavaScript v3, AWS CLI named profiles/temporary credentials, AWS SAM CLI, CloudFormation, Lambda, API Gateway HTTP APIs, DynamoDB, and CloudWatch basics.
- Docker installed for any local SAM runtime testing; a small HTTP workload client capable of recording response times (for example, a documented Node.js script).
- A dedicated AWS sandbox account and temporary-credential deployment identity scoped to the lab resources.
- Review the official Lambda performance, concurrency, DynamoDB capacity, CloudWatch, and pricing references below before activation.

## Functional requirements

1. Provision a distinct Lab 09 stack with one API Gateway HTTP API, one Lambda function, one on-demand DynamoDB table, and only the IAM/logging/deployment resources required by SAM/CloudFormation. Tag or name resources so ownership is unambiguous.
2. Create a table keyed by `itemId` and seed no more than 20 synthetic records. Each record must have a small, documented response shape such as `itemId`, `label`, `category`, and `revision`. Do not use a table scan for the lookup API.
3. Implement `GET /items/{itemId}` using a DynamoDB key lookup. Return a stable success response for a known item and a clear `404` for an unknown item. Do not echo unsafe input, credentials, or internal errors.
4. Keep the initial handler uncached. Record the deployed source revision, configured memory/timeout, API response behavior, and baseline measurements before tuning.
5. Compare at least two Lambda memory settings. Use the same deployed code, table data, request set, concurrency, region, and measurement procedure for the compared runs. Record configured memory, observed maximum memory when available, client p50/p95 latency, Lambda duration, errors, throttles, cold-start observations, and request count.
6. Implement an in-process cache for evaluation. It must have an explicit enable/disable setting, a configurable short TTL, a hard maximum of 50 entries or fewer with a documented eviction behavior, and cache keys scoped to the item ID. Record cache hits/misses without high-cardinality metric dimensions or logging full request data.
7. Compare cache-disabled and cache-enabled behavior with repeated item IDs. Verify cache behavior across more than one execution environment where feasible and explain why this cache is local to each Lambda environment rather than shared globally.
8. Test freshness using a controlled synthetic record update made with a separate scoped test/deployment identity; keep the Lambda role read-only. Document how long a previously cached value may remain visible, observe expiry/invalidation behavior, and state whether that staleness is acceptable for the stated use case. Keep the maximum cache TTL at 30 seconds for this lab.
9. Keep reserved concurrency at or below 5 (or document a lower account-safe value), and use only a bounded synthetic workload: no more than 1,200 API requests total, concurrency no higher than 5, and no run longer than 10 minutes. Include a low-concurrency reference run and a short concurrency run; do not perform a sustained load or stress test.
10. Calculate and compare estimated Lambda invocation cost for the observed request volume/configurations using current regional pricing and measured duration. Include API Gateway, DynamoDB, CloudWatch logs/metrics, and any deployment artifact charges in the estimated lab total. State the assumptions; do not present estimates as actual billing.
11. Record all run parameters, raw or summarized sanitized results, and the final recommendation in `TESTS.md`. Keep account identifiers, endpoints where sensitive, credentials, and test data out of submitted evidence.

## Non-functional requirements

- Use temporary credentials for human/deployment access and a dedicated least-privilege Lambda execution role. The runtime role may read only the table and required log group; it must not have broad account access.
- Use DynamoDB `GetItem`/equivalent keyed access for the API path. Do not use scans, sustained load generators, or unbounded concurrency.
- Cache only synthetic, low-sensitivity records. Document the staleness bound and cache scope; do not imply an in-process cache provides cross-environment coherence.
- Keep logs structured and small. Do not log complete request events, credentials, request headers, or full records. Any custom metrics must use bounded dimensions such as operation/result, never item IDs or request IDs.
- Preserve the same API contract and data correctness in all compared variants. A lower latency result that increases errors, stale responses beyond the configured TTL, or violates the concurrency cap is not an acceptable optimization.
- Separate measured facts from estimates. Include variability and limitations such as cold starts, account contention, network time, low sample size, and service-side metric resolution.
- Confirm service state and actual deployed configuration. A successful SAM deployment, a single fast response, or a chart alone does not establish an optimization.

## Constraints and cost controls

- In scope: AWS SAM/CloudFormation, API Gateway HTTP API, Lambda, DynamoDB on-demand, CloudWatch Logs and the metrics needed to evaluate the workload, and S3 artifacts only if SAM requires them. No NAT gateway, database beyond the one DynamoDB table, custom domain, provisioned concurrency, API Gateway caching, DAX, ElastiCache, Step Functions, or third-party load service.
- Prefer `ap-southeast-1`. Verify service availability and current prices in-region. Do not silently move to another region.
- Use a short Lambda timeout (10 seconds or less), a log-retention period of seven days or less, reserved concurrency no higher than 5, no more than 20 seeded records, a cache TTL of 30 seconds or less, and at most 50 cache entries.
- Workload ceiling: 1,200 API requests total, concurrency at most 5, and each run at most 10 minutes. Stop when the approved request budget is reached, even if results are inconclusive.
- Keep the experiment to one baseline deployment and only the necessary tuning deployments/configuration updates. Reuse the same artifact where possible, and remove the stack promptly after evidence capture.
- **Estimated duration:** 3–4 hours for implementation, baseline, tuning runs, analysis, evidence, and cleanup, after prerequisite labs are assessed.
- **Estimated cost:** approximately **$0.25–$5 USD** under the bounded plan, depending on current regional API Gateway, Lambda, DynamoDB, CloudWatch, S3, and account pricing. This is a planning estimate, not a quote; do not assume free-tier eligibility. Review current prices and the planned resources before approval. Use **$5 USD as the maximum spend expectation** and pause if the plan could exceed it.
- Preparation creates no AWS resources and authorizes no spend. Before creating a stack, seeding data, or invoking the deployed API, review the account identity, region, exact resources, IAM scope, test bounds, estimated and maximum spend, duration, and cleanup steps with the learner and obtain explicit approval.

## Four exercise levels

### Level 1 — Guided implementation (first exercise when activated)

Build and measure the Catalog Lookup service using the milestones below and the progressive hints in `HINTS.md`. Compare the baseline against at least one memory alternative and evaluate the bounded cache. Complete `TESTS.md`, present the evidence and recommendation, and clean up.

### Level 2 — Independent implementation

After Level 1 is assessed, receive an outcome-only brief for a different read-heavy AWS application. Independently choose what to measure, which service settings or application-level optimization to test, and how to protect correctness and cost. The workload, constraints, and acceptance criteria will be supplied when Level 2 begins; no implementation design or optimization answer is given here.

### Level 3 — Troubleshooting

After Level 2 is assessed, investigate a supplied working-but-underperforming application and sanitized telemetry/workload evidence. Identify the smallest supported cause set, distinguish bottlenecks from noise, repair the issue, and demonstrate recovery without exceeding the supplied test limits. The faulty configuration and evidence packet will be issued when Level 3 begins; this document does not reveal them.

### Level 4 — Optimization

After Level 3 is assessed, optimize a working application against a supplied service objective and cost/performance baseline. Choose and justify changes across application behavior or AWS configuration, then show a repeatable improvement without weakening security, consistency, reliability, or workload safety. The workload and baseline will be supplied when Level 4 begins; no optimization recipe is provided in advance.

## Guided implementation milestones — Level 1

1. **Define the objective:** write down the expected request pattern, data freshness requirement, target metric, and what counts as a material regression. Identify what a short low-volume test cannot prove.
2. **Design the request path:** sketch client → API Gateway → Lambda → DynamoDB. Record the table key, Lambda role permissions, timeout, memory, concurrency cap, log retention, and expected cost sources.
3. **Build the smallest correct service:** deploy the read-only endpoint and seed synthetic records. Verify known-item, unknown-item, malformed-path, and response-safety behavior before measuring performance.
4. **Review and approve provisioning:** check the active identity/account, region, SAM change set, resource inventory, role policy, request ceilings, expected charge, and cleanup procedure. Obtain explicit approval before deployment or API invocation.
5. **Freeze the baseline procedure:** record code revision, configuration, dataset, exact synthetic request sequence, warm-up policy, concurrency, client location, timestamps, and how latency percentiles are calculated. Repeat the baseline run at least three times.
6. **Change one variable at a time:** compare memory settings with all other conditions held constant. Capture client and service observations; note cold starts and throttles. Do not claim cost savings from duration alone without calculating configured memory × billed duration.
7. **Evaluate caching:** test cache disabled/enabled on the same repeated-read workload. Capture hit/miss observations and data freshness after a controlled update. Explain per-environment scope and the consistency boundary.
8. **Select a final configuration:** compare p50/p95, Lambda duration, throttles/errors, memory use, freshness, and estimated cost/request. Choose the best evidence-supported trade-off; retaining baseline is valid when alternatives do not justify themselves.
9. **Validate, explain, and clean up:** complete all relevant rows in `TESTS.md`, explain the experiment design and limitations, then follow `CLEANUP.md` and verify the resources are gone.

## Acceptance criteria

1. **AC1 — Correct isolated service:** a distinct Lab 09 stack serves the documented synthetic read API, returns correct known/unknown results, uses keyed DynamoDB access, and leaves all unrelated stacks untouched.
2. **AC2 — Reproducible baseline:** the learner records the source revision, data, runtime settings, request procedure, concurrency, warm-up handling, client p50/p95, service-side duration, errors/throttles, and at least three baseline runs.
3. **AC3 — Memory decision:** at least two Lambda memory settings are measured under comparable conditions. The learner calculates the effect on latency and estimated invocation cost and explains the selected setting or why no change is justified.
4. **AC4 — Cache evaluation and correctness:** cache-off and cache-on behavior are compared on repeated reads, the cache has a documented eviction behavior, is bounded to 50 entries or fewer and TTL 30 seconds or less, and a controlled update demonstrates/establishes the freshness bound. The learner explains the per-execution-environment limitation.
5. **AC5 — Safe workload and operational evidence:** measured traffic stays within 1,200 requests, concurrency 5, and 10 minutes per run; Lambda reserved concurrency is no higher than 5; logs/metrics are sanitized and dimension cardinality is bounded.
6. **AC6 — Evidence-based recommendation:** the before/after table includes performance, reliability, freshness, and estimated cost. At least one candidate is evaluated against the baseline, and the final recommendation is supported by repeated evidence and limitations, not a single request or deployment success.
7. **AC7 — Security, cost, and cleanup:** deployed IAM is scoped, no secrets or personal/customer data are used, activity stays within the approved region and $5 expectation, Lab 09 resources are removed or explicitly justified as shared/retained, absence checks are recorded, and delayed billing review is noted.
8. **AC8 — Reasoning:** the learner explains the bottleneck hypothesis, controlled variables, percentile interpretation, memory/cost trade-off, concurrency behavior, cache freshness/scope, and what additional evidence would be needed before production rollout.

## Official references

- [DVA-C03 exam guide — Domain 4: Troubleshooting and Optimization](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03-domain4.html)
- [Task 4.3 — Optimize applications using AWS services and features](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03-domain4.html#dva-03-task4.3)
- [AWS Lambda best practices](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html)
- [Lambda function configuration — memory and timeout](https://docs.aws.amazon.com/lambda/latest/dg/configuration-memory.html)
- [Lambda concurrency and scaling](https://docs.aws.amazon.com/lambda/latest/dg/lambda-concurrency.html)
- [Lambda reserved concurrency](https://docs.aws.amazon.com/lambda/latest/dg/configuration-concurrency.html)
- [Lambda execution environment reuse](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtime-environment.html)
- [DynamoDB read/write capacity modes](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/on-demand-capacity-mode.html)
- [DynamoDB GetItem API](https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_GetItem.html)
- [CloudWatch metrics concepts and statistics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch_concepts.html)
- [CloudWatch Logs retention](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/Working-with-log-groups-and-streams.html)
- [AWS SAM Developer Guide](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html)
- [AWS IAM security best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [AWS Lambda pricing](https://aws.amazon.com/lambda/pricing/) · [API Gateway pricing](https://aws.amazon.com/api-gateway/pricing/) · [DynamoDB pricing](https://aws.amazon.com/dynamodb/pricing/) · [CloudWatch pricing](https://aws.amazon.com/cloudwatch/pricing/) · [S3 pricing](https://aws.amazon.com/s3/pricing/)

## Estimate

**Duration:** 3–4 hours after prerequisite assessments.  
**Cost:** approximately $0.25–$5 USD for the bounded short-lived exercise described above. Confirm current `ap-southeast-1` prices and account billing terms before approval; stop if the expected spend could exceed $5.

## DVA-C03 mapping

- **Domain 4 — Troubleshooting and Optimization** (22% in the current exam guide; verify the current guide before exam preparation).
- **Task 4.3 — Optimize applications using AWS services and features.** This lab emphasizes empirical performance/cost tuning, Lambda configuration and concurrency, data-access behavior, and caching trade-offs. Use the linked current official exam guide to confirm the exact skill statements and weighting.
- Reinforces Task 4.1/4.2 measurement and observability from Lab 08, plus Lambda/API, DynamoDB, IAM, and deployment skills from earlier labs. No production capacity claim is made from this bounded experiment.
