# Cleanup — Lab 01: Reliable asynchronous ticket intake

**Do this immediately after collecting evidence.** Use the same AWS profile and region that were approved for deployment.

1. Stop the producer and any test loop; do not leave messages or a worker running.
2. Delete the SAM/CloudFormation stack using the deployment tool or AWS CLI. For a CLI-created stack, confirm the exact stack name and region before running:

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

3. If you created a dedicated SAM deployment-artifact S3 bucket, empty and delete that bucket only after verifying it is exclusive to this lab. Do not delete a shared artifact bucket.
4. Verify the CloudFormation stack no longer exists and that the lab's Lambda function, CloudWatch log group, work queue, DLQ, event source mapping, and IAM role are absent. Check the S3 bucket separately if one was created.
5. Review Cost Explorer or the sandbox's billing view after usage data becomes available. Billing data can lag; absence of an immediate charge is not proof that all costs are final.
6. Record the cleanup timestamp and any remaining resources below. If deletion fails, resolve it before considering the lab cleaned up.

## Post-cleanup verification

- Stack name / region:
- Stack deletion completed at:
- Lambda and event source mapping absent:
- Lambda log group absent:
- Work queue and DLQ absent:
- Lab IAM role absent:
- Lab-only artifact bucket absent (or reason retained):
- Remaining resources / follow-up:
