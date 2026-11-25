# Lab 08 — Cleanup

Lab 08 creates a short-lived development API and observability resources only after activation and explicit provisioning approval. Do not delete pre-existing/shared SNS topics, subscriptions, log groups, dashboards, IAM roles, APIs, or AWS resources. Record the stack name, account/region, topic/subscription ownership, and resource names before cleanup.

## Before deletion

1. Stop test traffic and wait for the final deployment or rollback to finish. Cancel any active deployment deliberately; do not leave a deployment or load test running.
2. Save sanitized evidence required by `TESTS.md`, including the final source revision, stack status, alarm history, notification outcome, and resource inventory. Redact endpoints, email addresses, trace/log fields, and account details as appropriate.
3. Confirm the exact AWS account and region with the approved profile. Compare every resource against the Lab 08 stack, tags, and ownership record. Do not delete resources belonging to Labs 06–07 or another workload.

## Remove lab-created application and observability resources

1. Delete the Lab 08 development CloudFormation stack using its recorded exact name and region. If deletion is blocked by termination protection, retained resources, or a failed dependency, inspect and resolve it deliberately; do not force-delete shared resources.
2. Verify the stack is absent. Inspect for retained Lambda functions/versions, API Gateway API/stage, log groups, CloudWatch alarms/dashboard/custom metrics, X-Ray groups/sampling rules, and any SNS topic or subscription not removed by the stack.
3. Remove only Lab 08-owned leftovers after confirming ownership and that no other stack or application uses them. CloudWatch custom metric data expires according to AWS retention behavior; document any metric data that cannot be explicitly deleted.
4. Remove the SNS email subscription and topic only if they were created exclusively for this lab and no other publisher/subscriber depends on them. Otherwise document them as retained/shared and verify their approved scope.
5. Remove Lab 08-only IAM roles/policies only after checking for other attached resources or consumers. Never remove pre-existing or account-wide identities.
6. Verify the API endpoint no longer serves the lab workload and that no deployment is still active.

## Verification and cost follow-up

- Record before/after resource inventory and verify the development stack, API, Lambda, alarms, SNS resources (when lab-owned), and log groups are absent or explicitly documented as retained/shared.
- Confirm no further test traffic or deployment activity is running. Note that trace data and metric time series may remain available for their service-defined retention periods even after the producing resources are removed.
- Review the account cost/billing view after its normal reporting delay. An immediate resource inventory is not proof that no charges accrued.
- If temporary permissions or email authorization were granted, ask the administrator/subscriber to revoke them or verify the approved scope has expired. Do not revoke shared access.
- Inspect the repository and local workspace. Remove generated `.aws-sam/` builds, temporary failure fixtures, downloaded logs, trace exports, and packaged templates unless intentionally kept as sanitized teaching material. Keep only sanitized evidence.

## Completion record

- Lab 08 stack name and deletion status:
- API endpoint absent / verified:
- Lambda functions/versions and active deployments verified:
- Lab-owned alarms/dashboard/log groups verified:
- SNS topic/subscription deleted or retained with reason:
- X-Ray groups/sampling rules and telemetry retention noted:
- Lab-only IAM roles/policies removed or retained with reason:
- AWS caller/account and region (sanitized):
- Approved maximum, request/deployment counts, cost estimate, billing review date/source:
- Temporary permissions/notification authorization removed or confirmed expired:
- Local generated files cleaned:
- Cleanup verified by and date:
