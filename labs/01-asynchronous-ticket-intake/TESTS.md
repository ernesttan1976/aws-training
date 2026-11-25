# Validation — Lab 01: Reliable asynchronous ticket intake

Record the command/output, AWS CLI response, log evidence, and timestamp for each check. Redact account-sensitive identifiers where appropriate. Do not submit credentials.

| Criterion | Validation | Evidence to capture |
|---|---|---|
| AC1 — Asynchronous producer | Run the producer with one valid ticket. Confirm the producer only calls SQS; then observe the work queue's approximate visible/in-flight counts and the Lambda invocation. | Producer output, relevant `aws sqs get-queue-attributes` result, and code path showing `SendMessage` rather than direct Lambda invocation. |
| AC2 — Successful processing | Wait for the event source mapping to invoke the function. Inspect the function's log group and confirm the valid ticket is removed from the work queue. | Structured acceptance log entry with `ticketId` and event time; queue attributes before/after or the CloudWatch `NumberOfMessagesDeleted` metric. |
| AC3 — Bounded failure | Send a malformed ticket. Wait through the configured visibility/retry intervals. Inspect the DLQ and confirm the invalid message arrives there while a separate valid ticket still succeeds. | Configured `maxReceiveCount`, DLQ message count or received message, and evidence that the valid message processed. Allow for eventual consistency. |
| AC4 — Security | Inspect the producer identity/policy and Lambda execution role policies. Search the repository and staged diff for accidental credentials. | Sanitized IAM policy output; `git status`/secret scan output; explanation of why each action/resource is present. |
| AC5 — Safe operation | Verify resource region and stack contents against the approved plan; check current cost/pricing assumptions; run the cleanup procedure. | Stack/resource listing before deletion and empty/absent resource verification after deletion; cost estimate/usage evidence if available. |

## Suggested manual checks

Use the profile and region explicitly so evidence is reproducible. Adapt names to your deployment outputs:

```sh
aws sts get-caller-identity --profile <sandbox-profile>
aws configure get region --profile <sandbox-profile>
aws cloudformation describe-stacks --stack-name <stack-name> --profile <sandbox-profile> --region ap-southeast-1
aws lambda get-function-configuration --function-name <function-name> --profile <sandbox-profile> --region ap-southeast-1
aws lambda list-event-source-mappings --function-name <function-name> --profile <sandbox-profile> --region ap-southeast-1
aws sqs get-queue-attributes --queue-url <queue-url> --attribute-names All --profile <sandbox-profile> --region ap-southeast-1
```

Do not paste a live queue URL or account ID into public evidence. Confirm actual queue behavior and log output; template inspection alone does not satisfy the runtime criteria.

## Learner evidence and assessment

- AC1:
- AC2:
- AC3:
- AC4:
- AC5:
- Learner's explanation of retry/acknowledgement and permission choices:
- Assessor notes / gaps to reinforce:
