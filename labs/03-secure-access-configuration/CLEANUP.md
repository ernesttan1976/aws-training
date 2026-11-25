# Cleanup — Lab 03: Secure application access and configuration

**Do this immediately after collecting evidence.** Use the same approved AWS profile and region used for deployment. Secret deletion is scheduled with a recovery window; the secret is not immediately erased.

1. Stop test invocations and any local process using the lab profile. Delete the local synthetic-secret file if it remains. It must be outside the repository; do not use a real credential in this exercise.
2. Delete the SAM/CloudFormation stack and wait for deletion to finish:

   ```sh
   aws cloudformation delete-stack \
     --stack-name <stack-name> \
     --profile <sandbox-profile> \
     --region ap-southeast-1
   aws cloudformation wait stack-delete-complete \
     --stack-name <stack-name> \
     --profile <sandbox-profile> \
     --region ap-southeast-1
   ```

3. Schedule deletion of the standalone lab secret with the shortest supported recovery window (7 days). Do not force-delete it without recovery if there is any chance the secret is shared or contains real data:

   ```sh
   aws secretsmanager delete-secret \
     --secret-id <secret-arn> \
     --recovery-window-in-days 7 \
     --profile <sandbox-profile> \
     --region ap-southeast-1
   ```

4. Verify the secret has a `DeletedDate` and is no longer available as an active secret. Note that it remains recoverable during the scheduled window and may continue to incur secret charges until deletion completes. If immediate removal is required, use the service's force-delete option only for this disposable fake-data secret, understanding that recovery is impossible.
5. Verify that the Lambda function, execution role, secret resource policy, log group, and CloudFormation stack are deleted. If you used a dedicated SAM artifact bucket, empty/delete it only after confirming it contains no shared artifacts. Do not delete a shared bucket.
6. Review the sandbox billing view when usage data becomes available; billing data can lag, and no immediate charge is not proof that costs are final. Confirm the secret's scheduled deletion and final removal after the recovery window.
7. Record cleanup timestamps and any remaining resource or cost follow-up below. Resolve deletion failures before marking cleanup complete.

## Post-cleanup verification

- Stack name / region:
- Stack deletion completed at:
- Lambda function and execution role absent:
- Secret resource policy removed with stack:
- Secret ARN and scheduled `DeletedDate`:
- Secret fully removed after recovery window:
- Lambda log group absent:
- Lab-only artifact bucket absent (or reason retained):
- Remaining resources / follow-up:
- Billing review date / observed usage:
