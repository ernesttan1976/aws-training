# Cleanup — Lab 02: Data access patterns and lifecycle management

**Run immediately after collecting evidence.** Use the same approved AWS profile, stack name, and region used for deployment. Confirm names before deleting anything.

1. Stop the application and any test loops. Do not leave test processes uploading or querying data.
2. List the stack resources and confirm the stack is dedicated to this lab. Empty the lab bucket, including all object versions or delete markers if versioning was enabled, before deleting the stack. Do not empty or delete a shared bucket.

   ```sh
   aws s3 rm s3://<bucket-name> --recursive \
     --profile <sandbox-profile> --region ap-southeast-1
   aws cloudformation delete-stack --stack-name <stack-name> \
     --profile <sandbox-profile> --region ap-southeast-1
   aws cloudformation wait stack-delete-complete --stack-name <stack-name> \
     --profile <sandbox-profile> --region ap-southeast-1
   ```

   If bucket versioning was enabled, remove all versions and delete markers using the S3 API before deleting the stack. This lab does not require versioning.
3. Verify the CloudFormation stack is gone and the DynamoDB table, S3 bucket, and any lab-only deployment resources are absent. If stack deletion fails, inspect the stack events, resolve the cause, and retry cleanup.
4. If SAM created a dedicated deployment-artifact bucket outside the stack, empty and delete it only after verifying it is exclusive to this lab. Do not delete a shared artifact bucket.
5. Remove local test files and generated deployment artifacts if they contain report data. Keep source code and sanitized evidence; do not retain credentials or presigned URLs.
6. Review Cost Explorer or the sandbox billing view after usage data becomes available. Billing data can lag, so immediate absence of a charge is not final confirmation.
7. Record the cleanup time and any remaining resources below. Do not mark cleanup complete while a lab resource remains.

## Post-cleanup verification

- Stack name / region:
- Stack deletion completed at:
- DynamoDB table absent:
- S3 bucket absent:
- Lab-only SAM deployment-artifact bucket absent (or reason retained):
- Other lab-only deployment artifacts absent (or reason retained):
- Local report/attachment test data removed:
- Remaining resources / follow-up:
