# Lab 07 — Cleanup

Lab 07 creates an active pipeline and multiple AWS resources only after activation and explicit provisioning approval. Do not delete pre-existing/shared repositories, CodeConnections, buckets, IAM roles, log groups, or AWS resources. Record the stack names, connection ownership, bucket name/prefix, pipeline name, and account/region before cleanup begins.

## Before deletion

1. Stop new source commits and wait for all CodePipeline executions and CodeBuild builds to finish or cancel them using the console/CLI.
2. Record final pipeline execution IDs, artifact keys/digests, deployed stack names, Lambda alias/version, alarm state, and approved cost plan.
3. Confirm the exact AWS account and region using the approved profile. Check resource ownership and ensure no other project uses the pipeline, artifact bucket, roles, connection, or stacks.
4. Save sanitized evidence required by `TESTS.md`. Do not retain access tokens, environment dumps, or build logs containing sensitive values.

## Delete application resources

1. Delete the development and production application CloudFormation stacks using the approved stack names and region. If a stack has termination protection, retained resources, or a deletion failure, inspect and resolve it deliberately; do not force-delete shared resources.
2. Verify both stacks are absent and inspect for retained Lambda functions/versions, API Gateway APIs, CodeDeploy application/deployment group, CloudWatch alarms, and log groups. Remove only lab-owned leftovers that the stack did not delete.
3. Verify the intended application endpoints are no longer serving the lab API.

## Delete pipeline and artifacts

1. Delete the CodePipeline pipeline and any lab-created CodeBuild project after confirming no run is active.
2. Delete the pipeline/bootstrap CloudFormation stack after its dependent pipeline resources have been removed, following its dependency order if deletion is blocked.
3. Remove only this lab's S3 artifact objects and all versions/delete markers if bucket versioning is enabled. Inspect the bucket contents first; never issue a recursive delete against a shared bucket.
4. Delete the artifact bucket only if it was created exclusively for this lab, is empty (including versions), and its ownership is confirmed. Verify absence with `aws s3api head-bucket` or the S3 console.
5. Remove a CodeConnections connection only if it was created exclusively for Lab 07, is unused by other pipelines, and deletion is permitted. Otherwise document it as retained/shared and confirm its authorization scope.
6. Remove other lab-only CodeBuild/CodePipeline/CloudFormation roles and policies only when their ownership and lack of consumers are confirmed. Never delete account-wide or pre-existing roles.

## Verification and cost follow-up

- Confirm the application stacks, pipeline, CodeBuild project, lab-owned alarms/log groups, and artifacts are absent. Record any intentionally retained shared resources and why.
- Confirm no pipeline execution or build is still running and no endpoint from this lab remains accessible.
- Review the account's cost/billing view after its normal reporting delay. A resource inventory immediately after cleanup is not proof that no charges accrued.
- If a temporary permission or GitHub connection authorization was granted, ask the administrator/repository owner to remove it or verify its approved scope; do not revoke shared access.
- Inspect `git status` and repository contents. Keep sanitized source/evidence only; remove local `.aws-sam/` builds, logs, packaged templates, downloaded artifacts, and temporary synthetic failure fixtures unless intentionally retained as sanitized test material.

## Completion record

- Application stack names removed and verified:
- Pipeline and CodeBuild project removed and verified:
- Artifact objects/versions removed and verified:
- Artifact bucket removed / pre-existing shared bucket retained:
- CodeConnections connection removed / retained and reason:
- Lab-owned alarms/log groups/deployment resources verified:
- Deployed application endpoints absent:
- AWS caller/account and region (sanitized):
- Approved maximum, actual run count, cost estimate, billing review date/source:
- Temporary permissions/authorization removed or confirmed expired:
- Local generated files cleaned:
- Cleanup verified by and date:
