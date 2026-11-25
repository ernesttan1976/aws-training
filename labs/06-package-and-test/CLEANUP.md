# Lab 06 — Cleanup

This assignment creates no running application resources. The only expected AWS activity is the explicitly approved S3 packaging upload. Do not delete or alter a shared bucket, object, policy, profile, or role.

## Before packaging

1. Confirm the selected bucket is dedicated to this lab or that the approved prefix is isolated and owned by the learner.
2. Record the exact bucket and key prefix, AWS account/role, region, artifact size, and approval. Do not run bucket-wide deletion commands against a shared bucket.
3. Use an artifact that contains only the built application and required dependencies. Never upload `.aws`, `.env*`, credential files, logs, test reports, or source fixtures that contain anything other than synthetic data.

## Remove the packaged object

After evidence collection, remove only the uploaded object(s) for this lab. Use the recorded exact bucket and key/prefix. For a versioned bucket, remove the specific object version(s) as well as any delete marker if required by the bucket owner. Do not run recursive deletion against a shared prefix without verifying it contains only this lab's artifacts.

Example for a dedicated bucket with one known object (replace placeholders only after verifying them):

```sh
aws s3 rm "s3://<approved-bucket>/<lab06-prefix>/<artifact-key>" \
  --region ap-southeast-1 --profile "<sandbox-profile>"
aws s3api head-object --bucket "<approved-bucket>" \
  --key "<lab06-prefix>/<artifact-key>" \
  --region ap-southeast-1 --profile "<sandbox-profile>"
```

The `head-object` call should report that the key is absent (for example, a not-found response). For versioned buckets, inspect and remove the exact object versions using the bucket owner's approved process; a simple delete may leave billable versions behind.

## Remove a lab-created bucket

Only if a dedicated bucket was explicitly created for Lab 06, verify that it contains no unrelated data, remove its remaining versions/delete markers if versioning was enabled, and then delete the bucket. Verify its absence with `aws s3api head-bucket` (a not-found result for the intended bucket). Do not delete a pre-existing/shared artifact bucket.

## Local cleanup

1. Remove the generated packaged template, scratch event/output files, logs, local environment files, and temporary test artifacts unless they are sanitized evidence intentionally retained outside source control.
2. Remove `.aws-sam/` build output when no longer needed; it is reproducible. Keep only project source, synthetic fixtures, and sanitized test reports.
3. Inspect `git status` and the repository diff. Confirm no credentials, private environment values, generated package artifacts, or local config files were added.
4. Stop any local `sam local start-api` process and Docker containers created for the lab.

## AWS verification and completion record

- Confirm there is no CloudFormation stack, Lambda function, API Gateway API, or other deployed application resource from this lab.
- Confirm only the approved S3 object/prefix was touched and the artifact is absent after cleanup; if the lab created a bucket, confirm the bucket is absent.
- Review account billing/usage after the normal reporting delay. Do not state that charges are zero based only on immediate console/CLI results.
- If a lab-specific permission was granted by an administrator, ask them to remove it or confirm that the approved temporary session has expired. Do not remove pre-existing/shared access.

Record:

- Artifact object(s) removed and verified:
- Lab-created bucket removed and verified / not created:
- CloudFormation/application resources created (expected none):
- Lab-only permission removed or expired / not applicable:
- AWS caller/account and region (sanitized):
- Upload count, estimated cost, and billing review date/source:
- Local generated files and containers cleaned:
- Cleanup verified by and date:
