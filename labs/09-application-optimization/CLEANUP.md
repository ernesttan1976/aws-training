# Lab 09 — Cleanup

Lab 09 creates a short-lived API, Lambda function, DynamoDB table, and supporting resources only after activation and explicit provisioning approval. Do not delete pre-existing/shared resources. Record the exact stack name, AWS profile/identity, region, and ownership before cleanup.

## Before deletion

1. Stop the workload client. Confirm no invocation loop, local process, deployment, or CloudFormation operation is still running.
2. Save the sanitized evidence required by `TESTS.md`: final source revision, final configuration, run ledger, cost estimate, data-freshness result, and resource inventory. Do not preserve secrets or unnecessary raw records.
3. Confirm the active profile, account, region, and exact Lab 09 stack name. Compare the inventory to the stack's tags/resources. Do not delete resources from another lab or any shared stack.
4. Record the table contents count and confirm the seeded records contain synthetic lab data only. Preserve no customer data because none is permitted.

## Remove lab-created resources

Use the exact stack name and region recorded above. Do not paste credentials into commands or logs.

```sh
aws cloudformation delete-stack \
  --stack-name "$LAB09_STACK_NAME" \
  --region "$AWS_REGION" \
  --profile "$AWS_PROFILE"

aws cloudformation wait stack-delete-complete \
  --stack-name "$LAB09_STACK_NAME" \
  --region "$AWS_REGION" \
  --profile "$AWS_PROFILE"
```

If the stack uses a different approved deployment method, use that method's deletion workflow instead. If deletion fails, inspect stack events and dependencies, then resolve only Lab 09-owned resources deliberately; never force-delete a shared dependency.

## Verify deletion and billing follow-up

1. Confirm the stack no longer exists with `aws cloudformation describe-stacks` for the exact stack name. A `ValidationError` stating that the stack does not exist is expected after successful deletion; record the sanitized result.
2. Inspect the approved region for Lab 09-owned API Gateway APIs/stages, Lambda functions/versions, DynamoDB table, log groups, IAM roles/policies, and any SAM artifact objects or bucket. Verify absence or record a specific justified shared/retained item and its owner.
3. Confirm the endpoint no longer serves the lab API and no test/deployment process remains active.
4. Review CloudWatch logs and billing/cost data after the normal reporting delay. Resource absence immediately after deletion does not prove that no charges accrued; record the date and source of the delayed review.
5. Remove local `.aws-sam/` build output, temporary workload results, cached test data, logs, and packaged templates unless intentionally retained as sanitized learning evidence. Never remove local AWS credentials as part of this lab.

## Completion record

- Lab 09 stack name and deletion status:
- API endpoint absent / verified:
- Lambda function/versions and reserved concurrency verified:
- DynamoDB table absent / verified:
- Lab-owned log groups and IAM resources absent or documented as retained/shared:
- SAM artifacts/bucket removed or retained with owner/reason:
- AWS caller/account and region (sanitized):
- Approved maximum spend, actual request count, run count, and cost estimate:
- Delayed billing review date/source:
- Local generated files cleaned:
- Cleanup verified by and date:
