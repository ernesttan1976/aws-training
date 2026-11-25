# Lab 09 — Validation and evidence

Lab 09 is prepared but gated. Do not create a stack, seed DynamoDB, invoke a deployed endpoint, or run workload measurements until the curriculum gate is reached and the learner explicitly approves the account, identity, region, resource plan, request/concurrency/time bounds, maximum spend expectation, duration, and cleanup plan. Use synthetic records only. Sanitize all submitted evidence.

## Acceptance-criteria evidence matrix

| Criterion | Validation procedure | Evidence to submit |
|---|---|---|
| AC1 — Correct isolated service | Record the approved account/region and planned stack resources. Inspect the SAM template/change set and IAM scope. Exercise a seeded known ID, an unknown ID, and malformed path/input behavior. Inspect the data-access implementation or telemetry to confirm keyed lookup and no scan. | Sanitized identity/region, resource/change-set summary, source revision, API result table, and evidence of keyed table access. Show that unrelated stacks were not modified. |
| AC2 — Reproducible baseline | Freeze the run plan. Run the same bounded request set at least three times with baseline code/settings. Record request count, concurrency, client location, warm-up policy, timestamps, p50/p95, Lambda duration, errors/throttles, and cold-start observations. | Run sheet, three baseline result summaries, configuration/revision, and supporting sanitized API/CloudWatch output. |
| AC3 — Memory decision | Deploy or configure at least two Lambda memory settings with all other variables held constant. Repeat the run procedure. Calculate estimated Lambda compute cost at observed invocation count and compare latency, duration, memory, errors, and throttles. | Before/after configuration, repeated result table, cost formula/assumptions, and explanation of the selected setting or decision to retain baseline. |
| AC4 — Cache evaluation and correctness | Compare cache disabled/enabled on repeated IDs. Verify configured TTL, documented eviction behavior, and entry cap (50 or fewer). Update one synthetic record using the separate scoped test/deployment identity after a cache fill; verify the stored revision directly (use a strongly consistent read where applicable), then observe/document the maximum stale period and refreshed API value. Observe or explain cache behavior across execution environments. | Sanitized cache configuration, hit/miss evidence without item-ID dimensions, update/freshness timeline, response revisions, and explanation of per-environment cache scope. |
| AC5 — Safe workload and operational evidence | Reconcile all test runs against the request ledger: no more than 1,200 total calls, concurrency at most 5, and no run longer than 10 minutes. Inspect reserved concurrency, timeout, log retention, logs, and metric dimensions. | Run ledger, relevant settings, redacted structured logs/metric definitions, and confirmation that all ceilings were respected. |
| AC6 — Evidence-based recommendation | Compare baseline and candidate results across performance, reliability, freshness, and estimated cost. Use repeated measurements, state uncertainty, and identify the objective/trade-off behind the recommendation. | Final comparison table and short recommendation with evidence links/attachments, assumptions, limitations, and criteria for revisiting the decision. |
| AC7 — Security, cost, and cleanup | Review runtime IAM and test data; reconcile actual resources/activity with the approved plan. Follow `CLEANUP.md`; inspect CloudFormation and service APIs for absence and schedule delayed billing review. | Sanitized policy/resource review, approved-vs-actual ledger, cleanup command/result evidence, absence checks, and billing review source/date. |
| AC8 — Reasoning | Explain the bottleneck hypothesis, controlled variables, percentile method, memory/cost calculation, concurrency and cold-start interpretation, cache staleness/scope, and limitations of the test. | Learner explanation and assessor notes. |

## Minimum validation matrix

1. The Lab 09 development stack is isolated, and the API returns the right results for a known item and unknown item.
2. The handler uses a keyed DynamoDB lookup, not a table scan.
3. Three or more comparable baseline runs are recorded with test conditions and client p50/p95.
4. Two or more Lambda memory settings are compared under equivalent conditions with duration, memory, errors/throttles, and estimated invocation cost considered.
5. Cache-off and cache-on behavior are evaluated; the cache's configured TTL, bounded entry count, and eviction behavior are verified.
6. Cache size is bounded to 50 entries or fewer, TTL is 30 seconds or less, and a controlled update establishes the stale-data behavior.
7. Traffic remains within 1,200 total requests, concurrency 5, and 10 minutes per run; reserved concurrency is at most 5.
8. Evidence supports a recommendation across latency, reliability, freshness, and cost; sources of variation and what the test cannot prove are stated.
9. No secret, personal information, customer data, or high-cardinality metric dimension appears in submitted evidence.
10. Lab-created resources are removed or explicitly justified as shared/retained; absence checks and delayed billing review are recorded.

## Run record template

| Run | Revision | Memory (MiB) | Cache / TTL | Dataset | Requests | Concurrency | Warm-up | p50 (ms) | p95 (ms) | Lambda duration | Errors / throttles | Cache hit/miss | Estimated Lambda cost | Notes |
|---|---|---:|---|---:|---:|---:|---|---:|---:|---:|---|---|---:|---|
| Baseline 1 | | | | | | | | | | | | | | |
| Baseline 2 | | | | | | | | | | | | | | |
| Baseline 3 | | | | | | | | | | | | | | |
| Candidate 1 | | | | | | | | | | | | | | |
| Candidate 2 | | | | | | | | | | | | | | |

Use the same percentile calculation and record whether client times include API/network latency. Distinguish observed billing fields from estimates calculated using current pricing.

## Evidence checklist

- [ ] Lab 09 was active and explicit provisioning/workload approval was recorded before AWS resource creation or deployed API traffic.
- [ ] Approved account identity, region, resource plan, maximum spend, workload bounds, duration, and cleanup plan are recorded (sanitize identifiers in shared evidence).
- [ ] Source revision, deployed settings, synthetic dataset, and test procedure are reproducible.
- [ ] Three or more baseline runs and at least two memory configurations are compared.
- [ ] Cache behavior, bound, TTL, freshness after controlled update, and per-environment scope are evidenced.
- [ ] Workload request/concurrency/time limits and reserved concurrency are verified.
- [ ] Logs and metrics are sanitized and have bounded dimensions.
- [ ] Final recommendation addresses latency, errors/throttles, freshness, cost, and measurement uncertainty.
- [ ] Resources were deleted or explicitly justified as shared/retained; absence and delayed billing review are recorded.
- [ ] Learner explains what further test or production telemetry would be required before rollout.
