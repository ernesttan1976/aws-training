# Lab 02 — Data access patterns and lifecycle management

**Status:** Authored, planned — Lab 01 remains active pending learner evidence and assessment  
**Exercise on activation:** Level 1 — Guided implementation  
**Region:** `ap-southeast-1` (Singapore), unless a required feature is unavailable

## Learning objectives

- Translate concrete application queries into DynamoDB keys and indexes.
- Use AWS SDK for JavaScript v3 to write, retrieve, and query records without table scans.
- Keep binary attachments in S3 and reference them from DynamoDB metadata.
- Apply DynamoDB TTL and S3 lifecycle policies while accounting for their asynchronous behavior.
- Explain storage selection, data retention, least privilege, and cost trade-offs using runtime evidence.

## Scenario

A field-operations team submits short-lived reports with a small photo or document attachment. Staff need to retrieve a report by its ID and browse recent reports by status. The team retains reports only briefly, and attachments should be removed automatically after the retention window.

## Problem statement

Build a small Node.js command-line application called **Field Reports**. Store report metadata in one DynamoDB table and attachments in one private S3 bucket. Provision the resources using AWS SAM/CloudFormation. The app must create, retrieve, and list reports by status, and must treat expired reports as unavailable even while DynamoDB's background TTL deletion is pending.

This is a short-lived sandbox exercise, not a production records system. It does not require an API, frontend, Lambda runtime, database migration service, or a caching service.

## Prerequisite knowledge

- Node.js, npm, AWS SDK for JavaScript v3, basic JSON and command-line use.
- Basic AWS CLI and CloudFormation/SAM familiarity.
- An AWS sandbox account and a named CLI profile with permission to deploy and inspect this lab.
- Use the standard AWS credential provider chain; do not use root credentials, commit credentials, or create application access keys.
- Lab 01 is the current active lab. This package is prepared for later use; do not deploy it until the instructor activates it and you explicitly approve provisioning.

## Functional requirements

1. Provision one DynamoDB table with `reportId` as its string partition key and on-demand capacity, plus one S3 bucket. Block all public access to the bucket and enable server-side encryption.
2. Store a report record with a unique `reportId`, `status`, `createdAt`, `expiresAt`, and optional S3 object key. `expiresAt` must be a Unix epoch timestamp in **seconds**, suitable for DynamoDB TTL.
3. Implement the following application access patterns:
   - Create a report and optionally upload its attachment.
   - Retrieve a report by `reportId`.
   - List recent reports for a given `status`, newest first, using a DynamoDB `Query` against a suitable secondary index that supports the status key condition and time ordering; do not use `Scan` for this access pattern.
4. Treat an item with `expiresAt <= now` as expired in application behavior, even if DynamoDB still returns it. Do not rely on TTL as a synchronous delete operation.
5. Configure DynamoDB TTL for `expiresAt` and an S3 lifecycle expiration rule for the lab's attachment prefix. The lifecycle window is at least one day; do not claim that S3 deletes an object immediately when it becomes eligible.
6. Store attachment objects beneath the configured `attachments/` prefix. On a report-read request, return metadata and, when an attachment exists and the report is not expired, retrieve it or produce a short-lived presigned download URL. Never make the bucket public.
7. Keep table and bucket names, retention period, and region configurable. Use the AWS SDK v3 credential provider chain, not credentials in application configuration.
8. Create reports conditionally so a duplicate report ID does not silently overwrite an existing report. Handle a missing report explicitly and show the observed behavior for both cases.

## Non-functional requirements

- Use DynamoDB `GetItem` for ID lookup and `Query` for the status listing; do not perform full-table scans.
- Keep object keys non-sensitive and do not log attachment contents, credentials, or unnecessary report text.
- Restrict the application identity to the specific table/index and bucket/prefix operations it requires. Deployment permissions are separate from runtime application permissions.
- Keep the test small: no more than 100 reports and 25 attachments, each attachment at most 1 MiB.
- TTL and S3 lifecycle are asynchronous retention mechanisms. Runtime acceptance checks must test application-level expiry and inspect the actual service configuration, not wait for physical deletion during the lab.

## Constraints

- Use Node.js and AWS SDK for JavaScript v3 for the application; use AWS SAM or CloudFormation for resources, and AWS CLI for inspection.
- Use only DynamoDB, S3, and the deployment resources CloudFormation/SAM requires. Do not add API Gateway, Lambda, NAT gateways, DAX, ElastiCache, or other always-on resources.
- Use on-demand DynamoDB capacity. Keep the S3 bucket private, encrypted, and limited to the lab's attachment prefix.
- Prefer `ap-southeast-1`. Stop if a required feature is unavailable rather than silently switching regions.
- No billable resources may be deployed without the learner's explicit approval. Inspect the change set/resource plan first.
- **Estimated duration:** 2–3 hours including design, deployment, validation, evidence, and cleanup.
- **Estimated cost:** approximately **$0–$0.25 USD** for the small test and short-lived deployment, assuming no data transfer out, no large objects, and prompt cleanup. This estimate varies with current regional prices, free-tier eligibility, and storage duration. Keep a **$1 USD maximum spend expectation**; stop if the estimate exceeds it. Do not create a paid budget/monitoring resource for this lab.

## Four exercise levels

### Level 1 — Guided implementation (first exercise when activated)

Implement the Field Reports requirements above. Follow the milestones below and use the Level 1 hints in `HINTS.md`. Explain why each query can be served by the chosen key/index design and distinguish logical expiry from physical deletion.

### Level 2 — Independent implementation

After Level 1 is assessed, independently design a short-lived **equipment checkout** service with the same storage boundaries: searchable item metadata, an access pattern to find recent records by a non-primary attribute, and attachments retained for a stated period. Use a fresh schema and deployment definition; do not copy the Level 1 table/index layout. Submit your access-pattern table and justify every index before deploying. The learner receives requirements and constraints without a solution design.

### Level 3 — Troubleshooting

After Level 2 is assessed, diagnose a supplied deliberately faulty Field Reports implementation. It will include observable symptoms involving incorrect query behavior, expiry/retention, or attachment access. Identify the cause of each symptom, repair the smallest set of changes, and prove the fix with service-side evidence. The faulty starter, symptom set, and focused hints will be issued when this level begins; no fix is provided in advance.

### Level 4 — Optimization

After Level 3 is assessed, optimize a working implementation against a supplied workload and baseline evidence. Improve at least one measured access-path, latency, or cost characteristic without weakening privacy or retention guarantees. Compare before/after results, account for index/storage/write amplification, and justify whether a cache is warranted. Do not add a paid caching service; demonstrate any cache with a local/in-process option or a measured design proposal.

## Guided implementation milestones — Level 1

1. **Model the access patterns:** write a table showing the caller, input, result, expected frequency, and consistency needs for create, lookup by ID, and recent reports by status. Sketch keys and index attributes before writing infrastructure.
2. **Select boundaries:** decide which fields belong in DynamoDB and why attachments belong in S3. Define the meaning of `expiresAt` and how the app will behave while physical deletion is pending.
3. **Implement:** create the SAM/CloudFormation resources, Node.js SDK v3 commands, validation, expiry check, and error handling. Keep runtime permissions scoped to the resources/prefixes and operations used.
4. **Review before deployment:** inspect the template/change set, region, resource list, public-access controls, lifecycle/TTL settings, estimated cost, and cleanup plan. Request explicit provisioning approval before deployment.
5. **Deploy and exercise:** create reports in at least two statuses, add one small attachment, query by ID and status, test a duplicate ID and missing ID, and test expired-item behavior without waiting for TTL deletion.
6. **Prove and clean up:** complete every validation in `TESTS.md`, capture sanitized evidence, then follow `CLEANUP.md` and verify resources are removed.

## Acceptance criteria

1. **AC1 — Access-pattern fit:** lookup by ID uses the primary key; recent reports by status use a DynamoDB `Query` against an appropriate index and are returned newest first. Evidence shows no table scan for either path.
2. **AC2 — Separation of data:** metadata is stored in DynamoDB; an attachment is stored in the private S3 bucket and referenced by object key. An authorized read works without making the bucket public.
3. **AC3 — Expiry semantics:** the app rejects/omits a record whose expiry has passed even when the item remains physically present. DynamoDB TTL is enabled for the correct seconds-based attribute, and its asynchronous nature is explained.
4. **AC4 — Object lifecycle:** an S3 lifecycle rule scopes expiration to the lab attachment prefix with the configured retention window. The rule is verified in AWS; no immediate deletion claim is made.
5. **AC5 — Security and safe operation:** public access is blocked, encryption is enabled, runtime permissions are least-privilege, no credentials are committed, deployed resources match approved scope/region, and cleanup verification confirms removal.
6. **AC6 — Safe create/read behavior:** a duplicate ID does not overwrite the existing report, and a missing ID is handled explicitly.
7. **AC7 — Reasoning:** the learner explains the access patterns, key/index choice, DynamoDB-versus-S3 boundary, duplicate/missing behavior, retention timing, and a relevant cost/consistency trade-off.

## Official references

- [DVA-C03 exam guide — Domain 1: Development with AWS Services](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03-domain1.html)
- [Amazon DynamoDB Developer Guide](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html)
- [DynamoDB key design and data modeling](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-modeling-nosql.html)
- [DynamoDB Query operation](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Query.html)
- [DynamoDB secondary indexes](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/SecondaryIndexes.html)
- [DynamoDB TTL](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/TTL.html)
- [DynamoDB read consistency](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadConsistency.html)
- [Amazon S3 lifecycle management](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html)
- [Amazon S3 Block Public Access](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html)
- [Amazon S3 presigned URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/ShareObjectPreSignedURL.html)
- [AWS SDK for JavaScript v3 — DynamoDB examples](https://docs.aws.amazon.com/sdk-for-javascript/v3/developer-guide/javascript_dynamodb_code_examples.html)
- [AWS SDK for JavaScript v3 — S3 examples](https://docs.aws.amazon.com/sdk-for-javascript/v3/developer-guide/javascript_s3_code_examples.html)
- [DynamoDB pricing](https://aws.amazon.com/dynamodb/pricing/) · [S3 pricing](https://aws.amazon.com/s3/pricing/)

## Cost and duration estimate

The estimate and assumptions are in **Constraints**. Check current `ap-southeast-1` prices and the deployment change set before requesting approval. Actual cost is not guaranteed by the estimate.

## DVA-C03 mapping

- **Domain 1 — Development with AWS Services** (current exam-guide weighting; check the official guide for updates).
- **Task 1.3 — Use data stores during application development.** Focus on Skills 1.3.1 (data lifecycle), 1.3.2 (choose data stores based on access patterns), 1.3.3 (determine a caching solution), and 1.3.4 (configure DynamoDB for application access patterns, consistency, queries, keys, and indexes). The Level 4 optimization exercise explicitly considers whether caching is warranted.
- Practical extension: reliability, least-privilege access, retention, and cost-aware service selection.
