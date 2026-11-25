# Validation — Lab 02: Data access patterns and lifecycle management

Record commands/output, sanitized AWS CLI responses, timestamps, and the app version/commit for each check. Redact account IDs, bucket names, report content, and presigned URLs where appropriate. Never submit credentials. Runtime evidence is required; template inspection alone is insufficient.

| Criterion | Validation | Evidence to capture |
|---|---|---|
| AC1 — Access-pattern fit | Create reports in at least two statuses with distinct timestamps. Run ID lookup and status browse. Confirm status results are newest first. Inspect application command paths and DynamoDB request/CloudTrail or SDK debug evidence where available to confirm `GetItem`/`Query`, not `Scan`. | Sanitized command output showing newest-first ordering, relevant code path, table/index description, and request evidence showing the queried index/key condition. |
| AC2 — Separation of data | Create a report with an attachment. Confirm metadata is in DynamoDB and the object exists in S3. Read it using the app or a short-lived presigned URL, then verify anonymous/public access is denied. | Sanitized DynamoDB item (no sensitive message), `head-object` metadata, successful authorized read, and failed anonymous request or public-access configuration. |
| AC3 — Expiry semantics | Create or seed an item with an expiry timestamp in the past. Confirm application lookup/list omits or rejects it even while `GetItem` can still retrieve the underlying record. Inspect TTL configuration and confirm the configured attribute is `expiresAt`. | App output plus direct item evidence, `describe-time-to-live` result, and explanation that TTL deletion is asynchronous (may take days). |
| AC4 — Object lifecycle | Inspect the bucket lifecycle configuration and confirm the enabled expiration rule targets only the lab attachment prefix with the required window. Do not expect physical expiry during this test. | Sanitized `get-bucket-lifecycle-configuration` output and the rule's ID, prefix/filter, status, and expiration days. |
| AC5 — Security and safe operation | Inspect table/bucket policies and runtime identity permissions; verify bucket public access blocks and default encryption. Compare deployed resources with the approved plan, scan the Git diff for secrets, then run cleanup. | Sanitized IAM/resource-policy output, `get-public-access-block`, encryption/lifecycle settings, identity evidence, secret-scan or review output, and post-cleanup verification. |
| AC6 — Safe create/read behavior | Attempt to create the same report ID twice, then retrieve a nonexistent ID. Confirm the first report remains unchanged and the missing result is explicit. | Sanitized outputs for first create, duplicate create, and missing lookup, plus evidence of conditional create behavior. |
| AC7 — Reasoning | Explain the key/index design, data boundary, duplicate/missing behavior, expiry, retention, and one cost/consistency trade-off. | Learner's short written or spoken explanation and assessor notes. |

## Suggested manual AWS CLI checks

Use the approved profile and region. Substitute stack outputs; never paste live secrets or presigned URLs into shared notes.

```sh
aws sts get-caller-identity --profile <sandbox-profile>
aws cloudformation describe-stacks --stack-name <stack-name> --profile <sandbox-profile> --region ap-southeast-1
aws dynamodb describe-table --table-name <table-name> --profile <sandbox-profile> --region ap-southeast-1
aws dynamodb describe-time-to-live --table-name <table-name> --profile <sandbox-profile> --region ap-southeast-1
aws dynamodb get-item --table-name <table-name> --key '{"reportId":{"S":"<test-report-id>"}}' --profile <sandbox-profile> --region ap-southeast-1
aws s3api get-public-access-block --bucket <bucket-name> --profile <sandbox-profile> --region ap-southeast-1
aws s3api get-bucket-encryption --bucket <bucket-name> --profile <sandbox-profile> --region ap-southeast-1
aws s3api get-bucket-lifecycle-configuration --bucket <bucket-name> --profile <sandbox-profile> --region ap-southeast-1
aws s3api head-object --bucket <bucket-name> --key <attachment-key> --profile <sandbox-profile> --region ap-southeast-1
```

For the status query, run the app's `Query` path or an equivalent `aws dynamodb query` with the actual index name and key condition. Do not use a scan as the validation shortcut. DynamoDB TTL and S3 lifecycle deletion are eventual/asynchronous; configuration and application behavior are the acceptance evidence for this short lab.

## Learner evidence and assessment

- AC1:
- AC2:
- AC3:
- AC4:
- AC5:
- AC6:
- AC7 — learner's design and trade-off explanation:
- Assessor notes / gaps to reinforce:
